# ansible-posthog

Ansible-роль для self-hosted PostHog (hobby compose) на UnitPay.

## Операционные выводы после миграции на unitpay-tools-s1 (6–7 октября 2026)

Полный отчёт: [docs/posthog-migration.html](docs/posthog-migration.html).

Кратко, что закреплено в роли:

| Параметр | Значение / поведение | Зачем |
|---|---|---|
| `posthog_caddy_host` | `http://:8000` | За ALB Host не совпадает с site block — иначе Caddy отдаёт пустой 200 |
| `posthog_run_migrations_on_start` | `false` | Штатный `./bin/migrate` с непатченного `latest` падает на `InvalidBasesError` |
| `posthog_git_sha` | `master` | Checkout compose-файлов; тег образа задаётся отдельно через `posthog_app_tag` |
| Существующий `.env` | Не перезаписывается | Секреты сохраняются; `CADDY_HOST` / TLS-блоки дописываются через `lineinfile` |

### Чего не делать на уже мигрированном инстансе

- Не включать `posthog_run_migrations_on_start: true` и не гонять `manage.py migrate` с непатченного `latest`.
- Не менять `POSTHOG_SECRET` / `ENCRYPTION_SALT_KEYS` относительно восстановленной базы.
- Не поднимать стек без `--no-deps`, если нужно только web/worker/proxy: иначе compose пересоздаст ClickHouse/Postgres/Kafka.
- Traefik должен смотреть на `posthog-proxy-1:8000`, не напрямую на web.

### Переменные

См. [defaults/main.yml](defaults/main.yml). Секреты задаются через vault на хосте.

Для чистой установки без уже накатанной схемы выставьте `posthog_run_migrations_on_start: true`.
