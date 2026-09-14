<p align="center">
  <img src="docs/banner.jpg" alt="MMI Schedule System" width="100%">
</p>

# MMI Schedule System

**Информационна система за месечни работни графици** — служителите получават актуален график на телефона си чрез уеб приложение. Въвеждат вътрешен работен номер и виждат индивидуална информация за дневна/нощна смяна, отпуск, болничен и почивни дни.

Неофициален проект за **Мини Марица Изток 2 (MMI)**.

---

## ✨ Основни възможности

- 📥 **Импорт от Excel** — разпознава реалния MMI2 формат с множество блокове в един лист
- 📅 **Персонален календар** — responsive изглед с навигация между месеци и месечна статистика
- 🔐 **Сигурен достъп** — вход чрез работен номер + JWT; admin роли с scrypt hashing
- 🛠️ **First-run installer** — уеб wizard за SQLite/PostgreSQL, Alembic migrations и owner акаунт
- 📱 **REST API** — готов за бъдещо Android native приложение
- 🐳 **Docker** & **GitHub Actions CI** — лесен deployment и автоматични тестове
- 🔄 **Atomic import** + история на импортите с SHA-256 отпечатък
- ⚙️ **Self-update** и system diagnostics

---

## 📋 Легенда на смените

| Excel код | Значение          | API тип       |
|-----------|-------------------|---------------|
| `1`       | Дневна смяна      | `day`         |
| `2`       | Нощна смяна       | `night`       |
| `О` / `0` | Отпуск            | `leave`       |
| `Б`       | Болничен          | `sick_leave`  |
| *(празно)*| Почивка по график | `rest`        |

Всички други стойности се запазват като `unknown` (оригиналният код остава в `raw_code`).

---

## 🚀 Бърз старт

### Локално (manual setup)

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
python -m alembic upgrade head
uvicorn app.main:app --reload

# Linux / macOS
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python -m alembic upgrade head
uvicorn app.main:app --reload
```

Отвори:
- Приложение: http://127.0.0.1:8000
- Admin: http://127.0.0.1:8000/admin
- API docs: http://127.0.0.1:8000/docs

### Docker

```bash
cp .env.example .env
docker compose up --build
```

### First-run installer

Ако няма завършена инсталация, приложението автоматично пренасочва към `/install`. Wizard-ът:

1. Проверява Python/Alembic и права за запис
2. Позволява избор SQLite / PostgreSQL
3. Тества връзката и изпълнява `alembic upgrade head`
4. Създава единствения **owner** акаунт
5. Генерира JWT secret и записва `.env`
6. Заключва installer-а (`install/install.lock`)

> **Важно:** FTP само качва файловете. Нужен е Python ASGI runtime (Passenger, systemd, Docker и т.н.). При PostgreSQL базата трябва да съществува предварително.

Повече: [`install/README.md`](install/README.md)

---

## 🏗️ Архитектура

```text
Excel (.xlsx)
    │
    ├── period detector
    ├── preview (без запис)
    └── confirm import
            ├── Employee (work_number, full_name, team А/Б/В/Г)
            ├── ShiftEntry (date, type, raw_code)
            └── ImportHistory (filename, counts, SHA-256, conflicts)

Employee data
    ├── Web calendar
    └── REST API ──► Android app
```

**Стек:** FastAPI · SQLAlchemy · Alembic · OpenPyXL · JWT · scrypt

---

## 👥 Роли в административния панел

| Роля        | Права |
|-------------|-------|
| **owner**   | Пълен достъп. Единствен може да управлява admin/moderator акаунти. Не може да бъде понижен/деактивиран. |
| **admin**   | Preview/import, ръчни корекции, employee metadata, import & audit history. Без account management. |
| **moderator** | Preview/import, търсене на служители, редакция на дневни записи. Без metadata, history и account management. |

---

## 📡 API (кратък преглед)

| Метод | Endpoint | Описание |
|-------|----------|----------|
| `POST` | `/api/v1/auth/login` | Вход с `{"work_number": "12345"}` |
| `GET`  | `/api/v1/me` | Текущ служител (Bearer token) |
| `GET`  | `/api/v1/me/schedule/{year}/{month}` | Месечен график |
| `POST` | `/api/v1/admin/preview` | Preview на Excel (multipart) |
| `POST` | `/api/v1/admin/import` | Потвърден импорт |
| `GET`  | `/api/v1/admin/imports` | История на импортите |

Пълна документация: Swagger UI на `/docs` и файловете в [`docs/`](docs/).

---

## 📂 Документация

- [Admin accounts](docs/admin-accounts.md)
- [Database migrations](docs/database-migrations.md)
- [Employee API](docs/employee-api.md)
- [Hosting & deployment](docs/hosting-deployment.md)
- [Import behavior](docs/import-behavior.md)
- [Real Excel format](docs/real-excel-format.md)
- [Self-update](docs/self-update.md)
- [System diagnostics](docs/system-diagnostics.md)
- [Update checker](docs/update-checker.md)

---

## 🧪 Тестове & CI

```bash
pytest
```

GitHub Actions автоматично изпълнява migrations и unit тестове при всеки push.

---

## 📄 Лиценз

Неофициален вътрешен проект. Всички права запазени.

---

<p align="center">
  <sub>Изградено с ❤️ за работещите в MMI</sub>
</p>
