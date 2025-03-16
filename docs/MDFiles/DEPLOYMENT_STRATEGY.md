# MedSchedule Dağıtım Stratejisi

Bu doküman, MedSchedule projesinin dağıtım stratejisini ve altyapı konfigürasyonunu detaylandırır.

## Altyapı Mimarisi

### Sistem Mimarisi Diyagramı

```
                                    [CDN]
                                      ↑
[Client Browsers] ←→ [Load Balancer] ←→ [Web Servers] ←→ [Application Servers] ←→ [Database Servers]
                                      ↑                           ↑
                                  [Cache]                    [Message Queue]
                                                                 ↑
                                                         [Background Workers]
```

### Bileşenler

#### Frontend
- React SPA
- Nginx web sunucusu
- CloudFront CDN
- Redis önbellek

#### Backend
- FastAPI uygulama sunucusu
- Gunicorn WSGI sunucusu
- Uvicorn ASGI sunucusu
- RabbitMQ mesaj kuyruğu
- Celery worker'ları

#### Veritabanı
- PostgreSQL ana veritabanı
- TimescaleDB zaman serisi verileri için
- PgBouncer bağlantı havuzu

#### Monitoring
- Prometheus metrik toplama
- Grafana görselleştirme
- ELK Stack log yönetimi
- Sentry hata izleme

## Docker Konfigürasyonu

### Base Image
```dockerfile
# base.Dockerfile
FROM python:3.9-slim

# Sistem bağımlılıkları
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Python bağımlılıkları
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Uygulama kullanıcısı
RUN useradd -m appuser
USER appuser

# Çalışma dizini
WORKDIR /app
```

### Backend Service
```dockerfile
# backend.Dockerfile
FROM medschedule-base:latest

# Uygulama kodunu kopyala
COPY --chown=appuser:appuser . .

# Port
EXPOSE 8000

# Sağlık kontrolü
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Çalıştır
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "--worker-class", "uvicorn.workers.UvicornWorker", "app.main:app"]
```

### Frontend Service
```dockerfile
# frontend.Dockerfile
FROM node:16-alpine as builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### Worker Service
```dockerfile
# worker.Dockerfile
FROM medschedule-base:latest

COPY --chown=appuser:appuser . .

CMD ["celery", "-A", "app.tasks", "worker", "--loglevel=info"]
```

## Docker Compose Konfigürasyonu

```yaml
# docker-compose.yml
version: '3.8'

services:
  backend:
    build:
      context: .
      dockerfile: backend.Dockerfile
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/medschedule
      - REDIS_URL=redis://cache:6379/0
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - db
      - cache
      - rabbitmq
    networks:
      - backend-net
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure

  frontend:
    build:
      context: ./frontend
      dockerfile: frontend.Dockerfile
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - frontend-net
    deploy:
      replicas: 2

  worker:
    build:
      context: .
      dockerfile: worker.Dockerfile
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/medschedule
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - db
      - rabbitmq
    networks:
      - backend-net
    deploy:
      replicas: 2

  db:
    image: timescale/timescaledb:latest-pg13
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=medschedule
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend-net
    deploy:
      placement:
        constraints:
          - node.role == manager

  cache:
    image: redis:6-alpine
    volumes:
      - cache-data:/data
    networks:
      - backend-net

  rabbitmq:
    image: rabbitmq:3-management
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    networks:
      - backend-net

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    networks:
      - monitoring-net

  grafana:
    image: grafana/grafana
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - monitoring-net

networks:
  frontend-net:
  backend-net:
  monitoring-net:

volumes:
  db-data:
  cache-data:
  rabbitmq-data:
  prometheus-data:
  grafana-data:
```

## Kubernetes Konfigürasyonu

### Backend Deployment
```yaml
# backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: medschedule-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: medschedule-backend
  template:
    metadata:
      labels:
        app: medschedule-backend
    spec:
      containers:
      - name: backend
        image: medschedule-backend:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        - name: REDIS_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: redis_url
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Frontend Deployment
```yaml
# frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: medschedule-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: medschedule-frontend
  template:
    metadata:
      labels:
        app: medschedule-frontend
    spec:
      containers:
      - name: frontend
        image: medschedule-frontend:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
```

