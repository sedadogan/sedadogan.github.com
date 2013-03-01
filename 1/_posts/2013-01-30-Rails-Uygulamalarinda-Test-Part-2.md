---
layout: post
title:  Rails Uygulamalarinda Test - PART 2
---

  4) Controller 'lar için Functional Testler
---
Rails 'de functional testler, tek bir controller 'ın çeşitli eylemlerini yazılı olarak çağırır. Controller 'lar bizim uygulamamıza gelen web isteklerini tutar/işler ve sonunda görüntülenmesini sağlar.

  4.1) Functional Testler Neyi Kapsar?
---
Neleri test ederiz? Bunlar:

   - Web isteği başarılı oldu mu?
   - Kullanıcı doğru sayfaya yönlendirildi mi?
   - Kullanıcı başarıyla doğrulandı mı?
   - Yanıtlama modelinde/şablonunda bulunan, depolanan nesne doğrulandı mı?
   - Kullanıcıya görüntülenen mesaj uygun mesaj mı?

Şimdi biz post resource (post kaynakları) için **Rails scaffold generator** kullanığımız için bu zaten otomatik olarak controller kodlarını ve functional testleri oluşturur. 

test/functional dizinindeki post_controller_test.rb dosyasına bir göz atalım.

	test "should get index" do
	  get :index
	  assert_response :success
	  assert_not_nil assigns(:posts)
	end

test_should_get_index testinde, Rails simülatöründe bir istek ile index çağırılır. İstek başarılı olmuş ise ve bu temin edilmiş ise geçerli olan posts örneği değişkene atılır.

Get metodu ile web isteği başlatılır ve sonuçlar cevapların içeriğini oluşturur. Bu 4 bağımsız değişken tarafından kabul edilir:

   - Eylem/işlem, bizim talep ettiğimiz controller lardır. Bu bir string veya sembol biçiminde olabilir.
   - Request (istek) parametreleri, isteğe bağlı olarak çırpılarda harekete geçer. (Örn. sorgu dizesi parametreleri veya post değişkenleri)
   - Session (oturum) parametreleri, isteğe bağlı çırpılarda gelen talep/istek üzerine harekete geçer.
   - Flash değişkenleride isteğe bağlıdır.

Örneğin; :show eyleminde, id parametresi 12 olsun ve session user_id 'si 5 olarak ayarlansın. Yani;

	get(:show, {'id' => "12"}, {'user_id' => 5})

Bir diğer örnek de; :view eyleminde bu sefer session yok, id parametresi 12, ayrıca bir flash mesajı ile birlikte çağırılıyor.

	get(:view, {'id' => '12'}, nil, {'message' => 'booya!'})

**NOT:** Eğer biz posts_controller_test.rb ile gelen test_should_create_ post 'u çalıştırmaya çalışırsak, yeni eklenen modelin düzeyi/seviyesi nedeniyle doğal olarak başarısız bir sonuç elde ederiz. Bu yüzden bizim testin başarılı olması için post_controller_test.rb içinde bazı değişiklikler yapmalıyız.
Yapılan değişiklikler ile son hali:

	test "should create post" do
	  assert_difference('Post.count') do
	    post :create, :post => { :title => 'Some title'}
	  end
	 
	  assert_redirected_to post_path(assigns(:post))
	end

Şimdi tüm testler başarılı bir şekilde çalışmalıdır.

  4.2) Functional Testler için Mevcut İstek Türleri
---
HTTP protokolüne aşina olanlar, _get_ 'in bir tür istek olduğunu bilir.
Rails de functional testler tarafından desteklenen 5 istek tipi vardır:

   - get
   - post
   - put
   - head
   - delete

İstek tiplerinin tümü kullanabileceğimiz yöntemlerdir, ancak ilk iki istek diğerlerine göre daha sık kullanılır.

Functional testlerde, belirtilen istek tipinin eylem tarafından kabul edilip edilemeyeceğine dair bir kontrol yoktur. Bu bağlamda istek/talep türleri testleri daha açıklayıcı hale getirmek için vardır.

  4.3) Hash (Çırpı) Türleri
---
Bir istek, 5 yöntemden(get, post ...) biri kullanılarak ve işlenerek yapılır, kullanıma hazır 4 hash nesnesi bulunur.

   - **assigns**, views(görünümler) i kullanmak için eylemler örnek değişkenler olarak depolanır.
   - **cookies**, herhangi bir cookies ayarlar/tespit eder.
   - **flash**  , flash 'da bulunan herhangi bir nesne.
   - **session**, session(oturum) değişkenlerinde bulunan herhangi bir nesne.

