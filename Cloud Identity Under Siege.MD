#This lab is from SOC Simulator

#Scenario: Over the span of nine days, telemetry from a cloud-first Azure estate: Entra ID sign-ins, Windows endpoints, a Kubernetes cluster, an Azure Function, and the storage account behind them all reported into one SIEM. However, something inside that window of time didn't fit the shape the estate normally has. In this lab, I worked with the SIEM, XDR, firewall, and cloud panels, pinned each observation to a host and a principal, and rebuilt the sequence from the records.

#Before I began separating anomalous sign-ins from normal activity, I needed a reliable signal for what a clean, successful interactive logon looked like in the Windows security log. I examined marcus.aurelius's early sign-ins on corp-wks-8821 at the start of the window and determined the numeric Windows event identifier that records a successful logon.

