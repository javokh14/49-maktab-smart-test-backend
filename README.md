# 49-Maktab Smart Test — Backend

Node.js + Express + Turso/libSQL backend. Render Web Service uchun tayyor.

## Nimalar bor

- Administrator login
- O'qituvchilar CRUD + bloklash/yoqish
- Har o'qituvchiga alohida login/parol (parol DB da hash holatda)
- A/B/C testlar
- O'quvchi testga ruxsat so'raydi
- Admin/O'qituvchi: qabul / rad / qaytadan
- Server test natijasini o'zi hisoblaydi
- O'quvchiga ball qaytarmaydi
- Natijalarni kod bo'yicha ko'rish/o'chirish
- SSE live events (`/api/events`)
- Gemini API keyni Admin panelidan saqlash
- Gemini key DB da AES-256-GCM bilan shifrlanadi
- API limit/key xatosi uchun tayyor error kodi

## 1. Turso database

Turso'da yangi database yarating. Keyin `schema.sql` ichidagi SQL ni Turso shell/dashboard orqali bir marta ishga tushiring.

Sizga 2 ta qiymat kerak:

- `TURSO_DATABASE_URL`
- `TURSO_AUTH_TOKEN`

Turso CLI ishlatsangiz URL va tokenni olishning odatiy buyruqlari:

```bash
turso db show DATABASE_NOMI --url
turso db tokens create DATABASE_NOMI
```

## 2. GitHub

Bu papkadagi barcha fayllarni yangi repo rootiga joylang.

`.env` faylini GitHub'ga yuklamang. `.gitignore` allaqachon tayyor.

## 3. Render Web Service

Render -> New -> Web Service -> GitHub repo ni tanlang.

- Runtime: Node
- Build Command: `npm install`
- Start Command: `npm start`
- Health Check: `/api/health`

Environment Variables:

```text
NODE_ENV=production
TURSO_DATABASE_URL=libsql://...
TURSO_AUTH_TOKEN=...
JWT_SECRET=uzun-random-secret
ADMIN_PASSWORD=maktab4949
SETTINGS_ENCRYPTION_KEY=boshqa-uzun-random-secret
ALLOWED_ORIGINS=https://SIZNING-SAYTINGIZ.netlify.app
```

`JWT_SECRET` va `SETTINGS_ENCRYPTION_KEY` uchun uzun random matn ishlating.

## 4. Gemini API key

Gemini keyni `.env` ga qo'yish shart emas.

Frontend Administrator -> Sozlamalar orqali quyidagi endpointga yuboradi:

```http
PUT /api/admin/settings/gemini
Authorization: Bearer ADMIN_TOKEN
Content-Type: application/json

{
  "apiKey": "YANGI_GEMINI_KEY",
  "model": "gemini-3.5-flash-lite"
}
```

Backend API keyni shifrlab Turso'dagi `settings` jadvaliga saqlaydi va keyni hech qachon GET response bilan qaytarmaydi.

Agar Gemini 401/403/429 qaytarsa frontendga:

```json
{
  "error": "GEMINI_KEY_OR_LIMIT_ERROR",
  "message": "API key ishlamayapti yoki so‘rov limiti tugagan. Administrator → Sozlamalar bo‘limidan yangi Gemini API key o‘rnating."
}
```

keladi. Admin yangi keyni saqlashi bilan keyingi AI so'rov yangi keydan foydalanadi; Render restart shart emas.

## 5. Frontend ulash

Frontendda bitta API URL saqlaysiz:

```js
const API_BASE = 'https://YOUR-SERVICE.onrender.com';
```

Live yangilanish:

```js
const events = new EventSource(`${API_BASE}/api/events`);
events.addEventListener('results_changed', () => loadResults());
events.addEventListener('access_changed', () => loadAccessRequests());
events.addEventListener('tests_changed', () => loadTests());
```

Bu bilan bir qurilmada o'quvchi testni topshirganda boshqa qurilmadagi ochiq Admin/O'qituvchi paneli server eventini oladi va natijani qayta yuklaydi.

## Muhim

Hozirgi oldingi Gemini keyni repo ichiga yozmang. Chatda ko'ringan key bo'lsa, yangi keyga almashtirish tavsiya qilinadi.
