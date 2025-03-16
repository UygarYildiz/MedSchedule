# MedSchedule Veri Modeli Tasarımı

Bu doküman, MedSchedule projesinin veri modelini detaylandırır ve veritabanı şemasını tanımlar. Yapay zeka, bu veri modelini kullanarak veritabanı yapısını oluşturmalı ve uygulamanın veri katmanını geliştirmelidir.

## Veri Modeli Diyagramı

Aşağıda MedSchedule veri modelinin yüksek seviye ER (Entity-Relationship) diyagramı gösterilmiştir:

```
+----------------+     +----------------+     +----------------+
|     User       |     |     Staff      |     |   Department   |
+----------------+     +----------------+     +----------------+
| id             |1   *| id             |*   *| id             |
| username       |<--->| user_id        |<--->| name           |
| email          |     | first_name     |     | description    |
| password_hash  |     | last_name      |     | location       |
| role           |     | phone          |     | manager_id     |
| is_active      |     | email          |     | created_at     |
| created_at     |     | position       |     | updated_at     |
| updated_at     |     | department_id  |     +----------------+
+----------------+     | hire_date      |             ^
                       | created_at     |             |
                       | updated_at     |     +----------------+
                       +----------------+     |    Specialty   |
                               ^              +----------------+
                               |              | id             |
                               |              | name           |
+----------------+     +----------------+     | description    |
|   Preference   |     |     Skill      |     | created_at     |
+----------------+     +----------------+     | updated_at     |
| id             |     | id             |     +----------------+
| staff_id       |     | staff_id       |             ^
| day_of_week    |     | specialty_id   |             |
| start_time     |     | level          |     +----------------+
| end_time       |     | certified      |     |  Requirement   |
| weight         |     | created_at     |     +----------------+
| created_at     |     | updated_at     |     | id             |
| updated_at     |     +----------------+     | department_id  |
+----------------+                            | specialty_id   |
                                              | min_level      |
+----------------+     +----------------+     | min_staff      |
|    Schedule    |     |     Shift      |     | created_at     |
+----------------+     +----------------+     | updated_at     |
| id             |1   *| id             |     +----------------+
| name           |<--->| schedule_id    |
| department_id  |     | staff_id       |
| start_date     |     | start_time     |
| end_date       |     | end_time       |
| status         |     | position       |
| created_by     |     | is_backup      |
| created_at     |     | status         |
| updated_at     |     | created_at     |
+----------------+     | updated_at     |
                       +----------------+

+----------------+     +----------------+     +----------------+
| PatientFlow    |     | Notification   |     |  AuditLog      |
+----------------+     +----------------+     +----------------+
| id             |     | id             |     | id             |
| department_id  |     | user_id        |     | user_id        |
| date           |     | staff_id       |     | action         |
| hour           |     | type           |     | entity_type    |
| patient_count  |     | message        |     | entity_id      |
| created_at     |     | is_read        |     | details        |
| updated_at     |     | created_at     |     | ip_address     |
+----------------+     | updated_at     |     | created_at     |
                       +----------------+     +----------------+
```

## Veri Modeli Detayları

### User (Kullanıcı)

Sistemdeki kullanıcıları temsil eder. Her kullanıcı bir role sahiptir ve personel kaydıyla ilişkilendirilebilir.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| username | String | Kullanıcı adı | Benzersiz, boş olamaz |
| email | String | E-posta adresi | Benzersiz, boş olamaz |
| password_hash | String | Şifrelenmiş parola | Boş olamaz |
| role | Enum | Kullanıcı rolü (admin, manager, staff, viewer) | Boş olamaz, varsayılan: staff |
| is_active | Boolean | Kullanıcı aktif mi | Varsayılan: true |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### Staff (Personel)

