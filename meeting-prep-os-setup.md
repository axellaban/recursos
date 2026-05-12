# Meeting Prep OS — Guía de Setup Completo

Sistema autónomo que revisa tu Google Calendar cada mañana, detecta reuniones con `#prep` y genera dashboards completos de preparación: briefing ejecutivo, inteligencia competitiva, deep research, quiz interactivo y flashcards 3D. Todo en un HTML deployado en GitHub Pages.

---

## Stack necesario

| Componente | Para qué |
|---|---|
| **Claude Desktop + Cowork** | El agente que ejecuta todo |
| **Google Calendar MCP** | Leer eventos del calendario |
| **NotebookLM MCP** (custom) | Deep research + generación de contenido |
| **WebSearch** (built-in Cowork) | Research web de personas y empresas |
| **Bash / File tools** (built-in Cowork) | Crear archivos HTML y JSON |
| **GitHub repo** | Almacenar y versionar los dashboards |

---

## Paso 1 — Crear el repo en GitHub

```bash
# Crear repo público o privado llamado "meeting-prep"
# Estructura mínima necesaria:
meeting-prep/
├── index.html          # Dashboard principal (opcional)
├── meetings.json       # Índice de todas las reuniones
└── meetings/           # Una carpeta por reunión
```

Inicializá `meetings.json` con esto:
```json
{
  "meetings": []
}
```

---

## Paso 2 — Configurar Google Calendar MCP

1. En Claude Desktop → Settings → MCP Servers
2. Agregar el MCP de Google Calendar (oficial de Google o via `@anthropic-community/mcp-server-google-calendar`)
3. Autenticar con la cuenta de Google que usás para tu calendario
4. Verificar que puede leer eventos: `list_events` debe devolver tus próximos eventos

---

## Paso 3 — Configurar NotebookLM MCP (custom)

Este MCP no es oficial. Automatiza NotebookLM via su interfaz web.

**Opción recomendada:** usar el MCP de la comunidad disponible en npm/GitHub que wrappea la API no pública de NotebookLM.

```bash
# Instalar dependencias
npm install -g notebooklm-mcp  # o el package que uses

# Configurar en Claude Desktop settings.json:
{
  "mcpServers": {
    "notebooklm": {
      "command": "npx",
      "args": ["notebooklm-mcp"],
      "env": {
        "GOOGLE_EMAIL": "tu@gmail.com",
        "GOOGLE_PASSWORD": "tu-password-de-app"
      }
    }
  }
}
```

> **Nota:** Necesitás una Google App Password (no tu password normal). Generala en: myaccount.google.com → Seguridad → Contraseñas de aplicación.

Herramientas que debe exponer este MCP:
- `notebook_create`, `notebook_add_text`, `notebook_add_url`
- `research_start`, `research_status`, `research_import`
- `notebook_query`
- `quiz_create`, `flashcards_create`, `studio_status`

---

## Paso 4 — Configurar el GitHub Token

Para que el agente pueda hacer `git push` sin intervención manual:

```bash
# Generar un Personal Access Token en GitHub:
# github.com → Settings → Developer Settings → Personal Access Tokens → Fine-grained
# Permisos: Contents (read/write) sobre el repo meeting-prep

# El agente lo guarda así (lo hace automáticamente cuando se lo das):
echo "https://TU_USUARIO:ghp_TU_TOKEN@github.com" > ~/.git-credentials
git config --global credential.helper store
```

Dáselo al agente una sola vez y lo guarda para siempre.

---

## Paso 5 — Conectar el workspace en Cowork

1. Abrir Claude Desktop en modo Cowork
2. Seleccionar la carpeta donde está tu repo local: `meeting-prep/`
3. El agente necesita acceso de lectura/escritura a esa carpeta

---

## Paso 6 — Crear la tarea programada

En Cowork, crear un **Scheduled Task** llamado `meeting-prep-daily` con el siguiente prompt:

---

### PROMPT DE LA TAREA (copiar completo)

