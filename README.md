# Product Blueprint — site público (GitHub Pages)

O **Product Blueprint** vive em `docs/index.html` deste repo (privado). Toda vez que o arquivo é atualizado e mergeado na `main`, um workflow do GitHub Actions **espelha** automaticamente o HTML para um repo público separado — que serve via GitHub Pages.

- **Fonte (privado):** `MSM-Polymers-Group/impactrakr` · arquivo `docs/index.html`
- **Mirror (público, só o HTML):** `MSM-Polymers-Group/impactrakr-blueprint`
- **URL pública:** `https://msm-polymers-group.github.io/impactrakr-blueprint/`
- **Workflow:** `.github/workflows/publish-blueprint.yml`

## Setup inicial (faça uma vez)

### 1. Criar repo público vazio

No GitHub:

- Acesse: `https://github.com/organizations/MSM-Polymers-Group/repositories/new`
- Nome: `impactrakr-blueprint`
- Visibilidade: **Public**
- Descrição: `Public mirror of the Impactrakr Product Blueprint (auto-published).`
- **NÃO** marque "Add a README", "Add .gitignore" ou license — deixe completamente vazio
- Create repository

### 2. Gerar Personal Access Token (PAT)

- Acesse: `https://github.com/settings/tokens`
- Generate new token → **classic**
- Note: `impactrakr-blueprint-deploy`
- Expiration: `No expiration` (ou 1 ano, sua escolha)
- Scopes: marque apenas `repo` (Full control of private repositories)
- Generate token
- **Copie o token** (`ghp_…`) — só aparece uma vez

### 3. Adicionar token como Secret no repo privado

- Acesse: `https://github.com/MSM-Polymers-Group/impactrakr/settings/secrets/actions`
- New repository secret
- Name: `BLUEPRINT_DEPLOY_TOKEN`
- Secret: cole o token do passo anterior
- Add secret

### 4. Disparar o primeiro publish

Você tem 2 opções:

**a) Mergear este PR** com as mudanças do `docs/index.html` — o workflow dispara sozinho.

**b) Disparar manualmente** (workflow_dispatch):
- Acesse: `https://github.com/MSM-Polymers-Group/impactrakr/actions/workflows/publish-blueprint.yml`
- Run workflow → Branch: `main` → Run

Em ~30s o workflow termina. Confira em `https://github.com/MSM-Polymers-Group/impactrakr-blueprint` que apareceu o `index.html`.

### 5. Ativar GitHub Pages no repo público

> ⚠️ **Atenção à ordem.** Só ative o Pages **depois** que o primeiro push do workflow chegar no repo público. Repo vazio não tem branch `main`, então o dropdown de Pages aparece sem opção. O fluxo natural é: passos 2 → 3 → 4 (PR mergeado → workflow dispara → primeiro push → branch `main` criada) → 5.

Após o primeiro publish do workflow:

- Acesse: `https://github.com/MSM-Polymers-Group/impactrakr-blueprint/settings/pages`
- Source: **Deploy from a branch**
- Branch: `main` · Folder: `/ (root)`
- Save

Em ~1 minuto a URL `https://msm-polymers-group.github.io/impactrakr-blueprint/` estará no ar.

## Workflow de atualização (uso recorrente)

Daqui em diante, qualquer mudança no Blueprint segue este fluxo:

```
edita docs/index.html (local ou via agent)
  ↓
commit + push em branch
  ↓
abre PR pra main
  ↓
merge na main
  ↓
GitHub Actions dispara o workflow (~30s)
  ↓
HTML aparece no repo público
  ↓
GitHub Pages rebuilda (~1min)
  ↓
URL pública atualizada
```

## Manutenção

### A scheduled task diária

Toda noite às 23h um agente automatizado:
1. Lê as memórias do projeto (dailies, decisões, riscos)
2. Roda audit do repo
3. Atualiza o HTML local se houver mudanças significativas
4. Avisa no chat o que mudou

Você decide quando mergear pra disparar o publish.

### Custom domain (opcional, futuro)

Quando o domínio `impactrakr.com` estiver ativo, dá pra apontar `blueprint.impactrakr.com` pra GitHub Pages:

1. No repo público: Settings → Pages → Custom domain → `blueprint.impactrakr.com`
2. No DNS do `impactrakr.com`: criar registro `CNAME` apontando `blueprint` para `msm-polymers-group.github.io`
3. GitHub provisiona SSL automático

### O que NÃO vai pro repo público

Só `index.html`, `.nojekyll` e (opcionalmente) este `README.md`. Tudo mais permanece no repo privado:

- Código backend (`apps/api/`)
- Schema Prisma, migrations
- Pulumi/infra, packages
- ADRs e documentação técnica (`docs/ADR-*.md`, `docs/IMP-*.md`)
- Workflows internos

### Troubleshooting

| Sintoma | Causa provável | Solução |
|---|---|---|
| Workflow falha em "Clone target" | Secret `BLUEPRINT_DEPLOY_TOKEN` faltando ou expirou | Regenerar PAT, atualizar secret |
| Workflow falha em "git push" | Token sem scope `repo` | Regenerar com scope `repo` completo |
| Pages não atualiza | Esqueceu de ativar Pages no repo público | Settings → Pages no público |
| HTML aparece quebrado | Algum tag faltando | Conferir local com `python -m http.server 8000 --directory docs/` |
| 404 ao abrir URL | Pages ainda compilando | Aguardar ~1 min e dar refresh |
