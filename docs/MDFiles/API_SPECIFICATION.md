# MedSchedule API Spesifikasyonu

Bu doküman, MedSchedule projesinin RESTful API endpoints'lerini ve kullanım detaylarını tanımlar.

## API Genel Bilgileri

### Temel URL
```
https://api.medschedule.com/v1
```

### Kimlik Doğrulama
- JWT (JSON Web Token) tabanlı kimlik doğrulama
- Token formatı: `Bearer <token>`
- Token yenileme endpoint'i: `/auth/refresh`

### Yanıt Formatı
```json
{
    "success": true,
    "data": {},
    "message": "İşlem başarılı",
    "errors": null
}
```

### Hata Yanıtı
```json
{
    "success": false,
    "data": null,
    "message": "Hata mesajı",
    "errors": {
        "field": ["Hata detayı"]
    }
}
```

### HTTP Durum Kodları
- 200: Başarılı
- 201: Oluşturuldu
- 400: Geçersiz İstek
- 401: Yetkisiz
- 403: Yasaklandı
- 404: Bulunamadı
- 422: İşlenemez Varlık
- 500: Sunucu Hatası

## API Endpoints

### Kimlik Doğrulama

#### POST /auth/login
Kullanıcı girişi yapar ve JWT token döndürür.

**İstek**
```json
{
    "username": "string",
    "password": "string"
}
```

