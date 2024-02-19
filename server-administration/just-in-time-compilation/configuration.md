# 32.3. Configuration

The configuration variable [jit](https://www.postgresql.org/docs/15/runtime-config-query.html#GUC-JIT) determines whether JIT compilation is enabled or disabled. If it is enabled, the configuration variables [jit\_above\_cost](https://www.postgresql.org/docs/15/runtime-config-query.html#GUC-JIT-ABOVE-COST), [jit\_inline\_above\_cost](https://www.postgresql.org/docs/15/runtime-config-query.html#GUC-JIT-INLINE-ABOVE-COST), and [jit\_optimize\_above\_cost](https://www.postgresql.org/docs/15/runtime-config-query.html#GUC-JIT-OPTIMIZE-ABOVE-COST) determine whether JIT compilation is performed for a query, and how much effort is spent doing so.

[jit\_provider](https://www.postgresql.org/docs/15/runtime-config-client.html#GUC-JIT-PROVIDER) determines which JIT implementation is used. It is rarely required to be changed. See [Section 32.4.2](https://www.postgresql.org/docs/15/jit-extensibility.html#JIT-PLUGGABLE).

For development and debugging purposes a few additional configuration parameters exist, as described in [Section 20.17](https://www.postgresql.org/docs/15/runtime-config-developer.html).
