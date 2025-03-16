# MedSchedule: Akıllı Sağlık Personeli Çizelgeleme ve Vardiya Optimizasyon Sistemi

## Yapay Zeka Geliştirme Prompt'u

Bu doküman, MedSchedule projesinin yapay zeka tarafından geliştirilmesi için kapsamlı bir kılavuzdur. Aşağıdaki talimatları takip ederek, hastanelerde personel çizelgeleme ve iş yükü dengelemesi için optimize edilmiş bir sistem geliştireceksiniz.

### Proje Tanımı

MedSchedule, hastanelerde sağlık personelinin vardiya planlamasını, iş yükü dağılımını ve kaynak tahsisini optimize eden yapay zeka tabanlı bir platformdur. Sistem, hasta yoğunluğu tahminleri, personel yetkinlikleri, yasal çalışma süreleri ve personel tercihleri gibi çok sayıda faktörü dikkate alarak, hem hasta bakım kalitesini artıran hem de personel memnuniyetini yükselten optimum çalışma çizelgeleri oluşturur.

### Geliştirme Prensipleri

1. **Modüler Mimari**: Sistemi birbirinden bağımsız çalışabilen modüller halinde geliştir.
2. **Test Odaklı Geliştirme**: Her modül için kapsamlı test senaryoları oluştur.
3. **Dokümantasyon**: Kodun her bölümünü detaylı şekilde dokümante et.
4. **Ölçeklenebilirlik**: Sistem küçük ve büyük hastanelerde çalışabilecek şekilde tasarlanmalı.
5. **Kullanıcı Deneyimi**: Arayüz sezgisel ve kullanımı kolay olmalı.
6. **Güvenlik**: Personel ve hasta verilerinin güvenliği en üst düzeyde sağlanmalı.

### Teknik Gereksinimler

1. **Programlama Dili**: Python 3.9+
2. **Web Framework**: Flask veya FastAPI
3. **Veritabanı**: PostgreSQL
4. **Frontend**: React.js
5. **Optimizasyon Kütüphaneleri**: PuLP, OR-Tools
6. **Makine Öğrenmesi**: Scikit-learn, TensorFlow veya PyTorch
7. **Veri İşleme**: Pandas, NumPy
8. **Görselleştirme**: Plotly, D3.js
9. **API Dokümantasyonu**: Swagger/OpenAPI
10. **Konteynerizasyon**: Docker
11. **CI/CD**: GitHub Actions

### Proje Modülleri

Aşağıdaki modülleri sırasıyla geliştir ve her modül için ayrı bir klasör, test ve dokümantasyon oluştur:

#### 1. Veri Modeli ve Veritabanı Tasarımı

- Personel, vardiya, departman, yetkinlik ve tercih modellerini oluştur
- İlişkisel veritabanı şemasını tasarla
- Veri doğrulama ve bütünlük kurallarını tanımla
- Örnek veri oluşturma ve veritabanı migrasyon scriptlerini hazırla

#### 2. Hasta Yoğunluğu Tahmin Modülü

- Geçmiş verilere dayalı hasta akışı tahmin algoritması geliştir
- Mevsimsel, haftalık ve günlük yoğunluk paternlerini öğrenen model oluştur
- Özel durumlar (tatiller, salgınlar) için ayarlanmış tahmin motorları tasarla
- Model performans metrikleri ve değerlendirme araçları ekle

#### 3. Personel Çizelgeleme Optimizasyon Motoru

- Çok faktörlü kısıt optimizasyonu algoritması geliştir
- Personel tercihlerini ve yetkinliklerini dikkate alan eşleştirme sistemi oluştur
- İş yükü dengeleme ve adil dağılım sağlayan model tasarla
- Optimizasyon sonuçlarını değerlendirme metrikleri ekle

#### 4. Acil Durum Yeniden Planlama Sistemi

- Beklenmedik personel eksikliklerinde hızlı çözüm üreten algoritma geliştir
- Minimum değişiklikle maksimum verimlilik sağlayan optimizasyon tasarla
- Kritik görevler için yedekleme stratejileri oluştur
- Hızlı yanıt süresi ve çözüm kalitesi arasında denge kur

#### 5. API Katmanı

- RESTful API endpoints tasarla
- Güvenlik ve kimlik doğrulama mekanizmaları ekle
- Rate limiting ve API kullanım izleme özellikleri ekle
- API dokümantasyonu oluştur

#### 6. Web Arayüzü

- Yönetici dashboard'u tasarla
- Personel çizelgeleme ve görüntüleme arayüzü oluştur
- İnteraktif raporlama ve analiz ekranları geliştir
- Mobil uyumlu responsive tasarım uygula

#### 7. Raporlama ve Analiz Modülü

- Çizelgeleme kalitesini değerlendiren metrikler geliştir
- Personel verimliliği ve hasta bakım kalitesi ilişkisini analiz eden model oluştur
- Çizelgeleme iyileştirmeleri için öneriler sunan sistem tasarla
- İnteraktif görselleştirmeler ve dashboard'lar ekle

