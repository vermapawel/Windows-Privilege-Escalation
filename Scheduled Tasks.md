**Scheduled Tasks**

There are two methods to escalate privilege with the help of Scheduled Task

**Method 1:-**

***schtasks***

<img width="828" height="398" alt="image" src="https://github.com/user-attachments/assets/1970003a-20a4-4533-b023-8b75fc88d045" />

***schtasks /fo LIST***

<img width="786" height="313" alt="image" src="https://github.com/user-attachments/assets/53627be9-fb7c-448d-96e4-00b1745c95cd" />

It will show all the scheduled task in list format.

By default windows runs multiple scheduled tasks. However we are not interested in default windows scheduled tasks.

How to find such tasks

***schtasks /fo LIST | findstr /I taskname | findstr /I /V microsoft***

<img width="563" height="322" alt="image" src="https://github.com/user-attachments/assets/758f744c-3880-4561-8292-08dd67b729c5" />

We have found one Scheduled Task which is not running by microsoft.

<img width="786" height="83" alt="image" src="https://github.com/user-attachments/assets/9ffaca2d-6561-45a0-a345-6ffcea712501" />

***schtasks /tn SaveCred /fo LIST /v***

<img width="557" height="167" alt="image" src="https://github.com/user-attachments/assets/99581dcd-b0b8-44cf-8900-82c49cd022d2" />

<img width="786" height="408" alt="image" src="https://github.com/user-attachments/assets/f5849e4a-ef0a-4b46-9756-6f47ba24faf9" />

We can see that this Scheduled Task is run by Administrator at the time of logon. It will run savecred.bat file which is in C folder.

Lets see what this file is doing

***type C:\PrivEsc\savecred.bat***

<img width="786" height="188" alt="image" src="https://github.com/user-attachments/assets/e1242195-d4bc-48dc-a65b-e745d5468d54" />

As the extension of the file is .bat, its a Batch file.

A batch file is a text file containing a series of Windows commands that are executed automatically in sequence by the command prompt (cmd.exe).

<img width="786" height="294" alt="image" src="https://github.com/user-attachments/assets/2797e386-5fb2-4a54-a04f-6fa5ab6daf23" />

Now, here inside a batch file there is a script

***runas /savecred /user:admin “cmd.exe /C exit”***

This batch file is ruing runas command to login as user:admin with the saved credentials.

Now the runas will ask for a password.

***WScript.CreateObject(“WScript.Shell”).SendKeys(“password123{ENTER}”);***

This script will automatically type password as password123 and will hit Enter key.

So we got username and admin and password as password123

Lets try to login

<img width="786" height="120" alt="image" src="https://github.com/user-attachments/assets/ded594bf-f3f0-44a0-b8c0-e0120d85bc8c" />

And a new cmd will be opened as admin user

<img width="745" height="253" alt="image" src="https://github.com/user-attachments/assets/8d5ecad6-ae39-443c-b29a-58b7861ae10e" />

**Method 2:-**

Lets go to the C:\ and see what files are there

<img width="786" height="390" alt="image" src="https://github.com/user-attachments/assets/4c7d44bb-fd48-4c7f-9a23-42f7e1ce4487" />

There is a folder called DevTools. Lets check whats inside

<img width="783" height="321" alt="image" src="https://github.com/user-attachments/assets/153afacf-6ccd-4d3c-ae20-15f41e099fe4" />

There is a PowerShell file named CleanUp.ps1

Lets see that this file does.

<img width="786" height="117" alt="image" src="https://github.com/user-attachments/assets/8b3f729f-54f5-4ac8-a152-741003ccd3dc" />

There is a script that is removing all the logs every minute

Now, we need to check if we have write permissions on this file

***icacls <filename>***

<img width="786" height="229" alt="image" src="https://github.com/user-attachments/assets/bcf7cc41-aab0-47c4-8c3f-8da46a802356" />

Every user that is a part of Users group has Modify (M) access. It means it can Read, Write and Execute access.

NT AUTHORITY has Full Access (F). It means it has Read, Write and Execute permission.

Users having Full Access can change permission of the file. But user having Modify access cannot change permission of the file. For example, NT AUTHORITY can change permission M to F for user.

<img width="786" height="246" alt="image" src="https://github.com/user-attachments/assets/639eea5a-e271-4cb9-a9c3-3de4506d640d" />

Now, the bottom three will tell what permissions we have on the folder (DevTools) where the file is located.

NT AUTHORITY has Full access on the Folder.

Now, if any file has read permission for a user and that user has Full access on the Folder, the user will also have Full access on the File as Folder access has more weightage that the Fill access.

Now, as we have Modify access to this file and this file is executing every minute, we can put a reverse shell in it.

Lets create a reverse shell

https://www.revshells.com/

<img width="706" height="647" alt="image" src="https://github.com/user-attachments/assets/967989dd-125e-46d0-8090-094f8fc98f75" />

Reverse shell have been generated. Lets copy it

On our machine lets start a listener o port 9001

<img width="786" height="164" alt="image" src="https://github.com/user-attachments/assets/09a0244b-7fbf-491f-a883-a82b66781ba2" />

Now we will put the reverse shell into the file and will for its execution.

<img width="786" height="234" alt="image" src="https://github.com/user-attachments/assets/45695d0d-9e04-4303-9ac4-c407bc49dd98" />

Lets wait for the file to get executed

<img width="786" height="205" alt="image" src="https://github.com/user-attachments/assets/56249f9c-a8fb-4b21-898d-bb10bbb6159e" />

And we are the Root user.
