# Lab 02: Hashcat Password Cracking

**Date:** 2026-10-20
**Platform:** Windows (PowerShell) & Kali Linux
**Difficulty:** Medium
**Target:** Local Lab Environment
**Skill Focus:** Password Cracking · Hash Analysis · Rule-Based Attacks

---

## Objective

Demonstrate the process of cracking MD5 password hashes using Hashcat. This lab explored Dictionary Attacks, Hybrid Attacks (using rules), and Brute Force (Mask) attacks, highlighting the differences in efficiency and success rates between them.

## Setup & Troubleshooting

This lab was initially started in Kali Linux but transitioned to Windows PowerShell to match the instructor's environment.

**Environment Issues Resolved:**
* **Line Endings:** The provided `passwords-archive.lst` file had Windows line endings (`\r\n`). When running on Linux, hashcat threw a `Token length exception`. Resolved by converting the file to Linux line endings using `sed -i 's/\r$//'` (or by migrating to Windows).
* **Missing Rulesets:** The instructor's environment used `best64.rule`, which was not present in the newer Hashcat version installed. We successfully located and used alternative rulesets like `d3ad0ne.rule` and `dive.rule` for the hybrid attacks.
* **File Transfers:** Transferred the 51MB lab archive from Windows to Kali using `scp` over SSH.

## Execution & Results

### 1. Dictionary Attack
**Command:**
`.\hashcat -a 0 -m 0 .\passwords-archive.lst .\rockyou.txt`

**Result:** Recovered 29 out of 172 hashes. The `rockyou.txt` wordlist successfully identified common passwords like `1234567890`, `password123`, and `abc123`.

### 2. Hybrid Attacks (Rules)
**Command:**
`.\hashcat -a 0 -m 0 .\passwords-archive.lst .\rockyou.txt -r .\rules\d3ad0ne.rule`

**Result:** Recovered 25 new hashes. By applying mutation rules (e.g., appending numbers, capitalizing), we cracked more complex variations like `Pa$$w0rd01!` and `June21st1@`.

### 3. Brute Force (Mask Attack)
**Command:**
`.\hashcat -a 3 -m 0 .\passwords-archive.lst ?d?d?d?d?d?d?d?d`
*(Attempted `?a?a?a?a?a?a` but stopped due to CPU time constraints).*

**Result:** Exhausted 100 million numeric combinations in 5 seconds, but yielded 0 new hashes, as those had already been found by the dictionary attacks.

**Total Recovered: 56/172 (32.56%)**

## Lessons Learned

* **Hardware limits matter:** Testing the `?a?a?a?a?a?a` mask attack revealed a massive keyspace. On a CPU, this would take days. In a real engagement, GPU offloading or cloud cracking services are essential for brute-forcing full keyspaces.
* **Rules are powerful:** The Hybrid Attack was the most effective, finding over 25 passwords that the raw dictionary missed by simply guessing the way humans modify words.
* **Troubleshooting environment issues:** Encountering file formatting and line-ending errors is common in real-world engagements. Learning to use `sed` and navigate Windows/Linux discrepancies is a crucial skill.

## References

- Hashcat Documentation: https://hashcat.net/wiki/
- Rockyou Wordlist: https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
