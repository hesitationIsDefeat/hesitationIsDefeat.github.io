---
title: "Concurrency #0"
description: "Basics"
image: "/assets/img/article/concurrency.jpg"
date: 2025-08-15 20:00:00 +0300
categories: [concurrency, multiprocessing]
tags: [concurrency, atomic, mutex, semaphore, spinlock]     
---
[For English press 9](#basics)

## Temel Bilgiler
***

Herkese selamlar. Bu yazıda concurrency kavramına giriş yapacağız. Her zaman olduğu gibi, bir bilgi inşası yapabilmek için önce temelimizi atacağız. Bu yazıda bu temeli oturtacağız ve inşa için kullanacağımız yapı taşlarını kısaca inceleyeceğiz. Dilerseniz başlayalım.

### Concurrency Nedir?

Bir restoran düşünelim. Bu restoranda çalışan tek bir garson olsun. Bu garson, sipariş vermek isteyen her müşteriye kısa bir süre ayırıp hızlı bir şekilde müşteriler arasında git gel yaparsa, dışarıdan bakıldığında müşterilerin siparişleri aynı anda alınıyormuş gibi gözükür. Bu duruma concurrency diyoruz. Kelimenin tamını yapacak olursak; bir işi, birbirinden bağımsız şekilde halledilebilecek parçalara bölmek diyebiliriz. Concurrency, bir projenin tasarımında kullanılan yöntemi ifade eder; yani 

### Thread Nedir?

Thread, işlemcinin kullandığı en küçük işlem birimidir. 

### Race Condition Nedir?

Race condition, threadlerin aynı anda aynı kaynağa bağlı olarak işlem yapmalarından ve sonucun bu işlemlerin sırasına göre değişiklik göstermesinden kaynaklanan bir problemdir. Threadler, düzgün bir sıraya sokulmadıkları için birbirleri ile yarışır konumdadırlar ve bu durum, threadlerin arka arkaya yapmaları gereken işlerin arasına başka threadlerin işleri girdiği için birden fazla şekilde sonuçlanabilir.

### Critical Section Nedir?

Critical section, bir threadin yapacağı ve o işi yaparken kullandığı kaynaklara o an için sadece o threadin erişmesi gereken kısıları ifade eder. Daha iyi anlamak için bir okuma/yazma (input/output) durumunu inceleyelim. Bir müşteri restarona girer ve garsonlardan (threadlerden) birine ana yemek sipariş ettiğini söyler. Garson müşterinin siparişini bir yere kaydeder. Ilerleyen zamanda siparişine ekleme yapmak isteyen müşteri, Garson 1'e "Siparişime yan yemek istiyorum", Garson 2'ye "Siparişime içecek eklemek istiyorum" der. Bu noktada; eğer garsonların siparişi güncellemesi düzgün ayarlanmamışsa, şu durum yaşanabilir:

1. Garson 1 siparişi okur, siparişin sadece "ana yemek" olduğunu görür
2. Garson 2 siparişi okur, siparişin sadece "ana yemek" olduğunu görür
3. Garson 1 siparişi "ana yemek + yan yemek" olarak günceller
4. Garson 2 siparişi "ana yemek + içecek" olarak günceller

Bu durum race conditiona bir örnektir. Müşterinin beklediği siparişin içerisinde yan yemek de vardır ancak  güncellemelerin "concurrent" bir şekilde yapılmamasından dolayı gelen sipariş eksiktir.

### Multiprocessing Nedir?

Multiprocessing, işlem yaptığımız bilgisayarın birden fazla çekirdeğe sahip olması sayesinde aynı anda birden fazla işi yapabilme kabiliyetidir. Restoran örneğini düşündüğümüzde, restoranın birden fazla garsonun aynı anda çalışması için gerekli bütün alet edevata sahip olması multiprocessingdir.

### Paralellism Nedir?

Paralellism, aynı anda birden fazla işlem yapılmaya uygun bir bilgisayarın birden fazla işi aynı anda yapmasıdır. Yeterli ekipmana sahip bir restoranda birden fazla garsonun aynı anda çalışması olarak düşünebiliriz. 

### Yöntemler

Threadlerimizin concurrent bir şekilde çalışmalarını sağlamak için kullanabileceğimiz yöntemlere bakalım. 

#### Atomik Operasyonlar

Atomik kelimesi bizim bağlamımızda "bölünemez" anlamına geliyor. Bir operasyon atomik ise bu operasyonun bütün adımları bölünmeyen bir bütün olarak, arka arkaya tamamlanıyor. Bu sayede tek bir kaynak üzerinden işlem yapan birden fazla thread, yaptıkları işlemleri atomik olarak gerçekleştirdekleri için düzgün bir sıra halinde, birbirlerinin işlemlerinin arasına kendi işlemlerini sıkıştıramadıkları için concurrent bir şekilde çalışabiliyorlar.

Atomik operasyonlar; CPU tarafından desteklenen atomik talimatlar, talimatların yeniden sıralanmasının (instruction reordering) belli noktalarda engellenmesi, önbellek (cache) değerlerinin her değişimde güncellenmesi gibi donanıma yakın seviyelerde sağlanan özellikler sayesinde kullanılabilmektedir.

#### MUTual EXclusion

#### Semaphore

#### Spinlock

### Gelecek Sefer

Bu yazıda temel kavramlar hakkında bilgi sahibi olduk. Bir sonraki yazıda vermiş olduğum restoran örneğini kod üzerinden inceleyeceğiz. Sağlıcakla ve Rock'n'Roll'la kalın 🤘

## Basics
***
