# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Что это

Конфигурация для [Shadowrocket](https://apps.apple.com/app/shadowrocket/id932747118) (iOS proxy-клиент). Здесь нет кода, сборки, тестов и линтеров — только декларативные текстовые файлы конфигурации. Все комментарии и коммиты — на русском.

## Модель деплоя (главное, что нужно понимать)

Shadowrocket загружает конфигурацию **по сети с GitHub**, а не из локальных файлов:

- `config.conf` — корневой файл. Shadowrocket периодически перечитывает его по `update-url` (строка в `[General]`), который указывает на `raw.githubusercontent.com/.../master/config.conf`.
- `config/*.list` — наборы правил. В `config.conf` каждый подключён директивой `RULE-SET` с **абсолютным URL** на `raw.githubusercontent.com/.../master/config/<имя>.list`.

Следствия:
- **Изменения вступают в силу только после `git push` в `master`** и последующего обновления конфига на устройстве. Локальная правка ни на что не влияет.
- При добавлении нового `.list` файла нужно **двумя действиями**: создать файл И добавить ссылку `RULE-SET,<raw-url>,PROXY` в `config.conf`. Файл без ссылки игнорируется; ссылка без файла даёт ошибку загрузки на устройстве.
- Все `RULE-SET` URL захардкожены на ветку `master`. При переименовании файла или ветки URL надо править вручную.

## Логика маршрутизации

Последняя строка `config.conf` — `FINAL,DIRECT`. Это **allowlist-модель (split tunnel)**: по умолчанию весь трафик идёт напрямую, через прокси (`PROXY`) идут **только** домены/IP из подключённых RULE-SET. То есть добавить домен в `.list` = «пустить этот домен через прокси». Это противоположность глобального VPN.

Порядок правил важен: правила проверяются сверху вниз, срабатывает первое совпадение, затем `FINAL`.

## Формат `.list` файлов

Одна директива на строку. Используемые типы:
- `DOMAIN,<host>` — точное совпадение хоста.
- `DOMAIN-SUFFIX,<domain>` — домен и все поддомены.
- `DOMAIN-KEYWORD,<substr>` — любой хост, содержащий подстроку.
- `IP-CIDR,<cidr>` / `IP-CIDR6,<cidr>` — диапазоны IP; к ним добавляют `,no-resolve`, чтобы не делать DNS-резолв.

Политику (`PROXY`) задаёт директива `RULE-SET` в `config.conf` — в самих `.list` файлах политику в конце строки писать не нужно (только опции правила вроде `no-resolve` для IP-CIDR).

## Файлы

- `config.conf` — `[General]` (DNS, bypass, TUN-маршруты) + `[Rule]` (внешние RULE-SET + кастомные RULE-SET из `config/`).
- `config/` — тематические списки доменов: `ai.list`, `jetbrains.list`, `hosting_providers.list`, `messengers.list`, `payment_systems.list`, `security.list`, `news.list`, `other.list`.
- Внешние (не наши) RULE-SET в `config.conf` тянутся из репозиториев `misha-tgshv` и `helmiau` — их не редактируем, только подключаем.

## Проверка перед коммитом

Автотестов нет. Полезная ручная проверка — что каждый `config/*.list` подключён в `config.conf` (и наоборот):

```sh
# файлы без ссылки в config.conf
for f in config/*.list; do grep -q "config/$(basename "$f")" config.conf || echo "не подключён: $f"; done
```

## Коммиты

История репозитория — короткие сообщения на русском в свободной форме (например, «Добавлен cotypist.ai», «Удалён домен xethub.hf.co»). Придерживайся этого стиля. Не коммить и не пушь без явной просьбы пользователя.
