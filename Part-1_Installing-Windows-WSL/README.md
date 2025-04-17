# Installing Windows Subsystem for Linux (WSL)

## Steps

	- Windows Subsystem for Linux (WSL)
		○ Windows compatibility layer for Linux
		○ Run Linux alongside Windows
		○ Linux command-line tools and applications on Windows
		○ Bridges gap between operating systems
		○ Ideal for developers without dual-boot or virtual machines

	- To start I'll head to "Windows Features" and turn on the "Virtual Machine Platform" checkbox and the "Windows Subsystem for Linux" checkbox, as well as the "Windows Hypervisor Platform" checkbox
		○ I'll then need to restart the computer

![image](https://github.com/user-attachments/assets/1e7952af-a93a-4c04-8cbb-b3900350368c)

	- This prepares the VM for WSL, and essentially does the same thing as the 'wsl --install' command if you did it from the terminal

	- If I open file explorer now, there is a directory called Linux now
		○ It's empty because I still need to actually install WSL

![image](https://github.com/user-attachments/assets/d687c004-8071-405c-8c43-c37740a44683)

	- I need to install WSL
		○ To do that I need to open the command prompt in administrator mode

	- I'll used 'wsl --install' command again to confirm, it was installed. It was.
									
	- So I currently I don't have any Linux distributions installed
		○ But I can check which distributions are availabel online
		○ Using 'wsl --list --online'

![image](https://github.com/user-attachments/assets/2233b152-7c5a-4123-a490-593849f52e1a)

	- I would like to install Ubuntu
		○ I'll head to a browser and search for "Ubuntu releases"
		○ This is so I can confirm the latest version of Ubuntu

![image](https://github.com/user-attachments/assets/3c5e0a2b-b762-49dd-8394-f0bdccd11da7)

	- The latest version of Ubuntu is Ubuntu 24.04.2 LTS 
		○ So this is the version I'll install

	- To install it:
		○ 'wsl --install -d Ubuntu-24.04'

![image](https://github.com/user-attachments/assets/945c733c-21f3-4292-aaf8-31c29cdc8f84)

	- This brings up a new window called "Ubuntu 24.04.LTS"

![image](https://github.com/user-attachments/assets/dc95387a-48d2-4514-9b0a-8456906dfa59)

	- Now I need to create a username and password for Ubuntu.

![image](https://github.com/user-attachments/assets/d76cad63-2fc9-4a93-aad2-e6fe25f23da2)

	- So now I am in the Linux distro

	- There are multiple ways to start the Ubuntu terminal. One way is to search 'WSL' in the search bar. 

	- The way I want to do it, is through the "Windows Terminal" application
		○ I'll need to download this app through the Microsoft store
		○ It will allow me to have multiple Windows terminals in the same Window

![image](https://github.com/user-attachments/assets/9431bb63-4ba2-45c7-94b6-39120323d345)

	- Once it downloads, I can open the application.
		○ I went ahead and pinned it to start and the taskbar
		○ By default it starts with Windows PowerShell

![image](https://github.com/user-attachments/assets/01cba0c8-ce57-4719-8661-a24dfb6ffeec)

	- But if I click the drop down arrow next to the 'new tab' plus sign, I actually have the option to the launch the Ubuntu terminal

![image](https://github.com/user-attachments/assets/f3b808c6-a0a6-431d-a871-e56acd5096b4)

	- If I click the Ubuntu option, it should bring up the Ubuntu terminal

![image](https://github.com/user-attachments/assets/75115dfe-bb71-4b4a-b2d0-1c0e04f41993)
	
	- Now I don't want to manually open this terminal everytime

	- To fix it I can go to settings in Terminal
		○ I need to click the arrow again and click the "Settings" option

	- Then underneath the "Default profile" option, I'll need select 'Ubuntu 24.04.1 LTS'

![image](https://github.com/user-attachments/assets/70d0f9db-f1c8-4af0-b6d3-0575cf977e89)

	- Then I'll need to save it, and close the terminal

	- Now when I run terminal, the Ubuntu terminal automatically boots up

	- To confirm the Ubuntu version I'll run the following command
		○ 'lsb_release -a'

	- I would like to update Ubuntu

	- 'sudo apt update'

![image](https://github.com/user-attachments/assets/78137cb6-fbe5-4e73-8d54-b42611429fc3)

	- After I provide the password I configured, it will start to list a bunch of packages that can be updated

![image](https://github.com/user-attachments/assets/8f18d30f-1519-478e-b7d6-a409c996b886)

	- As you can see 148 packages can be upgraded 
		○ 'sudo apt upgrade'

	- After entering this command, all 148 packages should be upgraded.  

![image](https://github.com/user-attachments/assets/cc940e27-86cd-4ea3-b4dc-9451d4fc4c67)

	- Then I need to type y for yes

	- It should then begin updating 

	- I can also access WSL through PowerShell or command prompt

	- In command prompt or PowerShell, I just need to type 'wsl' to open it

![image](https://github.com/user-attachments/assets/0d901b3e-9e04-4bea-8fe3-7c6d345d7ccf)

	- To make sure I'm running WSL 2 and not WSL 1, I can go to the command prompt or powershell and type the following command.
		○ 'wsl --status'

![image](https://github.com/user-attachments/assets/1bd15f81-4cac-47e4-89a1-ac3b1534d813)

	- So I confirmed from the "Default Version" that I'm running WSL 2 and not 1

	- Why does this matter? Well, let's look at what both do.

![image](https://github.com/user-attachments/assets/acd9cabe-4fbe-43b5-8aac-da74c4d0bfe5)
