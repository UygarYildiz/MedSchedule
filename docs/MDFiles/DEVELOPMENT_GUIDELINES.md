# MedSchedule Geliştirme Kılavuzu

Bu doküman, MedSchedule projesinin geliştirilmesi sırasında izlenecek standartları, en iyi uygulamaları ve geliştirme süreçlerini tanımlar. Yapay zeka, projeyi geliştirirken bu kılavuzu takip etmelidir.

## Kod Standartları

### Python Kod Standartları

1. **PEP 8 Uyumluluğu**
   - [PEP 8](https://www.python.org/dev/peps/pep-0008/) stil kılavuzunu takip et
   - Satır uzunluğu maksimum 100 karakter olmalı
   - İçe aktarmaları alfabetik olarak sırala
   - Fonksiyon ve değişken isimleri için `snake_case` kullan
   - Sınıf isimleri için `PascalCase` kullan

2. **Dokümantasyon**
   - Her modül, sınıf ve fonksiyon için docstring ekle
   - [Google stil docstring](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html) formatını kullan
   - Karmaşık kod bloklarını açıklayan yorumlar ekle
   - Tüm parametreleri, dönüş değerlerini ve istisnaları dokümante et

3. **Tip Belirteçleri**
   - Fonksiyon parametreleri ve dönüş değerleri için tip belirteçleri kullan
   - Karmaşık veri yapıları için tip belirteçleri ekle
   - Uygun olduğunda `Optional`, `Union`, `List`, `Dict` gibi tip belirteçlerini kullan

4. **Kod Organizasyonu**
   - İlgili fonksiyonları ve sınıfları mantıksal olarak grupla
   - Sınıf metotlarını mantıksal bir düzende sırala (örn. init, property, public, private)
   - Uzun fonksiyonları daha küçük, yeniden kullanılabilir fonksiyonlara böl
   - Tekrarlanan kodu ortak yardımcı fonksiyonlara çıkar

### JavaScript/React Kod Standartları

1. **ESLint ve Prettier Uyumluluğu**
   - Projenin ESLint ve Prettier yapılandırmalarına uy
   - Satır uzunluğu maksimum 100 karakter olmalı
   - Tek tırnak yerine çift tırnak kullan
   - Noktalı virgülleri kullan

2. **Bileşen Yapısı**
   - Fonksiyonel bileşenler ve React Hook'ları kullan
   - Bileşenleri mantıksal olarak küçük, yeniden kullanılabilir parçalara böl
   - Prop'lar için PropTypes veya TypeScript tip tanımları ekle
   - Bileşen dosyalarını tek sorumluluk prensibine göre organize et

3. **State Yönetimi**
   - Redux için düzenli aksiyon ve reducer yapısı kullan
   - Yerel state için React Hook'larını kullan
   - Context API'yi global state için uygun şekilde kullan
   - Asenkron işlemler için Redux Thunk veya Redux Saga kullan

4. **Stil Yaklaşımı**
   - CSS-in-JS yaklaşımını kullan (styled-components veya emotion)
   - Tutarlı tema değişkenleri kullan
   - Responsive tasarım için medya sorguları ekle
   - Erişilebilirlik standartlarına uy

## Veritabanı Standartları

1. **Şema Tasarımı**
   - Tablo ve sütun isimleri için `snake_case` kullan
   - İlişkisel bütünlüğü sağlamak için yabancı anahtar kısıtlamaları ekle
   - Uygun indeksleme stratejisi uygula
   - Normalizasyon kurallarını takip et

2. **Migrasyon Yönetimi**
   - Her şema değişikliği için migrasyon dosyaları oluştur
   - Migrasyonları geri alınabilir şekilde tasarla
   - Migrasyon dosyalarını anlamlı şekilde adlandır
   - Migrasyon testleri ekle

3. **Veritabanı Erişimi**
   - ORM (SQLAlchemy) kullan
   - Ham SQL sorgularından kaçın
   - Veritabanı işlemleri için transaction'ları kullan
   - N+1 sorgu probleminden kaçınmak için eager loading kullan

4. **Performans Optimizasyonu**
   - Karmaşık sorgular için indeksler ekle
   - Büyük veri kümeleri için sayfalama uygula
   - Sık kullanılan sorgular için önbellek kullan
   - Sorgu performansını düzenli olarak analiz et

## API Tasarım Standartları

1. **RESTful API Prensipleri**
   - HTTP metotlarını doğru şekilde kullan (GET, POST, PUT, DELETE)
   - URL'leri kaynak odaklı tasarla
   - Uygun HTTP durum kodlarını kullan
   - API versiyonlamasını uygula

2. **Güvenlik**
   - Tüm API endpoint'leri için kimlik doğrulama ve yetkilendirme uygula
   - Rate limiting ekle
   - CORS yapılandırmasını güvenli şekilde ayarla
   - Hassas verileri API yanıtlarından filtrele

3. **Dokümantasyon**
   - OpenAPI (Swagger) ile API dokümantasyonu oluştur
   - Her endpoint için örnek istek ve yanıtlar ekle
   - Tüm parametreleri ve yanıt alanlarını dokümante et
   - Hata yanıtlarını dokümante et

4. **Yanıt Formatı**
   - Tutarlı JSON yanıt yapısı kullan
   - Hata yanıtları için standart format belirle
   - Sayfalama, filtreleme ve sıralama için tutarlı parametreler kullan
   - İsteğe bağlı olarak HATEOAS prensiplerini uygula

## Test Standartları

1. **Birim Testleri**
   - Her modül için kapsamlı birim testleri yaz
   - Mock ve stub'ları uygun şekilde kullan
   - Test senaryolarını anlamlı şekilde adlandır
   - Edge case'leri test et

2. **Entegrasyon Testleri**
   - Bileşenler arası etkileşimleri test et
   - API endpoint'lerini test et
   - Veritabanı etkileşimlerini test et
   - Asenkron işlemleri test et

3. **Uçtan Uca Testleri**
   - Kullanıcı senaryolarını simüle eden testler yaz
   - Cypress veya Selenium gibi araçları kullan
   - Kritik iş akışlarını test et
   - Farklı tarayıcı ve cihazlarda test et

4. **Performans Testleri**
   - API endpoint'leri için yük testleri yap
   - Veritabanı sorguları için performans testleri yap
   - Ölçeklenebilirlik testleri yap
   - Bellek kullanımını ve sızıntılarını test et

## Güvenlik Standartları

1. **Kimlik Doğrulama ve Yetkilendirme**
   - JWT veya OAuth 2.0 tabanlı kimlik doğrulama kullan
   - Rol tabanlı erişim kontrolü uygula
   - Güçlü parola politikaları uygula
   - Çok faktörlü kimlik doğrulama seçeneği ekle

2. **Veri Güvenliği**
   - Hassas verileri şifrele (hem iletimde hem de saklamada)
   - Kişisel sağlık verilerini HIPAA standartlarına uygun şekilde işle
   - Veritabanı erişimini kısıtla ve izle
   - Düzenli güvenlik denetimleri yap

3. **Güvenli Kodlama Uygulamaları**
   - Input validasyonu uygula
   - SQL enjeksiyonu, XSS ve CSRF saldırılarına karşı koruma ekle
   - Bağımlılıkları düzenli olarak güncelle ve güvenlik açıklarını tara
   - Güvenlik açıklarını raporlama mekanizması ekle

4. **Loglama ve İzleme**
   - Güvenlik olaylarını logla
   - Şüpheli etkinlikleri tespit etmek için izleme mekanizmaları ekle
   - Log'ları güvenli şekilde sakla ve yedekle
   - Log'larda hassas verileri maskeleme

## Geliştirme İş Akışı

1. **Git İş Akışı**
   - Feature branch iş akışını kullan
   - Commit mesajları için [Conventional Commits](https://www.conventionalcommits.org/) standardını kullan
   - Pull request'ler için şablon kullan
   - Düzenli olarak main/master branch'i ile senkronize et

2. **Code Review Süreci**
   - Her pull request için en az bir code review gerekli
   - Code review kontrol listesi kullan
   - Otomatik kod kalite kontrolleri ekle
   - Yapıcı geri bildirim ver

3. **CI/CD Pipeline**
   - Her commit için otomatik testleri çalıştır
   - Kod kalitesi ve test kapsamı kontrolleri ekle
   - Otomatik dağıtım pipeline'ı kur
   - Dağıtım öncesi onay mekanizması ekle

4. **Sürüm Yönetimi**
   - [Semantic Versioning](https://semver.org/) kullan
   - Her sürüm için değişiklik günlüğü tut
   - Sürüm etiketleri oluştur
   - Geriye dönük uyumluluğu koru

## Performans Optimizasyon Kılavuzu

1. **Backend Performans**
   - Veritabanı sorgularını optimize et
   - N+1 sorgu probleminden kaçın
   - Önbellek stratejisi uygula (Redis veya Memcached)
   - Asenkron işleme için kuyruk sistemleri kullan (Celery veya RabbitMQ)

2. **Frontend Performans**
   - Kod bölme ve lazy loading uygula
   - Statik varlıkları optimize et ve CDN kullan
   - Gereksiz yeniden render'ları önle
   - Performans metrikleri izle (Core Web Vitals)

3. **API Performans**
   - Gereksiz veri transferini önlemek için GraphQL veya özel endpoint'ler kullan
   - Sayfalama, filtreleme ve sıralama parametreleri ekle
   - API yanıtlarını önbellekle
   - Rate limiting ve throttling uygula

4. **Algoritma Optimizasyonu**
   - Çizelgeleme algoritmasının performansını optimize et
   - Tahmin modellerinin çalışma süresini minimize et
   - Bellek kullanımını optimize et
   - Paralelleştirme ve çoklu işleme teknikleri kullan

## Dokümantasyon Standartları

1. **Kod Dokümantasyonu**
   - Her modül, sınıf ve fonksiyon için docstring ekle
   - Karmaşık algoritmaları açıkla
   - Örnekler ve kullanım senaryoları ekle
   - Kısıtlamaları ve bilinen sorunları belirt

2. **API Dokümantasyonu**
   - OpenAPI (Swagger) ile API dokümantasyonu oluştur
   - Her endpoint için örnek istek ve yanıtlar ekle
   - Hata kodlarını ve anlamlarını dokümante et
   - Postman koleksiyonu oluştur

3. **Kullanıcı Dokümantasyonu**
   - Farklı kullanıcı rolleri için ayrı kılavuzlar oluştur
   - Ekran görüntüleri ve videolar ekle
   - Adım adım talimatlar sağla
   - SSS bölümü ekle

4. **Teknik Dokümantasyon**
   - Sistem mimarisini dokümante et
   - Kurulum ve yapılandırma talimatları ekle
   - Bağımlılıkları ve sistem gereksinimlerini belirt
   - Sorun giderme kılavuzu ekle

## Optimizasyon Algoritmaları Kılavuzu

1. **Çizelgeleme Optimizasyonu**
   - Kısıt programlama (Constraint Programming) yaklaşımını kullan
   - Genetik algoritma veya benzetimli tavlama gibi meta-sezgisel yöntemleri değerlendir
   - Çok amaçlı optimizasyon için Pareto optimizasyonu uygula
   - Gerçek zamanlı yeniden optimizasyon için etkin stratejiler geliştir

2. **Kısıt Modelleme**
   - Sert kısıtları (hard constraints) ve yumuşak kısıtları (soft constraints) ayır
   - Personel tercihleri için ağırlıklı kısıtlar kullan
   - Yasal çalışma saati kısıtlarını modelle
   - Departman ve rol bazlı kısıtları tanımla

3. **Çözüm Değerlendirme**
   - Çizelge kalitesini ölçmek için çok boyutlu metrikler tanımla
   - Personel memnuniyeti, iş yükü dengesi ve kaynak kullanımı gibi faktörleri değerlendir
   - Çözüm kalitesini görselleştirme araçları geliştir
   - Farklı çözümleri karşılaştırma mekanizmaları ekle

4. **Performans İyileştirme**
   - Büyük problem uzaylarını etkili şekilde araştırmak için sezgisel yöntemler kullan
   - Paralel hesaplama teknikleri uygula
   - Artımlı optimizasyon stratejileri geliştir
   - Önceden hesaplanmış çözümleri önbelleğe al

## Makine Öğrenmesi Modelleri Kılavuzu

1. **Hasta Akışı Tahmin Modeli**
   - Zaman serisi analizi için ARIMA, Prophet veya LSTM modelleri kullan
   - Mevsimsellik, trend ve tatil etkilerini modelle
   - Tahmin doğruluğunu değerlendirmek için RMSE, MAE ve MAPE metriklerini kullan
   - Anormal durumları tespit etmek için anomali algılama ekle

2. **İş Yükü Tahmin Modeli**
   - Hasta akışı ve iş yükü arasındaki ilişkiyi modellemek için regresyon modelleri kullan
   - Departman ve rol bazlı iş yükü farklılıklarını dikkate al
   - Personel yetkinliklerini ve deneyimini modele dahil et
   - Tahmin belirsizliğini kantifiye et ve güven aralıkları sağla

3. **Model Eğitimi ve Değerlendirme**
   - Çapraz doğrulama ile model performansını değerlendir
   - Hiperparametre optimizasyonu için grid search veya Bayesian optimizasyon kullan
   - Overfitting'i önlemek için regularizasyon teknikleri uygula
   - Model yorumlanabilirliği için özellik önem analizi ekle

4. **Model Dağıtımı ve İzleme**
   - Modelleri servisleştirmek için Flask veya FastAPI kullan
   - Model versiyonlama ve A/B testing mekanizmaları ekle
   - Model performansını izleme ve yeniden eğitim tetikleme mekanizmaları ekle
   - Model çıktılarını açıklamak için SHAP veya LIME gibi araçlar kullan

## İlerleme Raporlama Standartları

Yapay zeka, proje geliştirme sürecinde düzenli ilerleme raporları oluşturmalıdır. Her ilerleme raporu aşağıdaki bölümleri içermelidir:

1. **Tamamlanan Görevler**
   - Geliştirilen modüller ve özellikler
   - Çözülen sorunlar
   - Yapılan iyileştirmeler
   - Oluşturulan testler

2. **Mevcut Durum**
   - Projenin genel durumu
   - Tamamlanma yüzdesi
   - Karşılaşılan zorluklar
   - Kritik yol analizi

3. **Sonraki Adımlar**
   - Planlanan görevler
   - Öncelikler
   - Tahmini tamamlanma süreleri
   - Gerekli kaynaklar

4. **Riskler ve Azaltma Stratejileri**
   - Tanımlanan riskler
   - Olası etkiler
   - Azaltma stratejileri
   - Beklenmedik durum planları

Her ilerleme raporu, proje klasöründe `progress_reports/` dizini altında tarih damgalı bir Markdown dosyası olarak saklanmalıdır.

## Sonuç

Bu geliştirme kılavuzu, MedSchedule projesinin yüksek kaliteli, bakımı kolay ve güvenli bir şekilde geliştirilmesi için standartları ve en iyi uygulamaları tanımlar. Yapay zeka, bu kılavuzu takip ederek, projenin tüm gereksinimlerini karşılayan ve modern yazılım geliştirme prensiplerine uygun bir sistem oluşturabilir.
