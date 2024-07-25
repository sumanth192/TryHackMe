# Mr. Robot CTF

## Initial Scan
Scan the target IP with nmap to identify open ports and services:
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_30_09](https://github.com/user-attachments/assets/a85bb534-6a81-4e13-b89d-36a6a281a1e6)
nmap -sC -sV -oA scan 10.10.95.162
Results: Three ports identified two ports are open.

## Web Exploration
Visiting the target IP in a web browser redirects to a unique and stylized web page inspired by the TV show "Mr. Robot." The web page offers several commands:
prepare: Displays a video.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_30_53](https://github.com/user-attachments/assets/2b68247d-8759-41a5-b167-427367449fe3)
fsociety: Displays a video.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_31_45](https://github.com/user-attachments/assets/8747e6a4-21df-482f-b3b9-a9dc24afbec2)
inform: Shows pictures.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_32_12](https://github.com/user-attachments/assets/f5a7297f-4158-498a-a22d-9fdf75059b9e)
question: Shows pictures.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_32_59](https://github.com/user-attachments/assets/6ca56c64-0738-4211-9138-62ea1fbcae04)
wakeup: Shows a film.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_33_35](https://github.com/user-attachments/assets/36e3ecb2-f3d2-42fc-822d-fa09ab4bdd87)
join: A prompt to join.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_34_07](https://github.com/user-attachments/assets/7984608a-3b0c-42c5-8676-3692738125df)

## Source Code Analysis
View the source code of the website.
Findings: Message YOU ARE NOT ALONE.
Hint: Suggests checking 'robots'

## First Key and Wordlist
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_37_19](https://github.com/user-attachments/assets/3331d791-c00a-4565-9568-395c40d7b4d2)
Navigate to http://10.10.10.10/robots.txt.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_37_45](https://github.com/user-attachments/assets/a0ea2758-d095-4c52-809f-5938c24e04cb)
Findings: First key and fsocity.dic wordlist.

## Directory Enumeration
Use gobuster to find additional web pages:
Command: `gobuster dir -u http://10.10.10.10/ -w /usr/share/wordlists/dirbuster/directory-small.txt`
   - Found the `/wp-login` page indicating a WordPress site.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_38_21](https://github.com/user-attachments/assets/6b36f9b2-41a4-440b-a633-0b2416077cbe)

## Brute Forcing Login Credentials
Attempting to brute force the username using fsocity.dic as the wordlist:
Command: `hydra -L fsocity.dic -p test 10.10.10.10 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:invalid username" -t 30`
   - Successfully found the username: `Elliot`.
