---
title: Presence — Frota 162
description: Mapa do time + onboarding interativo (POC wireframes)
---

# Presence — Frota 162

Plataforma interna que combina **mapa de presença do time** (estilo Valve "user check") com **onboarding interativo personalizado** para novos colaboradores.

## 🎯 Demo (wireframes)

👉 **[Começar pelo login →](poc/onboarding-login.html)**

### Fluxo sugerido

| # | Tela | Persona |
|---|---|---|
| 1 | [Login SSO](poc/onboarding-login.html) | Novo colab |
| 2 | [Manual dia 1 (empty state)](poc/onboarding-empty.html) | Novo colab |
| 3 | [Manual dia 5 (em uso)](poc/onboarding-manual.html) | Colab + buddy + EM |
| 4 | [Mapa do time](poc/presence-wireframe.html) | Todos |
| 5 | [Settings (privacy)](poc/presence-settings.html) | Self |
| 6 | [Template editor](poc/onboarding-template-editor.html) | People Ops |
| 7 | [People dashboard](poc/people-dashboard.html) | People Ops + EM |
| 8 | [Dashboard empty state](poc/people-dashboard-empty.html) | People Ops |
| 9 | [Stats deep dive](poc/people-stats.html) | People Ops |

## 📋 RFC

- **[RFC v2 (current)](rfcs/0001-presence-map-v2.md)** — Slack 1ª classe, onboarding, Google Workspace, gating progressivo
- [RFC v1 (superseded)](rfcs/0001-presence-map.md)

## ⚙️ Stack proposta

- **Backend:** NestJS + TypeORM + PostgreSQL + Redis + Bull
- **Frontend:** React SPA
- **Auth:** Google Workspace SSO (OIDC) + JWT
- **Real-time:** WebSocket Gateway
- **Integrations:** Slack API + Events · GitLab webhooks · Jira webhooks · Google Drive/Calendar/People

## 🚀 Status

**Fase atual:** Wireframes + RFC v2 prontos. Aguardando privacy review antes do POC funcional.

---

[Source no GitHub →](https://github.com/guibes/user-organization-frota)