```
## Meeting Prep OS — Daily Pipeline

Sos un agente autónomo de preparación de reuniones. Tu misión: revisar el Google Calendar y generar dashboards completos de preparación para cada reunión marcada con #prep.

---

### PASO 1 — Revisar Google Calendar

Buscá eventos de HOY y MAÑANA:
- timeZone: America/Argentina/Buenos_Aires (ajustar a tu timezone)
- Filtro: "#prep" en el título

Si no encontrás eventos con #prep → terminá con: "No hay reuniones con #prep para hoy/mañana."

---

### PASO 2 — Extraer datos de cada evento

Por cada evento con #prep, extraé:
- person_name: nombre de la persona (del título del evento)
- company_name: empresa (del título o descripción)
- meeting_date: YYYY-MM-DD
- meeting_time: HH:MM
- meeting_type: tipo de reunión (o "Reunión de negocio" por defecto)

Generá un meeting_id: YYYY-MM-DD-nombre-apellido (minúsculas, guiones)

Si la carpeta meetings/[meeting_id]/ ya existe → saltá esa reunión (ya fue procesada).

---

### PASO 3 — Research de persona y empresa

Buscá en la web (WebSearch):
1. Quién es la persona: cargo, empresa, trayectoria, LinkedIn
2. La empresa: qué hace, tamaño, modelo de negocio, noticias recientes
3. Contexto del mercado/industria
4. Noticias recientes (últimos 6 meses)

Sintetizá todo en un perfil detallado.

---

### PASO 4 — NotebookLM Pipeline

4.1 Crear notebook:
  notebook_create(title="Meeting Prep - [person_name]")

4.2 Agregar perfil sintetizado:
  notebook_add_text(notebook_id, title="[person_name] — Profile", text=[perfil])

4.3 Agregar URLs relevantes (3-5 URLs del research):
  notebook_add_url(notebook_id, url=[cada URL])

4.4 Iniciar deep research:
  research_start(notebook_id, query="[person_name] [company_name] industria competitive landscape 2025 2026", source="web", mode="fast")

4.5 Esperar y verificar con research_status, luego importar:
  research_import(notebook_id, task_id=[task_id])

---

### PASO 5 — Generar los 3 documentos via notebook_query

**Executive Briefing:**
"Creá un documento ejecutivo de preparación pre-reunión con estas secciones exactas en español: 1) Perfil de la Persona (background, educación, roles actuales, logros clave, contexto de la reunión), 2) Perfil de la Empresa (qué hace, posición en el mercado, modelo de negocio, clientes, ventajas competitivas), 3) Oportunidad de Mercado (cifras específicas en dólares, tasas de crecimiento, tendencias relevantes), 4) Puntos Clave de Conversación (5 temas numerados, accionables, con preguntas sugeridas), 5) Manejo de Objeciones (tabla con columnas Objeción y Respuesta Recomendada), 6) Próximos Pasos Recomendados (3 acciones concretas post-reunión). Usá formato markdown."

**Competitive Intel:**
"Creá un cheat sheet de inteligencia competitiva en español con: TOP 3 COSAS A SABER (cada una con titular en negrita, 3-4 bullets de evidencia, y una recomendación 'Tu ángulo'), seguido de 'NÚMEROS PARA SOLTAR EN LA CONVERSACIÓN' con 8-10 estadísticas específicas con dólares y porcentajes, y finalmente 'QUIÉN MÁS ESTÁ EN SU ÓRBITA' con aliados y socios clave. Usá formato markdown."

**Deep Research Report:**
"Escribí un reporte de investigación profunda en español que resuma las tendencias macro que afectan el mundo de esta persona/empresa en los próximos 2 años. Incluí: 1) Resumen Ejecutivo, 2) Tabla de las 10 fuentes más importantes descubiertas (columnas: Fuente, Insight Clave, Por Qué Importa), 3) Análisis profundo de 5 temas clave. Usá formato markdown con headers."

---

### PASO 6 — Quiz y Flashcards via notebook_query

**Quiz (8 preguntas):**
"Generá exactamente 8 preguntas de opción múltiple sobre [person_name], su empresa y el mercado. Para cada pregunta: texto de la pregunta, 4 opciones (A, B, C, D), letra correcta. Formato: Q1: [pregunta]\nA) ...\nB) ...\nC) ...\nD) ...\nANSWER: [letra]"

**Flashcards (10 tarjetas):**
"Generá exactamente 10 flashcards Q&A sobre los hechos más importantes a memorizar antes de reunirse con [person_name]. Formato:\nQ: [pregunta]\nA: [respuesta]\n\n---"

---

### PASO 7 — Construir el HTML Dashboard

Crear: meetings/[meeting_id]/index.html

Diseño:
- Dark mode (#08080c) con gradientes purple/blue
- Font: Inter (Google Fonts), Icons: Font Awesome 6
- Navbar con branding y fecha
- Header con avatar (iniciales), nombre, cargo, empresa, tags de colores
- Stats row con 3 métricas clave del mercado
- Sidebar con 6 tabs: Executive Briefing, Competitive Intel, Deep Research, Knowledge Test, Flashcards, NotebookLM
- Tabs 1-3: render markdown con marked.js
- Tab 4: quiz interactivo con feedback visual correcto/incorrecto y score counter
- Tab 5: flashcards con flip 3D (CSS perspective), navegación flechas, dots de progreso
- Tab 6: link directo al notebook de NotebookLM
- Colores acento: azul #3b82f6 y dorado #f0b429
- Todo el contenido embebido directamente en el HTML (funciona offline)

---

### PASO 8 — Actualizar meetings.json

Leer el archivo actual y agregar:
{
  "id": "[meeting_id]",
  "date": "[YYYY-MM-DD]",
  "time": "[HH:MM]",
  "person": "[person_name]",
  "title": "[cargo]",
  "company": "[empresa]",
  "type": "[tipo]",
  "tags": ["tag1", "tag2", "tag3"],
  "notebooklm_url": "[URL del notebook]",
  "sources_count": [número],
  "path": "meetings/[meeting_id]/index.html",
  "generated": "[timestamp ISO]"
}

---

### PASO 9 — Git commit y push

```bash
git add .
git commit -m "feat: meeting prep [person_name] - [YYYY-MM-DD]"
git push origin main
```

Para el push necesitás el GitHub token configurado (ver setup).

---

### PASO 10 — Reporte final

Mostrar resumen por cada reunión procesada:
✅ Meeting Prep generado para: [person_name]
📅 Fecha: [fecha y hora]
📓 NotebookLM: [URL]
🌐 Dashboard: meetings/[meeting_id]/index.html
📊 Fuentes analizadas: [número]

---

### NOTAS IMPORTANTES

- Si hay múltiples reuniones con #prep → procesalas TODAS en paralelo
- Si meetings/[meeting_id]/ ya existe → saltarla (ya procesada)
- Si NotebookLM falla o timeout → generar los documentos directamente con la info del research web (modo degradado)
- El HTML debe funcionar sin servidor (todos los assets son CDN)
- Nunca exponer API keys o tokens en el HTML
- El campo "path" en meetings.json debe ser relativo a la raíz del repo
```

