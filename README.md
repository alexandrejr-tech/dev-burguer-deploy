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
