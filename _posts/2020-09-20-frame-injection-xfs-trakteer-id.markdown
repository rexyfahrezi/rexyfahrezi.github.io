---
layout: post
title:  "Cross-Frame Scripting (XFS) Pada Trakteer.id [Bug Bounty]"
date:   2020-09-20 18:13:51 +0700
categories: bug-bounty
tags: [Bug Bounty]
---

![trakteer_logo](https://trakteer.id/images/mix/navbar-logo-lite-beta.png "Trakteer.id")
## Intro
Pertama kita bahas dulu apa itu Cross-Frame Scripting (XFS) ini.  
Berikut adalah deskripsi XFS menurut owasp.org

> Cross-Frame Scripting (XFS) is an attack that combines malicious JavaScript with an iframe that loads a legitimate page in an effort to steal data from an unsuspecting user. This attack is usually only successful when combined with social engineering.

Intinya XFS atau bisa juga disebut IFrame Injection Attack adalah serangan yang menggunakan iframe sebagai media *attacker* untuk melakukan serangannya. *Attacker* menyisipkan malicious kode javascript pada iframe tersebut untuk melakukan serangan tanpa sepengetahuan *victim* atau korban.  
IFrame Injection memungkinkan *attacker* untuk mengarahkan korban ke situs web berbahaya lainnya yang digunakan untuk menyebarkan malware, phishing dan banyak serangan lain yang serupa. IFrame Injection terjadi ketika vulnerable web page berhasil menampilkan web page lain via user controllable input.

Biasanya XFS ini dilakukan pada web server nya *attacker* dan memasukan *legitimate website* pada iframe didalam web *attacker* tersebut. Jadi korban ditipu seolah-olah ia sedang mengunjungi web yang asli padahal sebenarnya itu bukan.  

Namun pada kasus kali ini implementasi nya terbalik dengan yang saya sebutkan diatas. Jadi saat korban mengunjungi page asli, page tersebut akan me-load web nya si *attacker* via iframe pada web asli tersebut.  

## Attack Scenario

Nah sekarang saya akan mencoba untuk melakukan serangan dengan skenario seperti ini :
- *"Attacker"* membuat malicious web yang berisi virus/malware dan akan mendownload secara otomatis ketika web tersebut dibuka
- *"Attacker"* melakukan XFS pada web profil trakteer.id untuk memasukkan web nya
- Ketika korban membuka profil trakteer.id nya *"attacker"*, malicious script akan berjalan secara otomatis
- Malware/Virus pun terdownload pada device korban

## Affected Endpoint
- /manage/my-page/[redacted]

## Steps to Reproduce
- Pergi ke "My Page", klik "Edit Video"
- Masukkan URL Youtube yang valid
- Intercept Request
![xfs-trakteer-1](/assets/images/trakteer/1.png)  

- Rubah “video_url“ ke url malicious web yang sudah dibuat oleh *attacker*
![xfs-trakteer-1](/assets/images/trakteer/2.png)  

- Berikut adalah response yang diterima
![xfs-trakteer-1](/assets/images/trakteer/3.png)  

- Url berhasil di upload dan page akan melakukan redirect
- Untuk membuktikannya, pergi ke halaman Profile atau “View Page”
![xfs-trakteer-1](/assets/images/trakteer/4.png)  

- Saat profil trakteer terbuka (trakteer.id/kirakira), maka secara otomatis “malicious web” milik si *“attacker”* pun ter-load pada iframe **"about"** dan akan mendownload “evil.exe” yang telah disisipi pada web si *"attacker"*.
![xfs-trakteer-1](/assets/images/trakteer/5.png)  

- Berikut isi script pada malicious web *attacker*
![xfs-trakteer-1](/assets/images/trakteer/6.png)  
Disclaimer : Saya menggunakan url “sciensec.github.io” (“Malicious web” yang saya buat) hanya sebagai contoh untuk membuktikan impact dari bug ini.  

Berikut adalah demonstrasi pada device korban ketika profil penyerang di akses oleh korban :

<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed//onTkFCb1YcA' frameborder='0' allowfullscreen></iframe></div>


## Impact
- Penyerang mungkin menggunakan kerentanan ini untuk mengarahkan pengguna ke situs web berbahaya lainnya yang digunakan untuk phishing dan serangan serupa. Selain itu, mereka mungkin menempatkan form login palsu di frame, yang dapat digunakan untuk mencuri kredensial dari pengguna Anda.

- Perlu dicatat bahwa penyerang juga dapat menyalahgunakan frame yang diinjeksi untuk menghindari mekanisme client side security tertentu. 

- Jika penyerang menggunakan javascript: URL sebagai atribut src dari iframe, kode JavaScript berbahaya dijalankan di bawah origin situs web yang rentan. Namun, ia memiliki akses ke window objek baru tanpa fungsi yang ditimpa.

## Remediation
- Jika memungkinkan jangan gunakan user input untuk URL.
- Pastikan web hanya menerima URL dengan domain yang sesuai.
- Gunakan CSP untuk whitelist iframe source URLs.

## Timeline
- 28 Aug 2020 : Reported to Trakteer.id
- 1 Sep 2020 : Trakteer.id confirm the report
- 3 Sep 2020 : The bug has been fixed
- 6 Sep 2020 : Trakteer.id sent merchandise & put my name on their HoF as appreciation

## Hall of Fame
Hall of Fame Trakteer.id bisa diakses di sini :
- [trakteer.id/our-heroes](https://trakteer.id/our-heroes)

## Reference 
Beberapa link yang mungkin berguna sebagai referensi :
- [https://owasp.org/www-community/attacks/Cross_Frame_Scripting](https://owasp.org/www-community/attacks/Cross_Frame_Scripting)
- [https://www.checkmarx.com/knowledge/knowledgebase/XFS](https://www.checkmarx.com/knowledge/knowledgebase/XFS)
- [https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/frame-injection/](https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/frame-injection/)
- [https://www.netsparker.com/blog/web-security/frame-injection-attacks/](https://www.netsparker.com/blog/web-security/frame-injection-attacks/)
- [https://medium.com/bugbountywriteup/when-i-found-iframe-injection-and-illegal-redirect-dom-based-cfbbcec21a7](https://medium.com/bugbountywriteup/when-i-found-iframe-injection-and-illegal-redirect-dom-based-cfbbcec21a7)
