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

  2.2) Rails Testi İçin Yapılması Gereken Ayarlar
---
Rails aynı zamanda bir test klasörü oluşturur. Bu test klasörünün içeriğini listelemek ve görmek istiyorsak komut satırına;
	$ ls -F test/
	
	fixtures/  functional/  integration/  performance/  test_helper.rb  unit/

Buradaki unit klasöründe; oluşturulan modellerin testleri tutulur.
Functional klasörü; kontrol ve denetleme testlerini tutar.
İntegration klasörü; controller ile etkileşimli testler burada tutulur.Bunların birlikte çalışmasını sağlar.
Fixtures; test verilerinin organize bir şekilde tutulmasını sağlar.
test_helper.rb dosyasında ise testler için varsayılan yapılandırmalar bulunur.

  2.3) Fixtures
---
İyi testler oluşturmak için , Rails 'de fixtureler özelleştirilebilir ve tanımlayarak işlenerek oluşturulabilir.

  2.3.1) Fixtures nedir?
---
Fixtures, örnek veriler için kullanılır. Fixtures, testlerimizi çalıştırmadan önce, önceden tanımlanmış olan veriler ile test veritabanının doldurulmasına izin verir. Fixtures bağımsız veritabanı ve bunda kullanılan varsayılan format YAML 'dır.
Fixtures, test/fixtures dizininin altında bulunur.Yeni Rails modelleri oluşturmak için çalıştırdığımızda, fixtures taslaklarıda otomatik olarak oluşturulmuş olur ve bu dizine yerleştirilir.

  2.3.2) YAML
---
YAML formatı örnek verileri tanımlamak için kullanılır. Bu tür fixtures 'ler .yml dosya uzantısına sahiptirler. Örneğin; 'users.yml'
YAML fixture dosyasına bir örnek verecek olursak;
	# lo & behold! I am a YAML comment!
	david:
 	 name: David Heinemeier Hansson
 	 birthday: 1979-10-15
 	 profession: Systems development
 
	steve:
 	 name: Steve Ross Kellock
 	 birthday: 1974-09-27
	 profession: guy with keyboard

Kayıtlar birer boşluk ile ayrılır. İlk satırda # karakteri kullanılarak fixture dosyasına yorum eklenebilir.

  2.3.3) ERB
---
ERB, şablonların içine Ruby kodu gömmek için kullanılır. Fixtures 'ler yüklendiği zaman, YAML fixture formatı ERB ile önişlemden geçer. Bu bize Ruby kullanarak bazı örnek veriler oluşturmamıza yardımcı olur. Örneğin;
	
	<% earth_size = 20 %>
	mercury:
	  size: <%= earth_size / 50 %>
	  brightest_on: <%= 113.days.ago.to_s(:db) %>
 
	venus:
	  size: <%= earth_size / 2 %>
	  brightest_on: <%= 67.days.ago.to_s(:db) %>
 
	mars:
	  size: <%= earth_size - 69 %>
	  brightest_on: <%= 13.days.from_now.to_s(:db) %>

<% %> bu taglar içindekiler Ruby kodları olarak kabul edilir. Bu fixture yüklendiği zaman bu 3 kaydın boyut özelliği sırasıyla 20/50, 20/2, 20-69 olarak ayarlanmış olur. brightes_on özelliğide değerlendirilir ve veritabanı ile uyumlu olması için Rails tarafından biçimlendirilir.

  2.3.4) Fixtures Eylemleri
---
Bizim unit ve functional klasörlerimiz için Rails varsayılan tüm test/fixtures dosyalarını otomatik olarak yükler. Yükleme işlemi 3 adımda oluşur;
   
   - Fixture karşılık tabloda tekabül eden mevcut veriyi kaldırır.
   - Tabloya fixture verilerini yükler.
   - Bir duruma doğrudan erişmek istediğimizde fixture veri dökümü yapar.

  2.3.5) Özel Yetkili Hash 'ler
---
Fixture 'nin temeli Hash nesneleridir. Fixture test durumunda yerel bir değişken olarak otomatik kurulum yaptığı için hash nesnelerine doğrudan erişim sağlayabiliriz.

	# Bu David isimli fixture için Hash döndürecektir.
	users(:david)
	# Bu ise David 'in id sini döndürür.
	users(:david).id

Fixture aynı zamanda orijinal sınıf formunda olmasını sağlar. Böylece yalnızca o sınıf için mevcut yöntemler kullanma imkanımız olur.

	# find metodu kullanarak gerçek kullanıcılara ulaşabiliriz.
	david = users(:david).find
	# şimdi ki erişim yöntemi ise yalnızca bir kullanıcı sınıfı için geçerlidir.
	email(david.girlfriend.email, david.location-tonight)

  3) Unit Test Modelleri
