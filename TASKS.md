# Proyecto: Agente de Automatización de Inteligencia Artificial (AAIA)

## Descripción y Necesidad
El AAIA es un agente autónomo desplegado en una VM Ubuntu que opera dentro de una infraestructura privada (VPN). 
Recibe instrucciones en lenguaje natural vía WhatsApp, ejecuta un pipeline automatizado de descarga web, 
persistencia validada en base de datos, análisis inteligente mediante modelo local, 
generación de reportes y notificación multicanal. Todo el ecosistema corre sobre software libre, 
sin costos de licenciamiento ni envío de datos a la nube, garantizando soberanía total de la información.


## FASE 0: Preparación de Infraestructura VM (Entorno base limpio)
Objetivo: Disponer una máquina virtual Ubuntu Server 24.04 LTS aislada, 
segura y accesible remotamente, estableciendo las bases de red, usuarios y seguridad inicial 
sobre las cuales se desplegará todo el ecosistema del agente.

### 0.1 Instalación y Configuración Base del Sistema Operativo
- 0.1.1 Descargar imagen ISO oficial Ubuntu Server 24.04 LTS desde ubuntu.com/download/server
- 0.1.2 Crear VM en hipervisor con 4 vCPU, 16 GB RAM y 100 GB SSD (thin provisioning)
- 0.1.3 Instalar Ubuntu Server con perfil mínimo (sin paquetes snap innecesarios, sin entorno gráfico)
- 0.1.4 Configurar interfaz de red con IP estática, gateway y DNS
- 0.1.5 Establecer hostname `agente-ai` y dominio local
- 0.1.6 Crear usuario operativo `deploy` y agregarlo a grupos `sudo` y `docker`
- 0.1.7 Actualizar índice de paquetes y aplicar upgrades de seguridad
- 0.1.8 Instalar paquetes base: `curl`, `wget`, `git`, `vim`, `htop`, `net-tools`, `tmux`, `ufw`, `software-properties-common`
- 0.1.9 Configurar firewall UFW: denegar todo entrante, permitir SSH (22/tcp) y HTTP del agente (5000/tcp), denegar acceso externo a Redis y Ollama
- 0.1.10 Configurar acceso SSH exclusivo por clave pública, deshabilitar autenticación por contraseña
- 0.1.11 Sincronizar reloj del sistema con NTP: establecer timezone y habilitar `systemd-timesyncd`
- 0.1.12 Crear snapshot de VM denominado `00-fresh-install`


## FASE 1: Instalación del Stack Base (Runtimes y motores del ecosistema)
Objetivo: Instalar todos los intérpretes, motores y dependencias del sistema 
necesarios para que el agente Python y sus componentes satélites operen sin fricciones en el entorno Linux.

### 1.1 Instalación de Python y Entorno de Desarrollo
- 1.1.1 Verificar que Ubuntu traiga Python 3.12 instalado; si no, instalarlo desde repositorios oficiales
- 1.1.2 Instalar `python3-pip`, `python3-venv`, `python3-dev` y `build-essential`
- 1.1.3 Instalar `python-is-python3` para unificar comando `python`
- 1.1.4 Verificar versión: `python --version` debe reportar `3.12.x`

### 1.2 Instalación de Docker y Docker Compose
- 1.2.1 Instalar Docker CE, Docker Compose plugin y containerd desde repositorios oficiales Docker
- 1.2.2 Agregar usuario `deploy` al grupo `docker`
- 1.2.3 Cerrar sesión y volver a iniciar para aplicar membresía de grupo
- 1.2.4 Verificar instalación: `docker run hello-world`

### 1.3 Instalación de Node.js (para Bridge WhatsApp)
- 1.3.1 Instalar Node.js 20 LTS vía NodeSource o descarga binaria oficial
- 1.3.2 Verificar instalación: `node -v` y `npm -v`

### 1.4 Instalación de Redis Server
- 1.4.1 Instalar Redis Server desde repositorios Ubuntu
- 1.4.2 Configurar `redis.conf` para escuchar únicamente `127.0.0.1`
- 1.4.3 Deshabilitar persistencia en disco si se usa exclusivamente como broker de colas en memoria
- 1.4.4 Habilitar e iniciar servicio Redis: `systemctl enable redis-server && systemctl start redis-server`
- 1.4.5 Verificar conexión local: `redis-cli ping`

