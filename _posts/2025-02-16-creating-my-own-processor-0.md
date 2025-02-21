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