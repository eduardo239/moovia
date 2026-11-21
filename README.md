# Moovia

Descubra filmes e séries, veja trailers, elenco e onde assistir, e organize sua própria lista de "ver depois" e "já assisti".

O Moovia é um monorepo com um frontend em React e um backend em Fastify que funciona como proxy seguro para a API do [TMDB](https://www.themoviedb.org/). Assim, a chave da API nunca fica exposta no navegador.

## Funcionalidades

- Filmes e séries populares, em alta e próximos lançamentos, com busca e filtro por gênero
- Páginas de detalhes de filmes, séries (com temporadas e episódios) e pessoas (filmografia)
- Trailers, elenco, títulos similares e provedores onde assistir
- Login com Google (Firebase Authentication)
- Listas pessoais "Ver depois" e "Já assisti", separadas por usuário e salvas no navegador
- Tema claro e escuro
- Rate limit, CORS restrito e headers de segurança (Helmet) no backend

## Stack

| Camada   | Tecnologias                                                  |
| -------- | ------------------------------------------------------------ |
| Frontend | React 19, TypeScript, Vite, React Router 7, Axios, Firebase  |
| Backend  | Node.js, Fastify 5, TypeScript, Axios                        |
| Infra    | Docker, Nginx, GitHub Actions, Google Cloud Run              |

## Estrutura

```
.
├── backend/              # API Fastify (proxy do TMDB)
│   └── src/
│       ├── routes/       # movies, tv, person, genres, search, trending
│       ├── services/     # cliente do TMDB
│       └── lib/          # validação e tratamento de erros
├── frontend/             # SPA React
│   └── src/
│       ├── pages/        # Home, Movies, TV, detalhes, Minha Lista, Busca
│       ├── components/
│       ├── context/      # Auth, Movie (listas) e Theme
│       ├── api/          # chamadas ao backend
│       └── lib/firebase.ts
├── docker-compose.yml
└── .github/workflows/    # deploy do backend e do frontend
```

## Pré-requisitos

- Node.js 20 ou superior
- [pnpm](https://pnpm.io/)
- Uma chave de API do [TMDB](https://www.themoviedb.org/settings/api)
- Um projeto no [Firebase](https://console.firebase.google.com/) com o login Google habilitado

## Como rodar localmente

1. Instale as dependências:

   ```bash
   pnpm install
   pnpm --dir backend install
   pnpm --dir frontend install
   ```

2. Configure as variáveis de ambiente:

   ```bash
   cp backend/.env.example backend/.env
   cp frontend/.env.example frontend/.env
   ```

   Preencha `TMDB_API_KEY` em `backend/.env` e as variáveis `VITE_FIREBASE_*` em `frontend/.env`.

3. Inicie backend e frontend juntos:

   ```bash
   pnpm dev
   ```

   - Frontend: http://localhost:5173
   - Backend: http://localhost:3001

## Variáveis de ambiente

**Backend** (`backend/.env`)

| Variável       | Descrição                                                                                  |
| -------------- | ------------------------------------------------------------------------------------------ |
| `TMDB_API_KEY` | Chave da API do TMDB                                                                       |
| `PORT`         | Porta do servidor (padrão `3001`)                                                          |
| `CORS_ORIGIN`  | Origens permitidas, separadas por vírgula. Obrigatória em produção; sem ela, só localhost  |

**Frontend** (`frontend/.env`)

| Variável                            | Descrição                                |
| ----------------------------------- | ---------------------------------------- |
| `VITE_API_URL`                      | URL do backend (ex.: `http://localhost:3001`) |
| `VITE_FIREBASE_API_KEY`             | Configuração do Firebase                 |
| `VITE_FIREBASE_AUTH_DOMAIN`         | Configuração do Firebase                 |
| `VITE_FIREBASE_PROJECT_ID`          | Configuração do Firebase                 |
| `VITE_FIREBASE_STORAGE_BUCKET`      | Configuração do Firebase                 |
| `VITE_FIREBASE_MESSAGING_SENDER_ID` | Configuração do Firebase                 |
| `VITE_FIREBASE_APP_ID`              | Configuração do Firebase                 |

## Docker

Com as variáveis `TMDB_API_KEY` e `VITE_FIREBASE_*` exportadas no ambiente:

```bash
docker compose up --build
```

- Frontend: http://localhost
- Backend: http://localhost:3001

As credenciais do Firebase são injetadas no build do frontend como Docker secrets.

## API

Todas as rotas ficam sob `/api` e aceitam apenas `GET`.

| Rota                                                    | Descrição                          |
| ------------------------------------------------------- | ---------------------------------- |
| `/api/movies/popular`, `/upcoming`, `/search`           | Listagens e busca de filmes        |
| `/api/movies/:id`                                       | Detalhes do filme                  |
| `/api/movies/:id/credits`, `/videos`, `/similar`, `/watch-providers` | Dados complementares  |
| `/api/tv/popular`, `/trending`                          | Listagens de séries                |
| `/api/tv/:id`                                           | Detalhes da série                  |
| `/api/tv/:id/season/:seasonNumber`                      | Episódios de uma temporada         |
| `/api/tv/:id/credits`, `/videos`, `/similar`, `/watch-providers` | Dados complementares  |
| `/api/person/:id`, `/api/person/:id/movie_credits`      | Detalhes e filmografia de pessoas  |
| `/api/genres/movie`, `/api/genres/tv`                   | Gêneros                            |
| `/api/search`                                           | Busca geral                        |
| `/api/trending`                                         | Em alta                            |

O backend limita as requisições a 60 por minuto por IP em produção (120 em desenvolvimento).

## Scripts

Na raiz:

| Comando             | Ação                                  |
| ------------------- | ------------------------------------- |
| `pnpm dev`          | Sobe backend e frontend em paralelo   |
| `pnpm dev:backend`  | Sobe só o backend                     |
| `pnpm dev:frontend` | Sobe só o frontend                    |

Em `frontend/`: `build`, `lint`, `preview`. Em `backend/`: `build`, `start`.

## Deploy

Os workflows em [.github/workflows](.github/workflows) publicam backend e frontend no Google Cloud Run a cada push na `main` que altere `backend/**` ou `frontend/**`, respectivamente. A autenticação usa Workload Identity Federation.

## Créditos

Dados e imagens fornecidos pelo [TMDB](https://www.themoviedb.org/). Este produto usa a API do TMDB, mas não é endossado nem certificado por ele.
