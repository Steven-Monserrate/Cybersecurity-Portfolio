#This lab is from SOC Simulator

#A finance analyst at MegaCorp Logistics reported that colleagues were receiving multiple replies to messages that she never sent, and her account was disabled while the investigation was running. I reviewed SIEM feeds and endpoint XDR telemetry for the workstation estate and the mail platform, covering the week around the report. I reconstructed what happened, producing endpoint evidence, what its browser reached, how the account was taken over, and what was left behind on the mail platform.

#Summary - This case began as a mailbox compromise on a finance analyst's account. Before tracing how the intrusion unfolded, I needed to scope it and determine which internal workstation produced the endpoint telemetry behind the incident.

#The first workstation identified as compromised was corp-wks-082, amanda.garcia

(PHOTO)

#Now that I knew which workstation was affected, I followed what its browser did during the duration of the attack. Shortly after the user opened a downloaded file, the host opened an HTTPS connection to an external host.

#I noticed in the SIEM logs that amanda.garcia's account was originally in Boston (08:30:40), but then a log in was successfully completed in Frankfurt, DE only 2 hours later (10:18:45). This constituted impossible travel. The logs recorded that once ywnjb.com was accessed at 10:16:11, a false login attempt was successful.

(PHOTO)

#The connection that I traced raised a detection on the affected host in the XDR Behaviors panel. My next step was to report the MITRE ATT&CK ID the platform mapped to that detection.

#In this instance, the attack pattern was T1557 - AiTM. AiTM are intercept attacks that stand between legitimate processes and steal information. When amanda.garcia went to the fraudulent website, it appeared legit, but was actually a reroute to the attacker's server. The attacker then puts them through to the legitimate page that they're looking for, but already stole essential login information.

#A second detection was noticed on the same host a few minutes after the first, on the artifact the browser was holding at that point. T1539. Steal Web Session Cookie, was the essential information that was stolen during the AiTM earlier. With this, the attacker could log in as amanda.garcia, appear legitimate, avoid MFA, and have access to her account.

#Both T1557 and T1539 are credential access attacks, as listed in MITRE. Understanding this helped me to determine the best course of action in expelling the attacker.
  #https://attack.mitre.org/tactics/TA0006/

#With the endpoint activity determined, I went back to how it all started. A file arrived via email and was opened from amanda.garcia's Downloads folder shortly before the browser session.

#According to the SIEM logs, invoice.svg was the malicious attachment.

(PHOTO)

#I reviewed the XDR tool and found that msedge.exe was the original file that began the attack. Once opened, it launched a headless automation binary that ran on the same host.

(PHOTO)

#The attacker tried to utilize MITRE ATT&CK T1114.003, an email forwarding rule, so that they could gain persistent access to amanda.garcia's account. Once that trait was discovered, I removed the rule and ended the session token, thus removing access to the account from the attacker. I also made sure to do the same for all other affected accounts, guaranteeing that the attacker couldn't gain access through them.

https://www.socsimulator.com/achievements/room/evilginx-aitm-session-cookie-hijack?ref=share&via=sheepdogcyber&utm_source=copy&utm_medium=social
