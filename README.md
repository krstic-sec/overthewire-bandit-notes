# 🐧 OverTheWire: Bandit — Beginner Lab Notes (Levels 0–20)

**Date:** November 9, 2026  
**Author:** Luka Krstić (`krstic-sec`)  
**Environment:** Ubuntu Linux (ThinkPad T560)  

---

## 📝 About These Notes

Today was my first time sitting down and seriously using the Linux terminal. I decided to start learning Linux CLI fundamentals through the **OverTheWire: Bandit** wargame. 

While working through the levels, I kept a physical notebook beside my keyboard to write down commands, error messages, and tricks I learned along the way. I created this repository to transfer my handwritten notes into digital documentation and keep track of my journey into cybersecurity.

> ⚠️ **Note:** To respect the OverTheWire rules, **no passwords, solutions, or flags are included here**. This is purely an educational log of commands, syntax pitfalls, and basic concepts.

---

## 📓 Level-by-Level Breakdown

### Level 0 ➔ Level 1: First Steps & Reading Files
- **Goal:** Connect to the remote server and read the starting file.
- **My Note:** Connect using SSH on port 2220 (`ssh bandit0@bandit.labs.overthewire.org -p 2220`). Once inside, standard files are read using the `cat` command.
- **Command:**
  ```bash
  cat readme
  ```

---

### Level 1 ➔ Level 2: The Dash `-` File Mystery
- **Goal:** Open a file that is simply named `-`.
- **My Note:** If you type `cat -`, the terminal hangs because Linux interprets `-` as standard input (`stdin`) instead of a file name. To tell the terminal it's actually a file located in the current directory, you have to add `./` in front of it.
- **Command:**
  ```bash
  cat ./-
  ```

---

### Level 2 ➔ Level 3: Spaces and Dashes in Filenames
- **Goal:** Open a file called `--spaces in this filename--`.
- **My Note:** This file has two problems at once: spaces (which trick the shell into thinking these are separate commands) and leading dashes (which look like command options). Wrapping the name in quotation marks and using `./` (or the `--` delimiter) prevents the terminal from getting confused.
- **Command:**
  ```bash
  cat -- "--spaces in this filename--"
  # or:
  cat "./--spaces in this filename--"
  ```

---

### Level 3 ➔ Level 4: Finding Hidden Files
- **Goal:** Find the hidden file inside the `inhere` folder.
- **My Note:** Running a normal `ls` shows an empty directory. Hidden files in Linux start with a dot (`.`), so you need the `-a` or `-al` switch to display them.
- **Commands:**
  ```bash
  ls -al inhere
  cat inhere/.hidden
  ```

---

### Level 4 ➔ Level 5: Inspecting File Formats
- **Goal:** Identify the only human-readable file inside several cluttered files.
- **My Note:** Instead of guessing or opening every single file with `cat`, the `file` command inspects the actual data type of the files.
- **Command:**
  ```bash
  file inhere/*
  cat inhere/-file07
  ```

---

### Level 5 ➔ Level 6: Finding Files with Specific Properties
- **Goal:** Find a file inside `inhere` that is human-readable, exactly 1033 bytes, and not executable.
- **My Note:** Doing this manually through dozens of folders is impossible. The `find` command can filter files using exact attributes. The letter `c` tells find to count size in bytes, and `!` inverts the rule (meaning NOT executable).
- **Command:**
  ```bash
  find inhere -type f -size 1033c ! -executable
  ```

---

### Level 6 ➔ Level 7: Searching the Entire System & Handling Errors
- **Goal:** Find a 33-byte file owned by user `bandit7` and group `bandit6` anywhere on the server.
- **My Note:** Searching from the root directory (`/`) prints a huge wall of `Permission denied` errors because we don't have root access. Adding `2>/dev/null` silences all the errors so only the actual result is printed on the screen.
- **Command:**
  ```bash
  find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
  ```

---

### Level 7 ➔ Level 8: Searching Inside Large Files
- **Goal:** Find the line next to the word `millionth` in `data.txt`.
- **My Note:** The file has thousands of lines. Instead of scrolling, the `grep` command grabs only the lines containing the target keyword.
- **Command:**
  ```bash
  grep "millionth" data.txt
  ```

---

