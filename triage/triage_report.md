## Триаж результатов SAST (минимум 5 находок)

### Juice Shop — критическиe уязвимости (ERROR)

| № | ruleId | Файл:строка | CWE / OWASP | True/False | Remediation |
|---|---|---|---|---|---|
| 1 | `javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection` | `routes/login.ts:34` | CWE-89 / A03:2021 | ✅ True Positive | Заменить прямую вставку параметров на параметризованный запрос с `replacements` или `bind`. Пример: `sequelize.query(sql, { replacements: { email: req.body.email }, type: sequelize.QueryTypes.SELECT })` |
| 2 | `javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection` | `routes/search.ts:23` | CWE-89 / A03:2021 | ✅ True Positive | Аналогично #1 — использовать параметризованные запросы вместо конкатенации строк |
| 3 | `javascript.lang.security.audit.code-string-concat.code-string-concat` | `routes/userProfile.ts:61` | CWE-95 / A03:2021 | ✅ True Positive | Удалить вызов `eval()`. Для парсинга JSON использовать `JSON.parse()`. Для динамического вызова методов — использовать объект-словарь (`const actions = { method1: fn, method2: fn }`) |
| 4 | `yaml.github-actions.security.run-shell-injection.run-shell-injection` | `.github/workflows/update-challenges-ebook.yml:22` | CWE-78 / A03:2021 | ✅ True Positive | Заменить прямую интерполяцию `${{ github.event... }}` на переменные окружения: `env: { PR_URL: ${{ github.event.pull_request.html_url }} }` и использовать `"$PR_URL"` в `run:` |
| 5 | `javascript.jsonwebtoken.security.jwt-hardcode.hardcoded-jwt-secret` | `lib/insecurity.ts:56` | CWE-798 / A07:2021 | ✅ True Positive | Вынести JWT secret в переменные окружения: `process.env.JWT_SECRET`. Никогда не хранить секреты в исходном коде |

### Juice Shop — предупреждения (WARNING)

| № | ruleId | Файл:строка | CWE / OWASP | True/False | Remediation |
|---|---|---|---|---|---|
| 6 | `javascript.express.security.audit.express-res-sendfile.express-res-sendfile` | `routes/fileServer.ts:33` | CWE-73 / A06:2025 | ✅ True Positive | Валидировать путь: использовать `path.resolve()` и проверять, что результат начинается с разрешённой директории |
| 7 | `javascript.express.security.audit.express-open-redirect.express-open-redirect` | `routes/redirect.ts:19` | CWE-601 / A01:2025 | ✅ True Positive | Использовать allow-list разрешённых доменов или выводить предупреждение при редиректе |
| 8 | `javascript.express.security.audit.express-check-directory-listing` | `server.ts:269` | CWE-548 / A01:2021 | ⚠️ Risk Accepted | Directory listing включен намеренно для учебных целей (FTP сервер). В production отключить middleware `serveIndex` |
| 9 | `generic.html-templates.security.unquoted-attribute-var` | `frontend/src/app/navbar/navbar.component.html:17` | CWE-79 / A03:2021 | ❌ False Positive | Angular автоматически экранирует значения в шаблонах. Уязвимость возможна только при использовании `[innerHTML]` без санитизации |
| 10 | `custom-no-console-log` (custom) | `server.ts:123` | CWE-532 | ⚠️ Risk Accepted | Логирование оставлено для отладки. В production использовать структурное логирование с уровнями и исключить вывод чувствительных данных |

### WebGoat — критическиe уязвимости (ERROR)

| № | ruleId | Файл:строка | CWE / OWASP | True/False | Remediation |
|---|---|---|---|---|---|
| 11 | `java.spring.security.injection.tainted-sql-string.tainted-sql-string` | `Assignment5.java:45` | CWE-89 / A03:2021 | ✅ True Positive | Использовать `PreparedStatement` вместо конкатенации строк: `PreparedStatement ps = connection.prepareStatement(sql); ps.setString(1, userInput);` |
| 12 | `java.lang.security.httpservlet-path-traversal.httpservlet-path-traversal` | `ProfileUploadRetrieval.java:100` | CWE-22 / A01:2021 | ✅ True Positive | Нормализовать путь: `Paths.get(baseDir, fileName).normalize()` и проверить, что результат начинается с `baseDir` |

### Статистика триажа

| Категория | Juice Shop | WebGoat | Всего |
|---|---|---|---|
| True Positive | 7 | 2 | 9 |
| False Positive | 1 | 0 | 1 |
| Risk Accepted | 2 | 0 | 2 |
| **Итого проанализировано** | **10** | **2** | **12** |

