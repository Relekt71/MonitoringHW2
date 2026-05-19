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
    docker-compose up -d

Результат

Grafana доступна по адресу: http://localhost:3000
Логин: admin
Пароль: admin
Скриншот Datasource

    Вставьте сюда скриншот: Configuration → Data Sources → Prometheus со статусом "Working"

https://screenshots/datasource.png
Задание 2 — Dashboard и PromQL запросы
PromQL запросы для панелей
Панель	PromQL запрос
Утилизация CPU (%)	100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
Load Average 1/5/15	node_load1, node_load5, node_load15
Свободная RAM (байты)	node_memory_MemFree_bytes
Свободное место на ФС (%)	(node_filesystem_avail_bytes{mountpoint="/etc/hostname"} / node_filesystem_size_bytes{mountpoint="/etc/hostname"}) * 100
Скриншот Dashboard

    Вставьте сюда скриншот созданного Dashboard с 4 панелями

https://screenshots/dashboard.png
Задание 3 — Alert Rules и уведомления в Telegram
Настроенные алерты
Название алерта	Запрос	Условие	Период ожидания	Severity
High CPU Usage	100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m]))*100)	> 85	5m	warning
High Load Average 15m	node_load15	> 2.0	5m	warning
Low Memory Available	node_memory_MemFree_bytes	< 500 MB	2m	critical
Low Disk Space on /	(node_filesystem_avail_bytes{mountpoint="/etc/hostname"} / node_filesystem_size_bytes{mountpoint="/etc/hostname"}) * 100	< 10%	5m	critical
Настройка Contact Point для Telegram

    BOT API Token: <токен бота от BotFather>

    Chat ID: <ваш chat_id>

Скриншот Alert Rules

    Вставьте сюда скриншот: Alerting → Alert rules (видны 4 правила)

https://screenshots/alert-rules.png
Скриншот Telegram уведомления

    Вставьте сюда скриншот сообщения от бота в Telegram

https://screenshots/telegram-alert.png
Задание 4 — JSON модель Dashboard

Файл дашборда экспортирован и сохранён как dashboard.json.
Содержимое файла dashboard.json
json

{
  "apiVersion": "dashboard.grafana.app/v2",
  "kind": "Dashboard",
  "metadata": {
    "name": "node-exporter-dashboard",
    "uid": "293e3e3c-3e69-4d6a-b979-cf2b5066e9b8"
  },
  "spec": {
    "title": "Node Exporter Monitoring",
    "timeSettings": {
      "from": "now-6h",
      "to": "now",
      "autoRefresh": "30s"
    },
    "elements": {
      "panel-1": {
        "kind": "Panel",
        "spec": {
          "title": "CPU Utilization (%)",
          "data": {
            "spec": {
              "queries": [
                {
                  "spec": {
                    "query": {
                      "spec": {
                        "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
                        "legendFormat": "CPU Usage"
                      }
                    }
                  }
                }
              ]
            }
          }
        }
      },
      "panel-2": {
        "kind": "Panel",
        "spec": {
          "title": "Load Average (1/5/15 minutes)",
          "data": {
            "spec": {
              "queries": [
                {
                  "spec": {
                    "query": {
                      "spec": {
                        "expr": "node_load1",
                        "legendFormat": "Load 1m"
                      }
                    }
                  }
                },
                {
                  "spec": {
                    "query": {
                      "spec": {
                        "expr": "node_load5",
                        "legendFormat": "Load 5m"
                      }
                    }
                  }
                },
                {
                  "spec": {
                    "query": {
                      "spec": {
                        "expr": "node_load15",
                        "legendFormat": "Load 15m"
                      }
                    }
                  }
                }
              ]
            }
          }
        }
      },
      "panel-3": {
        "kind": "Panel",
        "spec": {
          "title": "Free RAM Memory (bytes)",
          "data": {
            "spec": {
              "queries": [
                {
                  "spec": {
                    "query": {
                      "spec": {
                        "expr": "node_memory_MemFree_bytes",
                        "legendFormat": "Free RAM"
                      }
                    }
                  }
                }
              ]
            }
          }
        }
      },
      "panel-4": {
        "kind": "Panel",
        "spec": {
          "title": "Free Disk Space (bytes)",
          "data": {
            "spec": {
              "queries": [
                {
                  "spec": {
                    "query": {
                      "spec": {
                        "expr": "node_filesystem_avail_bytes",
                        "legendFormat": "Free Space - {{mountpoint}}"
                      }
                    }
                  }
                }
              ]
            }
          }
        }
      }
    },
    "layout": {
      "kind": "GridLayout",
      "spec": {
        "items": [
          { "spec": { "x": 0, "y": 0, "width": 12, "height": 8, "element": { "name": "panel-1" } } },
          { "spec": { "x": 12, "y": 0, "width": 12, "height": 8, "element": { "name": "panel-2" } } },
          { "spec": { "x": 0, "y": 8, "width": 12, "height": 8, "element": { "name": "panel-3" } } },
          { "spec": { "x": 12, "y": 8, "width": 12, "height": 8, "element": { "name": "panel-4" } } }
        ]
      }
    }
  }
}

    Полная версия JSON сохранена в файле dashboard-1779197497696.json (прилагается к решению).

Вывод

Все требования домашнего задания выполнены:
Задание	Статус
Задание 1 — развёртывание Prometheus + Grafana + подключение Datasource	✅
Задание 2 — Dashboard с 4 панелями и PromQL запросы	✅
Задание 3 — Alert Rules + Telegram уведомления	✅
Задание 4 — JSON экспорт Dashboard	✅

Дата выполнения: 19.05.2026
Среда выполнения: Docker Compose, Grafana 11+, Prometheus, Node Exporter
text


---

## Структура файлов для сдачи:

├── README.md # основной файл с решением
├── dashboard-1779197497696.json # экспортированный JSON дашборда
├── docker-compose.yml # конфигурация для запуска
├── prometheus/
│ └── prometheus.yml # конфигурация Prometheus
└── screenshots/ # папка со скриншотами
├── datasource.png
├── dashboard.png
├── alert-rules.png
└── telegram-alert.png