---
layout: post
title:  "GEMASTIK 13 CTF [Write-up]"
date:   2020-09-24 20:41:25 +0700
categories: ctf
tags: [ctf]
---
![gemastik-logo](https://gemastik13.telkomuniversity.ac.id/dashboard/assets/images/Gemastik12-Logo-default.png "GEMASTIK 13")
## GEMASTIK
GEMASTIK atau Pagelaran Mahasiswa Nasional Bidang Teknologi Informasi dan Komunikasi, merupakan program Pusat Prestasi Nasional, Kementerian Pendidikan dan Kebudayaan.  

GEMASTIK 13 cabang Cyber Security / Keamanan Jaringan ini bertujuan untuk menguji kemampuan peserta dalam menghadapi kasus keamanan sistem komputer dan jaringan yang telah disiapkan, termasuk di dalamnya keamanan data. Daya analisis dan kreativitas peserta ditantang untuk mencari kelemahan dalam suatu sistem yang telah dirancang untuk memiliki celah atau informasi tertentu yang memungkinan terjadinya peretasan pada sistem tersebut.
Berikut adalah writeup dari beberapa soal yang berhasil saya dan tim solve.  

## Web Exploitation
### Please Hack Me
Diberikan sebuah page seperti berikut :

![gemastik13-1](/assets/images/gemastik13/1.png)  

Setelah dianalisis, intinya untuk mendapatkan flag kita harus memberikan token base64 yang berisi array yang sudah di serialize dan berisi username & password,   
lalu server akan menerima token tersebut dan memasukkannya pada variable **<span>$data.</span>** **<span>$data</span>** tadi akan di hash lalu ada pengecekan dimana nilai dari **<span>$data['hmac']</span>** harus sama dengan **<span>\$data['admin_hash']</span>**  
berikut solver yang saya gunakan untuk bypass **HMAC** nya :

```exploit.php
<?php $data = array(); 
$data["user"] = "admin"; 
$data["password"] = "admin"; 
$data["hmac"] = "b"; 
$data["hmac"] = &$data["admin_hash"]; 
echo base64_encode(serialize($data)); ?>
```

**<span>$data[“hmac”]</span>** akan selalu sama nilainya dengan **<span>$data[“admin_hash”]</span>**, dan ketika dibandingkan memakai strict comparison (===) hasilnya akan sama. Jadi pemeriksaan **hmac** akan lolos.

![gemastik13-2](/assets/images/gemastik13/2.png)  

Lalu tinggal akses webnya dengan token yang sudah dibuat dan didapatkan flagnya.

![gemastik13-3](/assets/images/gemastik13/3.png)  


**Flag : gemastik13{B4ckr3ferenc3}**

### My Information is Leaking
Diberikan sebuah website yang hanya berisi text "information leakage on this website"  
setelah dianalisis ternyata terdapat self signed cert pada web tersebut, ketika kita lihat detailnya

![gemastik13-4](/assets/images/gemastik13/4.png)  

Terdapat banyak informasi dari ssl certificate tersebut dan kami menemukan internal hostname di situ.  
Langsung saja eksekusi dengan curl menggunakan internal hostname tersebut dan didapatkan flagnya.

```CURL
curl -k -H "host: break.hackmehackme.id" https://180.250.135.70/
```

![gemastik13-5](/assets/images/gemastik13/5.png)  

**Flag : gemastik13{s4mpaikan_saja_salamku_tuk_kekasihmu_y4ng_b4ru}**


## Steganography

### Missing Something
Diberikan file image seperti berikut,  

![gemastik13-6](/assets/images/gemastik13/6.png)  

kemudian lakukan analisis dengan tools online [https://magiceye.ecksdee.co.uk/](https://magiceye.ecksdee.co.uk/)

![gemastik13-7](/assets/images/gemastik13/7.png)  

dan didapatkan flag yang sedikit kurang jelas, lalu kami coba perjelas dengan stegsolve.

![gemastik13-8](/assets/images/gemastik13/8.png)  

Flag pun terlihat lumayan jelas:  

![gemastik13-9](/assets/images/gemastik13/9.png)  

**Flag : gemastik13{P4nd3m1C_m4k3s_m3\_stay_@\_H0m4}**

### AH↓HA↑HA↑HA↑HA↑
Diberikan file wav, lalu kami analisis dengan audacity :

![gemastik13-10](/assets/images/gemastik13/10.png)  

setelah itu kami coba split stereo track :

![gemastik13-11](/assets/images/gemastik13/11.png)  

kami berasumsi ini adalah SSTV, klik tanda silang yang atas dan play yang bawah dan record output menggunakan robot36 :

![gemastik13-12](/assets/images/gemastik13/12.png)  

Benar saja, flag pun berhasil didapatkan.

**Flag : gemastik13{yougotme_peko}**


## Digital Forensic
### Congratulations
Diberikan sebuah file png yang corrupt, kami coba analisis dengan pngcheck dan terdapat error :

![gemastik13-13](/assets/images/gemastik13/13.png)  

kemudian kami asumsikan pngchunk, dan untuk recover png ini kita harus melakukan correction pada IHDR, sRGB, pHYs, dan IDAT. 
Solve dengan script berikut :

![gemastik13-14](/assets/images/gemastik13/14.png)  
![gemastik13-15](/assets/images/gemastik13/15.png)  

Flag pun berhasil didapatkan.

**Flag : gemastik13{1d4t_s1z3\_4lw4ys_s4m3\_3xc3pt_l45t_0N3}**

## Mix
### Around The World
Tujuan soal ini adalah mencari potongan-potongan flag sesuai dengan instruksi dan clue yang diberikan pada soal.

**Potongan Pertama**  
Pada announcement di discord gemastik didapatkan direct link dan hasilnya kami di arahkan menuju sebuah grup WhatsApp. Deskripsi di grup WA tadi kami diberikan sebuah petunjuk berikutnya sekaligus pecahan pertama.  
**Pecahan Pertama : BeRsamA**

![gemastik13-16](/assets/images/gemastik13/16.png)  

**Potongan Kedua**  
Setelah mendownload file dari google drive tadi, di dalam file tersebut terdapat zip yang dikunci, dan pdf yang berisi text berwarna putih. Didapatkan pecahan kedua sekaligus petunjuk untuk pecahan selanjutnya pada pdf tersebut.  
**Pecahan Kedua : beRJuANG**

![gemastik13-16](/assets/images/gemastik13/17.png)  

**Potongan Ketiga**  
Extract zip dengan password yang didapatkan, terdapat private key bernama myPrivate.ppk yang di generate dari PuTTYGen. Lalu convert menjadi openssh key dan connect ke ssh nya dengan user gema@180.250.135.6 dan menggunakan private key yang sudah diconvert tadi.  
Sebelumnya kami menemukan banyak junk folder pada direktori home gema dan terdapat 1 file bernama “hehe.txt” berisi bash history, namun kami stuck pada soal ini dan kami lewati sembari mengerjakan soal yang lainnya, kami mencatat beberapa informasi yang mungkin penting.  

Selang beberapa jam kami mencoba untuk connect kembali namun sayangnya direktori home pada soal ini telah diacak - acak dan dihapus oleh peserta lain yang iseng karna permission diatur full untuk user gema :(

Untungnya kami telah mencatat sebagian isi dari "hehe.txt" tadi, dan informasi yang kami dapatkan adalah :  

Base64 berisi link grup telegram, lalu ada kunci.db pada /folder7/kunci.db yang ternyata berisi potongan ke 3  

base64 decode UmFpaA== di kunci.db : Raih  
**Pecahan Ketiga : Raih**  

![gemastik13-18](/assets/images/gemastik13/18.png)  

**Potongan Keempat**  
Decode link telegram (dapat link telegram dari hehe.txt isinya dump dari bash history, ada base64 lalu di decode dapat link telegram). Tadinya disini tidak ada apa-apa namun setelah beberapa jam base64 dari potongan flag ke 4 ditaruh pada deskripsi grup (mungkin adminnya lupa hehe).  
**Pecahan Keempat : keMENANGan**  

![gemastik13-19](/assets/images/gemastik13/19.png)  

**Final Flag : gemastik13{BeRsamA_beRJuANG_Raih_keMENANGan}**