# Presence — Frota 162

Plataforma interna que combina **mapa de presença do time** (estilo Valve "user check") com **onboarding interativo personalizado** para novos colaboradores.

## 🎯 Demo (wireframes)

👉 **[Abrir demo ao vivo](https://geovaneguibes.github.io/user-organization-frota/poc/onboarding-login.html)**

Fluxo sugerido para navegação:

1. **[Login SSO](https://geovaneguibes.github.io/user-organization-frota/poc/onboarding-login.html)** — primeiro acesso via Google Workspace
2. **[Manual dia 1](https://geovaneguibes.github.io/user-organization-frota/poc/onboarding-empty.html)** — empty state com tour 30s
3. **[Manual dia 5](https://geovaneguibes.github.io/user-organization-frota/poc/onboarding-manual.html)** — em uso, com comentários do time
4. **[Mapa do time](https://geovaneguibes.github.io/user-organization-frota/poc/presence-wireframe.html)** — quem trabalha em quê agora
5. **[Settings (privacy)](https://geovaneguibes.github.io/user-organization-frota/poc/presence-settings.html)** — opt-in granular por fonte
6. **[Template editor](https://geovaneguibes.github.io/user-organization-frota/poc/onboarding-template-editor.html)** — admin/people ops
7. **[People dashboard](https://geovaneguibes.github.io/user-organization-frota/poc/people-dashboard.html)** — acompanhar onboardings
8. **[Dashboard empty state](https://geovaneguibes.github.io/user-organization-frota/poc/people-dashboard-empty.html)** — primeiro mês operando
9. **[Stats deep dive](https://geovaneguibes.github.io/user-organization-frota/poc/people-stats.html)** — analytics agregadas

## 📋 RFC

- **[RFC v2 (current)](docs/rfcs/0001-presence-map-v2.md)** — Slack 1ª classe, onboarding, Google Workspace, gating progressivo
- [RFC v1 (superseded)](docs/rfcs/0001-presence-map.md) — versão inicial

## 🏗️ Estrutura

```
docs/
├── poc/                    # 9 wireframes HTML standalone
│   ├── presence-wireframe.html
│   ├── presence-settings.html
│   ├── onboarding-login.html
│   ├── onboarding-empty.html
│   ├── onboarding-manual.html
│   ├── onboarding-template-editor.html
│   ├── people-dashboard.html
│   ├── people-dashboard-empty.html
│   └── people-stats.html
└── rfcs/
    ├── 0001-presence-map.md       # v1
    └── 0001-presence-map-v2.md    # v2 (current)
```

## ⚙️ Stack proposta (RFC §9)

- **Backend:** NestJS + TypeORM + PostgreSQL + Redis + Bull
- **Frontend:** React SPA
- **Auth:** Google Workspace SSO (OIDC) + JWT
- **Real-time:** WebSocket Gateway
- **Integrations:** Slack API + Events · GitLab webhooks · Jira webhooks · Google Drive/Calendar/People

## 🚀 Status

**Fase atual:** Wireframes + RFC v2 prontos. Aguardando privacy review com 5 voluntários antes do POC funcional.

Próximos passos (RFC §17):
1. Privacy review com 5 voluntários
2. Spike técnico: Drive API + Slack Events API
3. Validar decisões D6-D12 (LGPD, formatura, mobile)
4. POC funcional Fase 1
