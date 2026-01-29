---
title: "API Reference"
slug: "libacars-api"
excerpt: "Complete reference for libacars decoding functions and data types"
hidden: false
---

This page documents all public functions, classes, and data types in the libacars Python binding.

## Core Decoding Functions

### decode_acars_apps()

Decode ACARS application-layer message content and return structured JSON data.

```python
def decode_acars_apps(
    label: str,
    text: str,
    direction: MsgDir = MsgDir.UNKNOWN,
    *,
    reg: Optional[str] = None,
    reassembly_ctx: Optional[ReassemblyContext] = None,
    timestamp: Optional[float] = None,
    use_cache: bool = True,
    raise_on_error: bool = False,
    timeout: Optional[float] = None
) -> Optional[dict]:
```

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `label` | `str` | required | ACARS message label (e.g., "H1", "SA", "Q0") |
| `text` | `str` | required | Message text content to decode |
| `direction` | `MsgDir` | `UNKNOWN` | Message direction (air-to-ground or ground-to-air) |
| `reg` | `str` | `None` | Aircraft registration (for reassembly) |
| `reassembly_ctx` | `ReassemblyContext` | `None` | Context for multi-part message reassembly |
| `timestamp` | `float` | `None` | Unix timestamp (for reassembly) |
| `use_cache` | `bool` | `True` | Whether to use the decode cache |
| `raise_on_error` | `bool` | `False` | Raise exceptions instead of returning None |
| `timeout` | `float` | `None` | Decode timeout (for API compatibility) |

**Returns:** `dict` with decoded message structure, or `None` if decoding failed.

**Example:**

```python
from skyspy_common.libacars import decode_acars_apps, MsgDir

result = decode_acars_apps(
    label="H1",
    text="/BOMASYA.ADS.VT-ANE072501A095CA95C1...",
    direction=MsgDir.AIR2GND,
)

if result:
    # Access decoded fields
    msg_type = result.get("type")
    payload = result.get("payload")
```

### decode_acars_apps_text()

Decode ACARS message and return human-readable formatted text.

```python
def decode_acars_apps_text(
    label: str,
    text: str,
    direction: MsgDir = MsgDir.UNKNOWN,
    *,
    raise_on_error: bool = False,
    timeout: Optional[float] = None
) -> Optional[str]:
```

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `label` | `str` | required | ACARS message label |
| `text` | `str` | required | Message text content |
| `direction` | `MsgDir` | `UNKNOWN` | Message direction |
| `raise_on_error` | `bool` | `False` | Raise exceptions instead of returning None |
| `timeout` | `float` | `None` | Decode timeout |

**Returns:** Formatted text string, or `None` if decoding failed.

**Example:**

```python
from skyspy_common.libacars import decode_acars_apps_text, MsgDir

text_output = decode_acars_apps_text(
    label="H1",
    text="/BOMASYA.ADS.VT-ANE072501...",
    direction=MsgDir.AIR2GND,
)

if text_output:
    print(text_output)
    # ADS-C:
    #   Report type: periodic
    #   Latitude: 12.345
    #   Longitude: 67.890
    #   ...
```

### extract_sublabel_mfi()

Extract sublabel and MFI (Message Function Identifier) from ACARS messages.

```python
def extract_sublabel_mfi(
    label: str,
    text: str,
    direction: MsgDir = MsgDir.UNKNOWN
) -> tuple[Optional[str], Optional[str], int]:
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `label` | `str` | ACARS message label |
| `text` | `str` | Message text content |
| `direction` | `MsgDir` | Message direction |

**Returns:** Tuple of `(sublabel, mfi, bytes_consumed)` or `(None, None, 0)` if extraction failed.

**Example:**

```python
from skyspy_common.libacars import extract_sublabel_mfi, MsgDir

sublabel, mfi, consumed = extract_sublabel_mfi(
    label="H1",
    text="- #MDADS...",
    direction=MsgDir.AIR2GND,
)

print(f"Sublabel: {sublabel}, MFI: {mfi}, Consumed: {consumed} bytes")
```

## Async Variants

### decode_acars_apps_async()

Async version of `decode_acars_apps()` using a thread pool executor.

```python
async def decode_acars_apps_async(
    label: str,
    text: str,
    direction: MsgDir = MsgDir.UNKNOWN,
    timeout: Optional[float] = None,
    **kwargs
) -> Optional[dict]:
```

**Example:**

```python
import asyncio
from skyspy_common.libacars import decode_acars_apps_async, MsgDir

