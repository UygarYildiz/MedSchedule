# MedSchedule - AI Tabanlı Sağlık Personeli Planlama Sistemi

MedSchedule, hastanelerde sağlık personeli çizelgeleme ve iş yükü dağılımını optimize etmek için yapay zeka tabanlı bir platformdur.

## Özellikler

- 🤖 Yapay zeka destekli otomatik çizelgeleme
- 📊 Gerçek zamanlı iş yükü analizi
- 👥 Personel tercih ve kısıtlama yönetimi
- 🏥 Departman bazlı kaynak optimizasyonu
- 📱 Mobil uyumlu arayüz
- 🔔 Gerçek zamanlı bildirimler
- 📈 Detaylı raporlama ve analitik

## Teknolojiler

### Backend
- FastAPI (Python web framework)
- PostgreSQL (Veritabanı)
- SQLAlchemy (ORM)
- TensorFlow (ML modelleri)
- Redis (Önbellekleme)
- Alembic (Veritabanı migrasyonları)

### Frontend
- React (UI kütüphanesi)
- TypeScript (Programlama dili)
- Material-UI (UI bileşenleri)
- Redux Toolkit (Durum yönetimi)
- React Router (Rotalama)
- Socket.IO (Gerçek zamanlı iletişim)

## Kurulum

### Gereksinimler
- Python 3.10+
- Node.js 18+
- PostgreSQL 14+
- Redis 6+

### Backend Kurulumu

```bash
# Virtual environment oluştur
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Bağımlılıkları yükle
cd backend
pip install -r requirements.txt

# Veritabanını oluştur
createdb medschedule

# Migrasyonları uygula
alembic upgrade head

# Sunucuyu başlat
uvicorn app.main:app --reload
```

### Frontend Kurulumu

```bash
# Bağımlılıkları yükle
cd frontend
npm install

# Geliştirme sunucusunu başlat
npm start
```

## Kullanım

1. Tarayıcıda `http://localhost:3000` adresine git
2. Sistem yöneticisi hesabıyla giriş yap
3. Departman ve personel bilgilerini gir
4. Çizelgeleme kısıtlamalarını tanımla
5. Otomatik çizelgeleme işlemini başlat

## Katkıda Bulunma

1. Bu depoyu fork'layın
2. Feature branch'i oluşturun (`git checkout -b feature/amazing-feature`)
3. Değişikliklerinizi commit'leyin (`git commit -m 'feat: Add amazing feature'`)
4. Branch'inizi push'layın (`git push origin feature/amazing-feature`)
5. Pull Request oluşturun

## Lisans

Bu proje MIT lisansı altında lisanslanmıştır. Detaylar için [LICENSE](LICENSE) dosyasına bakın.

## İletişim

Proje Yöneticisi - [@projeYoneticisi](https://twitter.com/projeYoneticisi)

Proje Linki: [https://github.com/kullaniciAdi/medschedule](https://github.com/kullaniciAdi/medschedule) 
