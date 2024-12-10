---
layout: post
title:  "Cyber Jawara 2020 [Write-up]"
date:   2020-09-18 11:13:25 +0700
categories: [security, ctf]
tags: [ctf]
---
![cj-logo](https://cyberjawara.id/images/cb-logo.png "Cyber Jawara")
## Cyber Jawara
Adalah kompetisi keamanan siber nasional dengan metode online dan on-site. 
Kompetisi CYBER JAWARA ini memainkan permainan Computer Network Defence, Penetration Test, Capture The Flag dan Forensic Analysis.  
Babak eliminasi CJ 2020 dilaksanakan dari tanggal 15 September 2020 - 16 September 2020. Berikut adalah writeup dari beberapa soal yang berhasil saya dan tim solve.  

## Web Exploitation
### AWS
![cj2020-1](/assets/img/cj2020/1.png)  
Isi dari file Credentials :

```credentials
[default]
aws_access_key_id = AKIA6QOBT5TWKXCV6PUO
aws_secret_access_key = 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
```

Disini tujuannya sudah cukup jelas kita hanya perlu untuk mengakses bucket tersebut dan melihat isinya, langsung saja akses ke aws s3 bucket tersebut menggunakan credential yang sudah disediakan.  
Pertama saya coba untuk membuka web tersebut tanpa menggunakan credentialnya :

![cj2020-2](/assets/img/cj2020/2.png)  

Ternyata yang didapatkan adalah AccessDenied, lalu saya menggunakan AWS CLI untuk mengaksesnya:

![cj2020-3](/assets/img/cj2020/3.png)  

Terdapat file flag-c72411d2642162555c7010141be4f0bd.txt disitu, lalu download ke local dengan command aws s3 cp dan didapatkan flagnya.  


**Flag : CJ2020{so_many_data_breaches_because_of_AWS_s3}**

### Toko Masker 1
![cj2020-4](/assets/img/cj2020/4.png)  
Jadi intinya pada soal ini kita harus membeli masker n99 sebanyak 100 buah pada web tersebut untuk mendapatkan flagnya, namun harga 1 maskernya <span>$100</span>, sedangkan balance yang kita punya hanya <span>$100.</span>

![cj2020-5](/assets/img/cj2020/5.png)  

Intercept request nya, dapat dilihat terdapat variable 
‘price’ dan ‘quantity’ :

![cj2020-6](/assets/img/cj2020/6.png)  

Rubah ‘price’ menjadi 1 dan ‘quantity’ menjadi 100 :

![cj2020-7](/assets/img/cj2020/7.png)  

Harga masker pun berubah menjadi <span>$1</span> perbuah nya :

![cj2020-8](/assets/img/cj2020/8.png)  

Langsung saja checkout barangnya dan didapatkan flagnya :

![cj2020-9](/assets/img/cj2020/9.png)  

**Flag : CJ2020{ez_price_tampering_for_bonus}**

### Toko Masker 2
![cj2020-10](/assets/img/cj2020/10.png)  

Sama seperti Toko Masker 1, kita diminta untuk membeli masker n99 sebanyak 100 buah, namun sekarang ketika kita tamper value dari variable ‘price’ dan ‘quantity’ price tidak akan berubah.

![cj2020-11](/assets/img/cj2020/11.png)  

Kita analisis dulu menggunakan request dan response yang diberikan.
Pertama pada **/api/v1/getState** kita mengirimkan request berisi json data lalu di response data yang kita kirim tadi dirubah menjadi “state”.

![cj2020-12](/assets/img/cj2020/12.png)  

Selanjutnya pada **/api/v1/getSelectedItems** kita mengirimkan “state” yang didapatkan sebelumnya dan akan mendapatkan response kembali, namun kali ini variable “price” tetap 100.

![cj2020-13](/assets/img/cj2020/13.png)  

Sedangkan di soal sebelumnya (Toko Masker 1) kita bisa mendapatkan “state” yang bisa dirubah price nya. Jadi rencana nya kita akan craft request ke **tokomasker1.web.cyber.jawara.systems/api/v1/getState** untuk membuat “state” yang berisi “price: 1”

![cj2020-14](/assets/img/cj2020/14.png)  

```state
{"state": "iLA3sw5MkwVQVLzXbrZcbStgA2EI7vX6XSHfkMh4wO03VXuTpfDsnL9ZfeUYrfdAak2phm4Wj5yjAJ+m5p12EcCRt808tFwlcpcZS9pV3rQCr5Dk6TeTqUTkXcr1za2Ex12UtTdSjEC3ojyQrC9T7l0fZmWH9GlH/D5xp++yVLD6StTXQ6YnL04CNiypaKRk"}
```

Lalu kita gunakan “state” tersebut untuk membuat post request ke **api/v1/getSelectedItems** pada toko masker 2.

![cj2020-15](/assets/img/cj2020/15.png)  

Price pun berhasil dirubah, terakhir hanya tinggal membuat request ke **api/v1/getInvoice** untuk mendapatkan flagnya.

![cj2020-16](/assets/img/cj2020/16.png)  

Dan flagpun berhasil didapatkan.

**Flag : CJ2020{another_variant_of_price_tampering_from_real_case}**

## Digital Forensic

### FTP
![cj2020-17](/assets/img/cj2020/17.png)  

Diberikan file pcap buka dengan wireshark, terdapat paket ftp icmp, lakukan filter ftp-data :

![cj2020-18](/assets/img/cj2020/18.png)  

Setelah melakukan filter ftp-data, lakukan pengamatan pada line-base text, setiap nilai berubah berdasarkan STOR :

![cj2020-19](/assets/img/cj2020/19.png)  

Kemudian lakukan pengurutan berdasarkan STOR , dan susun flag nya.

![cj2020-20](/assets/img/cj2020/20.png)  

Tiap karakter pada flag terdapat pada bagian “Line-based text data”.

**Flag : CJ2020{plzuse_tls_kthxx}**

### Image PIX
![cj2020-21](/assets/img/cj2020/21.png)  

Diberikan file image bernama pix.png, berikut image nya :

![cj2020-22](/assets/img/cj2020/22.png)  

lakukan analisis menggunakan zsteg dan disini kami sekaligus mencoba memasukan format flag "CJ2020" :

```bash
zsteg -a pix.png | grep -i "CJ2020"
```

![cj2020-23](/assets/img/cj2020/23.png)  

Dan didapatkan flagnya.

**Flag : CJ2020{A_Study_in_Scarlet}**

### Home Folder
![cj2020-24](/assets/img/cj2020/24.png)  

Berikut isi dari file cj.zip yang diberikan :

![cj2020-25](/assets/img/cj2020/25.png)  

Disitu terdapat “flag.zip” namun file tersebut di password, dan kemungkinan passwordnya ada di dalam “pass.txt” ,dan ternyata passwordnya salah :

![cj2020-26](/assets/img/cj2020/26.png)  

Kita analisis dulu file “.bash_history” , kemungkinan ada petunjuk disitu :

![cj2020-27](/assets/img/cj2020/27.png)  

Dan benar saja, ternyata password dari **“flag.zip”** ada pada **“pass.txt”**, namun **“pass.txt”** tersebut telah di modifikasi dengan command **“truncate”**.

```bash
“truncate –s -2 pass.txt”
```

Setelah mencari informasi dan membaca panduan tentang command “truncate” ternyata “-s” digunakan untuk mengatur atau sesuaikan ukuran file dengan SIZE byte.

![cj2020-28](/assets/img/cj2020/28.png)  

Jadi intinya ada karakter yang hilang pada file **“pass.txt”** tersebut, maka solusinya adalah dengan brute password zip tersebut.  

Pertama kita buat wordlist nya berdasarkan **“pass.txt”** tersebut, karna passwordnya adalah hexadecimal, maka kita hanya butuh 0-9,a-f:

```bash
crunch 32 32 0123456789abcdef -t c10a41a5411b992a9ef7444fd6346a4% -t c10a41a5411b992a9ef7444fd6346a4@ -o word
```

![cj2020-29](/assets/img/cj2020/29.png)  

Lalu tinggal crack zip tersebut menggunakan wordlist yang sudah dibuat :

```bash
fcrackzip -b -D -p word -u flag.zip
```
![cj2020-30](/assets/img/cj2020/30.png)  

Password pun didapatkan : **c10a41a5411b992a9ef7444fd6346a44**

Unzip flag.zip menggunakan password yang sudah didapat, dan flag pun didapatkan.

![cj2020-31](/assets/img/cj2020/31.png)  

**Flag : CJ2020{just_to_check_if_you_are_familiar_with_linux_or_not}**

## Reverse Engineering
### Snake 2020
![cj2020-32](/assets/img/cj2020/32.png)  

Diberikan file Snake.jar yang merupakan sebuah game, berikut gamenya :

![cj2020-33](/assets/img/cj2020/33.png)  

Sesuai dengan soal sepertinya kita harus mencapai score 8172 untuk mendapatkan flagnya, namun terdapat hint tidak bisa menggunakan cheat engine.
Jadi saya coba untuk decompile dulu jar tersebut untuk melihat source code dan menganalisis nya.
Terdapat beberapa class didalamnya :

![cj2020-34](/assets/img/cj2020/34.png)  

Setelah dianalisis, terdapat variable MILESTONE yang saya asumsikan berisi score yang harus dicapai agar bisa mendapatkan flag nya :

![cj2020-35](/assets/img/cj2020/35.png)  

Lalu setelah di analisis lagi, fungsi Update() digunakan untuk menambahkan point pada game ketika ular berhasil memakan cherry nya :

![cj2020-36](/assets/img/cj2020/36.png)  

Nah disitu terdapat beberapa baris kode yang menarik :

![cj2020-37](/assets/img/cj2020/37.png)  

Jadi ketika nilai **“this.pivot > 0”** maka variable **“letters”** akan diisi dengan nilai yang didapatkan dari **this.pivot** dikurangi dengan **this.lastPivot** yang lalu dirubah menjadi char.

Contoh ketika kita mencapai score 5271, maka :
```func
This.letters = this.letters + (char)(5721-5191)
```

Nilai Pivot sekarang dikurangi nilai Pivot sebelumnya, lalu dirubah ke char. Dan itu dimulai index ke 0 sampai terakhir pada variable MILESTONE.

![cj2020-35-2](/assets/img/cj2020/35.png)  

Lalu setelah menganalisis saya buat solvernya, berikut solver yang dibuat :

```solver.py
flag = ''
temp = 0
pivot = 0
c = ''

with open('milestone.txt') as f:
   for line in f:
       pivot = int(line) - temp
       temp = int(line)
       c = int(pivot)
       if c <= 256:
          flag += chr(c)
print(flag)
```

Isi dari array MILESTONE saya letakkan pada milestone.txt, lalu jalankan scriptnya dan didapatkan flagnya :

![cj2020-38](/assets/img/cj2020/38.png)  

Walaupun ada beberapa character yang tidak terbaca, namun isi flag tetap terlihat jelas.

**Flag : CJ2020{ch34t1ng_15_54t15fy1ng}**

### Baby Baby
![cj2020-39](/assets/img/cj2020/39.png)  

Diberikan sebuah binary, buka menggunakan IDA berikut isi dari fungsi main :

![cj2020-40](/assets/img/cj2020/40.png)  

Intinya kita harus memasukan 3 angka dimana angka tersebut nanti akan di cek pada line 12.

Ketika 3 angka tersebut benar, maka program akan menjalankan looping untuk melakukan XOR antara array lel[] dan 3 angka yang diinput tadi.

![cj2020-41](/assets/img/cj2020/41.png)  

Kita asumsikan lel[] tersebut berisi flag yang sudah di enkripsi dengan XOR. Karna flag hanya di XOR, maka jika kita XOR flag yang telah ter-encrypt dengan format flag asli (CJ2020) kita akan mendapatkan nilai dari v4, v5, dan v6 tersebut.

Jadi kita lihat dulu isi dari array lel[] tersebut :

![cj2020-42](/assets/img/cj2020/42.png)  

Lihat menggunakan Hex View, lalu ambil hexnya.

![cj2020-43](/assets/img/cj2020/43.png)  

Lalu XOR semua hex tersebut dengan format flag yang asli (CJ2020), berikut solver yang saya buat :

```solver.py
flag = 'CJ2020'
list = [0x58,0x1b,0x36,0x2b,0x63,0x34,0x60,0x33,0x30,0x5a,0x65,0x65,0x2f,0x13,0x46,0x79,0x33,0x33,0x62,0x28,0x79]

for i in range(len(flag)):
	print(ord(flag[i])^list[i])
```

Jalankan script tersebut dan didapatkan 3 angka yang kita cari. Lalu tinggal jalankan binary BabyBaby nya untuk mendapatkan flagnya :

![cj2020-44](/assets/img/cj2020/44.png)  

**Flag : CJ2020{b4A4a4BBbb7yy}**