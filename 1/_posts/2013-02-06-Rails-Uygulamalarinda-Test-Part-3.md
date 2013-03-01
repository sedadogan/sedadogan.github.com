---
layout: post
title:  Rails Uygulamalarinda Test - PART 3
---

  7) Test::Unit Hakkında Kısa Not
---
Bir kütüphane için küçük bir gem olan Test::Unit, Ruby unit testi için bir frameworktur. Burada sözü edilen iddialar aslında Test::Unit::Assertion 'u tanımalar.
**Not:** Test::Unit hakkında daha fazla bilgiye <a href="http://ruby-doc.org/stdlib-1.9.3/libdoc/test/unit/rdoc/">buradan</a> ulaşabilirsiniz.

  8) Kurulum ve Teardown
---
Bir kod bloğunu çalıştırmak istiyor isek, her test sonunda ve başka bir kod bloğu başlamadan önce kurtarma için iki tane özel çağrı bulunur. Örneğin <tt>Posts</tt> controller 'da functional testlerimiz için verilen örneğe bakacak olursak:

	require 'test_helper'
 
	class PostsControllerTest < ActionController::TestCase
	 
	  # called before every single test
	  def setup
	    @post = posts(:one)
	  end
	 
	  # called after every single test
	  def teardown
	    # as we are re-initializing @post before every test
	    # setting it to nil here is not essential but I hope
	    # you understand how you can use the teardown method
	    @post = nil
	  end
	 
	  test "should show post" do
	    get :show, :id => @post.id
	    assert_response :success
	  end
	 
	  test "should destroy post" do
	    assert_difference('Post.count', -1) do
	      delete :destroy, :id => @post.id
	    end
	 
	    assert_redirected_to posts_path
	  end
	 
	end

Yukarıdaki kurulum metodu, her test öncesinde çağırılabilir ve @post 'da her test için kullanılabilir. Rails, <tt>ActiveSupport::Callbacks</tt> da <tt>kurulum</tt> ve <tt>teardown</tt> uygular. Testler için bu metodları aslında sadece <tt>kurulum</tt> ve <tt>teardown</tt> için kullanmamız gerekmez. Aşağıdaki ifadeleri kullanarak da belirtebiliriz:

   - blok
   - bir yöntem (önceki örnekte olduğu gibi)
   - yöntem adı bir sembol olacak
   - bir lambda 

Bir sembol olarak bir yöntem adı ve kuruluma özgü geriçağırım belirterek önceki örneği tekrar ele alacak olursak:

	require '../test_helper'
 
	class PostsControllerTest < ActionController::TestCase
	 
	  # called before every single test
	  setup :initialize_post
	 
	  # called after every single test
	  def teardown
	    @post = nil
	  end
	 
	  test "should show post" do
	    get :show, :id => @post.id
	    assert_response :success
	  end
	 
	  test "should update post" do
	    put :update, :id => @post.id, :post => { }
	    assert_redirected_to post_path(assigns(:post))
	  end
	 
	  test "should destroy post" do
	    assert_difference('Post.count', -1) do
	      delete :destroy, :id => @post.id
	    end
	 
	    assert_redirected_to posts_path
	  end
	 
	  private
	 
	  def initialize_post
	    @post = posts(:one)
	  end
	 
	end

  9) Testing Routes(Yönlendirme Testi)
---
Rails uygulamalarında ki herşey gibi, routes(yönlendirme) leride test etmemeiz tavsiye edilir. Örneğin testte <tt>Posts</tt> controller 'ın varsayılan <tt>show</tt> eylemi bir route için örnek testi aşağıdaki gibi olmalıdır:

	test "should route to post" do
	  assert_routing '/posts/1', { :controller => "posts", :action => "show", :id => "1" }
	end 

  10) Testing Your Mailers(Mailer Testi)
---
Mailer sınıflarının testinde bazı özel araçları kullanarak kapsamlı bir iş yapmak gerekir.

  10.1) Keeping the Postman in Check
---
Mailer sınıfları (Rails uygulamamızın diğer herbir parçası gibi) beklendiği gibi çalıştığından emin olmak için test edilmelidir. Mailer sınıfında testin amacına ulaşması için şunlar yapılır:

   - e-posta 'lar işleniyor.(oluşturulması ve gönderilmesi)
   - e-posta içeriğinin doğruluğu kontrol ediliyor.(konu, gönderen,vb.)
   - doğru e-posta 'lar doğru zamanlarda gönderiliyor.

  10.1.1) From All Sides
---
Mailer testlerinin iki durumu vardır; unit testleri ve functional testler. Unit testleri, sıkı kontrol girişleri ile mailer 'ı çalıştırır ve bilinen bir değer için çıkış ile karşılaştırır(fixture.) Functional testlerde mailer tarafından üretilen dakika ayrıntılarını test etmek yerine bizim controller ve models 'ların doğru bir şekilde mailer kullandıklarını test etmeliyiz. Doğru zamanda doğru e-postanın gönderildiğini kanıtlamak için test edilir.

  10.2) Unit Testi
---
Mailer 'ın beklenildiği gibi çalışıp çalışmadığını test etmek için, üretilenin ne olması gerektiği önceden yazılı örnekler ile belirtilir ve mailer 'ın gerçek sonuçları ile karşılaştırılır, bunu yapabilmek için ise unit testi kullanılır.

  10.2.1) Fixture 'nin İntikamı :)
