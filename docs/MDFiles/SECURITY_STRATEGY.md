# MedSchedule Güvenlik Stratejisi

Bu doküman, MedSchedule projesinin güvenlik stratejisini ve güvenlik önlemlerini detaylandırır.

## 1. Kimlik Doğrulama ve Yetkilendirme

### JWT Implementasyonu

```python
# security/jwt.py
from datetime import datetime, timedelta
from jose import JWTError, jwt
from passlib.context import CryptContext

SECRET_KEY = "your-secret-key"  # Üretim ortamında güvenli bir şekilde saklanmalı
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

def verify_token(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        return None

def get_password_hash(password: str):
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str):
    return pwd_context.verify(plain_password, hashed_password)
```

### Rol Tabanlı Erişim Kontrolü (RBAC)

```python
# security/rbac.py
from enum import Enum
from typing import List
from fastapi import HTTPException, Security
from fastapi.security import OAuth2PasswordBearer

class Role(str, Enum):
    ADMIN = "admin"
    MANAGER = "manager"
    STAFF = "staff"
    VIEWER = "viewer"

class Permission(str, Enum):
    READ_SCHEDULE = "read:schedule"
    WRITE_SCHEDULE = "write:schedule"
    MANAGE_STAFF = "manage:staff"
    MANAGE_DEPARTMENT = "manage:department"
    VIEW_REPORTS = "view:reports"
    MANAGE_SYSTEM = "manage:system"

ROLE_PERMISSIONS = {
    Role.ADMIN: [p.value for p in Permission],
    Role.MANAGER: [
        Permission.READ_SCHEDULE,
        Permission.WRITE_SCHEDULE,
        Permission.MANAGE_STAFF,
        Permission.VIEW_REPORTS
    ],
    Role.STAFF: [
        Permission.READ_SCHEDULE,
        Permission.VIEW_REPORTS
    ],
    Role.VIEWER: [
        Permission.READ_SCHEDULE
    ]
}

def check_permission(user_role: Role, required_permission: Permission):
    if required_permission.value not in ROLE_PERMISSIONS[user_role]:
        raise HTTPException(
            status_code=403,
            detail="Permission denied"
        )
```

## 2. Veri Güvenliği

### Veri Şifreleme

```python
# security/encryption.py
from cryptography.fernet import Fernet
from base64 import b64encode
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC

class DataEncryption:
    def __init__(self, key: bytes):
        self.fernet = Fernet(key)
    
    def encrypt(self, data: str) -> bytes:
        return self.fernet.encrypt(data.encode())
    
    def decrypt(self, token: bytes) -> str:
        return self.fernet.decrypt(token).decode()
    
    @staticmethod
    def generate_key(password: str, salt: bytes) -> bytes:
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=32,
            salt=salt,
            iterations=100000,
        )
        key = b64encode(kdf.derive(password.encode()))
        return key
```

### Hassas Veri Maskeleme

```python
# security/masking.py
import re

class DataMasking:
    @staticmethod
    def mask_email(email: str) -> str:
        pattern = r'(^[^@]{3})[^@]*(@.*)'
        return re.sub(pattern, r'\1***\2', email)
    
    @staticmethod
    def mask_phone(phone: str) -> str:
        return re.sub(r'(\d{3})\d+(\d{2})', r'\1****\2', phone)
    
    @staticmethod
    def mask_national_id(national_id: str) -> str:
        return re.sub(r'(\d{3})\d+(\d{2})', r'\1******\2', national_id)
```

## 3. API Güvenliği

### Rate Limiting

```python
# security/rate_limit.py
from fastapi import HTTPException
from redis import Redis
from datetime import datetime

class RateLimiter:
    def __init__(self, redis_client: Redis):
        self.redis = redis_client
        self.rate_limit = 100  # requests per minute
        self.window = 60  # seconds
    
    async def check_rate_limit(self, client_id: str):
        current = datetime.now().timestamp()
        key = f"rate_limit:{client_id}"
        
        pipeline = self.redis.pipeline()
        pipeline.zremrangebyscore(key, 0, current - self.window)
        pipeline.zcard(key)
        pipeline.zadd(key, {str(current): current})
        pipeline.expire(key, self.window)
        results = pipeline.execute()
        
        request_count = results[1]
        
        if request_count > self.rate_limit:
            raise HTTPException(
                status_code=429,
                detail="Too many requests"
            )
```

### API Güvenlik Başlıkları