### 1.5 Instalación de Ollama (Motor LLM Local)
- 1.5.1 Instalar Ollama mediante script oficial o descarga binaria para Linux
- 1.5.2 Verificar servicio Ollama respondiendo en `http://localhost:11434`
- 1.5.3 Descargar modelo `gemma:4b` vía comando `ollama pull gemma:4b`
- 1.5.4 Realizar prueba de inferencia local con `curl` al endpoint `/api/generate`

### 1.6 Instalación de Dependencias del Sistema para Playwright y OCR
- 1.6.1 Instalar dependencias de sistema para Playwright: `libnss3`, `libatk-bridge2.0-0`, `libxcomposite1`, `libxdamage1`, `libxrandr2`, `libgbm1`, `libasound2`, `libpangocairo-1.0-0`
- 1.6.2 Instalar `wkhtmltopdf` para soporte de generación de PDFs
- 1.6.3 Instalar Tesseract OCR y paquete de idioma español: `tesseract-ocr-spa`
- 1.6.4 Instalar `ffmpeg` para procesamiento de medios

### 1.7 Creación de Estructura de Directorios del Proyecto
- 1.7.1 Crear árbol base en `/opt/agente-ai/`: `src`, `data`, `logs`, `backups`, `scripts`, `config`, `reports`, `tmp`, `whatsapp-bridge`
- 1.7.2 Asignar propietario `deploy:deploy` a todo el árbol de trabajo
- 1.7.3 Crear snapshot de VM `01-stack-base`

## FASE 2: Configuración de Red y VPN (Túnel seguro hacia orígenes de datos)
Objetivo: Establecer y asegurar el túnel SoftEther hacia los portales web corporativos 
de donde el agente extraerá la información, garantizando reconexión automática y monitoreo de salud.

### 2.1 Instalación y Configuración de SoftEther VPN Client
- 2.1.1 Instalar cliente SoftEther VPN (`softether-vpnclient`) desde repositorios o compilación
- 2.1.2 Crear directorio de configuración en `/opt/agente-ai/config/vpn/`
- 2.1.3 Crear archivo de configuración de conexión separando credenciales en archivo con permisos `600`
- 2.1.4 Crear script `vpn-connect.sh` que ejecute `vpncmd` para levantar túnel específico
- 2.1.5 Crear script `vpn-disconnect.sh` para cierre controlado del túnel

### 2.2 Validación y Health Check del Túnel
- 2.2.1 Ejecutar conexión manual de prueba y verificar IP de salida con `curl ifconfig.me`
- 2.2.2 Validar que los portales destino responden a través del túnel
- 2.2.3 Implementar script `vpn-healthcheck.sh` que verifique cada 60 segundos la conectividad al host destino
- 2.2.4 Crear unidad systemd `softether-vpn.service` para auto-inicio del túnel
- 2.2.5 Crear unidad systemd timer `vpn-healthcheck.timer` que ejecute el script cada 1 minuto
- 2.2.6 Documentar en `/opt/agente-ai/docs/red-vpn.md` las rutas, IPs y procedimientos de troubleshooting
- 2.2.7 Crear snapshot `02-vpn-configurada`


## FASE 3: Scaffold del Proyecto Python y Arquitectura Base (Cimiento del código)
Objetivo: Establecer la estructura de paquetes Python, el entorno virtual aislado, 
la API con FastAPI y el worker con Celery, garantizando separación de responsabilidades 
y compilación limpia desde el inicio.

### 3.1 Creación del Entorno Virtual y Dependencias Iniciales
- 3.1.1 Crear entorno virtual en `/opt/agente-ai/src/venv`
- 3.1.2 Activar entorno virtual y actualizar `pip`
- 3.1.3 Crear archivo `requirements.txt` con dependencias iniciales: `fastapi`, `uvicorn[standard]`, `celery[redis]`, `redis`, `sqlalchemy`, `alembic`, `pydantic`, `pydantic-settings`, `httpx`, `python-dotenv`, `jinja2`, `pandas`, `openpyxl`, `pymupdf`, `weasyprint`, `pytesseract`, `pillow`, `playwright`, `pytest`, `pytest-asyncio`, `black`, `ruff`
- 3.1.4 Instalar dependencias iniciales: `pip install -r requirements.txt`
- 3.1.5 Instalar browsers de Playwright: `playwright install chromium`

### 3.2 Estructura de Paquetes Python
- 3.2.1 Crear directorio raíz de paquete `app/` dentro de `src/`
- 3.2.2 Crear subpaquetes: `app/api/`, `app/core/`, `app/skills/`, `app/workers/`, `app/services/`
- 3.2.3 Crear archivos `__init__.py` en cada subpaquete
- 3.2.4 Crear directorios auxiliares: `alembic/`, `tests/`, `scripts/`

