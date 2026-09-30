# Теги и ключи образов

MCP-серверы публикуются в одном канале: теги `latest`, `light` и `arm64`, без суффикса. Образы от 27.09.2026 — текущие; что вошло в каждую сборку, записано в [Выпусках образов](vypuski.md).

> Состав тегов ниже проверен в Docker Hub 27.09.2026. Актуальный список всегда смотрите по ссылке на образ.

## Доступные теги

| Сервер | Образ | Теги |
|--------|-------|------|
| HelpSearchServer | [`comol/1c_help_mcp`](https://hub.docker.com/r/comol/1c_help_mcp/tags) | `latest`, `light`, `arm64` |
| Graph Metadata Search | [`comol/1c_graph_metadata`](https://hub.docker.com/r/comol/1c_graph_metadata/tags) | `latest`, `light`, `arm64` |
| CodeMetadataSearchServer | [`comol/1c_code_metadata_mcp`](https://hub.docker.com/r/comol/1c_code_metadata_mcp/tags) | `latest`, `light`, `arm64` |
| SSLSearchServer | [`comol/mcp_ssl_server`](https://hub.docker.com/r/comol/mcp_ssl_server/tags) | `latest`, `light`, `arm64` |
| TemplatesSearchServer | [`comol/template-search-mcp`](https://hub.docker.com/r/comol/template-search-mcp/tags) | `latest`, `light`, `arm64` |
| SyntaxCheckServer | [`comol/1c_syntaxcheck_mcp`](https://hub.docker.com/r/comol/1c_syntaxcheck_mcp/tags) | `latest`, `arm64` (варианта `light` нет) |
| 1CCodeChecker | [`comol/1c-code-checker`](https://hub.docker.com/r/comol/1c-code-checker/tags) | `latest`, `light`, `arm64` |
| MCP QA | [`comol/qa_mcp`](https://hub.docker.com/r/comol/qa_mcp/tags) | `latest` (только amd64; вариантов `light` и `arm64` нет) |

В дистрибутиве тег задаётся так:

```text
IMAGE_TAG = IMAGE_VARIANT
```

Непустой `IMAGE_TAG` в `config.env` перекрывает `IMAGE_VARIANT`.

{% hint style="warning" %}
Не используйте теги с префиксом `staging-` как канал поставки. Это технические теги сборки; их наличие и срок жизни не гарантируются.
{% endhint %}

## Лицензионные ключи

У каждого сервера один ключ. В `config.env` дистрибутива он хранится в `LICENSE_KEY_<СЕРВЕР>`, контейнеру передаётся как `LICENSE_KEY`.

| Сервер | Переменная в `config.env` |
|--------|---------------------------|
| HelpSearchServer | `LICENSE_KEY_HELP` |
| Graph Metadata Search | `LICENSE_KEY_GRAPH` |
| CodeMetadataSearchServer | `LICENSE_KEY_CODEMETADATA` |
| SSLSearchServer | `LICENSE_KEY_SSL` |
| TemplatesSearchServer | `LICENSE_KEY_TEMPLATES` |
| SyntaxCheckServer | `LICENSE_KEY_SYNTAX` |
| 1CCodeChecker | `LICENSE_KEY_CODECHECKER` |
| MCP QA | `LICENSE_KEY_QA` |

Ключ одного сервера другой сервер не принимает. Образы от 27.09.2026 принимают только ключи, выпущенные 27.09.2026; ключ, выданный раньше, отклоняется с `Invalid LICENSE_KEY`, и контейнер сразу завершается. Действующий ключ — в `config.env` текущего дистрибутива MCP_Distr или в личном кабинете https://vibecoding1c.ru/.

## Переход на образы от 27.09.2026

27.09.2026 beta-образы стали стабильными: теги `latest`, `light` и `arm64` указывают на те же манифесты, что и `latest-beta`, `light-beta` и `arm64-beta` на эту дату. Отдельного beta-канала больше нет, теги `*-beta` не поддерживаются, в `config.env` нет параметра `RELEASE_CHANNEL` и переменных `LICENSE_KEY_<СЕРВЕР>_BETA`.

{% hint style="info" %}
Ключи, выпущенные 27.09.2026 при ротации beta-ключей, теперь и есть ключи stable. Отдельных beta-ключей больше нет.
{% endhint %}

Порты (80xx), URL `/mcp`, имена серверов в `mcp.json` и `connection_id` — стандартные, как на страницах серверов. Что делать при переходе, зависит от того, какие образы стояли раньше.

### Со stable до 27.09.2026

Нужны новый ключ и новые каталоги данных. Старые индексы новые образы не переиспользуют: например, HelpSearchServer хранит индекс в `/app/index`, а не в ChromaDB `/app/chroma_db`. Индекс строится заново.

1. Возьмите новый ключ `LICENSE_KEY_<СЕРВЕР>` из текущего дистрибутива или личного кабинета.
2. Остановите старый контейнер и переименуйте его в `<имя>_backup_<дата>` — он нужен для отката.
3. Создайте новый каталог данных.
4. Скачайте образ заново (`docker pull`) и запустите контейнер по команде со страницы сервера — с новым ключом и новым каталогом.
5. Проверьте readiness и `tools/list`, а не только состояние `running`.
6. После проверки удалите резервную копию или оставьте её для быстрого отката.

### С бывшего beta

Ключ тот же, выпущенный 27.09.2026: в `config.env` он теперь называется `LICENSE_KEY_<СЕРВЕР>`. Каталоги `*_beta` совместимы с новыми образами, переиндексация не нужна.

1. Остановите beta-контейнер и сохраните его как `<имя>_backup_<дата>`.
2. Создайте контейнер со стандартным именем и портом 80xx из тега без суффикса и подключите к нему каталог `*_beta`.
3. Проверьте readiness и `tools/list`.
4. Уберите из `mcp.json` подключения с суффиксом `-beta` (порты 81xx): остаются стандартные подключения.

{% hint style="danger" %}
Не подключайте один каталог данных одновременно к двум работающим контейнерам — например, к резервной копии и к новому контейнеру.
{% endhint %}

### Если контейнер не стартует

* **`Invalid LICENSE_KEY`, контейнер сразу завершается.** Ключ не подходит к образу: это ключ, выданный до 27.09.2026, или ключ другого сервера. Возьмите `LICENSE_KEY_<СЕРВЕР>` этого сервера из текущего дистрибутива или личного кабинета и пересоздайте контейнер.
* **`manifest unknown` при `docker pull` или `docker run`.** У образа нет такого тега. Сверьте его с таблицей выше: у SyntaxCheckServer нет `light`; тегов `latestbeta` и `lightbeta` не было никогда, а теги `*-beta` не поддерживаются — замените их тегом без суффикса.

Команды запуска и особенности томов приведены на страницах каждого сервера.

## MCP QA

`comol/qa_mcp:latest` — MCP QA 0.7.7, только Linux/amd64; тот же образ опубликован
под тегом `0.7.7`. Контейнер сам работает менеджером тестирования
и подключается к тест-клиенту, запущенному человеком (`MCP_QA_TESTCLIENT`).
Вариантов `light` и `arm64` нет, поэтому общие
`IMAGE_VARIANT`/`IMAGE_TAG` к QA не применяются: образ задаёт `QA_IMAGE` в `config.env`.
Ключ обязателен: `LICENSE_KEY_QA` передаётся серверу как `LICENSE_KEY`; ключ
тот же, что у 0.4.4–0.7.5. Прежние образы доступны по тегам `0.7.5`, `0.7.4`, `0.7.3`, `0.7.2`, `0.7.1`, `0.7.0`
(для входа из контейнера нужно имя-заглушка `MCP_QA_TESTCLIENT_USER`), `0.6.0`
(нативный менеджер с частью операций), `0.5.0` (сквозной прокси к
Windows-серверу). Тегов `0.4.3` и `0.4.4` на Docker Hub нет. [Инструкция QA](servery/qa/README.md).
