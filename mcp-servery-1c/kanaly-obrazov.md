# Каналы образов: stable и beta

MCP-серверы публикуются в двух независимых каналах. **Stable** предназначен для обычной эксплуатации, **beta** — для предварительного доступа к новым контрактам, инструментам и форматам хранения.

> Состав тегов ниже проверен в Docker Hub 24.08.2026. Актуальный список всегда смотрите по ссылке на образ.

## Как выбрать канал

| Канал | Теги | Когда использовать |
|-------|------|--------------------|
| `stable` | `latest`, `light`, `arm64` — если вариант существует у сервера | Нужна максимально консервативная поставка |
| `beta` | `latest-beta`, `light-beta`, `arm64-beta` — если вариант существует у сервера | Нужны последние функции и контракты, описанные в этой документации |

Суффикс beta пишется **через дефис**. Тегов `latestbeta` и `lightbeta` нет.

В дистрибутиве итоговый тег вычисляется так:

```text
IMAGE_TAG = IMAGE_VARIANT + ("-beta", если RELEASE_CHANNEL=beta)
```

Непустой `IMAGE_TAG` в `config.env` перекрывает `RELEASE_CHANNEL` и `IMAGE_VARIANT`.

## Доступные теги

| Сервер | Образ | Stable | Beta |
|--------|-------|--------|------|
| HelpSearchServer | [`comol/1c_help_mcp`](https://hub.docker.com/r/comol/1c_help_mcp/tags) | `latest`, `light`, `arm64` | `latest-beta`, `light-beta`, `arm64-beta` |
| Graph Metadata Search | [`comol/1c_graph_metadata`](https://hub.docker.com/r/comol/1c_graph_metadata/tags) | `latest`, `light`, `arm64` | `latest-beta`, `light-beta`, `arm64-beta` |
| CodeMetadataSearchServer | [`comol/1c_code_metadata_mcp`](https://hub.docker.com/r/comol/1c_code_metadata_mcp/tags) | `latest`, `light`, `arm64` | `latest-beta`, `light-beta`, `arm64-beta` |
| SSLSearchServer | [`comol/mcp_ssl_server`](https://hub.docker.com/r/comol/mcp_ssl_server/tags) | `latest`, `light`, `arm64` | `latest-beta`, `light-beta`, `arm64-beta` |
| TemplatesSearchServer | [`comol/template-search-mcp`](https://hub.docker.com/r/comol/template-search-mcp/tags) | `latest`, `light`, `arm64` | `latest-beta`, `light-beta`, `arm64-beta` |
| SyntaxCheckServer | [`comol/1c_syntaxcheck_mcp`](https://hub.docker.com/r/comol/1c_syntaxcheck_mcp/tags) | только `latest` | `latest-beta`, `arm64-beta` |
| 1CCodeChecker | [`comol/1c-code-checker`](https://hub.docker.com/r/comol/1c-code-checker/tags) | `latest`, `arm64` | `latest-beta`, `light-beta`, `arm64-beta` |

{% hint style="warning" %}
Не используйте теги с префиксом `staging-` как канал поставки. Это технические теги сборки; их наличие и срок жизни не гарантируются.
{% endhint %}

## Лицензионные ключи

Выбирайте ключ того же канала, что и образ. В `config.env` стабильный ключ хранится в `LICENSE_KEY_<СЕРВЕР>`, beta-ключ — в `LICENSE_KEY_<СЕРВЕР>_BETA`.

| Сервер | Stable | Beta |
|--------|--------|------|
| HelpSearchServer | `LICENSE_KEY_HELP` | `LICENSE_KEY_HELP_BETA` |
| Graph Metadata Search | `LICENSE_KEY_GRAPH` | `LICENSE_KEY_GRAPH_BETA` |
| CodeMetadataSearchServer | `LICENSE_KEY_CODEMETADATA` | `LICENSE_KEY_CODEMETADATA_BETA` |
| SSLSearchServer | `LICENSE_KEY_SSL` | `LICENSE_KEY_SSL_BETA` |
| TemplatesSearchServer | `LICENSE_KEY_TEMPLATES` | `LICENSE_KEY_TEMPLATES_BETA` |
| SyntaxCheckServer | `LICENSE_KEY_SYNTAX` | `LICENSE_KEY_SYNTAX_BETA` |
| 1CCodeChecker | `LICENSE_KEY_CODECHECKER` | `LICENSE_KEY_CODECHECKER_BETA` |

Ключи каналов считайте невзаимозаменяемыми, даже если конкретный ключ временно принимается обеими сборками. Неверная пара образа и ключа приводит к `Invalid LICENSE_KEY` и немедленному завершению контейнера.

## Совместимость каналов

Порты, URL `/mcp`, имена серверов в `mcp.json` и `connection_id` при переключении канала не меняются. Меняются тег образа, лицензионный ключ и иногда формат постоянных данных.

{% hint style="danger" %}
Не подключайте один каталог индекса одновременно к stable- и beta-контейнерам. Для отката сохраняйте старый контейнер и используйте отдельный каталог данных. У HelpSearchServer это обязательно: stable хранит ChromaDB в `/app/chroma_db`, новые beta-сборки — поколения zvec в `/app/index`.
{% endhint %}

Безопасное переключение:

1. Остановите старый контейнер.
2. Переименуйте его в резервную копию с датой.
3. Скачайте тег выбранного канала.
4. Запустите новый контейнер с ключом этого канала и отдельным каталогом данных, если формат хранения отличается.
5. Проверьте readiness и `tools/list`, а не только состояние `running`.
6. После проверки удалите резервную копию или оставьте её для быстрого отката.

Команды запуска и особенности томов приведены на страницах каждого сервера.
