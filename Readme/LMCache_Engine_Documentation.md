# LMCache Engine Documentation

------------------------------------------------------------------------

# 1. Configuration System (`config.py`) --- The Control Plane

The `config.py` file is the central backbone of the LMCache Engine.\
It defines, loads, validates, and finalizes every runtime parameter that
governs KV cache behavior --- including memory limits, storage tiers,
feature toggles, and backend configuration.

This file acts as the **control plane** of the engine.

------------------------------------------------------------------------

## 1.1 Architectural Pattern --- Dynamic Factory Model

Instead of defining a static configuration class, `config.py`
dynamically constructs `LMCacheEngineConfig` using:

    create_config_class (from config_base.py)

### Why this design?

-   Automatic type conversion (e.g., `"true"` → `True`, `"10GB"` →
    parsed size)
-   Consistent behavior across modules
-   Centralized validation logic
-   Reduced boilerplate for future config additions

### Backward Compatibility

`_CONFIG_ALIASES` ensures older variable names remain functional.

Example: - `enable_xpyd` (legacy) → mapped to → `enable_pd` (current)

This prevents breaking changes across releases.

------------------------------------------------------------------------

## 1.2 Configuration Precedence (Hierarchical Loading)

LMCache resolves configuration values using strict priority order.

If a parameter exists in multiple places, the highest priority source
overrides the others:

1.  **Explicit Python Overrides**
2.  **Environment Variables (`LMCACHE_*`)**
3.  **YAML Configuration File**
4.  **Default Definitions (`_CONFIG_DEFINITIONS`)**

### Example

If:

-   YAML sets `max_local_cpu_size = 8GB`
-   Environment sets `LMCACHE_MAX_LOCAL_CPU_SIZE=16GB`

→ Final value will be **16GB**.

This deterministic override model eliminates ambiguity in production
deployments.

------------------------------------------------------------------------

## 1.3 Core Lifecycle Functions

### `load_engine_config_with_overrides()`

Primary entry point. - Aggregates all config sources - Resolves
aliases - Applies type conversion - Runs validation - Returns finalized
`LMCacheEngineConfig`

------------------------------------------------------------------------

### `_validate_config()`

Sanity enforcement layer.

Prevents: - Negative cache sizes - Unsupported feature combinations -
Hardware-incompatible settings - Invalid storage paths

Ensures the engine never boots in an unstable state.

------------------------------------------------------------------------

### `_log_config()`

Outputs active configuration at runtime. Used for: - Debugging
deployments - Verifying environment override behavior - Production
diagnostics

------------------------------------------------------------------------

## 1.4 Environment Parsing Utilities

Since environment variables are strings, `config_base.py` provides
robust converters:

### `_to_bool()`

Safely parses: - `"1"`, `"true"`, `"yes"` → `True` - `"0"`, `"false"` →
`False`

------------------------------------------------------------------------

### `_parse_local_disk()`

Validates and normalizes disk paths for cold-tier storage.

------------------------------------------------------------------------

### `_apply_env_converter_safely()`

Protects against malformed environment variables. Prevents crashes from
invalid formats.

------------------------------------------------------------------------

## 1.5 Operational Importance

`config.py` is the **single source of truth** for:

-   CPU hot cache size
-   Disk cold tier location
-   Promotion/demotion toggles
-   Feature flags
-   Performance behavior

If LMCache behaves unexpectedly, configuration resolution should be the
first diagnostic checkpoint.

------------------------------------------------------------------------

# End of Document
