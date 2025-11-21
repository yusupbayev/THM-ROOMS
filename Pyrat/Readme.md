TITLE:Pyrat

Open Ports:22(OpenSSH 8.2p1) 8000(http-alt SimpleHTTP/0.6 Python/3.11.2)

Tools used:nmap nc

Task 1:user flag.I first tried to visit the page ip:8000 and it gave me "try more basic connection" so i used curl and it gave me the same response then i tried ncat for connection "nc ip 8000" and i wrote the python codes and it actually worked so i found the reverse shell code for python and used that to get a shell.I logged in as www-data user which was not enough for me to get the flag so i have to find a way to escalate my priv.I send the linpeas.sh via creating a server on my machine using "python3 -m http.server 8000" and on the target machine i connected to it using wget and got the linpeas.sh and run it on on target machine.Btw before running it i did "chmod +x" to make sure it is executable.And i found out the directory /opt/dev/.git and there was a file named config which included name email username and password for 'think'.I entered as think via ssh and got the user flag.

Task 2.root flag.