---

## Paso 7 — Cómo usar el sistema

### Marcar una reunión para prep
En Google Calendar, agregá `#prep` al título de cualquier evento:
```
Reunión con Juan Pérez - Mercado Libre #prep
Call con María García - Rappi #prep
```

### Trigger manual
En el chat de Cowork:
```
corre el proceso de meeting prep
```
o para una fecha específica:
```
corre el proceso pero para el calendar del lunes
```

### Trigger automático
Configurar la tarea como diaria a las 7:00 AM para que corra sola cada mañana.

---

## Troubleshooting frecuente

| Problema | Solución |
|---|---|
| `git push` falla con "no credentials" | Dar el GitHub PAT al agente una vez, lo guarda solo |
| NotebookLM timeout | El agente corre en modo degradado usando solo WebSearch |
| `index.lock` en el repo | El agente lo resuelve automáticamente vía `git commit-tree` |
| LinkedIn URLs no se agregan a NotebookLM | LinkedIn bloquea scraping — el agente lo skipea silenciosamente |
| "No hay reuniones con #prep" | Verificar que el título del evento tiene exactamente `#prep` |

---

## Resultado final

Por cada reunión `#prep` detectada, el agente genera:

| Artefacto | Contenido |
|---|---|
| **NotebookLM Notebook** | Todas las fuentes indexadas para consulta posterior |
| **Executive Briefing** | Perfil + empresa + mercado + preguntas + objeciones + próximos pasos |
| **Competitive Intel** | Top 3 cosas a saber + números para la conversación + órbita del contacto |
| **Deep Research** | Tendencias macro + tabla de 10 fuentes + análisis de 5 temas |
| **Quiz interactivo** | 8 preguntas de opción múltiple con feedback visual |
| **Flashcards 3D** | 10 tarjetas con flip animation para memorizar antes de la reunión |
| **HTML Dashboard** | Todo lo anterior en una sola página dark mode con tabs |
| **Git commit** | Versionado automático en GitHub |
