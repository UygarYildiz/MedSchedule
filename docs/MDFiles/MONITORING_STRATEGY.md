# MedSchedule İzleme ve Analiz Stratejisi

Bu doküman, MedSchedule projesinin izleme, loglama ve analiz stratejisini detaylandırır.

## 1. Metrik Toplama

### Sistem Metrikleri

```python
# monitoring/system_metrics.py
from prometheus_client import Counter, Gauge, Histogram
import psutil

class SystemMetrics:
    def __init__(self):
        self.cpu_usage = Gauge('cpu_usage_percent', 'CPU usage in percent')
        self.memory_usage = Gauge('memory_usage_bytes', 'Memory usage in bytes')
        self.disk_usage = Gauge('disk_usage_percent', 'Disk usage in percent')
        
    def collect_metrics(self):
        self.cpu_usage.set(psutil.cpu_percent())
        self.memory_usage.set(psutil.virtual_memory().used)
        self.disk_usage.set(psutil.disk_usage('/').percent)
```

### Uygulama Metrikleri

```python
# monitoring/app_metrics.py
from prometheus_client import Counter, Histogram
from functools import wraps
import time

# HTTP istekleri
request_count = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

# İstek süresi
request_latency = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

def monitor_endpoint(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):
        start_time = time.time()
        try:
            response = await func(*args, **kwargs)
            request_count.labels(
                method=request.method,
                endpoint=request.url.path,
                status=response.status_code
            ).inc()
            return response
        finally:
            request_latency.labels(
                method=request.method,
                endpoint=request.url.path
            ).observe(time.time() - start_time)
    return wrapper
```

### İş Metrikleri

```python
# monitoring/business_metrics.py
from prometheus_client import Counter, Gauge

class BusinessMetrics:
    def __init__(self):
        # Aktif kullanıcılar
        self.active_users = Gauge(
            'active_users_total',
            'Number of active users'
        )
        
        # Vardiya planları
        self.scheduled_shifts = Counter(
            'scheduled_shifts_total',
            'Total number of scheduled shifts',
            ['department']
        )
        
        # Çakışan vardiyalar
        self.shift_conflicts = Counter(
            'shift_conflicts_total',
            'Number of shift conflicts detected'
        )
```

## 2. Loglama Stratejisi

### Log Yapılandırması

```python
# logging/config.py
import logging
from logging.handlers import RotatingFileHandler

def setup_logging():
    # Ana logger
    logger = logging.getLogger('medschedule')
    logger.setLevel(logging.INFO)
    
    # Dosya handler'ı
    file_handler = RotatingFileHandler(
        'logs/medschedule.log',
        maxBytes=10485760,  # 10MB
        backupCount=10
    )
    
    # Format
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    file_handler.setFormatter(formatter)
    
    logger.addHandler(file_handler)
    return logger
```

### Structured Logging

```python
# logging/structured.py
import json
from datetime import datetime

class StructuredLogger:
    def __init__(self, logger):
        self.logger = logger
    
    def log_event(self, event_type: str, **kwargs):
        log_entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'event_type': event_type,
            'data': kwargs
        }
        self.logger.info(json.dumps(log_entry))
```

## 3. Analiz ve Raporlama

### Veri Analizi

```python
# analytics/data_analysis.py
import pandas as pd
from typing import Dict, List

class ShiftAnalytics:
    def analyze_shift_patterns(self, shifts: List[Dict]) -> Dict:
        df = pd.DataFrame(shifts)
        
        return {
            'total_shifts': len(df),
            'shifts_per_department': df.groupby('department').size().to_dict(),
            'avg_shift_duration': df['duration'].mean(),
            'common_conflicts': self._analyze_conflicts(df)
        }
    
    def _analyze_conflicts(self, df: pd.DataFrame) -> Dict:
        # Çakışma analizi
        pass
```

### Performans Analizi

```python
# analytics/performance.py
from typing import List, Dict
import numpy as np

class PerformanceAnalytics:
    def analyze_response_times(self, latencies: List[float]) -> Dict:
        return {
            'avg_response_time': np.mean(latencies),
            'p95_response_time': np.percentile(latencies, 95),
            'p99_response_time': np.percentile(latencies, 99)
        }
```

