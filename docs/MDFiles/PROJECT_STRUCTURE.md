# MedSchedule Proje Yapısı

Bu doküman, MedSchedule projesinin klasör yapısını ve dosya organizasyonunu tanımlar. Yapay zeka, projeyi geliştirirken bu yapıyı takip etmelidir.

## Kök Dizin Yapısı

```
medschedule/
├── backend/           # Backend uygulaması
├── frontend/          # Frontend uygulaması
├── docs/              # Proje dokümantasyonu
├── data/              # Veri setleri ve veri işleme scriptleri
├── tests/             # Test dosyaları
├── scripts/           # Yardımcı scriptler
├── deployment/        # Dağıtım yapılandırmaları
├── .github/           # GitHub Actions yapılandırmaları
├── .gitignore         # Git tarafından yok sayılacak dosyalar
├── README.md          # Proje ana dokümantasyonu
├── LICENSE            # Lisans bilgisi
└── requirements.txt   # Python bağımlılıkları
```

## Backend Dizin Yapısı

```
backend/
├── app/                      # Ana uygulama paketi
│   ├── __init__.py           # Paket başlatıcı
│   ├── main.py               # Uygulama giriş noktası
│   ├── config.py             # Konfigürasyon ayarları
│   ├── database.py           # Veritabanı bağlantısı
│   ├── api/                  # API endpoints
│   │   ├── __init__.py
│   │   ├── routes.py         # Ana API rotaları
│   │   ├── auth.py           # Kimlik doğrulama API'leri
│   │   ├── schedules.py      # Çizelge API'leri
│   │   ├── staff.py          # Personel API'leri
│   │   ├── departments.py    # Departman API'leri
│   │   └── analytics.py      # Analitik API'leri
│   ├── models/               # Veritabanı modelleri
│   │   ├── __init__.py
│   │   ├── user.py           # Kullanıcı modeli
│   │   ├── staff.py          # Personel modeli
│   │   ├── schedule.py       # Çizelge modeli
│   │   ├── department.py     # Departman modeli
│   │   ├── skill.py          # Yetkinlik modeli
│   │   ├── preference.py     # Tercih modeli
│   │   └── patient_flow.py   # Hasta akışı modeli
│   ├── schemas/              # Pydantic şemaları
│   │   ├── __init__.py
│   │   ├── user.py           # Kullanıcı şemaları
│   │   ├── staff.py          # Personel şemaları
│   │   ├── schedule.py       # Çizelge şemaları
│   │   └── department.py     # Departman şemaları
│   ├── services/             # İş mantığı servisleri
│   │   ├── __init__.py
│   │   ├── auth_service.py   # Kimlik doğrulama servisi
│   │   ├── schedule_service.py # Çizelgeleme servisi
│   │   ├── optimization_service.py # Optimizasyon servisi
│   │   ├── prediction_service.py # Tahmin servisi
│   │   ├── notification_service.py # Bildirim servisi
│   │   └── analytics_service.py # Analitik servisi
│   ├── core/                 # Çekirdek fonksiyonlar
│   │   ├── __init__.py
│   │   ├── security.py       # Güvenlik fonksiyonları
│   │   ├── config.py         # Konfigürasyon yönetimi
│   │   └── exceptions.py     # Özel istisnalar
│   ├── utils/                # Yardımcı fonksiyonlar
│   │   ├── __init__.py
│   │   ├── db_utils.py       # Veritabanı yardımcıları
│   │   ├── time_utils.py     # Zaman işleme yardımcıları
│   │   └── validators.py     # Doğrulama fonksiyonları
│   └── ml/                   # Makine öğrenmesi modülleri
│       ├── __init__.py
│       ├── patient_flow_predictor.py # Hasta akışı tahmin modeli
│       ├── workload_predictor.py # İş yükü tahmin modeli
│       ├── schedule_optimizer.py # Çizelge optimizasyon modeli
│       └── emergency_replanner.py # Acil durum planlayıcı
├── alembic/                  # Veritabanı migrasyonları
│   ├── versions/             # Migrasyon versiyonları
│   ├── env.py                # Alembic ortam ayarları
│   └── alembic.ini           # Alembic konfigürasyonu
├── tests/                    # Backend testleri
│   ├── __init__.py
│   ├── conftest.py           # Test yapılandırması
│   ├── test_api/             # API testleri
│   ├── test_models/          # Model testleri
│   ├── test_services/        # Servis testleri
│   └── test_ml/              # ML modeli testleri
├── Dockerfile                # Backend Docker yapılandırması
├── requirements.txt          # Backend bağımlılıkları
└── .env.example              # Örnek çevre değişkenleri
```

## Frontend Dizin Yapısı