### 3.3 Configuración de la Aplicación FastAPI
- 3.3.1 Crear `app/config.py` usando `pydantic-settings` para variables de entorno: `DATABASE_URL`, `REDIS_URL`, `OLLAMA_BASE_URL`, `WHATSAPP_BRIDGE_URL`, `API_HOST`, `API_PORT`
- 3.3.2 Crear `app/main.py` instanciando `FastAPI` con configuración de CORS, logging estructurado y lifespan events
- 3.3.3 Configurar Uvicorn para escuchar en `http://127.0.0.1:5000`
- 3.3.4 Crear endpoint `GET /health` que reporte estado de SQLite, Redis, Ollama y VPN

### 3.4 Configuración de Celery y Workers
- 3.4.1 Crear `app/workers/celery_app.py` configurando broker y backend en Redis (`redis://localhost:6379/0`)
- 3.4.2 Crear `app/workers/tasks.py` con tarea dummy de prueba
- 3.4.3 Verificar que Celery worker inicie sin errores: `celery -A app.workers.celery_app worker --loglevel=info`

### 3.5 Verificación del Scaffold
- 3.5.1 Ejecutar API localmente con Uvicorn y verificar respuesta en `http://localhost:5000/health`
- 3.5.2 Ejecutar worker de Celery y verificar procesamiento de tarea dummy
- 3.5.3 Crear snapshot `03-scaffold-python`


## FASE 4: Motor de Persistencia de Datos (SQLite + SQLAlchemy)
Objetivo: Garantizar integridad transaccional, auditoría completa y validación de negocio en la capa de datos, implementando un motor de persistencia que el agente invoque de forma segura sin exponer SQL al modelo de lenguaje.

### 4.1 Diseño del Modelo de Datos
- 4.1.1 Definir entidad `Costo` con campos: `id`, `codigo_articulo`, `descripcion`, `costo_unitario`, `moneda`, `fecha_vigencia`, `proveedor`, `margen_calculado`, `estado` (activo/pendiente_aprobacion)
- 4.1.2 Definir entidad `CostoHistorial` con campos: `id`, `costo_id`, `campo_modificado`, `valor_anterior`, `valor_nuevo`, `origen`, `usuario_origen`, `fecha_cambio`
- 4.1.3 Definir entidad `SesionConversacion` con campos: `id`, `numero_telefono`, `estado`, `contexto_json`, `ultima_actividad`
- 4.1.4 Definir entidad `JobEnCola` con campos: `id`, `tipo`, `payload`, `estado`, `intentos`, `creado`, `procesado`
- 4.1.5 Definir entidad `NumeroAutorizado` con campos: `id`, `telefono`, `rol`, `nombre`, `activo`

### 4.2 Configuración de SQLAlchemy y Migraciones
- 4.2.1 Crear `app/core/database.py` con engine SQLite, `SessionLocal` y `Base` declarativa
- 4.2.2 Inicializar Alembic: `alembic init alembic`
- 4.2.3 Configurar `alembic.ini` apuntando a la base de datos del agente
- 4.2.4 Generar migración inicial: `alembic revision --autogenerate -m "Initial schema"`
- 4.2.5 Aplicar migración: `alembic upgrade head`

### 4.3 Implementación del Repositorio y Unidad de Trabajo
- 4.3.1 Implementar clase `Repository` genérica con operaciones `add`, `get`, `update`, `delete`, `commit`
- 4.3.2 Implementar `UnitOfWork` que gestione sesiones SQLAlchemy con context manager (`__enter__`, `__exit__`)
- 4.3.3 Implementar manejo de excepciones con rollback automático ante errores

### 4.4 Implementación de la Skill de Persistencia
- 4.4.1 Crear `app/skills/persistence.py` con clase `DataPersistenceSkill`
- 4.4.2 Implementar método `validar_schema_excel`: leer primeras filas y verificar columnas obligatorias (código, descripción, costo_unitario, moneda, fecha_vigencia)
- 4.4.3 Implementar método `sincronizar_costos` con apertura de transacción explícita
- 4.4.4 Implementar lógica de upsert nativa SQLite usando `INSERT ... ON CONFLICT ... DO UPDATE`
- 4.4.5 Implementar `audit_logger`: dentro de la misma transacción, insertar registros en `CostoHistorial`
- 4.4.6 Implementar `pre_flight_backup`: antes del upsert, copiar registros del periodo afectado a tabla `_backup_temp`
- 4.4.7 Implementar `margin_guard`: calcular margen proyectado; si es menor al 30%, cambiar `estado` a `pendiente_aprobacion`
- 4.4.8 Implementar `detectar_anomalias`: comparar costos del periodo actual vs anterior, reportar variaciones mayores al 5%