---
Bir mailer unit testinin amaçları için; fixture, bir çıktının nasıl görüneceğini bir örnek olarak sunabilmek için de kullanılır. Çünkü bu örnek e-postalar diğer fixture 'ler gibi ActiveRecord verisi değildir, onlar diğer fixture 'lerin dışında kendi alt dizininde tutulur. <tt>Test/fixtures</tt> içindeki dizinin adı doğrudan mailer adına karşılık gelir. Yani bir mailer adı olan <tt>UserMailer</tt> için, fixture <tt>test/fixture/user_mailer</tt> dizininde olmalıdır. 
Eğer biz mailer oluşturduğumuzda(generate ile); generator, mailer eylemlerinin her biri için temel/saplama fixture 'ler oluşturur. Eğer generator işe yaramadıysa bu dosyaları kendimiz oluşturmak zorundayız.

  10.2.2) Temel TestCase 'i
---
Burada ki eylem bir arkadaşımıza davetiye göndermek için kullanılan <tt>UserMailer</tt> adındaki mailer testini sınamak için kullanılan bir unit testi bulunmaktadır. Burada bir davet eylemi için generator tarafından oluşturulan temel testin uyarlanmış hali/versiyonudur.

	require 'test_helper'
	 
	class UserMailerTest  ActionMailer::TestCase
	  tests UserMailer
	  test "invite" do
	    @expected.from    = 'me@example.com'
	    @expected.to      = 'friend@example.com'
	    @expected.subject = "You have been invited by #{@expected.from}"
	    @expected.body    = read_fixture('invite')
	    @expected.date    = Time.now
	 
	    assert_equal @expected.encoded, UserMailer.create_invite('me@example.com', 'friend@example.com', @expected.date).encoded
	  end
	 
	end

Bu testte, @expected bizim testlerimizde kullanabileceğimiz bir <tt>TMail::Mail</tt> örneğidir. Bu <tt>ActionMailer::TestCase</tt> içine tanımlanır. Yukarıdaki testte @expected kullanımları bir e-posta oluşturur, daha sonra özel mailer tarafında oluşturulan e-posta ile belirtilir. Davet fixture bir e-posta gövdesidir ve örnek içeriğe karşı savunma olarak kullanılır. <tt>read_fixture</tt> helper 'ı ise bu dosya içerigini okumak için kullanılır.
Aşağıda davet fixture 'sinin içeriği bulunmaktadır:

<pre>
Hi friend@example.com,

You have been invited.

Cheers!
</pre>

<tt>config/environments/test.rb</tt> içindeki <tt>ActionMailer::Base.delivery_method = :test</tt> satırı bu e-posta teslim edilmeyecek şekilde teslim yöntemini test moduna ayarlanabilir(Bu, test ederken spam kullanıcılarını önlemek için yararlıdır) ama bunun yerine bir dizi eklenir(<tt>ActionMailer::Base.deliveries</tt>). 
Ancak genellikle unit testleriyle, yukarıdaki örnekte olduğu gibi basit bir inşa ile e-postanın hassas içeriğinin ne olması gerektiği gerektiği kontrol edilir ve buna göre postalar gönderilmez.

  10.3) Functional Testi
---
Mailer 'lar için functional testleri, sadece e-posta gövdesi, alıcıları ve doğru olduğunun kontrolünden daha fazlasını içerir. Functional mail testleri posta teslim yöntemleri sunar ve uygun e-postaların teslim listesine ekli olup olmadığını kontrol etmemizi sağlar.
Örneğin arkadaş davet çalışmasını uygun bir e-posta göndererek kontrol edebiliriz:

	require 'test_helper'
	 
	class UserControllerTest < ActionController::TestCase
	  test "invite friend" do
	    assert_difference 'ActionMailer::Base.deliveries.size', +1 do
	      post :invite_friend, :email => 'friend@example.com'
	    end
	    invite_email = ActionMailer::Base.deliveries.last
	 
	    assert_equal "You have been invited by me@example.com", invite_email.subject
	    assert_equal 'friend@example.com', invite_email.to[0]
	    assert_match(/Hi friend@example.com/, invite_email.body)
	  end
	end

  11) Diğer Test Yaklaşımları
---
Yerleşik <tt>test/unit</tt> tabanlı test, Rails uygulamalarını test etmek için tek yol değildir. Rails geliştiricileride dahil olmak üzere, test için diğer yaklaşımlar ve buna yardımcı ifadeler ile geniş bir yelpaze sunuluyor:

   - <a href="http://avdi.org/projects/nulldb/">NullDB</a>, veritabanı kullanımından kaçınarak testi hızlandırmak için bir yoldur.
   - <a href="https://github.com/thoughtbot/factory_girl">Factory Girl</a>, fixture 'ler için yedektir.
   - <a href="https://github.com/notahat/machinist">Machinist</a>, fixture 'ler için diğer bir yedektir.
   - <a href="http://www.thoughtbot.com/community">Shoulda</a>, ek helpers, makrolar ve iddialar ile <tt>test/unit</tt> için bir uzantıdır.
   - <a href="https://www.relishapp.com/rspec">RSpec</a>, bir davranış odaklı geliştirme framework 'udur.

**Yararlanılan Kaynak**

[Ruby on Rails](http://guides.rubyonrails.org/)
