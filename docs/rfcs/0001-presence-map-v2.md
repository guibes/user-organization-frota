# RFC 0001 v2 — Presence Map & Onboarding (Frota 162)

**Status:** Draft v2
**Autor:** Geovane Guibes (via Sisyphus)
**Data inicial:** 2026-06-16
**Última revisão:** 2026-06-16 · v2
**Supersedes:** [`0001-presence-map.md`](./0001-presence-map.md) (v1)
**Inspiração:** [Valve Handbook, p.43 — "user check"](https://www.valvesoftware.com/en/publications) · GitLab Handbook · Notion · Linear

> **Changelog v1 → v2:**
> - Slack promovido de "sinal" para **integração de primeira classe** (deeplinks DM/call, presence cross-source)
> - Adicionado **Onboarding** como produto irmão do mapa — não-objetivo virou pilar
> - **Google Workspace** definido como camada de auth + Drive sync (manuais ancorados em Google Docs)
> - **Manual pessoal** por colab (não compartilhado) com gating progressivo
> - **Template editor** versionado para people ops
> - Decisões D1-D5 resolvidas (ver §13)
> - Roadmap refeito em 5 fases (era 4)
> - Adicionadas decisões NOVAS D6-D12 (LGPD, formatura, mobile, etc)

---

## 1. Resumo

Plataforma interna `presence` da Frota 162 com **dois produtos integrados**:

1. **Mapa do time** — versão remote-first do "user check" da Valve. Mostra quem está trabalhando agora, em qual projeto, em qual status (disponível / foco / reunião / off), com bio, stack, fuso, e atalhos pra chamar no Slack.

2. **Onboarding interativo** — todo novo colab recebe um **manual pessoal** (Google Doc + wrapper UI). Ele edita, marca tarefas, recebe comentários do buddy e EM. Templates versionados por role.

Auth: **Google Workspace SSO** (domain restrict `@frota162.com`).

Não é vigilância. Não é time-tracking. É **presença social digital** + **onboarding humanizado**.

---

## 2. Motivação

### Problema (mapa)
- Time 100% remoto → sem "passar na mesa do fulano".
- Saber em que projeto alguém está exige perguntar no Slack ou caçar no Jira/GitLab.
- Pings interrompem deep work — falta visibilidade do contexto da pessoa.
- "Cadê o fulano?" virou expressão diária.

### Problema (onboarding)
- Onboarding atual: PDF + reunião + sortes de buddy → frio, esquecível, sem rastro.
- Novos colabs não sabem quem faz o quê → toma 2-3 semanas pra mapear o time.
- Conhecimento institucional vive em DMs dispersos → perde quando alguém sai.
- People ops não tem dados de onde colabs travam ou desistem.

### Por que agora
- Boilerplate NestJS + TypeORM maduro (`dashboard-devs-tool/backend`).
- Google Workspace já é a auth padrão da empresa.
- Slack já é a fonte primária de comunicação assíncrona.
- GitLab/Jira webhooks já existem em outras integrações.
- Time cresceu 60% em 12 meses → onboarding manual não escala mais.

### Não-objetivos
- ❌ Vigilância / produtividade tracking (NÃO é Hubstaff).
- ❌ Time-tracking automático (NÃO substitui Toggl/Clockify).
- ❌ Status forçado (NÃO obriga ninguém a aparecer).
- ❌ Substituir Slack ou Confluence (complementa).
- ❌ Treinar habilidades técnicas (onboarding é socialização + acessos, NÃO bootcamp).

---

## 3. Cenários de uso

### 3.1 Mapa do time

| Persona | Cenário | O que vê |
|---|---|---|
| Dev | "Quem mexe em billing?" | Lista quem commitou em `billing/*` últimos 30d |
| PM | "Pair agora com quem tá livre?" | Devs `available`, fora de foco profundo |
| Onboarding | "Quem é o João?" | Bio, projetos atuais, stack, fuso, Slack status |
| Tech Lead | "Estado do time hoje" | Heatmap: focando / em reunião / off |
| Qualquer um | "Cadê fulano? Posso chamar?" | Status + projeto + Slack DM/call em 1 clique |

### 3.2 Onboarding

| Persona | Cenário | O que faz |
|---|---|---|
| Novo colab (dia 1) | Acaba de logar | Cai no manual zerado + tour 30s |
| Novo colab (dia 5) | Em progresso | 7/12 etapas, anotações pessoais, conversa com buddy nos comments |
| Buddy | Designado pra alguém | Recebe notificação Slack quando colab abre, comenta nas margens |
| EM | Acompanha onboarding | Vê % conclusão, recebe alerta se colab parou há 3 dias |
| People ops | Itera template | Edita template-base, vê stats (tempo médio, satisfação), versiona |

---

## 4. Proposta — UX (mapa)

### 4.1 Tela principal `/presence`

3 seções: trabalhando agora · em reunião · off. Cada pessoa = card com avatar, status pill, atividade atual (ex: `MR !142 · presence map`), projeto, fonte do sinal, last-active.

**Slack badge inline no card** (dot + emoji + custom status). Botões `💬 DM` e `📞 chamar` direto no card (com fallback web se app não instalado).

Filtros: por status · por especialidade (backend/frontend/qa/devops) · por projeto · por fuso.

### 4.2 Detail panel (click no card)

Bio · TZ · working hours · stack · especialidades · projetos ativos · **seção Slack dedicada** (presence + custom status + DM/call buttons + warning contextual se DND).

**Wireframe:** [`docs/poc/presence-wireframe.html`](../poc/presence-wireframe.html)

### 4.3 Settings

Página `/presence/settings`:

- **Pause global** (1h/4h/24h/indefinido) no topo
- **Conexões opt-in** por fonte (GitLab, Jira, Slack, Calendar, IDE)
- **Visibilidade** (radio): contexto rico · apenas status · invisível
- **Granular toggles**: expor TZ, working hours, timestamp exato, aparecer em busca
- **Perfil**: bio (280 chars), role, TZ, stack, especialidades, working hours
- **Dados**: retention 30d explícita, export JSON
- **Danger zone**: apagar histórico · sair de vez

**Wireframe:** [`docs/poc/presence-settings.html`](../poc/presence-settings.html)

---

## 5. Proposta — UX (onboarding)

### 5.1 Fluxo

```
Google SSO (@frota162.com domain restrict)
    ↓
manual zerado (dia 1) + tour 30s + gating progressivo
    ↓ (5 dias evoluindo)
manual em uso (anotações, tarefas, comments do buddy)
    ↓ (30 dias)
formatura 🎓 → vira "veterano", pode ser buddy de outros
```

### 5.2 Manual pessoal — 3 colunas

**Left (TOC):**
- Progresso (X/12 + barra + ETA)
- Seções agrupadas por fase: primeira semana · primeiro mês · contínuo
- Check ✓ em completas, 🔒 em gated

**Center (conteúdo):**
- Banner "esse manual é seu" + link "📄 abrir no Drive"
- Live collaborators avatars com dot verde "editando agora"
- 7 tipos de bloco (§5.3)

**Right (activity):**
- Tabs: contribuições · histórico · menções
- **Top contribuidores** com contagem (28 edits · 14 comments...)
- Atividade recente (timeline com ícone por tipo)
- Drive sync status
- Acesso granular (owner, EM, buddy, time channel)

### 5.3 Tipos de bloco

| Tipo | Borda | Função |
|---|---|---|
| 📝 **texto** | — | conteúdo descritivo (markdown) |
| ✏️ **editável** | roxa | área do colab anotar — placeholder + hint chips |
| ✅ **task** | verde | checkbox + ação tipada (GitLab/Slack/Calendar/copy/link) |
| 🎯 **quiz** | azul | pergunta + opções + correta + feedback |
| 🎬 **embed** | laranja | vídeo (YouTube/Loom/Drive) · form · doc · image · link |
| 💬 **comments** | amarela | thread inline, @mentions, Slack notify |
| 🔌 **connections** | verde | card pra conectar GitLab/Slack/Calendar inline |
| ➖ **divider** | — | separador visual |

### 5.4 Gating progressivo

- Dia 1: seções 1-3 visíveis, 4-5 com 🔒, 6-12 colapsadas
- Conforme avança: desbloqueia sequencialmente
- **Por quê:** evita ansiedade de "12 etapas hoje". 5h totais quebrados em ~1h/dia.

### 5.5 Empty state (dia 1)

- Hero card: "comece pela seção 1 — 10 minutinhos"
- Tour 30s (3 steps) auto-abre, pular fácil
- Hint chips nos editáveis: 🎯 minha meta · 🧠 background · ☕ hobbies · ❓ pergunta
- Empty comments com invitation
- Avatars opacos pra buddy/EM offline

**Wireframes:**
- [`docs/poc/onboarding-login.html`](../poc/onboarding-login.html)
- [`docs/poc/onboarding-empty.html`](../poc/onboarding-empty.html)
- [`docs/poc/onboarding-manual.html`](../poc/onboarding-manual.html)

### 5.6 Template editor (people ops)

- **Sections list** drag-drop, agrupadas por fase, ★ obrigatórias
- **Editor central** com toolbar lateral por bloco
- **Add block panel** — grid 8 tipos
- **Inspector right** com 4 tabs:
  - **seção**: toggles + fase + tempo + roles
  - **stats inline** dessa seção (90d): N viram, % conclusão, tempo médio, satisfação, **alertas inteligentes**
  - **versões**: histórico com "N manuais herdaram"
- **Versionamento**: cada save = draft → publicar = nova versão. Rollback disponível.

**Wireframe:** [`docs/poc/onboarding-template-editor.html`](../poc/onboarding-template-editor.html)

### 5.7 Role-based scoping

Mesma seção 5 ("a stack") com conteúdo diferente por role:
- Backend → NestJS + TypeORM + DDD + Kafka
- Frontend → React + design system + Storybook
- QA → Cypress + Playwright + ambientes
- DevOps → k8s + Terraform + ArgoCD

Template editor tag roles por seção. Manual herda só aplicáveis.

---

## 6. Proposta — Sinais (mapa)

Coleta **passiva e opt-in** de múltiplas fontes.

| Sinal | Fonte | Detecta | Custo impl |
|---|---|---|---|
| **Atividade Git** | GitLab webhook | Projeto atual, último ato | Baixo |
| **Issue ativa** | Jira webhook | Card "in progress" | Baixo |
| **Slack presence** | Slack API + Events | active/dnd/away + custom status | Médio |
| **Calendário** | Google Calendar API | Em reunião / livre | Médio |
| **IDE plugin** | VSCode extension (opcional) | File atual, idle, foco | Alto |
| **Status manual** | UI button | Override explícito | Trivial |

### 6.1 Precedência

```
manual_override > calendar_busy > slack_dnd > ide_focus > git_active > slack_active > offline
```

> **Mudança v1→v2:** Slack subiu. `slack_dnd` ganhou peso quase máximo (intent explícita).

---

## 7. Integração Slack (1ª classe)

> Promovido de "sinal" para integração dedicada na v2.

### 7.1 Funcionalidades

- **Presence sync** via Slack Events API (`presence_change`)
- **Custom status** (emoji + texto + expires_at)
- **DND state** → presence `focus` automático
- **Deeplinks**:
  - DM: `slack://user?team={TEAM_ID}&id={USER_ID}`
  - Call: `slack://call?team={TEAM_ID}&id={USER_ID}`
  - Fallback web: `https://app.slack.com/client/{TEAM_ID}/{USER_ID}`
- **Notifications** (Slack Bot):
  - Buddy: "📖 Aline abriu o manual de onboarding agora"
  - EM: "⚠️ Aline parou de progredir há 3 dias"
  - Colab: "🎉 João comentou na sua seção 'a stack'"
  - Time channel: "🎓 Aline formou-se no onboarding hoje!"

### 7.2 Strategy de deeplink

```javascript
function openSlack(userId, action) {
  const deep = `slack://${action}?team=${TEAM_ID}&id=${userId}`;
  const web  = `https://app.slack.com/client/${TEAM_ID}/${userId}`;
  let opened = false;
  window.addEventListener("blur", () => opened = true, { once: true });
  window.location.href = deep;
  setTimeout(() => { if (!opened) window.open(web, "_blank"); }, 800);
}
```

### 7.3 OAuth — escopos mínimos

- `users:read` — listar membros + identidade
- `users:read.email` — mapear pra Google Workspace
- `users.profile:read` — custom status + emoji
- `dnd:read` — Do Not Disturb state
- `chat:write` — enviar notifications via bot

**Não pediremos:** `channels:read`, `groups:read`, `im:read`, `mpim:read` — privacidade.

---

## 8. Integração Google Workspace

### 8.1 Auth (SSO)

- Login: Google OAuth 2.0 (OIDC)
- Domain restrict: apenas `@frota162.com`
- Erro UX claro se domínio não permitido

### 8.2 Drive integration (manuais)

> **Decisão arquitetural:** manual de onboarding **ancora num Google Doc real**. UI presence é wrapper.

**Por quê:**
- Edição colaborativa nativa (cursors, suggestions, version history)
- Familiaridade — todo mundo já usa Docs
- Backup automático no Workspace
- Permissões aproveitam Drive ACL
- Exportação trivial (Docs → PDF)

**Como funciona:**

```
manual criado → cria Google Doc em Drive/Onboarding/{user}-{date}.gdoc
              → ACL: owner=user, editors=[buddy, EM], viewers=[#team-channel]
              → blocos do template viram seções do Doc
              → comments do Docs aparecem na sidebar do presence
              → edits do Docs sincronizam pro presence em real-time
```

**Trade-offs aceitos:**
- Latência de sync ~5s (eventual consistency)
- Google Doc é source of truth pra texto, presence pra blocos custom
- Blocos interativos (quiz, task) vivem só no presence (anotação no Doc apontando)

### 8.3 Outros serviços (Fase 3+)

- **Calendar API** — eventos busy → status `meeting`
- **People API** — auto-popular bio com Google Profile
- **Chat API** (opcional, futuro) — notifications via Google Chat

---

## 9. Arquitetura técnica

### 9.1 Stack

- **Backend:** NestJS + TypeORM (boilerplate `dashboard-devs-tool/backend`)
- **Frontend:** React SPA (padrão `frota162-backoffice`)
- **DB:** PostgreSQL
- **Cache:** Redis (real-time, TTL 5min)
- **Queue:** Bull (Redis-backed) para webhooks e Drive sync
- **Real-time:** NestJS WebSocket Gateway
- **Auth:** Google OIDC + JWT internos
- **External:** GitLab API, Jira API, Slack API + Events, Google Drive/Calendar/People API

### 9.2 Módulos

```
src/
├── _shared/
│   ├── auth/                                 # Google OIDC + JWT
│   └── google-workspace/                     # cliente Drive/Calendar/People
│
├── presence/                                 # MÓDULO 1: MAPA
│   ├── domain/
│   │   ├── presence-status.entity.ts
│   │   ├── presence-signal.entity.ts
│   │   └── user-profile.entity.ts
│   ├── application/
│   │   ├── compute-status.use-case.ts        # precedência §6.1
│   │   ├── ingest-signal.use-case.ts
│   │   ├── get-team-snapshot.use-case.ts
│   │   ├── pause-presence.use-case.ts
│   │   └── update-settings.use-case.ts
│   ├── infrastructure/
│   │   ├── gitlab-webhook.controller.ts
│   │   ├── jira-webhook.controller.ts
│   │   ├── slack-events.controller.ts
│   │   ├── presence.gateway.ts               # WebSocket
│   │   └── presence.repository.ts
│   └── presentation/
│       └── presence.controller.ts
│
├── onboarding/                               # MÓDULO 2: ONBOARDING
│   ├── domain/
│   │   ├── manual.entity.ts                  # instance por colab
│   │   ├── template.entity.ts                # versioned
│   │   ├── block.entity.ts                   # 8 tipos
│   │   ├── contribution.entity.ts
│   │   └── completion.entity.ts
│   ├── application/
│   │   ├── create-manual-from-template.use-case.ts
│   │   ├── publish-template-version.use-case.ts
│   │   ├── record-contribution.use-case.ts
│   │   ├── unlock-section.use-case.ts        # gating
│   │   ├── sync-drive-doc.use-case.ts
│   │   └── compute-onboarding-stats.use-case.ts
│   ├── infrastructure/
│   │   ├── drive-sync.service.ts
│   │   ├── slack-notify.service.ts
│   │   ├── onboarding.gateway.ts             # live collabs
│   │   └── onboarding.repository.ts
│   └── presentation/
│       ├── manual.controller.ts
│       └── template-admin.controller.ts
│
└── shared-integrations/                      # MÓDULO 3: INTEGRAÇÕES
    ├── slack/
    │   ├── slack-api.service.ts
    │   ├── slack-bot.service.ts
    │   └── slack-oauth.controller.ts
    ├── gitlab/
    └── jira/
```

### 9.3 Modelo de dados

```typescript
// presence

user_profile {
  id: UUID
  google_user_id: string                       // sub do OIDC
  email: string                                 // @frota162.com
  bio: string (max 280)
  stack: string[]
  specialties: string[]
  timezone: string
  working_hours: JSON
  role: enum (backend|frontend|qa|devops|design|product|em|data|mobile)
  avatar_url: string
  slack_user_id: string?
  slack_team_id: string?
  visibility: enum (rich|status-only|invisible)
  settings: JSON                                // opt-in flags
  created_at, updated_at
}

presence_signal {                               // append-only, partitioned by day, retention 30d
  id, user_id, source, signal_type
  payload: JSON
  project_ref: string?
  occurred_at: timestamp
}

presence_status {                               // 1 por user
  user_id (PK)
  status: enum (available|focus|meeting|off)
  slack_presence: enum (active|dnd|away)?
  slack_emoji: string?
  slack_text: string?
  current_project: string?
  current_activity: string?
  source: enum
  expires_at: timestamp                         // TTL
  updated_at
}

// onboarding

template {
  id, name, role_target: string[]
  current_version_id: UUID
  is_default_for_role: boolean
  created_at, updated_at
}

template_version {
  id, template_id, version: string (semver)
  sections: JSON                                // estrutura completa
  published: boolean
  published_by: UUID
  published_at: timestamp
  draft_of: UUID?
  rollback_of: UUID?
}

manual {
  id, user_id, template_version_id
  drive_doc_id: string                          // Google Doc
  drive_doc_url: string
  state: enum (draft|in_progress|completed|paused)
  started_at, completed_at?
  buddy_id: UUID?
  em_id: UUID?
  acl: JSON
}

manual_section_state {
  manual_id, section_id (composite PK)
  locked: boolean
  completed_at: timestamp?
  edits_count: int
  comments_count: int
}

contribution {                                  // append-only
  id, manual_id, section_id, block_id?
  author_id, type: enum (comment|edit|task_complete|quiz_answer|reaction)
  payload: JSON
  occurred_at
}
```

### 9.4 Fluxos críticos

**Novo colab cria manual**
```
Google SSO → /onboarding/bootstrap
  → resolve role (People API ou campo manual)
  → busca template default da role
  → cria Manual + ManualSectionState (locked exceto §1-3)
  → cria Google Doc via Drive API a partir do template
  → ACL: owner=colab, editors=[buddy, EM]
  → Slack bot notifica buddy: "📖 {nome} acabou de chegar"
  → redirect → /onboarding (empty state com tour)
```

**Webhook GitLab → presence**
```
GitLab push → POST /webhooks/gitlab
  → valida HMAC signature
  → enfileira no Bull
  → worker: IngestSignalUseCase
     → grava presence_signal
     → ComputeStatusUseCase (§6.1)
     → upsert presence_status (TTL 30min)
     → emit WS 'presence:update' → clientes atualizam
```

**Comment no manual**
```
User comenta → POST /onboarding/manuals/:id/contributions
  → grava contribution
  → @mention → SlackNotify (DM)
  → emit WS 'manual:contribution' → outros editores veem
  → cria comment no Google Doc (sync bidirecional)
  → atualiza counter "top contribuidores"
```

### 9.5 APIs

```
# Auth
POST   /auth/google                            # OIDC callback
GET    /auth/me

# Presence
GET    /presence/team                          # snapshot
GET    /presence/user/:id
PATCH  /presence/me                            # override manual
POST   /presence/me/pause                      # pause N minutos
POST   /presence/profile
GET    /presence/settings
PATCH  /presence/settings
WS     /presence/live                          # real-time

# Onboarding (colab)
GET    /onboarding/me
GET    /onboarding/manuals/:id
PATCH  /onboarding/manuals/:id/sections/:secId/blocks/:blockId
POST   /onboarding/manuals/:id/contributions
GET    /onboarding/manuals/:id/contributions
WS     /onboarding/manuals/:id/live            # live collabs

# Onboarding (admin)
GET    /admin/onboarding/templates
GET    /admin/onboarding/templates/:id/versions
POST   /admin/onboarding/templates/:id/versions
PATCH  /admin/onboarding/templates/:id/versions/:vId
POST   /admin/onboarding/templates/:id/versions/:vId/publish
GET    /admin/onboarding/templates/:id/versions/:vId/stats
GET    /admin/onboarding/dashboard

# Webhooks
POST   /webhooks/gitlab
POST   /webhooks/jira
POST   /webhooks/slack/events
POST   /webhooks/google/drive
```

---

## 10. Privacidade & Trust (CRÍTICO)

> Sem isso bem-feito, vira **vigilância** e o time rejeita.

### 10.1 Princípios não-negociáveis

1. **Opt-in por fonte.** Cada sinal desligável individualmente.
2. **Sem histórico exposto.** UI mostra "agora" + agregados 30d. NUNCA timeline minuto-a-minuto pra outros.
3. **Self-view completa.** Usuário vê TUDO sobre si.
4. **Cargo ≠ acesso.** Tech lead e EM **NÃO** veem dados extras. Hierarquia não espia.
5. **Pause global** (1h/4h/24h/indefinido).
6. **Sem export bulk.** API não tem endpoint de download massivo.
7. **Retention 30 dias** para sinais brutos. Só agregados sobrevivem.

### 10.2 Específico do onboarding

- Manual é do colab. EM e buddy têm read/comment **por padrão**, revogável.
- Stats no template editor são **anônimas por seção** — nunca "Aline travou na seção 5".
- People ops NÃO vê conteúdo dos blocos editáveis pessoais — apenas progresso.

---

## 11. Roadmap

### Fase 1 — MVP Mapa (2 sprints)
- [ ] `user_profile` + Google SSO + CRUD perfil
- [ ] `presence_signal` ingestão via GitLab webhook
- [ ] `presence_status` compute simplificado (git_active + manual)
- [ ] REST `/presence/team` + UI grid básica
- [ ] Settings page (opt-in por fonte)

### Fase 2 — Slack & Real-time (2 sprints)
- [ ] **Slack OAuth + Events API** (presence + custom status)
- [ ] **Deeplinks DM/call** com fallback web
- [ ] WebSocket gateway → updates ao vivo
- [ ] Jira webhook
- [ ] Slack bot pra notifications

### Fase 3 — Onboarding MVP (3 sprints)
- [ ] `template` + `manual` entities
- [ ] **Drive integration** — criar Google Doc por manual
- [ ] Template default `backend-engineer` v1.0 (hardcoded)
- [ ] Manual UI (3 colunas) + 4 tipos de bloco (texto, editável, task, comments)
- [ ] Gating progressivo
- [ ] Slack notifications (buddy/EM)
- [ ] Empty state + tour

### Fase 4 — Template Editor & Quizzes (2 sprints)
- [ ] Template editor admin UI
- [ ] Template versionamento (draft → publish → rollback)
- [ ] Blocos restantes: quiz, embed, connections, divider
- [ ] Stats por seção
- [ ] Role-based scoping

### Fase 5 — Sinais ricos & inteligência (futuro)
- [ ] Calendar integration (busy → meeting)
- [ ] VSCode extension
- [ ] Top contribuidores cross-manual
- [ ] Heatmap timezone
- [ ] "Pessoas similares" (mesma stack)
- [ ] Sugestão de mentor por especialidade
- [ ] People dashboard com alertas

---

## 12. Riscos & mitigações

| Risco | Severidade | Mitigação |
|---|---|---|
| Vira vigilância | 🔴 Alta | §10 é gate de lançamento. Privacy review com 5 voluntários antes da Fase 1 GA. |
| Onboarding overwhelming | 🟡 Média | Gating §5.4. Hint chips quebram inércia. |
| Drive sync inconsistente | 🟡 Média | Eventual consistency 5s SLA. Google Doc source of truth pra texto. |
| Ninguém atualiza bio | 🟢 Baixa | Auto-popular do Google People API no onboarding. |
| Template editor abandonado | 🟡 Média | Stats forçam ownership. Alerta "tempo > estimado" gera review. |
| Sinais Slack falsos | 🟢 Baixa | Slack expira custom status. Sincronizar `expires_at`. |
| Webhooks rate limit | 🟢 Baixa | Bull + retry exponencial. |
| Adoção baixa | 🟡 Média | Lançar com 5 voluntários (Fase 1) + onboarding novo da Frota usa Fase 3 (adoção forçada por uso). |

---

## 13. Decisões abertas → fechadas (v2)

### ~~D1: WebSocket vs SSE?~~ → **WebSocket**
**Razão:** onboarding precisa bidirecional (live collabs, cursor sync). Mapa poderia ser SSE, mas usar 1 stack simplifica deploy.

### ~~D2: IDE plugin agora ou Fase 3?~~ → **Fase 5 (futuro)**
**Razão:** alto custo de manutenção (VSCode + IntelliJ + Cursor + ...). Slack DND + Calendar cobre 80% do "em foco" com 10% do esforço.

### ~~D3: Mostrar "última vez online" pros outros?~~ → **Não. Apenas aproximado.**
**Razão:** timestamp exato pesa pra ansiedade ("ela não respondeu em 12 min"). UI mostra "há ~poucos minutos / horas / dias". Toggle nas settings (default off) permite expor exato.

### ~~D4: Integra com dashboard-devs-tool ou standalone?~~ → **Standalone, mesmo monorepo**
**Razão:** `user-organization-frota` é o lugar. Compartilha boilerplate, mas app separado. Escala e release independentes.

### ~~D5: Owner/stakeholder?~~ → **People Ops + EM Engineering**
- **Owner produto**: People Ops (Rafael Souza ou substituto)
- **Owner técnico**: EM Engineering (a definir)
- **Buddy program**: People Ops decide alocação
- **Template content**: People Ops + tech leads por role

---

## 14. Decisões NOVAS abertas (v2)

- [ ] **D6:** Manual pode ser auditado por compliance (LGPD)? Anotações pessoais do colab são "dados pessoais sensíveis"?
- [ ] **D7:** "Formatura" 🎓 (manual completo) gera certificado? Vira público no perfil?
- [ ] **D8:** Buddy program é opt-in pra veteranos ou allocação manual da People Ops? Existe "stake" (XP, gamificação) pra ser buddy?
- [ ] **D9:** Quando alguém sai da empresa, manual vira read-only ou é deletado? Comments dele em outros manuais ficam?
- [ ] **D10:** Template editor permite "fork" entre roles (clonar backend → editar pra mobile) ou só copiar seção a seção?
- [ ] **D11:** Notificações Slack são opt-out pelo colab? Ele pode silenciar buddy/EM no presence?
- [ ] **D12:** Mobile? PWA ou app nativo? (mapa é mais útil no desktop, mas check-in mobile faz sentido)

---

## 15. Métricas de sucesso

### 15.1 Mapa
- **Adoção:** % de colabs com perfil + ≥1 fonte conectada (Fase 1: 60%/30d; Fase 2: 85%/30d)
- **Engajamento:** sessions/semana por colab (target ≥3)
- **Slack actions:** DMs/calls via deeplink (target ≥1/dia/colab ativo)
- **Pause global:** % uso ≥1x/mês (saudável: 20-40%; >60% = problema cultural)

### 15.2 Onboarding
- **Conclusão:** % manuais completos em 30d (target 80%)
- **Tempo médio:** dias até "formatura" (target 21-30)
- **Engajamento buddy:** % manuais com ≥3 comments buddy (target 90%)
- **Satisfação:** NPS do onboarding (target ≥40)
- **Iteração template:** versões/trimestre (saudável 2-4)

---

## 16. Alternativas consideradas

| Alternativa | Por que não |
|---|---|
| **Slack custom status** apenas | Não agrega cross-fonte. Ninguém atualiza. Sem onboarding. |
| **Geekbot/standup bot** | Assíncrono pontual, não "agora". Sem onboarding. |
| **Tandem/Pragli (SaaS)** | Caro. Dados fora. Sem Jira/GitLab interno. Sem onboarding. |
| **Confluence/Notion** pra onboarding | Sem interatividade (quiz/connections/gating). Estático. |
| **HRIS (Gupy, Sólides)** | Foco em RH (folha, avaliações), não onboarding técnico vivo. |
| **GitLab Handbook clone** | Wiki público ≠ manual pessoal. Pode coexistir, não substitui. |

**Build > buy** porque: integração profunda (NestJS + Workspace + Slack), controle de privacidade (sem exportar pra SaaS), custo marginal baixo dado boilerplate, dupla função (mapa + onboarding) em uma plataforma.

---

## 17. Next steps

1. **Privacy review** com 5 voluntários (foco §10) — 1 semana
2. **Spike técnico**: Drive API + Slack Events API (2 dias) — validar latência e custos
3. **Validar D6-D12** com People Ops + Legal (LGPD) — 1 semana
4. **POC funcional Fase 1** com 5 voluntários (1 sprint dedicada)
5. **GitLab issues** por fase via `/gh-issue-craft`
6. **Catalog-info.yaml** para registrar no IDP

---

## 18. Wireframes (POC HTML)

| Tela | Arquivo | Status |
|---|---|---|
| Mapa do time | [`docs/poc/presence-wireframe.html`](../poc/presence-wireframe.html) | ✅ |
| Settings (privacy) | [`docs/poc/presence-settings.html`](../poc/presence-settings.html) | ✅ |
| Onboarding login (SSO) | [`docs/poc/onboarding-login.html`](../poc/onboarding-login.html) | ✅ |
| Onboarding dia 1 (empty + tour) | [`docs/poc/onboarding-empty.html`](../poc/onboarding-empty.html) | ✅ |
| Onboarding dia 5 (em uso) | [`docs/poc/onboarding-manual.html`](../poc/onboarding-manual.html) | ✅ |
| Template editor (admin) | [`docs/poc/onboarding-template-editor.html`](../poc/onboarding-template-editor.html) | ✅ |
| People dashboard | _pendente_ | ⏳ |
| Heatmap timezone | _pendente_ | ⏳ |

---

## 19. Referências

- Valve Employee Handbook (2012), p.43
- [GitLab Handbook](https://about.gitlab.com/handbook/)
- [Notion onboarding templates](https://www.notion.so/templates/category/onboarding)
- [Tandem.chat](https://tandem.chat)
- [Slack Events API](https://api.slack.com/events)
- [Slack deeplinks](https://api.slack.com/reference/deep-linking)
- [Google Drive API v3](https://developers.google.com/drive/api/v3/reference)
- [Google OIDC](https://developers.google.com/identity/openid-connect/openid-connect)
- [GitLab webhook events](https://docs.gitlab.com/ee/user/project/integrations/webhook_events.html)
- [LGPD](http://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm)
