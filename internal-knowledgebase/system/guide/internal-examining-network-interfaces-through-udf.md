---
tool_name: internal-knowledgebase
doc_type: guide
category: system
title: "Internal - Examine network interfaces through UDF"
summary: "Use a Python UDF reading /proc/net/dev to estimate per-interface RX/TX throughput by node process."
---
# Internal - Examine network interfaces through UDF

## Purpose

Inspect per-interface traffic patterns from inside database execution context to identify external/internal traffic paths.

## UDF script

```sql
CREATE OR REPLACE PYTHON3 SCALAR SCRIPT netstat(iproc INT)
EMITS (
  iproc INT,
  interface VARCHAR(20),
  step INT,
  rx_bytes INT,
  tx_bytes INT
)
AS
from time import sleep

def run(ctx):
    get_network_bytes(ctx, 0)
    sleep(10)
    get_network_bytes(ctx, 1)

def get_network_bytes(meta, st):
    for line in open('/proc/net/dev', 'r'):
        d1 = line.split(':')
        if len(d1) > 1:
            data = d1[1].split()
            meta.emit(meta.iproc, d1[0], st, int(data[0]), int(data[8]))
/
```

## Example query

```sql
WITH k1 AS (
  SELECT netstat(iproc) FROM EXA_LOADAVG
)
SELECT
  iproc,
  interface,
  (MAX(rx_bytes)-MIN(rx_bytes))/10 AS rx_sec,
  (MAX(tx_bytes)-MIN(tx_bytes))/10 AS tx_sec
FROM k1
WHERE interface NOT LIKE '%lo%'
  AND interface NOT LIKE '%\_\_%'
GROUP BY 1,2
ORDER BY 2,1;
```

## Interpretation guidance

- High traffic on one interface may indicate client-facing/export traffic.
- High traffic on another interface may indicate internal node-to-node exchange.
- Large internal/external ratio can indicate compression inefficiency, prefetch behavior, or buffering.

## Notes

- This is an internal diagnostic pattern.
- Ensure UDF filesystem access policy allows reading `/proc/net/dev` in your environment.