```python
# security/headers.py
from fastapi import FastAPI
from starlette.middleware.cors import CORSMiddleware
from starlette.middleware.base import BaseHTTPMiddleware

security_headers = {
    "X-Frame-Options": "DENY",
    "X-Content-Type-Options": "nosniff",
    "X-XSS-Protection": "1; mode=block",
    "Content-Security-Policy": "default-src 'self'",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
    "Referrer-Policy": "strict-origin-when-cross-origin"
}

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        for key, value in security_headers.items():
            response.headers[key] = value
        return response

def configure_security(app: FastAPI):
    app.add_middleware(SecurityHeadersMiddleware)
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["https://medschedule.com"],
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )
```

## 4. Güvenlik İzleme ve Denetim

### Güvenlik Logları

```python
# security/audit.py
from datetime import datetime
from typing import Optional
from pydantic import BaseModel
from sqlalchemy.orm import Session

class AuditLog(BaseModel):
    user_id: str
    action: str
    resource: str
    ip_address: str
    details: Optional[dict]
    timestamp: datetime = datetime.utcnow()

class AuditLogger:
    def __init__(self, db: Session):
        self.db = db
    
    async def log_action(self, audit_log: AuditLog):
        self.db.add(audit_log)
        await self.db.commit()
    
    async def get_user_actions(self, user_id: str, start_date: datetime, end_date: datetime):
        return self.db.query(AuditLog).filter(
            AuditLog.user_id == user_id,
            AuditLog.timestamp.between(start_date, end_date)
        ).all()
```

### Güvenlik İhlali Tespiti

```python
# security/intrusion_detection.py
from typing import List
from datetime import datetime, timedelta
from redis import Redis

class IntrusionDetection:
    def __init__(self, redis_client: Redis):
        self.redis = redis_client
        self.failed_login_threshold = 5
        self.suspicious_ip_threshold = 10
        self.window = 300  # 5 minutes
    
    async def check_failed_logins(self, user_id: str, ip_address: str):
        key = f"failed_login:{user_id}"
        current = datetime.now().timestamp()
        
        # Son başarısız girişleri temizle
        self.redis.zremrangebyscore(key, 0, current - self.window)
        
        # Yeni başarısız girişi ekle
        self.redis.zadd(key, {str(current): current})
        self.redis.expire(key, self.window)
        
        # Başarısız giriş sayısını kontrol et
        failed_attempts = self.redis.zcard(key)
        
        if failed_attempts >= self.failed_login_threshold:
            await self.block_user(user_id)
            await self.block_ip(ip_address)
            return True
        return False
    
    async def block_user(self, user_id: str):
        self.redis.setex(f"blocked_user:{user_id}", 3600, 1)  # 1 saat bloke
    
    async def block_ip(self, ip_address: str):
        self.redis.setex(f"blocked_ip:{ip_address}", 3600, 1)  # 1 saat bloke
    
    async def is_blocked(self, user_id: str, ip_address: str) -> bool:
        return (
            self.redis.exists(f"blocked_user:{user_id}") or
            self.redis.exists(f"blocked_ip:{ip_address}")
        )
```

## 5. Güvenli Dosya İşlemleri

### Güvenli Dosya Yükleme

```python
# security/file_upload.py
import magic
from pathlib import Path
from fastapi import UploadFile, HTTPException

class SecureFileUpload:
    ALLOWED_EXTENSIONS = {'.pdf', '.jpg', '.jpeg', '.png', '.doc', '.docx'}
    MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB
    
    @staticmethod
    async def validate_file(file: UploadFile):
        # Dosya boyutu kontrolü
        file_size = 0
        chunk_size = 8192
        
        while True:
            chunk = await file.read(chunk_size)
            if not chunk:
                break
            file_size += len(chunk)
            if file_size > SecureFileUpload.MAX_FILE_SIZE:
                raise HTTPException(
                    status_code=400,
                    detail="File too large"
                )
        
        await file.seek(0)
        
        # Dosya uzantısı kontrolü
        file_ext = Path(file.filename).suffix.lower()
        if file_ext not in SecureFileUpload.ALLOWED_EXTENSIONS:
            raise HTTPException(
                status_code=400,
                detail="File type not allowed"
            )
        
        # MIME type kontrolü
        mime = magic.Magic(mime=True)
        file_type = mime.from_buffer(await file.read(1024))
        await file.seek(0)
        
        allowed_mime_types = {
            'application/pdf',
            'image/jpeg',
            'image/png',
            'application/msword',
            'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
        }
        
        if file_type not in allowed_mime_types:
            raise HTTPException(
                status_code=400,
                detail="Invalid file content"
            )
```

