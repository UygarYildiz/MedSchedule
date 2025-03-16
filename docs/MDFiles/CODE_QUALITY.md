# MedSchedule Kod Kalitesi ve Standartları

Bu doküman, MedSchedule projesinin kod kalitesi standartlarını ve en iyi pratiklerini detaylandırır.

## 1. Kod Stili

### Python Stil Kuralları

```python
# Doğru örnek
class UserManager:
    def __init__(self, database: Database):
        self.database = database
        self._cache = {}  # Private değişken için alt çizgi

    async def get_user_by_id(self, user_id: int) -> Optional[User]:
        """Kullanıcıyı ID'ye göre getirir.

        Args:
            user_id: Kullanıcı ID'si

        Returns:
            User nesnesi veya None
        """
        if user_id in self._cache:
            return self._cache[user_id]
        
        user = await self.database.fetch_one(
            "SELECT * FROM users WHERE id = :id",
            {"id": user_id}
        )
        return User(**user) if user else None

    @property
    def active_users(self) -> List[User]:
        return [user for user in self._cache.values() if user.is_active]
```

### TypeScript Stil Kuralları

```typescript
// Doğru örnek
interface UserProps {
  id: number;
  name: string;
  email: string;
  role: UserRole;
}

class UserComponent extends React.Component<UserProps> {
  private formatName(name: string): string {
    return name.trim().toLowerCase();
  }

  public render(): JSX.Element {
    const { name, email, role } = this.props;
    
    return (
      <div className="user-card">
        <h2>{this.formatName(name)}</h2>
        <p>{email}</p>
        <RoleTag role={role} />
      </div>
    );
  }
}
```

## 2. Kod Organizasyonu

### Dizin Yapısı

```
src/
├── api/
│   ├── routes/
│   ├── middleware/
│   └── validators/
├── core/
│   ├── models/
│   ├── services/
│   └── utils/
├── db/
│   ├── migrations/
│   └── repositories/
└── config/
    ├── settings.py
    └── logging.py
```

### Modül Organizasyonu

```python
# core/models/user.py
from datetime import datetime
from typing import Optional
from pydantic import BaseModel, EmailStr

class User(BaseModel):
    id: int
    email: EmailStr
    name: str
    created_at: datetime
    updated_at: Optional[datetime]

    class Config:
        orm_mode = True
```

## 3. Dokümantasyon Standartları

### Fonksiyon Dokümantasyonu

```python
def calculate_shift_overlap(
    shift1: Shift,
    shift2: Shift
) -> Optional[timedelta]:
    """İki vardiya arasındaki çakışma süresini hesaplar.

    Args:
        shift1: İlk vardiya
        shift2: İkinci vardiya

    Returns:
        Çakışma süresi veya çakışma yoksa None

    Raises:
        ValueError: Geçersiz vardiya zamanları
    """
    if not (shift1.start_time and shift1.end_time and
            shift2.start_time and shift2.end_time):
        raise ValueError("Vardiya zamanları eksik")

    overlap_start = max(shift1.start_time, shift2.start_time)
    overlap_end = min(shift1.end_time, shift2.end_time)

    return (
        overlap_end - overlap_start
        if overlap_end > overlap_start
        else None
    )
```

### Sınıf Dokümantasyonu

```python
class ShiftManager:
    """Vardiya yönetimi için ana sınıf.

    Bu sınıf vardiyaların oluşturulması, güncellenmesi ve
    çakışmaların kontrolü gibi temel vardiya operasyonlarını yönetir.

    Attributes:
        database: Veritabanı bağlantısı
        cache: Önbellek yöneticisi
        conflict_checker: Vardiya çakışma kontrolcüsü

    Example:
        ```python
        manager = ShiftManager(database, cache)
        shift = await manager.create_shift(
            start_time=datetime.now(),
            end_time=datetime.now() + timedelta(hours=8),
            staff_id=1
        )
        ```
    """
```

## 4. Test Standartları

### Birim Test Örneği

