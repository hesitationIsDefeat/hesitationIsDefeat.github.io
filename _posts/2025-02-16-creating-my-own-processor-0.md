---
title: "Creating My Own Processor #0"   
date: 2025-02-16 20:00:00 +0300
categories: [processor, risc-v]
tags: [assembly, risc-v, low level programming, processor, isa]     
---

# What is RISC-V?

![Future Processor](/assets/img/article/Leonardo_Phoenix_10_A_cinematic_black_and_white_photograph_of_1.jpg)

(Türkçe)

(Not: Bu yazıda mümkün olduğunca Türkçe karşılıklar kullanılmış, anlam bütünlüğünün daha iyi sağlandığı noktalarda İngilizce kavramlara yer verilmiştir. İyi okumalar!)

Herkese selamlar. Uzun ve ilgilendiğim hangi konu hakkında daha da derinlemesine araştırma yapıp bir yazı serisine başlama kararı almamın zor olduğu bir aranın sonuna gelmiş bulunuyoruz.

Bir süredir assembly öğrenerek low code yazma isteğim olmasına karşın yeterli vakti ayırma şansım olmamıştı. Boğaziçi Üniversitesi'nde son dönemimi okurken Bilgisayar Mühendisliği'nden Computer Architecture dersini alarak kendimi zaman ayırmak zorunda bıraktım ve bu sayede yazılımın daha "derinine" doğru bir yolculuğa başladım. Öğrenim yolculuğumu hazırlayacağım yazılar ile dökümante ederek hem eksik kaldığım yönleri daha iyi fark edecek hem de süreçten aldığım keyfi arttıracağım.

Dersin projesinde RISC-V mimarisi ile çalışıyor olacağız. Ben de öğrenim yolculuğumun ilk kısmını RISC-V mimarisini anlamaya ayırdım. İlerleyen süreçlerde donanımın da üzerine düşmeye başlayacağım, özellikle FPGA ve HDL konularına. Nihai amacım ise kendi işlemcimi ve üzerinde koşacak kodu yazmak.

RISC-V'in ne olduğunu anlamak için öncelikle belli kavramlara hakim olmamız gerekiyor. Bu yüzden yazımı 5 bölüme ayırdım. Sırasıyla:

1. Assembly dili nedir?

2. ISA nedir?

3. Open Standard ISA nedir?

4. RISC nedir?

5. RISC-V nedir?


## Assembly dili nedir?
Assembly dili, veya kısaca assembly, yazılım dilleri hiyerarşisinde makine kodunun hemen üzerinde bulunur. Görevi, assembler yardımı ile işlemcinin anlayacağı dilden konuşmak, yani verilen komutları binary formata çevirmektir. Human readable (insan gözüyle bakıldığında bir anlam ifade eden) sayılan en low level kodlama şeklidir. Bir bakıma Python, Java, C, Rust gibi high-level diller (ki bu diller arasında da bir high-low hiyerarşisi bulunur) ile binary format arasında bir köprü görevi görür.

Yani, Python ile yazdığımız bir kod dönüp dolaşıp assembly formatına geri döner. Bu duruma temsili bir örnek olarak

```python
sum = 1 + 2
```

kod satırının (kabaca)

```
addi t0, zero, 1 
addi t1, zero, 2
add a0, t0, t1
```

assembly koduna dönüşmesi verilebilir. Aynı şekilde

```
addi t0, zero, 1
```

assembly komutu

```
00000000000100000000001010010011 (0x00100293 in hex)
```

olacak şekilde bir binary komutuna dönüştürülür ve işlemcinin idrak edebileceği bir hale gelir. Bu sayede insan-bilgisayar iletişimi sağlanır.

## ISA nedir?

Instruction Set Architecture, kısaca ISA, bir işlemci ile düzgün etkileşim kurabilmek için uyulması gereken kuralları ve yöntemleri belirleyen tanımlar bütünüdür. Bir ISA'in temel parçaları şu şekildedir:

- Instruction Set: İşlemcinin uygulamayı bildiği komutlardır