#### 8. Entegrasyon Modülü

- Hastane bilgi sistemleri ile entegrasyon API'leri geliştir
- Personel yönetim sistemleri ile veri senkronizasyonu sağla
- Elektronik sağlık kayıtları sistemleri ile entegrasyon oluştur
- Üçüncü parti takvim ve bildirim sistemleri ile bağlantı kur

#### 9. Bildirim ve İletişim Sistemi

- E-posta, SMS ve uygulama içi bildirim mekanizmaları geliştir
- Çizelge değişikliklerini ilgili personele otomatik bildirme sistemi oluştur
- Onay ve red mekanizmaları tasarla
- İletişim tercihlerini yönetme arayüzü ekle

#### 10. Dağıtım ve Dokümantasyon

- Docker konteynerizasyonu ve dağıtım scriptleri hazırla
- Kullanıcı kılavuzları ve teknik dokümantasyon oluştur
- Sistem yönetici kılavuzu hazırla
- Eğitim materyalleri ve video demoları hazırla

### Geliştirme Aşamaları

Her modül için aşağıdaki aşamaları takip et ve her aşama için bir Markdown dosyası oluştur:

1. **Planlama ve Tasarım**
   - Gereksinim analizi
   - Sistem mimarisi
   - Veri modeli
   - API tasarımı
   - UI/UX wireframes

2. **Prototip Geliştirme**
   - Temel fonksiyonalite
   - Basit arayüz
   - Konsept kanıtlama

3. **Temel Geliştirme**
   - Veritabanı implementasyonu
   - Backend servisleri
   - Frontend bileşenleri
   - Temel algoritma implementasyonu

4. **İleri Geliştirme**
   - Optimizasyon algoritmaları
   - Makine öğrenmesi modelleri
   - İnteraktif görselleştirmeler
   - Gelişmiş kullanıcı arayüzü

5. **Test ve Doğrulama**
   - Birim testleri
   - Entegrasyon testleri
   - Performans testleri
   - Kullanıcı kabul testleri

6. **Dağıtım ve Dokümantasyon**
   - Dağıtım scriptleri
   - Kullanıcı dokümantasyonu
   - Teknik dokümantasyon
   - Eğitim materyalleri

### Veri Kaynakları

Aşağıdaki veri kaynaklarını kullanarak model eğitimi ve test için veri setleri oluştur:

1. **Nurse Scheduling Problem Dataset**
   - Kaynak: http://www.schedulingbenchmarks.org/
   - İçerik: Hemşire çizelgeleme problemleri ve çözümleri
   - Format: CSV ve özel formatlar

2. **Hospital Patient Flow Data**
   - Kaynak: https://archive.ics.uci.edu/ml/datasets/Diabetes+130-US+hospitals+for+years+1999-2008
   - İçerik: Hastane hasta akışı ve yoğunluk verileri
   - Format: CSV formatında yapılandırılmış veri

3. **Healthcare Staff Scheduling Benchmarks**
   - Kaynak: https://www.kaggle.com/datasets/joniarroba/noshowappointments
   - İçerik: Randevu ve personel planlama verileri
   - Format: CSV formatında yapılandırılmış veri

4. **Healthcare Workforce Analytics**
   - Kaynak: https://www.kaggle.com/datasets/nehaprabhavalkar/av-healthcare-analytics-ii
   - İçerik: Sağlık personeli performans ve iş yükü verileri
   - Format: CSV formatında yapılandırılmış veri

5. **Emergency Department Operations**
   - Kaynak: https://data.cms.gov/provider-data/
   - İçerik: Acil servis operasyonları ve personel verileri
   - Format: CSV formatında yapılandırılmış veri

### Sentetik Veri Üretimi

Gerçek verilerin yetersiz olduğu durumlarda aşağıdaki sentetik veri üretim yaklaşımlarını kullan:

1. **StaffSim™ - Personel Davranış Simülasyonu**:
   - Çeşitli personel tiplerinin tercih, yetkinlik ve performans profillerini simüle eden model
   - Farklı çalışma koşullarında personel davranışını modelleyen veri üretimi
   - İzin, hastalık ve beklenmedik durumları içeren senaryolar

2. **PatientFlowGen™ - Hasta Akışı Simülasyonu**:
   - Günlük, haftalık ve mevsimsel hasta yoğunluğu paternlerini simüle eden model
   - Acil durumlar ve salgın senaryolarında hasta akışını modelleyen veri
   - Departman bazlı hasta dağılımını simüle eden veri üretimi

### Değerlendirme Metrikleri

Sistemin başarısını ölçmek için aşağıdaki metrikleri kullan:

1. **Çizelgeleme Kalitesi**
   - Personel tercihlerine uyum oranı
   - İş yükü dağılımı eşitliği
   - Minimum personel gereksinimlerini karşılama oranı

