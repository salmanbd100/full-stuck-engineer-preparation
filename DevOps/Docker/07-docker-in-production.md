# Docker in Production

## Overview

Running Docker in production requires careful planning for high availability, monitoring, logging, resource management, and deployment strategies. This guide covers production-ready patterns and best practices for enterprise deployments.

## Table of Contents
- [Production Architecture](#production-architecture)
- [Health Checks and Monitoring](#health-checks-and-monitoring)
- [Logging Strategies](#logging-strategies)
- [Resource Management](#resource-management)
- [Deployment Strategies](#deployment-strategies)
- [High Availability](#high-availability)
- [Backup and Disaster Recovery](#backup-and-disaster-recovery)
- [Interview Questions](#interview-questions)

## Production Architecture

### Multi-Tier Architecture

```yaml
version: '3.8'

services:
  # Load Balancer (Entry Point)
  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
    networks:
      - frontend
    depends_on:
      - api
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M

  # Application Tier
  api:
    image: myapp:${VERSION:-latest}
    restart: always
    environment:
      NODE_ENV: production
      DATABASE_URL: postgres://postgres:${DB_PASSWORD}@db:5432/myapp
      REDIS_URL: redis://redis:6379
    networks:
      - frontend
      - backend
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"

  # Data Tier
  db:
    image: postgres:14-alpine
    restart: always
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init:/docker-entrypoint-initdb.d:ro
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G

  # Cache Tier
  redis:
    image: redis:7-alpine
    restart: always
    command: redis-server --appendonly yes --maxmemory 1gb --maxmemory-policy allkeys-lru
    volumes:
      - redis-data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G

  # Background Workers
  worker:
    image: myapp:${VERSION:-latest}
    restart: always
    command: npm run worker
    environment:
      NODE_ENV: production
      REDIS_URL: redis://redis:6379
    networks:
      - backend
    depends_on:
      - redis
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '1'
          memory: 1G

volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
```

### Microservices Architecture

```yaml
version: '3.8'

services:
  # API Gateway
  gateway:
    image: myorg/api-gateway:${VERSION}
    restart: always
    ports:
      - "8080:8080"
    environment:
      USER_SERVICE_URL: http://user-service:3000
      ORDER_SERVICE_URL: http://order-service:3000
      PRODUCT_SERVICE_URL: http://product-service:3000
    networks:
      - frontend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      rollback_config:
        parallelism: 1
        delay: 5s
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3

  # User Service
  user-service:
    image: myorg/user-service:${VERSION}
    restart: always
    environment:
      DATABASE_URL: postgres://user-db:5432/users
      JWT_SECRET: ${JWT_SECRET}
    networks:
      - frontend
      - user-net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  user-db:
    image: postgres:14-alpine
    restart: always
    volumes:
      - user-db-data:/var/lib/postgresql/data
    networks:
      - user-net

  # Order Service
  order-service:
    image: myorg/order-service:${VERSION}
    restart: always
    environment:
      DATABASE_URL: postgres://order-db:5432/orders
      KAFKA_BROKERS: kafka:9092
    networks:
      - frontend
      - order-net

  order-db:
    image: postgres:14-alpine
    restart: always
    volumes:
      - order-db-data:/var/lib/postgresql/data
    networks:
      - order-net

  # Message Queue
  kafka:
    image: confluentinc/cp-kafka:latest
    restart: always
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
    networks:
      - order-net
      - product-net

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    restart: always
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    networks:
      - order-net

volumes:
  user-db-data:
  order-db-data:

networks:
  frontend:
  user-net:
    internal: true
  order-net:
    internal: true
  product-net:
    internal: true
```

## Health Checks and Monitoring

### Container Health Checks

```dockerfile
# In Dockerfile
HEALTHCHECK --interval=30s --timeout=10s --retries=3 --start-period=40s \
  CMD curl -f http://localhost:3000/health || exit 1

# Exit codes:
# 0: healthy
# 1: unhealthy
```

**Health Check Endpoints:**
```javascript
// Node.js health endpoint
app.get('/health', async (req, res) => {
  try {
    // Check database
    await db.query('SELECT 1');

    // Check Redis
    await redis.ping();

    // Check disk space
    const diskSpace = await checkDiskSpace();
    if (diskSpace < 10) {
      throw new Error('Low disk space');
    }

    res.status(200).json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      checks: {
        database: 'ok',
        redis: 'ok',
        diskSpace: 'ok'
      }
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      error: error.message
    });
  }
});

// Liveness probe (is container alive?)
app.get('/healthz', (req, res) => {
  res.status(200).send('OK');
});

// Readiness probe (is container ready for traffic?)
app.get('/ready', async (req, res) => {
  if (await checkDependencies()) {
    res.status(200).send('Ready');
  } else {
    res.status(503).send('Not Ready');
  }
});
```

### Prometheus Metrics

```yaml
services:
  app:
    image: myapp
    ports:
      - "3000:3000"
      - "9090:9090"  # Metrics endpoint

  prometheus:
    image: prom/prometheus:latest
    restart: always
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    ports:
      - "9091:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'

  grafana:
    image: grafana/grafana:latest
    restart: always
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3001:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_PASSWORD}

volumes:
  prometheus-data:
  grafana-data:
```

**prometheus.yml:**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'app'
    static_configs:
      - targets: ['app:9090']

  - job_name: 'docker'
    static_configs:
      - targets: ['cadvisor:8080']
```

### Container Monitoring with cAdvisor

```yaml
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    restart: always
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"
```

## Logging Strategies

### Structured Logging

```javascript
// Use structured logging (JSON format)
const winston = require('winston');

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console()
  ]
});

logger.info('User logged in', {
  userId: user.id,
  email: user.email,
  ip: req.ip,
  userAgent: req.headers['user-agent']
});

// Output:
// {"level":"info","message":"User logged in","timestamp":"2024-01-15T10:30:00.000Z","userId":123,"email":"user@example.com"}
```

### Centralized Logging with ELK Stack

```yaml
version: '3.8'

services:
  app:
    image: myapp
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "environment,service"
    labels:
      logging: "promtail"

  # Elasticsearch
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.5.0
    restart: always
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  # Logstash
  logstash:
    image: docker.elastic.co/logstash/logstash:8.5.0
    restart: always
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

  # Kibana
  kibana:
    image: docker.elastic.co/kibana/kibana:8.5.0
    restart: always
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    depends_on:
      - elasticsearch

  # Filebeat (Log Shipper)
  filebeat:
    image: docker.elastic.co/beats/filebeat:8.5.0
    restart: always
    user: root
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    command: filebeat -e -strict.perms=false
    depends_on:
      - logstash

volumes:
  elasticsearch-data:
```

### Log Rotation

```yaml
services:
  app:
    image: myapp
    logging:
      driver: "json-file"
      options:
        max-size: "10m"      # Max size per file
        max-file: "5"        # Keep 5 files
        compress: "true"     # Compress rotated logs

  # OR use syslog
  api:
    logging:
      driver: "syslog"
      options:
        syslog-address: "tcp://logserver:514"
        tag: "api-service"

  # OR AWS CloudWatch
  web:
    logging:
      driver: "awslogs"
      options:
        awslogs-region: "us-east-1"
        awslogs-group: "myapp"
        awslogs-stream: "web"
```

## Resource Management

### Resource Limits

```yaml
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '2.0'        # Max 2 CPUs
          memory: 2G         # Max 2GB RAM
          pids: 100          # Max 100 processes
        reservations:
          cpus: '1.0'        # Reserve 1 CPU
          memory: 1G         # Reserve 1GB RAM

    # Alternative short syntax (Compose v2)
    cpus: 2
    mem_limit: 2g
    mem_reservation: 1g
```

### OOM (Out of Memory) Handling

```yaml
services:
  app:
    image: myapp
    mem_limit: 2g
    mem_reservation: 1g

    # Prevent OOM killer from killing container
    oom_score_adj: -500

    # Or use memory swap
    memswap_limit: 3g  # Total memory + swap
```

### CPU Quotas

```bash
# Limit container to 50% of one CPU
docker run --cpus=0.5 myapp

# Limit to specific CPU cores
docker run --cpuset-cpus="0,1" myapp

# CPU shares (relative weight)
docker run --cpu-shares=512 myapp  # Half priority (default 1024)
```

## Deployment Strategies

### Blue-Green Deployment

```bash
# Current: blue (v1.0) running
docker-compose -f docker-compose.yml up -d

# Deploy green (v2.0) on different port
VERSION=2.0 PORT=8081 docker-compose -f docker-compose.green.yml up -d

# Test green environment
curl http://localhost:8081/health

# Switch traffic (update nginx/load balancer)
# Update nginx config to point to :8081

# Reload nginx
docker exec nginx nginx -s reload

# Monitor for issues

# If successful: Remove blue
docker-compose -f docker-compose.yml down

# If issues: Rollback by switching nginx back to :8080
```

### Rolling Update

```yaml
services:
  api:
    image: myapp:${VERSION}
    deploy:
      replicas: 4
      update_config:
        parallelism: 1      # Update 1 at a time
        delay: 10s          # Wait 10s between updates
        failure_action: rollback
        monitor: 60s        # Monitor for 60s
        max_failure_ratio: 0.3
      rollback_config:
        parallelism: 1
        delay: 5s
```

### Canary Deployment

```yaml
services:
  # Stable version (90% traffic)
  api-stable:
    image: myapp:1.0
    deploy:
      replicas: 9

  # Canary version (10% traffic)
  api-canary:
    image: myapp:2.0
    deploy:
      replicas: 1

  # Load balancer distributes traffic
  nginx:
    volumes:
      - ./nginx-canary.conf:/etc/nginx/nginx.conf:ro
```

**nginx-canary.conf:**
```nginx
upstream backend {
    server api-stable:3000 weight=9;
    server api-canary:3000 weight=1;
}
```

### Zero-Downtime Deployment

```yaml
services:
  app:
    image: myapp:${VERSION}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 40s

    # Graceful shutdown
    stop_grace_period: 60s
    stop_signal: SIGTERM

    deploy:
      update_config:
        order: start-first  # Start new before stopping old
```

**Application Graceful Shutdown:**
```javascript
// Node.js graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, starting graceful shutdown');

  // Stop accepting new requests
  server.close(() => {
    console.log('HTTP server closed');
  });

  // Close database connections
  await db.close();

  // Close Redis connections
  await redis.quit();

  console.log('Graceful shutdown complete');
  process.exit(0);
});

// Kubernetes/Docker waits for stop_grace_period before SIGKILL
```

## High Availability

### Load Balancing

```yaml
services:
  # HAProxy load balancer
  lb:
    image: haproxy:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
      - "8404:8404"  # Stats page
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro

  # Multiple app instances
  app:
    image: myapp:latest
    restart: always
    deploy:
      replicas: 3
```

**haproxy.cfg:**
```
frontend http_front
    bind *:80
    stats uri /stats
    default_backend http_back

backend http_back
    balance roundrobin
    option httpchk GET /health
    http-check expect status 200
    server app1 app:3000 check
    server app2 app:3000 check
    server app3 app:3000 check
```

### Database Replication

```yaml
services:
  # Primary database
  postgres-primary:
    image: postgres:14-alpine
    restart: always
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_REPLICATION_USER: replicator
      POSTGRES_REPLICATION_PASSWORD: ${REPL_PASSWORD}
    volumes:
      - postgres-primary-data:/var/lib/postgresql/data
      - ./postgres/primary-init.sh:/docker-entrypoint-initdb.d/init.sh

  # Read replica
  postgres-replica:
    image: postgres:14-alpine
    restart: always
    environment:
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - postgres-replica-data:/var/lib/postgresql/data
    command: |
      bash -c "
      until pg_basebackup -h postgres-primary -D /var/lib/postgresql/data/pgdata -U replicator -Fp -Xs -P -R
      do
        sleep 5
      done
      postgres
      "

volumes:
  postgres-primary-data:
  postgres-replica-data:
```

## Backup and Disaster Recovery

### Automated Backups

```yaml
services:
  db:
    image: postgres:14-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data

  # Automated backup service
  backup:
    image: postgres:14-alpine
    restart: always
    environment:
      PGHOST: db
      PGUSER: postgres
      PGPASSWORD: ${DB_PASSWORD}
      BACKUP_DIR: /backups
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      S3_BUCKET: my-backups
    volumes:
      - ./backup.sh:/backup.sh:ro
      - backups:/backups
    command: |
      sh -c "
      while true; do
        /backup.sh
        sleep 86400  # Daily backups
      done
      "

volumes:
  postgres-data:
  backups:
```

**backup.sh:**
```bash
#!/bin/bash
DATE=$(date +%Y%m%d-%H%M%S)
BACKUP_FILE="/backups/backup-${DATE}.sql.gz"

# Backup database
pg_dumpall | gzip > $BACKUP_FILE

# Upload to S3
aws s3 cp $BACKUP_FILE s3://${S3_BUCKET}/postgres/

# Keep only last 7 days locally
find /backups -name "backup-*.sql.gz" -mtime +7 -delete

echo "Backup completed: $BACKUP_FILE"
```

## Interview Questions

**Q1: What's the difference between restart policies?**
A:
- `no`: Never restart (default)
- `always`: Always restart, even after reboot
- `on-failure`: Only if exit code != 0
- `unless-stopped`: Always except if manually stopped
**Production**: Use `always` or `unless-stopped`

**Q2: How do you implement zero-downtime deployments?**
A:
1. Health checks configured
2. Rolling updates (update one at a time)
3. Graceful shutdown (SIGTERM handling)
4. `stop_grace_period` configured
5. `start-first` update order
6. Load balancer health checks

**Q3: What's the purpose of health checks?**
A: Health checks determine container health:
- **Liveness**: Is container alive? (restart if fails)
- **Readiness**: Is container ready for traffic?
- **Startup**: Wait for application to start
**Essential** for load balancers and orchestrators

**Q4: How do you implement logging in production?**
A:
1. Structured logging (JSON format)
2. Centralized logging (ELK, CloudWatch)
3. Log rotation configured
4. Log levels appropriate (INFO in prod)
5. Sensitive data not logged
6. Request IDs for tracing

**Q5: What resource limits should you set?**
A:
```yaml
deploy:
  resources:
    limits:
      cpus: '2'      # Prevent CPU hogging
      memory: 2G     # Prevent OOM
      pids: 100      # Prevent fork bombs
    reservations:
      cpus: '1'      # Guarantee minimum
      memory: 1G
```

**Q6: How do you handle database migrations in production?**
A:
1. Run migrations before deploying new code
2. Backward compatible migrations
3. Use init containers or separate jobs
4. Test migrations in staging
5. Have rollback plan
6. Backup before migrations

**Q7: What's a canary deployment?**
A: Deploy new version to subset of instances (e.g., 10%):
- Monitor metrics/errors
- Gradually increase traffic
- Rollback if issues
**Benefits**: Reduced risk, early issue detection

**Q8: How do you monitor Docker containers?**
A:
1. **Metrics**: Prometheus + Grafana
2. **Logs**: ELK Stack or CloudWatch
3. **Health**: Health checks + alerts
4. **Resources**: cAdvisor
5. **APM**: New Relic, Datadog
6. **Traces**: Jaeger, Zipkin

## Summary

**Production Essentials:**
- Health checks (liveness + readiness)
- Resource limits (CPU, memory, PIDs)
- Restart policies (always/unless-stopped)
- Logging (structured, centralized, rotated)
- Monitoring (metrics, alerts, dashboards)
- Graceful shutdown (SIGTERM handling)
- Security (non-root, scanning, secrets)

**Deployment Best Practices:**
- Blue-green or canary deployments
- Rolling updates with health checks
- Zero-downtime deployments
- Automated backups to external storage
- Disaster recovery procedures tested
- Database replication for HA

**Production Checklist:**
- [ ] Health checks configured
- [ ] Resource limits set
- [ ] Restart policy configured
- [ ] Logging centralized
- [ ] Monitoring and alerts set up
- [ ] Backup automation implemented
- [ ] Disaster recovery tested
- [ ] Security scanning enabled
- [ ] CI/CD pipeline configured
- [ ] Documentation updated

---

[← Docker Security](./06-docker-security.md) | [Docker with AWS →](./08-docker-with-aws.md)
