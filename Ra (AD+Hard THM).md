
---
Skills: Web enumeration, SMB, Phishing, Active-Directory, Data-exfiltration.
---
Tools: Enum4linux, smbclient, smbmap, crackmapexec, impacket-suite, mimikatz, responder, burpsuite, gobuster, whatweb, evil-winrm, powershell.
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

I've decided to have a closer look at how the requests were being sent to the reset password file:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/4de50a2b-b916-45a2-ab84-88e92d7f720c).

That form could potentially be vulnerable to pitchfork attacks using Burpsuite or maybe other kind of bruteforce attacks to reset the password assuming the usernames we listed before were valid.
## Port 7070, technical support IM technologies.
On the other hand, **port 7070** was only a website with a link pointing to what **HTTP Binding and BOSH technology were and how do they use XMPP protocol**:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/435f69b9-fd09-4a87-a239-2ecc4a9b9400)
I was curious about this and I thought it could be a possible lead to find the vulnerability in this machine. However I wanted to solve this as a real scenario, I focused on further enumeration instead of tunnel-visioning the finding which, anyways, could be baiting me into a rabbit-hole.
## Port 5985: WinRM

WinRM allows the user to execute commands on the Windows server and it does require authentication, some tools like **evil-winrm** allow to perform attacks like pash the hash to gain remote control with a shell. The other way to perform an attack against this service is to bruteforce using **crackmapexec** with the users I have previously found and also adding some built-in accounts like Administrator just in case. So I created a list of users with the e-mails gathered before and started the brute-force attack:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/607033fb-5dd8-476b-95bd-a50db768ff6f)
**Note**: After reviewing this explanation I have noticed this is not the best way to proceed but I was doing this writeup as I was solving the machine. You should always use **kerbrute** or any other tool to validate users before performing bruteforce!! 

# Gaining initial access.

After quite a long time, I was checking the website since It was looking like the only attack vector. Especially I was really curious about the reset password button. That had to be the way, but none of the users I've previously enumerated work, besides I had no idea about the security questions and I don't really wanted to bruteforce just yet. I got a clue from a friend who solved it already and he told me "Have you really looked at the entire website?". Normally in this kind of CTFs the main vulnerability is hidden like a riddle so I kept looking for possible usernames.

It turns out I realized there was a picture of a girl with a pet on the bottom of the website and I know one of the security questions was the name of the pet so I tried to examine further that image and I notice I could save the image and when I tried to download it... bingo:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/819f89f9-172d-45a9-a6b6-b9e6ea7419a2)First of all I enumerated them using **kerbrute** and I found they were, in fact, AD users (actually I noticed I should've tried this before trying bruteforce with crackmapexec):![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/7282ab11-a0f3-489a-8241-cdf1f0ee4a10)I found out the usernames were following a different format and they had nothing to do with the previous ones. Now I know a valid username: **lilyle** and the name of her pet: **Sparky**. So I went straight forward to request a password change.![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/5d5930a0-bd39-4a64-937e-9a0b0f3800bb).

Good, now I should have access to Lily's account. I tried using **evil-winrm** but it wasn't working, it wasn't going to be that easy.![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/10a252a7-e30c-461c-8922-b50c1b241e77).

I also tried RDP and nothing

## Authenticated enumeration:

Now it was time to use crackmapexec/enum4linux with credentials to try to enumerate further. Once I ran **enum4linux** I saw there was an account called **silvergorilla409** which was one of the firsts acounts I found and he was **Backup Operator**, that could be interesting. 

In addition I got some of the shares available within the domain:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/ee0df86c-5abe-466e-9e26-783a59a4ef54)The interesting ones here are **shared** and **users** so let's have a closer look into them using **smbclient** and trying to access as **Windcorp.thm\\lilyle**.![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/9de814d5-8a81-4ea8-838a-3dec7d2a49ec).

Nice, I have got the first flag which is great and I also have found some versions of spark which I know from the previous enumeration phase is the application the enterprise is using for the technical support and I knew there was an user connected since there was a green dot on the website. So that is definitely a hint to take into consideration.
I tried some enumeration of the **users*** share which only allowed me to have more accuracy on possible users:![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/f3f13290-3fee-45d0-80ab-89e4714b605d)
I was trying to find something related to **silvergorilla409** which was the backup operator but I wasn't lucky here. I also tried to get access to buse's directory since he was the only connected user in their website.

# Lateral movement:

Now it was time to download Spark and try to interact somehow with **Buse** in order to get more credentials so I could move within the domain. Just install Spark and use Lilyle credentials to authenticate. It's important to make some changes to the certificate validation of Spark in order to access:

![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/f5528af8-f2fa-45b4-aad3-6a29dbbb6b5b)
![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/1eb73de7-2590-465b-b31d-42e32e2befeb)

Once you have the user selected now it's time to find out a way to get his credentials. I knew the way here, was get the user to click a malicious link so I could get his credentials in some way. So I did some research and I found a tool called **responder** which is used for nbts-poisoning abusing LLMNR protocol which is still used for backward compatibility. If we execute Responder and we get a domain workstation to try to connect it will show the IP, the username and credential details **including NTLM hashes**.![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/55d6a659-590d-4b33-b824-a02209f63fec).