### 4.5 Optimización y Testing de la Capa de Datos
- 4.5.1 Crear índices en SQLite: `IX_costos_codigo_proveedor_periodo`, `IX_costo_historial_fecha_cambio`
- 4.5.2 Insertar seed data de prueba (10 registros, 2 periodos)
- 4.5.3 Escribir tests con `pytest`: transacción commit, transacción rollback, margin guard, validación de schema
- 4.5.4 Crear snapshot `04-persistencia`

## FASE 5: Web Automation y Scraping (Extracción automatizada de portales)
Objetivo: Automatizar la navegación, autenticación y descarga de archivos desde portales web corporativos usando Playwright para Python, con reconexión, validación y captura de errores.

### 5.1 Configuración de Playwright
- 5.1.1 Verificar que los browsers de Playwright estén instalados para Linux
- 5.1.2 Crear `app/skills/web_automation.py` con clase `WebAutomationSkill`
- 5.1.3 Implementar método `iniciar_navegador`: lanzar Playwright en modo headless con viewport fijo

### 5.2 Implementación de Acciones de Navegación
- 5.2.1 Implementar `navegar_a(url)`: ir a URL con timeout de 30 segundos
- 5.2.2 Implementar `autenticar(portal, usuario, password)`: llenar inputs de login y submit, esperar navegación post-login
- 5.2.3 Implementar `clic_opcion(selector_css)`: clic en elemento visible
- 5.2.4 Implementar `descargar_archivo(url_descarga, ruta_destino, formato)`: interceptar evento `download` y guardar
- 5.2.5 Implementar `extraer_datos_tabla(selector_tabla)`: leer HTML table a estructura lista de diccionarios

### 5.3 Robustez y Recuperación
- 5.3.1 Implementar persistencia de estado de sesión (`storage_state`) para evitar re-login en cada ejecución
- 5.3.2 Implementar captura de pantalla automática en caso de excepción (`screenshot_on_failure`)
- 5.3.3 Implementar reintentos con backoff exponencial (3 intentos: inmediato, 5s, 15s)
- 5.3.4 Crear archivo JSON de configuración por portal: URL, selectores de login, botón de descarga, timeout
- 5.3.5 Implementar validación post-descarga: archivo existe, tamaño mayor a 0 bytes, extensión correcta
- 5.3.6 Implementar `cerrar_navegador` para cierre ordenado de browser, context y page

### 5.4 Testing y Documentación del Scraper
- 5.4.1 Escribir test de integración usando un HTML estático local como mock de portal
- 5.4.2 Documentar selectores y flujos de cada portal en `/opt/agente-ai/docs/scraping.md`
- 5.4.3 Crear snapshot `05-web-automation`


## FASE 6: Integración con LLM Local (Ollama + Gemma-4)
Objetivo: Conectar el orquestador con el modelo de lenguaje local para parsing de intenciones, extracción de entidades y generación de análisis narrativo, manteniendo toda la inferencia dentro de la VM.

### 6.1 Cliente HTTP para Ollama
- 6.1.1 Crear `app/skills/llm_client.py` con clase `LlmClient`
- 6.1.2 Implementar método `generar_estructurado`: POST a `http://localhost:11434/api/generate` con system prompt y format JSON
- 6.1.3 Implementar método `chat`: POST a `/api/chat` manteniendo historial de mensajes
- 6.1.4 Configurar `httpx` con timeout de 60 segundos y pool de conexiones

### 6.2 Prompts y Resolución de Intenciones
- 6.2.1 Crear directorio `app/prompts/` con archivos de texto: `orquestador.txt`, `extractor_entidades.txt`, `analista_varianza.txt`
- 6.2.2 Implementar `IntentResolver`: envía texto del usuario + contexto de sesión a Ollama, recibe JSON con `accion` y `parametros`
- 6.2.3 Implementar `ResponseParser`: extraer bloque JSON de la respuesta del modelo usando índices o regex
- 6.2.4 Implementar validación de JSON contra schema mínimo antes de deserializar