### Level 8 ➔ Level 9: Finding the Unique Line
- **Goal:** Find the only line of text that occurs exactly once.
- **My Note:** The `uniq` utility only compares adjacent lines. That means you must first sort the file so duplicate lines sit next to each other, and then use `uniq -u` to extract the single unique line.
- **Command:**
  ```bash
  sort data.txt | uniq -u
  ```

---

### Level 9 ➔ Level 10: Extracting Strings from Binaries
- **Goal:** Read readable text preceded by several `=` characters inside a binary file.
- **My Note:** Opening binary data with `cat` scrambles the terminal. The `strings` command extracts only printable human characters, and piping (`|`) into `grep` lets us filter for the `=` characters.
- **Command:**
  ```bash
  strings data.txt | grep "==="
  ```

---

### Level 10 ➔ Level 11: Base64 Decoding
- **Goal:** Decode data stored in Base64 format.
- **My Note:** Simple and straightforward. Base64 is an encoding format, and the `-d` flag decodes the data back into plain text.
- **Command:**
  ```bash
  base64 -d data.txt
  ```
  ---

### Level 11 ➔ Level 12: ROT13 Cipher & Character Translation
- **Goal:** Decode the password stored in `data.txt`, where all lowercase and uppercase letters have been rotated by 13 positions (ROT13).
- **My Note:** Simply running `cat data.txt` prints a password-like string, but it's actually encrypted and won't work for the next level. You need to pipe (`|`) the output into the `tr` (translate) command with two character sets (`A-Za-z` mapped to `N-ZA-Mn-za-m`) to shift the letters back and reveal the real password.
- **Command:**
  ```bash
  cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
  ```
---
### Level 12 ➔ Level 13: Working in `/tmp` & Multi-Layer Decompression
- **Goal:** Reverse a hexdump and decompress a file that has been repeatedly compressed multiple times.
- **My Note:** You don't have write permissions in the home directory, so you have to work inside `/tmp`. Following the hint, I used `mktemp -d` to create a temporary folder with write access, copied `data.txt` into it, and verified it with `ls`. From there, the workflow is converting the hexdump back to binary (`xxd -r`) and iteratively checking the file type (`file`) to decompress it layer by layer (`gzip`, `bzip2`, `tar`).
- **Commands:**
  ```bash
  cd $(mktemp -d)
  cp ~/data.txt .
  ls
  xxd -r data.txt fajl
  file fajl
  # Rename with proper extension (.gz, .bz2, .tar) and decompress iteratively
  ```
  ---
  ### Level 13 ➔ Level 14: SSH Private Keys & File Permissions
- **Goal:** Authenticate into `bandit14` using a private SSH key instead of a password.
- **My Note:** Localhost SSH connections from the server are blocked, so I transferred `sshkey.private` to my machine using `scp` (uppercase `-P` for port). SSH ignores private keys if permissions are too open, so I locked the file with `chmod 600` before connecting with `-i`.
- **Commands:**
  ```bash
  scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private ~/bandit14.key
  chmod 600 ~/bandit14.key
  ssh -i ~/bandit14.key bandit14@bandit.labs.overthewire.org -p 2220
  cat /etc/bandit_pass/bandit14
  ```

---

### Level 14 ➔ Level 15: Submitting Data to a Local Network Port
- **Goal:** Send the `bandit14` password to port `30000` on `localhost`.
- **My Note:** Used `nc` (netcat) and a pipe (`|`) to transmit the password string directly over a local TCP socket.
- **Command:**
  ```bash
  cat /etc/bandit_pass/bandit14 | nc localhost 30000
  ```

---

### Level 15 ➔ Level 16: SSL/TLS Encrypted Sockets
- **Goal:** Submit the `bandit15` password to `localhost` on port `30001` using SSL/TLS encryption.
- **My Note:** Standard `nc` fails because the service expects an encrypted handshake. Used `openssl s_client` with `-ign_eof` so the connection stays open long enough to return the next password.
- **Command:**
  ```bash
  cat /etc/bandit_pass/bandit15 | openssl s_client -connect localhost:30001 -ign_eof
  ```

---

