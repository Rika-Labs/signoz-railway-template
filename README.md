# Deploy SigNoz on Railway

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https://github.com/Rika-Labs/signoz-railway-template)

**[SigNoz](https://signoz.io)** is an open-source observability platform for collecting, storing, and analyzing **traces, metrics, and logs** using OpenTelemetry.

## Quick Deploy

Click the button above to deploy SigNoz to Railway. The template will provision:

- **SigNoz** - Main dashboard and query service
- **SigNoz OTel Collector** - Receives telemetry data (OTLP)
- **ClickHouse** - Time-series database for storage
- **Zookeeper** - ClickHouse coordination

## Post-Deployment Setup

### 1. Set ClickHouse Connection Variables

After deployment, configure these environment variables on the **signoz** and **signoz-otel-collector** services:

| Variable | Value |
|----------|-------|
| `CLICKHOUSE_HOST` | `${{clickhouse.RAILWAY_PRIVATE_DOMAIN}}` |
| `CLICKHOUSE_PORT` | `9000` |

### 2. Run Schema Migrations

The schema migrators run automatically. If services fail on first boot:

1. Wait for `signoz-sync-schema-migrator` to complete
2. Redeploy `signoz-async-schema-migrator`
3. Redeploy `signoz`
4. Redeploy `signoz-otel-collector`

### 3. Configure Your Application

Point your OpenTelemetry SDK to the collector:

```bash
# gRPC (port 4317)
OTEL_EXPORTER_OTLP_ENDPOINT=http://signoz-otel-collector.railway.internal:4317

# HTTP (port 4318)
OTEL_EXPORTER_OTLP_ENDPOINT=http://signoz-otel-collector.railway.internal:4318
```

## Manual Deployment

If the template button doesn't work, deploy manually:

1. Create a new Railway project
2. Add services from this repo:
   - **clickhouse** - Use `clickhouse/clickhouse-server:24.1.2-alpine` image
   - **zookeeper** - Use `bitnami/zookeeper:3.8` image
   - **signoz** - Deploy from `signoz/` directory with `Dockerfile.signoz`
   - **signoz-otel-collector** - Deploy from `signoz/` directory with `Dockerfile.otel`

3. Add volumes:
   - ClickHouse: `/var/lib/clickhouse/`
   - SigNoz: `/var/lib/signoz/`

4. Set environment variables as described above

## Environment Variables Reference

### SigNoz Service

| Variable | Description | Default |
|----------|-------------|---------|
| `SIGNOZ_TELEMETRYSTORE_CLICKHOUSE_DSN` | ClickHouse connection string | - |
| `SIGNOZ_SQLSTORE_SQLITE_PATH` | SQLite database path | `/var/lib/signoz/signoz.db` |
| `SIGNOZ_ALERTMANAGER_PROVIDER` | Alertmanager provider | `signoz` |

### OTel Collector Service

| Variable | Description | Default |
|----------|-------------|---------|
| `CLICKHOUSE_HOST` | ClickHouse hostname | `clickhouse.railway.internal` |
| `CLICKHOUSE_PORT` | ClickHouse port | `9000` |
| `LOW_CARDINAL_EXCEPTION_GROUPING` | Exception grouping setting | `false` |

## Ingestion Endpoints

| Protocol | Port | Endpoint |
|----------|------|----------|
| OTLP gRPC | 4317 | `signoz-otel-collector.railway.internal:4317` |
| OTLP HTTP | 4318 | `signoz-otel-collector.railway.internal:4318/v1/traces` |

## Fixes in This Fork

This fork includes fixes for Railway deployment:

1. **ClickHouse hostname** - Uses environment variables instead of hardcoded `clickhouse`
2. **Start command** - Properly runs `./signoz server`
3. **Default env vars** - OTel collector has sensible defaults for Railway networking

## Resources

- [SigNoz Documentation](https://signoz.io/docs/)
- [OpenTelemetry Docs](https://opentelemetry.io/docs/)
- [Railway Docs](https://docs.railway.com/)

## License

Apache 2.0 - See [SigNoz License](https://github.com/SigNoz/signoz/blob/main/LICENSE)
