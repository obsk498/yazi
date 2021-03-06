Selamlar,

İnsan bir şeyle uğraştığında her olaya o anki uğraştığı konunun penceresinden yaklaşıyor. Bende bir zamanlar sadece kapsam genişletme konusuna odaklandığım için hep bu pencereden yaklaşıyordum. Binlerce lira harcayıp sunucular alınmış, IP adresleri kiralanmış, alan adları alınmış koskoca bir orman var ortada sadece 1 metre gidip bir kaç ağaca çıkıp/çıkamayıp geri dönmek içime sinmiyor. Koskoca ormanı iyice bir didik didik etmek ve tepesinde muz olan ağaca çıkmak gerekli, bunu da belli bir hizaya göre yapmak lazım tabii, onun için haritalandırma yapmak iyidir. Yani en azından ben böyle düşünüyorum. 

Haritalandırma yaparken temeli atmak önemli, kuruma ait alt alan adlarını elde ettik tamam ama bunlar hangi IP adresine işaretlenmiş? IP adresleri hedef şirkete mi ait? Kim adına kiralanmış? Nereden kiralanmış? IP adreslerinde başka hangi alan adları ve alt alan adları var? Bu doğrultuda verileri karıştırmadan tutmak zor olabiliyor bu yüzden haritalandırma işleminde [FreeMind](https://freemind.sourceforge.net) adındaki uygulamayı kullanıyorum, benim işime yarıyor.

Bu yazıda kısıtlı bilgiye sahip olduğumuz(blackbox) denetimlerde alan adı kapsam genişletme yöntemleri anlatmaya çalışacağım. Bölümler kısa kısa tutulup fazla konu yazmaya çalıştım, umarım yararlı olur. 

Öncelikle hedef alt ağını(yani IP kapsamını) araştırmak ve kurumun bu ağda nerede olduğunu öğrenmenin, ardından alan adları üzerinden giderek gerek alan adı kapsamının gerekse IP kapsamını genişletilmesinin ve ardından alt alan adlarına ayrılmanın daha mantıklı olduğunu düşünüyorum. Bu yazıda bahsettiğim sırayla ele almaya çalıştım.

Bu benim yoğurt yiyişimdir, herkes aynı yoldan gitmeyebilir, eminim daha iyi bir haritalandırma yolları izleyenler vardır. Siz de bunu kendinize göre harmanlayabilirsiniz.

# Alt Ağ Kapsamları ve Dahilindeki IPv4 Adreslerinin Keşifleri
Alan adlarını ele alarak kapsamı genişletmeden önce kurumun hangi IP adreslerini kullandığını öğrenmemiz faydalı olabilir. Bu keşif çalışması bize kapsamın büyüklüğü hakkında bilgi verecek ve hangi sunucuların kuruma ait olduğunu belirlememizde önemli rol oynayacaktır. Bu bilgileri belirlemekteki asıl amacımız dış sunucuların yerel ağ ile bağlantısını sorgulamaktır. Normal şartlarda kurum, web sunucularını kendi bünyesinde barındırıyorsa bu sunucuların bir DMZ ağına dahil olması gerekmektedir. Saldırganların buradaki stratejisinin dış ağdan yerel ağa erişim sağlamak olacağını varsayarsak DMZ bölgesine girip oradan yerel ağa ulaşmaya çalışacaklardır. Yerel ağa dışarıdan erişim sağlanabilecek noktaları tespit etmek için dış ağ kapsamının doğru şekilde belirlenmesi önemlidir.

## IPv4 Adresleri ve Alt Ağ Kapsamının Öğrenilmesi
Takip edilen senaryoda hedef kuruma ait olan bütün IP adreslerine ulaşmamız önemli olmakla birlikte bence gereklidir de. Bunu gerçekleştirmek için öncelikle kurum ağ yapısını temel olarak tanımak gerekiyor. MX, DNS gibi sunucuların nerede barındırıldığını, SPF kayıtlarında hangi IP adreslerine müsade edildiğini ve bu IP adreslerinin kimlere ait olduğunu öğrenmek iyi bir başlangıç olabilir.

### Genel Ağ Yapısının Haritalandırılması
Yukarıda bahsettiğimiz gibi her şeyden önce kapsamı tanımalıyız. Hedef kurum, sunucuları kendi bünyesinde mi barındırıyor yoksa dışarıdan mı kiralıyor? Kiralama ya da satın alma işlerinde hangi şirketler ile çalışıyor? gibi sorular sormak kapsamı belirlemek açısından temel oluşturmaktadır. 

Sorularımızın cevaplarına ulaşabilmek için alan adı(domain), internet servis sağlayıcısı(ISP), IP adresleri ve bu IP adreslerinin alt ağları(subnet), DNS sunucuları ve IP adresleri, MX sunucuları ve IP adresleri, her bir farklı adresin AS numarasını öğrenip bu AS numaralarının ortak bir kümede kesişip kesişmediğini incelemeliyiz. Bu bilgileri elde ettikten sonra genişletilebilecek kapsamı öğrenmiş olacağız. 

Yukarıda bahsi geçen tüm bilgiler çevrimdışı yada çevrimiçi uygulamalardan elde edilebilir, benim genelde kullandığım çevrimiçi uygulamalar: [TCPIPUtils](https://www.tcpiputils.com), [ViewDNS](http://www.viewdns.info), [BGP-HE](https://bgp.he.net/). Tabii bunların dışında basit bir dig sorgusu da yapabiliriz.

~~~
dig any intel.com
Alan adı: intel.com (13.91.95.74)
Alan adı sahibi: Microsoft Corporation
ISS(ISP): Microsoft Corporation
IP aralığı: 13.64.0.0/11 - ASN: 8075

DNS sunucuları: 
ns1.intel.com (192.55.52.33)
ns2.intel.com (143.182.124.19)
ns3.intel.com (143.183.152.22)
ns4.intel.com (192.102.198.240)

MX sunucuları:
mga03.intel.com (134.134.136.65)
mga04.intel.com (192.55.52.120)
mga09.intel.com (134.134.136.24)
mga14.intel.com (192.55.52.115)
...
~~~
{: .language-bash}

Elde ettiğimiz kayıtlar sonucunda alan adı ve internet servis sağlayıcısı(ISS)'nın `Microsoft Corporation` şirketine ait olduğunu, DNS ve MX sunucularına baktığımızda farklı IP adreslerine işaretlenmiş olan servisler olduğunu görüyoruz. AS numarasını incelediğimiz zaman çok fazla IP aralığı görmekteyiz. Önerim; bir AS numarasına bağlı bu kadar fazla IP aralığı görürseniz öncelikle IP ve alt ağlarını bi yere not edip oradan çıkın, henüz elimizdeki alt ağı aydınlatmadığımız için ortalığı diğerleriyle karıştırmamalıyız. 

AS numarasının takibini yine yukarıdaki web sitelerinden bir tanesinde ya da aşağıdaki gibi bir `whois` sorgusu ile yapabilirsiniz.

~~~
whois -h asn.shadowserver.org 'prefix 8075'

13.64.0.0/11
13.107.14.0/24
13.107.20.0/24
13.104.0.0/14
...
~~~
{: .language-bash}

Elde ettiğimiz bilgiler doğrultusunda ilk aşamada genişletmemiz gereken 2 kapsam bulunuyor; IP ve alan adı. Bulduğumuz alan adlarında karşımıza farklı IP adresleri çıkabileceği gibi IP aramalarında da farklı alan adlarıyla karşılaşabiliriz. 

Buraya kadar ki yapının çıktısı aşağı yukarı şu şekilde olmalı:

![resim](../eg/freemind-intel.png)

## Alan Adlarının IP Geçmişleri
Yukarıdaki aramalarda elde ettiğimiz IP alt ağlarının haricinde başka alt ağlar da mevcut olabilir. Bunları öğrenmek için elimizdeki alan adlarının IP geçmişlerini sorgulayabiliriz. Sorgulamanın sonucunda alan adının önceden kullanmış olduğu IP adreslerini görebiliriz. Araştırdığımız alan adları aynı alt ağ içerisinde farklı sunuculara taşınmış olabilir, eğer karşımıza bildiğimizden farklı IP adresleri çıkarsa onlarında kurum ile ilişiğini araştırmamız gerekebilir.

Aşağıda, `intel.com` adresinin IP geçmişini sorguladığımızda bu alan adının daha önceden `Intel Corporation` adı altında alınmış IP adreslerinde saklandığını görebiliriz.

http://www.viewdns.info/iphistory/?domain=intel.com

Bu IP adreslerinin ISS'nı sorguladığımızda IP alt ağının `Intel` şirketinde olduğu doğrulanmaktadır, bu da kapsama yeni bir alt ağ ekleyeceğimiz anlamına gelmektedir.

Yukarıdaki adımlardakine benzer bir döngüyü `MX`, `NS` gibi sunucuları da işin içine katıp devam ettirerek IP kapsamının genişletilmesi mümkün. Önümüzdeki bölümlerde IP kapsamını genişletmek ziyade alan adı kapsamına daha çok değinmeye çalışacağım.

# Alan Adı ve Alt Alan Adı Keşifleri
Alan adlarını temel olarak sunuculara erişim sağlamak için kullanılan IP maskeleri olarak tanımlayabiliriz. Alan adları, alan adı sağlayıcısı ve sunucu sağlayıcılarının anlaşıp DNS sunucularını aracı olarak tutmasıyla eşleşiyor ve yine DNS aracılığı ile yayımlanıyorlar. 

Küçük çaplı firmalar için çoğu zaman bir alan adı yeterli olabiliyor ancak bazı firmalar birincil olarak kullandıkları alan adlarının yanında başka alan adları da alabiliyorlar bu da kapsamı genişletme açısından oldukça önemlidir. Bir banka kendi alan adının yanında iştirakleri için veya kampanyaları için de alan adları alabilir ve alan adları bankanın kendi IP alt ağına bağlı bir sunucularda tutuluyor olabilir, bu durumda kapsam daha da çok genişletebilir.

Bu bölümde birden fazla başlık ele alacaktım ancak alt alan adları kısmında yazmanın daha derli toplu olabileceğini düşündüm. Şu an sadece alan adı kapsamını genişletebileceğimiz "Açık Kaynak Araştırmaları" başlığına odaklanacağız.

## Alan Adı Keşiflerinde Açık Kaynak Araştırmaları
Aslında adından çağrıştırdığı üzere Açık Kaynak İstihbarat'ı adı altında da ele alınabilir ama istihbarat kelimesini bu duruma uyduramadığım için sadece sorgulama diyeceğim. Bu başlıkta temel olarak alan adının tescil bilgileri, DNS, MX, TXT gibi kayıtları ele alarak arama motorlarında araştırma yapacağız.

### Tescil Bilgileri ile Alan Adlarının Araştırılması
Tescil bilgileri firmanın alan adı alma işlemlerinde belirtilen adres, e-posta, telefon gibi bilgilerdir. Tescil işlemleri genelde `{dns.admin}@firmaadi.com.tr` şeklinde bir e-posta ya da bu işlerden sorumlu kişilerin üzerinden gerçekleştirilirler. 

Temel mantık internete yayın yapan "whois" sitelerindeki tescil bilgilerinin, arama motorları tarafından kaydı altına alınmış olanlarını bulmak ve aynı tescil bilgileri ile alınmış yeni alan adlarını keşfetmektir. Bu alan adları genelde kampanya ve iştirakler için alınan alan adları olabilirler. Bunların yanında keşfettiğimiz alan adlarının bazı tescil bilgileri farklı olabilir, bunun sebebi olarak farklı bir kampanya için alınacak alan adından sorumlu bölümün farklı olması veya sorumlu kişilerin değişmiş olma ihtimallerini düşünebiliriz. Mesela `firmaadi.com.tr` adresinin kayıt edildiği e-posta adresinin `dns.yonetim@firmaadi.com.tr` olduğunu ve telefon numarasının `(0212)111-22-3344` olduğunu varsayalım, bu telefon numarası ile kaydı alınmış ama `isim.soyisim@firmaadi.com.tr` e-posta adresini gösteren alan adları da olabilir. Böyle bir durumda yeni e-posta adreslerini öğrenebilir ve kapsamımızı bu e-posta adresi üzerinden araştırma yaparak daha da genişletebiliriz. Google'da ilgili e-posta/telefon/faks/organizasyon bilgilerini aratınca yeni alan adlarıyla ilgili bir çok sonuç izlenebilir. Sonuçlarda listelenen `whois` siteleri bilgileri okunabilir metin olarak tuttukları için Google gibi arama motorları ilgili içerikleri daha sonraki aramalar için kayıt altına(index'lemek) almaktadır, arama motorlarının bu yönünden faydalanıp yeni alan adları için keşif yapabiliriz. 

Öncelikle elimizdeki alan adı kapsamını genişletmek için tescil bilgilerini öğrenmeliyiz, bunun için şirketin tescil bilgileri eğer gizlenmediyse(büyük şirketlerde genelde gizlenmez) basit bir "whois" sorgusu sonucu bu bilgilere ulaşılabilenecektir. 

![eg-alan-adi-kayit](../eg/alanadi-kayit-intel.png)

"""
Yukarıdaki bilgileri kullanarak arama motorları, pastebin gibi alanlar aracılığı ile arama yapılabilir. 

Örnek bir senaryo olma açısından, whois bilgisinden elde edilen e-posta adresiyle arama motorlarına bir göz atalım. 

![eg-alan-adi-arama](../eg/alanadi-arama-intel.png)

Arama motorunda biraz gezindikten sonra yukarıdaki alan adına ulaşılabilinir. Whois bilgilerine bakıldığında ise "Intel Corporation" firmasına ait olduğu görülecektir.
"""

Bu süreçte Google haricindeki diğer arama motorları faydalı olabilir. Aynı işlemi telefon numarası için gerçekleştirdiğinizde farklı sonuçlar elde edebilirsiniz. Materyale(bu durumda alan adları oluyor) atanan her türlü yarı-bağımsız değer bir arama parametresi olabilir.

Elde edilen alan adlarında yeni tescil bilgileri arayabilir ve aynı genişletme işlemlerini yaparak kapsamı daha da çok genişletebilirsiniz. Ayrıca bu konuda interneti kullanmak tamamen bilginize ve hayal gücünüze kalmış.

### Aynı DNS Sunucusunu Kullanan Alan Adlarının Kayıtlarına Ulaşmak
Hedef alan adı ile aynı DNS sunucularını paylaşan web sitelerin araştırılması kapsamın genişletilmesinde rol oynayabilir. 

İnternet üzerinde anlık olarak her türden tarama yapan bir çok uygulama bulunuyor, yeni açılan alan adlarını takip edenler, güncel PTR ve IP kayıtlarını alanlar, sitelerin anlık kaydını alanlar ve benzerleri. Bunların dışında, sürekli olarak DNS adreslerini sorgulayıp veritabanlarında kayıt altına alan ve istek doğrultusunda elindeki bu verileri ücretli veya ücretsiz paylaşan bazı servisler bulunmakta. Bu servislerden faydalanılarak alan adı listesi çıkartılabilir. 

İlk aşamalarda temel DNS sorgularına dayanılarak temel bir DNS haritası çıkartılmalı. Kendisine ait DNS sunucusu olan bir kurumun, bu DNS sunucusuna bağlı diğer alan adları sorgulandığında, aynı DNS sunucusu tarafından kontrol edilen DNS yapısını görebilmek mümkün olabilir. Sorgulama `intel.com` alan adının DNS adresi olan `ns1.intel.com` üzerinden gerçekleştirildiğinde, aşağıdaki gibi bir çok sonuca ulaşmış olunması beklenir. İlgili sorgulama, yine açık kaynaktan bilgi alınabilecek [ViewDNS](http://www.viewdns.info/), [who.is](https://who.is/nameserver/ns1.intel.com) veya [GWebTools](http://www.gwebtools.com/ns-spy/) gibi web uygulamalarından yapılabilir. 

~~~
intel-academic-program.com
intel-designcenter.com
intel-developer-ed.com
intel-developer.com
intel-inside.biz
intel-inside.com
intel-inside.info
intel-inside.net
intel-inside.org
intel-itcenter.com
intel-lgf.com
intel-mktg.com
intel-pentium.com
...
~~~

Yapılan arama sonucunda bulunan adresler hedef firmanın DNS sunucusunda kayıtlı olan diğer adreslerdir. Herkese açık(public) herhangi bir kuruma özelleştirilmemiş DNS sunucusunun sorgulama sonucunda daha karmaşık bir alan adı listeli gözükecektir.

`ns3.p16.dynect.net` paylaşımlı DNS sunucusunun kayıtları aşağıdaki gibidir:

~~~
crenshawcoastdentistry.com
seventhandbelldentalgroup.com
mysmilegenoffice.com
oakbroker.com
rockchalktalk.com
secrettwist.com
nuttreedentaloffice.com
packwoodcreekdentaloffice.com
etrailer-brakes.com
propelenergynugget.info
irvinedentistoffice.com
omolenechallenge.com
clearclaim.net
catturbo.com
concorddentaloffice.com
...
~~~

Yaklaşık olarak 20 bin kayıt listelendi. Kullanıma açık DNS sunucusu olduğu için kayıtlar oldukça kabarık, her birinin IP adresini sorgulayıp hedefteki firmanın alt ağında bulunup bulunmadığını kontrol etmek için de biraz zaman gerekli. Bu sorgulamalar sonucunda ayrıyetten yeni IP adresleri veya paylaşımlı olan IP adresleri keşfedilebilir. 

Bu bölüme dahil olabilecek bazı konuları alt alan adı keşiflerine taşıdım, orası tam yeri gibi. O bölümde yaptığınız araştırmalar sonucunda alan adı da keşfedebilirsiniz. 

## Alt Alan Adlarının Keşfedilmesi
Alan adlarının uzantısı olan, IP adreslerini ya da başka alt alan adlarını/alan adlarını maskelemiş(CNAME) isimlere alt alan adları denenebilir. Yani bir tane alt alan adı bulunduğu zaman CNAME sorgulaması atılırsa belki yeni bir alan adı keşfedilebilir. Bu adımdan sonra başka alan adları da tanımlanabiliyor yani a.b.c.com gibi isimlerle de karşılaşılanabilir.

### IP Adresinden Alan Adı Sorgulamaları
Hedef kurumun sağladığı DNS sunucularına bağlı diğer alan adlarını bulmak, farklı uygulamalara erişmek ve farklı tescil bilgileri elde etmek açısından oldukça önemlidir.

Peki IP'den alan adı sorgulamak nedir ve nasıl sorgulanır?

Normalde alan adına istek yapılarak IPv4 türünde işaret ettiği adres çözülür, yani DNS dilinde; alan adının adres defterindeki "A" kaydı sorgulanır. Bu sorgunun tersi PTR olarak geçmekte, PTR sorguları IPv4 adreslerinin hangi makineyi işaret ettiğini sorgular. Bu iki kayıt türü DNS sunucusunda birbirinden bağımsız olarak saklanır yani ortaya "PTR ile A kaydı eşittir" gibi bir sonuç çıkmamalı. PTR kayıtları, tespit edilen IP adreslerinde sorgulanıp sonuçlarına bakılması ortaya farklı alt alan adları veya alan adları çıkarabilir ayrıca kelime listeleri ile alt alan adı tarama işlemlerinde bulunamayacak kadar karmaşık alan adları listelenebilir. PTR sorguları `nslookup`, `dig` veya `host` gibi uygulamalarla gerçekleştirmek mümkün ancak /24 makineye sorgulama yapılacaksa ufak betikler kullanılması gerekebilir. 

Yazının başlarında, alan adından IP alt ağının öğrenilmesi hakkında bir şeyler yazmıştık, hikayenin bütünlüğü açısından o kapsam üzerinde bir PTR taraması gerçekleştirerek makine isimleri listeleyeceğiz. Aşağıda `nmap` aracılığı ile yapılan örnek bir sorgu bulunmaktadır:

~~~
$nmap -sL -Pn a.b.c.0/24

Nmap scan report for m.intel.net (a.b.c.5)
Nmap scan report for o.intel.com.tr (a.b.c.52)
Nmap scan report for r.intelmuz.com (a.b.c.53)
Nmap scan report for i.abvintel.com (a.b.c.56)
...
~~~

Taramalar sonucunda yukarıdaki gibi sonuçlar elde edilebilir. Bunun yanında her bölümde olduğu gibi bu bölümde de açık kaynaktan araştırma yapılmasını tavsiye ederim, biraz dış kaynaklardan veri toplamak iyi bir fikir olabilir. Yukarılarda bir yerlerde bazı sistemlerin sürekli olarak internette taramalar gerçekleştirdiğinden bahsetmiştik. Açık kaynaktan sorgulama yapılabilecek bir kaç çevrimiçi siteyi buraya bırakıyorum:

https://reverse.report/<br>
https://hackertarget.com/reverse-dns-lookup/

###
Karşılaştırma işlemi için ufak bir araç yazmıştım ama sanırım şu an stabil değil, burada not olarak kalsın;

`./dig.sh liste.txt kapsam.txt`
~~~
#!/bin/sh
for IP in `cat $1`
do
	SATIR = `dig $1 |grep "ANSWER SECTION" -A1|grep "IN"`
	for KAPSAMIP in `cat $2`
	do
		KAPSAM = `echo $KAPSAMIP | cut -d'/' -f2`
		if [ KAPSAM -eq 24 ]; then
			if [ `echo $KAPSAM | cut -d'.' -f1,2,3` -eq `echo $SATIR | awk '{print $5}' | cut -d'.' -f1,2,3` ]; then
				echo "`echo $SATIR | awk '{print $5,\"\\t\",$1}'` -> $KAPSAM"
		else [ KAPSAM -eq 23 ]; then
			if [ `echo $KAPSAM | cut -d'.' -f1,2` -eq `echo $SATIR | awk '{print $5}' | cut -d'.' -f1,2` ]; then
				echo "`echo $SATIR | awk '{print $5,\"\\t\",$1}'` -> $KAPSAM"
~~~
{: .language-bash}
###

### DNS Araştırmaları
Burası daha çok DNS üzerindeki kayıtları öğrenmek için yapılacak sorguların ele alınacağı bölüm olacak, aslında şu ana kadar yapılan ve aşağıda yapılacak çoğu sorgu da DNS araştırmaları kapsamında bulunuyor. 

Bu bölümde yararlı olması açısından `fierce` ve `dnsrecon` araçlarını incelemenizi öneririm.

#### DNS Kayıtlarının Sorgulanması
DNS kayıtları basit sorgular ve açık kaynak kullanılarak toplanabilir.  Alan adını kullanarak bilgi alabileceğimiz DNS kayıtları aşağıdaki listede bulunmaktadır. 

Bu bölümde DNS kayıtlarının ne işe yaradığından ziyade hangi DNS kaydında nasıl bir veriye ulaşabileceğimize dair bilgi bulunabilinecektir.

<b>PTR:</b> Hedef alt ağ(subnet) kapsamındaki alt alan adları öğrenilebilir.<br>
<b>MX:</b> Kurum tarafından kullanılan e-posta sunucularını listeler.<br>
<b>CNAME:</b> Sorgulama yapılan adrese bağlı alan adı veya alt alan adı kaydı öğrenilebilir.<br>
<b>TXT:</b> SPF kayıtları bu sorguda görülebilir. SPF kayıtları ise kurumun e-posta sunucuları ile iletişim kurmaya izinli olan e-posta sunucularını gösterir.<br>
<b>SOA:</b> Bu kayıt ile tercih edilen DNS sunucu bilgisini, alan adı yöneticisinin e-posta adresi elde edilebilir.<br>
<b>SRV:</b> Kurumun DNS'e tanıttığı servislerini kapsar, VoIP servisleri keşfedilebilir.

Bu sorguların hepsi `dig`, `nslookup` gibi uygulamalar ile yapılabilmektedir. Aşağıda örnek bir dig sorgusu bulunmaktadır.

~~~
dig any intel.com
;; Truncated, retrying in TCP mode.

; <<>> DiG 9.8.3-P1 <<>> any intel.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 61577
;; flags: qr rd ra; QUERY: 1, ANSWER: 26, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;intel.com.			IN	ANY

;; ANSWER SECTION:
intel.com.		145	IN	A	13.91.95.74
intel.com.		745	IN	NS	ns2.intel.com.
intel.com.		745	IN	NS	ns3.intel.com.
intel.com.		745	IN	NS	ns1.intel.com.
intel.com.		745	IN	NS	ns4.intel.com.
intel.com.		10645	IN	SOA	ns1.intel.com. hostmaster.intel.com. 127524 43200 300 1209600 900
intel.com.		145	IN	MX	1000 mga07.intel.com.
intel.com.		145	IN	MX	1000 mga12.intel.com.
intel.com.		145	IN	MX	1000 mga17.intel.com.
intel.com.		745	IN	TXT	"h0BlPoSHQX8pdc0+JRlGcTauZJd/RdYLewvr8jnP5v5ZmgWg0RUnoxz+cP3r+6bI4Kd49XTv31Uy/lCOaDYeQg=="
intel.com.		745	IN	TXT	"docusign=46a68707-4a57-4782-bf77-1373777e73e8"
intel.com.		745	IN	TXT	"_spf.q4press.com."
intel.com.		745	IN	TXT	"adobe-idp-site-verification=12d5bea8-aab4-4b2e-9c77-8f69ad4734f0"
intel.com.		745	IN	TXT	"docusign=ff4d259b-5b2b-4dc7-84e5-34dc2c13e83e"
intel.com.		745	IN	TXT	"v=spf1 include:_spf.intel.com include:_spf.google.com  include:_spf.q4press.com -all"
intel.com.		3445	IN	TXT	"MS=B03F616C5688CE657CC2FA94EF4E72109431092B"
~~~

`dnsrecon` aracı ile bazı süreçleri otomatize edebilir ve daha detaylı sonuçlar alınabilir.

#### DNS Kayıtlarının Güvensiz Sunuculara Aktarılması
Bu durum en temel anlatımıyla alan adı sunucularının yabancı kaynaklardan gelen sorgulara açık olmasından kaynaklanmaktadır. Yani hedef DNS sunucusuna, tamamen yabancı bir IP adresi, kayıtların aktarılması(zone transfer) için DNS sorgusu yapar ve hedef DNS bütün kayıtlarını bu kişiye aktarır. Bu zafiyetin varlığı `dig` ile kolaylıkla yoklanabilir. 

Hemen aşağıya başarılı bir yetkisiz kayıt aktarma(zone transfer) saldırısının örnek çıktısını bırakıyorum.

~~~
dig axfr @nsztm1.digi.ninja zonetransfer.me

; <<>> DiG 9.8.3-P1 <<>> axfr @nsztm1.digi.ninja zonetransfer.me
; (1 server found)
;; global options: +cmd
zonetransfer.me.	7200	IN	SOA	nsztm1.digi.ninja. robin.digi.ninja. 2017042001 172800 900 1209600 3600
zonetransfer.me.	300	IN	HINFO	"Casio fx-700G" "Windows XP"
zonetransfer.me.	301	IN	TXT	"google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"
zonetransfer.me.	7200	IN	MX	0 ASPMX.L.GOOGLE.COM.
zonetransfer.me.	7200	IN	MX	10 ALT1.ASPMX.L.GOOGLE.COM.
zonetransfer.me.	7200	IN	MX	10 ALT2.ASPMX.L.GOOGLE.COM.
zonetransfer.me.	7200	IN	MX	20 ASPMX2.GOOGLEMAIL.COM.
zonetransfer.me.	7200	IN	MX	20 ASPMX3.GOOGLEMAIL.COM.
zonetransfer.me.	7200	IN	MX	20 ASPMX4.GOOGLEMAIL.COM.
zonetransfer.me.	7200	IN	MX	20 ASPMX5.GOOGLEMAIL.COM.
zonetransfer.me.	7200	IN	A	5.196.105.14
zonetransfer.me.	7200	IN	NS	nsztm1.digi.ninja.
zonetransfer.me.	7200	IN	NS	nsztm2.digi.ninja.
_sip._tcp.zonetransfer.me. 14000 IN	SRV	0 0 5060 www.zonetransfer.me.
14.105.196.5.IN-ADDR.ARPA.zonetransfer.me. 7200	IN PTR www.zonetransfer.me.
asfdbauthdns.zonetransfer.me. 7900 IN	AFSDB	1 asfdbbox.zonetransfer.me.
asfdbbox.zonetransfer.me. 7200	IN	A	127.0.0.1
asfdbvolume.zonetransfer.me. 7800 IN	AFSDB	1 asfdbbox.zonetransfer.me.
canberra-office.zonetransfer.me. 7200 IN A	202.14.81.230
cmdexec.zonetransfer.me. 300	IN	TXT	"\; ls"
contact.zonetransfer.me. 2592000 IN	TXT	"Remember to call or email Pippa on +44 123 4567890 or pippa@zonetransfer.me when making DNS changes"
dc-office.zonetransfer.me. 7200	IN	A	143.228.181.132
deadbeef.zonetransfer.me. 7201	IN	AAAA	dead:beaf::
dr.zonetransfer.me.	300	IN	LOC	53 20 56.558 N 1 38 33.526 W 0.00m 1m 10000m 10m
DZC.zonetransfer.me.	7200	IN	TXT	"AbCdEfG"
email.zonetransfer.me.	2222	IN	NAPTR	1 1 "P" "E2U+email" "" email.zonetransfer.me.zonetransfer.me.
email.zonetransfer.me.	7200	IN	A	74.125.206.26
home.zonetransfer.me.	7200	IN	A	127.0.0.1
~~~

#### DNS Kaydı Bulunmayan Alan Adlarının Takibi
Bazen alan adına ulaşmak istenildiğinde "Alan adının IP adresini çözülemedi." gibi bir bildiri çıkar. Bu demek oluyor ki; ulaşılabilen DNS sunucuları arasında, istek gerçekleştirilen alan adına eşleştirilmiş bir IP adresi tanımlı değil. Böyle bir durumun olmasının sebepleri DNS kaydının silinmiş veya tanımlanmamış olması olabilir. 

IP adresi bulunmayan bir alan adı/alt alan adı ile karşılaşıldığında bir kenara atılmadan önce kurumun alt ağ kapsamında bulunan sunuculara bu alan adıyla istek yapmak DNS sunucusuna kaydı girilmemiş uygulamalara ulaşma konusunda önemli bir etkendir. Bu teknikte sunucuda bulunan sanal konaktaki(vhosts) uygulamaya ulaşmak için gerçekleştirilebilinecek başlıca yöntemler; kapsamdaki IP adreslerine ilgili alan adı ile istek gerçekleştirmek ve alan adına bağlı IP geçmişini sorgulamaktır. Eğer DNS kaydı kaldırılmışsa ve alan adı hâlâ aynı IP adresine bağlı sunucudaysa o zaman bu yöntemler kullanılabilir. 

IP alt ağının /24 olduğu varsayıldığında 94.1.1.0/24 adreslerinin hepsine IP adresi bulunamayan alan adı ile gidilirse ilk yöntem başarıyla gerçekleştirilmiş olunur. Bu sorgular `curl` uygulamasıyla etkileşimli olan ufak bir betik yardımıyla gerçekleştirilebilir.

IP geçmişi ile alakalı yakın zamanda `Hackerone` sitesinde yayınlanmış bir [bulgudaki](https://hackerone.com/reports/314594) web adresini incelenebilir.

##### Zafiyetin açıklaması:

Bulguyu bulan kişi, raporda bahsi geçen [addlive.io](http://addlive.io) web adresine alan adı ele geçirme
(domain-takeover) denemesi gerçekleştirmiş ve başarılı olmuştur. Amazon bu alan adını tanıdığı için ilgili sunucuyu aramış ama bulamamıştır bu durumda saldırgan bu durumu kendi lehine çevirmiş ve o sunucu boşluğunu Amazon'dan açtığı bir [bucket](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html) ile doldurmuştur, böylelikle alan adına gerçekleştiren istekleri karşılayacak bir Amazon buketi olmuştur. Daha sonrasında ilgili zafiyet alan adı tarafındaki Amazon DNS kaydı kaldırıldıktan sonra çözüme ulaşmıştır, sorun yok gibi gözüküyor. Ama biz basit bir kayıt ile Amazon'daki uygulamaya halen erişebilmekteyiz zira Amazon tarafındaki uygulama, "applive.io" alan adı tarafından çağırılınca gelecek şekilde belirlenmiştir ve bu kayıt kaldırılmamıştır.

http://www.viewdns.info/iphistory/?domain=addlive.io

![addliveio-IP-gecmisi](../eg/IP-gecmisi-addlive.png)

addlive.io adresinin geçmişine baktığımızda Amazon'a ait bir çok IP adres kaydı görmekteyiz, bu IP'lerden herhangi birine addlive.io alan ismiyle gittiğimiz zaman karşımıza önceki uygulamanın geldiğini göreceğiz.

![addliveio-host-istegi](../eg/addlive-host-istegi.png)

Host-IP sorgulamasını HTTP Host başlığı ve hosts dosyasına girilen kayıt ile gerçekleştirebilirsiniz.

Yukarıdaki uygulama tam olarak başlığı karşılayamıyor olabilir ama bunun da bir örnek olacağını düşündüğüm için yazdım.

### Kelime Listesi ile Alt Alan Adı Keşifleri
Alt alan adlarını araştırırken önemli adımlardan biri seçilen kelime listesi ile DNS sunucularına yönelik gerçekleştirilen deneme-bulma keşifleridir. Temel mantık alt alan adının varlığını sorgulamaktır.

Deneme-bulma tekniğinde temel nokta elinizdeki sözcük listesinin iyi olmasıdır. Github'daki bazı [depo](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)larda böyle kelime listeleri bulunmaktadır. 

<b>Ufak bir not:</b> Eğer hedef firmadaki tüm alt alan adları tek bir IP adresini çözüyorsa(wildcard) o zaman DNS araçlarının bu durum için yazılmış özelliklerine bakılması gerekir, bu özellikler genelde wildcard alt alan adlarını bulduğunda kenara atar ve kullanıcıya göstermez. Ayrıca bütün DNS isteklerini topluca gerçekleştirmek için `fierce`, `dnsrecon` gibi araçlar kullanılabilir, özellikle `fierce` aracı verilen DNS sunucu ve IP alt ağında sorgulama gerçekleştirme gibi konularda oldukça iyidir.

Örnek bir alt alan adı taraması aşağıdaki gibi gözükmektedir:

![dnsrecon-altalanadi](../eg/altalanadi-dnsrecon.png)

Kelime listesi ile alt adı alan araştırmalarındaki diğer bir adımda yine yukarıda bahsedilmiş olan alt alan adlarındaki alt alanları aramaktır. `b.intel.com` alan adına tekrar bir tarama gerçekleştirildiğinde `h.b.intel.com` gibi sonuçlara ulaşılması mümkün. Alt alan adlarında 2. bir alt alan adı olma olasılığı SSL sertifikalarına bakarak anlaşılabilir genelde `*.a.intel.com` gibi bir tanımlama yapılır ve bu tanımlama da boşuna yapılmaz.

### Akıllı Alt Alan Adı Keşifleri
Kelime listesiyle alt alan adı araştırmalarında, kurum tarafından özelleştirilmiş alt alanları bulunmak istenirse bu başlıkta belirtildiği gibi akıllı taramalar gerçekleştirilebilir. 

Akıllı taramalardan kasıt kuruma veya kurumun bulunduğu sektöre özel oluşturulmuş kelime listeleri, deneme-bulma tekniği ile kullanılarak alt alan adlarına yönelik yapılan keşiftir. Kullanılacak kelime listeleri; kurumun bulunduğu sektör hakkında bilgi toplamak, kuruma ait alan adları, çalışmalar sırasında kuruma göre özelleştirilmiş bir alan adı elde edildiyse bu alan adı mantığına bakıp yeni olasılıkları takip etmek, kurumun web sitesinde bulunan kelimeler ve yine sektöründe bulunan kelimeleri toplayarak elde edilebilir. 

Ek olarak, bilgi toplama tekniklerinin dışında bilgiye erişme teknikleri de bulunmaktadır. Bunlardan en bilindiği web sitenin eski bir tarihteki sürümüne ulaşıp araştırmaktır. Web sitedeki kelimeler/erişim noktaları/dosya isimleri zamanla değiştirilebilir, değiştirilen kelimeleri takip etmek için https://web.archive.org gibi web sitelerini kullanabilir ve kelimeleri geliştirme çalışmalarında bu yöntemle ilerleyebilirsiniz.

#### Kelime Listesi Oluşturma
Ufak bir google araştırması ile bu sektöre özel hazırlanmış kelime listelerine ulaşılabilir. Ayrıca web sitelerin içerisinde bulunan kelimeler de işe yarayabilirler. O yüzden web sitelerin içindeki kelimeleri ayıklanması ve araştırmalar için kenara atılması ileri çalışmalar için gereklidir. 

Bu işlemleri otomatik hale getirmek için tercih ettiğim 2 araç: 

- CeWL ile web sitesi içerisindeki kelimeleri toplanılması.
CeWL uygulaması belirtilen web site içerisindeki kelimeleri toplamakta ve isteğe göre dosyaya kayıt etmektedir. Genelde parola saldırıları için kullanılıyor ama ben bu yöntem için de kullanmanızı öneririm. [https://tools.kali.org/password-attacks/cewl]

- Web sitesinde pasif olarak gezinirken arkada çalışan bir uygulama ile kelimeleri takip etmek.
Bir diğeri ise siz web site üzerinde keşif çalışmalarınızı gerçekleştirirken kelime listesini kendi başına oluşturan bir eklenti oluyor. BurpSuite proxy uygulamasında böyle bir eklenti bulunuyor, pasif olarak sitede gezinirken alınan HTTP cevaplarını kurcalayıp içerisinden kelimeleri ayıklıyor ve kayıt ediyor. Web site üzerindeki gizli dizinleri bulmak için tercih ediliyor ama bu yöntem için de kullanılabileceğini düşünüyorum, yani 2 yöntem için de işe yarayacaktır. 
Eklentiler: BurpSmartBuster, Site Map Extractor

Kurumun faaliyet gösterdiği sektörün finans olduğunu varsayalım. Finans ile ilgili bütün kelimeleri kenarda tutulması araştırmalar açısından iyi bir yaklaşım olacaktır. Önceden finans sektöründe bir çalışma gerçekleştirilmişse veya ileride gerçekleştirilecekse bulunan alan adlarının bir kenarda tutulması, ileri çalışmalarda işe yarayabilir. 

#### Hedef Kurumun Alt Alan Adı Söz Dizimi	
Çoğu kurumda son kullanıcıların doğrudan erişimi gerekmeyen(API,web servisleri vb.) sunuculardaki alt alan adları belirlenirken belli bir söz dizimi kullanılır. Genelde yerel sunucularda bu tip alan adlarına sık rastlanır, yerel ağdaki yöneticiler/yazılımcılar/güvenlikçiler açısından oturmuş bir alan adı yapısı diğerlerine nazaran çok daha anlaşılabilir ve kontrol edilebilir oluyor elbette başka nedenleri de olabilir.

Bu söz dizimlerine bazı örnekler vererek durumu açıklığa kavuşturalım, kurumumuzun adı `teslanatif` ve sektörü de `enerji` olsun ve bu kuruma göre potansiyel söz dizimlerine:

~~~
[kurumadı][sunucutürü][prod/test].teslanatif.com.tr - [teslanatif][webservice][prod]
[Kurum adı baş harfi ve sektör baş harfi][sunucutürü][prod/test].teslanatif.com.tr - [te][api][test]
[sunucutürü][numara].teslanatif.com.tr - [exchange][20].teslanatif.com.tr
~~~

gibi örnekler verilebilir. Bunu kurum bazlı olarak analiz etmelisiniz çok farklı söz dizimleri çıkabilir. Bizim hedefimizin 1.seçeneği kullanıyor olduğunu düşünelim, bunu nasıl öğrendik? En basitinden örnek vermek gerekirse ana sayfası `teslanatifwebservice.teslanatif.com.tr` iletişime geçiyor olsun. Söz dizimini biliyoruz, geriye potansiyel servisleri yoklayacak kelime listesini çıkartmak işlemi kalıyor.

En başında kurum adı, ardına servis adı ve hemen sonrasında prod/test sunucu olduğunu yazacağız burada kontrol etmemiz gereken yer servis adı kısmı. Web sitesinden topladığımız kelimeleri burada da kullanabiliriz. Sözcük listesini hazır etmek için `teslanatif[olustur][olustur2].teslanatif.com.tr` kelimesindeki `[olustur]` yazan yere kelime listesini, `[olustur2]` yazan yere ise test/prod kelimelerini yazarak bir liste oluşturabilir ve taramamızı gerçekleştirebiliriz.

### SSL Sertifikalarının Araştırılması
SSL sertifikalarında, sertifikanın ait olduğu yer hakkında bir çok değişken bulunuyor. Hangi kuruma ait? Hangi web sitesine işaretlenmiş? gibi bilgilerle farklı alan adlarına/alt alan adlarına ulaşmamız mümkün. 

SSL sertifikaları genelde kurumun web sitelerine özel olarak imzalanmaktadır. Bu sertifikalarda `Organization(O)`, `Common Name(CN)` ve `Subject Alternative Name(SAN)` gibi denetimci açısından yararlı bazı başlıklar bulunmaktadır. Her biri sertifikanın kuruma ve ilgili alan adına ait olduğunu doğrulamak için önemlidir. Ancak yanlış veya mecburi yapılandırmalar sonucunda bir çok alan adı ve alt alan adı ortalıkta olmaktadır.

deneme.com sunucusu ve alt alan adları için atanan SSL sertifikaları genelde bütün alt alan adlarını kapsayacak şekilde yazılır. Bu atama SSL sertifikasının deneme.com alan adının altındaki bütün alt alan adlarına güvenebilirim şeklinde anlamlandırılabilir, eğer bu durum sağlanmazsa SSL sertifikasının hatalı yapılandırıldığı anlamına gelir. Benim karşılaştığım hatalar arasında, kuruma ait başka bir alan adının, ana alan adına işaretlenmesi ve yerel sertifikaları dış sunuculara işaretlemesi bulunmaktadır.

Biz alt alan adı araştırmalarımızı yukarıda saydığım `Organization(O)`, `Common Name(CN)` ve `Subject Alternative Name(SAN)` değikenleri arasında yapacağız. İlgili bilgileri öğrenmek için Linux'ta bulunan `openssl` uygulamasını, çevrimiçi olarakta [Shodan](https://www.shodan.io/), [Censys](https://censys.io/) gibi uygulamaları kullanabiliriz. 

Örnek olarak intel.com adresinin sertifika bilgilerine göz atalım.

~~~
openssl s_client -showcerts -connect intel.com:443 < /dev/null | openssl x509 -text

Certificate:
    Data:
        ...
        Subject: C=US, ST=California, L=Santa Clara, O=Intel Corporation, OU=Information Technology, CN=*.intel.com
        ...
        X509v3 extensions:
            X509v3 Subject Alternative Name: 
                DNS:*.intel.com, DNS:intel.com
           ...
~~~

İlgili yerleri kesip aldıktan sonra elimizde böyle bir veri kümesi kalıyor. Burada şu anlık işimize yarayacak bilgileri `SAN`, `O` ve `CN` belirteçlerinden elde edebiliriz. `CN` ve `SAN` birbiriyle neredeyse aynı işlevlere sahiptir ancak biz iki çıktıyı da alalım yine de.

~~~
Subject: O=Intel Corporation, CN=*.intel.com
X509v3 Subject Alternative Name: DNS:*.intel.com, DNS:intel.com
~~~

Bu araştırmaları sadece tek web site üzerinden gerçekleştirdik. Ama açık kaynaklar yine unutulmamalı, bu araştırmaları yine açık kaynak yoluyla gerçekleştirerek daha geniş bir kapsama erişebiliriz. SSL üzerinde bulunan neredeyse bütün bilgileri gruplanmış olarak açık kaynak üzerinden elde edebiliriz. Burada istediğimiz belli bir kuruma ait SSL sertifikalarını listelemek ve bilgi toplamak olduğu için özelleştirilmiş isimleri süzgeçten geçirerek listelemeliyiz. Bu işlemi [crt.sh](https://crt.sh), [Censys](https://censys.io/) ve Google'ın sertifikaları izleyip kayıt ettiği [servis](https://transparencyreport.google.com/https/certificates) ile gerçekleştirelim. Bu işlemin mantığı belirteçleri açık kaynak servisleri aracılığı ile sorgulayarak ilgili belirteçleri aynı olan sertifikaları bulmaktır.

crt.sh sayfasında "Advanced Search" seçeneğine tıklayarak arama tipini seçebilirsiniz. Öncelikle kurum adına alınmış sertifikaları listelemek yani "OrganizationName" belirteciyle sorgulama yapmak o kurum adına alınmış sertifikaları listeleyecektir.

![sertifika-sorgulama](../eg/crtsh-o-sertifika-sorgulama.png)

Aramayı yaptığınız takdirde karşınıza o kuruma ait sertifikalar gelecektir. Sertifikaların detaylarına "crt.sh ID" altındaki numaralar ile ulaşabilirsiniz. Detaylar kısmındaki SAN veya CN belirteçleriyle alan adı/alt alan adını elde edebilirsiniz. 

![sertifika-listeleme](../eg/crtsh-o-sertifika-listeleme.png)

Censys uygulaması da oldukça detaylı sonuçlar çıkartabilmektedir. Doğrudan aşağıdaki gibi bir arama yaptığınızda içerisinde intel.com geçen her sertifikayı karşınıza getirir. 

https://censys.io/certificates?q=intel.com

![sertifika-listeleme-censys](../eg/censys-sertifika-listeleme.png)

crt.sh adresinde Censys içerisinde de arama seçeneği var isterseniz doğrudan onu da kullanabilirsiniz.

Google'ın sağladığı servisin ise şöyle bir ekranı var:

![sertifika-sorgulama-google](../eg/google-sertifika-sorgulama.png)

Alttaki seçenekleri mutlaka işaretlemeliyiz. İlk seçenek olan `Include certificates that have expired` süresi bitmiş sertifikaları da bul ikinci seçenek `Include subdomains` ise alt alan adlarını da dahil et anlamına geliyor.

### E-Posta Başlıklarındaki Alt Alan Adları
MX kayıtlarının yanlış yönlendirildiği durumlarda MX sorgularında kurumun spam sunucusunu görebiliriz. Bu durumlarda kurum e-posta adresinden size iletilen e-postanın başlık bilgilerine bakarak e-posta sunucusunun alan adını öğrenebilirsiniz. E-posta başlıklarında `Received` kelimesini aratarak hedef kurumun MX sunucusunun alan adı bilgisini elde edebilirsiniz.

~~~
Received: from mail1.intel.com (mail1.intel.com [12.12.12.12])
~~~

### Web Sitelerindeki Adreslerin Bulunması
Web sitelerinde farklı web adreslerinin izlerine rastlamak mümkün. Bunun sebeplerinden bazıları dış bir Javascript dosyası çağırmak, API ile haberleşmek veya resim/müzik/video dosyaları çağırmak olabilir. Dış kaynakları takip etmek kuruma ait yeni alan adlarını bulmak konusunda oldukça yardımcı olabilir.

Şöyle bir durum da var ki dış kaynaklarda da kurumun web sitelerine veya uygulamalarına rastlamak mümkündür. 

Bu bölümde hedef web siteler ile bağlantısı olan uygulamaları ve web siteleri belirleme yöntemleri üzerinde çalışacağız.

#### Eklentilerin İzlerini Takip Etmek
Bazı kurumlar web sitelerini analiz etmek, internet üzerindeki sayfaları(Facebook,twitter) web sitelerine bağlamak amacıyla facebook, twitter, google gibi internet siteleriyle etkileşim halinde olurlar. Her etkileşimi belirtebilmek için bir ID değeri bulunur bu ID değerleri kurumun web sitelerinde istemci taraflı tutulur ve arama motorları tarafından aramalarda listelenmek üzere kayda alınır. İlgili ID değerlerinin platform bazlı izini sürerek bu ID değerlerini paylaşan yeni alan adlarını gün yüzüne çıkartabiliriz. Aşağıda bir kaç örnek eklentiyi yazmaya çalıştım.

Google'ın performans analiz sistemi olan Google Analytics bir çok web sitesinde kullanılmaktadır. Bu servisi kurmak için js dosyasından ziyade bir de kurum ile ilişkili izleme kodu değeri web sitesinin her sayfasına yerleştirilir. Kurumlar izlemek istediği her internet sitesine bu kodu eklerler. Genelde bu kod gizlenen alan adlarından ziyade insanların erişimine açık olan web sitelerine yazılır yani bu yöntem ile elde edebileceğimiz web siteleri çoğunlukla iştirakler ya da kampanya sayfaları olur. 

Bu izleme kodunu paylaşan siteleri öğrenmek için yine açık kaynaktan faydalanacağız. Aynı izleme değerini kullanan web sitelerini aşağıdaki web adresleriyle listeleyebiliriz.

http://moonsearch.com/analytics/

Ayrıca kurumlar Facebook, Twitter gibi paylaşım sayfalarını yine web sitelerinin etiketlerine eklerler, örneğin intel.com adresinde fb:admins olarak eklenen değeri Google ile arattığımızda karşımıza aynı değeri paylaşan farklı web siteleri çıkmaktadır.

~~~
<meta property="fb:admins" content="267985180046218"/
~~~

![fb-admins-google](../eg/intel-fb-admins.png)

#### HTTP Başlıklarındaki Alan Adları
İlgili web sitesi haricindeki dış kaynaklar ile veri alışverişlerinin gerçekleşmesi için HTTP protokolünde CORS ve CSP başlıkları kullanılarak haberleşme sağlanacak kaynaklara izin verilmesi gerekmektedir, aksi takdirde veri alışverişi gerçekleşemez. Temelde iki HTTP başlığı da güvensiz kaynaklardan gerçekleştirilen izinsiz iletişim isteklerini engellemek amacıyla kullanılır. Kurumlar bu başlıklara sadece güvendikleri ve iletişim halinde olması gereken kaynakları eklerler, bu kaynaklar kuruma ait tarayıcı eklentileri, API adresleri, statik dosya adresleri vs. olabilir. Bu yazının konusunu ele alıp ilgili başlık bilgilerine baktığımızda bu başlıklardan hedef kuruma ait veya hedef kurumun bağlantıda olduğu web sitelerini çıkartabileceğimizi düşünebiliriz.

Örnek olarak Facebook'un başlık bilgilerine göz atalım:

~~~
content-security-policy:default-src * data: blob:;script-src *.facebook.com *.fbcdn.net *.facebook.net *.google-analytics.com *.virtualearth.net *.google.com 127.0.0.1:* *.spotilocal.com:* 'unsafe-inline' 'unsafe-eval' fbstatic-a.akamaihd.net fbcdn-static-b-a.akamaihd.net *.atlassolutions.com blob: data: 'self';style-src data: blob: 'unsafe-inline' *;connect-src *.facebook.com facebook.com *.fbcdn.net *.facebook.net *.spotilocal.com:* *.akamaihd.net wss://*.facebook.com:* https://fb.scanandcleanlocal.com:* *.atlassolutions.com attachment.fbsbx.com ws://localhost:* blob: *.cdninstagram.com 'self' chrome-extension://boadgeojelhgndaghljhdicfkmllpafd chrome-extension://dliochdbjfkdbacpmhlcpmleaejidimm;
~~~

Görüldüğü gibi bir çok dış kaynak mevcut. Bu kaynakların hangilerinin Facebook'a ait olduğunu bulmak veya bu kaynakların neden Facebook ile iletişim halinde olduğunu araştırmak bize kapsam genişletme konusunda yardımcı olacaktır.

#### Kaynakların İncelenmesi
Hedef web sitesinin haberleştiği ve haberleşmediği ama kaynağında tutulan dış kaynakların takibi neticesinde kuruma ait farklı alan adlarını elde edebiliriz. Bu bölümde inceleyebileceğimiz kaynakları inceleyeceğiz.

Bu kaynaklar web sitesinde bulunan HTML ve JS kodlarıdır. HTML ve JS dosyalarının içinde birden fazla alan adına rastlayabiliriz o yüzden bu kaynakları mutlaka incelememiz gerekmekte. İncelemekten kastım ilgili kaynaklara erişim sağlayıp farklı URL'lerin takibi olduğundan dolayı herhangi bir tekniği bulunmamaktadır.

#### Destek Veren Firmaların İncelenmesi
Kuruma destek veren yazılımcı, sunucu veya sosyal firmalarının referanslarının incelenmesi hedef kurumun iştiraklerini bulma konusunda yardımcı olabilir. 

Keşif aşamasında dikkate alınmaya değer bir şey de bazı yazılım firmalarının yaptıkları işlerin test sürümlerini kendi sunucularında tutmalarıdır. Yani uygulamayı test sürümünde karşı firmaya sunduktan sonra alt alan adı olarak kendi sunucularında test sürümlerini tutarlar. Bu yazılımlara erişmek için yazılım firmasının alt alan adı söz dizimini bilmek gerekebilir, ya da sadece "kurumadi.yazilimfirmasi.com" şeklinde de erişim sağlayabilir ve test paneline giriş yapabiliriz. Teslanatif firmasını bir daha ele alalım, diyelim ki Teslanatif yazılımcı firması olan Tetech firması hizmet verdiği kurumları kendi alt alan adında tutuyor bu durumda teslanatif.tetech.com sorgulamasıyla teslanatif.com uygulamasının test sürümüne erişim sağlayabiliriz.

Alan adlarının araştırılmasında kullanılan teknikleri dilim döndüğünce anlatmaya çalıştım, umarım faydalı olmuştur. 

Sevgiler.

### Yardımcı Çevrimiçi Uygulamalar:
https://punkspider.org<br>
https://ivre.rocks<br>
https://www.zoomeye.org<br>
https://www.censys.io<br>
https://www.shodan.io<br>
http://www.dogpile.com/<br>
https://www.virustotal.com/#/home/url<br>
https://github.com/k4ch0w/pwnback<br>

### Kaynaklar:
https://digi.ninja/projects/zonetransferme.php<br>
https://evren.ninja/recon-is-everywhere.html<br>
https://www.youtube.com/watch?v=C4ZHAdI8o1w<br>
https://www.youtube.com/watch?v=1Kg0_53ZEq8<br>
https://www.youtube.com/watch?v=VtFuAH19Qz0<br>
https://www.youtube.com/watch?v=e_Gq99CKAys<br>
http://www.diaryofinjector.com/2014/10/hedef-siteye-nasl-erisilir.html<br>
http://pastebin.com/raw/cRYvK4jb<br>
https://pentestlab.blog/2012/11/13/dns-reconnaissance-dnsrecon/<br>
