
---
Skills: Web enumeration, SMB, Phishing, Active-Directory, Data-exfiltration.
---

# Enumeration (Recon phase):

During basic enumeration it seems like I have found a possible subdomain or maybe the name of the machine, while using **whatweb: fire.windcorp.thm**. Also I've found some e-mails too. ![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/366db624-c0e2-4f51-96dc-bc2cd82febeb)

## Nmap results:

After launching nmap I got a quite large amount of services. Let's look closer into them:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/43ebee4c-a004-49bb-ad9c-e291bb7948fa)
Some of the typical services we'll find in an AD environment. Here I can see LDAP, RDP, SMB. We find the possible subdomain it's nothing but the name of the computer in the windcorp.thm domain. 
Next I've found jabber which resulted to be an extension of XML utilized to send information between. I found a possible exploit but this version is not old enough for it to work.![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/6dab3177-8a55-43ae-8cf1-d213e212ba00)
Then I found WinRM open and some http ports open hosting a java-based webserver using the application called **jetty**. It acts as a java web-server or servlet-container according to google results:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/9fd89f8e-2acb-4cfb-a5be-7cec4714caf9)