- Registers: İşlemcinin içerisinde bulunan, küçük miktarda veri tutabilen, çok hızlı bir şekilde erişilen depolama alanlarıdır

- Addressing Modes: Verilen komutların hafızaya erişim biçimini belirler

- Memory Model: İşlemcinin hafızaya nasıl eriştiğini ve hafızayı nasıl organize ettiğini belirtir

- Data Types: İşlemcinin çalışabildiği veri tiplerini içerir

- Interrupt & Exception Handling: Dışarıdan (kodu durdurma komutu) veya içeriden (kodda yapılan bir hata) kaynaklanan ve anlık ilgilenme gerektiren olaylara verilen tepkiyi belirler


## Open Standard ISA nedir?

Open Standard ISA, isminden de anlaşılacağı üzere açık kaynaktır. Bu sayede herhangi bir ücret ödemeden ve bir lisansa bağlı kalmadan kendi işlemcinizi üretmenizi sağlar. Ayrıca Open Standard ISA'ler çoğunlukla modüler yapıdadır ve kurallar setini ihtiyacımıza göre düzenlememizi sağlar. Bu özelliği önümüzdeki yazımda detaylı bir şekilde aktaracağım.
Bunların yanında, Open Standard ISA'ların en önemli avantajlarından biri de oluşan binary komutun işlemci tarafından nasıl anlaşıldığının açık bir şekilde dökümente edilmesidir. Yukarıdaki örnek üzerinden gidecek olursak:

```
00000000000100000000001010010011
```

binary komutu işlemci tarafından

```
000000000001 00000 000 00101 0010011
```

olacak şekilde bölünür ve komut bu değerlere göre uygulanır. İlk yazıdan görece daha karmaşık kısımlara girmek istemediğim için bu değerlerin neye tekabül ettiğini ilerleyen yazılarımda açıklayacağım. Şimdilik, binary komutun nasıl işlendiğini biliyor olmamızın ileride kendi işlemcimizi oluşturmamız için çok kıymetli bir durum olduğunu bilmemiz yeterli.

## RISC nedir?

Reduced Instruction Set Computing, kısaca RISC, bir ISA oluşturma ilkeleri bütünü olarak düşünülebilir. RISC'in amacı az miktarda ve efektif komutlar kullanarak hızlı performans elde etmektir. Temel özelliklerini sıralamak gerekirse:

- Bir ya da birkaç clock cycle süren basit komutlar içerir

- Komutların çoğu sabit bir büyüklüğe sahip olduğu için işlemcinin komutları daha etkili bir biçimde anlamasını sağlar

- Diğer mimarilere kıyasla daha fazla general purpose register'a sahip olması hafızaya erişme zorunluluğunu azaltır ve performansı iyileştirir

- Sahip olduğu basit komutlar sebebiyle diğer mimarilere kıyasla pipeline'dan (komutların paralelde işlenmesi) daha çok faydalanır

- Addressing mode sayısı diğer mimarilere kıyasla daha az olduğu için benzer şekilde pipeline'dan daha çok faydalanır

## RISC-V nedir?

RISC-V, RISC ilkeleri üzerine kurulu bir Open Standard ISA'dir. Sahip olduğu assembly dili (RISC-V Assembly) sayesinde makine dilinde kod üretmemizi sağlar.

Fevkalade! Artık RISC-V'in ne olduğu konusunda temel bir fikre sahibiz.

## Gelecek Sefer:
Gelecek yazımda RISC-V'in modüler yapısından, register'lardan ve temel komutlardan bahsedeceğim.

Sağlıcakla kalın!

## 

![Future Processor](/assets/img/article/Leonardo_Phoenix_10_A_cinematic_black_and_white_photograph_of_1.jpg)

(English)

(Note: The following translation was generated with some help due to the time limitations, not the lack of skills on my end. Enjoy the reading!)

Hello everyone. We have reached the end of a long break in which it was difficult for me to decide on a topic I wanted to research in depth and start a series of articles.

