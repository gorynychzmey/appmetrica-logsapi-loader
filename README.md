# AppMetrica LogsAPI Loader

Python-script for automatic loading from AppMetrica LogsAPI into local ClickHouse DB. And yes, there is a Docker container too.

## How to use this image

To use this image you should generate [OAuth token][OAUTH-DOCS] for [AppMetrica Logs API][LOGSAPI-DOCS]. This token should be provided via environment variable `TOKEN`.

Also you should copy app's numeric IDs. You could find them in General Settings of your app ("Application ID"). All IDs should be provided as JSON-array via environment variable `APP_IDS`.

### Start manualy
```bash
$ docker run -d --name clickhouse --ulimit nofile=262144:262144 yandex/clickhouse-server
$ docker run -d --name appmetrica-funnels --link clickhouse -p 5000:5000 \
    -e 'CH_HOST=http://clickhouse:8123' \
    -e 'TOKEN=YOUR_OAUTH_TOKEN' \
    -e 'APP_IDS=["YOUR_APP_ID"]' \
    -e 'SOURCE_NAME=events' \
    -e 'FIELDS=["event_name","event_timestamp","appmetrica_device_id","os_name","country_iso_code","city"]' \
    yandex/appmetrica-funnels
```

More information about [ClickHouse server image][CLICKHOUSE-SERVER].

### Start with Docker Compose
Download this repository and run:
```bash
$ echo 'TOKEN=YOUR_OAUTH_TOKEN' > .env
$ echo 'APP_IDS=["YOUR_APP_ID"]' >> .env
$ docker-compose up -d
```

This commands create `.env` file with variables required by script. 

## Configuration

All configuration properties can be passed through environment variables.
 
#### Main variables
* `TOKEN` - *(required)* Logs API OAuth token.
* `APP_IDS` - *(required)* JSON-array of numeric AppMetrica app identifiers.
* `SOURCE_NAME` - *(required)* Logs API endpoint to download from. Supported values: `events`, `crashes`.
* `FIELDS` - *(required)* List of fields to download. See [LogsAPI endpoints][LOGSAPI-ENDPOINTS] and `fields/declaration.py` file for more information.
* `KEY_FIELDS` - List of fields for deduplication. All fields are used by default.

#### ClickHouse related
* `CH_HOST` - Host of ClickHouse DB to store events. (default: `http://localhost:8123`)
* `CH_USER` - Login of ClickHouse DB. (default: empty)
* `CH_PASSWORD` - Password of ClickHouse DB. (default: empty)
* `CH_DATABASE` - Database in ClickHouse to create tables in. (default: `mobile`)
* `CH_TABLE` - Table in database to upload in. (default: `events_all`)

#### LogsAPI related
* `LOGS_API_HOST` - Host of LogsAPI api. (default: `https://api.appmetrica.yandex.ru`)
* `REQUEST_CHUNK_ROWS` - Size of chunks to process at once. (default: `10000`)

#### Scheduling configuration
* `UPDATE_LIMIT` - Count of days for the first events fetch. (default: `30`)
* `FRESH_LIMIT` - Count of days for which events are treated as fresh. (default: `7`)
* `UPDATE_INTERVAL` - Interval of time in hours between events fetches from Logs API. (default: `12`)

#### Other variables
* `DEBUG` - Enables extended logging. Possible values: `0`, `1`. (default: `0`)
* `STATE_FILE_PATH` - Path to file with script state. (default: `state.json`)

## License
License agreement on use of Yandex AppMetrica is available at [EULA site][LICENSE]


[LOGSAPI-DOCS]: https://tech.yandex.com/appmetrica/doc/mobile-api/logs/about-docpage/ "AppMetrica LogsAPI documentation"
[OAUTH-DOCS]: https://tech.yandex.com/appmetrica/doc/mobile-api/intro/authorization-docpage/ "Yandex OAuth documentation"
[CLICKHOUSE-SERVER]: https://hub.docker.com/r/yandex/clickhouse-server/ "ClickHouse Server Docker image page"
[LOGSAPI-ENDPOINTS]: https://tech.yandex.com/appmetrica/doc/mobile-api/logs/endpoints-docpage/ "AppMetrica LogsAPI Endpoints documentation"
[LICENSE]: https://yandex.com/legal/metrica_termsofuse/ "Yandex AppMetrica agreement"