### 6.3 Resiliencia y Monitoreo del LLM
- 6.3.1 Implementar Circuit Breaker: si Ollama no responde 3 veces consecutivas, marcar estado degradado y notificar
- 6.3.2 Crear endpoint de diagnóstico `GET /api/diagnostico/llm` para verificar salud del modelo
- 6.3.3 Configurar parámetros de generación: `temperature: 0.1` para extracción, `0.7` para conversación
- 6.3.4 Implementar logging de prompts (sin datos sensibles de credenciales)

### 6.4 Testing del Motor de Lenguaje
- 6.4.1 Testear `IntentResolver` con 10 ejemplos de mensajes de WhatsApp variados
- 6.4.2 Verificar que el parser maneja correctamente respuestas malformadas
- 6.4.3 Crear snapshot `06-llm-integrado`


## FASE 7: Gateway de Comunicación WhatsApp (Bidireccional)
Objetivo: Habilitar recepción y envío de mensajes, archivos y comandos vía WhatsApp, utilizando Baileys como bridge Node.js y exponiendo webhooks hacia la API FastAPI del agente.

### 7.1 Implementación del Bridge Node.js (Baileys)
- 7.1.1 Crear directorio `/opt/agente-ai/whatsapp-bridge/` e inicializar proyecto Node.js: `npm init -y`
- 7.1.2 Instalar dependencias: `@whiskeysockets/baileys`, `qrcode-terminal`, `axios`, `express`, `pino`
- 7.1.3 Implementar `index.js`: conexión a WhatsApp Web usando Baileys con `useMultiFileAuthState`
- 7.1.4 Implementar persistencia de sesión en `/opt/agente-ai/data/whatsapp-auth/`
- 7.1.5 Implementar endpoint Express `POST /send`: recibe `numero`, `mensaje`, opcional `ruta_archivo`
- 7.1.6 Implementar handler `messages.upsert`: al recibir mensaje, descargar media si existe y reenviar vía POST a `http://localhost:5000/api/webhook/whatsapp`
- 7.1.7 Implementar handler `connection.update`: loggear estado (connecting, open, close)
- 7.1.8 Crear servicio systemd `whatsapp-bridge.service` con auto-restart

### 7.2 Recepción y Procesamiento en FastAPI
- 7.2.1 Implementar endpoint `POST /api/webhook/whatsapp` en FastAPI
- 7.2.2 Implementar `WhatsAppReceiver`: deserializar payload, validar firma/número, encolar `JobEnCola` en Celery
- 7.2.3 Implementar `WhatsAppSender`: al finalizar pipeline, llamar bridge `POST /send` con resultado
- 7.2.4 Implementar `AuthService`: verificar `NumeroAutorizado` antes de procesar cualquier mensaje

### 7.3 Seguridad y Control de Acceso
- 7.3.1 Implementar rate limiting: máximo 10 mensajes/minuto por número usando Redis o diccionario en memoria con expiración
- 7.3.2 Implementar comandos de control: `/ayuda`, `/estado`, `/cancelar`, `/confirmar`
- 7.3.3 Iniciar bridge, escanear QR desde terminal y verificar sesión abierta

### 7.4 Testing del Flujo Bidireccional
- 7.4.1 Testear flujo bidireccional completo: enviar mensaje de prueba → recibir respuesta del agente
- 7.4.2 Testear envío de archivo PDF desde agente hacia número de prueba
- 7.4.3 Crear snapshot `07-whatsapp-gateway`


## FASE 8: Procesamiento de Documentos (PDF / Excel / OCR)
Objetivo: Extraer, transformar y comparar datos de archivos descargados o adjuntados por los usuarios, 
soportando Excel estructurado, PDF nativo y PDF escaneado mediante OCR.

### 8.1 Procesamiento de Archivos Excel
- 8.1.1 Crear `app/skills/document_processor.py` con clase `DocumentProcessorSkill`
- 8.1.2 Implementar `leer_excel`: recibe `BytesIO` o ruta, devuelve lista de diccionarios mapeando columnas por nombre usando `pandas` + `openpyxl`
- 8.1.3 Implementar validación de hojas: verificar que exista hoja esperada y tenga datos
- 8.1.4 Implementar mapeo dinámico de columnas a modelo `Costo`

### 8.2 Procesamiento de Archivos PDF
- 8.2.1 Implementar `leer_pdf_texto`: extracción de texto plano con `PyMuPDF` (fitz)
- 8.2.2 Implementar `leer_pdf_escaneado`: conversión de página a imagen temporal + llamada a `pytesseract` para OCR
- 8.2.3 Implementar `comparar_versiones`: recibe dos rutas de Excel, genera diff con valores anterior/nuevo/porcentaje usando `pandas`