async def process_message():
    result = await decode_acars_apps_async(
        label="H1",
        text="...",
        direction=MsgDir.AIR2GND,
        timeout=5.0,
    )
    return result

# In an async context
result = await process_message()

# Or run directly
result = asyncio.run(process_message())
```

### decode_acars_apps_text_async()

Async version of `decode_acars_apps_text()`.

```python
async def decode_acars_apps_text_async(
    label: str,
    text: str,
    direction: MsgDir = MsgDir.UNKNOWN,
    timeout: Optional[float] = None,
    **kwargs
) -> Optional[str]:
```

## Batch Processing

### decode_batch()

Process multiple messages efficiently in a single call.

```python
def decode_batch(
    messages: List[BatchMessage],
    output_format: str = "json",
    use_cache: bool = True
) -> List[BatchResult]:
```

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `messages` | `List[BatchMessage]` | required | List of messages to decode |
| `output_format` | `str` | `"json"` | Output format: `"json"` or `"text"` |
| `use_cache` | `bool` | `True` | Whether to use decode cache |

**Returns:** List of `BatchResult` objects in the same order as input.

**Example:**

```python
from skyspy_common.libacars import decode_batch, BatchMessage, MsgDir

messages = [
    BatchMessage(
        label="H1",
        text="...",
        direction=MsgDir.AIR2GND,
        id="msg-001",
        reg="N12345",
        timestamp=1706400000.0,
    ),
    BatchMessage(
        label="SA",
        text="...",
        direction=MsgDir.AIR2GND,
        id="msg-002",
    ),
]

results = decode_batch(messages)

for result in results:
    if result.success:
        print(f"{result.id}: Decoded in {result.decode_time_ms:.2f}ms")
    else:
        print(f"{result.id}: Failed - {result.error}")
```

### decode_batch_async()

Async batch processing with concurrency control.

```python
async def decode_batch_async(
    messages: List[BatchMessage],
    output_format: str = "json",
    max_concurrency: int = 4
) -> List[BatchResult]:
```

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `messages` | `List[BatchMessage]` | required | List of messages to decode |
| `output_format` | `str` | `"json"` | Output format |
| `max_concurrency` | `int` | `4` | Maximum concurrent decode operations |

**Example:**

```python
import asyncio
from skyspy_common.libacars import decode_batch_async, BatchMessage

async def process_batch():
    messages = [BatchMessage(label="H1", text="...") for _ in range(100)]
    results = await decode_batch_async(messages, max_concurrency=8)
    return results
```

## Data Types

### MsgDir

Enumeration for ACARS message direction.

```python
class MsgDir(IntEnum):
    UNKNOWN = 0   # Direction not specified
    GND2AIR = 1   # Uplink (Ground to Air)
    AIR2GND = 2   # Downlink (Air to Ground)
```

**Example:**

```python
from skyspy_common.libacars import MsgDir

# Use enum values
direction = MsgDir.AIR2GND
print(f"Direction: {direction.name} ({direction.value})")  # AIR2GND (2)

# Convert from integer
direction = MsgDir(2)  # MsgDir.AIR2GND
```

### BatchMessage

Input data class for batch processing.

```python
@dataclass
class BatchMessage:
    label: str                          # ACARS message label
    text: str                           # Message text content
    direction: MsgDir = MsgDir.UNKNOWN  # Message direction
    id: Optional[str] = None            # Optional message identifier
    reg: Optional[str] = None           # Aircraft registration
    timestamp: Optional[float] = None   # Unix timestamp
```

### BatchResult

Result from batch processing operations.

```python
@dataclass
class BatchResult:
    id: Optional[str]                   # Message identifier (from BatchMessage)
    success: bool                       # Whether decode succeeded
    data: Optional[Union[dict, str]]    # Decoded data (dict for JSON, str for text)
    error: Optional[str] = None         # Error message if failed
    cached: bool = False                # Whether result came from cache
    decode_time_ms: float = 0.0         # Time taken to decode
