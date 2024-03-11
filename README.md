## API Documentation
App service API can be found [here](https://app.co:8004/docs/index.html)
Dashboard service API can be found [here](https://app.co:8010/docs/index.html)

## Config File
The app supports a config file in YAML format.
The name of this file is `app.yaml`, the location is the `root directory`.

## Environment Variables
| Name                                  | Default                                       | Description                                                             |
|---------------------------------------|-----------------------------------------------|-------------------------------------------------------------------------|
| S3_DB_IP_BUCKET                       | `dev-db-ip`                                   | S3 bucket where DB-IP file stored is                                    |
| API_ADDRESS                           | `:8004`                                       | Starts huawei-proxy server with a given address                         |
| PRE_BID_ADDRESS                       | `http://localhost:8000`                       | Prebid server endpoint                                                  |
| ENVIRONMENT_TYPE                      | `local`                                       | Available types: local, dev, test, prod                                 |
| LOGGER_ADDRESS                        | `https://ads.app.co`                     | Endpoint for storing data to mongoDB                                    |
| LOG_TICK_TIME                         | `8`                                           | Number of seconds for collection logs before sending to CloudWatch      |
| AWS_REGION                            | `eu-central-1`                                | AWS Region                                                              |
| AWS_SECRET_ID                         | `app-dev-secret`                        | AWS Secret ID                                                           |
| HOSTNAME                              | the value from OS                             | Hostname uses for logging                                               |
| HTTP_SHUTDOWN_TIMEOUT                 | `3`                                           | HTTP Server shutdown timeout in seconds                                 |
| SERVICE_SHUTDOWN_TIMEOUT              | `10`                                          | Service shutdown timeout in seconds (to complete log and db operations) |
| AD_BLOCK_LATENCY_LIMIT                | `100`                                         | Max time for adblock checking in milliseconds                           |
| FETCHING_BUNDLES_INTERVAL             | `60`                                          | The interval in minutes app fetches the third party apps data     |
| DASHBOARD_ADDRESS                     | `http://localhost:3000,http://localhost:8006` | Address of frontend dashboard                                           |
| FETCHING_NATIVEX_INTERVAL             | `30`                                          | The interval in minutes app fetches the nativex posts             |
| FETCHING_AFFILIATE_INTERVAL           | `60`                                          | The interval in minutes app fetches the affiliate posts           |
| RESIZE_SERVER_ADDRESS                 | `https://img.app.co`                 | Endpoint for image resizer                                              |


## Flags
| Name    | Default |
|---------|---------|
| tracing | `false` |

To show errors with stack traces in the API, use the `tracing` flag.
```bash
$ go run main.go --tracing=true
```
or
```bash
$ go run main.go --tracing
```

## Start project locally
1) Clone repository:
```bash
$ git clone git@github.com:app/app-git
$ cd app
```
2) [Install dependencies](#install-dependencies)

3) Copy config file:
```bash
$ cp app.yaml.example app.yaml
```
4) Run postgres and redis
```bash
$ docker compose up -d postgres redis
```
5) Run:
```bash
$ go run main.go
```

## Settings Priority
There are several ways how to set up the value of some variable.
It can be a config file, an environment variable, a flag or the variable can have no value (and uses default value).
The priority of these sources are below (from lower to higher):

`Defaults` &#8594; `Config File` &#8594; `Environment Variables` &#8594; `Flags`

## Secrets
| Name                    | Description                                                                                                                                  |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| DASHBOARD_USERNAME      | Dashboard username                                                                                                                           |
| DASHBOARD_PASSWORD      | Dashboard password                                                                                                                           |
| LOGGER_USERNAME         | Logger username (for db client)                                                                                                              |
| LOGGER_PASSWORD         | Logger password (for db client)                                                                                                              |
| PG_HOST                 | Postgres host                                                                                                                                |
| PG_PORT                 | Postgres port                                                                                                                                |
| PG_LOGIN                | Postgres username                                                                                                                            |
| PG_PASSWORD             | Postgres password                                                                                                                            |
| PG_DB_NAME              | Postgres database name                                                                                                                       |
| PG_SCHEMA               | Postgres schema name (default is public)                                                                                                     |

## Install dependencies.
This project uses Go Modules.

To set up your environment to work with private repositories you can run
```bash
$ go env -w GOPRIVATE=github.com/app/*
$ git config --global url."git@github.com:".insteadOf "https://github.com/"
```

In order to update all dependencies to the newest (non major) versions and add the missing dependencies, you can run the command below:

```bash
$ go mod tidy
```

## Test coverage

```bash
$ go test -coverpkg ./... ./... -tags=integration -coverprofile cover.out; go tool cover -func cover.out
```

This packages we are using inside our system like a secondary service.

* Encoder - USING ONLY IN DASHBOARD.
* Filter - used to filter clients for request.

## Filters
In package (helpers/filters) we have added structure Settings
1. We are generating all keys which could be with current clients settings.
2. For each key we are adding names of clients which could be used for it.
3. When request coming we are generating SplitKey For this request and using only clients from
   Split service which have match by SplitKey.
#### Settings
This structure using in each client, and by this data we are filtering client which doesn't support incoming request.
Also, that settings using for creation SplitKey.

#### Filtering
1. To add the additional filter it's should implement method Filter, new filter should be set in property of last previous filter
2. If current settings have match with incoming key then it should be added in SplitService.
3. If you have added new key to the split key, logic for this key should be described in method GenerateKeys.
4. Add to InitSplitService your filter.
5. Don't forget check test work.

## Logger
For logging, we are using package (repositories/logger).
Each instance logging data to separate stream with hostname prefix. We are collect data for some time and send it like a batch.
1. We are launching method LaunchLoggingData with specific Duration.
2. We are adding new messages to slice with cloudwatchlogs.InputLogEvent
3. if message doesn't bigger than 262144 bytes and batch doesn't bigger than 1048576 bytes
