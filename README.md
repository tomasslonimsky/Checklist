# StyleChecks

Checklists colaborativas con columnas custom, realtime y push notifications.

**Stack:** Next.js 14 (App Router) · Supabase (Postgres + Auth + Realtime) · Web Push nativo · Vercel.

---

## Setup local

### 1. Clonar e instalar

```bash
git clone <tu-repo>
cd stylechecks
npm install
```

### 2. Crear proyecto en Supabase

- Entrá a https://supabase.com y creá un proyecto nuevo (free tier alcanza).
- Anotá: **Project URL** y **anon public key** (en Project Settings → API).
- Copiá también el **service_role** key (secreto, nunca lo pongas en código cliente).
- Andá al **SQL Editor** y pegá todo el contenido de `supabase/migrations/0001_init.sql`. Dale Run.

### 3. Configurar auth en Supabase

En **Authentication → URL Configuration**:
- Site URL: `http://localhost:3000` (después cambialo a tu dominio de Vercel)
- Redirect URLs: agregá `http://localhost:3000/auth/callback` y el de producción

En **Authentication → Email templates** podés customizar el template del magic link (opcional).

### 4. Generar VAPID keys para push

```bash
npm run generate-vapid
```

Copiá `NEXT_PUBLIC_VAPID_PUBLIC_KEY` y `VAPID_PRIVATE_KEY` al `.env.local`.

### 5. Generar CRON_SECRET

```bash
openssl rand -hex 32
```

### 6. Crear `.env.local`

Copiá `.env.example` a `.env.local` y completá:

```
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGci...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGci...
NEXT_PUBLIC_VAPID_PUBLIC_KEY=...
VAPID_PRIVATE_KEY=...
VAPID_SUBJECT=mailto:tu@email.com
CRON_SECRET=...
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

### 7. Correr

```bash
npm run dev
```

Abrí http://localhost:3000 y entrá con magic link.

---

## Deploy en Vercel

1. Subí el repo a GitHub.
2. En https://vercel.com, "Add New Project" → importás el repo.
3. En **Environment Variables**, pegá todas las del `.env.local`. Cambiá `NEXT_PUBLIC_SITE_URL` a tu dominio de Vercel (ej: `https://stylechecks.vercel.app`).
4. En Supabase → Authentication → URL Configuration, agregá tu dominio de Vercel al Site URL y a los Redirect URLs.
5. Deploy. El cron de recordatorios diarios se activa automáticamente (ver `vercel.json`).

### Push notifications

- **Desktop (Chrome/Edge/Firefox):** funciona de una, tocá "Activar notificaciones" en el panel de notis.
- **Android:** igual que desktop.
- **iOS/iPadOS:** requiere iOS 16.4+ y que la app esté instalada como PWA desde Safari (Compartir → Agregar a pantalla de inicio). Solo ahí aparece el permiso de notis.

### Cron (recordatorios diarios)

El endpoint `/api/cron/daily-reminders` corre cada hora en Vercel. Revisa a todos los usuarios y manda push solo a quienes tengan ese horario configurado en su timezone. Gratis en el hobby tier.

---

## Estructura

```
src/
  app/
    page.tsx                    # home: redirige a primera checklist u onboarding
    c/[id]/page.tsx             # vista de una checklist
    login/page.tsx              # magic link login
    auth/callback/route.ts      # callback OAuth
    settings/page.tsx           # ajustes de notificaciones
    api/
      push/subscribe/route.ts   # guardar/borrar push sub
      push/send/route.ts        # enviar push (self test + service)
      cron/daily-reminders/     # cron horario → recordatorios diarios
  components/
    Sidebar.tsx                 # lista de checklists
    ChecklistBoard.tsx          # tabla principal con realtime
    TaskRow.tsx                 # fila editable
    ShareModal.tsx              # invitar miembros
    ColumnsConfig.tsx           # editar columnas
    NotificationsPanel.tsx      # activar push + prefs
  lib/
    supabase-browser.ts
    supabase-server.ts
    push-client.ts
    types.ts
  middleware.ts                 # auth refresh + guards
supabase/
  migrations/0001_init.sql      # schema completo
public/
  sw.js                         # service worker (push)
  manifest.json                 # PWA manifest
  icons/                        # íconos 192 y 512
scripts/
  generate-vapid.mjs
vercel.json                     # cron config
```

---

## Roadmap corto

- [ ] Drag & drop para reordenar tareas y columnas
- [ ] Notificaciones cuando te asignan una tarea (trigger en DB)
- [ ] Undo de borrado de tarea
- [ ] Exportar checklist a CSV / Excel
- [ ] Dark mode manual (por ahora sigue prefers-color-scheme)
- [ ] Templates (sushi class, outreach, etc.)

---

Hecho con cariño. Reportá bugs en los issues del repo.