### Database StatefulSet
```yaml
# database-statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: medschedule-db
spec:
  serviceName: medschedule-db
  replicas: 1
  selector:
    matchLabels:
      app: medschedule-db
  template:
    metadata:
      labels:
        app: medschedule-db
    spec:
      containers:
      - name: postgres
        image: timescale/timescaledb:latest-pg13
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
        volumeMounts:
        - name: db-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: db-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

## CI/CD Pipeline

### GitHub Actions Workflow
```yaml
# .github/workflows/deploy.yml
name: Deploy Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_HUB_USERNAME }}
        password: ${{ secrets.DOCKER_HUB_TOKEN }}

    - name: Build and push Backend
      uses: docker/build-push-action@v2
      with:
        context: .
        file: ./backend.Dockerfile
        push: true
        tags: medschedule/backend:latest

    - name: Build and push Frontend
      uses: docker/build-push-action@v2
      with:
        context: ./frontend
        file: ./frontend.Dockerfile
        push: true
        tags: medschedule/frontend:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    - name: Set up kubectl
      uses: azure/k8s-set-context@v1
      with:
        kubeconfig: ${{ secrets.KUBE_CONFIG }}

    - name: Deploy to Kubernetes
      run: |
        kubectl apply -f k8s/
        kubectl rollout status deployment/medschedule-backend
        kubectl rollout status deployment/medschedule-frontend
```

## Monitoring ve Logging

### Prometheus Konfigürasyonu
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'medschedule-backend'
    static_configs:
      - targets: ['backend:8000']

  - job_name: 'medschedule-frontend'
    static_configs:
      - targets: ['frontend:80']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

### Grafana Dashboard
```json
{
  "dashboard": {
    "id": null,
    "title": "MedSchedule Overview",
    "panels": [
      {
        "title": "HTTP Request Rate",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{path}}"
          }
        ]
      },
      {
        "title": "Response Time",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          }
        ]
      }
    ]
  }
}
```

### Fluentd Konfigürasyonu
```yaml
# fluentd.conf
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<match **>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  logstash_prefix medschedule
  <buffer>
    @type file
    path /var/log/fluentd-buffers/medschedule
    flush_mode interval
    retry_type exponential_backoff
    flush_interval 5s
    retry_forever false
    retry_max_interval 30
    chunk_limit_size 2M
    queue_limit_length 8
    overflow_action block
  </buffer>
</match>
```

## Yedekleme Stratejisi

### Veritabanı Yedekleme
```bash
#!/bin/bash
# backup.sh

# Değişkenler
BACKUP_DIR="/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
DB_CONTAINER="medschedule-db"
DB_NAME="medschedule"
DB_USER="user"

# Yedek al
docker exec $DB_CONTAINER pg_dump -U $DB_USER $DB_NAME | gzip > $BACKUP_DIR/db_backup_$TIMESTAMP.sql.gz

# Eski yedekleri temizle (30 günden eski)
find $BACKUP_DIR -name "db_backup_*.sql.gz" -mtime +30 -delete

# S3'e yükle
aws s3 cp $BACKUP_DIR/db_backup_$TIMESTAMP.sql.gz s3://medschedule-backups/
```

### Cron Job
```yaml
# backup-cronjob.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: db-backup
spec:
  schedule: "0 1 * * *"  # Her gün saat 01:00'de
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: medschedule/backup:latest
            command: ["/backup.sh"]
            volumeMounts:
            - name: backup-volume
              mountPath: /backups
          volumes:
          - name: backup-volume
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

## Güvenlik Konfigürasyonu

### Network Policies
```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: medschedule-backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: medschedule-frontend
    ports:
    - protocol: TCP
      port: 8000
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: medschedule-db
    ports:
    - protocol: TCP
      port: 5432
```

### SSL/TLS Konfigürasyonu
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: medschedule-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - medschedule.com
    secretName: medschedule-tls
  rules:
  - host: medschedule.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: medschedule-backend
            port:
              number: 8000
      - path: /
        pathType: Prefix
        backend:
          service:
            name: medschedule-frontend
            port:
              number: 80
```

## Ölçeklendirme Stratejisi

### Horizontal Pod Autoscaler
```yaml
# hpa.yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: medschedule-backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## Felaket Kurtarma Planı

1. **Veri Yedekleme**
   - Günlük tam yedekleme
   - Saatlik artımlı yedekleme
   - Yedeklerin farklı bölgelerde depolanması

2. **Yüksek Erişilebilirlik**
   - Multi-AZ dağıtım
   - Otomatik failover
   - Load balancer kullanımı

3. **Kurtarma Prosedürleri**
   - RTO (Recovery Time Objective): 1 saat
   - RPO (Recovery Point Objective): 5 dakika
   - Otomatik kurtarma scriptleri

## Sonuç

Bu dağıtım stratejisi, MedSchedule projesinin dağıtım ve altyapı gereksinimlerini detaylandırır. Yapay zeka, bu stratejiyi kullanarak güvenli, ölçeklenebilir ve yüksek erişilebilir bir sistem kurmalıdır. Strateji, modern DevOps pratiklerini ve cloud-native yaklaşımları benimser.