Hastane personelini temsil eder. Her personel bir kullanıcı hesabına ve bir departmana bağlıdır.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| user_id | UUID | Bağlı kullanıcı | Yabancı anahtar (User), benzersiz, null olabilir |
| first_name | String | Ad | Boş olamaz |
| last_name | String | Soyad | Boş olamaz |
| phone | String | Telefon numarası | Null olabilir |
| email | String | E-posta adresi | Boş olamaz |
| position | String | Pozisyon/Unvan | Boş olamaz |
| department_id | UUID | Bağlı departman | Yabancı anahtar (Department), boş olamaz |
| hire_date | Date | İşe başlama tarihi | Boş olamaz |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### Department (Departman)

Hastane departmanlarını temsil eder.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| name | String | Departman adı | Benzersiz, boş olamaz |
| description | Text | Departman açıklaması | Null olabilir |
| location | String | Fiziksel konum | Null olabilir |
| manager_id | UUID | Departman yöneticisi | Yabancı anahtar (Staff), null olabilir |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### Specialty (Uzmanlık)

Personelin sahip olabileceği uzmanlık alanlarını temsil eder.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| name | String | Uzmanlık adı | Benzersiz, boş olamaz |
| description | Text | Uzmanlık açıklaması | Null olabilir |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### Skill (Yetkinlik)

Personelin sahip olduğu yetkinlikleri temsil eder. Her yetkinlik bir uzmanlık alanına bağlıdır.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| staff_id | UUID | Personel | Yabancı anahtar (Staff), boş olamaz |
| specialty_id | UUID | Uzmanlık alanı | Yabancı anahtar (Specialty), boş olamaz |
| level | Integer | Yetkinlik seviyesi (1-5) | Boş olamaz, 1-5 arası |
| certified | Boolean | Sertifikalı mı | Varsayılan: false |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### Requirement (Gereksinim)

Departmanların ihtiyaç duyduğu uzmanlık alanlarını ve minimum personel sayısını temsil eder.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| department_id | UUID | Departman | Yabancı anahtar (Department), boş olamaz |
| specialty_id | UUID | Uzmanlık alanı | Yabancı anahtar (Specialty), boş olamaz |
| min_level | Integer | Minimum yetkinlik seviyesi | Boş olamaz, 1-5 arası |
| min_staff | Integer | Minimum personel sayısı | Boş olamaz, pozitif |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### Preference (Tercih)

Personelin çalışma tercihleri ve kısıtlarını temsil eder.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| staff_id | UUID | Personel | Yabancı anahtar (Staff), boş olamaz |
| day_of_week | Integer | Haftanın günü (0-6, 0=Pazartesi) | Boş olamaz, 0-6 arası |
| start_time | Time | Başlangıç saati | Boş olamaz |
| end_time | Time | Bitiş saati | Boş olamaz |
| weight | Integer | Tercih ağırlığı (1-10) | Boş olamaz, 1-10 arası |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### Schedule (Çizelge)

Bir departman için oluşturulan çalışma çizelgesini temsil eder.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| name | String | Çizelge adı | Boş olamaz |
| department_id | UUID | Departman | Yabancı anahtar (Department), boş olamaz |
| start_date | Date | Başlangıç tarihi | Boş olamaz |
| end_date | Date | Bitiş tarihi | Boş olamaz |
| status | Enum | Durum (draft, published, archived) | Boş olamaz, varsayılan: draft |
| created_by | UUID | Oluşturan kullanıcı | Yabancı anahtar (User), boş olamaz |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### Shift (Vardiya)

Çizelgedeki her bir vardiyayı temsil eder. Her vardiya bir personele atanır.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| schedule_id | UUID | Çizelge | Yabancı anahtar (Schedule), boş olamaz |
| staff_id | UUID | Personel | Yabancı anahtar (Staff), boş olamaz |
| start_time | DateTime | Başlangıç zamanı | Boş olamaz |
| end_time | DateTime | Bitiş zamanı | Boş olamaz |
| position | String | Vardiya pozisyonu | Boş olamaz |
| is_backup | Boolean | Yedek personel mi | Varsayılan: false |
| status | Enum | Durum (scheduled, confirmed, completed, cancelled) | Boş olamaz, varsayılan: scheduled |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### PatientFlow (Hasta Akışı)

