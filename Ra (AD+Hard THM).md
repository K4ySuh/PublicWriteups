
---
Skills: Web enumeration, SMB, Phishing, Active-Directory, Data-exfiltration.
---

# Enumeration (Recon phase):

During basic enumeration it seems like I have found a possible subdomain or maybe the name of the machine, while using **whatweb: fire.windcorp.thm**. Also I've found some e-mails too. ![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/366db624-c0e2-4f51-96dc-bc2cd82febeb)

## Nmap results:

After launching nmap I got a quite large amount of services. Let's look closer into them:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/43ebee4c-a004-49bb-ad9c-e291bb7948fa)
Some of the typical services we'll find in an AD environment. Here I can see LDAP, RDP, SMB. We find the possible subdomain it's nothing but the name of the computer in the windcorp.thm domain. 
Next I've found jabber which resulted to be an extension of XML utilized to send information like messages for example. I found a possible exploit but this version is not old enough for it to work.![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/6dab3177-8a55-43ae-8cf1-d213e212ba00)
Then I found WinRM open and some http ports open hosting a java-based webserver using the application called **jetty**. It acts as a java web-server or servlet-container according to google results:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/9fd89f8e-2acb-4cfb-a5be-7cec4714caf9)
The remainings of the scan are RCP ports and nothing of interest.

# Possible attacks (Enumeration):

I have found a decent amount of services that could be attack vectors, let's enumerate a bit further and try to get more details about the services running on this machine:
## Port 80 HTTP web server.

In port 80 I've found what it seems to be the main website of the enterprise: ![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/d9ad8ef3-91c6-4988-a55d-01f0ea6f21dd)
After reviewing the site I could find out what Openfire was about and what were, those ports using jabber, doing. It is a service combined with Spark which helps the support team to exchange messages,calls,etc through a safe IM platform. Put it this way: Jabber uses XMPP protocol, Openfire is the server and Spark is the application. All 3 combined conform the tool used for technical support in this enterprise. 
I've had a look over the few links and tools that are available on the website, one of them was the reset password blue button:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/67a0a8a7-ad0d-4e64-8848-b15b245f3c56)
I found this interesting, maybe I could use some of the usernames I've found before with **whatweb** and try to get access somewhere (?). But there's also another finding, since it's using IIS technology the extension of the files running in the server are most likely gonna be **.asp** and after confirming checking the reset-password view I've performed a gobuster scan on the website:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/7996e44a-d9aa-4b35-bd0a-19daddfdcf09)Sadly I couldn't find anything ussefull at all.


