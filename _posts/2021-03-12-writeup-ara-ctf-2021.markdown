---
layout: post
title:  "ARA CTF 2021 [Write-up]"
date:   2021-03-12 14:44:00 +0700
categories: ctf
tags: [ctf]
---
![ara-logo](https://arek.its.ac.id/ara/assets/images/logo_black_ara.png "ARA CTF 2021")
## ARACTF 2021
HMIT ITS mengadakan event dengan berbagai cabang lomba, salah satunya Cybersecurity yang dilaksanakan pada tanggal 6-7 Maret 2021 (untuk kategori Cybersecurity).
Berikut adalah writeup dari soal web exploitation yang berhasil saya solve. 

## Web Exploitation
### Home

![ara2021-1](/assets/images/ara2021/home1.png)  

Diberikan sebuah website dan katanya didalamnya terdapat flag yang disembunyikan :

![ara2021-2](/assets/images/ara2021/home2.png) 

Sesuai dengan apa yang ditampilkan di web tersebut, karna IP kita ditolak kemungkinan hanya IP lokal yang diterima oleh server, maka tambahkan “X-Forwarded-For” pada header request untuk mem-bypass filter IP nya :

![ara2021-3](/assets/images/ara2021/home3.png) 

Dilihat dari response yang didapatkan, disitu terdapat sebuah page yang berisi gambar (images/home.jpg) dan 3 tombol yang di proses oleh select.php dan mengarah ke :


**Kitchen > room=kitchen.php**<br>
**Living Room > room=livingroom.php**<br>
**Bedroom > room=bedroom.php**<br>


Cek satu persatu isinya, dan berikut response dari kitchen.php :

![ara2021-4](/assets/images/ara2021/home4.png)  

Ternyata kita langsung diberikan flag, namun hanya potongan pertama (\$flag1), asumsi saya potongan flag sisanya ada di **livingroom.php** dan **bedroom.php**
Berikut response dari **livingroom.php** :

![ara2021-5](/assets/images/ara2021/home5.png)  

“Something hidden in this **PAGE**”, awalnya saya belum mengerti apa maksudnya jadi saya lewati ke **bedroom.php** :

![ara2021-6](/assets/images/ara2021/home6.png) 
 
Kita diberi petunjuk bahwa flag terakhir ada di /etc/flag3.txt, saya asumsikan web ini vuln terhadap LFI.
Selanjutnya kita cek dulu source php yang ada di **bedroom.php** menggunakan php://filter pada parameter room yang ada di select.php tadi :

![ara2021-7](/assets/images/ara2021/home7.png) 
![ara2021-8](/assets/images/ara2021/home8.png) 

saya berhasil mendapatkan potongan flag yang ke2, maka sejauh ini flag nya :

``ara2021{127.0.0.1_Is_ wH3re_0uR_``

Selanjutnya sesuai dengan petunjuk yang diberikan di bedroom.php tadi, bahwa flag terakhir ada di /etc/flag3.txt :
 
![ara2021-9](/assets/images/ara2021/home9.png) 

saya mendapatkan pesan error berupa :

```Warning include(php://filter/convert.base64-encode/resource=/etc/flag3.):
Failed to open stream: operation failed in <b>/var/www/html/select.php</b> on line <b>27```

setelah dianalisis ternyata “txt” dari “flag3.txt” telah difilter, jadi tujuan saya sekarang untuk melewati filter tersebut dengan payload berikut :

``php://filter/convert.base64-encode/resource=/etc/flag3.ttxtxtxtt``

Yang nantinya akan dibaca sebagai /etc/flag3.txt pada server

![ara2021-10](/assets/images/ara2021/home10.png) 
![ara2021-11](/assets/images/ara2021/home11.png) 

Dan semua potongan flag pun telah didapatkan.

**Flag : ara2021{127.0.0.1\_Is_ wH3re_0uR_ St0rY_B3Gins}**


### Oven

![ara2021-12](/assets/images/ara2021/oven1.png) 

Diberikan sebuah website dan sourcecode nya 

![ara2021-13](/assets/images/ara2021/oven2.png)  

![ara2021-14](/assets/images/ara2021/oven3.png)  

Setelah menganalisis kode tersebut, untuk mendapatkan flag kita harus melewati beberapa pengecekan.

![ara2021-15](/assets/images/ara2021/oven4.png) 

Token berada di cookie bake_here, berikut hasil decode nya :

![ara2021-16](/assets/images/ara2021/oven5.png) 

Kita harus meng-craft token dengan username = admin, password > len \$pass_verif, dan mempunyai hasil hash sha256 yang sama dengan 
\$pass_verif

Karena adanya pengecekan panjang password, maka kita tdk bisa menggunakan string yang ada pada \$pass_verif untuk melewatinya

Setelah melakukan research, saya menemukan magic hash sha256 nya dan saya craft tokennya seperti berikut :

``O:5:"Token":2:{s:8:"username";s:5:"admin";s:8:"password";s:14:"34250003024812";}``

``Tzo1OiJUb2tlbiI6Mjp7czo4OiJ1c2VybmFtZSI7czo1OiJhZG1pbiI7czo4OiJwYXNzd29yZCI7czoxNDoiMzQyNTAwMDMwMjQ4MTIiO30=``

Buat request menggunakan token tersebut dan didapatkan flagnya :

![ara2021-17](/assets/images/ara2021/oven6.png) 

**Flag : ara2021{cl4551c_typ3_ju66ling}**


### Not Secure

![ara2021-18](/assets/images/ara2021/nsecure1.png) 

Diberikan sebuah web berikut tampilannya :

![ara2021-19](/assets/images/ara2021/nsecure2.png) 

Karna tidak ada apapun disana maka tujuan saya adalah untuk scan service, direktori dan mencari informasi sebanyak2nya pada web tersebut ,berikut hasil scan menggunakan nmap :

![ara2021-20](/assets/images/ara2021/nsecure3.png) 

Terdapat ssl-cert yang muncul, lalu saya inisiatif untuk cek web yang tertera di organizationName tersebut :

![ara2021-21](/assets/images/ara2021/nsecure4.png) 

Dan ternyata flagnya ada disana.

**Flag : ara2021{p3nt1n6nya53rt1vik4sih}**


Soal web exploitation pada kompetisi ini hanya 3 soal, write-up soal yang lain oleh tim kami bisa di cek <a href="https://drive.google.com/file/d/1zhGxwLqKlvAXraRsH-lvMr66yg4uGvJu/view">disini</a>.