**Yanıt**
```json
{
    "access_token": "string",
    "refresh_token": "string",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

#### POST /auth/refresh
Access token'ı yeniler.

**İstek**
```json
{
    "refresh_token": "string"
}
```

**Yanıt**
```json
{
    "access_token": "string",
    "expires_in": 3600
}
```

### Kullanıcı Yönetimi

#### GET /users/me
Mevcut kullanıcının bilgilerini döndürür.

**Yanıt**
```json
{
    "id": "uuid",
    "username": "string",
    "email": "string",
    "role": "string",
    "is_active": true,
    "created_at": "datetime",
    "updated_at": "datetime"
}
```

#### PUT /users/me
Mevcut kullanıcının bilgilerini günceller.

**İstek**
```json
{
    "email": "string",
    "current_password": "string",
    "new_password": "string"
}
```

### Personel Yönetimi

#### GET /staff
Personel listesini döndürür.

**Sorgu Parametreleri**
- department_id: UUID
- position: string
- page: integer
- per_page: integer

**Yanıt**
```json
{
    "items": [
        {
            "id": "uuid",
            "first_name": "string",
            "last_name": "string",
            "email": "string",
            "position": "string",
            "department": {
                "id": "uuid",
                "name": "string"
            }
        }
    ],
    "total": 0,
    "page": 1,
    "per_page": 10
}
```

#### POST /staff
Yeni personel oluşturur.

**İstek**
```json
{
    "first_name": "string",
    "last_name": "string",
    "email": "string",
    "phone": "string",
    "position": "string",
    "department_id": "uuid",
    "hire_date": "date"
}
```

#### GET /staff/{id}
Belirli bir personelin detaylarını döndürür.

**Yanıt**
```json
{
    "id": "uuid",
    "first_name": "string",
    "last_name": "string",
    "email": "string",
    "phone": "string",
    "position": "string",
    "department": {
        "id": "uuid",
        "name": "string"
    },
    "skills": [
        {
            "id": "uuid",
            "specialty": {
                "id": "uuid",
                "name": "string"
            },
            "level": 1,
            "certified": true
        }
    ],
    "preferences": [
        {
            "id": "uuid",
            "day_of_week": 1,
            "start_time": "time",
            "end_time": "time",
            "weight": 1
        }
    ]
}
```

### Departman Yönetimi

#### GET /departments
Departman listesini döndürür.

**Yanıt**
```json
{
    "items": [
        {
            "id": "uuid",
            "name": "string",
            "description": "string",
            "location": "string",
            "manager": {
                "id": "uuid",
                "name": "string"
            }
        }
    ]
}
```

#### POST /departments
Yeni departman oluşturur.

**İstek**
```json
{
    "name": "string",
    "description": "string",
    "location": "string",
    "manager_id": "uuid"
}
```

### Çizelge Yönetimi

#### GET /schedules
Çizelge listesini döndürür.

**Sorgu Parametreleri**
- department_id: UUID
- start_date: date
- end_date: date
- status: string
- page: integer
- per_page: integer

**Yanıt**
```json
{
    "items": [
        {
            "id": "uuid",
            "name": "string",
            "department": {
                "id": "uuid",
                "name": "string"
            },
            "start_date": "date",
            "end_date": "date",
            "status": "string",
            "created_by": {
                "id": "uuid",
                "name": "string"
            },
            "created_at": "datetime"
        }
    ],
    "total": 0,
    "page": 1,
    "per_page": 10
}
```

#### POST /schedules
Yeni çizelge oluşturur.

**İstek**
```json
{
    "name": "string",
    "department_id": "uuid",
    "start_date": "date",
    "end_date": "date"
}
```

#### GET /schedules/{id}
Belirli bir çizelgenin detaylarını döndürür.

**Yanıt**
```json
{
    "id": "uuid",
    "name": "string",
    "department": {
        "id": "uuid",
        "name": "string"
    },
    "start_date": "date",
    "end_date": "date",
    "status": "string",
    "shifts": [
        {
            "id": "uuid",
            "staff": {
                "id": "uuid",
                "name": "string"
            },
            "start_time": "datetime",
            "end_time": "datetime",
            "position": "string",
            "is_backup": false,
            "status": "string"
        }
    ],
    "created_by": {
        "id": "uuid",
        "name": "string"
    },
    "created_at": "datetime",
    "updated_at": "datetime"
}
```

#### POST /schedules/{id}/generate
Otomatik çizelge oluşturur.

**İstek**
```json
{
    "optimization_parameters": {
        "max_consecutive_shifts": 3,
        "min_rest_hours": 12,
        "preference_weight": 0.7,
        "workload_balance_weight": 0.3
    }
}
```

#### PUT /schedules/{id}/publish
Çizelgeyi yayınlar.

#### POST /schedules/{id}/shifts
Çizelgeye vardiya ekler.

**İstek**
```json
{
    "staff_id": "uuid",
    "start_time": "datetime",
    "end_time": "datetime",
    "position": "string",
    "is_backup": false
}
```

### Vardiya Yönetimi

#### PUT /shifts/{id}
Vardiyayı günceller.

**İstek**
```json
{
    "staff_id": "uuid",
    "start_time": "datetime",
    "end_time": "datetime",
    "position": "string",
    "is_backup": false,
    "status": "string"
}
```

#### DELETE /shifts/{id}
Vardiyayı siler.

### Hasta Akışı Analizi

#### GET /patient-flow
Hasta akışı verilerini döndürür.

**Sorgu Parametreleri**
- department_id: UUID
- start_date: date
- end_date: date
- granularity: string (hourly, daily, weekly)

**Yanıt**
```json
{
    "items": [
        {
            "department_id": "uuid",
            "date": "date",
            "hour": 0,
            "patient_count": 0
        }
    ]
}
```

#### POST /patient-flow/predict
Gelecek hasta akışını tahmin eder.

**İstek**
```json
{
    "department_id": "uuid",
    "start_date": "date",
    "end_date": "date",
    "granularity": "string"
}
```

### Bildirim Yönetimi

#### GET /notifications
Bildirimleri döndürür.

**Sorgu Parametreleri**
- is_read: boolean
- page: integer
- per_page: integer

**Yanıt**
```json
{
    "items": [
        {
            "id": "uuid",
            "type": "string",
            "message": "string",
            "is_read": false,
            "created_at": "datetime"
        }
    ],
    "total": 0,
    "page": 1,
    "per_page": 10
}
```

#### PUT /notifications/{id}/read
Bildirimi okundu olarak işaretler.

### Rapor ve Analiz

#### GET /reports/staff-workload
Personel iş yükü raporunu döndürür.

**Sorgu Parametreleri**
- department_id: UUID
- start_date: date
- end_date: date

**Yanıt**
```json
{
    "items": [
        {
            "staff_id": "uuid",
            "staff_name": "string",
            "total_hours": 0,
            "shift_count": 0,
            "overtime_hours": 0
        }
    ]
}
```

#### GET /reports/schedule-compliance
Çizelge uyum raporunu döndürür.

**Sorgu Parametreleri**
- schedule_id: UUID

**Yanıt**
```json
{
    "total_shifts": 0,
    "completed_shifts": 0,
    "cancelled_shifts": 0,
    "compliance_rate": 0.0,
    "preference_satisfaction_rate": 0.0
}
```

## WebSocket API

### Gerçek Zamanlı Bildirimler

**Bağlantı URL'i**
```
wss://api.medschedule.com/v1/ws
```

**Olaylar**
- schedule_updated: Çizelge güncellendiğinde
- shift_assigned: Vardiya atandığında
- shift_changed: Vardiya değiştiğinde
- emergency_alert: Acil durum olduğunda

**Örnek Mesaj**
```json
{
    "event": "shift_assigned",
    "data": {
        "shift_id": "uuid",
        "schedule_id": "uuid",
        "staff_id": "uuid",
        "start_time": "datetime",
        "end_time": "datetime"
    }
}
```

## API Kullanım Kuralları

### Hız Sınırları
- Kimlik doğrulamalı istekler: 100 istek/dakika
- Kimlik doğrulamasız istekler: 20 istek/dakika

### Sayfalama
- Varsayılan sayfa boyutu: 10
- Maksimum sayfa boyutu: 100

### Önbellek
- GET istekleri için ETag desteği
- 304 Not Modified yanıtları

### CORS
- Tüm origins için CORS desteği
- Preflight istekleri için OPTIONS desteği

## Güvenlik

### Rate Limiting
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1583850969
```