## 6. Güvenlik Konfigürasyonu

### Güvenlik Ayarları

```python
# config/security_settings.py
from pydantic import BaseSettings

class SecuritySettings(BaseSettings):
    # JWT Ayarları
    JWT_SECRET_KEY: str
    JWT_ALGORITHM: str = "HS256"
    JWT_ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    JWT_REFRESH_TOKEN_EXPIRE_DAYS: int = 7
    
    # Parola Politikası
    PASSWORD_MIN_LENGTH: int = 12
    PASSWORD_REQUIRE_UPPERCASE: bool = True
    PASSWORD_REQUIRE_LOWERCASE: bool = True
    PASSWORD_REQUIRE_NUMBERS: bool = True
    PASSWORD_REQUIRE_SPECIAL: bool = True
    
    # Rate Limiting
    RATE_LIMIT_PER_MINUTE: int = 100
    RATE_LIMIT_BURST: int = 200
    
    # Session Güvenliği
    SESSION_EXPIRE_MINUTES: int = 60
    SESSION_REFRESH_EACH_REQUEST: bool = True
    
    # CORS Ayarları
    CORS_ORIGINS: list = ["https://medschedule.com"]
    CORS_ALLOW_CREDENTIALS: bool = True
    
    # SSL/TLS Ayarları
    SSL_CERT_FILE: str = "/etc/ssl/certs/medschedule.crt"
    SSL_KEY_FILE: str = "/etc/ssl/private/medschedule.key"
    
    # Dosya Yükleme
    UPLOAD_MAX_SIZE: int = 10 * 1024 * 1024  # 10MB
    ALLOWED_UPLOAD_EXTENSIONS: list = [".pdf", ".jpg", ".jpeg", ".png"]
    
    class Config:
        env_file = ".env"
```

## 7. Güvenlik Test Senaryoları

### Güvenlik Testleri

```python
# tests/security/test_authentication.py
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_login_rate_limit():
    # Rate limit aşılana kadar login dene
    for _ in range(101):
        response = client.post(
            "/auth/login",
            json={"username": "test", "password": "test"}
        )
    
    assert response.status_code == 429
    assert "Too many requests" in response.json()["detail"]

def test_password_complexity():
    # Zayıf parola ile kayıt dene
    response = client.post(
        "/auth/register",
        json={
            "username": "test",
            "password": "weak"
        }
    )
    
    assert response.status_code == 400
    assert "Password does not meet complexity requirements" in response.json()["detail"]

def test_sql_injection():
    # SQL injection denemesi
    response = client.post(
        "/auth/login",
        json={
            "username": "' OR '1'='1",
            "password": "' OR '1'='1"
        }
    )
    
    assert response.status_code == 401
    assert "Invalid credentials" in response.json()["detail"]
```

## 8. Güvenlik Politikaları

### Parola Politikası

```python
# security/password_policy.py
import re
from pydantic import BaseModel, validator

class PasswordPolicy:
    MIN_LENGTH = 12
    REQUIRE_UPPERCASE = True
    REQUIRE_LOWERCASE = True
    REQUIRE_NUMBERS = True
    REQUIRE_SPECIAL = True
    SPECIAL_CHARS = "!@#$%^&*()_+-=[]{}|;:,.<>?"
    
    @classmethod
    def validate_password(cls, password: str) -> bool:
        if len(password) < cls.MIN_LENGTH:
            return False
        
        if cls.REQUIRE_UPPERCASE and not re.search(r'[A-Z]', password):
            return False
        
        if cls.REQUIRE_LOWERCASE and not re.search(r'[a-z]', password):
            return False
        
        if cls.REQUIRE_NUMBERS and not re.search(r'[0-9]', password):
            return False
        
        if cls.REQUIRE_SPECIAL and not any(c in cls.SPECIAL_CHARS for c in password):
            return False
        
        return True

class PasswordChange(BaseModel):
    current_password: str
    new_password: str
    
    @validator('new_password')
    def validate_new_password(cls, v):
        if not PasswordPolicy.validate_password(v):
            raise ValueError("Password does not meet security requirements")
        return v
```

### Oturum Politikası

