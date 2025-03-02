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

 Bu ve bahsetmediğim diğer eklentiler hakkında daha detaylı bilgi edinmek isterseniz sizi [buraya](https://github.com/hesitationIsDefeat/RISC-V-Plaground/blob/main/modules.md) davet ediyorum.

 ### Registerlar

 Daha önce register'ların işlemci içerisinde bulunan küçük bellek alanları olduğundan bahsetmiştim. RISC-V mimarisinde bu arkadaşlardan temel olarak 32 adet bulunuyor. Bunlara integer registers dendiğine de şahit olabilirsiniz. Nitekim floating-point işlemlerin yapılabilmesi için bir 32 tane daha eklememiz gerekiyor ancak ben olabildiğince temele odaklacağım, bu sebepten biz temel 32 ile ilgileniyor olacağız. Bu arkadaşları inceleyelim.

#### Genel Amaçlı Registerlar (GPR)

| İsim  | Takma Adı | Kullanım Amacı | 
|-------|-------|---------| 
| x0    | zero  | Her zaman 0 | 
| x1    | ra    | Dönüş adresi |
| x2    | sp    | Stack pointer |
| x3    | gp    | Global pointer |
| x4    | tp    | Thread pointer |
| x5    | t0    | Geçici register 0 |
| x6    | t1    | Geçici register 1 |
| x7    | t2    | Geçici register 2 |
| x8    | s0/fp | Kaydedilmiş register 0 / Frame pointer |
| x9    | s1    | Kaydedilmiş register 1 |
| x10   | a0    | Fonksiyon argümanı 0 / Dönüş değeri |
| x11   | a1    | Fonksiyon argümanı 1 / Dönüş değeri |
| x12   | a2    | Fonksiyon argümanı 2 |
| x13   | a3    | Fonksiyon argümanı 3 |
| x14   | a4    | Fonksiyon argümanı 4 |
| x15   | a5    | Fonksiyon argümanı 5 |
| x16   | a6    | Fonksiyon argümanı 6 |
| x17   | a7    | Fonksiyon argümanı 7 |
| x18   | s2    | Kaydedilmiş register 2 |
| x19   | s3    | Kaydedilmiş register 3 |
| x20   | s4    | Kaydedilmiş register 4 |
| x21   | s5    | Kaydedilmiş register 5 |
| x22   | s6    | Kaydedilmiş register 6 |
| x23   | s7    | Kaydedilmiş register 7 |
| x24   | s8    | Kaydedilmiş register 8 |
| x25   | s9    | Kaydedilmiş register 9 |
| x26   | s10   | Kaydedilmiş register 10 |
| x27   | s11   | Kaydedilmiş register 11 |
| x28   | t3    | Geçici register 3 |
| x29   | t4    | Geçici register 4 |
| x30   | t5    | Geçici register 5 |
| x31   | t6    | Geçici register 6 |

> Burada göstermiş olduğum takma adlar ve kullanım amaçları ABI tarafından belirleniyor. ABI (Application Binary Interface), yazılımı oluşturan parçaların binary seviyede nasıl etkileşeceğini belirleyen kurallar bütünüdür. Bu kurallar register'lar hakkında convention'lar belirlemekle beraber bellek düzeni ve işletim sistemi interaksiyonları gibi geniş bir çerçeveyi etkiler. Şimdilik bu kadarını bilmemiz yeterli.

Şimdi bu Register'ların genel özelliklerine bakacak olursak:

- Her bir register'ın sahip olduğu bellek aynı ve kullanılan mimariye bağlı olarak 32, 64 ya da 128-bit miktarda belleğe sahip oluyorlar.

- Her bir register'ın xi şeklinde bir adı ve kullanım amacını belirten bir de takma adı var. Assembly yazarken register'a atıfta bulunurken istediğimizi kullanabiliriz. Ben takma adın kullanımının özellikle okunurluğu ciddi miktarda arttırmasından dolayı tercih ve tavsiye ediyorum.

- İlk 5 register'ı ayrı tutacak olursak; a, s ve t register'larını birbirlerinin yerine kullanmak mümkün ancak işleri kendimiz için daha da zor hale getirmemek adın ABI'de belirtildiği şekilde kod yazmak en doğrusu olacaktır. 

#### Özel Registerlar

| İsim  | Kullanım Amacı |
|-------|---------|
| pc    | Program Counter |
| f0-f31 | Floating-Point Registers |
| csr   | Control and Status Registers (various) |

pc register'ında işlenecek bir sonraki komutun adresi bulunur. f tipi register'lar [F eklentisi](https://github.com/hesitationIsDefeat/RISC-V-Plaground/blob/main/registers.md) ile gelen floating-point işlemler için kullanılıyorlar. csr register'ları ise sistem ve işletim sistemi ile alakalı şu an için görece karmaşık işlemler için kullanılıyorlar.

Register'lar ile ilgili daha detaylı bilgi edinmek için [buraya](https://github.com/hesitationIsDefeat/RISC-V-Plaground/blob/main/registers.md) bakabilirsiniz.

### RISC-V Assembly Komutları

RISC-V Assembly yazarken bize sunulan komutlardan faydalanıyoruz. Komutlardan kastım bizim için önceden tanımlanmış, işlemcinin binary bir formata dönüştürmeyi bildiği makine komutlarından bahsediyorum. Bu sayede teker teker 0 ve 1 girmek yerine çok daha hızlı bir şekilde, (benim tecrübeme göre) human readable olan en alt katmanda ne istediğimizi beyan edebiliyoruz.

#### Komut Tipleri

Toplamda 6 adet komut tipimiz var. Bu komut tipleri, yerine getirdikleri komut tipine göre sınıflandırılmış durumdalar:

1. R-Tipi Komutlar (Register-Register): Register'ların değerlerini kullanarak bir register'a değer atama işlemleri yaparken kullanılan komutlar

2. I-Tipi Komutlar (Immediate): Register'lara sabit bir değer atama işlemlerinde kullanılan komutlar

3. U-Tipi Komutlar (Upper Immediate): 12 bit'ten daha fazla yer kaplayan sabit değerleri içeren işlemlerde kullanılan komutlar

4. S-Tipi Komutlar (Save): Bellekten okuma ya da belleğe yazma işlemlerinde kullanılan komutlar

5. J-Tipi Komutlar (Jump): Belirtilen bir adresteki komutu uygulamak için kullanılan komutlar

6. B-Tipi Komutlar (Branch): Şartlı jump (bir üstteki tip) komutlarını uygulamak için kullanılan komutlar

#### Komutların Binary Forma Dönüşümü

Yukarıda bahsi geçen her bir komutun işlemci tarafından anlaşılması için işlemciye binary formda sunulması gerekiyor. Bu binary form ise her komut tipi için farklı bir düzene sahip. Her birine kısaca göz atalım: 

| **Komut Tipi**  | **Komut Formu**                                       | **Binary Formu** |
|-----------|------------------------------------|-------------| 
| **R-Type** | `opcode rd, rs1, rs2`             | func7[25:31] rs2[20:24] rs1[15:19] func3[12:14] rd[7:11] opcode[0:6] |
| **I-Type** | `opcode rd, rs1, imm`             | imm[31:20] rs1[15:19] func3[12:14] rd[7:11] opcode[0:6] |
| **U-Type** | `opcode rd, imm`                  | imm[31:12] rd[7:11] opcode[0:6] |
| **S-Type** | `opcode rs2, offset(rs1)`         | imm[25:31] rs2[20:24] rs1[15:19] func3[12:14] imm[7:11] opcode[0:6] |
| **J-Type** | `opcode rd, offset`               | imm[12:19] imm[11] imm[1:10] imm[20] rd[7:11] opcode[0:6] |
| **B-Type** | `opcode rs1, rs2, offset`         | imm[12] imm[10:5] rs2[20:24] rs1[15:19] func3[12:14] imm[4:1] imm[11] opcode[0:6] |



> Little-Endian: Teknik tanımına bakacak olursak little-endianness kıymeti en az olan byte'ın en küçük adreste tutulması olarak tanımlanıyor. Bir örnek üzerinden gidelim:

```
lw t0, 4(sp) -> 000000000100 00010 010 00101 0000011 binary formu -> 0x00412203 hex formu
```
 
Burada girdiğimiz komutu derlendiğinde bu binary (ve dolayısıyla hex) formuna dönüşüyor. Bu komutu belleğe kaydetmek istediğimizde

```
0x00412203 komut hex formu -> 0x03224100 bellekteki komut hex formu
```

şekilde ters çeviriyoruz ve belleğe

```
0x1000 -> 0x03
0x1001 -> 0x22
0x1002 -> 0x41
0x1003 -> 0x00
```

şeklinde kaydediyoruz. Dolayısıyla byte sırası tersine dönüyor, ancak byte'ın içerisindeki bitlerin sırası aynı kalıyor. Komutumuzun uygulanma zamanı geldiğinde işlemci bu komutu soldan sağa olacak şekilde, yani en küçük bellek adresinden başlayarak byte'lar halinde çekiyor. Bu örnekte ilk çektiğimiz byte 0x03 ve içerisinde opcode bilgisini içeriyor. Bu da işlemcimize diğer byte'ları çekerken aynı zamanda elde ettiği opcode bilgisi sayesinde uygulanacak komut hakkında ön bilgi sahibi olmasını sağlıyor. Bu sayede bir komutun uygulanması için gerekli süreçleri paralelde hallederek geçen süreyi kısaltmış oluyor.

#### Pseudo Komutlar

Komutlardan bahsetmişken pseudo komutlara yer vermemek olmaz. Pseudo komutlar, makine komutlarının aksine, direkt olarak binary forma dönüşemeyen; assembler tarafından assembly time'da tekabül ettiği makine komutlarına dönüşen komutlardır. Bir örnekle daha açık bir hale getireyim.

t0 register'ına 12-bit in üzerinde bir değer, mesela 100000 değerini atamak istediğimizi varsayalım. Bu durumda

```
li t0, 100000
```

pseudo komutunu kullanabiliriz. Bunu makine komutlarını kullanarak başarmak istediğimizde

```
lui t0, 0x186A0   # upper 20 bits 
addi t0, t0, 0     # lower 12 bits
```

komularını çağırmamız gerekiyor.

Pseudo komutlar sayesinde assembly kodu yazma hızımızı arttırmanın yanında kodumuzun daha rahat okunur bir hale getiriyoruz.

### Gelecek Sefer:
Gelecek yazımda RISC-V'in modüler yapısından, register'lardan ve temel komutlardan bahsedeceğim.

Sağlıcakla kalın!
