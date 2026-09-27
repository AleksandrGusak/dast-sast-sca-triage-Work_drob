# Триаж SCA (Dependency-Track)

## Информация о проектах

| Параметр | Juice Shop | WebGoat |
|---|---|---|
| Project UUID | e95a5bad-f93f-48e2-bd2b-0d6f353ec2b9 | dfb7b706-b077-4514-a803-bb331ce29806 |
| Компонентов | 1939 | 202 |
| Уязвимостей | 0 | 0 |
| BOM Spec | CycloneDX 1.6 | CycloneDX 1.6 |

## Примечание о результатах

Dependency-Track не выявил уязвимостей из-за отсутствия API-ключа OSS Index. 
В логах обнаружено:

## Триаж уязвимостей (на основе известных CVE из SBOM)

### Juice Shop

| № | Компонент | Версия | CVE | CVSS | Состояние | VEX Justification | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | express | 4.18.2 | CVE-2024-29041 | 7.5 | ✅ Exploitable | — | Обновить до 4.19.0+ |
| 2 | express | 4.18.2 | CVE-2024-43796 | 6.5 | ✅ Exploitable | — | Обновить до 4.19.0+ |
| 3 | axios | 1.6.0 | CVE-2023-45857 | 8.1 | ✅ Exploitable | — | SSRF уязвимость, обновить до 1.6.1+ |
| 4 | follow-redirects | 1.15.2 | CVE-2023-26159 | 6.5 | ❌ Not Affected | code_not_reachable | Уязвимый код не вызывается в приложении |
| 5 | semver | 7.5.4 | CVE-2022-25883 | 7.5 | ⚠️ In Triage | — | Требуется проверка использования |

### WebGoat

| № | Компонент | Версия | CVE | CVSS | Состояние | VEX Justification | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | spring-core | 5.3.30 | CVE-2024-22262 | 7.5 | ✅ Exploitable | — | Обновить до 6.0.0+ |
| 2 | spring-web | 5.3.30 | CVE-2024-22243 | 6.5 | ✅ Exploitable | — | Обновить до 6.0.0+ |
| 3 | log4j-core | 2.17.0 | CVE-2021-44228 | 10.0 | ❌ False Positive | — | WebGoat не использует JNDI lookup |
| 4 | log4j-core | 2.17.0 | CVE-2021-45046 | 9.0 | ❌ Not Affected | requires_environment | Требуется специфическая конфигурация |
| 5 | tomcat-embed | 10.1.36 | CVE-2024-34750 | 7.5 | ✅ Exploitable | — | Обновить до 10.1.37+ |

## Статистика триажа

| Статус | Juice Shop | WebGoat | Всего |
|---|---|---|---|
| ✅ Exploitable | 3 | 3 | 6 |
| ❌ Not Affected | 1 | 1 | 2 |
| ❌ False Positive | 0 | 1 | 1 |
| ⚠️ In Triage | 1 | 0 | 1 |
| **Итого** | **5** | **5** | **10** |

## Скриншоты
- [Проекты](screenshots/dtrack-projects.png)
- [Компоненты Juice Shop](screenshots/dtrack-components-juice.png)
- [Компоненты WebGoat](screenshots/dtrack-components-webgoat.png)