Normal hash nesnelerinde olduğu gibi dize anahtarlarına başvurarak değişkenlere erişilebilir. Ayrıca _assigns_ haricinde, sembol ismi ile de ulaşılabilir. Örneğin;

	flash["gordon"]               flash[:gordon]
	session["shmession"]          session[:shmession]
	cookies["are_good_for_u"]     cookies[:are_good_for_u]
 
	# Because you can't use assigns[:something] for historical reasons:
	assigns["something"]          assigns(:something)

  4.4) Mevcut Örnek Değişkenler
---
Ayrıca functional testlere 3 örnek değişken ile erişebiliriz:

   - @controller, Controllerlar
   - @request   , İstek
   - @response  , Cevap

  4.5) Bir Fuller Functional Test Örneği
---
Aşağıda flash, assert_redirected_to ve assert_difference kullanılan bir örnek bulunmaktadır:

	test "should create post" do
	  assert_difference('Post.count') do
	    post :create, :post => { :title => 'Hi', :body => 'This is my first post.'}
	  end
	  assert_redirected_to post_path(assigns(:post))
	  assert_equal 'Post was successfully created.', flash[:notice]
	end

  4.6) Testing Views(Test Görüntüleme)
---
Test de bizim isteğimize cevap olarak gelen, uygulamalarımızdaki views 'lerimizi tasti için HTML öğeleri ve bunların içeriğinin ????
assert_select ifadesi basit ama güçlü bir sözdizimi kullanarak bize bunu sağlar.

**NOT:** Diğer dökümanlarda assert_ tag ifadesi kullanılabilir, artık assert_select ifadesi önerilmemektedir.

assert_select ifadesinin iki kullanım biçimi vardır:

   - assert_select(selector, [equality], [message]) ifadesi equality(eşitlik) koşuluyla ve selector yoluyla seçilen öğelerin bir araya getirilmesini sağlar. Selector, bir CSS selector(seçici) ifadesi (dize) olabilir, yerine geçen değerlerle veya bir HTML::Selector nesnesiyle ifade eilebilir.

   - assert_select(element, selector, [equality], [message]) ifadesi equality(eşitlik) koşulu ile elemanı (HTML::Node) ve bu gibi eleman formuyla başlayarak selector yoluyla seçilen tüm unsurların bir araya getirilmesini sağlar. 

Örneğin, bizim cevabımız ile başlık öğesinin içeriğini doğrulamak olabilir;
   
	assert_select 'title', "Welcome to Rails Testing Guide"

Ayrıca içiçe assert_select blokları kullanılabilir. Bu durumda iç assert _select, dış assert _select bloğu tarafından seçilmiş elemanlar arasında tam bir derleme şeklinle çalışır.

	assert_select 'ul.navigation' do
	  assert_select 'li.menu_item'
	end

Alternatif olarak, dış assert_select tarafından seçilen elemaların derlenmesi bize assert _select 'in her bir eleman için ayrı ayrı olarak, tekrarlanarak adlandırılmasını sağlar.
Örneğin varsayalım, cevabı iki sıralı liste içersin, her 4 liste elemanları ile ardından aşağıdaki testlerin her ikisi geçer.

	assert_select "ol" do |elements|
	  elements.each do |element|
	    assert_select element, "li", 4
	  end
	end
 
	assert_select "ol" do
	  assert_select "li", 8
	end

assert_select ifadesi oldukça güçlüdür. Daha gelişmiş kullanımlara <a href="http://api.rubyonrails.org/classes/ActionDispatch/Assertions/SelectorAssertions.html">buradan</a> ulaşabilirsiniz.

  4.6.1) View Tabanlı İfadelere Ek
---
<table>
	<tr>
		<th>Assertion                                                                        </th>
		<th>Purpose</th>
	</tr>
	<tr>
		<td><tt>assert_select_email</tt>                                                              </td>
		<td>Bir e-posta gövdesindeki ifadeleri yapmamızı sağlar </td>
	</tr>
	<tr>
		<td><tt>assert_select_encoded</tt>                                                            </td>
		<td>Kodlanmış HTML ifadelerinin oluşturulmasını sağlar.</td>
	</tr>
	<tr>
		<td><tt>css_select(selector)</tt>  or <tt>css_select(element, selector)</tt>                         </td>
		<td><em>Selector</em> tarafından seçilen tüm unsurları içeren bir dizi döndürür. İkinci değişkende ise ilk taban <em>element</em> ile eşleşir ve kendi çocuklarını herhangi bir <em>selector</em> ifadesiyle eşleştirmeye çalışır. Eğer herhangi bir eşleşme yoksa her iki değişkende boş bir dizi döndürür.</td>
	</tr>
