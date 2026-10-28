# Host & Network Penetration Testing: The Metasploit Framework CTF 2

## Overview

This repository documents my hands-on experience completing the INE **Host & Network Penetration Testing: The Metasploit Framework CTF 2** lab using Kali Linux.

The lab involved enumerating exposed services, investigating an RSYNC server, exploiting a vulnerable Roxy-WI web application with Metasploit, and examining scheduled tasks to locate four flags.

## Lab Environment

- **Platform:** INE
- **Operating system:** Kali Linux
- **Target 1:** `target1.ine.local`
- **Target 2:** `target2.ine.local`
- **Tools:** Nmap, RSYNC client, Metasploit Framework, Meterpreter

## Objectives

- Enumerate the open RSYNC port and inspect its response.
- Explore the RSYNC module and retrieve exposed files.
- Identify and exploit the Roxy-WI web application on the second target.
- Investigate scheduled-task configuration files for the final flag.

## Techniques Practiced

- Service and version detection with Nmap
- RSYNC module enumeration
- Remote file listing and retrieval
- Metasploit module search and configuration
- Exploitation of a vulnerable web application in a lab
- Meterpreter filesystem navigation
- Scheduled-task inspection

## Repository Contents

- [`lab-notes.md`](lab-notes.md) — detailed, step-by-step walkthrough of all four objectives, commands, explanations, and captured flags.

## Learning Outcomes

This lab helped me practise identifying exposed services, following enumeration results, configuring an exploit module, verifying a successful Meterpreter session, and inspecting system configuration files.

## Disclaimer

This repository is intended for educational purposes and authorized cybersecurity training. Perform security testing only on systems for which you have explicit permission. Check INE's current rules before publishing lab solutions or flags publicly.