### API Anahtarları
- Production ortamı için API anahtarı zorunlu
- API anahtarları yönetim panelinden oluşturulur

### IP Kısıtlamaları
- Güvenilir IP adresleri listesi tanımlanabilir
- IP bazlı erişim kontrolü

## Hata Kodları

### Genel Hatalar
- 1000: Geçersiz istek formatı
- 1001: Eksik zorunlu alan
- 1002: Geçersiz değer

### Kimlik Doğrulama Hataları
- 2000: Geçersiz kimlik bilgileri
- 2001: Süresi dolmuş token
- 2002: Yetkisiz erişim

### İş Mantığı Hataları
- 3000: Çakışan vardiya
- 3001: Yetersiz dinlenme süresi
- 3002: Maksimum vardiya sınırı aşıldı

## API Versiyonlama

### Versiyon Geçişleri
- Yeni sürümler /v2, /v3 şeklinde
- Eski sürümler 6 ay desteklenir
- Sürüm geçişleri önceden duyurulur

### Geriye Dönük Uyumluluk
- Breaking changes yeni sürümlerde
- Patch güncellemeleri geriye dönük uyumlu

## API Dokümantasyonu

### Swagger/OpenAPI
- Swagger UI: /docs
- OpenAPI şeması: /openapi.json

### Postman Koleksiyonu
- Örnek istekler içeren Postman koleksiyonu
- Ortam değişkenleri yapılandırması

## Sonuç

Bu API spesifikasyonu, MedSchedule projesinin tüm API endpoints'lerini ve kullanım detaylarını tanımlar. Yapay zeka, bu spesifikasyonu kullanarak API'yi implementasyon etmeli ve dokümante etmelidir. API, güvenli, ölçeklenebilir ve kullanıcı dostu olmalıdır.
