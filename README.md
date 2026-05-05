# Splitcard

Tarjeta compartida para gastos del hogar. Divide boletas, compras físicas y cuentas en restaurantes entre los miembros de un grupo.

## Estado actual

El proyecto vive en **dos modos**:

| Modo | Cómo se levanta | Estado |
|---|---|---|
| **Demo** | `cd frontend && npm run dev` | ✅ Producción funcional. Mock state in-memory. Lo que se usa hoy. |
| **Producción** | `npm run dev` (raíz) | ⚠️ En cableado. BFF + Core listos, frontend modular esqueleto. |

> El archivo `frontend/src/splitcard-v2.jsx` (~3.700 LOC) es el monolito actual. La meta es migrarlo a `frontend/src/features/*` y conectarlo al backend.

## Arquitectura objetivo

```
┌─────────────┐    HTTPS    ┌──────────┐   internal    ┌──────────────┐
│  Frontend   │ ──────────▶ │   BFF    │ ────────────▶ │ Core Service │
│ (Vite+React)│             │(Fastify) │               │  (Fastify)   │
└─────────────┘             └──────────┘               └──────┬───────┘
       │                          │                            │
       │ Clerk JWT                │ X-Internal-Secret          │
       │                          ▼                            ▼
       │                    Rate-limit                  ┌──────────────┐
       │                                                 │ PostgreSQL +  │
       └─ /api/v1/* ────────────────────────────────────▶│ Redis (cache) │
                                                         └──────────────┘
```

## Stack

- **Frontend**: React 19, Vite 8, TailwindCSS 4, React Router 7, React Query 5, Clerk (auth)
- **BFF**: Fastify 5 · puerto `3000` · proxy + auth + rate-limit
- **Core**: Fastify 5 · puerto `3001` · Postgres + Redis
- **Infra dev**: docker-compose (Postgres `5433`, Redis `6380`)

## Setup local

```bash
# 1. Variables
cp .env.example .env

# 2. Levantar infra (Postgres + Redis)
npm run dev:infra

# 3. Migraciones + seeds
npm run dev:setup

# 4. Levantar todo (Core + BFF + Frontend)
npm run dev
```

## Modo demo (lo más rápido)

Si solo quieres ver la UI sin levantar el backend:

```bash
cd frontend
npm install
npm run dev
# → http://localhost:5173
```

El frontend en demo mode carga `frontend/src/splitcard-v2.jsx` con mock data hardcoded. Útil para iterar UI sin backend.

## Estructura

```
splicard-main/
├── frontend/                 # Frontend modular (target)
│   └── src/
│       ├── splitcard-v2.jsx  # Monolito demo (legacy, en migración)
│       ├── features/         # Pages por feature (home, services, etc.)
│       ├── lib/api.js        # Cliente HTTP con Clerk token
│       └── main-demo.jsx     # Entry demo (carga splitcard-v2.jsx)
├── bff/                      # Backend-for-frontend
│   └── src/routes/           # Endpoints /api/v1/*
├── core-service/             # Servicio core
│   └── src/modules/          # billing, groups, payments, users, etc.
├── docs/superpowers/specs/   # Specs de arquitectura
└── scripts/legacy/           # Scripts ad-hoc archivados
```

## Tests

- **Core**: `cd core-service && npm test` (8 suites: split.calculator, bill.service, payment.service, encryption, etc.)
- **BFF**: pendiente
- **Frontend**: pendiente

## Roadmap

- **Sprint 1** — Cableado: extraer mocks, conectar primer endpoint, mantener demo como fallback
- **Sprint 2** — Khipu real: gateway + webhook + cron de fetch de boletas día 1
- **Sprint 3** — Pagos: MercadoPago para cobros Splitcard Go, persistir sessions en backend