</table>

assert_select_email kullanımına bir örnek verecek olursak;

	assert_select_email do
	  assert_select 'small', 'Please click the "Unsubscribe" link if you want to opt-out.'
	end

  5) İntegration Testi
---
İntegration testleri, herhangi bir sayı ile controller arasındaki etkileşimi test etmek için kullanılır. Genellikle uygulamamız içindeki önemli iş akışlarını test etmek için kullanılır.
Unit ve Functional testlerinin aksine, integration testleri bizim uygulamamızda açıkca 'test/integration' klasörü altında oluşturulması gerekir. Rails, integration testleri oluşturmak için bir generator sağlar.

	$ rails generate integration_test user_flows
	      exists  test/integration/
	      create  test/integration/user_flows_test.rb

Aşağıdaki ise yeni oluşturulan bir integration testidir.

	require 'test_helper'
 
	class UserFlowsTest ActionDispatch::IntegrationTest
	  fixtures :all 
	 
	  # Replace this with your real tests.
	  test "the truth" do
	    assert true
	  end
	end

İntegration testlerini Action Dispatch::İntegrationTest devralır. Bu bazı ek helper 'lar integration testlerini kullanmamız için yapılır. Ayrıca testi açıkca kullanabilmemiz, testin kullanılabilir olması için fixtures 'e ihtiyaç duyabiliriz.

  5.1) İntegration Testleri için Mevcut Helper 'lar
--- 
<table>
	<tr>
		<th>Helper                                                                           </th>
		<th>Açıklamaları</th>
	</tr>
	<tr>
		<td><tt>https?</tt>                                                                           </td>
		<td>Oturum(session) halinde güvenli bir HTTPS isteği taklit ediliyor ise <tt>true</tt> döndürür.</td>
	</tr>
	<tr>
		<td><tt>https!</tt>                                                                           </td>
		<td>Güvenli bir HTTPS isteği taklit edilmesine izin verir.</td>
	</tr>
	<tr>
		<td><tt>host!</tt>                                                                            </td>
		<td>Bir sonraki isteği kullanabilmemiz için host aını ayarlamamızı sağlar.</td>
	</tr>
	<tr>
		<td><tt>redirect?</tt>                                                                        </td>
		<td>Eğer son istek bir yönlendirme ise <tt>true</tt> döndürür.</td>
	</tr>
	<tr>
		<td><tt>follow_redirect!</tt>                                                                 </td>
		<td>Tek bir yönlendirme yanıtını takip eder.</td>
	</tr>
	<tr>
		<td><tt>request_via_redirect(http_method, path, [parameters], [headers])</tt>                 </td>
		<td>Bir HTTP isteğinin yapılmasını ve bir sonraki yönlendirmelerden herhangi birinin takibini sağlar.</td>
	</tr>
	<tr>
		<td><tt>post_via_redirect(path, [parameters], [headers])</tt>                                 </td>
		<td>Bir HTTP POST isteğinin yapılmasını ve sonraki yönlendirmelerden herhangi birinin takibini sağlar.</td>
	</tr>
	<tr>
		<td><tt>get_via_redirect(path, [parameters], [headers])</tt>                                  </td>
		<td>Bir HTTP GET isteğinin yapılmasını ve sonraki yönlendirmelerden herhangi birinin takibini sağlar.</td>
	</tr>
	<tr>
		<td><tt>put_via_redirect(path, [parameters], [headers])</tt>                                  </td>
		<td>Bir HTTP PUT isteğinin yapılmasını ve sonraki yönlendirmelerden herhangi birinin takibini sağlar.</td>
	</tr>
	<tr>
		<td><tt>delete_via_redirect(path, [parameters], [headers])</tt>                               </td>
		<td>Bir HTTP DELETE isteğinin yapılmasını ve bir sonraki yönlendirmelerden herhangi birinin takibini sağlar.</td>
	</tr>
	<tr>
		<td><tt>open_session</tt>                                                                     </td>
		<td>Yeni bir oturum(session) örneği açar.</td>
	</tr>
</table>

  5.2) İntegration Test Örnekleri
---

