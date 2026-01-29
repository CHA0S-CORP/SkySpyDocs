---
title: "Observability"
slug: "libacars-observability"
excerpt: "Metrics collection, Prometheus export, and health monitoring"
hidden: false
---

The libacars binding includes comprehensive observability features for monitoring performance, tracking errors, and integrating with existing monitoring infrastructure.

## Metrics Collection

### MetricsCollector

The central metrics collector tracks all decode operations:

```python
from skyspy_common.libacars import get_metrics_collector

metrics = get_metrics_collector()

# Record a custom timing
with metrics.time_operation("custom_operation"):
    do_something()

# Increment a counter
metrics.increment("custom_counter")

# Set a gauge value
metrics.set_gauge("queue_size", 42)

# Get all metrics
all_metrics = metrics.get_all_metrics()
print(all_metrics)
```

### Metrics Types

The collector supports three metric types:

| Type | Description | Example |
| :--- | :--- | :--- |
| **Counter** | Monotonically increasing value | `decode_success`, `cache_hits` |
| **Gauge** | Current value that can go up/down | `circuit_state`, `cache_size` |
| **Timing** | Duration measurements with statistics | `decode_duration_ms` |

### Built-in Metrics

The binding automatically tracks these metrics:

**Counters:**

| Metric | Description |
| :--- | :--- |
| `decode_attempts` | Total decode attempts |
| `decode_success` | Successful decodes |
| `decode_failures` | Failed decodes |
| `decode_failures_{category}` | Failures by error category |
| `cache_hits` | Cache hit count |
| `cache_misses` | Cache miss count |

**Timings:**

| Metric | Description |
| :--- | :--- |
| `decode` | Decode operation duration (ms) |

**Gauges:**

| Metric | Description |
| :--- | :--- |
| `circuit_state` | Circuit breaker state (0=closed, 1=half_open, 2=open) |

## Prometheus Export

### export_prometheus_metrics()

Export all metrics in Prometheus exposition format:

```python
from skyspy_common.libacars import export_prometheus_metrics

prometheus_output = export_prometheus_metrics()
print(prometheus_output)
```

**Output format:**

```prometheus
# HELP libacars_uptime_seconds Time since metrics collector started
# TYPE libacars_uptime_seconds gauge
libacars_uptime_seconds 3600.00

# HELP libacars_decode_attempts_total Total count of decode_attempts
# TYPE libacars_decode_attempts_total counter
libacars_decode_attempts_total 10000

# HELP libacars_decode_success_total Total count of decode_success
# TYPE libacars_decode_success_total counter
libacars_decode_success_total 8500

# HELP libacars_decode_failures_total Total count of decode_failures
# TYPE libacars_decode_failures_total counter
libacars_decode_failures_total 500

# HELP libacars_cache_hits_total Total count of cache_hits
# TYPE libacars_cache_hits_total counter
libacars_cache_hits_total 2000

# HELP libacars_decode_duration_ms Duration of decode operations
# TYPE libacars_decode_duration_ms summary
libacars_decode_duration_ms{quantile="0"} 0.50
libacars_decode_duration_ms{quantile="1"} 150.00
libacars_decode_duration_ms_sum 85000.00
libacars_decode_duration_ms_count 8500

# HELP libacars_decode_avg_ms Average duration of decode
# TYPE libacars_decode_avg_ms gauge
libacars_decode_avg_ms 10.00

# HELP libacars_circuit_state Current value of circuit_state
# TYPE libacars_circuit_state gauge
libacars_circuit_state 0
```

### Flask Integration

```python
from flask import Flask, Response
from skyspy_common.libacars import export_prometheus_metrics

app = Flask(__name__)

@app.route('/metrics')
def metrics():
    return Response(
        export_prometheus_metrics(),
        mimetype='text/plain; charset=utf-8'
    )
```

### FastAPI Integration