---
Rails 'de Unit testleri, bizim modelleri test edebilmemiz için yazılmaktadır. Bu kılavuz için Rails scaffolding kullanılmaktadır. Bu bize yerdeğiştirme, denetleme ve tek bir işlem ile yeni bir kaynak görünümü sağlamak için model oluşturmaktadır.
Diğer şeylerin yanı sıra bir kaynak için Rails scaffold kullandığımızda bu test/unit klasörü için bir test saptaması oluşturur:

	$ rails generate scaffold post title:string body:text
	...
	create  app/models/post.rb
	create  test/unit/post_test.rb
	create  test/fixtures/posts.yml
	...

Bu satırı yazdığımızda projemizde test dosyaları oluşmaktadır. test/unit/post_test.rb 'nin içeriği şöyle olur;

	require 'test_helper'
 
	class PostTest < ActiveSupport>::TestCase
  	  # Replace this with your real tests.
  	  test "the truth" do
            assert true
          end
        end

Satır incelemesi yapmak, Rails testinin kod ve terminolojisini anlamada bize yardımcı olur.
	
	require 'test_helper'

Bilindiği gibi, 'test_helper.rb' dosyası bizim testlerimizi çalıştırabilmemiz için gerekli yapılandırmaları içerir. Böylece bu dosyaya eklenen herhangi bir yöntem, içerdiği testler de dahil tüm testler için kullanılabilir.

	class PostTest < ActiveSupport>::TestCase 

PostTest sınıfı burada bir TestCase tanımlar çünkü; bu TestCase ActiveSupport 'tan miras alır(devralır).
Test:Unit test durumunda tanımlanmış, 'test'(küçük harf duyarlı) ile başlayan herhangi yöntem bir testtir. Bundan dolayı 'test_password', 'test_valid_password' ve 'testValidPassword' gibi tüm test isimleri 'TestCase' çalıştığında otomatik olarak çalışır. 
Rails, test metoduna bir test ismi ve bloğu ekler. Burada eklenen 'test_' öneki normal bir Test:Unit durumu oluşturur. Yani;

	test "the truth" do
  	  assert true
	end

Eğer böyle yazılmış olsaydı;

	def test_the_truth
          assert true
        end

sadece daha okunabilir bir test adı vermiş olurdu.Ayrıca method isimleri altı çizili boşluklar ile vurgulanır.

	assert true

Bu işlem, satır onaylama işlemi olarak adlandırılır. Onaylama, beklenen sonuçlar için bir nesneyi (veya ifadeyi) değerlendiren bir kod satırıdır. Örneğin bir onaylama işleminin kontrolü:

   - bu değer = o değer mi?
   - nesne sıfır mı?
   - bu kod satırı kural dışı mı?
   - kullanıcının parolası 5 karakterden daha mı büyük? 

Her test bir veya birden fazla onaylama içerir. Test tüm onaylardan başarılı olduğu zaman geçerli ifade sayılabilecektir.

  3.1) Test için Uygulama Hazırlama
---
Testlerimizin çalışması için önce, mevcut bir test veritabanı bulunup bulunmadığını kontrol etmeliyiz. Bunun için aşağıdaki komutlar kullanılır:

	$ rake db:migrate
     
Burada ilk migrate döner, bir sonraki geçerli değildir. Yani bizim ürün tablomuz mahvolabilir.Bu yüzden ilk db:test:prepare çalıştırmak mantıklıdır.

	...
	$ rake db:test:load   #Bu satır ile test veritabanı yeniden oluşturulur.

Sonraki girişimler üzerinde; ilk olarak db:test:prepare çalıştırmak bekleyen migrate kontrolleri yapılması ve bizi uyarması için iyi bir fikirdir.

Not: Eğer db/shema.rb yoksa db:test:prepare eror vererek bize hata durumunu bildirir.

  3.1.1) Test Uygulamalarının Hazırlanmasında Rake 'in Görevi
---

   - rake db:test:clone ; Geçerli olan veritabanı şemasında bir test veritabanı oluşturmak için
   - rake db:test:clone_structure ; Test veritabanının gelişme yapısının yeniden oluşturulması için
   - rake db:test:load ; shema.rb 'den gelen test veritanını oluşturmak için
   - rake db:test:prepare ; Bekleyen migrate ve test şemasının kontrolü için
   - rake db:test:purge ; veri tabanını boşaltmak için kullanılır.

'rake --tasks-- describe' satırını çalıştırarak tüm rake 'lere ve bunların görevlerine açıklamaları ile birlikte ulaşabiliriz.

  3.2) Testlerin Çalıştırılması
---
Bir testi çalıştırmak için basit bir şekilde Ruby test durumunda ki bir dosya çağırılır:

	$ ruby -Itest test/unit/post_test.rb

	Loaded suite unit/post_test
	Started
	.
	Finished in 0.023513 seconds.
	 
	1 tests, 1 assertions, 0 failures, 0 errors