### Level 16 ➔ Level 17: Port Scanning & Extracting SSH Keys
- **Goal:** Find the single SSL-enabled port between `31000` and `32000`, submit the current password, and retrieve the next private key.
- **My Note:** Used `nmap -sV` to identify which open port spoke SSL (`ssl/unknown`), piped the password via `openssl`, filtered the exact key block using `sed` (to avoid `invalid format` errors from manual copy-pasting), and transferred it via `scp`.
- **Commands:**
  ```bash
  nmap -sV -p 31000-32000 localhost
  cat /etc/bandit_pass/bandit16 | openssl s_client -connect localhost:<PORT> -ign_eof 2>/dev/null | sed -n '/-----BEGIN/,/-----END/p' > /tmp/k17
  ```

---

### Level 17 ➔ Level 18: Comparing Files with `diff`
- **Goal:** Find the single modified line between `passwords.old` and `passwords.new`.
- **My Note:** The `diff` utility compares files line by line, marking the old line with `<` and the updated password with `>`.
- **Command:**
  ```bash
  diff passwords.old passwords.new
  ```

---

### Level 18 ➔ Level 19: Bypassing `.bashrc` Auto-Logout
- **Goal:** Read `readme` when `.bashrc` immediately terminates interactive SSH logins (`Byebye !`).
- **My Note:** Passing the read command directly inside the `ssh` call executes it before the interactive shell loads `.bashrc`.
- **Command:**
  ```bash
  ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
  ```

---

### Level 19 ➔ Level 20: Exploiting SetUID Binaries
- **Goal:** Use the `bandit20-do` binary to read the password owned by `bandit20`.
- **My Note:** The binary has the SetUID bit set (`-rwsr-x---`), meaning any command passed as an argument executes with `bandit20` privileges.
- **Command:**
  ```bash
  ./bandit20-do cat /etc/bandit_pass/bandit20
  ```

---

### Level 20 ➔ Level 21: SetUID Binaries & Background Netcat Listeners
- **Goal:** Make the SetUID `suconnect` binary connect to a local port serving the `bandit20` password.
- **My Note:** Started a background `nc` listener (`&`) on a free local port that outputs the current password upon connection, then pointed `./suconnect` to that port.
- **Commands:**
  ```bash
  cat /etc/bandit_pass/bandit20 | nc -l -p 4444 &
  ./suconnect 4444
  ```

---

## 💡 What I Learned So Far

1. In Linux, special characters like `-`, spaces, and `.` always have special meanings to the shell and require escaping.
2. Piping (`|`) commands together (like `sort | uniq` or `strings | grep`) is where the real power of the CLI comes from.
3. Always redirect errors (`2>/dev/null`) when searching system-wide directories.
4. Writing down notes in a physical notebook while troubleshooting makes remembering the syntax much easier.
5. **Character Translation & ROT13 (`tr`):** Learned how classical substitution ciphers like ROT13 shift characters and how to reverse them using `tr 'A-Za-z' 'N-ZA-Mn-za-m'`.
6. **Working Safely in `/tmp`:** When you don't have write permissions in a directory, `/tmp` (combined with `mktemp -d`) is the standard isolated workspace for manipulating and unpacking files.
7. **Hex Dumps & Layered Decompression (`xxd`, `file`):** Learned how to convert raw hexadecimal dumps back into binary files using `xxd -r` and iteratively inspect file headers (`file`) to unpack multi-layered archives (`gzip`, `bzip2`, `tar`).
8. **SSH Key Hygiene & Transfers (`scp`, `chmod 600`):** Learned how to transfer private keys with `scp -P` and why SSH strictly requires `600` permissions (`rw-------`) before accepting an identity file.
9. **Network Sockets & Encryption (`nc`, `openssl`):** Practiced interacting with local TCP ports via `nc` and handling SSL/TLS-encrypted services using `openssl s_client -ign_eof`.
10. **Service Reconnaissance & Clean Extraction (`nmap`, `sed`):** Used `nmap -sV` to pinpoint SSL services across port ranges and `sed` to cleanly extract PEM/OpenSSH key blocks without copy-paste formatting errors.
11. **File Diffing & Non-Interactive SSH (`diff`, `ssh "..."`):** Used `diff` to isolate single-line file changes and passed remote commands directly through `ssh` to bypass `.bashrc` auto-logout traps.
12. **Privilege Execution & Background Jobs (`SetUID`, `&`):** Understood how SetUID binaries (`-rwsr-x---`) execute with owner privileges and how to pair background `nc -l` listeners (`&`) with local client binaries.
