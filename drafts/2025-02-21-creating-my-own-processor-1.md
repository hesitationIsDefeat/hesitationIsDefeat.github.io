---
title: "Creating My Own Processor #1"
description: "Modularity, Registers and Instructions of RISC-V"
image: "/assets/img/article/Leonardo_Phoenix_10_A_cinematic_black_and_white_photograph_of_1.jpg"
date: 2025-02-21 20:00:00 +0300
categories: [processor, risc-v]
tags: [assembly, risc-v, low level programming, processor, isa]     
---

## Modularity, Registers and Instructions of RISC-V
***

Herkese selamlar. Son yazımda bahsettiğim üzere bu hafta RISC-V'ın modüler yapısından bahsederek başlayacağım. Devamında RISC-V mimarisinde bulunan register'lar üzerinde duracağım. Bu konuların üzerine cila olması için son olarak RISC-V Assembly komutlarından bahsederek yazıyı tamamlayacağım. İyi okumalar!

Başlamadan önce, aldığım notları ve yazılarımda verdiğim örneklerin daha birçoğunu eklediğim bir repo oluşturdum. [Buradan](https://github.com/hesitationIsDefeat/RISC-V-Plaground) ulaşabilirsiniz. Ayrıca ileride yazdığım projeleri de bu repoya ekliyor ve içeriği zenginleştiriyor olacağım.

### Sayın RISC-V, neden modülerlik?

RISC-V mimarisindeki modüllerin ne olduğundan bahsetmeden önce bu modüllerin neden olduğunu anlamamız gerekiyor. Bunun başlıca nedenleri şu şekilde:

1. **Minimal Başlangıç Noktası:** Son yazımda RISC'in temel özelliklerinden birinin basit komutlar içermesi olduğundan bahsetmiştim. RISC-V'ın modüler yapısı, bize bir başlangıç noktası olarak en minimal modülü (RV32I) sunuyor. Bu sayede ihtiyacımız olmadığı ve bilerek yeni bir modül eklemediğimiz takdirde basit komutlar ile çalışıyor ve RISC felsefeine uymaya devam ediyoruz.  

2. **Özel Eklentiler:** Bize yeni özellikler kazandıran hazır modüller ekleyebildiğimiz gibi, dilersek kendi istediğimiz özellikleri barındıran yeni modüller oluşturabiliyor ve diğer modüller ile birlikte kullanabiliyoruz. (Yaşasın açık kaynak!)

3. **Ölçeklenebilirlik:** Ister oldukça basit bir kullanımı olan gömülü sistem işlemcisi üzerinde çalışıyor olalım, istersek de oldukça karmaşık bir yapısı olan bir bilgisayar işlemcisi yazıyor olalım; RISC-V'ı dilediğimiz şekilde ölçeklendirerek kullanım amacımıza uygun hale getirebiliyoruz.

### Temeller ve Modüller

RISC-V mimarisini 4 farklı temelden başlatma şansına sahibiz. Her birinin kendine göre özellikleri mevcut. Bu temeller ve özellikleri şu şekilde:

 1. RV32I: Toplama/çıkarma, mantık, yükleme/kaydetme gibi en temel özellikleri içeren, 32-bit bir mimaridir. 

 2. RV32E: RV32I'nin gömülü sistemler için özelleşmiş, küçük mikroişlemcilerde daha iyi performans ile çalışan halidir; 32 yerin 16 adet register içerir.

 3. RV64I: RV32I'nin 64-bit sistemler için özelleşmiş halidir, her bir register ve bellek adresi 64-bit'tir.

 4. RV128I: RV64I'nin 128-bit sistemler için özelleşmiş halidir, her bir register ve bellek adresi 128-bit'tir.

 Tabi ki sadece bu özellikler ile sınırlı değiliz. Hedefimize en uygun temeli seçtikten sonra üzerine ekleyeceğimiz eklentiler ile kendimize yeni özellikler katma şansına sahibiz. Bu eklentilerden bazılarını incelemk gerekirse:

 - M: Çarpma, bölme, kalan gibi aritmetik işlemleri içerir

 - A: "Atomic" operasyon desteği içerir

 - C: Sıkıştırılmış komutları içerir

 - F: 32-bit floating-point işlemleri içerir

 - S: İşletim sistemi desteği içerir

 Bu ve bahsetmediğim diğer eklentiler hakkında daha detaylı bilgi edinmek isterseniz sizi [buraya]() davet ediyorum.

 ### Registerlar

 Daha önce register'ların işlemci içerisinde bulunan küçük bellek alanları olduğundan bahsetmiştim. RISC-V mimarisinde bu arkadaşlardan temel olarak 32 adet bulunuyor. Bunlara integer registers dendiğine de şahit olabilirsiniz. Nitekim floating-point işlemlerin yapılabilmesi için bir 32 tane daha eklememiz gerekiyor ancak ben olabildiğince temele odaklacağım, bu sebepten biz temel 32 ile ilgileniyor olacağız.