Bu test durumunda tüm test yöntemleri çalışacaktır. Bu test_helper.rb bir test dizinidir, dolayısıyla -I anahtarı kullanarak yükleme yolunun eklenmesi gerekir.
Ayrıca test yönteminin adı ile -n anahtarı kullanarak test durumunda belirli bir test yöntemini çalıştırabiliriz. Örneğin;

	$ ruby -Itest test/unit/post_test.rb -n test_the_truth
 
	Loaded suite unit/post_test
	Started
	.
	Finished in 0.023513 seconds.
 
	1 tests, 1 assertions, 0 failures, 0 errors

.(nokta) yukarıda geçen testi belirtmektedir. Bir test başarısız olduğu zaman 'F', hata verdiği zaman 'E' harfi görünür. Çıktının son satırı ise bir özettir.
Bir test hatasının bildirisini sınamak için; post_test.rb 'ye başarısız bir durum ekleyebiliriz.Örneğin;

	test "should not save post without title" do
 	  post = Post.new
	  assert !post.save
	end

Bu yeni eklenen test çalıştırıldığında;

	$ ruby unit/post_test.rb -n test_should_not_save_post_without_title
	Loaded suite -e
	Started	
	F
	Finished in 0.102072 seconds.
 
	  1) Failure:
	test_should_not_save_post_without_title(PostTest) [/test/unit/post_test.rb:6]:
	<false> is not true.
	 
	1 tests, 1 assertions, 1 failures, 0 errors

Çıktıdaki 'F' harfi bize başarısız olduğunu belirtir. Altında ise başarısız test adı ile birlikte bu uyarıyı görebiliriz. Varsayılan onaylama mesajları da hatayı belirlemede ve düzeltmede yardımcı olmak için yeterli bilgi sağlar.
Onaylama işlemi hatası mesajını daha okunabilir hale getirmek için, her onaylama aşağıda gösterildiği gibi isteğe bağlı mesaj parametresi içerebilir:
 
	test "should not save post without title" do
	  post = Post.new
	  assert !post.save, "Saved the post without a title"
	end

Bu testi çalıştırdığımızda aşağıdaki onaylama mesajını göstermektedir: 

	1) Failure:
	test_should_not_save_post_without_title(PostTest) [/test/unit/post_test.rb:6]:
	Saved the post without a title.
	<false> is not true.

Şimdi bu testteki 'failure' ı yok etmek için başlık alanına bir model düzeyi doğrulma eklenebilir;

	class Post < ActiveRecord::Base
	  validates :title, :presence => true
	end

Şimdi test geçerli olur. Tekrar çalıştırılınca şöyle bir sonuç alırız;

	$ ruby unit/post_test.rb -n test_should_not_save_post_without_title
	Loaded suite unit/post_test
	Started
	.
	Finished in 0.193608 seconds.
	 
	1 tests, 1 assertions, 0 failures, 0 errors

Şimdi farkedileceği gibi önce istenen işlev için başarısız bir test yazıldı, sonra işleve bazı kodlar eklenerek testin başarılı olması sağlandı. Yazılım geliştirmede bu yaklaşım Test Odaklı Geliştirme (TDD) dir.

* Bir çok Rails geliştiricisi bu uygulamayı kullanır (TDD). Bu egzersiz, uygulamamızın her parçası için bir test paketi oluşturmada mükemmel bir yoldur. TDD hakkında ayrıntılı bilgiye <a href="http://andrzejonsoftware.blogspot.com/2007/05/15-tdd-steps-to-create-rails.html">buradan</a> ulaşılabilir. 

Bir hata raporunun nasıl olduğunu görmek istiyorsak;

	test "should report error" do
	  # some_undefined_variable is not defined elsewhere in the test case
	  some_undefined_variable
	  assert true
	end

Şimdi bu test için aşağıdaki satır sonucunda şöyle bir çıktı alırız;

	$ ruby unit/post_test.rb -n test_should_report_error

	Loaded suite -e
	Started
	E
	Finished in 0.082603 seconds.
 
	  1) Error:
	test_should_report_error(PostTest):
	NameError: undefined local variable or method `some_undefined_variable' for #<PostTest:0x249d354>
	    /test/unit/post_test.rb:6:in `test_should_report_error'
	 
	1 tests, 0 assertions, 0 failures, 1 errors

Çıktıdaki 'E' ye dikkat edecek olursak bu bir hata göstergesidir.

Not: Her test metodu uygulamasında, bir hata veya onaylama işlemi hatası aldığında en kısa sürede durur ve test paketi sonraki yöntem ile devam eder. Bütün test yöntemleri alfabetik sıraya göre yürütülür.
	
##Yararlanılan Kaynak

	<a href="http://guides.rubyonrails.org/index.html">RubyonRails</a>
	
 