2. **Algoritma Performansı**
   - Çözüm üretme süresi
   - Optimizasyon kalitesi (hedef fonksiyonu değeri)
   - Kısıt ihlali sayısı

3. **Tahmin Doğruluğu**
   - Hasta yoğunluğu tahmin RMSE
   - Personel ihtiyacı tahmin doğruluğu
   - Anomali tespit başarısı

4. **Sistem Performansı**
   - API yanıt süreleri
   - Veritabanı sorgu performansı
   - Ölçeklenebilirlik testleri

5. **Kullanıcı Memnuniyeti**
   - Arayüz kullanım kolaylığı
   - Öğrenme eğrisi
   - Kullanıcı geri bildirimleri

### Teslim Edilecek Ürünler

Proje sonunda aşağıdaki ürünleri teslim et:

1. **Kaynak Kodu**
   - Tam kaynak kodu ve bağımlılıklar
   - Geliştirme, test ve üretim ortamları için yapılandırma dosyaları
   - Veritabanı şema ve migrasyon scriptleri

2. **Dokümantasyon**
   - Sistem mimarisi dokümantasyonu
   - API dokümantasyonu
   - Kullanıcı kılavuzları
   - Teknik referans kılavuzu
   - Kurulum ve dağıtım kılavuzu

3. **Test Sonuçları**
   - Birim test raporları
   - Entegrasyon test raporları
   - Performans test sonuçları
   - Kullanıcı kabul testi sonuçları

4. **Dağıtım Paketi**
   - Docker imajları
   - Docker Compose yapılandırması
   - CI/CD pipeline yapılandırması
   - Dağıtım scriptleri

5. **Demo ve Sunumlar**
   - Canlı demo ortamı
   - Sunum slaytları
   - Demo videoları
   - Vaka çalışmaları

### Başarı Kriterleri

Projenin başarılı sayılması için aşağıdaki kriterleri karşılamalıdır:

1. Sistem, en az %95 oranında geçerli çizelgeler üretebilmeli
2. Hasta yoğunluğu tahminleri %85'in üzerinde doğrulukla yapılabilmeli
3. Acil durum yeniden planlama 5 dakikadan kısa sürede çözüm üretebilmeli
4. Sistem, 1000 personel ve 50 departmana kadar ölçeklenebilmeli
5. Kullanıcı arayüzü sezgisel ve kullanımı kolay olmalı
6. Tüm API çağrıları 500ms'den kısa sürede yanıt verebilmeli
7. Sistem, gerçek dünya senaryolarında test edilmiş ve doğrulanmış olmalı

### Zaman Çizelgesi

Projeyi aşağıdaki zaman çizelgesine göre geliştir:

1. **Hafta 1-2**: Planlama, Tasarım ve Veri Modeli
2. **Hafta 3-4**: Veritabanı ve Temel API Geliştirme
3. **Hafta 5-6**: Hasta Yoğunluğu Tahmin Modülü
4. **Hafta 7-9**: Personel Çizelgeleme Optimizasyon Motoru
5. **Hafta 10-11**: Acil Durum Yeniden Planlama Sistemi
6. **Hafta 12-13**: Web Arayüzü Geliştirme
7. **Hafta 14-15**: Raporlama ve Analiz Modülü
8. **Hafta 16-17**: Entegrasyon ve Bildirim Sistemi
9. **Hafta 18-19**: Test ve Doğrulama
10. **Hafta 20**: Dağıtım ve Dokümantasyon

### Geliştirme Kuralları

1. **Kod Kalitesi**
   - PEP 8 standartlarına uygun Python kodu
   - ESLint kurallarına uygun JavaScript kodu
   - %80'in üzerinde kod kapsama oranına sahip testler
   - Düzenli code review ve refactoring

2. **Versiyon Kontrolü**
   - Git flow branching stratejisi
   - Anlamlı commit mesajları
   - Pull request tabanlı geliştirme
   - Semantic versioning

3. **Dokümantasyon**
   - Kod içi dokümantasyon (docstrings)
   - README dosyaları
   - API dokümantasyonu
   - Mimari diyagramlar

4. **Güvenlik**
   - OWASP güvenlik prensipleri
   - Düzenli güvenlik taramaları
   - Hassas verilerin şifrelenmesi
   - Güvenli API tasarımı

5. **Performans**
   - Veritabanı indeksleme ve optimizasyon
   - API caching stratejileri
   - Frontend performans optimizasyonu
   - Düzenli performans testleri

Bu prompt'u takip ederek, MedSchedule projesini başarıyla geliştirebilir ve bitirme projeniz için kapsamlı bir sistem oluşturabilirsiniz. Her aşamada ilerlemeyi Markdown dosyaları ile belgeleyerek, projenin şeffaf ve izlenebilir olmasını sağlayın.