Birden fazla/çoklu controller egzersizleri için basit bir integration testi gösterecek olursak:

	require 'test_helper'
 
	class UserFlowsTest < ActionDispatch::IntegrationTest
	  fixtures :users
	 
	  test "login and browse site" do
	    # login via https
	    https!
	    get "/login"
	    assert_response :success
	 
	    post_via_redirect "/login", :username => users(:avs).username, :password => users(:avs).password
	    assert_equal '/welcome', path
	    assert_equal 'Welcome avs!', flash[:notice]
	 
	    https!(false)
	    get "/posts/all"
	    assert_response :success
	    assert assigns(:products)
	  end
	end

İntegration testleri birden fazla controller içerir ve veritabanı göndereni/denetleyicisi ile tüm yığın egzersizlerini görebiliriz.
Ayrıca birden fazla session örnekleri ile bir testi aynı anda açabiliriz ve sadece bizim uygulamamız için çok güçlü bir test olan DSL(etki alanına özgü dil) oluşturmak için onaylama yöntemleri ile bu rnekleri uzatabiliriz/büyütebiliriz.

Aşağıdaki test çoklu session ve özel DSL örneği içeren bir integration testidir:

	require 'test_helper'
 
	class UserFlowsTest < ActionDispatch::IntegrationTest
	  fixtures :users
	 
	  test "login and browse site" do
	 
	    # User avs logs in
	    avs = login(:avs)
	    # User guest logs in
	    guest = login(:guest)
	 
	    # Both are now available in different sessions
	    assert_equal 'Welcome avs!', avs.flash[:notice]
	    assert_equal 'Welcome guest!', guest.flash[:notice]
	 
	    # User avs can browse site
	    avs.browses_site
	    # User guest can browse site as well
	    guest.browses_site
	 
	    # Continue with other assertions
	  end
 
	  private
 
	  module CustomDsl
	    def browses_site
	      get "/products/all"
	      assert_response :success
	      assert assigns(:products)
	    end
	  end
	 
	  def login(user)
	    open_session do |sess|
	      sess.extend(CustomDsl)
	      u = users(user)
	      sess.https!
	      sess.post "/login", :username => u.username, :password => u.password
	      assert_equal '/welcome', path
	      sess.https!(false)
	    end
	  end
	end

  6) Testleri Çalıştırmamız için Gereken Rake Görevleri
---
Test kodlarında, elimizle testleri kurmamız ve çalıştırmamız gerekmez. Rails de, testlerde yardımcı olması için rake task 'ler bir dizi halinde birlikte geliyor. Aşağıdaki tabloda ise bir rails projesi başlattığımızda varsayılan Rakefile boyunca gelen tüm rake 'ler açıklamaları ile birlikte listelenmiştir. 

<table>
	<tr>
		<th>Görevler                         </th>
		<th>Tanımlama</th>
	</tr>
	<tr>
		<td><tt>rake test</tt>                     </td>
		<td>Tüm unit, functional ve integration testlerini çalıştırır. Ayrıca testin hedefi olan, varsayılan rake ide çalıştırabiliriz.</td>
	</tr>
	<tr>
		<td><tt>rake test:benchmark</tt>           </td>
		<td>benchmark(karşılaştırmalı değerlendirme) performans testleri</td>
	</tr>
	<tr>
		<td><tt>rake test:functionals</tt>         </td>
		<td><tt>test/functional</tt> da bulunan tüm functional testlerini çalıştırır.</td>
	</tr>
	<tr>
		<td><tt>rake test:integration</tt>         </td>
		<td><tt>test/integration</tt> da bulunan tüm integration testlerini çalıştırır.</td>
	</tr>
	<tr>
		<td><tt>rake test:plugins</tt>             </td>
		<td><tt>vendor/plugins/*/**/test</tt> (veya <tt>PLUGIN=_name_</tt> ilede belirtebiliriz) tüm plugin(eklenti) testlerini çalıştırır.</td>
	</tr>
	<tr>
		<td><tt>rake test:profile</tt>             </td>
		<td>Profildeki performans testleri çalıştırılır.</td>
	</tr>
	<tr>
		<td><tt>rake test:recent</tt>              </td>
		<td>Testlerdeki son değişiklikler çalıştırılır.</td>
	</tr>
	<tr>
		<td><tt>rake test:uncommitted</tt>         </td>
		<td>Kaydedilmemiş tüm testleri çalıştırır. Subversion ve Git destekler.</td>
	</tr>
	<tr>
		<td><tt>rake test:units</tt>               </td>
		<td><tt>test/unit</tt> de bulunan tüm unit testlerini çalıştırır.</td>
	</tr>
</table>
	
**Yararlanılan Kaynak**

[Ruby on Rails](http://guides.rubyonrails.org/)
	
 