```python
# security/session_policy.py
from datetime import datetime, timedelta
from typing import Optional
from fastapi import HTTPException, Cookie
from redis import Redis

class SessionPolicy:
    def __init__(self, redis_client: Redis):
        self.redis = redis_client
        self.session_timeout = timedelta(minutes=60)
    
    async def create_session(self, user_id: str, ip_address: str) -> str:
        session_id = self.generate_session_id()
        session_data = {
            "user_id": user_id,
            "ip_address": ip_address,
            "created_at": datetime.utcnow().isoformat(),
            "last_activity": datetime.utcnow().isoformat()
        }
        
        self.redis.setex(
            f"session:{session_id}",
            int(self.session_timeout.total_seconds()),
            str(session_data)
        )
        
        return session_id
    
    async def validate_session(
        self,
        session_id: Optional[str] = Cookie(None)
    ) -> dict:
        if not session_id:
            raise HTTPException(
                status_code=401,
                detail="No session found"
            )
        
        session_data = self.redis.get(f"session:{session_id}")
        if not session_data:
            raise HTTPException(
                status_code=401,
                detail="Invalid or expired session"
            )
        
        # Oturum süresini yenile
        self.redis.expire(
            f"session:{session_id}",
            int(self.session_timeout.total_seconds())
        )
        
        return eval(session_data)
```

## 9. Güvenlik İzleme ve Raporlama

### Güvenlik Metrikleri

```python
# monitoring/security_metrics.py
from prometheus_client import Counter, Histogram
from datetime import datetime

# Metrikler
failed_login_attempts = Counter(
    'failed_login_attempts_total',
    'Total number of failed login attempts',
    ['username', 'ip_address']
)

successful_login_attempts = Counter(
    'successful_login_attempts_total',
    'Total number of successful login attempts',
    ['username', 'ip_address']
)

request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration in seconds',
    ['method', 'endpoint']
)

blocked_requests = Counter(
    'blocked_requests_total',
    'Total number of blocked requests',
    ['reason', 'ip_address']
)

class SecurityMetrics:
    @staticmethod
    def record_login_attempt(username: str, ip_address: str, success: bool):
        if success:
            successful_login_attempts.labels(
                username=username,
                ip_address=ip_address
            ).inc()
        else:
            failed_login_attempts.labels(
                username=username,
                ip_address=ip_address
            ).inc()
    
    @staticmethod
    def record_blocked_request(reason: str, ip_address: str):
        blocked_requests.labels(
            reason=reason,
            ip_address=ip_address
        ).inc()
```

## 10. Acil Durum Müdahale Planı

### Güvenlik İhlali Müdahale

```python
# security/incident_response.py
from enum import Enum
from datetime import datetime
from typing import List, Optional
from pydantic import BaseModel
from fastapi import BackgroundTasks

class IncidentSeverity(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    CRITICAL = "critical"

class SecurityIncident(BaseModel):
    id: str
    severity: IncidentSeverity
    type: str
    description: str
    affected_users: List[str]
    affected_systems: List[str]
    detection_time: datetime
    resolution_time: Optional[datetime]
    actions_taken: List[str]

class IncidentResponse:
    def __init__(self, background_tasks: BackgroundTasks):
        self.background_tasks = background_tasks
    
    async def handle_incident(self, incident: SecurityIncident):
        # 1. Acil durum ekibini bilgilendir
        self.background_tasks.add_task(
            self.notify_security_team,
            incident
        )
        
        # 2. Etkilenen sistemleri izole et
        if incident.severity in [IncidentSeverity.HIGH, IncidentSeverity.CRITICAL]:
            await self.isolate_affected_systems(incident.affected_systems)
        
        # 3. Etkilenen kullanıcıları bilgilendir
        self.background_tasks.add_task(
            self.notify_affected_users,
            incident.affected_users
        )
        
        # 4. Olayı kaydet
        await self.log_incident(incident)
    
    async def isolate_affected_systems(self, systems: List[str]):
        # Sistemleri izole et
        pass
    
    async def notify_security_team(self, incident: SecurityIncident):
        # Güvenlik ekibini bilgilendir
        pass
    
    async def notify_affected_users(self, users: List[str]):
        # Kullanıcıları bilgilendir
        pass
    
    async def log_incident(self, incident: SecurityIncident):
        # Olayı kaydet
        pass
```

## Sonuç

Bu güvenlik stratejisi, MedSchedule projesinin güvenlik gereksinimlerini ve önlemlerini detaylandırır. Yapay zeka, bu stratejiyi kullanarak güvenli bir sistem implementasyon etmelidir. Strateji, modern güvenlik pratiklerini ve standartlarını benimser ve düzenli olarak güncellenmelidir.
