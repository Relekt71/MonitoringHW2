# Домашнее задание к занятию 14 «Средство визуализации Grafana»

## Задание 1 — Подключение Datasource

### Использованные конфигурации

**Файл `docker-compose.yml`**:

    ```yaml
    version: '3.8'

    services:
    prometheus:
        image: prom/prometheus:latest
        container_name: prometheus
        volumes:
        - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
        - prometheus_data:/prometheus
        ports:
        - "9090:9090"
        command:
        - '--config.file=/etc/prometheus/prometheus.yml'
        - '--storage.tsdb.path=/prometheus'
        networks:
        - monitoring

    node-exporter:
        image: prom/node-exporter:latest
        container_name: node-exporter
        ports:
        - "9100:9100"
        networks:
        - monitoring

    grafana:
        image: grafana/grafana:latest
        container_name: grafana
        ports:
        - "3000:3000"
        environment:
        - GF_SECURITY_ADMIN_PASSWORD=admin
        volumes:
        - grafana_data:/var/lib/grafana
        dns:
        - 8.8.8.8
        - 8.8.4.4
        networks:
        - monitoring

    volumes:
    prometheus_data:
    grafana_data:

    networks:
    monitoring:
        driver: bridge

    Файл prometheus/prometheus.yml:
    yaml

    global:
    scrape_interval: 15s

    scrape_configs:
    - job_name: 'node-exporter'
        static_configs:
        - targets: ['node-exporter:9100']

Запуск
    
    bash
    cd monitoring
    docker compose up -d

Результат

Grafana доступна по адресу: http://localhost:3000
Логин: admin
Пароль: admin
Скриншот Datasource

<img width="1505" height="501" alt="1" src="https://github.com/user-attachments/assets/d36f29eb-51c8-4144-8d20-74eca3522151" />

## Задание 2 — Dashboard и PromQL запросы

### PromQL запросы для панелей

   Панель PromQL запрос
    
    Утилизация CPU (%)	100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
    Load Average 1/5/15	node_load1, node_load5, node_load15
    Свободная RAM (байты)	node_memory_MemFree_bytes
    Свободное место на ФС (%)	(node_filesystem_avail_bytes{mountpoint="/etc/hostname"} / node_filesystem_size_bytes{mountpoint="/etc/hostname"}) * 100
    Скриншот Dashboard

<img width="1492" height="800" alt="2" src="https://github.com/user-attachments/assets/f8159ab1-b2e4-4a18-9e0c-ce54a833c342" />
   
## Задание 3 — Alert Rules и уведомления в Telegram
### Настроенные алерты

    Название алерта	Запрос	Условие	Период ожидания	Severity
    
    High CPU Usage	100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m]))*100)	> 85	5m	warning
    High Load Average 15m	node_load15	> 2.0	5m	warning
    Low Memory Available	node_memory_MemFree_bytes	< 500 MB	2m	critical
    Low Disk Space on /	(node_filesystem_avail_bytes{mountpoint="/etc/hostname"} / node_filesystem_size_bytes{mountpoint="/etc/hostname"}) * 100	< 10%	5m	    critical
    
Скриншот Alert Rules

<img width="885" height="394" alt="3" src="https://github.com/user-attachments/assets/a4e575ba-effb-46e1-b40e-17ba63130020" />

Скриншот Telegram уведомления

<img width="478" height="321" alt="4" src="https://github.com/user-attachments/assets/d8a62160-b5d8-424b-8e3d-f96ce58e32fc" />

## Задание 4 — JSON модель Dashboard

### Файл дашборда экспортирован и сохранён как dashboard-1779197497696.json

dashboard-1779197497696.json прилагается в репозитории

