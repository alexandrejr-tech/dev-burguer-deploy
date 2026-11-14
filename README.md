# dev-burguer-deploy

Orquestração do DevBurguer com Docker Compose: API ([dev-burguer-api](https://github.com/alexandrejr-tech/dev-burguer-api)), front ([dev-burguer-interface](https://github.com/alexandrejr-tech/dev-burguer-interface)), Postgres e Mongo.

## Teste local

Os três repositórios precisam estar lado a lado na mesma pasta:

```
projetos/
├── dev-burguer-api/
├── dev-burguer-interface/
└── dev-burguer-deploy/
```

```bash
cp .env.example .env   # preencha os valores
docker compose -f docker-compose.yml -f docker-compose.local.yml up -d --build --wait
```

- Front: http://localhost:8080
- API: http://localhost:3001

Parar: `docker compose -f docker-compose.yml -f docker-compose.local.yml down` (os dados ficam nos volumes; `down -v` apaga os volumes).

## Produção (Dokploy)

Use só o `docker-compose.yml`. Cadastre as variáveis do `.env.example` no Dokploy, com `API_CONTEXT` e `FRONT_CONTEXT` apontando para as URLs do Git, e configure os domínios na aba Domains (front na porta 80, API na porta 3001).

## Como cada mudança chega à produção

| O que mudou | Como entra em produção |
|---|---|
| Código da API ou do front (push na branch principal) | Automático: o GitHub Actions do repositório roda o CI e, se passar, pede um **redeploy** ao Dokploy e espera o resultado. |
| Este repositório (`docker-compose.yml`) | **Deploy normal**: botão **Deploy** do serviço no Dokploy (ou o auto-deploy deste repositório, se estiver ligado). |
| Variáveis de ambiente no Dokploy | Botão **Deploy** do serviço, depois de salvar as variáveis. |

Por que o CI usa redeploy e não deploy:

- O **deploy** baixa de novo este repositório e reconstrói tudo, mas no fim troca o título no histórico pelo último commit **deste** repositório. Ele também envia as notificações de build do Dokploy.
- O **redeploy** mantém o título enviado pelo CI (ex.: `dev-burguer-api@abc1234`) e reconstrói as imagens com `--build`, buscando o código atual da API e do front pelas URLs do Git. Ele **não** baixa de novo este repositório: uma mudança no `docker-compose.yml` só entra com um deploy normal.
- O redeploy não envia as notificações do Dokploy. Por isso o job `deploy` do GitHub Actions espera o resultado: se o build falhar, o job fica vermelho e o GitHub avisa por e-mail.

Os workflows da API e do front usam os secrets `DOKPLOY_URL`, `DOKPLOY_API_KEY` e `DOKPLOY_COMPOSE_ID`, cadastrados em cada repositório (Settings → Secrets and variables → Actions).