## 4. Alerting

### Alert Kuralları

```python
# monitoring/alerts.py
from typing import Callable
from datetime import datetime

class AlertRule:
    def __init__(
        self,
        name: str,
        condition: Callable,
        threshold: float,
        window: int = 300  # 5 dakika
    ):
        self.name = name
        self.condition = condition
        self.threshold = threshold
        self.window = window
        self.last_triggered = None
    
    def check(self, value: float) -> bool:
        if self.condition(value, self.threshold):
            now = datetime.utcnow()
            if (not self.last_triggered or 
                (now - self.last_triggered).seconds > self.window):
                self.last_triggered = now
                return True
        return False

class AlertManager:
    def __init__(self):
        self.rules = []
        
    def add_rule(self, rule: AlertRule):
        self.rules.append(rule)
    
    async def check_alerts(self, metrics: dict):
        for rule in self.rules:
            if rule.check(metrics.get(rule.name)):
                await self.send_alert(rule)
    
    async def send_alert(self, rule: AlertRule):
        # Alert gönderme mantığı
        pass
```

## 5. Görselleştirme

### Grafana Dashboard Konfigürasyonu

```json
{
  "dashboard": {
    "id": null,
    "title": "MedSchedule Overview",
    "panels": [
      {
        "title": "System Resources",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "cpu_usage_percent",
            "legendFormat": "CPU Usage"
          },
          {
            "expr": "memory_usage_bytes",
            "legendFormat": "Memory Usage"
          }
        ]
      },
      {
        "title": "HTTP Requests",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ]
      }
    ]
  }
}
```

## 6. İzleme Konfigürasyonu

### Prometheus Konfigürasyonu

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'medschedule'
    static_configs:
      - targets: ['localhost:8000']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
```

### Grafana Konfigürasyonu

```yaml
# grafana.ini
[server]
domain = medschedule.com
root_url = https://medschedule.com/grafana

[security]
admin_user = admin
admin_password = secure_password

[auth]
disable_login_form = false
```

## 7. Performans İzleme

### Endpoint Performansı

```python
# monitoring/performance.py
from fastapi import FastAPI
from prometheus_client import Histogram
import time

class PerformanceMonitor:
    def __init__(self, app: FastAPI):
        self.app = app
        self.endpoint_latency = Histogram(
            'endpoint_latency_seconds',
            'Endpoint latency in seconds',
            ['endpoint', 'method']
        )
    
    def instrument_endpoints(self):
        for route in self.app.routes:
            original_endpoint = route.endpoint
            
            async def instrumented_endpoint(*args, **kwargs):
                start_time = time.time()
                try:
                    return await original_endpoint(*args, **kwargs)
                finally:
                    self.endpoint_latency.labels(
                        endpoint=route.path,
                        method=route.methods[0]
                    ).observe(time.time() - start_time)
            
            route.endpoint = instrumented_endpoint
```

## 8. Sağlık Kontrolleri

### Sağlık Kontrolü Endpoint'i

```python
# monitoring/health.py
from fastapi import FastAPI, HTTPException
from typing import Dict
import psutil

class HealthCheck:
    def __init__(self, app: FastAPI):
        self.app = app
        self.register_endpoints()
    
    def register_endpoints(self):
        @self.app.get("/health")
        async def health_check() -> Dict:
            return {
                "status": "healthy",
                "system": self.check_system(),
                "database": await self.check_database(),
                "cache": await self.check_cache()
            }
    
    def check_system(self) -> Dict:
        return {
            "cpu_usage": psutil.cpu_percent(),
            "memory_usage": psutil.virtual_memory().percent,
            "disk_usage": psutil.disk_usage('/').percent
        }
    
    async def check_database(self) -> Dict:
        # Veritabanı sağlık kontrolü
        pass
    
    async def check_cache(self) -> Dict:
        # Önbellek sağlık kontrolü
        pass
```

## Sonuç

Bu izleme stratejisi, MedSchedule projesinin izleme, loglama ve analiz gereksinimlerini detaylandırır. Yapay zeka, bu stratejiyi kullanarak kapsamlı bir izleme sistemi kurmalıdır. Strateji, modern izleme pratiklerini ve araçlarını benimser.
