# MedSchedule Test Stratejisi

Bu doküman, MedSchedule projesinin test stratejisini ve test planını detaylandırır. Yapay zeka, bu stratejiyi kullanarak kapsamlı test senaryoları oluşturmalı ve uygulamalıdır.

## Test Seviyeleri

### 1. Birim Testleri (Unit Tests)

#### Backend Birim Testleri

##### Veri Modelleri
```python
# test_models.py
def test_staff_model():
    staff = Staff(
        first_name="John",
        last_name="Doe",
        email="john@example.com",
        position="Doctor"
    )
    assert staff.full_name == "John Doe"
    assert staff.is_active is True

def test_schedule_model():
    schedule = Schedule(
        name="Week 1",
        start_date=date(2024, 1, 1),
        end_date=date(2024, 1, 7)
    )
    assert schedule.duration_days == 7
    assert schedule.is_active is True
```

##### Servisler
```python
# test_services.py
def test_schedule_generation_service():
    service = ScheduleGenerationService()
    schedule = service.generate_schedule(
        department_id=1,
        start_date=date(2024, 1, 1),
        end_date=date(2024, 1, 7)
    )
    assert len(schedule.shifts) > 0
    assert all(shift.staff_id for shift in schedule.shifts)

def test_patient_flow_prediction_service():
    service = PatientFlowPredictionService()
    predictions = service.predict_flow(
        department_id=1,
        date=date(2024, 1, 1)
    )
    assert len(predictions) == 24  # Saatlik tahminler
    assert all(p >= 0 for p in predictions)  # Negatif değer olmamalı
```

##### Yardımcı Fonksiyonlar
```python
# test_utils.py
def test_date_utils():
    dates = generate_date_range("2024-01-01", "2024-01-07")
    assert len(dates) == 7
    assert dates[0].strftime("%Y-%m-%d") == "2024-01-01"

def test_validation_utils():
    result = validate_shift_times("08:00", "16:00")
    assert result is True
    
    result = validate_shift_times("16:00", "08:00")
    assert result is False
```

#### Frontend Birim Testleri

##### React Bileşenleri
```javascript
// ScheduleCalendar.test.js
describe('ScheduleCalendar', () => {
    it('renders calendar grid correctly', () => {
        const { container } = render(<ScheduleCalendar />);
        const days = container.querySelectorAll('.calendar-day');
        expect(days).toHaveLength(7);
    });

    it('highlights current day', () => {
        const { container } = render(<ScheduleCalendar />);
        const today = container.querySelector('.current-day');
        expect(today).toBeInTheDocument();
    });
});

// ShiftCard.test.js
describe('ShiftCard', () => {
    const shift = {
        id: 1,
        staff: { name: 'John Doe' },
        startTime: '08:00',
        endTime: '16:00'
    };

    it('displays shift information correctly', () => {
        const { getByText } = render(<ShiftCard shift={shift} />);
        expect(getByText('John Doe')).toBeInTheDocument();
        expect(getByText('08:00 - 16:00')).toBeInTheDocument();
    });
});
```

##### Redux/Zustand Store
```javascript
// scheduleSlice.test.js
describe('Schedule Slice', () => {
    it('should handle initial state', () => {
        expect(scheduleReducer(undefined, {})).toEqual({
            schedules: [],
            loading: false,
            error: null
        });
    });

    it('should handle fetchSchedules.fulfilled', () => {
        const schedules = [{ id: 1, name: 'Week 1' }];
        const action = { type: 'schedules/fetchSchedules/fulfilled', payload: schedules };
        const state = scheduleReducer({ schedules: [], loading: true }, action);
        expect(state.schedules).toEqual(schedules);
        expect(state.loading).toBe(false);
    });
});
```

### 2. Entegrasyon Testleri (Integration Tests)

#### API Entegrasyon Testleri
```python
# test_api_integration.py
def test_schedule_creation_flow():
    # 1. Departman oluştur
    department = create_test_department()
    
    # 2. Personel oluştur
    staff = create_test_staff(department.id)
    
    # 3. Çizelge oluştur
    schedule_data = {
        "department_id": department.id,
        "start_date": "2024-01-01",
        "end_date": "2024-01-07"
    }
    response = client.post("/api/schedules", json=schedule_data)
    assert response.status_code == 201
    schedule_id = response.json()["id"]
    
    # 4. Vardiya ekle
    shift_data = {
        "schedule_id": schedule_id,
        "staff_id": staff.id,
        "start_time": "2024-01-01T08:00:00",
        "end_time": "2024-01-01T16:00:00"
    }
    response = client.post(f"/api/schedules/{schedule_id}/shifts", json=shift_data)
    assert response.status_code == 201
    
    # 5. Çizelgeyi yayınla
    response = client.put(f"/api/schedules/{schedule_id}/publish")
    assert response.status_code == 200
    
    # 6. Çizelge durumunu kontrol et
    response = client.get(f"/api/schedules/{schedule_id}")
    assert response.json()["status"] == "published"
```

