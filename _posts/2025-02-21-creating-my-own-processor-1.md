---
title: "Creating My Own Processor #1"
description: "Modularity, Registers and Instructions of RISC-V"
image: "/assets/img/article/Leonardo_Phoenix_10_A_cinematic_black_and_white_photograph_of_1.jpg"
date: 2025-02-21 20:00:00 +0300
categories: [processor, risc-v]
tags: [assembly, risc-v, low level programming, processor, isa]     
---
[Go to English translation](#modularity-registers-and-instructions-of-risc-v)

## RISC-V'in Modülerliği, Registerları ve Talimatları
***

Herkese selamlar. Son yazımda bahsettiğim üzere bu sefer RISC-V'ın modüler yapısından bahsederek başlayacağım. Devamında RISC-V mimarisinde bulunan register'lar üzerinde duracağım. Son olarak RISC-V Assembly komutlarından ve bellek modelinden bahsederek yazıyı tamamlayacağım. İyi okumalar!

Başlamadan önce, aldığım notları ve yazılarımda verdiğim örneklerin daha birçoğunu eklediğim bir repo oluşturdum. [Buradan](https://github.com/hesitationIsDefeat/RISC-V-Plaground) ulaşabilirsiniz. Ayrıca ileride yazdığım projeleri de bu repoya ekliyor ve içeriği zenginleştiriyor olacağım.

### Sayın RISC-V, neden modülerlik?

RISC-V mimarisindeki modüllerin ne olduğundan bahsetmeden önce bu modüllerin neden olduğunu anlamamız gerekiyor. Bunun başlıca nedenleri şu şekilde:

1. **Minimal Başlangıç Noktası:** Son yazımda RISC'in temel özelliklerinden birinin basit komutlar içermesi olduğundan bahsetmiştim. RISC-V'ın modüler yapısı, bize bir başlangıç noktası olarak en minimal modülü (RV32I) sunuyor. Bu sayede ihtiyacımız olmadığı ve bilerek yeni bir modül eklemediğimiz takdirde basit komutlar ile çalışıyor ve RISC felsefeine uymaya devam ediyoruz.  

2. **Özel Eklentiler:** Bize yeni özellikler kazandıran hazır modüller ekleyebildiğimiz gibi, dilersek kendi istediğimiz özellikleri barındıran yeni modüller oluşturabiliyor ve diğer modüller ile birlikte kullanabiliyoruz. (Yaşasın açık kaynak!)

3. **Ölçeklenebilirlik:** Ister oldukça basit bir kullanımı olan gömülü sistem işlemcisi üzerinde çalışıyor olalım, istersek de oldukça karmaşık bir yapısı olan bir bilgisayar işlemcisi yazıyor olalım; RISC-V'ı dilediğimiz şekilde ölçeklendirerek kullanım amacımıza uygun hale getirebiliyoruz.

### Temeller ve Modüller

RISC-V mimarisini 4 farklı temelden başlatma şansına sahibiz. Her birinin kendine göre özellikleri mevcut. Bu temeller ve özellikleri şu şekilde:

 1. **RV32I**: Toplama/çıkarma, mantık, yükleme/kaydetme gibi en temel özellikleri içeren, 32-bit bir mimaridir. 

 2. **RV32E**: RV32I'nin gömülü sistemler için özelleşmiş, küçük mikroişlemcilerde daha iyi performans ile çalışan halidir; 32 register yerine 16 adet register içerir.

 3. **RV64I**: RV32I'nin 64-bit sistemler için özelleşmiş halidir, her bir register ve bellek adresi 64-bit'tir.

 4. **RV128I**: RV64I'nin 128-bit sistemler için özelleşmiş halidir, her bir register ve bellek adresi 128-bit'tir.

 Tabi ki sadece bu özellikler ile sınırlı değiliz. Hedefimize en uygun temeli seçtikten sonra üzerine ekleyeceğimiz eklentiler ile kendimize yeni özellikler katma şansına sahibiz. Bu eklentilerden bazılarını incelemk gerekirse:

 - **M**: Çarpma, bölme, kalan gibi aritmetik işlemleri içerir

 - **A**: "Atomic" operasyon desteği içerir

 - **C**: Sıkıştırılmış komutları içerir

 - **F**: 32-bit floating-point işlemleri içerir

 - **S**: İşletim sistemi desteği içerir

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

- Her bir register'ın `xi` şeklinde bir adı ve kullanım amacını belirten bir de takma adı var. Assembly yazarken register'a atıfta bulunurken istediğimizi kullanabiliriz. Ben takma adın kullanımının özellikle okunurluğu ciddi miktarda arttırmasından dolayı tercih ve tavsiye ediyorum.

- İlk 5 register'ı ayrı tutacak olursak; a, s ve t register'larını birbirlerinin yerine kullanmak mümkün ancak işleri kendimiz için daha da zor hale getirmemek adın ABI'de belirtildiği şekilde kod yazmak en doğrusu olacaktır. 

#### Özel Registerlar

| İsim  | Kullanım Amacı |
|-------|---------|
| pc    | Program Counter |
| f0-f31 | Floating-Point Registers |
| csr   | Control and Status Registers (various) |

`pc` register'ında işlenecek bir sonraki komutun adresi bulunur. `f` tipi register'lar F eklentisi ile gelen floating-point işlemler için kullanılıyorlar. `csr` register'ları ise sistem ve işletim sistemi ile alakalı şu an için görece karmaşık işlemler için kullanılıyorlar.

Register'lar ile ilgili daha detaylı bilgi edinmek için [buraya](https://github.com/hesitationIsDefeat/RISC-V-Plaground/blob/main/registers.md) bakabilirsiniz.

### RISC-V Assembly Komutları

RISC-V Assembly yazarken bize sunulan komutlardan faydalanıyoruz. Komutlardan kastım bizim için önceden tanımlanmış, işlemcinin binary bir formata dönüştürmeyi bildiği makine komutlarından bahsediyorum. Bu sayede teker teker 0 ve 1 girmek yerine çok daha hızlı bir şekilde, (benim tecrübeme göre) human readable olan en alt katmanda ne istediğimizi beyan edebiliyoruz.

#### Komut Tipleri

Toplamda 6 adet komut tipimiz var. Bu komut tipleri, yerine getirdikleri komut tipine göre sınıflandırılmış durumdalar:

1. **R-Tipi Komutlar (Register-Register)**: Register'ların değerlerini kullanarak bir register'a değer atama işlemleri yaparken kullanılan komutlar

2. **I-Tipi Komutlar (Immediate)**: Register'lara sabit bir değer atama işlemlerinde kullanılan komutlar

3. **U-Tipi Komutlar (Upper Immediate)**: 12 bit'ten daha fazla yer kaplayan sabit değerleri içeren işlemlerde kullanılan komutlar

4. **S-Tipi Komutlar (Save)**: Bellekten okuma ya da belleğe yazma işlemlerinde kullanılan komutlar

5. **J-Tipi Komutlar (Jump)**: Belirtilen bir adresteki komutu uygulamak için kullanılan komutlar

6. **B-Tipi Komutlar (Branch)**: Şartlı jump (bir üstteki tip) komutlarını uygulamak için kullanılan komutlar

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

#### Komutların Belleğe Yazılması

Yazdığımız komutların binary formlarının belleğe nasıl yazılacağı çalıştığımız mimarinin endianness'ına bağlı. RISC-V default olarak little-endian formatı destekliyor. Teknik tanımına bakacak olursak little-endianness kıymeti en az olan byte'ın en küçük adreste tutulması olarak tanımlanıyor.

Bir örnek üzerinden gidelim:

```
lw t0, 4(sp) # 000000000100 00010 010 00101 0000011 binary formu (0x00412203 hex formu)
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

t0 register'ına 12-bit'ten büyük bir değer, mesela 100000 değerini atamak istediğimizi varsayalım. Bu durumda

```
li t0, 100000
```

pseudo komutunu kullanabiliriz. Bunu makine komutlarını kullanarak başarmak istediğimizde

```
lui t0, 0x18    # üst 20 bit
addi t0, t0, 0x6A0   # alt 12 bit
```

komutlarını çağırmamız gerekiyor.

Pseudo komutlar sayesinde assembly kodu yazma hızımızı arttırmanın yanında kodumuzun daha rahat okunur bir hale getiriyoruz.

### Gelecek Sefer:
Gelecek yazımda RISC-V'in adresleme modlarından, privilege seviyelerinden ve güvenlik adreslerinden bahsedeceğim.

Sağlıcakla kalın!

## Modularity, Registers, and Instructions of RISC-V
***

Hello everyone. As mentioned in my last post, this time I will start by discussing the modular structure of RISC-V. Then, I will focus on the registers found in the RISC-V architecture. Finally, I will wrap up the post by discussing RISC-V Assembly commands and the memory model. Happy reading!

Before we begin, I have created a repository where I included many of the notes I took and examples I shared in my posts. You can access it [here](https://github.com/hesitationIsDefeat/RISC-V-Plaground). Additionally, I will continue adding my future projects to this repository, enriching the content.

### Dear RISC-V, why modularity?

Before discussing what the modules in the RISC-V architecture are, we need to understand why they exist. The main reasons for this are as follows:

1. **Minimal Starting Point:** In my previous post, I mentioned that one of the basic features of RISC is that it contains simple instructions. The modular structure of RISC-V gives us the minimal module (RV32I) as a starting point. This way, unless we add a new module deliberately, we work with simple instructions and continue to adhere to the RISC philosophy.

2. **Custom Extensions:** We can add predefined modules that bring new features or create new modules with our desired features and use them alongside other modules. (Long live open source!)

3. **Scalability:** Whether we are working on an embedded system processor with simple usage or writing a computer processor with a more complex structure, we can scale RISC-V as needed to make it fit our purpose.

### Fundamentals and Modules

We have the opportunity to start the RISC-V architecture from 4 different fundamentals. Each one has its own characteristics. These fundamentals and their features are as follows:

1. **RV32I:** A 32-bit architecture that includes the most basic features like addition/subtraction, logic, and load/store operations.

2. **RV32E:** A version of RV32I specialized for embedded systems, which works better on smaller microprocessors and contains 16 registers instead of 32.

3. **RV64I:** A version of RV32I customized for 64-bit systems, where each register and memory address is 64-bit.

4. **RV128I:** A version of RV64I customized for 128-bit systems, where each register and memory address is 128-bit.

Of course, we are not limited to these features. After choosing the base that best suits our goal, we have the chance to add new features with extensions. Some of these extensions include:

- **M:** Includes arithmetic operations like multiplication, division, and remainder
- **A:** Includes support for "atomic" operations
- **C:** Includes compressed instructions
- **F:** Includes 32-bit floating-point operations
- **S:** Includes operating system support

If you want to learn more about these extensions and others I haven’t mentioned, I invite you to check out [this link](https://github.com/hesitationIsDefeat/RISC-V-Plaground/blob/main/modules.md).

### Registers

Previously, I mentioned that registers are small memory areas inside the processor. In the RISC-V architecture, there are 32 of these registers. These are often referred to as integer registers. In order to perform floating-point operations, we need to add 32 more, but for now, I will focus on the basic 32. Let's take a look at them.

#### General Purpose Registers (GPR)

| Name  | Alias | Purpose | 
|-------|-------|---------| 
| x0    | zero  | Always 0 | 
| x1    | ra    | Return address |
| x2    | sp    | Stack pointer |
| x3    | gp    | Global pointer |
| x4    | tp    | Thread pointer |
| x5    | t0    | Temporary register 0 |
| x6    | t1    | Temporary register 1 |
| x7    | t2    | Temporary register 2 |
| x8    | s0/fp | Saved register 0 / Frame pointer |
| x9    | s1    | Saved register 1 |
| x10   | a0    | Function argument 0 / Return value |
| x11   | a1    | Function argument 1 / Return value |
| x12   | a2    | Function argument 2 |
| x13   | a3    | Function argument 3 |
| x14   | a4    | Function argument 4 |
| x15   | a5    | Function argument 5 |
| x16   | a6    | Function argument 6 |
| x17   | a7    | Function argument 7 |
| x18   | s2    | Saved register 2 |
| x19   | s3    | Saved register 3 |
| x20   | s4    | Saved register 4 |
| x21   | s5    | Saved register 5 |
| x22   | s6    | Saved register 6 |
| x23   | s7    | Saved register 7 |
| x24   | s8    | Saved register 8 |
| x25   | s9    | Saved register 9 |
| x26   | s10   | Saved register 10 |
| x27   | s11   | Saved register 11 |
| x28   | t3    | Temporary register 3 |
| x29   | t4    | Temporary register 4 |
| x30   | t5    | Temporary register 5 |
| x31   | t6    | Temporary register 6 |

> The aliases and purposes shown here are determined by the ABI. The ABI (Application Binary Interface) is a set of rules that determine how the components of a software interact at the binary level. These rules affect the conventions for registers, memory layout, and interactions with the operating system. For now, it’s enough to know this much.

Now, let’s look at the general characteristics of these registers:

- Each register has the same memory size, and depending on the architecture, each can have 32, 64, or 128-bit memory.

- Each register has an `xi` style name and an alias describing its purpose. When writing assembly, we can refer to whichever we prefer. I recommend using the alias, as it significantly increases readability.

- Except for the first 5 registers, it’s possible to use the a, s, and t registers interchangeably, but following the ABI guidelines will make the code clearer.

#### Special Registers

| Name  | Purpose |
|-------|---------|
| pc    | Program Counter |
| f0-f31 | Floating-Point Registers |
| csr   | Control and Status Registers (various) |

The `pc` register holds the address of the next instruction to be executed. The `f` registers are used for floating-point operations with the F extension. The `csr` registers are used for more complex system-related operations.

For more detailed information about registers, check out [this link](https://github.com/hesitationIsDefeat/RISC-V-Plaground/blob/main/registers.md).

### RISC-V Assembly Instructions

When writing RISC-V Assembly, we make use of predefined instructions, which are machine commands that the processor knows how to convert into binary form. This way, instead of manually entering 0s and 1s, we can express what we want in a much faster and (in my experience) more human-readable format at the lowest level.

#### Instruction Types

There are 6 types of instructions. These instruction types are classified based on what the instruction performs:

1. **R-Type Instructions (Register-Register):** Used to assign a value to a register using the values from other registers.

2. **I-Type Instructions (Immediate):** Used to assign a fixed value to a register.

3. **U-Type Instructions (Upper Immediate):** Used for operations involving constants that occupy more than 12 bits.

4. **S-Type Instructions (Save):** Used for reading from or writing to memory.

5. **J-Type Instructions (Jump):** Used to execute an instruction at a specified address.

6. **B-Type Instructions (Branch):** Used for conditional jumps (as in the previous type).

### Conversion of Commands to Binary Form

Each of the commands mentioned above needs to be presented to the processor in binary form for the processor to understand it. This binary form has a different structure for each type of command. Let's briefly look at each of them:

| **Command Type**  | **Command Format**                                       | **Binary Format** |
|-------------------|----------------------------------------------------------|-------------------| 
| **R-Type** | `opcode rd, rs1, rs2`             | func7[25:31] rs2[20:24] rs1[15:19] func3[12:14] rd[7:11] opcode[0:6] |
| **I-Type** | `opcode rd, rs1, imm`             | imm[31:20] rs1[15:19] func3[12:14] rd[7:11] opcode[0:6] |
| **U-Type** | `opcode rd, imm`                  | imm[31:12] rd[7:11] opcode[0:6] |
| **S-Type** | `opcode rs2, offset(rs1)`         | imm[25:31] rs2[20:24] rs1[15:19] func3[12:14] imm[7:11] opcode[0:6] |
| **J-Type** | `opcode rd, offset`               | imm[12:19] imm[11] imm[1:10] imm[20] rd[7:11] opcode[0:6] |
| **B-Type** | `opcode rs1, rs2, offset`         | imm[12] imm[10:5] rs2[20:24] rs1[15:19] func3[12:14] imm[4:1] imm[11] opcode[0:6] |

### Writing Commands to Memory

How the binary forms of the commands we wrote are stored in memory depends on the endianness of the architecture we are working with. RISC-V by default supports little-endian format. Technically, little-endian means that the byte with the lowest value is stored at the smallest memory address.

Let's go through an example:

```
lw t0, 4(sp) # 000000000100 00010 010 00101 0000011 binary form (0x00412203 hex form)
```
 
When the command we entered is compiled, it turns into this binary (and thus hex) form. When we want to store this command in memory, we reverse it like this:

```
0x00412203 command hex form -> 0x03224100 command hex form in memory
```

and store it in memory as:

```
0x1000 -> 0x03
0x1001 -> 0x22
0x1002 -> 0x41
0x1003 -> 0x00
```

Thus, the byte order is reversed, but the order of the bits within each byte remains the same. When the command is executed, the processor fetches the bytes starting from the smallest memory address, from left to right. In this case, the first byte fetched is 0x03, which contains the opcode information. This allows the processor to know the type of command that will be executed as it fetches the other bytes. This parallel handling of command fetches helps to reduce execution time.

#### Pseudo Commands

When talking about commands, it's impossible not to mention pseudo commands. Pseudo commands, unlike machine commands, cannot be directly converted into binary form. Instead, they are converted into corresponding machine commands by the assembler during assembly time. Let's make this clearer with an example.

Let's say we want to assign a value greater than 12 bits, such as 100000, to the t0 register. In this case, we can use the pseudo command:

```
li t0, 100000
```

To achieve this with machine commands, we would have to call:

```
lui t0, 0x18    # upper 20 bits
addi t0, t0, 0x6A0   # lower 20 bits
```

Using pseudo commands helps increase our assembly coding speed and also makes our code more readable.

### Next Time:
In my next post, I will cover RISC-V's addressing modes, privilege levels, and security addresses.

Stay healthy!