### 8.3 Gestión y Limpieza de Archivos
- 8.3.1 Implementar cálculo de hash SHA256 para cada archivo procesado (evitar reprocesamiento)
- 8.3.2 Implementar limpieza de archivos temporales en `/opt/agente-ai/tmp/` mayores a 24 horas
- 8.3.3 Escribir tests con archivos mock (Excel con 100 filas, PDF con tabla simple)
- 8.3.4 Crear snapshot `08-documentos`


## FASE 9: Generación de Reportes Parametrizados
Objetivo: Crear documentos profesionales (PDF y Excel) a partir de datos analizados, utilizando plantillas parametrizables y motor de renderizado nativo.

### 9.1 Motor de Plantillas y Renderizado
- 9.1.1 Crear directorio `app/templates/` para plantillas Jinja2 HTML
- 9.1.2 Crear plantilla base `reporte_base.html` con encabezado, tabla de datos, sección de totales y pie de página
- 9.1.3 Definir modelo `ReporteParametros`: título, subtítulo, periodo, filtros aplicados, lista de registros

### 9.2 Generación de Reportes en PDF y Excel
- 9.2.1 Implementar `generar_pdf`: renderizar HTML con Jinja2 y convertir a PDF con `WeasyPrint`
- 9.2.2 Implementar `generar_excel`: usar `openpyxl` para crear archivo `.xlsx` con formato condicional (celdas rojas si varianza mayor al 10%)
- 9.2.3 Guardar reportes generados en `/opt/agente-ai/reports/` con nombre único incluyendo timestamp

### 9.3 Exposición y Testing
- 9.3.1 Crear endpoint `GET /api/reports/{id}/download` para descarga directa
- 9.3.2 Testear generación con datos de prueba y validar apertura en Excel/Adobe Reader
- 9.3.3 Crear snapshot `09-reportes`


## FASE 10: Sentinel (Alertas y Monitoreo Proactivo)
Objetivo: Transformar al agente de reactivo a proactivo mediante reglas programadas que monitorean portales web y notifican anomalías sin intervención humana.

### 10.1 Configuración del Scheduler con Celery Beat
- 10.1.1 Crear `app/skills/sentinel.py` con clase `SentinelService`
- 10.1.2 Definir entidad `ReglaMonitoreo`: nombre, cron expression, url portal, selector datos, condición alerta, destinatarios WhatsApp
- 10.1.3 Configurar Celery Beat en `celery_app.py` con scheduler `DatabaseScheduler` o `PersistentScheduler`

### 10.2 Lógica de Monitoreo y Disparo
- 10.2.1 Implementar tarea Celery `monitoreo_job`: ejecutar scraping, comparar contra último valor conocido en BD
- 10.2.2 Implementar lógica de disparo: solo notificar si la condición se cumple (ej. costo cambió más del 5%)
- 10.2.3 Implementar endpoint CRUD `/api/sentinel/rules` para administrar reglas vía API
- 10.2.4 Testear regla con expresión cron frecuente (cada 30 segundos) contra mock local
- 10.2.5 Crear snapshot `10-sentinel`


## FASE 11: Orquestación y Pipeline de Ejecución
Objetivo: Unificar todas las skills bajo un flujo coherente gestionado por el agente, permitiendo ejecución secuencial, manejo de errores por etapa y cancelación de jobs en curso.

### 11.1 Diseño del Orquestador
- 11.1.1 Crear `app/services/orchestrator.py` con clase `AgentOrchestrator`
- 11.1.2 Implementar máquina de estados del pipeline: `idle` → `descargando` → `persistiendo` → `analizando` → `reportando` → `enviando` → `completado`

### 11.2 Gestión de Errores y Cancelación
- 11.2.1 Implementar manejo de errores por etapa: si falla descarga, notificar error sin ejecutar persistencia
- 11.2.2 Implementar timeout global por pipeline (10 minutos) usando `Celery` `soft_time_limit` y `time_limit`
- 11.2.3 Implementar comando `/cancelar`: revocar tarea Celery activa mediante `AsyncResult.revoke(terminate=True)`
- 11.2.4 Implementar persistencia de estado del pipeline en `SesionConversacion` para recuperación ante reinicio

### 11.3 Integración Final
- 11.3.1 Integrar todas las skills en el contenedor de dependencias de FastAPI
- 11.3.2 Testear flujo completo end-to-end con datos reales o de prueba
- 11.3.3 Crear snapshot `11-orquestacion`


