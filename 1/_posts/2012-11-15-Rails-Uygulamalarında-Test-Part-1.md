---
layout: post
title:  Rails Uygulamalarında Test - PART 1
---

  1) Rails Uygulamalarında Neden Test Yazılır?
---
Maddeler halinde yazacak olursak;

   - Rails uygulamalarında test işimizi kolaylaştırır.Test kısmı modelleme ve kontrolürler oluşturulurken arka planda iskelet kodu üretir.
   - Yazılan bazı büyük kodlarda işlevselliği yakalamak için Rails testleri çalıştırılır.
   - Rails testleri ile tarayıcı isteklerinin benzetimini yapabiliriz. Böylece tarayıcı aracılığıyla test etmek zorunda kalmadan kodlarımızın tepkisini test edebiliriz. 

  2) Teste Giriş
---
  Test kısmı kodlamanın başlangıcından itibaren Rails kodları içine yerleştirilmiştir.Hemen hemen her Rails uygulaması bir veri tabanı ile 
etkileşimlidir.Bunun için Rails testleri içinde bir veritabanına ihtiyacımız vardır.Test kısımlarının doğru çalışması için,verimli Rails 
testleri yazabilmemiz için bir veritabanı hazırlamalı ve örnek verilerle doldurmamız gerekir.

  2.1) 3 Bölümden Oluşan Rails Uygulamaları
---
Oluşturacağımız her Rails uygulamasının 3 ayağı vardır.

   - Üretim
   - Gelişme
   - Test

Bu ayrım config/database.yml dosyasında bulunmaktadır.Bu YAML yapılandırma dosyası; 3 farklı veritabanı kurulumlarını tanımlayan 3 farklı 
bölümden oluşur.Bunu kurarak, testin üretim aşamasında, bu ortamda veri değiştirme tehlikesi olmadan test verileri ile etkileşimi sağlar.
Eğer test kısmının sonuna geldiğimizde bazı olabilecek sebeplerden ötürü veritabanı yoksa veritabanı içinde tanımlanan özelliklere göre onu 
yeniden oluşturabiliriz.Ve bunu da db:test:prepare diyerek yapabiliriz.




  