```python
# tests/test_shift_manager.py
import pytest
from datetime import datetime, timedelta
from app.core.services import ShiftManager
from app.core.models import Shift

class TestShiftManager:
    @pytest.fixture
    def manager(self):
        return ShiftManager(MockDatabase(), MockCache())

    @pytest.mark.asyncio
    async def test_create_shift(self, manager):
        # Arrange
        start_time = datetime.now()
        end_time = start_time + timedelta(hours=8)
        staff_id = 1

        # Act
        shift = await manager.create_shift(
            start_time=start_time,
            end_time=end_time,
            staff_id=staff_id
        )

        # Assert
        assert shift.start_time == start_time
        assert shift.end_time == end_time
        assert shift.staff_id == staff_id

    @pytest.mark.asyncio
    async def test_overlapping_shifts(self, manager):
        # Arrange
        shift1 = Shift(
            start_time=datetime.now(),
            end_time=datetime.now() + timedelta(hours=4),
            staff_id=1
        )
        shift2 = Shift(
            start_time=datetime.now() + timedelta(hours=2),
            end_time=datetime.now() + timedelta(hours=6),
            staff_id=1
        )

        # Act & Assert
        with pytest.raises(ValueError):
            await manager.validate_shift_overlap(shift1, shift2)
```

## 5. Hata Yönetimi

### Özel Hatalar

```python
class MedScheduleError(Exception):
    """Temel hata sınıfı."""
    pass

class ValidationError(MedScheduleError):
    """Veri doğrulama hatası."""
    pass

class ResourceNotFoundError(MedScheduleError):
    """Kaynak bulunamadı hatası."""
    pass

class DatabaseError(MedScheduleError):
    """Veritabanı hatası."""
    pass

def handle_errors(func):
    """Hata yakalama dekoratörü."""
    @wraps(func)
    async def wrapper(*args, **kwargs):
        try:
            return await func(*args, **kwargs)
        except ValidationError as e:
            raise HTTPException(
                status_code=400,
                detail=str(e)
            )
        except ResourceNotFoundError as e:
            raise HTTPException(
                status_code=404,
                detail=str(e)
            )
        except DatabaseError as e:
            raise HTTPException(
                status_code=500,
                detail="Database error occurred"
            )
        except Exception as e:
            logger.error(f"Unexpected error: {e}")
            raise HTTPException(
                status_code=500,
                detail="Internal server error"
            )
    return wrapper
```

## 6. Performans Optimizasyonu

### Veritabanı Sorguları

```python
class UserRepository:
    def __init__(self, database: Database):
        self.database = database

    async def get_users_with_shifts(
        self,
        department_id: int
    ) -> List[UserWithShifts]:
        """Kullanıcıları vardiyalarıyla birlikte getirir.

        N+1 sorgu probleminden kaçınmak için JOIN kullanır.
        """
        query = """
            SELECT 
                u.id,
                u.name,
                u.email,
                s.id as shift_id,
                s.start_time,
                s.end_time
            FROM users u
            LEFT JOIN shifts s ON u.id = s.user_id
            WHERE u.department_id = :department_id
            ORDER BY u.id, s.start_time
        """
        
        rows = await self.database.fetch_all(
            query,
            {"department_id": department_id}
        )
        
        return self._group_results(rows)

    def _group_results(self, rows: List[Record]) -> List[UserWithShifts]:
        """Sorgu sonuçlarını gruplar."""
        users = {}
        for row in rows:
            if row["id"] not in users:
                users[row["id"]] = UserWithShifts(
                    id=row["id"],
                    name=row["name"],
                    email=row["email"],
                    shifts=[]
                )
            
            if row["shift_id"]:
                users[row["id"]].shifts.append(
                    Shift(
                        id=row["shift_id"],
                        start_time=row["start_time"],
                        end_time=row["end_time"]
                    )
                )
        
        return list(users.values())
```

### Önbellekleme

