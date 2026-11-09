# 🐧 OverTheWire: Bandit — Beginner Lab Notes (Levels 0–11)

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

## 💡 What I Learned Today

1. In Linux, special characters like `-`, spaces, and `.` always have special meanings to the shell and require escaping.
2. Piping (`|`) commands together (like `sort | uniq` or `strings | grep`) is where the real power of the CLI comes from.
3. Always redirect errors (`2>/dev/null`) when searching system-wide directories.
4. Writing down notes in a physical notebook while troubleshooting makes remembering the syntax much easier.