```python
from fastapi import FastAPI
from fastapi.responses import PlainTextResponse
from skyspy_common.libacars import export_prometheus_metrics

app = FastAPI()

@app.get('/metrics')
async def metrics():
    return PlainTextResponse(
        content=export_prometheus_metrics(),
        media_type='text/plain; charset=utf-8'
    )
```

### Django Integration

```python
from django.http import HttpResponse
from skyspy_common.libacars import export_prometheus_metrics

def metrics_view(request):
    return HttpResponse(
        export_prometheus_metrics(),
        content_type='text/plain; charset=utf-8'
    )
```

## Health Checks

### get_health()

Get a simple health status for load balancers and orchestrators:

```python
from skyspy_common.libacars import get_health

health = get_health()
print(health)
# {
#     "healthy": True,
#     "available": True,
#     "backend": "cffi",
#     "circuit_state": "closed"
# }
```

**Health criteria:**
- `healthy`: True if library is available AND circuit is not OPEN
- `available`: True if library loaded successfully
- `backend`: "cffi", "ctypes", or "unavailable"
- `circuit_state`: "closed", "half_open", or "open"

### HealthChecker Class

For more comprehensive health checks:

```python
from skyspy_common.libacars import HealthChecker, is_available, get_circuit_breaker

checker = HealthChecker(
    error_rate_threshold=50.0,  # Unhealthy if > 50% errors
    min_checks_for_health=10,   # Need 10+ ops before meaningful
)

# Add custom health checks
checker.add_check(
    "library_available",
    lambda: (is_available(), "Library loaded" if is_available() else "Library not found")
)

checker.add_check(
    "circuit_breaker",
    lambda: (
        get_circuit_breaker().is_closed,
        f"Circuit {get_circuit_breaker().state.name}"
    )
)

# Run all checks
result = checker.check_health()
print(result)
# {
#     "healthy": True,
#     "timestamp": 1706400000.0,
#     "checks": {
#         "library_available": {"healthy": True, "message": "Library loaded"},
#         "circuit_breaker": {"healthy": True, "message": "Circuit CLOSED"}
#     }
# }
```

### Kubernetes Health Probes

```python
from flask import Flask, jsonify
from skyspy_common.libacars import get_health

app = Flask(__name__)

@app.route('/health/live')
def liveness():
    """Kubernetes liveness probe."""
    # Always healthy if process is running
    return jsonify({"status": "ok"}), 200

@app.route('/health/ready')
def readiness():
    """Kubernetes readiness probe."""
    health = get_health()
    if health["healthy"]:
        return jsonify(health), 200
    else:
        return jsonify(health), 503
```

## Statistics

### get_stats()

Get comprehensive operational statistics:

```python
from skyspy_common.libacars import get_stats

stats = get_stats()
print(stats)
```

**Output:**

```python
{
    # Availability
    "available": True,
    "disabled_env": False,
    "backend": "cffi",

    # Circuit breaker
    "circuit_state": "closed",

    # Cache
    "cache_enabled": True,
    "cache_size": 150,

    # Operation counts
    "total_calls": 10000,
    "successful": 8500,
    "failed": 500,
    "skipped": 1000,

    # Cache performance
    "cache_hits": 2000,
    "cache_misses": 6500,
    "cache_hit_rate": 23.53,

    # Error tracking
    "consecutive_errors": 0,

    # Performance
    "total_decode_time_ms": 85000.0,
    "avg_decode_time_ms": 10.0,
    "success_rate": 94.44,

    # Detailed circuit breaker stats
    "circuit_breaker": {
        "total_calls": 10000,
        "successful_calls": 8500,
        "failed_calls": 500,
        "rejected_calls": 0,
        "state_changes": 2,
        "current_state": "CLOSED",
        "time_in_current_state": 3600.0,
        "failure_counts": {"timeout": 300, "json": 150, "unknown": 50},
        "recovery_attempts": 1,
        "successful_recoveries": 1,
        "success_rate": 94.44
    },

    # Detailed cache stats
    "cache": {
        "hits": 2000,
        "misses": 6500,
        "evictions": 50,
        "expirations": 100,
        "current_size": 150,
        "max_size": 1000,
        "hit_rate": 23.53
    }
}
```

