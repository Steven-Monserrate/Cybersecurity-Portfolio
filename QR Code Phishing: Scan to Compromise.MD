#This lab is from SOC Simulator

#Summary: In this lab, I investigated a modern 'Quishing' (QR phishing) attack that bypassed traditional email filters by hiding its payload inside of an image. I traced the full chain: a spoofed MFA-enrollment lure sent from purpose-built infrastructure, a redirect server that concealed the final destination, and an Evilginx-style adversary-in-the-middle page that stole an authenticated session cookie despite MFA. I then followed the attacker's post-compromised moves (Graph API mailbox enumeration, SharePoint exfiltration, and a hidden inbox forwarding rule) and chose the containment action that actually evicted them.

#Quishing (QR-code phishing) - A social engineering technique where attackers embed a malicious URL inside a QR code image. When the victim scans it with their phone's camera, they are taken to a credential-harvesting page that mimics a trusted login portal. Because the destination is hidden inside an image, it slips past defense that only scan text and hyperlinks.
  #NOTE: Why is quishing so effective? QR codes are opaque: a human cannot read the encoded URL by looking at the image, and many email gateways treat the QR as a harmless picture. Worse, the victim usually scans it on a personal phone, which sits outside corporate web proxies, DNS filtering, and endpoint protection.

#Scenario: Around 47 employees received an email titled "Action Required: MFA Enrollment" claiming to be from the IT department. The email body contained a QR code instead of any link text. At least 3 users scanned the QR code on their phones. One account is confirmed compromised: sarah.jenkins (workstation corp-wks-7720). Shortly after the scan, Azure AD flagged a suspicious sign-in for sarah.jenkins, followed by Microsoft Graph API activity (mailbox access and SharePoint downloads).

#Available SIEM sources: Cloud-azure-signin, email-gateway, email-o365, sec-edr, sec-ids, windows-security, windows-sysmon. No indicators of compromised were presented at this time: finding the attacker's IPs and domains is the primary objective.

#When sarah.jenkins scanned the QR code on her phone, the encoded URL did not point directly to the credential harvesting page. Instead, attackers used a redirect chain, a common technique to evade URL scanners. The first hop in the chain was a redirect server that immediately forwarded the victim to the next stage.

#Our email gateway extracted the QR code image and decoded the embedded URL. Additionally, our IDS flagged suspicious redirect activity.

#NOTE: Why this matters: a redirect chain buys the operator resilience. Distribution and destination are decoupled, so when a defender burns the landing page every lure already printed, emailed or posted keeps working; the operator simply re-points the hop in the middle. You will meet this in production as an attribution problem, because the piece of infrastructure you can enumerate is rarely the piece that has to die for the campaign to stop.
#NOTE: Why quishing is so dangerous: sarah.jenkins scanned the QR code with her personal phone, not her corporate laptop. That moves the attack onto a device outside the corporate security perimeter: no web proxy, no endpoint EDR, and no corporate DNS filtering to block the malicious URL. Notice the IDS even logs the DNS query as coming from an external mobile device, not corporate DNS. This device shift is the core advantage of QR phishing: the SOC often sees only the email arriving and the eventual identity compromise in Azure AD, never the malicious page load itself. In this case we got lucky: her phone had joined the corporate guest Wi-Fi, so the perimeter IDS captured the redirect chain you are tracing here.

#What is the IP address of the redirect server, the first hop in the phishing chain that the QR code URL points to?
  #176.111.219.85

#Now that I found the first hop, I moved on to finding the next one. The 302 response from the redirect server sent sarah.jenkins to a credential harvesting page that closely mimicked the corporate Microsoft 365 login portal. The IDS and Azure AD sign-in logs captured what happened next.

#The page redirect landed her on an adversary-in-the-middle proxy. It relayed sarah.jenkins's login to the real Microsoft endpoint in real time, so she saw a genuine sign-in flow. After she entered her password and approved the MFA prompt on the real Microsoft page, the proxy kept what Microsoft issued at the end of it. Whoever held that artifact would be able to return without her password and without a fresh MFA code.

#Why this matters: An adversary-in-the-middle proxy defeats the control most organizations treat as final. The victim genuinely authenticates to Microsoft, the MFA prompt is genuinely approved, and the one-time code is spent the moment it is used, so it is not what the operator walks away with. Any sign-in that succeeds after this point deserves the question 'is this her, or is this the proxy's souvenir?'.

#sarah.jenkin's browser submitted her login credentials to 185.234.72.19, the attacker's server

#This is only part of the attacker's infrastructure. Once the proxy had what Microsoft issued at the end of her sign-in, the attacker switched to a different server to access Microsoft services with Sarah's stolen identity. This operational separation, one set of servers for phishing and another for post-compromise activity, is a hallmark of sophisticated campaigns.

#Post compromise 