```python
class CacheManager:
    def __init__(self, redis: Redis):
        self.redis = redis
        self.default_ttl = 3600  # 1 saat

    async def get_or_set(
        self,
        key: str,
        fetch_func: Callable,
        ttl: Optional[int] = None
    ) -> Any:
        """Önbellekten veri getirir veya yoksa oluşturur."""
        cached = await self.redis.get(key)
        if cached:
            return json.loads(cached)
        
        value = await fetch_func()
        await self.redis.set(
            key,
            json.dumps(value),
            ex=ttl or self.default_ttl
        )
        return value

    async def invalidate_pattern(self, pattern: str):
        """Belirli bir desene uyan tüm önbellek girişlerini siler."""
        keys = await self.redis.keys(pattern)
        if keys:
            await self.redis.delete(*keys)
```

## 7. Güvenlik Pratikleri

### Güvenli Veri İşleme

```python
class DataSanitizer:
    @staticmethod
    def sanitize_input(data: str) -> str:
        """Kullanıcı girdisini temizler."""
        return bleach.clean(data)

    @staticmethod
    def validate_email(email: str) -> bool:
        """Email adresini doğrular."""
        try:
            return bool(email_validator.validate_email(email))
        except email_validator.EmailNotValidError:
            return False

class PasswordManager:
    def __init__(self):
        self.pwd_context = CryptContext(
            schemes=["bcrypt"],
            deprecated="auto"
        )

    def hash_password(self, password: str) -> str:
        """Parolayı güvenli bir şekilde hashler."""
        return self.pwd_context.hash(password)

    def verify_password(
        self,
        plain_password: str,
        hashed_password: str
    ) -> bool:
        """Parolayı doğrular."""
        return self.pwd_context.verify(
            plain_password,
            hashed_password
        )
```

## 8. Kod İnceleme Kontrol Listesi

### Genel Kontroller

1. Kod stili kurallara uygun mu?
2. Fonksiyonlar ve sınıflar doğru dokümante edilmiş mi?
3. Birim testler yazılmış mı?
4. Hata yönetimi uygun şekilde yapılmış mı?
5. Güvenlik açıkları kontrol edilmiş mi?
6. Performans optimizasyonları yapılmış mı?
7. Kod tekrarı var mı?
8. Değişken ve fonksiyon isimleri anlamlı mı?

### Pull Request Şablonu

```markdown
## Değişiklik Açıklaması
<!-- Yapılan değişiklikleri kısaca açıklayın -->

## Kontrol Listesi
- [ ] Kod stili kurallara uygun
- [ ] Birim testler yazıldı ve başarılı
- [ ] Dokümantasyon güncellendi
- [ ] Performans etkileri değerlendirildi
- [ ] Güvenlik kontrolleri yapıldı

## Test Sonuçları
<!-- Test sonuçlarını ekleyin -->

## Ek Notlar
<!-- Varsa ek notları ekleyin -->
```

## 9. Sürekli İyileştirme

### Kod Kalitesi Metrikleri

```python
class CodeQualityMetrics:
    def __init__(self):
        self.metrics = {
            'test_coverage': 0,
            'code_complexity': 0,
            'maintainability_index': 0,
            'technical_debt': 0
        }

    def calculate_metrics(self, codebase_path: str):
        """Kod kalitesi metriklerini hesaplar."""
        # Test coverage
        coverage_result = coverage.Coverage()
        coverage_result.load()
        self.metrics['test_coverage'] = coverage_result.report()

        # Code complexity
        complexity = 0
        for path in glob.glob(f"{codebase_path}/**/*.py"):
            complexity += self._calculate_complexity(path)
        self.metrics['code_complexity'] = complexity

        return self.metrics

    def _calculate_complexity(self, file_path: str) -> int:
        """Dosya karmaşıklığını hesaplar."""
        # McCabe complexity calculation
        pass
```

## Sonuç

Bu kod kalitesi ve standartları dokümanı, MedSchedule projesinin kod kalitesi gereksinimlerini detaylandırır. Yapay zeka, bu standartları kullanarak yüksek kaliteli ve sürdürülebilir bir kod tabanı oluşturmalıdır. Standartlar, modern yazılım geliştirme pratiklerini ve en iyi uygulamaları benimser.
