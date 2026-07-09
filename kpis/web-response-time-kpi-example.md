# ITSI KPI Definition: Web Service Response Health

Example KPI configuration for monitoring a customer-facing web service inside an ITSI service model, including the adaptive thresholding approach used for predictive alerting.

## Base Search

```spl
index=web_access sourcetype=access_combined_wcookie
| eval response_time_ms=response_time*1000
| stats avg(response_time_ms) as avg_response_time, perc95(response_time_ms) as p95_response_time, count as request_volume by host
```

## KPI Definition (itsi_kpi_base_search.conf pattern)

```ini
[service_health::web_response_time]
title = Web Service - Response Time (P95)
search = `base_search_web_health`
metric = p95_response_time
unit = ms
is_service_entity_filter = 1
aggregate_statop = avg
entity_statop = avg
```

## Thresholding Strategy

Static thresholds fail on services with natural daily/weekly traffic cycles — a threshold that's correct at 3am flags constantly at peak business hours. This KPI uses ITSI's adaptive thresholding (time-policy-based) instead:

| Time Policy | Severity: Medium | Severity: Critical |
|---|---|---|
| Business hours (weekdays 8am-6pm) | P95 > 800ms | P95 > 2000ms |
| Off-hours / weekends | P95 > 400ms | P95 > 1000ms |

**Why this matters:** a fixed 800ms threshold applied uniformly produces false alerts during low-traffic windows (where 800ms might indicate a real problem at 2am but be normal jitter at noon) and misses real degradation during peak load (where 800ms might be dangerously slow at 2pm). Time-policy thresholds tied to actual traffic patterns catch both directions correctly.

## KPI-to-Entity Mapping

Each KPI should map to entities via the `entity_id_fields` so that service health rolls up correctly when you have multiple instances behind a load balancer — alert on the aggregate service health, drill down to the specific unhealthy entity, not the reverse.

---
*Part of the [Splunk-ITSI-Predictive-Monitoring-Engine](../README.md) repository.*
