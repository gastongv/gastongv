<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:1a1b27,50:24283b,100:7aa2f7&height=200&section=header&text=Gast%C3%B3n%20Villagra&fontSize=48&fontColor=c0caf5&fontAlignY=36&desc=Infraestructura%20%C2%B7%20Automatizaci%C3%B3n%20%C2%B7%20IA%20aplicada%20en%20producci%C3%B3n&descSize=16&descAlignY=58&descColor=a9b1d6&animation=fadeIn" width="100%" />

<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=600&size=22&duration=2800&pause=800&color=7AA2F7&center=true&vCenter=true&width=760&lines=%24+uptime+--mis-clientes+%E2%86%92+producci%C3%B3n+real%2C+24%2F7;Construyo+la+infra+que+nadie+ve+hasta+que+falla;IA+donde+suma.+Determin%C3%ADstico+donde+importa.;Si+se+hace+dos+veces+a+mano%2C+se+automatiza." alt="Typing SVG" />

📍 Salta, Argentina &nbsp;·&nbsp; 🕒 UTC-3 &nbsp;·&nbsp; 💼 Infrastructure Engineer · DevOps · AI Automation

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/gaston010gv/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/gastongv)
![Profile views](https://komarev.com/ghpvc/?username=gastongv&style=flat-square&color=7aa2f7&label=views)

</div>

---

> [!NOTE]
> **TL;DR** —  Opero un stack multi-tenant en producción (n8n, Chatwoot, WhatsApp Business API, Supabase, Redis, Docker) para clientes reales, lo monitoreo con Prometheus/Grafana, y le sumo **IA donde tiene sentido**: agentes conversacionales, caché semántico de LLMs, MCP servers y sub-agentes que revisan la infra mientras duermo.
>
> Abajo no hay una lista de tecnologías. Hay **cómo se armó cada cosa, qué se rompió y cómo lo resolví.** 👇

---

## 🗺️ El mapa: lo que opero todos los días

```mermaid
flowchart LR
    U([👤 Usuarios<br/>WhatsApp · Instagram]) --> META[Meta Graph API]
    META -->|webhooks| PX[🛡️ Webhook proxy<br/>Nginx + watchdog]
    PX --> N8N[⚙️ n8n<br/>orquestación]
    N8N <--> CW[💬 Chatwoot<br/>multi-inbox]
    N8N <--> AG[🤖 Agentes IA]
    AG <--> GW[⚡ LLM Cache Gateway]
    GW --> R[(Redis<br/>exact match)]
    GW --> V[(pgvector<br/>semántico)]
    GW --> LLM[☁️ LLM API<br/>prompt caching]
    N8N --> DB[(🐘 Supabase<br/>multi-tenant)]

    subgraph OBS [🔭 Observabilidad]
      EXP[Exporters Python] --> PROM[Prometheus] --> GRAF[Grafana · Loki]
      PROM --> ALR[🚨 Alertas<br/>dedupe + horario hábil]
    end

    subgraph NIGHT [🌙 Escuadrón Nocturno]
      SA1[Agente infra] & SA2[Agente containers] & SA3[Agente código]
    end

    N8N -.-> EXP
    CW -.-> EXP
    NIGHT -.revisa.-> OBS
    DB -.backup diario.-> GH[(GitHub<br/>DDL + data)]
```

---

## 🤖 IA aplicada: qué hago, cómo y dónde NO la uso

La IA no es el producto, es una **capa más del stack** con los mismos requisitos que el resto: costos medibles, fallas controladas y credenciales seguras.

### Mi regla de diseño

```text
Capa 1 → Determinístico   (checks, reglas, SQL, regex)   → siempre corre, barato, auditable
Capa 2 → LLM              (clasificar, correlacionar, redactar) → opcional, cacheado, con límites
Capa 3 → Humano           (decisiones irreversibles)      → la IA propone, no ejecuta
```

### Dónde la aplico

| Caso de uso | Qué hace la IA | Cómo lo implemento | Qué la mantiene bajo control |
|---|---|---|---|
| 💬 **Agentes conversacionales** (WhatsApp / Instagram) | Atiende, califica y deriva consultas; se integra a un CRM | n8n + API oficial de WhatsApp Business + Chatwoot para handoff humano | Handoff a humano, contexto por tenant, prompts versionados |
| ⚡ **LLM Cache Gateway** | Evita pagar dos veces por la misma respuesta | Proxy FastAPI con 3 capas: Redis (exacta) → pgvector (semántica) → prompt caching nativo | Umbral de similitud, TTL por tipo de consulta, métricas de hit-rate |
| 🌙 **Monitoreo nocturno autónomo** | Revisa infra, containers y código; reporta hallazgos | Claude Code CLI con 3 sub-agentes especializados, disparado por cron | Límites de seguridad a nivel OS, solo lectura, tracking real de tokens y costo |
| 🧩 **MCP Servers** | Permite operar herramientas internas (tickets) en lenguaje natural | MCP server en Node.js sobre la API de Redmine | Scopes acotados, versiones pineadas del SDK |
| 🛡️ **Auditoría de infraestructura** | Correlaciona hallazgos y redacta el reporte | Motor determinístico + pipeline LangGraph opcional | El LLM nunca decide si algo *es* un hallazgo; solo lo explica |

### Lo que sé hacer con LLMs

<table>
<tr>
<td valign="top" width="50%">

**🏗️ Arquitectura**
- Tool use / function calling
- MCP servers (diseño, deploy, debugging)
- Agentes y sub-agentes con roles separados
- Orquestación con LangGraph y n8n
- RAG con pgvector

</td>
<td valign="top" width="50%">

**⚙️ Operación**
- Caching exacto, semántico y prompt caching
- Control de costos por token y por tenant
- Guardrails y límites de permisos a nivel OS
- Human-in-the-loop para acciones críticas
- Observabilidad de agentes (logs, costo, latencia)

</td>
</tr>
</table>

<details>
<summary><b>🔍 Ver cómo funciona el LLM Cache Gateway por dentro</b></summary>

<br/>

```mermaid
sequenceDiagram
    autonumber
    participant C as Cliente
    participant G as Gateway (FastAPI)
    participant R as Redis
    participant V as pgvector
    participant L as LLM API

    C->>G: prompt
    G->>R: hash(prompt normalizado)
    alt Hit exacto
        R-->>G: respuesta
        G-->>C: ⚡ ~ms, costo 0
    else Miss
        G->>V: embedding + búsqueda por similitud
        alt Similitud ≥ umbral
            V-->>G: respuesta semántica
            G-->>C: 🧠 costo embedding
        else Miss
            G->>L: prompt con cache_control
            L-->>G: respuesta
            G->>R: guardar
            G->>V: guardar embedding
            G-->>C: ☁️ costo completo (con prefijo cacheado)
        end
    end
```

```python
async def resolve(prompt: str, tenant: str) -> Answer:
    key = cache_key(tenant, normalize(prompt))

    if hit := await redis.get(key):                      # Capa 1: exacta
        return Answer(hit, layer="exact")

    emb = await embed(prompt)
    if hit := await pgvector.nearest(tenant, emb, min_sim=THRESHOLD):  # Capa 2: semántica
        return Answer(hit.text, layer="semantic")

    text = await llm.complete(prompt, cache_system_prompt=True)       # Capa 3: prompt caching
    await asyncio.gather(
        redis.set(key, text, ex=ttl_for(prompt)),
        pgvector.upsert(tenant, emb, text),
    )
    return Answer(text, layer="llm")
```

</details>

---

## 🔥 Cómo se armó: historias de producción

Cada proyecto tiene un problema real detrás. Estos son los postmortems cortos.

<details>
<summary><b>📡 Meta empezó a rechazar webhooks… y el problema no era mi código</b></summary>

<br/>

| | |
|---|---|
| **Síntoma** | Webhooks de WhatsApp fallando de forma intermitente. El código y los certificados estaban bien. |
| **Causa raíz** | Rate-limiting de Meta aplicado **por ASN del proveedor de VPS**, no por IP ni por app. |
| **Fix** | Reverse proxy dedicado en otro proveedor, con Nginx configurado para **resolución DNS dinámica** (Nginx cachea el DNS del upstream al arrancar) + script watchdog en cron que verifica y levanta el servicio. |
| **Aprendizaje** | Cuando todo "está bien" y falla igual, bajá de capa: red, ASN, DNS. |

</details>

<details>
<summary><b>🚨 La plataforma de alertas me alertaba… de todo, todo el tiempo</b></summary>

<br/>

| | |
|---|---|
| **Síntoma** | Spam de alertas. Nadie las leía, que es lo mismo que no tener alertas. |
| **Causa raíz** | Reglas con `for: 0m`: cualquier pico de un scrape disparaba, se resolvía y volvía a disparar (flapping). |
| **Fix** | Ventanas `for:` razonables por severidad + workflow en n8n con **deduplicación por estado** y filtro de horario hábil para alertas no críticas. |
| **Aprendizaje** | Una alerta tiene que ser accionable. Si no, es ruido con formato. |

</details>

<details>
<summary><b>🧩 El MCP server dejó de funcionar sin que yo tocara nada</b></summary>

<br/>

| | |
|---|---|
| **Síntoma** | La integración de Redmine con Claude Desktop dejó de responder de un día para otro. |
| **Causa raíz** | Breaking change en una versión mayor del SDK de MCP, instalada automáticamente por no tener versión fijada. |
| **Fix** | Pinear la versión del SDK en la configuración del cliente y reparar el entorno Python (`uv`) corrupto. |
| **Aprendizaje** | En tooling de IA, que se mueve rápido, **pinear versiones no es opcional.** |

</details>

<details>
<summary><b>🌙 ¿Y si la infraestructura se auditara sola de noche?</b></summary>

<br/>

```mermaid
flowchart TB
    CRON[⏰ cron] --> ORQ[Orquestador Bash]
    ORQ --> A1[🖥️ Sub-agente Infra<br/>disco · memoria · servicios]
    ORQ --> A2[🐳 Sub-agente Containers<br/>estado · restarts · logs]
    ORQ --> A3[📝 Sub-agente Código<br/>cambios · config drift]
    A1 & A2 & A3 --> REP[📋 Reporte consolidado]
    ORQ --> COST[💰 Tracking de tokens y costo real]
    SEC{{🔒 Boundaries a nivel OS<br/>usuario sin privilegios · solo lectura}} -.-> A1 & A2 & A3
```

**La clave no fue el prompt, fue el sandbox:** cada sub-agente corre con un usuario del sistema sin privilegios y acceso de solo lectura. Aunque el modelo "quiera" hacer algo, el sistema operativo no lo deja.

</details>

<details>
<summary><b>💾 Backups de Supabase multi-tenant sin depender de nadie</b></summary>

<br/>

| | |
|---|---|
| **Problema** | Varios tenants en Supabase, sin un backup completo propio (esquema + datos) y versionado. |
| **Solución** | Función PL/pgSQL `export_full_backup()` que genera DDL + data, orquestada desde n8n, versionada en GitHub y con reporte automático por email. |
| **Resultado** | Cada backup es un commit: se puede ver qué cambió en el esquema entre dos días con un `git diff`. |

</details>

<details>
<summary><b>🔁 Migrar todo el stack sin que los clientes se enteren</b></summary>

<br/>

Migración de VPS legado a nueva infraestructura con Coolify + Nginx: TTL de DNS bajado días antes, servicios levantados en paralelo, sincronización de datos, validación de SSL y corte por DNS. **Sin downtime perceptible** para clientes en producción.

</details>

---

## 🚀 Proyectos

| | Proyecto | En una línea | Stack |
|---|---|---|---|
| 🔒 | **Plataforma de Monitoreo Multi-Cliente** | Dashboards aislados por cliente, alertas y panel admin | `Flask` `JWT+refresh` `TOTP 2FA` `RBAC` `Prometheus` `Grafana` `Loki` |
| 🌙 | **Escuadrón Nocturno** | Sub-agentes de IA que auditan la infra cada noche | `Claude Code` `Bash` `Cron` `OS sandboxing` |
| ⚡ | **LLM Cache Gateway** | 3 capas de caché para bajar costo y latencia de LLMs | `FastAPI` `Redis` `pgvector` |
| 🤖 | **Agentes IA para mensajería** | Agente + CRM + dashboard sobre WhatsApp / Instagram | `n8n` `WhatsApp Business API` `Chatwoot` |
| 🧩 | **Redmine MCP Server** | Operar tickets en lenguaje natural + recordatorios diarios | `Node.js` `MCP` `Ruby` |
| 📡 | **WhatsApp Webhook Proxy** | Esquivar rate-limiting por ASN | `Nginx` `Bash watchdog` |
| 💾 | **Backup Multi-Tenant** | DDL + data versionado en Git | `n8n` `PL/pgSQL` `GitHub` |
| 🛡️ | **Audit Analyzer Shell** `🚧 open source` | Auditoría one-shot por SSH con reportes asistidos por LLM | `Python` `LangGraph` `Textual` `PySide6` |

---

## 🛠️ Stack

<div align="center">

<img src="https://skillicons.dev/icons?i=linux,docker,nginx,githubactions,bash,git,python,django,flask,fastapi,js,nodejs,postgres,supabase,redis,grafana,prometheus&theme=dark&perline=9" />

<br/><br/>

<img src="https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white" />
<img src="https://img.shields.io/badge/WhatsApp_Business_API-25D366?style=flat-square&logo=whatsapp&logoColor=white" />
<img src="https://img.shields.io/badge/Meta_Graph_API-0866FF?style=flat-square&logo=meta&logoColor=white" />
<img src="https://img.shields.io/badge/Chatwoot-1F93FF?style=flat-square&logo=chatwoot&logoColor=white" />
<img src="https://img.shields.io/badge/Evolution_API-1F2937?style=flat-square" />
<img src="https://img.shields.io/badge/Coolify-6366F1?style=flat-square&logo=coolify&logoColor=white" />
<img src="https://img.shields.io/badge/Traefik-24A1C1?style=flat-square&logo=traefikproxy&logoColor=white" />
<img src="https://img.shields.io/badge/Loki-F46800?style=flat-square&logo=grafana&logoColor=white" />
<img src="https://img.shields.io/badge/pgvector-336791?style=flat-square&logo=postgresql&logoColor=white" />
<br/>
<img src="https://img.shields.io/badge/Claude_API-D97757?style=flat-square&logo=anthropic&logoColor=white" />
<img src="https://img.shields.io/badge/Claude_Code-D97757?style=flat-square&logo=anthropic&logoColor=white" />
<img src="https://img.shields.io/badge/MCP-000000?style=flat-square" />
<img src="https://img.shields.io/badge/LangGraph-1C3C3C?style=flat-square&logo=langchain&logoColor=white" />

</div>

<details>
<summary><b>🎓 Formación y background</b></summary>

<br/>

- 🎓 **Tecnicatura en Gestión de Infraestructura Cloud y DevOps** — UPATecO *(en curso)*
- 🎓 **Tecnicatura en Desarrollo de Software** — completa
- 🏢 **SAP Basis / Security** — roles y usuarios (PFCG, SU01), RFC (SM59), certificados (STRUST), SSO con SAML2 / Azure AD, migraciones ALE/IDoc, Fiori Launchpad

</details>

---

## 📊 Actividad

<div align="center">

<img height="165" src="https://github-stats-extended.vercel.app/api?username=gastongv&show_icons=true&theme=tokyonight&hide_border=true&include_all_commits=true&count_private=true" />
<img height="165" src="https://github-stats-extended.vercel.app/api/top-langs/?username=gastongv&layout=compact&theme=tokyonight&hide_border=true&langs_count=8" />

<img src="https://streak-stats.demolab.com?user=gastongv&theme=tokyonight&hide_border=true" />

<img src="https://github-readme-activity-graph.vercel.app/graph?username=gastongv&theme=tokyo-night&hide_border=true&area=true" width="100%" />

</div>

---

<div align="center">

### 💬 ¿Hablamos?

Infraestructura que tiene que funcionar a las 3 AM, automatizaciones que ahorran horas reales o IA que tiene que rendir cuentas de lo que cuesta: **me interesa.**

[![LinkedIn](https://img.shields.io/badge/Escribime_en_LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/gaston010gv/)

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:7aa2f7,50:24283b,100:1a1b27&height=110&section=footer" width="100%" />

</div>
