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

Herkese selamlar. Son yazımda bahsettiğim üzere bu hafta RISC-V'ın modüler yapısından bahsederek başlayacağım. Devamında RISC-V mimarisinde bulunan register'lar üzerinde duracağım. Bu konuların üzerine cila olması için son olarak RISC-V Assembly komutlarından bahsederek yazıyı tamamlayacağım. Iyi okumalar!

Başlamadan önce, aldığım notları ve yazılarımda verdiğim örneklerin daha birçoğunu eklediğim bir repo oluşturdum. [Buradan](https://github.com/hesitationIsDefeat/RISC-V-Plaground) ulaşabilirsiniz. Ayrıca ileride yazdığım projeleri de bu repoya ekliyor ve içeriği zenginleştiriyor olacağım.

### Sayın RISC-V, neden modülerlik?

RISC-V mimarisindeki modüllerin ne olduğundan bahsetmeden önce bu modüllerin neden olduğunu anlamamız gerekiyor. Bunun başlıca nedenleri şu şekilde:

1. **Minimal Başlangıç Noktası:** Son yazımda RISC'in temel özelliklerinden birinin basit komutlar içermesi olduğundan bahsetmiştim. RISC-V'ın modüler yapısı, bize bir başlangıç noktası olarak en minimal modülü (RV32I) sunuyor. Bu sayede ihtiyacımız olmadığı ve bilerek yeni bir modül eklemediğimiz takdirde basit komutlar ile çalışıyor ve RISC felsefeine uymaya devam ediyoruz.  

2. **Özel Eklentiler:** Bize yeni özellikler kazandıran hazır modüller ekleyebildiğimiz gibi, dilersek kendi istediğimiz özellikleri barındıran yeni modüller oluşturabiliyor ve diğer modüller ile birlikte kullanabiliyoruz. (Yaşasın açık kaynak!)

3. **Ölçeklenebilirlik:** Ister oldukça basit bir kullanımı olan gömülü sistem işlemcisi üzerinde çalışıyor olalım, istersek de oldukça karmaşık bir yapısı olan bir bilgisayar işlemcisi yazıyor olalım; RISC-V'ı dilediğimiz şekilde ölçeklendirerek kullanım amacımıza uygun hale getirebiliyoruz.

work in progress...