## FASE 12: Hardening, Testing y Documentación
Objetivo: Garantizar seguridad, estabilidad y operabilidad del sistema mediante validación exhaustiva, saneamiento de entradas y documentación técnica completa.

### 12.1 Seguridad y Validación
- 12.1.1 Implementar validación de entrada en todos los endpoints usando Pydantic schemas estrictos
- 12.1.2 Implementar sanitización de prompts: detectar intentos de inyección que modifiquen system prompt
- 12.1.3 Ejecutar `black` para formateo de código y `ruff` para análisis estático
- 12.1.4 Resolver todos los warnings críticos reportados por `ruff`

### 12.2 Testing Exhaustivo
- 12.2.1 Escribir tests unitarios con `pytest` para cada skill (mínimo 3 casos por skill)
- 12.2.2 Escribir tests de integración para endpoints críticos de la API
- 12.2.3 Escribir test de estrés: encolar 20 jobs simultáneos y verificar que Celery no pierda mensajes
- 12.2.4 Verificar cobertura de código con `pytest-cov`; objetivo mínimo 70%

### 12.3 Operaciones y Mantenimiento
- 12.3.1 Revisar permisos de archivos: todo en `/opt/agente-ai` debe pertenecer a `deploy:deploy`, nada a root
- 12.3.2 Configurar `logrotate` para archivos en `/opt/agente-ai/logs/` con retención de 30 días
- 12.3.3 Crear script `backup-diario.sh` que copie `agente_data.db` y `config/` a `/opt/agente-ai/backups/` con fecha
- 12.3.4 Documentar `README.md` con arquitectura, instalación y troubleshooting
- 12.3.5 Documentar `OPERACIONES.md` con comandos systemd, logs y procedimientos de recuperación
- 12.3.6 Crear snapshot `12-hardening`


## FASE 13: Despliegue y Operación (Puesta en producción)
Objetivo: Publicar el agente como servicios systemd persistentes, establecer monitoreo continuo y validar operatividad total desde WhatsApp hasta la base de datos.

### 13.1 Publicación de la Aplicación
- 13.1.1 Congelar dependencias: `pip freeze > requirements.txt`
- 13.1.2 Verificar que entorno virtual contenga todas las dependencias necesarias
- 13.1.3 Sincronizar código fuente final al directorio de despliegue

### 13.2 Configuración de Servicios Systemd
- 13.2.1 Crear unidad systemd `agente-ai-api.service` ejecutando Uvicorn sobre `app.main:app` en `127.0.0.1:5000`
- 13.2.2 Crear unidad systemd `agente-ai-worker.service` ejecutando Celery worker
- 13.2.3 Crear unidad systemd `agente-ai-beat.service` ejecutando Celery Beat para Sentinel
- 13.2.4 Configurar `Restart=always` y `RestartSec=10` en todas las unidades
- 13.2.5 Establecer `WorkingDirectory=/opt/agente-ai/src` y usuario `deploy` en todas las unidades

### 13.3 Arranque y Verificación
- 13.3.1 Habilitar todos los servicios para inicio automático: `systemctl enable agente-ai-api agente-ai-worker agente-ai-beat`
- 13.3.2 Iniciar servicios y verificar estado: `systemctl status agente-ai-api`
- 13.3.3 Verificar logs en tiempo real: `journalctl -u agente-ai-api -f`
- 13.3.4 Verificar que Ollama, Redis, VPN y WhatsApp Bridge están activos antes de declarar operativo

### 13.4 Prueba de Humo Final
- 13.4.1 Ejecutar prueba de humo completa: mensaje de WhatsApp → descarga → BD → análisis → reporte PDF → respuesta WhatsApp
- 13.4.2 Validar que el flujo Sentinel ejecuta correctamente según cron programado
- 13.4.3 Establecer política de backups automáticos vía cron del sistema
- 13.4.4 Crear snapshot final `13-produccion`


## Sección de Recursos y Descargas

### Descripción del Proyecto y su Necesidad
El AAIA resuelve el problema de la **fragmentación operativa**: hoy, un analista debe manualmente conectarse a un portal vía VPN, descargar un Excel, validar que los costos no rompan el margen, convertir eso en un reporte y enviarlo por correo o WhatsApp. Este agente automatiza todo ese pipeline desde una VM local, sin costos de nube ni suscripciones, manteniendo los datos financieros dentro de la infraestructura propia.