Departmanlardaki hasta akışını ve yoğunluğunu temsil eder. Tahmin modelleri için kullanılır.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| department_id | UUID | Departman | Yabancı anahtar (Department), boş olamaz |
| date | Date | Tarih | Boş olamaz |
| hour | Integer | Saat (0-23) | Boş olamaz, 0-23 arası |
| patient_count | Integer | Hasta sayısı | Boş olamaz, pozitif veya sıfır |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### Notification (Bildirim)

Kullanıcılara gönderilen bildirimleri temsil eder.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| user_id | UUID | Kullanıcı | Yabancı anahtar (User), boş olamaz |
| staff_id | UUID | İlgili personel | Yabancı anahtar (Staff), null olabilir |
| type | Enum | Bildirim tipi (schedule_change, shift_assignment, etc.) | Boş olamaz |
| message | Text | Bildirim mesajı | Boş olamaz |
| is_read | Boolean | Okundu mu | Varsayılan: false |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |
| updated_at | DateTime | Güncellenme zamanı | Otomatik |

### AuditLog (Denetim Günlüğü)

Sistemdeki önemli işlemleri ve değişiklikleri kaydeder.

| Alan | Tür | Açıklama | Kısıtlar |
|------|-----|----------|----------|
| id | UUID | Benzersiz tanımlayıcı | Birincil anahtar |
| user_id | UUID | İşlemi yapan kullanıcı | Yabancı anahtar (User), null olabilir |
| action | String | Yapılan işlem | Boş olamaz |
| entity_type | String | Etkilenen varlık tipi | Boş olamaz |
| entity_id | UUID | Etkilenen varlık ID | Null olabilir |
| details | JSON | İşlem detayları | Null olabilir |
| ip_address | String | IP adresi | Null olabilir |
| created_at | DateTime | Oluşturulma zamanı | Otomatik |

## İlişkiler

1. **User - Staff**: Bir kullanıcı bir personele bağlı olabilir (one-to-one).
2. **Staff - Department**: Bir personel bir departmana bağlıdır (many-to-one).
3. **Department - Staff (Manager)**: Bir departmanın bir yöneticisi olabilir (many-to-one).
4. **Staff - Skill**: Bir personelin birden fazla yetkinliği olabilir (one-to-many).
5. **Skill - Specialty**: Bir yetkinlik bir uzmanlık alanına bağlıdır (many-to-one).
6. **Department - Requirement**: Bir departmanın birden fazla uzmanlık gereksinimi olabilir (one-to-many).
7. **Requirement - Specialty**: Bir gereksinim bir uzmanlık alanına bağlıdır (many-to-one).
8. **Staff - Preference**: Bir personelin birden fazla çalışma tercihi olabilir (one-to-many).
9. **Department - Schedule**: Bir departmanın birden fazla çizelgesi olabilir (one-to-many).
10. **Schedule - Shift**: Bir çizelgenin birden fazla vardiyası olabilir (one-to-many).
11. **Staff - Shift**: Bir personel birden fazla vardiyaya atanabilir (one-to-many).
12. **Department - PatientFlow**: Bir departmanın birden fazla hasta akışı kaydı olabilir (one-to-many).
13. **User - Notification**: Bir kullanıcının birden fazla bildirimi olabilir (one-to-many).
14. **Staff - Notification**: Bir bildirim bir personelle ilişkilendirilebilir (many-to-one).
15. **User - AuditLog**: Bir kullanıcının birden fazla denetim kaydı olabilir (one-to-many).

## İndeksler

Veritabanı performansını optimize etmek için aşağıdaki indeksler oluşturulmalıdır:

