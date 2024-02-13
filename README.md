#  İş gereksinimi
Bir müşterimizin SAP HANA sisteminde bulunan bazı finansal ana verilerin üçüncü parti bir sistemde de bulunması gerekiyordu. 
Normalde bu veriler için OData servisleri geliştirecektik. Fakat üçüncü parti sistem, SAP'den API çağıramayacağını bildirdi.
Bu sebeple verilerin CSV dosyaları olarak üretilmesine ve uygulama sunucusunda bir dizine yazılmasına karar verildi.

## Mevcut tasarım
Verileri ilgili tablolardan okuyup CSV formatında belirtilen hedefe yazan bir program geliştirdim.
Programa yardımcı olmak için iki sınıf yazdım. Bunlar `ZCL_CSV_BUILDER` ve `ZCL_CSV_WRITER`'dır.

`ZCL_CSV_BUILDER` sınıfı aşağıdaki üyelere sahiptir:
- `-string header`
- `-string_table rows`
- `+set_header(any)` :information_source: _bu metot, private 'header'i set eder_
- `+append_row(any)` :information_source: _bu metot, structure tipinde gönderilen veriyi CSV formatına dönüştürüp, private 'rows'a bir satır olarak ekler_
- `+append_rows(any table)` :information_source: _bu metot, tablo tipinde gönderilen verideki her bir satır için 'append_row' metodunu çağırır_
- `+serialize():string` :information_source: _bu metot, 'header' ve 'rows' değişkenlerindeki bilgileri birleştirir bir string döndürür_

`ZCL_CSV_WRITER` sınıfı aşağıdaki üyelere sahiptir:
- `+save_file_on_local(filename,content)`
- `+save_file_on_server(filename,content)`

## Yeni tasarım
Yeni tasarımda iki arayüz var: `ZIF_CSV` ve `ZIF_FILE`.

`ZIF_CSV` aşağıdaki yöntemlere sahiptir:
- `+set_header(any)`
- `+append_row(any)`
- `+append_rows(any table)`
- `+serialize():string`

`ZIF_FILE` şimdilik aşağıdaki yöntemlere sahiptir:
- `+write(content)`

Yeni sınıflar aşağıdaki gibidir:
- `ZIF_CSV` arayüzünü kullanan `ZCL_CSV` isminde yeni bir sınıf var. Aslında bu hemen hemen `ZCSV_BUILDER`'ın aynısı. Sadece arayüz kullanan ve 'verb' yerine 'noun' isimli versiyonu.
- `ZCL_CSV_WRITER` sınıfı yerine ise `ZIF_FILE` arayüzünü kullanan iki tane yeni sınıf var: `ZCL_LOCAL_FILE` ve `ZCL_SERVER_FILE`. Bunlar `private` sınıflar.
- `ZCL_LOCAL_FILE` ve `ZCL_SERVER_FILE` sınıflarını kullanan yeni bir sınıf var: `ZCL_FILE`. Bu sınıfta bir `constructor` yöntem ve bu yöntemde bir `destination` parametresi var. `ZCL_FILE` sınıfı bir `factory` sınıf veya `singleton` bir sınıf olabilir. Sonuçta `ZCL_LOCAL_FILE` ve `ZCL_SERVER_FILE` sınıflarını kontrol edecek bir ana sınıftır.

Bu tasarımda program aşağıdakileri yapacaktır:
- Program verileri okuyacak ve `ZCL_CSV` sınıfı ile CSV formatında içerik üretecektir.
- Daha sonra `ZCL_CSV` sınıfının `SERIALIZE` metodunu çağıracak ve dosyaya yazılacak içeriği alacaktır.
- Seçim ekranındaki "local/server" seçimine göre `ZCL_FILE` sınıfını çağıracaktır. `ZCL_FILE`, verilen `destination` parametresine göre `ZCL_LOCAL_FILE` veya `ZCL_SERVER_FILE` sınıfını kullanacaktır.
- En sonunda `ZCL_FILE` sınıfının `WRITE` metodunu çağırarak dosyayı oluşturacaktır.

UML tasarımını da ekledim.
Yeni tasarımın doğruluğu konusunda görüşlerinizi, tavsiyelerinizi rica ederim.
