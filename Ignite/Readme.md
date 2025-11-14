TITLE:Ignite(Easy)

Open ports:80(Apache httpd 2.4.18)

Tools used:nmap searchsploit nc 

Task 1:user.txt
I visited the page and tried to go /robots.txt directory and i saw that /fuel directory was in disallow section and when i visited it,it lead me to login page /fuel/login.
And also in main page when i scrolled down a bit i saw username and password admin:admin and i logged in as admin.But as i checked everything there,i could not find anything and i checked fuel cms 1.4 exploits and i found an exploit for it and i executed it and got command line then i write an reverse shell and got shell on my end.Then i get the flag for user

2.Find the root.txt

