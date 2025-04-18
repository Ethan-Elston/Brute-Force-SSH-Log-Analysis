# Useful Linux commands for analysis

## Steps

	- Now that I have WSL installed, I'll go ahead and navigate to HacktheBox and start a Sherlock lab called "Brutus"
		○ The scenario is below

![image](https://github.com/user-attachments/assets/aaf83d8c-accc-4d51-86a2-01dfa4385bba)


Understanding the scenario:
							
	- "auth.log" (Authentication Log)
		○ Locations: /var/log/auth.log (Debian/Ubuntu)
		○ What it logs:
			§ SSH logins
			§ 'sudo' usage
			§ Failed login attempts
			§ PAM (pluggable Authentication Module) events
		○ Why it matters (Blue Team)
			§ Detect brute force attacks
			§ Spot privilege escalation
			§ Monitor SSH usage
			§ Investigate unauthorized access

	- "wtmp" (Login Record Log)
		○ Location: /var/log/wtmp
		○ What it logs:
			§ All login/logout events
			§ System boots and shutdowns
			§ Records user sessions
		○ How to read:
			§ Use 'last', 'who', or 'utmpdump'
		○ Why it matters (Blue Team)
			§ Track when users accessed the system
			§ See who was logged in during a security event
			§ Detect unusual session patterns or reboots

	- Confluence server:
		○ A proprietary product made by Atlassian
		○ Used by companies as a self-hosted collaboration and documentation platform
		○ Also used for internal wikis, knowledges bases, and team documentation
		○ "Server" version means that you host it yourself (rather than Atlassian hosting it via the cloud)
		○ These usually hold very sensitive information 

	- So I know that the attacker successfully authenticated to this server.

	- SSH (Secure Shell) is used to remotely access and control a server via the command line
		○ Attacker gets SSH access = bad things. They usually have full control if they gained access via SSH.


Understanding Linux commands commonly used in analysis:

	- After I downloaded the Brutus.zip file, I moved it to the desktop and extracted the contents using 7-zip to a directory called "Brutus\"

![image](https://github.com/user-attachments/assets/ffcbde91-880a-4f78-b31e-e744b54ae956)

	- I now have a new directory called "Brutus" in file explorer
		○ In it is 'auth.log' and 'wtmp'

![image](https://github.com/user-attachments/assets/471d79cf-c65a-437e-bb2d-67fbf21f6ae3)

	- Now since I have WSL installed, I can hold shift and right click anywhere in the folder and select the "Open Linux shell here" option

![image](https://github.com/user-attachments/assets/98f4e7c9-4ddf-4870-be20-4ff2ac6611e6)

	- This will open up a Linux terminal

![image](https://github.com/user-attachments/assets/fd4fad1d-c36d-48cc-b372-eff54aecc92c)

	- So I'm in the correct directory, to see the files, I'll need to use the 'ls' command
		○ Remember the 'auth.log' file will contain logs related to authentication
		○  The 'wtmp' log file will track successful login activities (which I know the attacker did successfully gain access). It will also provide the timestamp and the IP of the successful logon

	- To read the file, I used the 'cat' command

![image](https://github.com/user-attachments/assets/995940b8-bfe8-4d4b-8c10-f341397b5c35)

	- So there is a lot of data to sort through, to make things easier I can use the 'cut' command

	- The 'cut' command essentially slices columns that are delimited with a character or special character that I provide
		○ If I'm interested in the IP field of this log file, I can use the cut command to extract this field and display it
		○ Every time you use the 'cut' command, you must first read the log file using the 'cat' command and then append the command with a pipe '|'
		○ For example:

	- "cat auth.log | cut -d ' ' -f 1"

	- What does this do?
		○ The '-d' flag specifies the delimiter we'll be using; which is a space
		○ The '-f' flag is the field/column that I want to use
		○ 1 is the field/column I've selected

![image](https://github.com/user-attachments/assets/67916e1a-2bad-4edf-953c-aa774a012dab)

	- As you can see, it now only displays the first column, which was the month of the timestamp (March)

	- Why did the command do that? 

	- Look above; after I used the 'cat' command, notice that there is space between "Mar" and "6" in the log file, and because I selected 1, I'm seeing the first field
		○ What happens if I select field 2?

	- "cat auth.log | cut -d ' ' -f 2"

![image](https://github.com/user-attachments/assets/b186ac3c-3d83-4426-a02e-eefd5cc10321)

	- I get nothing…

	- Well the output is terchnically showing something, really it's showing field 2 of the log file. The only column after "Mar" (field 1), is a space. So it displays the space that's in field 2
		○ So it's output the space that's in column 2

	- If I wanted to output the day of the month (6), I would have to select field 3

	- "cat auth.log | cut -d ' ' -f 3"

![image](https://github.com/user-attachments/assets/ae0646c0-4979-4d0a-9aba-01e764a19626)

	- With the cut command I can actually select more than one field, I can exclude the column 2 (which is just a space), and display only the month and day of the month

	- I can do this by using a comma in the command

	- "cat auth.log | cut -d ' ' -f 1,3"

![image](https://github.com/user-attachments/assets/6062bf91-916b-435d-bc1d-187b2bf4287f)

	- With the cut command, I can also provide a range of fields by using a dash
		○ So using a comma means "select the fields 1 AND 3"
		○ Using a dash means "select fields 1 THROUGH 3"
		○ So…

	- "cat auth.log | cut -d ' ' -f 1-3"

![image](https://github.com/user-attachments/assets/bba3eeb9-5a1e-4862-8cb7-26a5209e6cfc)

	- So now it includes the two spaces that are in column 2

	- Another very useful command to help find information is 'grep'
		○ I'll cat out the auth.log file again to view the contents and find a situation where I can use grep

	- So below I found a very interesting log

![image](https://github.com/user-attachments/assets/83da3c17-ba26-4741-b365-9158664038a3)

	- The account 'cyberjunkie' successfully logged in
		○ I wonder what other accounts have been accepted?

	- I can use the 'grep' command to search for a specific key word; such as "accepted"
		○ I can use 'grep accepted auth.log'
		○ This is using grep to look for the keyword of accepted under the target file 'auth.log'

![image](https://github.com/user-attachments/assets/db18081b-ba1b-4f93-9cbd-dbb2a14f3129)

	- I don't get any hits…
		○ I know I saw the word accepted, so what's the deal?

	- The 'grep' command is case sensitive!
		○ 'grep Accepted auth.log'

![image](https://github.com/user-attachments/assets/9bc61c3f-9577-43b7-ac2b-c6768007cf8c)

	- I now have 4 entries

	- You can use the '-i' flag to tell grep to ignore case sensitivity
		○ 'grep -i accepted auth.log'

![image](https://github.com/user-attachments/assets/4d89b08d-1f08-4498-a4ab-9c269ad0458e)

	- Now I get 4 entries again

	- Another important flag for grep is '-v'
		○ This flag excludes key words

	- Let's say I want to exclude all events coming from the keyword 'root' in the output above

	- 'grep -i accepted auth.log | grep -iv root'

	- So in the command above I added the pipe '|' to add on to the command, then to exclude root I used '-iv'.
		○ Because I already used a dash to ignore case sensitivity with 'i' I can just add on 'v' behind it to exclude the keyword

![image](https://github.com/user-attachments/assets/d1262fc1-de10-414f-a119-da663e3a1c92)

	- Now I only have 1 entry



	- These are essentially all the commands I need to perform log analysis in Linux, let's start actually analyzing the file.