For some time, I have wanted to learn assembly and write low-code, but I have not had the opportunity to dedicate enough time to it. While studying my final semester at Boğaziçi University, I took the Computer Architecture course in the Computer Engineering department, which forced me to allocate time, thus beginning a journey into the “deeper” aspects of software. By documenting my learning journey through the articles I will prepare, I will both better identify my shortcomings and increase the enjoyment I get from the process.

For the course project, we will be working with the RISC-V architecture. I have dedicated the first part of my learning journey to understanding the RISC-V architecture. In the later stages, I will start focusing on hardware as well, particularly on FPGA and HDL topics. My ultimate goal is to design my own processor and the code that will run on it.

To understand what RISC-V is, we first need to grasp certain concepts. For this reason, I have divided my article into five sections, as follows:

1. What is assembly language?

2. What is ISA?

3. What is an open standard ISA?

4. What is RISC?

5. What is RISC-V?


## What is assembly?
Assembly language, or simply assembly, is located just above machine code in the hierarchy of programming languages. Its purpose is to communicate in a language that the processor can understand with the help of an assembler, that is, to translate given commands into binary format. It is considered the lowest level form of coding that is human-readable. In a way, it acts as a bridge between high-level languages like Python, Java, C, and Rust (which also have their own high-low hierarchy) and binary format.

In other words, code written in Python eventually translates back into assembly format. As a representative example:

The code line
```python
sum = 1 + 2
```

roughly translates into the following assembly code:

```
addi t0, zero, 1 
addi t1, zero, 2
add a0, t0, t1
```

Similarly, the assembly command

```
addi t0, zero, 1
```

is converted into a binary instruction:

```
00000000000100000000001010010011 (0x00100293 in hex)
```

making it understandable by the processor. This is how human-computer communication is established.

## What is ISA?

Instruction Set Architecture, or ISA for short, is a set of definitions that establish the rules and methods necessary for proper interaction with a processor. The fundamental components of an ISA are as follows:

- Instruction Set: The commands the processor knows how to execute

- Registers: Small storage locations inside the processor that can hold a small amount of data and be accessed very quickly

- Addressing Modes: Determine how given commands access memory

- Memory Model: Specifies how the processor accesses and organizes memory

- Data Types: Includes the data types the processor can handle

- Interrupt & Exception Handling: Defines the processor’s response to events that require immediate attention, whether originating externally (a command to stop the code) or internally (an error in the code)


## What is an open standard ISA?

As the name suggests, an open standard ISA is open-source. This allows you to create your own processor without paying any fees or being tied to a license. Additionally, open standard ISAs are usually modular, enabling you to customize the set of rules according to your needs. I will explain this feature in detail in my next article.

One of the most important advantages of open standard ISAs is that the way the binary instruction is interpreted by the processor is clearly documented. For example, taking the previous binary instruction:

```
00000000000100000000001010010011
```

the processor breaks it down as follows:

```
000000000001 00000 000 00101 0010011
```

and executes the command based on these values. Since I do not want to delve into more complex topics in the first article, I will explain what these values correspond to in my future articles. For now, it is enough to understand that knowing how a binary instruction is processed is crucial for creating our own processor in the future.

## What is RISC?

Reduced Instruction Set Computing, or RISC for short, can be considered a set of principles for designing an ISA. The goal of RISC is to achieve high performance by using a small number of efficient commands. Its key features include:

- Consists of simple instructions that take one or a few clock cycles

- Since most instructions have a fixed size, the processor can interpret them more efficiently

- Has more general-purpose registers compared to other architectures, reducing the need to access memory and improving performance

- Due to its simple instructions, it benefits more from pipelining (processing commands in parallel) compared to other architectures

- Since it has fewer addressing modes compared to other architectures, it similarly benefits more from pipelining

## What is RISC-V?

RISC-V is an open standard ISA based on RISC principles. Thanks to its assembly language (RISC-V Assembly), it enables us to produce code in machine language.

Excellent! Now we have a basic idea of what RISC-V is.

## Next time:
In my next article, I will talk about the modular structure of RISC-V, registers, and basic instructions.

Stay safe!
