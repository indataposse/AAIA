# AAIA - Agente de Automatización de Inteligencia Artificial

> Agente autónomo local para extracción, validación y distribución inteligente de datos operativos vía WhatsApp.

---

## 📋 Índice

- [Descripción](#descripción)
- [Arquitectura](#arquitectura)
- [Stack Tecnológico](#stack-tecnológico)
- [Características](#características)
- [Requisitos](#requisitos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Uso](#uso)
- [Pipeline de Ejecución](#pipeline-de-ejecución)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Seguridad](#seguridad)
- [Roadmap](#roadmap)
- [Licencia](#licencia)

---

## 🎯 Descripción

**AAIA** es un agente de inteligencia artificial desplegado 100% en infraestructura privada (VM Ubuntu + VPN) que opera de forma autónoma para eliminar la intervención manual en la recopilación, validación y distribución de datos operativos críticos — específicamente costos y márgenes — provenientes de portales web corporativos.

El agente recibe instrucciones en **lenguaje natural vía WhatsApp**, ejecuta un pipeline automatizado de descarga web, persistencia validada en base de datos, análisis inteligente mediante modelo local, generación de reportes y notificación multicanal. Todo el procesamiento ocurre dentro de la VM, sin enviar datos a la nube, garantizando **soberanía total de la información** y **costo operativo cero** en licenciamiento.

### ¿Qué problema resuelve?

Hoy, un analista debe:
1. Conectarse manualmente a un portal vía VPN
2. Descargar un Excel de costos
3. Validar que los márgenes no se rompan
4. Generar un reporte
5. Enviarlo por correo o WhatsApp

**AAIA automatiza todo este flujo** desde una única instrucción por chat.

---

## 🏗️ Arquitectura

```
┌─────────────────┐
│   WhatsApp      │
│  (Mensaje de    │
│   usuario)      │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────┐
│  WhatsApp Bridge (Baileys)  │  Node.js - Gateway bidireccional
│  localhost:3000             │
└────────┬────────────────────┘
         │ POST /api/webhook/whatsapp
         ▼
┌─────────────────────────────┐
│      FastAPI (Python)       │  Orquestador REST
│      localhost:5000         │
│  • Validación de remitente  │
│  • Parser de intenciones    │
│  • Enrutamiento de jobs     │
└────────┬────────────────────┘
         │ Encola tarea Celery
         ▼
┌─────────────────────────────┐
│   Celery Worker (Python)    │  Ejecución asíncrona
│  • Web Scraping             │
│  • Persistencia en BD       │
│  • Análisis con LLM         │
│  • Generación de reportes   │
│  • Envío de notificaciones  │
└────────┬────────────────────┘
         │
    ┌────┴────┬────────┬──────────┬─────────┐
    ▼         ▼        ▼          ▼         ▼
┌───────┐ ┌──────┐ ┌──────┐ ┌─────────┐ ┌────────┐
│SQLite │ │Ollama│ │Redis │ │SoftEther│ │SMTP/   │
│(Datos)│ │Gemma4│ │(Cola)│ │  VPN    │ │WhatsApp│
└───────┘ └──────┘ └──────┘ └─────────┘ └────────┘
```

---

## 🛠️ Stack Tecnológico

| Capa | Tecnología | Licencia | Costo |
|------|-----------|----------|-------|
| **SO** | Ubuntu Server 24.04 LTS | GPL | $0 |
| **Lenguaje** | Python 3.12 | PSF | $0 |
| **API Web** | FastAPI + Uvicorn | MIT | $0 |
| **Workers** | Celery + Redis | BSD | $0 |
| **Base de Datos** | SQLite | Public Domain | $0 |
| **ORM** | SQLAlchemy 2.0 + Alembic | MIT | $0 |
| **LLM Runtime** | Ollama | MIT | $0 |
| **Modelo IA** | Gemma-4 | Google (uso local) | $0 |
| **Scraping** | Playwright (Python) | Apache 2.0 | $0 |
| **PDF** | PyMuPDF + WeasyPrint | AGPL / BSD | $0 |
| **Excel** | Pandas + OpenPyXL | BSD | $0 |
| **OCR** | Pytesseract + Pillow | Apache 2.0 | $0 |
| **WhatsApp** | Baileys (Node.js) | MIT | $0 |
| **VPN** | SoftEther VPN Client | GPL | $0 |
| **Testing** | Pytest | MIT | $0 |
| **Linting** | Ruff + Black | MIT | $0 |

---

## ✨ Características

### Skills Principales

| Skill | Descripción |
|-------|-------------|
| **🌐 Web Automation** | Navegación automatizada con Playwright: login, clic, descarga de archivos y extracción tabular desde portales web corporativos. |
| **💾 Data Persistence** | Motor transaccional SQLite con validación de schema, upsert idempotente, auditoría completa (`CostoHistorial`) y `MarginGuard` (bloqueo si margen < 30%). |
| **🧠 LLM Orchestrator** | Integración con Ollama + Gemma-4 para parsing de intenciones, extracción de entidades y generación de análisis narrativo. |
| **📄 Document Intelligence** | Lectura de Excel estructurado, PDF nativo y PDF escaneado (OCR). Comparación de versiones y detección de anomalías. |
| **📊 Report Generator** | Generación parametrizada de reportes en PDF (WeasyPrint) y Excel (OpenPyXL) con formato condicional. |
| **💬 WhatsApp Gateway** | Comunicación bidireccional vía Baileys. Recepción de mensajes, comandos, adjuntos y envío de resultados. |
| **📧 Email Sender** | Envío de correos SMTP con adjuntos mediante MailKit. |
| **🛡️ Sentinel** | Monitoreo proactivo con reglas programadas (Celery Beat). Alertas automáticas ante cambios de costos o anomalías. |
| **🎙️ Voice Commander** | *(Futuro)* Transcripción de notas de voz con Whisper local para comandos hands-free. |

### Capacidades del Agente

- **Reactivo:** Responde a instrucciones por WhatsApp en lenguaje natural.
- **Proactivo:** Ejecuta reglas de monitoreo programadas y alerta sin intervención humana.
- **Transaccional:** Garantiza integridad de datos con rollback automático ante fallos.
- **Auditado:** Cada cambio en costos queda registrado con valor anterior, nuevo, origen y timestamp.
- **Seguro:** Whitelist de números autorizados, rate limiting y sanitización de prompts anti-inyección.

---

## 💻 Requisitos

### Hardware (VM)

| Recurso | Mínimo | Recomendado |
|---------|--------|-------------|
| CPU | 4 vCores | 6-8 vCores |
| RAM | 8 GB | 16 GB |
| Disco | 60 GB SSD | 100 GB SSD |
| Red | 1 NIC + VPN | 1 NIC + VPN dedicada |

### Software

- Ubuntu Server 24.04 LTS (fresh install, sin GUI)
- Python 3.12+
- Node.js 20 LTS (solo para bridge WhatsApp)
- Docker CE (opcional, para contenedores auxiliares)
- Acceso SSH con clave pública

---

## 🚀 Instalación

### 1. Preparar el Sistema

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git vim htop net-tools tmux ufw     software-properties-common python3 python3-pip python3-venv     python3-dev build-essential tesseract-ocr tesseract-ocr-spa     wkhtmltopdf ffmpeg redis-server
```

### 2. Instalar Ollama y descargar Gemma-4

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull gemma:4b
```

### 3. Instalar Playwright y dependencias del sistema

```bash
pip install playwright
playwright install chromium
```

### 4. Instalar Node.js (para WhatsApp Bridge)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

### 5. Clonar y configurar el proyecto

```bash
git clone <repo-url> /opt/agente-ai
cd /opt/agente-ai
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 6. Configurar variables de entorno

```bash
cp .env.example .env
# Editar .env con tus valores:
# DATABASE_URL=sqlite:///opt/agente-ai/data/agente.db
# REDIS_URL=redis://localhost:6379/0
# OLLAMA_BASE_URL=http://localhost:11434
# WHATSAPP_BRIDGE_URL=http://localhost:3000
# API_HOST=127.0.0.1
# API_PORT=5000
```

### 7. Inicializar base de datos

```bash
alembic upgrade head
```

### 8. Iniciar servicios

```bash
# Terminal 1 - API
uvicorn app.main:app --host 127.0.0.1 --port 5000

# Terminal 2 - Worker Celery
celery -A app.workers.celery_app worker --loglevel=info

# Terminal 3 - Beat (Sentinel)
celery -A app.workers.celery_app beat --loglevel=info

# Terminal 4 - WhatsApp Bridge (Node.js)
cd whatsapp-bridge && node index.js
```

### 9. (Opcional) Configurar como servicios systemd

```bash
sudo cp systemd/*.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable agente-ai-api agente-ai-worker agente-ai-beat whatsapp-bridge
sudo systemctl start agente-ai-api agente-ai-worker agente-ai-beat whatsapp-bridge
```

---

## ⚙️ Configuración

### Configuración de VPN (SoftEther)

```bash
sudo apt install softether-vpnclient
# Configurar en /opt/agente-ai/config/vpn/
# Ejecutar: sudo ./scripts/vpn-connect.sh
```

### Autorización de Números WhatsApp

Los números deben estar pre-registrados en la base de datos:

```python
from app.core.database import SessionLocal
from app.persistence.models import NumeroAutorizado

db = SessionLocal()
db.add(NumeroAutorizado(telefono="5215512345678", rol="admin", nombre="Admin", activo=True))
db.commit()
```

### Reglas de Sentinel (Monitoreo Proactivo)

Crear reglas vía API:

```bash
curl -X POST http://localhost:5000/api/sentinel/rules   -H "Content-Type: application/json"   -d '{
    "nombre": "Alerta Costos Proveedor X",
    "cron_expression": "0 8 * * 1",
    "url_portal": "https://intranet.proveedor.com/reportes",
    "condicion_alerta": "variacion > 0.05",
    "destinatarios": ["5215512345678"]
  }'
```

---

## 📱 Uso

### Comandos Vía WhatsApp

| Comando | Descripción |
|---------|-------------|
| `"Descarga el reporte de costos del mes y envíalo por email"` | Pipeline completo: descarga → BD → análisis → reporte → email |
| `"Compara el reporte de esta semana contra el del viernes pasado"` | Análisis de varianza entre versiones |
| `"¿Cuáles artículos tienen margen menor al 30%?"` | Consulta directa a base de datos |
| `/estado` | Ver estado de jobs en ejecución |
| `/cancelar` | Abortar operación en curso |
| `/ayuda` | Listar comandos disponibles |

### Ejemplo de Flujo Completo

```
Usuario: "Descarga los costos del portal, actualiza la base y avísame 
          si algo subió más del 10%"

Agente:  "⏳ Iniciando descarga desde portal..."
         "✅ Datos descargados (142 registros)"
         "💾 Persistiendo en base de datos..."
         "⚠️ Detecté 3 artículos con variación > 10%:"
         "   • Tornillo 1/4: +18% ($4.50 → $5.31)"
         "   • Tuerca M8: +12% ($2.10 → $2.35)"
         "   • Arandela: +15% ($0.80 → $0.92)"
         "📄 Generando reporte detallado..."
         "✅ Reporte enviado. ¿Deseas que notifique al grupo de compras?"
```

---

## 🔄 Pipeline de Ejecución

```
Mensaje WhatsApp
      │
      ▼
┌─────────────┐
│   Parser    │  Gemma-4 extrae intención y parámetros
│   de LLM    │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Descargar  │────▶│  Validar    │────▶│  Persistir  │
│   Excel     │     │   Schema    │     │    en BD    │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                                          [MarginGuard]
                                               │
                        ┌──────────────────────┼──────────────────────┐
                        │                      │                      │
                        ▼                      ▼                      ▼
                   [Margen OK]          [Margen < 30%]         [Error]
                        │                      │                      │
                        ▼                      ▼                      ▼
                   ┌─────────┐          ┌─────────────┐      ┌─────────────┐
                   │ Analizar│          │ Marcar como  │      │ Notificar   │
                   │  Datos  │          │ Pendiente    │      │   Error     │
                   └────┬────┘          │ Aprobación   │      └─────────────┘
                        │               └─────────────┘
                        ▼
                   ┌─────────┐
                   │ Generar │
                   │ Reporte │
                   └────┬────┘
                        │
                        ▼
                   ┌─────────┐
                   │ Enviar  │
                   │WhatsApp/│
                   │  Email  │
                   └─────────┘
```

---

## 📁 Estructura del Proyecto

```
/opt/agente-ai/
├── src/
│   ├── app/
│   │   ├── main.py                 # Entrypoint FastAPI
│   │   ├── config.py               # Pydantic Settings
│   │   ├── api/
│   │   │   ├── routes/
│   │   │   │   ├── health.py
│   │   │   │   ├── webhook.py      # Recepción WhatsApp
│   │   │   │   ├── sentinel.py     # CRUD reglas
│   │   │   │   └── reports.py      # Descarga de reportes
│   │   │   └── dependencies.py
│   │   ├── core/
│   │   │   ├── database.py         # SQLAlchemy engine
│   │   │   └── security.py         # Auth, rate limiting
│   │   ├── services/
│   │   │   ├── orchestrator.py     # Pipeline maestro
│   │   │   ├── auth_service.py     # Validación de números
│   │   │   └── state_manager.py    # Sesiones conversacionales
│   │   ├── skills/
│   │   │   ├── persistence.py      # Skill de BD
│   │   │   ├── web_automation.py   # Playwright
│   │   │   ├── llm_client.py       # Ollama client
│   │   │   ├── document_processor.py
│   │   │   ├── report_generator.py
│   │   │   └── sentinel.py         # Jobs programados
│   │   ├── workers/
│   │   │   ├── celery_app.py       # Config Celery
│   │   │   └── tasks.py            # Tareas definidas
│   │   ├── prompts/
│   │   │   ├── orquestador.txt
│   │   │   ├── extractor_entidades.txt
│   │   │   └── analista_varianza.txt
│   │   └── templates/
│   │       └── reporte_base.html
│   ├── alembic/                    # Migraciones
│   ├── tests/                      # Suite pytest
│   └── requirements.txt
├── whatsapp-bridge/                # Node.js + Baileys
│   ├── index.js
│   └── package.json
├── data/
│   ├── agente.db                   # SQLite principal
│   ├── whatsapp-auth/              # Sesión Baileys
│   └── backups/                    # Backups diarios
├── config/
│   ├── vpn/                        # Config SoftEther
│   └── portals.json                # Selectores scraping
├── reports/                        # Reportes generados
├── logs/                           # Logs rotativos
├── tmp/                            # Archivos temporales
├── scripts/
│   ├── vpn-connect.sh
│   ├── vpn-disconnect.sh
│   ├── vpn-healthcheck.sh
│   └── backup-diario.sh
├── systemd/                        # Unidades de servicio
└── docs/
    ├── scraping.md
    ├── red-vpn.md
    └── operaciones.md
```

---

## 🔒 Seguridad

- **Autenticación:** Solo números pre-registrados en `NumeroAutorizado` pueden interactuar con el agente.
- **Autorización:** Roles diferenciados (`admin`, `operator`, `viewer`).
- **Rate Limiting:** Máximo 10 mensajes/minuto por número.
- **Sanitización:** Detección de prompt injection antes de enviar a Ollama.
- **Transaccionalidad:** Operaciones de BD atómicas con rollback automático.
- **Auditoría:** Tabla `CostoHistorial` registra todo cambio con trazabilidad completa.
- **Red:** UFW activo, Redis y Ollama solo escuchan `127.0.0.1`, VPN aislada.
- **Credenciales:** Almacenadas en archivos con permisos `600`, nunca en el repositorio.

---

## 🗺️ Roadmap

| Fase | Estado | Descripción |
|------|--------|-------------|
| 0 | ⬜ | Preparación de Infraestructura VM |
| 1 | ⬜ | Instalación del Stack Base |
| 2 | ⬜ | Configuración de Red y VPN |
| 3 | ⬜ | Scaffold del Proyecto Python |
| 4 | ⬜ | Motor de Persistencia de Datos |
| 5 | ⬜ | Web Automation y Scraping |
| 6 | ⬜ | Integración con LLM Local |
| 7 | ⬜ | Gateway de Comunicación WhatsApp |
| 8 | ⬜ | Procesamiento de Documentos |
| 9 | ⬜ | Generación de Reportes Parametrizados |
| 10 | ⬜ | Sentinel (Alertas Proactivas) |
| 11 | ⬜ | Orquestación y Pipeline |
| 12 | ⬜ | Hardening, Testing y Documentación |
| 13 | ⬜ | Despliegue y Operación |

### Futuras Extensiones

- 🎙️ **Voice Commander:** Transcripción de notas de voz con Whisper local.
- 🌐 **Dashboard Web:** Interfaz gráfica con Streamlit para administración de reglas y visualización de datos.
- 🔗 **API REST pública:** Endpoints documentados con OpenAPI para integración con otros sistemas.
- 📊 **BI Local:** Integración con Apache Superset o Metabase para dashboards analíticos.

---

## 📄 Licencia

Este proyecto se distribuye bajo la licencia **MIT**.

Todo el stack tecnológico utilizado es software libre y de código abierto. No se requieren licencias de pago para su operación.

---

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor, abre un issue para discutir cambios mayores antes de enviar un pull request.

1. Fork del repositorio
2. Crear rama feature (`git checkout -b feature/nueva-funcionalidad`)
3. Commit de cambios (`git commit -m 'Agrega nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/nueva-funcionalidad`)
5. Abrir Pull Request

---

## 📞 Soporte

Para reportes de errores, consultas técnicas o propuestas de mejora, utiliza la sección de **Issues** del repositorio.

> **Nota operativa:** Este agente está diseñado para ejecutarse en entornos controlados con acceso VPN a portales corporativos. No expongas los endpoints de la API directamente a Internet sin las debidas medidas de seguridad.

---

<p align="center">
  <b>Construido con software libre · Sin costos de licenciamiento · 100% local</b>
</p>