### TimingStats

Detailed timing statistics:

```python
from skyspy_common.libacars import get_metrics_collector

metrics = get_metrics_collector()
timing = metrics.get_timing("decode")

if timing:
    print(f"Count: {timing['count']}")
    print(f"Total: {timing['total_ms']:.2f}ms")
    print(f"Average: {timing['avg_ms']:.2f}ms")
    print(f"Min: {timing['min_ms']:.2f}ms")
    print(f"Max: {timing['max_ms']:.2f}ms")
    print(f"Last: {timing['last_ms']:.2f}ms")
```

### CounterStats

Counter values with metadata:

```python
metrics = get_metrics_collector()

# Get counter value
value = metrics.get_counter("decode_success")
print(f"Successful decodes: {value}")

# Increment counter
new_value = metrics.increment("custom_counter", delta=5)
```

## Resetting Metrics

### reset_stats()

Reset decode statistics (keeps cache entries):

```python
from skyspy_common.libacars import reset_stats

reset_stats()
# Stats counters are now zero
# Cache entries are preserved
```

### reset_metrics()

Reset the entire metrics collector:

```python
from skyspy_common.libacars import reset_metrics

reset_metrics()
# All counters, timings, and gauges reset
# Uptime restarts from zero
```

## Monitoring Dashboard Example

```python
from skyspy_common.libacars import get_stats, get_health, get_circuit_breaker

def print_dashboard():
    stats = get_stats()
    health = get_health()
    breaker = get_circuit_breaker()
    analysis = breaker.get_failure_analysis()

    print("=" * 50)
    print("LIBACARS MONITORING DASHBOARD")
    print("=" * 50)

    # Health Status
    status = "HEALTHY" if health["healthy"] else "UNHEALTHY"
    print(f"\nStatus: {status}")
    print(f"Backend: {stats['backend']}")
    print(f"Circuit: {stats['circuit_state']}")

    # Performance
    print(f"\n--- Performance ---")
    print(f"Total calls: {stats['total_calls']:,}")
    print(f"Success rate: {stats['success_rate']:.1f}%")
    print(f"Avg decode time: {stats['avg_decode_time_ms']:.2f}ms")

    # Cache
    print(f"\n--- Cache ---")
    print(f"Hit rate: {stats['cache_hit_rate']:.1f}%")
    print(f"Size: {stats['cache_size']}/{stats['cache']['max_size']}")

    # Errors
    if stats['failed'] > 0:
        print(f"\n--- Errors ---")
        print(f"Failed: {stats['failed']}")
        print(f"Likely cause: {analysis.get('likely_cause', 'N/A')}")
        print(f"Recommendation: {analysis.get('recommendation', 'N/A')}")

    print("=" * 50)

# Run periodically
print_dashboard()
```

## Alerting Integration

### Example: Alert on High Error Rate

```python
from skyspy_common.libacars import get_stats

def check_alerts():
    stats = get_stats()
    alerts = []

    # High error rate
    if stats['success_rate'] < 90:
        alerts.append({
            "level": "warning",
            "message": f"Low success rate: {stats['success_rate']:.1f}%"
        })

    # Circuit breaker open
    if stats['circuit_state'] == "open":
        alerts.append({
            "level": "critical",
            "message": "Circuit breaker is OPEN"
        })

    # High consecutive errors
    if stats['consecutive_errors'] > 3:
        alerts.append({
            "level": "warning",
            "message": f"Consecutive errors: {stats['consecutive_errors']}"
        })

    return alerts
```

## Next Steps

- **API Reference** - Complete function documentation. [Learn more →](/docs/libacars/api)
- **Configuration** - Tune metrics and monitoring settings. [Learn more →](/docs/libacars/configuration)