1. **User**: `username`, `email`
2. **Staff**: `user_id`, `department_id`, `last_name`
3. **Department**: `name`, `manager_id`
4. **Specialty**: `name`
5. **Skill**: `staff_id`, `specialty_id`
6. **Requirement**: `department_id`, `specialty_id`
7. **Preference**: `staff_id`, `day_of_week`
8. **Schedule**: `department_id`, `start_date`, `end_date`, `status`
9. **Shift**: `schedule_id`, `staff_id`, `start_time`, `end_time`, `status`
10. **PatientFlow**: `department_id`, `date`, `hour`
11. **Notification**: `user_id`, `is_read`, `created_at`
12. **AuditLog**: `user_id`, `entity_type`, `entity_id`, `created_at`

## Kısıtlar ve Doğrulama Kuralları

1. **Shift Zamanları**: Vardiya başlangıç zamanı bitiş zamanından önce olmalıdır.
2. **Schedule Tarihleri**: Çizelge başlangıç tarihi bitiş tarihinden önce olmalıdır.
3. **Preference Zamanları**: Tercih başlangıç saati bitiş saatinden önce olmalıdır.
4. **Skill Level**: Yetkinlik seviyesi 1 ile 5 arasında olmalıdır.
5. **Requirement Min Level**: Minimum yetkinlik seviyesi 1 ile 5 arasında olmalıdır.
6. **Preference Weight**: Tercih ağırlığı 1 ile 10 arasında olmalıdır.
7. **PatientFlow Hour**: Saat 0 ile 23 arasında olmalıdır.
8. **PatientFlow Patient Count**: Hasta sayısı negatif olmamalıdır.

## Veritabanı Migrasyon Stratejisi

Veritabanı şeması, Alembic migrasyon aracı kullanılarak yönetilmelidir. Migrasyon stratejisi şu adımları içermelidir:

1. **Temel Şema**: İlk migrasyon, temel veri modelini oluşturmalıdır.
2. **İndeksler**: İkinci migrasyon, performans optimizasyonu için indeksleri eklemelidir.
3. **Kısıtlar**: Üçüncü migrasyon, veri bütünlüğünü sağlamak için kısıtları eklemelidir.
4. **Örnek Veriler**: Dördüncü migrasyon, test ve geliştirme için örnek verileri eklemelidir.

Her migrasyon, geri alınabilir olmalı ve hem ileri hem de geri yönde test edilmelidir.

## Veri Erişim Katmanı

Veri erişim katmanı, SQLAlchemy ORM kullanılarak implementasyon edilmelidir. Her veri modeli için aşağıdaki bileşenler oluşturulmalıdır:

1. **Model Sınıfı**: SQLAlchemy model sınıfı
2. **Şema Sınıfı**: Pydantic şema sınıfı (API giriş/çıkış doğrulama için)
3. **CRUD İşlemleri**: Create, Read, Update, Delete işlemleri için fonksiyonlar
4. **İlişki Yükleme**: İlişkili verileri verimli şekilde yüklemek için fonksiyonlar

## Veri Güvenliği

Veri güvenliğini sağlamak için aşağıdaki önlemler alınmalıdır:

1. **Hassas Veriler**: Parolalar gibi hassas veriler her zaman şifrelenmelidir.
2. **Erişim Kontrolü**: Rol tabanlı erişim kontrolü uygulanmalıdır.
3. **Veri Doğrulama**: Tüm kullanıcı girdileri doğrulanmalıdır.
4. **Denetim Günlüğü**: Önemli işlemler denetim günlüğüne kaydedilmelidir.
5. **Veri Maskeleme**: API yanıtlarında hassas veriler maskelenmelidir.

## Sonuç

Bu veri modeli tasarımı, MedSchedule projesinin temel veri yapısını tanımlar. Yapay zeka, bu tasarımı kullanarak veritabanı şemasını oluşturmalı, veri erişim katmanını implementasyon etmeli ve veri güvenliğini sağlamalıdır. Veri modeli, projenin gereksinimlerine göre zaman içinde genişletilebilir ve iyileştirilebilir.