Remember to set the interface as **tun0** since you're connected through the vpn and send a link for him to click. So now we have the NTLMv2 hash from this user, we could try to perform pash the hash through evil-winrm but we can also try to crack the hash while we perform other tasks. The way to do it with hashcat is as follows:
**hashcat -m 5600 Buse.txt rockyou.txt --force**
![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/783c084f-d11b-48ef-88ac-95297a2cd2e3)
So we have the password: **uzunLM+3131**, so far so good. This user has WinRM access: ![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/6e048bd1-1068-451a-9d2d-71388eba48c3)
Now I've got the second flag:
![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/2e1ad9e0-7873-48b9-8383-cb95e83dffbc)

# Getting Domain admin:

To enumerate AD is very ussefull to upload the module PowerUp.ps1 so you'll get many functions to help you enumerating the AD. Some ussefull commands are the following:
```powershell
(Get-ADUser $env:USERNAME -Properties *).MemberOf  # This will give you some important information about your user.
  
net user; net group # etc to perform further enumeration there's a huge list of cmdlets and commands.
```

This user does not seem very interesting at all but he has WinRM access to the DC so he could have access to the domain accounts registered within the domain that we enumerated before using smbclient. I found the following explanation after reviewing some writeups once I finished the machine and I thought it was really well explained: 
>"This is because they’re nested in the Account Operators AD group. This builtin group by default has privileges to login to DCs and manage all non-protected users & groups. By protected we mean those whose Attribute AdminCount = 1. These users and groups get their DACL from the AdminSDHolder and do not inherit their DACL from any OUs that they are placed in by a careless administrator. This is to stop a system administrator from shooting themselves in the foot by accident, much like the PowerShell execution policy. It will not stop an attacker from shooting you in the foot on purpose.
The VM’s author meant for us to poke around and notice a folder C:\scripts with a checkservers.ps1 file inside. This PS1 pulls values from a text file stored in a user’s folder, does some stuff, and passes the result to Invoke-Expression.
I have said before that I am not sure that anyone other than attackers and malware writers use Invoke-Expression. More accurately they tend to use an obscured version of its alias iex. In this case we are the attacker and we were meant to find this. I am probably preaching to the choir, but Invoke-Expression takes a string as input and runs it as a command."

This bassically means that the user has acccess to WinRM in the DC so he should be a member of that Account Operators group, we can manage most of the users within our domain registered in this DC. So in order to abuse this we can perform a Command injection in the scheduled task we found after performing enumeration looking at **.ps1** files and services executing this kind of files. 

**Note**: Performing enumeration in Windows looking for services or scripts being executed is similar to cronjobs in Linux, but in this case is **.ps1** instead of **.sh**
```powershell
Set-ADAccountPassword -Identity brittanycr -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "EasyP4Sword!!" -Force)
```

Since brittany does not have WinRM permissons, we'll have to create a new file with the same name of the file that's being executed in the script. We can find that out after changing brittany's password and login in via **smbclient** and having a look the file is called **hosts.txt** so we just need to create a new one in our machine and upload it using the same tool.
**Note**: It is important to use put with the exact same name to avoid any kind of conflicts, deleting the file and uploading a new one may cause some issues.
```powershell
;Add-ADGroupMember -Identity "Domain Admins" -Members "buse" ; Add-ADGroupMember -Identity "Administrators" -Members "buse"
```

Then upload it to brittanycr’s user folder on the DC.
```bash
cd /home/kaysuh/machines/thm/Active-Directory/Windcorp/Ra/exploits  
smbclient //10.10.46.34/users -U Windcorp.thm\\brittanycr  
cd brittanycr  
put hosts.txt
```

You can have a look at log.txt to have a guess of how long are you going to wait (won't be much).
![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/b4dba456-21ac-4d27-b5e7-dd92a4de4f5e)
Once we get the Domain Admin we can try to retrieve the last flag:
![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/b893dffc-c3da-490f-89d4-9566051dd12e)

Now it is time to upload Invoke-Mimikatz.ps1 to our evil-winrm's powershell.
```powershell
# From evil-winrm:
upload <FullPathToResource>/Invoke-Mimikatz.ps1
. .\Invoke-Mimikatz.ps1
# Now the module is loaded to your powershell.
```
I was trying to dump the SAM to get Administrator access to the local machine but unfortunately for me there was something interfeering (I also tried elevating token, ) and I researched a little bit and having Domain Admin permissons does not guarantee that you'll be able to extract all credentials locally with mimikatz not even using `lsadump::dcsync \user:Administrator \domain:Windcorp.thm`.

![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/281e5062-cfdc-4489-9a34-ce3a8f86f199)

So I went to impacket and I tried using **impacket-secretdumps** and wait for it to retrieve the hashes.
![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/c9499c0e-e978-4964-9d6a-3a74a8cc977b)

Sadly for me I think I broke WinRM at the end of the machine since I couldn't log in anymore using **evil-winrm** but anyway I got what I was looking for and I simply used **smbclient** to perform pash the hash and download the 3º flag from the Admin Desktop:
![image](https://github.com/K4ySuh/PublicWriteups/assets/147923141/e07a9e67-d3b8-4dac-a56d-b7c8309be0c7)











