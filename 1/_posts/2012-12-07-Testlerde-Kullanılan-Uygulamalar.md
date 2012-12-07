---
layout: post
title:  Testlerde Kullanılan Uygulamalar ve Amaçları
---
## 3.4) Mevcut Uygulamalar ve Amaçları

<table>
  <tr>
    <td>assert(boolean, [msg])</td>
    <td>Sonucu değerlendirir.Bu yöntem program geliştirme esnasında mantık hatalarını tanımlamak için kullanılır.</td>
  </tr>
  <tr>
    <td>assert_equal(obj1, obj2, [msg])</td>
    <td>Obj1 ile Obj2 eşit olmalıdır.Bu eşitliği teyit eder.</td>
  </tr>
  <tr>
    <td>assert_not_equal(obj1, obj2, [msg])</td>
    <td>Obj1 ile Obj2 eşit olmadığını belirtir.</td>
  </tr>
  <tr>
    <td>assert_same(obj1, obj2, [msg])</td>
    <td>Eğer iki nesne aynı kimliğe sahip ise aynıdır.</td>
  </tr>
  <tr>
    <td>assert_not_same(obj1, obj2, [msg])</td>
    <td>İki nesne aynı değildir.</td>
  </tr>
  <tr>
    <td>assert_nil(obj, [msg])</td>
    <td>Obj = nil olması beklenir. Obje nil olmayınca başarısızdır.</td>
  </tr>
  <tr>
    <td>assert_not_nil(obj, [msg])</td>
    <td>Obj 'nin nil olmadığını belirtir.</td>
  </tr>
  <tr>
    <td>assert_match(regexp, string, [msg])</td>
    <td>Bir dize ile düzenli ifaenin eşleşmesini sağlar.</td>
  </tr>
  <tr>
    <td>assert_no_match(regexp, string, [msg])</td>
    <td>Bir dize ile bir düzenli ifade eşleşmesi olmadığını belirtir.</td>
  </tr>
  <tr>
    <td>assert_in_delta(expecting, actual, delta, [msg])</td>
    <td>Beklenen değerin ve gerçek değerin deltadan fazla farkı olmadığını belirtir.</td>
  </tr>
  <tr>
    <td>assert_throws(symbol, [msg]) {block}</td>
    <td>Verilen sembolü bloğa atar.</td>
  </tr>
  <tr>
    <td>assert_raise(exception1, exception2, ...) {block}</td>
    <td>Testlerde verilen istisnaları bloklarda tutmaya yarar.</td>
  </tr>
  <tr>
    <td>assert_nothing_raised(exception1, exception2, ...) {block}</td>
    <td>Testlerde verilen istisnalardan hiçbiri bloklara yerleştirilmez.</td>
  </tr>
  <tr>
    <td>assert_instance_of(class, obj, [msg])</td>
    <td>Verilen objenin, verilen sınıfın bir örneği olduğunu belirtir.Değilse başarısız.</td>
  </tr>
  <tr>
    <td>assert_kind_of(class, obj, [msg])</td>
    <td>Verilen objenin, verilen sınıfın bir türü olmadığı sürece başarısız.</td>
  </tr>
  <tr>
    <td>assert_respond_to(obj, symbol, [msg])</td>
    <td>Verilen nesnenin symbol olarak adlandırılan bir yöntemi olduğunu belirtir.</td>
  </tr>
  <tr>
    <td>assert_operator(obj1, operator, obj2, [msg])</td>
    <td>Operator kullanılarak obj1 ile obj2 karşılaştırılır.</td>
  </tr>
  <tr>
    <td>assert_send(array, [msg])</td>
    <td>Burada array bir mesaj,alıcı ve değişkendir.Gönderme işlemi, gerçek bir değeri döndürür ise başarılıdır.</td>
  </tr>
  <tr>
    <td>flunk([msg])</td>
    <td>Başarısız durum belirtir.</td>
  </tr>

</table>


## 3.5) Özel Rails Uygulamaları ve Amaçları


  <table>


<tr>


<td>assert_valid(record)</td>


<td>Geçmiş kayıtların aktif kayıt standartlarına göre geçerli durumda olmasını kontrol eder eğer 

değilse hata mesajı döndürür.</td>


</tr>


<tr>


<td>assert_difference(expressions, difference = 1, message = nil) {...}</td>


<td>Bir blok değerlendirilir ve sonuç olarak ifadenin dönüş değeri arasındaki farka bakılır.</td>


</tr>


<tr>


<td>assert_no_difference(expressions, message = nil, &block)</td>


<td>Bir ifadenin sayısal sonucu değerlendirilmeden önce bloktan çağırılır ve daha önce bu ifade 

de değişiklik yapılmadığını onaylar.</td>


</tr>


<tr>


<td>assert_recognizes(expected_options, path, extras={}, message=nil)</td>


<td>Verilen yol tam anlamıyla ele alınır ve ayrıştırılır, temelde bu expected_options tarafından 

verilen Rails 'de tanımlanan bir yoldur.</td>


</tr>


<tr>


<td>assert_generates(expected_path, options, defaults={}, extras = {}, message=nil)</td>


<td>Bu sağlanılan seçeneklerin bir yol oluşturmak için kullanılacağını belirtir.Bu 

assert_recognizes'in tersidir.Ek parametre değerlerinin bir sorgu dizesi olacağını belirtir.</td>


</tr>


<tr>


<td>assert_response(type, message = nil)</td>


<td>Yanıtı belirli bir durum kodu ile belirtir. success, redirect, missing, error olmak üzere 4 tür 

yanıt verir ve bunları durum kodları ile belirtir.</td>


</tr>


<tr>


<td>assert_redirected_to(options = {}, message=nil)</td>


<td>Yeniden yönlendirme seçeneklerinin, son eylem olarak adlandırılan yönlendirme ile 

eşleştiğini belirtir.</td>


</tr>


<tr>


<td>assert_template(expected = nil, message=nil)</td>


<td>Bu, uygun bir şablon dosyası ile işlendiğini belirtir.</td>


</tr>


  </table>