```
frontend/
├── public/                   # Statik dosyalar
│   ├── index.html           # Ana HTML dosyası
│   ├── favicon.ico          # Site ikonu
│   └── assets/              # Statik varlıklar
├── src/                      # Kaynak kodları
│   ├── index.js             # Uygulama giriş noktası
│   ├── App.js               # Ana uygulama bileşeni
│   ├── components/          # Yeniden kullanılabilir bileşenler
│   │   ├── common/          # Genel bileşenler
│   │   │   ├── Navbar.js    # Navigasyon çubuğu
│   │   │   ├── Sidebar.js   # Kenar çubuğu
│   │   │   ├── Footer.js    # Alt bilgi
│   │   │   ├── Button.js    # Düğme bileşeni
│   │   │   ├── Card.js      # Kart bileşeni
│   │   │   ├── Table.js     # Tablo bileşeni
│   │   │   └── Modal.js     # Modal bileşeni
│   │   ├── schedule/        # Çizelge bileşenleri
│   │   │   ├── Calendar.js  # Takvim görünümü
│   │   │   ├── ShiftCard.js # Vardiya kartı
│   │   │   ├── ScheduleGrid.js # Çizelge ızgarası
│   │   │   └── FilterPanel.js # Filtreleme paneli
│   │   ├── staff/           # Personel bileşenleri
│   │   │   ├── StaffList.js # Personel listesi
│   │   │   ├── StaffCard.js # Personel kartı
│   │   │   └── StaffForm.js # Personel formu
│   │   ├── department/      # Departman bileşenleri
│   │   ├── analytics/       # Analitik bileşenleri
│   │   └── notifications/   # Bildirim bileşenleri
│   ├── pages/               # Sayfa bileşenleri
│   │   ├── Dashboard.js     # Dashboard sayfası
│   │   ├── SchedulePage.js  # Çizelge sayfası
│   │   ├── StaffPage.js     # Personel sayfası
│   │   ├── DepartmentPage.js # Departman sayfası
│   │   ├── AnalyticsPage.js # Analitik sayfası
│   │   ├── SettingsPage.js  # Ayarlar sayfası
│   │   ├── LoginPage.js     # Giriş sayfası
│   │   └── ProfilePage.js   # Profil sayfası
│   ├── services/            # API servisleri
│   │   ├── api.js           # API istemcisi
│   │   ├── authService.js   # Kimlik doğrulama servisi
│   │   ├── scheduleService.js # Çizelge servisi
│   │   ├── staffService.js  # Personel servisi
│   │   └── analyticsService.js # Analitik servisi
│   ├── store/               # State yönetimi
│   │   ├── index.js         # Store yapılandırması
│   │   ├── actions/         # Redux aksiyonları
│   │   ├── reducers/        # Redux reducer'ları
│   │   ├── selectors/       # Redux seçicileri
│   │   └── types/           # Aksiyon tipleri
│   ├── utils/               # Yardımcı fonksiyonlar
│   │   ├── dateUtils.js     # Tarih işleme yardımcıları
│   │   ├── formatters.js    # Formatlama yardımcıları
│   │   ├── validators.js    # Form doğrulama
│   │   └── constants.js     # Sabitler
│   ├── hooks/               # Özel React hook'ları
│   │   ├── useAuth.js       # Kimlik doğrulama hook'u
│   │   ├── useSchedule.js   # Çizelge hook'u
│   │   └── useNotification.js # Bildirim hook'u
│   ├── contexts/            # React context'leri
│   │   ├── AuthContext.js   # Kimlik doğrulama context'i
│   │   └── ThemeContext.js  # Tema context'i
│   ├── assets/              # Uygulama varlıkları
│   │   ├── images/          # Resimler
│   │   ├── icons/           # İkonlar
│   │   └── styles/          # Stil dosyaları
│   └── config/              # Frontend yapılandırması
│       ├── routes.js        # Rota yapılandırması
│       └── theme.js         # Tema yapılandırması
├── package.json             # NPM paketi yapılandırması
├── package-lock.json        # NPM paket kilidi
├── .eslintrc.js             # ESLint yapılandırması
├── .prettierrc              # Prettier yapılandırması
├── jsconfig.json            # JavaScript yapılandırması
├── Dockerfile               # Frontend Docker yapılandırması
└── .env.example             # Örnek çevre değişkenleri
```

## Veri Dizin Yapısı

```
data/
├── raw/                     # Ham veri setleri
│   ├── nurse_scheduling/    # Hemşire çizelgeleme verileri
│   ├── patient_flow/        # Hasta akışı verileri
│   ├── staff_scheduling/    # Personel çizelgeleme verileri
│   ├── workforce_analytics/ # İş gücü analitik verileri
│   └── emergency_operations/ # Acil servis operasyon verileri
├── processed/               # İşlenmiş veri setleri
├── synthetic/               # Sentetik veri setleri
│   ├── staff_sim/           # Personel simülasyon verileri
│   └── patient_flow_gen/    # Hasta akışı simülasyon verileri
├── models/                  # Eğitilmiş modeller
├── scripts/                 # Veri işleme scriptleri
│   ├── preprocess.py        # Ön işleme scripti
│   ├── generate_synthetic.py # Sentetik veri üretimi
│   ├── data_validation.py   # Veri doğrulama
│   └── data_analysis.py     # Veri analizi
└── notebooks/               # Jupyter notebookları
    ├── exploratory_analysis.ipynb # Keşifsel veri analizi
    ├── model_training.ipynb # Model eğitimi
    └── visualization.ipynb  # Veri görselleştirme
```