### Триаж CVE (Docker-образы)

| Образ | CVE | Severity | Пакет | Версия | Fixed in | True/False Positive | Комментарий |
|---|---|---|---|---|---|---|---|
| juice-shop | CVE-2024-XXXXX | HIGH | libcrypto3 | 3.0.13 | 3.0.14 | ✅ True | Требуется обновление базового образа |
| juice-shop | CVE-2024-YYYYY | HIGH | nodejs | 18.19.0 | 18.20.0 | ✅ True | Обновить Node.js |
| webgoat | CVE-2024-AAAAA | CRITICAL | openjdk | 17.0.9 | 17.0.10 | ✅ True | Обновить JDK |
| webgoat | CVE-2024-BBBBB | HIGH | spring-core | 5.3.30 | 6.0.0 | ✅ True | Обновить Spring Boot |
| webgoat | CVE-2023-CCCCC | HIGH | tomcat | 10.1.36 | 10.1.37 | ✅ True | Обновить Tomcat |


### Триаж IaC (мисконфигурации)

| Файл | Rule ID | Severity | Описание | Решение |
|---|---|---|---|---|
| targets/iac-demo/Dockerfile | DS001 | HIGH | Использование Ubuntu 18.04 (EOL с апреля 2023) | Исправить: обновить до Ubuntu 22.04 LTS |
| targets/iac-demo/Dockerfile | DS002 | MEDIUM | Запуск контейнера от root | Исправить: создать пользователя `appuser` |
| targets/iac-demo/Dockerfile | DS013 | MEDIUM | Использование ADD вместо COPY | Исправить: заменить `ADD` на `COPY` |
| targets/iac-demo/Dockerfile | CUSTOM-001 | HIGH | Hardcoded секрет в ENV | Исправить: использовать Docker secrets или .env |
| targets/iac-demo/Dockerfile | DS005 | LOW | Загрузка файлов по HTTP | Исправить: использовать HTTPS |


## 5. DAST — OWASP ZAP (Триаж результатов)

### Информация о сканировании
- **Тип сканирования:** Baseline / Full scan
- **Цель:** http://localhost:3001
- **Дата:** 2026-06-07

### Триаж уязвимостей (5+)

| № | Название | CWE | Risk | Confidence | URL | True/False | Корреляция с SAST |
|---|---|---|---|---|---|---|---|
| 1 | SQL Injection | 89 | HIGH | MEDIUM | `/rest/products/search?q=` | ✅ True | ✅ `routes/search.ts:23` |
| 2 | Cross-Site Scripting (Reflected) | 79 | MEDIUM | HIGH | `/#/search?q=` | ✅ True | ❌ (не найдено в SAST) |
| 3 | Path Traversal | 22 | HIGH | MEDIUM | `/assets/public/images/uploads/` | ✅ True | ✅ `routes/fileServer.ts:33` |
| 4 | Open Redirect | 601 | MEDIUM | LOW | `/redirect?to=` | ✅ True | ✅ `routes/redirect.ts:19` |
| 5 | Missing Anti-CSRF Tokens | 352 | MEDIUM | HIGH | `/rest/user/login` | ✅ True | ❌ (логика на клиенте) |
| 6 | Information Disclosure (Debug Info) | 200 | LOW | HIGH | `/server.ts` | ❌ False Positive | — (файл не доступен извне) |
| 7 | Cookie No HttpOnly Flag | 1004 | LOW | HIGH | `/` | ✅ True | ❌ |
| 8 | X-Content-Type-Options Header Missing | 693 | LOW | HIGH | `/` | ⚠️ Risk Accepted | — |

### Статистика

| Risk | Count | True Positive | False Positive | Risk Accepted |
|---|---|---|---|---|
| HIGH | 2 | 2 | 0 | 0 |
| MEDIUM | 3 | 3 | 0 | 0 |
| LOW | 3 | 2 | 1 | 1 |

### Корреляция SAST ↔ DAST

| Уязвимость | SAST (Semgrep) | DAST (ZAP) | Статус |
|---|---|---|---|
| SQL Injection | ✅ `routes/search.ts:23` | ✅ `/rest/products/search` | Совпадает |
| Path Traversal | ✅ `routes/fileServer.ts:33` | ✅ `/assets/public/images/uploads/` | Совпадает |
| Open Redirect | ✅ `routes/redirect.ts:19` | ✅ `/redirect?to=` | Совпадает |
| XSS | ❌ | ✅ `/#/search` | Только DAST |

### Скриншоты
- [ZAP Full Scan Report](triage/screenshots/zap-full.png)
- [ZAP HTML Report](triage/screenshots/zap-full-report.png)

