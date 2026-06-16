# RFC 0001 — Presence Map (Frota 162)

**Status:** Draft
**Autor:** Geovane Guibes (via Sisyphus)
**Data:** 2026-06-16
**Inspiração:** [Valve Handbook, p.43 — "user check"](https://www.valvesoftware.com/en/publications)

---

## 1. Resumo

Página interna `/presence` (codename: **user**) que mostra **quem está trabalhando agora, em qual projeto, e o quê** — versão remote-first do mapa de escritório da Valve.

Não é geolocalização. É **presença social digital**: substitui o "está online?" do Slack/WhatsApp por uma visão única, ambient-aware, sobre o time.

---

## 2. Motivação

### Problema
- Time 100% remoto → não tem "passar na mesa do fulano".
- Saber em que projeto alguém está exige perguntar no Slack ou abrir Jira/GitLab.
- Onboarding novo dev → não sabe quem faz o quê.
- Reuniões/pings interrompem deep work sem visibilidade do contexto da pessoa.

### Por que agora
Frota 162 já tem fontes de sinal de presença espalhadas (GitLab, Jira, calendário, IDE). Falta **agregador**.

### Não-objetivos
- Vigilância / produtividade tracking (NÃO é Hubstaff).
- Time-tracking automático (NÃO substitui Toggl/Clockify).
- Status forçado (NÃO obriga ninguém a aparecer).

---

## 3. Cenários de uso

| Persona | Cenário | O que vê |
|---|---|---|
| Dev | "Quem mexe no módulo de billing?" | Lista de quem commitou em `billing/*` últimos 30d |
| PM | "Quem tá disponível pra pair agora?" | Devs com status `available`, não em foco profundo |
| Onboarding | "Quem é o João?" | Bio, projetos atuais, especialidades, fuso |
| Tech Lead | "Estado do time hoje" | Heatmap: focando / em reunião / off |
| Qualquer um | "Cadê fulano?" | Última atividade + projeto atual + status |

---

## 4. Proposta — UX

### 4.1 Tela principal `/presence`

```
┌─────────────────────────────────────────────────────────────┐
│  presence — frota 162                          [filtros ▾]  │
├─────────────────────────────────────────────────────────────┤
│  ⬤ AGORA TRABALHANDO (12)                                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐        │
│  │ 🟢 Geo   │ │ 🟢 Maria │ │ 🔴 João  │ │ 🟡 Lucas │        │
│  │ Gestão   │ │ Backoffice│ │ FOCO     │ │ Reunião  │        │
│  │ MR #142  │ │ JIRA-301 │ │ não pertu│ │ até 15h  │        │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘        │
│                                                              │
│  ⬤ FORA / OFF (3)                                           │
│  ...                                                         │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Card de pessoa (hover/click)

```
┌──────────────────────────────────────┐
│ 🟢 Geovane Guibes — @geovaneguibes   │
│ Senior Backend · TZ: America/Fortaleza│
├──────────────────────────────────────┤
│ 📌 Agora                              │
│    user-organization-frota           │
│    MR !142 (presence map)            │
│    Último commit: há 12min            │
│                                      │
│ 🎯 Projetos                           │
│    gestor-frota · frota162-backoffice│
│                                      │
│ 🧠 Stack                              │
│    NestJS · TypeORM · React          │
│                                      │
│ 💬 Bio                                │
│    "Café > tudo. Pergunte sobre DDD" │
│                                      │
│ ⏰ Disponibilidade                    │
│    Mais ativo: 9h-12h, 14h-18h BRT   │
└──────────────────────────────────────┘
```

### 4.3 Filtros
- Por projeto (multi-select)
- Por status (working / focus / meeting / off)
- Por especialidade (backend / frontend / qa / devops)
- Por timezone

---

## 5. Proposta — Sinais

Coleta **passiva e opt-in** de múltiplas fontes. Usuário controla quais expor.

| Sinal | Fonte | Detecta | Custo impl |
|---|---|---|---|
| **Atividade Git** | GitLab webhook (push/commit/MR) | Projeto atual, último ato | Baixo |
| **Issue ativa** | Jira webhook (status change) | Card "in progress" | Baixo |
| **Calendário** | Google Calendar API | Em reunião / livre | Médio |
| **IDE plugin** | VSCode extension (opcional) | File atual, idle, foco | Alto |
| **Status manual** | UI button | Override explícito | Trivial |
| **Slack presence** | Slack API | Online/away | Baixo |

### Status derivado (precedência)

```
manual_override > calendar_busy > ide_focus > git_active > slack_online > offline
```

- `manual_override`: usuário clicou "🔴 foco" ou "🟢 disponível"
- `calendar_busy`: evento "busy" no calendar agora
- `ide_focus`: VSCode plugin reporta digitação ativa últimos 5min
- `git_active`: commit/push últimos 30min
- `slack_online`: presença Slack
- senão: `offline`

---

## 6. Arquitetura técnica

### 6.1 Stack (alinhado ao boilerplate Frota 162)

- **Backend:** NestJS + TypeORM (matches `dashboard-devs-tool/backend`)
- **Frontend:** React SPA (matches `frota162-backoffice`)
- **DB:** PostgreSQL (presence atual + histórico de sinais)
- **Cache:** Redis (estado real-time, TTL 5min)
- **Real-time:** WebSocket (NestJS Gateway) para updates ao vivo
- **Auth:** SSO interno Frota 162 (reusa)

### 6.2 Módulos novos

```
src/
├── presence/
│   ├── domain/
│   │   ├── presence-status.entity.ts
│   │   ├── presence-signal.entity.ts
│   │   └── user-profile.entity.ts
│   ├── application/
│   │   ├── compute-status.use-case.ts
│   │   ├── ingest-signal.use-case.ts
│   │   └── get-team-snapshot.use-case.ts
│   ├── infrastructure/
│   │   ├── gitlab-webhook.controller.ts
│   │   ├── jira-webhook.controller.ts
│   │   ├── presence.gateway.ts        # WebSocket
│   │   └── presence.repository.ts
│   └── presentation/
│       └── presence.controller.ts     # REST API
```

### 6.3 Modelo de dados

```typescript
// user_profile
{
  id: UUID
  user_id: UUID (FK)
  bio: string (max 280)
  stack: string[]           // ["NestJS", "React"]
  specialties: string[]     // ["backend", "ddd"]
  timezone: string          // "America/Fortaleza"
  working_hours: JSON       // { mon: ["09:00","18:00"], ... }
  avatar_url: string
}

// presence_signal (append-only, particionada por dia)
{
  id: UUID
  user_id: UUID
  source: enum (gitlab|jira|calendar|ide|slack|manual)
  signal_type: enum (commit|mr|focus|meeting|status|...)
  payload: JSON
  project_ref: string?      // "user-organization-frota"
  occurred_at: timestamp
}

// presence_status (snapshot atual, 1 por user)
{
  user_id: UUID (PK)
  status: enum (available|focus|meeting|off)
  current_project: string?
  current_activity: string? // "MR !142"
  source: enum
  expires_at: timestamp     // TTL — após, volta a 'offline'
  updated_at: timestamp
}
```

### 6.4 Fluxo de evento

```
GitLab push → webhook → POST /webhooks/gitlab
  → IngestSignalUseCase
     → grava presence_signal
     → ComputeStatusUseCase
        → calcula status derivado (precedência)
        → upsert presence_status (TTL 30min)
        → emit WebSocket event 'presence:update'
  → Frontend recebe via WS → atualiza card
```

### 6.5 APIs

```
GET  /presence/team            # snapshot de todos
GET  /presence/user/:id        # detalhe + histórico recente
PATCH /presence/me             # override manual
POST /presence/profile         # editar bio/stack
WS   /presence/live            # updates real-time

POST /webhooks/gitlab          # GitLab push/MR
POST /webhooks/jira            # Jira issue events
POST /webhooks/calendar        # Calendar push notifications
```

---

## 7. Privacidade & Trust (CRÍTICO)

> Sem isso bem-feito, vira **vigilância** e o time rejeita.

### Princípios
1. **Opt-in por fonte.** Cada sinal pode ser desligado individualmente.
2. **Sem histórico exposto.** UI mostra "agora" + agregados de 30d, NUNCA timeline minuto-a-minuto pra outros.
3. **Self-view completa.** Usuário vê TUDO sobre si (transparência total).
4. **Cargo ≠ acesso.** Tech lead NÃO vê dados extras. Sem hierarquia espia.
5. **Pausa global.** Botão "pausar presence por 4h" — fica off pra todos.
6. **Sem export.** API não tem endpoint de bulk download de histórico.
7. **Retention 30 dias.** Sinais brutos expiram. Só agregados sobrevivem.

### Settings UI
```
[x] GitLab — mostrar projeto atual
[x] Jira — mostrar card ativo
[ ] Calendário — mostrar "em reunião"
[x] IDE — sinalizar foco
[x] Permitir colegas verem meu fuso
[ ] Mostrar última atividade timestamp
```

---

## 8. Roadmap proposto

### Fase 1 — MVP (2 sprints)
- [ ] `user_profile` entity + CRUD (bio/stack/tz)
- [ ] `presence_signal` ingestão via GitLab webhook
- [ ] `presence_status` compute básico (git_active + manual)
- [ ] REST `/presence/team`
- [ ] UI mínima: grid de cards com status

### Fase 2 — Real-time (1 sprint)
- [ ] WebSocket gateway
- [ ] Jira webhook
- [ ] Settings UI (opt-in por fonte)

### Fase 3 — Sinais ricos (2 sprints)
- [ ] Calendar integration
- [ ] VSCode extension (opcional)
- [ ] Filtros avançados

### Fase 4 — Inteligência (futuro)
- [ ] "Pessoas similares" (mesma stack)
- [ ] Sugestão de mentor por especialidade
- [ ] Heatmap de overlap timezone

---

## 9. Riscos & mitigações

| Risco | Severidade | Mitigação |
|---|---|---|
| Vira vigilância | Alta | Privacy section §7 imposta como gate de lançamento |
| Ninguém atualiza bio | Média | Bio puxa de GitLab profile no onboarding |
| Sinais falsos (commit antigo) | Média | TTL agressivo (30min) + precedência clara |
| Sobrecarga de webhooks | Baixa | Queue (Bull/Redis) + rate limit |
| Adoção baixa | Média | Lançar com 5 voluntários, expandir orgânico |

---

## 10. Alternativas consideradas

- **Slack custom status** — já existe, ninguém atualiza. Sem agregação cross-fonte.
- **Geekbot daily standup** — assíncrono mas pontual, não "agora".
- **Comprar SaaS (Tandem, Pragli)** — caro, dados fora, não integra Jira/GitLab interno.

**Build > buy** porque: integração com stack interna, controle total de privacidade, custo marginal baixo dado boilerplate já maduro.

---

## 11. Decisões abertas

- [ ] **D1:** WebSocket vs Server-Sent Events? (WS suporta bidirecional, SSE é mais simples)
- [ ] **D2:** IDE plugin agora ou só Fase 3? (alto valor, alto custo)
- [ ] **D3:** Mostrar "última vez online" pros outros? (privacidade vs utilidade)
- [ ] **D4:** Integra com `dashboard-devs-tool` ou standalone?
- [ ] **D5:** Quem é o stakeholder/owner do produto?

---

## 12. Next steps

1. Validar RFC com 2-3 devs do time (feedback de privacidade ESPECIALMENTE)
2. Decidir D1–D5
3. Abrir issues no GitLab por fase
4. POC Fase 1 com 5 voluntários

---

## Referências

- Valve Employee Handbook (2012), p.43 — origem do conceito
- [Tandem.chat](https://tandem.chat) — versão SaaS moderna
- [Slack presence API](https://api.slack.com/docs/presence-and-status)
- [GitLab webhook events](https://docs.gitlab.com/ee/user/project/integrations/webhook_events.html)