### Tabla de Recursos: Recomendado vs. Alternativas

| Recurso | Propósito | Recomendado (✓) | Alternativas | Licencia / Costo |
|---------|-----------|-----------------|--------------|------------------|
| **SO** 				| Sistema operativo base 				| **Ubuntu Server 24.04 LTS** 		| Debian 12, AlmaLinux 9 	| GPL / **$0** |
| **Lenguaje** 			| Desarrollo del agente 				| **Python 3.12** 					| Python 3.11, PyPy 		| PSF / **$0** |
| **API Web** 			| Servidor HTTP del agente 				| **FastAPI + Uvicorn** 			| Flask, Django, Quart 		| MIT / **$0** |
| **Worker / Colas** 	| Procesamiento asíncrono 				| **Celery + Redis** 				| RQ, Huey, Dramatiq 		| BSD / **$0** |
| **Base de Datos** 	| Persistencia estructurada 			| **SQLite** 						| PostgreSQL, MariaDB 		| Public Domain / **$0** |
| **ORM / Migraciones** | Acceso a datos 						| **SQLAlchemy 2.0 + Alembic** 		| Peewee, Tortoise ORM		| MIT / **$0** |
| **Validación** 		| Schemas y configuración 				| **Pydantic + pydantic-settings** 	| Marshmallow, attrs 		| MIT / **$0** |
| **Web Scraping** 		| Automatización de navegador 			| **Playwright (Python)** 			| Selenium, Scrapy 			| Apache 2.0 / **$0** |
| **Cliente HTTP** 		| Comunicación con APIs 				| **HTTPX** 						| Requests, aiohttp 		| BSD / **$0** |
| **Lectura Excel** 	| Procesamiento .xlsx 					| **Pandas + OpenPyXL** 			| xlrd, xlwings 			| BSD / **$0** |
| **Lectura PDF** 		| Extracción de texto PDF 				| **PyMuPDF** 						| pdfplumber, pdfminer 		| AGPL (uso interno libre) / **$0** |
| **Generación PDF** 	| Creación de reportes PDF 				| **WeasyPrint** 					| ReportLab, pdfkit 		| BSD / **$0** |
| **OCR** 				| Reconocimiento de texto en imágenes 	| **Pytesseract + Pillow** 			| EasyOCR, PaddleOCR 		| Apache 2.0 / **$0** |
| **Templates** 		| Plantillas HTML para reportes 		| **Jinja2** 						| Mako, Chameleon 			| BSD / **$0** |
| **WhatsApp Gateway** 	| Conexión bidireccional WA 			| **Baileys (Node.js)** 			| WhatsApp Business API (Meta, pago por conversación), Twilio (pago) | MIT / **$0** |
| **LLM Runtime** 		| Inferencia local del modelo 			| **Ollama** 						| llama.cpp, LocalAI 		| MIT / **$0** |
| **Modelo LLM** 		| Cerebro del agente 					| **Gemma-4 (vía Ollama)** 			| Llama 3, Mistral, Qwen 	| Peso propio (Google) / **$0** |
| **VPN Client** 		| Túnel hacia portales corporativos 	| **SoftEther VPN Client** 			| OpenVPN, WireGuard 		| GPL / **$0** |
| **Contenedores** 		| Aislamiento de procesos 				| **Docker CE** 					| Podman, LXC 				| Apache 2.0 / **$0** |
| **Testing** 			| Framework de pruebas 					| **Pytest + pytest-asyncio** 		| unittest, nose2 			| MIT / **$0** |
| **Linting / Format** 	| Calidad de código 					| **Ruff + Black** 					| flake8, pylint, autopep8 	| MIT / **$0** |
| **Node.js** 			| Runtime para bridge WhatsApp 			| **Node.js 20 LTS** 				| Deno, Bun 				| MIT / **$0** |

### Notas sobre Costo Cero y Software Libre
- **Todo el stack es reemplazable.** Si en el futuro necesitas escalar la BD, PostgreSQL es la migración natural y también es $0.
- **Ningún componente requiere licencia de uso.** El único costo posible es la infraestructura (VM, electricidad, ancho de banda), nunca por software.
- **Baileys vs. Business API:** Baileys es la única opción 100% gratuita. Si en el futuro el negocio requiere verificación oficial de Meta, el cambio a Business API solo afecta el bridge Node.js; tu código Python permanece intacto.
- **Gemma-4:** Aunque el modelo es de Google, su uso local vía Ollama no genera costos ni llamadas a API externas.