```

### DecodeResult

Detailed result from decode operations.

```python
@dataclass
class DecodeResult:
    success: bool                       # Whether decode succeeded
    data: Optional[Union[dict, str]]    # Decoded data
    decode_time_ms: float = 0.0         # Decode duration
    error: Optional[str] = None         # Error message if failed
    cached: bool = False                # Whether from cache
```

### ReassemblyContext

Context manager for reassembling multi-part ACARS messages.

```python
class ReassemblyContext:
    def __init__(self): ...
    def destroy(self) -> None: ...
    def __enter__(self) -> 'ReassemblyContext': ...
    def __exit__(self, ...): ...
```

**Example:**

```python
from skyspy_common.libacars import decode_acars_apps, ReassemblyContext, MsgDir
import time

# Use as context manager for automatic cleanup
with ReassemblyContext() as ctx:
    # Process multi-part message fragments
    result = decode_acars_apps(
        label="H1",
        text="...",
        direction=MsgDir.AIR2GND,
        reg="N12345",
        reassembly_ctx=ctx,
        timestamp=time.time(),
    )
```

### LibacarsConfig

Runtime configuration interface for the libacars library.

```python
class LibacarsConfig:
    @staticmethod
    def set(name: str, value: Union[bool, int, str]) -> bool:
        """Set a libacars configuration option."""
```

**Example:**

```python
from skyspy_common.libacars import LibacarsConfig

# Set configuration options
LibacarsConfig.set("decode_verbose", True)
LibacarsConfig.set("max_recursion", 10)
```

### LibacarsStats

Statistics from decode operations.

```python
@dataclass
class LibacarsStats:
    total_calls: int                    # Total decode attempts
    successful: int                     # Successful decodes
    failed: int                         # Failed decodes
    skipped: int                        # Skipped (disabled/circuit open)
    cache_hits: int                     # Results from cache
    cache_misses: int                   # Cache misses
    total_decode_time_ms: float         # Total time spent decoding
    consecutive_errors: int             # Current error streak

    # Computed properties
    avg_decode_time_ms: float           # Average decode time
    success_rate: float                 # Success percentage
    cache_hit_rate: float               # Cache hit percentage

    def to_dict(self) -> dict: ...
```

## State Management

### init_binding()

Initialize the libacars binding (loads the native library).

```python
def init_binding() -> None:
```

### is_available()

Check if libacars is available and ready for use.

```python
def is_available() -> bool:
```

### get_backend()

Get the name of the active FFI backend.

```python
def get_backend() -> str:
    """Returns: 'cffi', 'ctypes', or 'unavailable'"""
```

### get_stats()

Get comprehensive statistics about decode operations.

```python
def get_stats() -> dict:
```

**Returns:**

```python
{
    "available": True,
    "disabled_env": False,
    "backend": "cffi",
    "circuit_state": "closed",
    "cache_enabled": True,
    "cache_size": 150,
    "total_calls": 1000,
    "successful": 850,
    "failed": 50,
    "skipped": 100,
    "cache_hits": 200,
    "cache_misses": 650,
    "consecutive_errors": 0,
    "total_decode_time_ms": 5432.1,
    "avg_decode_time_ms": 8.36,
    "success_rate": 94.44,
    "cache_hit_rate": 23.53,
    "circuit_breaker": {...},
    "cache": {...}
}
```

### reset_stats()

Reset decode statistics (does not clear cache).

```python
def reset_stats() -> None:
```

### reset_error_state()

Reset error counters and circuit breaker state.

```python
def reset_error_state() -> None:
```

### shutdown()

Gracefully shutdown the binding (closes thread pool).

```python
def shutdown() -> None:
```

### get_health()

Get health status for monitoring.

```python
def get_health() -> dict:
```

**Returns:**

```python
{
    "healthy": True,
    "available": True,
    "backend": "cffi",
    "circuit_state": "closed"
}
```

### export_prometheus_metrics()

Export all metrics in Prometheus exposition format.

```python
def export_prometheus_metrics() -> str:
```

### LIBACARS_DISABLED

Boolean constant indicating if libacars is disabled via environment.

```python
LIBACARS_DISABLED: bool
```

## Next Steps

- **Configuration** - Configure caching, timeouts, and behavior. [Learn more →](/docs/libacars/configuration)
- **Error Handling** - Handle exceptions and validation errors. [Learn more →](/docs/libacars/error-handling)
