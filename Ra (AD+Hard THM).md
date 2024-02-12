
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
### BurpSuite

I've decided to have a closer look at how the requests were being sent to the reset password file:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/4de50a2b-b916-45a2-ab84-88e92d7f720c)
That form could potentially be vulnerable to pitchfork attacks using Burpsuite or maybe other kind of bruteforce attacks to reset the password assuming the usernames we listed before were valid.
## Port 7070, technical support IM technologies.
On the other hand, **port 7070** was only a website with a link pointing to what **HTTP Binding and BOSH technology were and how do they use XMPP protocol**:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/435f69b9-fd09-4a87-a239-2ecc4a9b9400)
I was curious about this and I thought it could be a possible lead to find the vulnerability in this machine. However I wanted to solve this as a real scenario, I focused on further enumeration instead of tunnel-visioning the finding which, anyways, could be baiting me into a rabbit-hole.
## Port 5985: WinRM

WinRM allows the user to execute commands on the Windows server and it does require authentication, some tools like **evil-winrm** allow to perform attacks like pash the hash to gain remote control with a shell. The other way to perform an attack against this service is to bruteforce using **crackmapexec** with the users I have previously found and also adding some built-in accounts like Administrator just in case. So I created a list of users with the e-mails gathered before and started the brute-force attack:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/607033fb-5dd8-476b-95bd-a50db768ff6f)
**Note**: After reviewing this explanation I have noticed this is not the best way to proceed but I was doing this writeup as I was solving the machine. You should always use **kerbrute** or any other tool to validate users before performing bruteforce!! 

# Gaining initial access.

After quite a long time, I was checking the website since It was looking like the only attack vector. Especially I was really curious about the reset password button. That had to be the way, but none of the users I've previously enumerated work, besides I had no idea about the security questions and I don't really wanted to bruteforce just yet. I got a clue from a friend who solved it already and he told me "Have you really looked at the entire website?". Normally in this kind of CTFs the main vulnerability is hidden like a riddle so I kept looking for possible usernames.

It turns out I realized there was a picture of a girl with a pet on the bottom of the website and I know one of the security questions was the name of the pet so I tried to examine further that image and I notice I could save the image and when I tried to download it... bingo:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/819f89f9-172d-45a9-a6b6-b9e6ea7419a2)First of all I enumerated them using **kerbrute** and I found they were, in fact, AD users (actually I noticed I should've tried this before trying bruteforce with crackmapexec):![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/7282ab11-a0f3-489a-8241-cdf1f0ee4a10)I found out the usernames were following a different format and they had nothing to do with the previous ones. Now I know a valid username: **lilyle** and the name of her pet: **Sparky**. So I went straight forward to request a password change.![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/5d5930a0-bd39-4a64-937e-9a0b0f3800bb)
Good, now I should have access to Lily's account. I tried using **evil-winrm** but it wasn't working, it wasn't going to be that easy. I also tried RDP and nothing.![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/10a252a7-e30c-461c-8922-b50c1b241e77)

## Authenticated enumeration:

Now it was time to use crackmapexec/enum4linux with credentials to try to enumerate further. Once I ran **enum4linux** I saw there was an account called **silvergorilla409** which was one of the firsts acounts I found and he was **Backup Operator**, that could be interesting. 

In addition I got some of the shares available within the domain:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/ee0df86c-5abe-466e-9e26-783a59a4ef54)The interesting ones here are **shared** and **users** so let's have a closer look into them using **smbclient** and trying to access as **Windcorp.thm\\lilyle**.![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/9de814d5-8a81-4ea8-838a-3dec7d2a49ec)
Nice, I have got the first flag which is great and I also have found some versions of spark which I know from the previous enumeration phase is the application the enterprise is using for the technical support and I knew there was an user connected since there was a green dot on the website. So that is definitely a hint to take into consideration.
I tried some enumeration of the **users*** share which only allowed me to have more accuracy on possible users:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/f3f13290-3fee-45d0-80ab-89e4714b605d)
I was trying to find something related to **silvergorilla409** which was the backup operator but I wasn't lucky here. I also tried to get access to buse's directory since he was the only connected user in their website.

# Lateral movement:

Now it was time to download Spark and try to interact somehow with **Buse** in order to get more credentials so I could move within the domain. Just install Spark and use Lilyle credentials to authenticate. It's important to make some changes to the certificate validation of Spark in order to access:











