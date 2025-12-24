TITLE:Rabbit Store

Open Ports:22,80

Used tools:ffuf,nmap,gobuster,burp,

Task1:User.txt
So from my vhost and directory bruteforcing i found login page with storage/cloudsite.thm and i could register new account and i captured that request with burp and there was jwt cookie and i tried to decode it and i saw that there was "subscription":"inactive" so i send that register request to repeater and at the end i added "subscription":"active" with my email and password above it,when i sent that request,it came back with "User created successfully" so then i logged in and i saw that there were options to upload files.