#### Veritabanı Entegrasyon Testleri
```python
# test_db_integration.py
def test_cascade_delete():
    # 1. Test verilerini oluştur
    department = create_test_department()
    schedule = create_test_schedule(department.id)
    shifts = create_test_shifts(schedule.id)
    
    # 2. Departmanı sil
    db.session.delete(department)
    db.session.commit()
    
    # 3. İlişkili kayıtların silindiğini doğrula
    assert Schedule.query.filter_by(id=schedule.id).first() is None
    assert Shift.query.filter_by(schedule_id=schedule.id).count() == 0

def test_transaction_rollback():
    # 1. Başlangıç durumunu kaydet
    initial_staff_count = Staff.query.count()
    
    # 2. Hatalı işlem dene
    try:
        with db.session.begin():
            staff1 = create_test_staff()
            staff2 = create_test_staff(email=staff1.email)  # Unique constraint ihlali
    except:
        pass
    
    # 3. Rollback sonrası durumu kontrol et
    final_staff_count = Staff.query.count()
    assert final_staff_count == initial_staff_count
```

### 3. End-to-End (E2E) Testleri

#### Cypress Testleri
```javascript
// schedule_management.spec.js
describe('Schedule Management', () => {
    beforeEach(() => {
        cy.login('admin', 'password');
        cy.visit('/schedules');
    });

    it('should create a new schedule', () => {
        // 1. "Yeni Çizelge" butonuna tıkla
        cy.get('[data-cy="new-schedule-btn"]').click();

        // 2. Form bilgilerini doldur
        cy.get('[data-cy="schedule-name"]').type('Ocak 2024');
        cy.get('[data-cy="department-select"]').select('Acil Servis');
        cy.get('[data-cy="start-date"]').type('2024-01-01');
        cy.get('[data-cy="end-date"]').type('2024-01-31');

        // 3. Formu gönder
        cy.get('[data-cy="submit-btn"]').click();

        // 4. Başarılı oluşturma mesajını kontrol et
        cy.get('[data-cy="success-toast"]')
            .should('be.visible')
            .and('contain', 'Çizelge başarıyla oluşturuldu');

        // 5. Çizelgenin listede görünmesini kontrol et
        cy.get('[data-cy="schedule-list"]')
            .should('contain', 'Ocak 2024');
    });

    it('should assign shifts to staff', () => {
        // 1. Var olan çizelgeyi aç
        cy.get('[data-cy="schedule-item"]').first().click();

        // 2. Vardiya ekle
        cy.get('[data-cy="add-shift-btn"]').click();
        cy.get('[data-cy="staff-select"]').select('John Doe');
        cy.get('[data-cy="shift-date"]').type('2024-01-01');
        cy.get('[data-cy="shift-start"]').type('08:00');
        cy.get('[data-cy="shift-end"]').type('16:00');
        cy.get('[data-cy="save-shift-btn"]').click();

        // 3. Vardiya atamasını kontrol et
        cy.get('[data-cy="shift-card"]')
            .should('contain', 'John Doe')
            .and('contain', '08:00 - 16:00');
    });
});

// staff_management.spec.js
describe('Staff Management', () => {
    beforeEach(() => {
        cy.login('admin', 'password');
        cy.visit('/staff');
    });

    it('should add new staff member', () => {
        // 1. "Yeni Personel" butonuna tıkla
        cy.get('[data-cy="new-staff-btn"]').click();

        // 2. Form bilgilerini doldur
        cy.get('[data-cy="first-name"]').type('Jane');
        cy.get('[data-cy="last-name"]').type('Smith');
        cy.get('[data-cy="email"]').type('jane.smith@example.com');
        cy.get('[data-cy="position"]').select('Hemşire');
        cy.get('[data-cy="department"]').select('Acil Servis');

        // 3. Formu gönder
        cy.get('[data-cy="submit-btn"]').click();

        // 4. Başarılı ekleme mesajını kontrol et
        cy.get('[data-cy="success-toast"]')
            .should('be.visible')
            .and('contain', 'Personel başarıyla eklendi');

        // 5. Personelin listede görünmesini kontrol et
        cy.get('[data-cy="staff-list"]')
            .should('contain', 'Jane Smith');
    });
});
```

