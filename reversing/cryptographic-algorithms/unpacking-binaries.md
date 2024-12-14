# Paketlenmiş ikililerin tanımlanması

* **dize eksikliği**: Paketlenmiş ikililerin neredeyse hiç dize içermediğini bulmak yaygındır.
* Birçok **kullanılmayan dize**: Ayrıca, bir kötü amaçlı yazılım bazı ticari paketleyiciler kullanıyorsa, çapraz referanssız birçok dize bulmak yaygındır. Bu dizeler mevcut olsa bile, bu durum ikilinin paketlenmediği anlamına gelmez.
* Bir ikilinin hangi paketleyici ile paketlendiğini bulmak için bazı araçlar kullanabilirsiniz:
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Language 2000](http://farrokhi.net/language/)

# Temel Öneriler

* Paketlenmiş ikiliyi **IDA'da alttan başlayarak analiz etmeye** başlayın ve yukarı doğru ilerleyin. Paket açıcılar, açılmış kod çıkınca çıkış yapar, bu nedenle paket açıcının açılmış koda başlangıçta yürütme geçirmesi olası değildir.
* **Kayıtlara** veya **bellek** **bölgelerine** **JMP** veya **CALL** arayın. Ayrıca, **argümanlar ve bir adres yönlendirmesi iten işlevler arayın ve ardından `retn` çağırın**, çünkü bu durumda işlevin dönüşü, yığına itilen adresi çağırmadan önce çağırabilir.
* `VirtualAlloc` üzerinde bir **kesme noktası** koyun, çünkü bu, programın açılmış kod yazabileceği bellek alanı ayırır. "Kullanıcı koduna koş" veya işlevi çalıştırdıktan sonra **EAX içindeki değere ulaşmak için F8** kullanın ve "**bu adrese dökümde takip et**". Açılmış kodun kaydedileceği bölge olup olmadığını asla bilemezsiniz.
* **`VirtualAlloc`** ile "**40**" değeri argüman olarak, Okuma+Yazma+Çalıştırma anlamına gelir (çalıştırılması gereken bazı kod burada kopyalanacak).
* **Kod açma** sırasında, **aritmetik işlemler** ve **`memcopy`** veya **`Virtual`**`Alloc` gibi işlevlere **birçok çağrı** bulmak normaldir. Eğer yalnızca aritmetik işlemler gerçekleştiren ve belki de bazı `memcopy` yapan bir işlevde bulursanız, öneri, **işlevin sonunu bulmaya çalışmak** (belki bir JMP veya bazı kayıtlara çağrı) **veya** en azından **son işleve yapılan çağrıyı bulmak** ve ardından çalıştırmaktır, çünkü kod ilginç değildir.
* Kod açma sırasında, **bellek bölgesini değiştirdiğinizde** not alın, çünkü bir bellek bölgesi değişikliği **açma kodunun başlangıcını** gösterebilir. Process Hacker kullanarak bir bellek bölgesini kolayca dökebilirsiniz (işlem --> özellikler --> bellek).
* Kod açmaya çalışırken, **açılmış kodla çalışıp çalışmadığınızı bilmenin** iyi bir yolu, **ikilinin dizelerini kontrol etmektir**. Eğer bir noktada bir atlama yaparsanız (belki bellek bölgesini değiştirerek) ve **çok daha fazla dize eklendiğini** fark ederseniz, o zaman **açılmış kodla çalıştığınızı** bilebilirsiniz.\
Ancak, eğer paketleyici zaten birçok dize içeriyorsa, "http" kelimesini içeren dize sayısını görebilir ve bu sayının artıp artmadığını kontrol edebilirsiniz.
* Bir bellek bölgesinden bir çalıştırılabilir dosyayı dökerken, bazı başlıkları [PE-bear](https://github.com/hasherezade/pe-bear-releases/releases) kullanarak düzeltebilirsiniz.
