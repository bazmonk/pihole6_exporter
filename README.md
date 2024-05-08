# pihole6_exporter: A Prometheus-style Exporter for Pi-hole ver. 6

This is a basic prometheus exporter for the new API in Pi-hole version 6, currently in beta.

[There is a Grafana Dashboard as well!](https://grafana.com/grafana/dashboards/21043-pi-hole-ver6-stats/)


## Running

```
usage: pihole6_exporter [-h] [-H HOST] [-p PORT] [-k KEY]

Prometheus exporter for Pi-hole version 6+

optional arguments:
  -h, --help            show this help message and exit
  -H HOST, --host HOST  hostname/ip of pihole instance (default localhost)
  -p PORT, --port PORT  port to expose for scraping (default 9666)
  -k KEY, --key KEY     authentication token
```

If using locally and you have the `Local clients need to authenticate to access the API` option un-selected, a key is not necessary.  This key is the "app password", not the session ID that is created with it.

The session ID should stay active as long as it is used at least every 5 minutes.  A typical scrape interval is 1m.  Currently, if the session ID expires, a restart of the exporter is necessary.

## Requirements

* Python 3
* The prometheus_client Python library (on Pi/Debian-like OSes: `sudo apt-get install python3-prometheus-client`)

## Installation

* Copy the exporter itself over to `/usr/local/bin`
* Copy the systemd service file over to `/etc/systemd/system/` (or anywhere systemd will find it)
    * Modify the `Exec=` line with any command line args (like a key) as needed.  Currently there is no config file.  
* `systemctl start pihole6-exporter` to start the exporter.
* `systemctl enable pihole6-exporter` to have it start automatically.

## Metrics Provided

All metrics derive from the `/stats/summary`, `/stats/upstreams` and `/queries` API calls, minus a few stats which can be derived from these metrics (e.g. the % of domains blocked).

| Metric | Description | Labels |
|--------|-------------|--------|
| `pihole_query_by_type` | Count of queries by type over the last 24h | `query_type` (A, AAAA, SOA, etc.) |
| `pihole_query_by_status` | Count of queries by status over the last 24h | `query_status` (FORWARDED, CACHE, GRAVITY, etc.) |
| `pihole_query_replies` | Count of query replies by type over the last 24h | `reply_type` (CNAME, IP, NXDOMAIN, etc.) |
| `pihole_query_count` | Query count totals over last 24h | `category` (total, blocked, unique, forwarded, cached) |
| `pihole_client_count` | Count of total/active clients | `category` (active, total) |
| `pihole_domains_being_blocked` | Number of domains being blocked | *None* |
| `pihole_query_upstream_count` | Counts of total queries in the last 24h by upstream destination | `ip`, `port`, `name` |
| `pihole_query_type_1m` | Per-1m count of queries by type | `query_type` |
| `pihole_query_status_1m` | Per-1m count of queries by status | `query_status` |
| `pihole_query_upstream_1m` | Per-1m count of queries by upstream destination | `query_upstream` |
| `pihole_query_reply_1m` | Per-1m count of queries by reply | `query_reply` |
| `pihole_query_client_1m` | Per-1m count of queries by client |  `query_client` |

## Questions/Comments?

Please open a git issue here.  I can't promise a particular reponse time but I'll do my best.