### 4. Performans Testleri

#### Yük Testleri (k6)
```javascript
// load_test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
    stages: [
        { duration: '1m', target: 50 },  // 1 dakikada 50 kullanıcıya çık
        { duration: '3m', target: 50 },  // 3 dakika 50 kullanıcıda kal
        { duration: '1m', target: 0 },   // 1 dakikada 0 kullanıcıya in
    ],
    thresholds: {
        http_req_duration: ['p(95)<500'],  // %95 istekler 500ms altında olmalı
        http_req_failed: ['rate<0.01'],    // %1'den az hata olmalı
    },
};

export default function () {
    // 1. Login
    const loginRes = http.post('http://api.medschedule.com/auth/login', {
        username: 'testuser',
        password: 'testpass',
    });
    check(loginRes, { 'login successful': (r) => r.status === 200 });
    const token = loginRes.json('token');

    // 2. Çizelge listesi
    const schedulesRes = http.get('http://api.medschedule.com/schedules', {
        headers: { 'Authorization': `Bearer ${token}` },
    });
    check(schedulesRes, { 'schedules retrieved': (r) => r.status === 200 });

    // 3. Personel listesi
    const staffRes = http.get('http://api.medschedule.com/staff', {
        headers: { 'Authorization': `Bearer ${token}` },
    });
    check(staffRes, { 'staff retrieved': (r) => r.status === 200 });

    sleep(1);
}
```

#### Stres Testleri
```javascript
// stress_test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
    scenarios: {
        stress: {
            executor: 'ramping-arrival-rate',
            preAllocatedVUs: 500,
            timeUnit: '1s',
            stages: [
                { duration: '2m', target: 10 },   // Normal yük
                { duration: '5m', target: 50 },   // Artan yük
                { duration: '2m', target: 100 },  // Stres yükü
                { duration: '1m', target: 0 },    // Soğuma
            ],
        },
    },
};

export default function () {
    // 1. Çizelge oluştur
    const scheduleRes = http.post('http://api.medschedule.com/schedules', {
        name: 'Stress Test Schedule',
        department_id: 1,
        start_date: '2024-01-01',
        end_date: '2024-01-07',
    });
    check(scheduleRes, { 'schedule created': (r) => r.status === 201 });

    // 2. Vardiyalar ekle
    const shifts = [];
    for (let i = 0; i < 50; i++) {
        shifts.push({
            staff_id: (i % 10) + 1,
            start_time: '2024-01-01T08:00:00',
            end_time: '2024-01-01T16:00:00',
        });
    }
    
    const shiftsRes = http.post(`http://api.medschedule.com/schedules/${scheduleRes.json('id')}/shifts/bulk`, 
        JSON.stringify(shifts)
    );
    check(shiftsRes, { 'shifts created': (r) => r.status === 201 });

    sleep(1);
}
```

### 5. Güvenlik Testleri

#### OWASP ZAP Testleri
```yaml
# zap-config.yaml
---
scan-rules:
  - 40012  # SQL Injection
  - 40014  # Cross Site Scripting
  - 40016  # Directory Traversal
  - 40018  # Remote File Inclusion
  - 40019  # External Redirect
  - 40020  # Integer Overflow
  - 40021  # CSRF
  - 40022  # SSI Injection
  - 40024  # Command Injection
  - 40025  # LDAP Injection

alerts-filters:
  - risk-level: HIGH
    confidence-level: HIGH
    status: FAIL
  - risk-level: MEDIUM
    confidence-level: HIGH
    status: WARN

authentication:
  type: form-based
  login-url: http://api.medschedule.com/auth/login
  username-field: username
  password-field: password
  credentials:
    username: pentester
    password: test123

target-urls:
  - http://api.medschedule.com/
  - http://api.medschedule.com/schedules
  - http://api.medschedule.com/staff
  - http://api.medschedule.com/departments
```

#### Güvenlik Birim Testleri
```python
# test_security.py
def test_password_hashing():
    password = "test123"
    hashed = hash_password(password)
    assert verify_password(password, hashed) is True
    assert verify_password("wrong", hashed) is False

def test_jwt_token():
    token = create_jwt_token({"user_id": 1})
    payload = verify_jwt_token(token)
    assert payload["user_id"] == 1
    
    expired_token = create_jwt_token({"user_id": 1}, expires_in=-1)
    with pytest.raises(TokenExpiredError):
        verify_jwt_token(expired_token)