## Dokümantasyon Dizin Yapısı

```
docs/
├── architecture/            # Mimari dokümantasyon
│   ├── system_architecture.md # Sistem mimarisi
│   ├── data_model.md        # Veri modeli
│   ├── api_design.md        # API tasarımı
│   └── diagrams/            # Mimari diyagramlar
├── user_guides/             # Kullanıcı kılavuzları
│   ├── admin_guide.md       # Yönetici kılavuzu
│   ├── staff_guide.md       # Personel kılavuzu
│   └── department_guide.md  # Departman kılavuzu
├── technical/               # Teknik dokümantasyon
│   ├── backend.md           # Backend dokümantasyonu
│   ├── frontend.md          # Frontend dokümantasyonu
│   ├── algorithms.md        # Algoritma dokümantasyonu
│   └── ml_models.md         # ML modelleri dokümantasyonu
├── api/                     # API dokümantasyonu
│   ├── auth.md              # Kimlik doğrulama API'leri
│   ├── schedules.md         # Çizelge API'leri
│   ├── staff.md             # Personel API'leri
│   └── analytics.md         # Analitik API'leri
├── development/             # Geliştirme dokümantasyonu
│   ├── setup.md             # Kurulum kılavuzu
│   ├── coding_standards.md  # Kodlama standartları
│   ├── testing.md           # Test stratejisi
│   └── contributing.md      # Katkıda bulunma kılavuzu
└── deployment/              # Dağıtım dokümantasyonu
    ├── installation.md      # Kurulum kılavuzu
    ├── configuration.md     # Yapılandırma kılavuzu
    ├── scaling.md           # Ölçeklendirme kılavuzu
    └── troubleshooting.md   # Sorun giderme kılavuzu
```

## Dağıtım Dizin Yapısı

```
deployment/
├── docker/                  # Docker yapılandırmaları
│   ├── docker-compose.yml   # Docker Compose yapılandırması
│   ├── docker-compose.dev.yml # Geliştirme ortamı yapılandırması
│   ├── docker-compose.prod.yml # Üretim ortamı yapılandırması
│   ├── backend.Dockerfile   # Backend Docker yapılandırması
│   └── frontend.Dockerfile  # Frontend Docker yapılandırması
├── kubernetes/              # Kubernetes yapılandırmaları
│   ├── backend-deployment.yaml # Backend dağıtımı
│   ├── frontend-deployment.yaml # Frontend dağıtımı
│   ├── database-statefulset.yaml # Veritabanı yapılandırması
│   └── ingress.yaml         # Ingress yapılandırması
├── scripts/                 # Dağıtım scriptleri
│   ├── deploy.sh            # Dağıtım scripti
│   ├── backup.sh            # Yedekleme scripti
│   └── restore.sh           # Geri yükleme scripti
└── config/                  # Dağıtım yapılandırmaları
    ├── nginx.conf           # Nginx yapılandırması
    ├── supervisor.conf      # Supervisor yapılandırması
    └── prometheus.yml       # Prometheus yapılandırması
```

## Test Dizin Yapısı

```
tests/
├── unit/                    # Birim testleri
│   ├── backend/             # Backend birim testleri
│   │   ├── test_models/     # Model testleri
│   │   ├── test_services/   # Servis testleri
│   │   └── test_utils/      # Yardımcı fonksiyon testleri
│   └── frontend/            # Frontend birim testleri
│       ├── test_components/ # Bileşen testleri
│       ├── test_services/   # Servis testleri
│       └── test_utils/      # Yardımcı fonksiyon testleri
├── integration/             # Entegrasyon testleri
│   ├── test_api/            # API entegrasyon testleri
│   └── test_workflow/       # İş akışı testleri
├── e2e/                     # Uçtan uca testleri
│   ├── test_scheduling/     # Çizelgeleme testleri
│   ├── test_staff_management/ # Personel yönetimi testleri
│   └── test_analytics/      # Analitik testleri
├── performance/             # Performans testleri
│   ├── test_api_performance/ # API performans testleri
│   ├── test_optimization/   # Optimizasyon performans testleri
│   └── test_load/           # Yük testleri
└── fixtures/                # Test verileri
    ├── users.json           # Kullanıcı test verileri
    ├── staff.json           # Personel test verileri
    ├── schedules.json       # Çizelge test verileri
    └── departments.json     # Departman test verileri
```

Bu proje yapısı, MedSchedule uygulamasının geliştirilmesi için kapsamlı bir şablon sağlar. Yapay zeka, bu yapıyı takip ederek modüler, test edilebilir ve bakımı kolay bir sistem geliştirebilir.
