<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=600&size=26&duration=3200&pause=900&color=7AA2F7&center=true&vCenter=true&width=720&lines=Gast%C3%B3n+Villagra+(Guille);Infrastructure+Engineer+%26+DevOps;Automation+%2B+LLM+Systems+Builder;Producci%C3%B3n+real%2C+sin+atajos+manuales" alt="Typing SVG" />

**Infraestructura multi-tenant · Automatización end-to-end · Sistemas con LLMs en producción**

📍 Salta, Argentina &nbsp;·&nbsp; 🕒 UTC-3 &nbsp;·&nbsp; 🌐 Español / English (técnico)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/gaston010gv/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/gastongv)
![Profile views](https://komarev.com/ghpvc/?username=gastongv&style=flat-square&color=7aa2f7&label=views)

</div>

---

## `$ whoami`

```yaml
name:      Gastón Villagra
role:      Infrastructure Engineer · DevOps · Automation Architect
location:  Salta, AR (remote-first)
focus:
  - Infraestructura multi-tenant en producción (Docker, Coolify, Nginx)
  - Observabilidad: Prometheus, Grafana, Loki + exporters propios
  - Automatización e integraciones: n8n, WhatsApp Business / Meta Graph API, Chatwoot
  - Sistemas con LLMs: caching semántico, agentes, MCP servers, LangGraph
principles:
  - "Si se hace dos veces a mano, se automatiza"
  - "Credenciales fuera del código, siempre (JWT, secrets, least privilege)"
  - "Sin métricas no hay producción, hay esperanza"
education:
  - Tecnicatura en Gestión de Infraestructura Cloud y DevOps — UPATecO (en curso)
  - Tecnicatura en Desarrollo de Software — completa
background: SAP Basis / Security (PFCG, SU01, SM59, STRUST, SAML2/SSO, ALE/IDoc)
```

---

## 🔭 En qué estoy ahora

- 🛡️ **Audit Analyzer Shell** — herramienta open source de auditoría de infraestructura *one-shot* vía SSH (sin agente en el host). Motor determinístico de checks + pipeline **LangGraph** opcional para correlación y reportes. CLI → TUI (Textual) → GUI (PySide6). Desarrollo guiado por specs (SDD). `🚧 en desarrollo`
- 🤖 **Agentes de IA para WhatsApp e Instagram** — agente conversacional + CRM + dashboard sobre la API oficial de WhatsApp Business.
- 🎯 Profundizando en **arquitecturas LLM de producción**: caching multi-capa, evaluación, costos y observabilidad de agentes.

---

## 🛠️ Stack

<div align="center">

**Infra & DevOps**

<img src="https://skillicons.dev/icons?i=linux,docker,nginx,githubactions,bash,git&theme=dark" />

<img src="https://img.shields.io/badge/Coolify-6366F1?style=flat-square&logo=coolify&logoColor=white" />
<img src="https://img.shields.io/badge/Traefik-24A1C1?style=flat-square&logo=traefikproxy&logoColor=white" />
<img src="https://img.shields.io/badge/Easypanel-0F172A?style=flat-square&logoColor=white" />
<img src="https://img.shields.io/badge/Cloudflare-F38020?style=flat-square&logo=cloudflare&logoColor=white" />

**Backend & Datos**

<img src="https://skillicons.dev/icons?i=python,django,flask,fastapi,js,nodejs,postgres,supabase,redis&theme=dark" />

<img src="https://img.shields.io/badge/pgvector-336791?style=flat-square&logo=postgresql&logoColor=white" />
<img src="https://img.shields.io/badge/PL%2FpgSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white" />

**Automatización & Mensajería**

<img src="https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white" />
<img src="https://img.shields.io/badge/WhatsApp_Business_API-25D366?style=flat-square&logo=whatsapp&logoColor=white" />
<img src="https://img.shields.io/badge/Meta_Graph_API-0866FF?style=flat-square&logo=meta&logoColor=white" />
<img src="https://img.shields.io/badge/Evolution_API-1F2937?style=flat-square&logoColor=white" />
<img src="https://img.shields.io/badge/Chatwoot-1F93FF?style=flat-square&logo=chatwoot&logoColor=white" />

**AI / LLM**

<img src="https://img.shields.io/badge/Claude_API-D97757?style=flat-square&logo=anthropic&logoColor=white" />
<img src="https://img.shields.io/badge/Claude_Code-D97757?style=flat-square&logo=anthropic&logoColor=white" />
<img src="https://img.shields.io/badge/MCP-000000?style=flat-square&logoColor=white" />
<img src="https://img.shields.io/badge/LangGraph-1C3C3C?style=flat-square&logo=langchain&logoColor=white" />

**Observabilidad**

<img src="https://skillicons.dev/icons?i=grafana,prometheus&theme=dark" />

<img src="https://img.shields.io/badge/Loki-F46800?style=flat-square&logo=grafana&logoColor=white" />
<img src="https://img.shields.io/badge/Alertmanager-E6522C?style=flat-square&logo=prometheus&logoColor=white" />

</div>

---

## 🧭 Cómo se ve lo que opero

```mermaid
flowchart LR
    U([Clientes finales<br/>WhatsApp · Instagram]) --> META[Meta Graph API<br/>WhatsApp Business]
    META -->|webhooks| PX[Webhook proxy<br/>Nginx + watchdog]
    PX --> N8N[n8n<br/>workflows]
    N8N <--> CW[Chatwoot<br/>multi-inbox]
    N8N <--> AI[Agentes IA<br/>+ LLM cache gateway]
    N8N --> DB[(Supabase / PostgreSQL<br/>multi-tenant)]
    AI --> R[(Redis)]
    AI --> DB

    subgraph OBS [Observabilidad]
      EXP[Exporters Python<br/>n8n · Chatwoot] --> PROM[Prometheus]
      PROM --> GRAF[Grafana / Loki]
      PROM --> ALR[Alertas<br/>dedupe + horario hábil]
    end

    N8N -.métricas.-> EXP
    CW -.métricas.-> EXP
    DB -.backup diario.-> GH[(GitHub<br/>DDL + data versionado)]
```

---

## 🚀 Proyectos destacados

| Proyecto | Qué resuelve | Stack |
|---|---|---|
| 🔒 **Plataforma de Monitoreo Multi-Cliente** | Observabilidad por cliente con dashboards aislados, alertas por email y panel admin. | `Flask` `JWT + refresh` `TOTP 2FA` `RBAC 3 roles` `Prometheus` `Grafana` `Loki` |
| 🌙 **Escuadrón Nocturno de Monitoreo** | Revisión autónoma nocturna de infra con sub-agentes especializados (infra / containers / código) y límites de seguridad a nivel OS. | `Claude Code CLI` `Bash` `Cron` `cost tracking real` |
| ⚡ **LLM Cache Gateway** | Reduce costo y latencia de LLMs con 3 capas de caché: exacta, semántica y prompt caching nativo. | `FastAPI` `Redis` `pgvector` `Prompt caching` |
| 🧩 **Redmine MCP Server** | Expone Redmine como herramientas MCP para operar tickets desde Claude + recordatorios diarios automáticos. | `Node.js` `MCP` `Ruby rake task` `Cron` |
| 📡 **WhatsApp Webhook Proxy** | Reverse proxy dedicado para evitar rate-limiting de Meta por ASN del proveedor, con resolución DNS dinámica y watchdog. | `Nginx` `DigitalOcean` `Bash watchdog` |
| 💾 **Backup Multi-Tenant (Supabase)** | Respaldo full DDL + data versionado en GitHub, con reporte automático por email. | `n8n` `PL/pgSQL` `GitHub` |
| 🔁 **Migración Zero-Downtime** | Migración completa de stack de VPS legado a nueva infraestructura sin downtime perceptible. | `Docker` `Coolify` `Nginx` `DNS/SSL` |
| 🛡️ **Audit Analyzer Shell** `🚧` | Auditoría de seguridad/infra one-shot por SSH con reportes asistidos por LLM. Open source. | `Python` `LangGraph` `Textual` `PySide6` |

---

## 📊 GitHub

<div align="center">

<img height="165" src="https://github-stats-extended.vercel.app/api?username=gastongv&show_icons=true&theme=tokyonight&hide_border=true&include_all_commits=true&count_private=true" />
<img height="165" src="https://github-stats-extended.vercel.app/api/top-langs/?username=gastongv&layout=compact&theme=tokyonight&hide_border=true&langs_count=8" />

<img src="https://streak-stats.demolab.com?user=gastongv&theme=tokyonight&hide_border=true" />

<img src="https://github-readme-activity-graph.vercel.app/graph?username=gastongv&theme=tokyo-night&hide_border=true&area=true" width="100%" />

</div>

---

<div align="center">

```bash
$ echo "Automatización, alta disponibilidad y seguridad — sin atajos manuales."
```

<sub>Última actualización: septiembre 2026</sub>

</div>