def test_rate_limiting():
    client = TestClient()
    
    # Rate limit aşılana kadar istek yap
    for _ in range(100):
        response = client.post("/auth/login", json={
            "username": "test",
            "password": "test"
        })
    
    # Rate limit aşıldığında 429 dönmeli
    response = client.post("/auth/login", json={
        "username": "test",
        "password": "test"
    })
    assert response.status_code == 429
```

## Test Otomasyonu

### CI/CD Pipeline
```yaml
# .github/workflows/test.yml
name: Test Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: medschedule_test
        ports:
          - 5432:5432
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5

    steps:
    - uses: actions/checkout@v2
    
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.9'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install -r requirements-dev.txt
    
    - name: Run unit tests
      run: pytest tests/unit
    
    - name: Run integration tests
      run: pytest tests/integration
    
    - name: Run security tests
      run: |
        pip install safety
        safety check
        bandit -r src/
    
    - name: Upload coverage
      uses: codecov/codecov-action@v2
      with:
        file: ./coverage.xml

  e2e:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '16'
    
    - name: Install dependencies
      run: |
        npm ci
        npx cypress install
    
    - name: Run Cypress tests
      run: npx cypress run
    
    - name: Upload Cypress videos
      uses: actions/upload-artifact@v2
      with:
        name: cypress-videos
        path: cypress/videos

  performance:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Run k6 tests
      uses: k6io/action@v0.1
      with:
        filename: tests/performance/load_test.js
```

### Test Raporlama
```python
# conftest.py
def pytest_html_report_title(report):
    report.title = "MedSchedule Test Raporu"

def pytest_html_results_table_header(cells):
    cells.insert(2, html.th('Description'))
    cells.pop()

def pytest_html_results_table_row(report, cells):
    cells.insert(2, html.td(report.description))
    cells.pop()
```

## Test Veri Yönetimi

### Test Fixture'ları
```python
# conftest.py
@pytest.fixture
def test_department():
    department = Department(
        name="Test Department",
        description="Test Description"
    )
    db.session.add(department)
    db.session.commit()
    yield department
    db.session.delete(department)
    db.session.commit()

@pytest.fixture
def test_staff(test_department):
    staff = Staff(
        first_name="Test",
        last_name="User",
        email="test@example.com",
        department_id=test_department.id
    )
    db.session.add(staff)
    db.session.commit()
    yield staff
    db.session.delete(staff)
    db.session.commit()

@pytest.fixture
def test_schedule(test_department):
    schedule = Schedule(
        name="Test Schedule",
        department_id=test_department.id,
        start_date=date(2024, 1, 1),
        end_date=date(2024, 1, 7)
    )
    db.session.add(schedule)
    db.session.commit()
    yield schedule
    db.session.delete(schedule)
    db.session.commit()
```

### Factory Pattern
```python
# factories.py
import factory
from factory.faker import Faker

class DepartmentFactory(factory.Factory):
    class Meta:
        model = Department

    name = Faker('company')
    description = Faker('catch_phrase')
    location = Faker('address')

class StaffFactory(factory.Factory):
    class Meta:
        model = Staff

    first_name = Faker('first_name')
    last_name = Faker('last_name')
    email = Faker('email')
    phone = Faker('phone_number')
    position = factory.Iterator(['Doctor', 'Nurse', 'Technician'])
    department = factory.SubFactory(DepartmentFactory)

class ScheduleFactory(factory.Factory):
    class Meta:
        model = Schedule

    name = Faker('sentence', nb_words=3)
    department = factory.SubFactory(DepartmentFactory)
    start_date = Faker('date_this_month')
    end_date = factory.LazyAttribute(
        lambda o: Faker('date_between', 
            start_date=o.start_date, 
            end_date='+7d'
        ).generate({})
    )
```

## Test Metrikleri

### Kapsam Hedefleri
- Birim Test Kapsamı: %90+
- Entegrasyon Test Kapsamı: %80+
- E2E Test Kapsamı: Kritik iş akışları %100

### Performans Hedefleri
- Sayfa Yüklenme: < 2 saniye
- API Yanıt Süresi: < 500ms
- Veritabanı Sorguları: < 100ms

### Kalite Hedefleri
- Kod Kalitesi: Sonar Grade A
- Test Başarı Oranı: %100
- Hata Oranı: < %1

## Sonuç

Bu test stratejisi, MedSchedule projesinin test gereksinimlerini ve yaklaşımını detaylandırır. Yapay zeka, bu stratejiyi kullanarak kapsamlı test senaryoları oluşturmalı ve uygulamalıdır. Testler, uygulamanın güvenilirliğini, performansını ve kalitesini sağlamak için sürekli olarak çalıştırılmalı ve güncellenmelidir.
