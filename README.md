# pihole6_exporter: A Prometheus-style Exporter for Pi-hole ver. 6

This is a basic prometheus exporter for the new API in Pi-hole version 6, currently in beta.

[There is a Grafana Dashboard as well!](https://grafana.com/grafana/dashboards/21043-pi-hole-ver6-stats/)

**UPDATE:** Also included here (see below) is a query logger meant to run as a systemd timer every minute.  It generates digestible logs out of the API's `/queries` call!

## pihole6_exporter

### Running

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

### Requirements

* Python 3
* The prometheus_client Python library (on Pi/Debian-like OSes: `sudo apt-get install python3-prometheus-client`)

### Installation

* Copy the exporter itself over to `/usr/local/bin`
* Copy the systemd service file over to `/etc/systemd/system/` (or anywhere systemd will find it)
    * Modify the `Exec=` line with any command line args (like a key) as needed.  Currently there is no config file.  
* `systemctl start pihole6-exporter` to start the exporter.
* `systemctl enable pihole6-exporter` to have it start automatically.

### Scraping with Alloy

Grafana Alloy is a popular way of scraping metrics off Raspberry Pi.  If you're using Alloy, add something like this to the end of your `/etc/alloy/config.alloy` file:

```
prometheus.scrape "pihole6_exporter_scraper" {
        targets    = [{"__address__" = "127.0.0.1:9666", "instance" = "cairon", "job" = "pihole6_exporter"}]
        forward_to = [prometheus.remote_write.metrics_service.receiver]
}
```

Then restart the alloy service (`systemctl restart alloy`).

### Metrics Provided

All metrics derive from the `/stats/summary`, `/stats/upstreams` and `/queries` API calls, minus a few stats which can be derived from these metrics (e.g. the % of domains blocked).

These are per-24h metrics provided by the API, like the ones used in the stats on the web admin dashboard.

| Metric | Description | Labels |
|--------|-------------|--------|
| `pihole_query_by_type` | Count of queries by type over the last 24h | `query_type` (A, AAAA, SOA, etc.) |
| `pihole_query_by_status` | Count of queries by status over the last 24h | `query_status` (FORWARDED, CACHE, GRAVITY, etc.) |
| `pihole_query_replies` | Count of query replies by type over the last 24h | `reply_type` (CNAME, IP, NXDOMAIN, etc.) |
| `pihole_query_count` | Query count totals over last 24h | `category` (total, blocked, unique, forwarded, cached) |
| `pihole_client_count` | Count of total/active clients | `category` (active, total) |
| `pihole_domains_being_blocked` | Number of domains being blocked | *None* |
| `pihole_query_upstream_count` | Counts of total queries in the last 24h by upstream destination | `ip`, `port`, `name` |

These are per-1m metrics.  These can be aggregated over time periods other than just 24h, and in various ways to derive the same stats as above and more.

| Metric | Description | Labels |
|--------|-------------|--------|
| `pihole_query_type_1m` | Per-1m count of queries by type | `query_type` |
| `pihole_query_status_1m` | Per-1m count of queries by status | `query_status` |
| `pihole_query_upstream_1m` | Per-1m count of queries by upstream destination | `query_upstream` |
| `pihole_query_reply_1m` | Per-1m count of queries by reply | `query_reply` |
| `pihole_query_client_1m` | Per-1m count of queries by client |  `query_client` |

## pihole6_query_logger

### Why? Pihole/dnsmaq/ftl/etc. make logs already.

Something that frustrates me from a data-analysis standpoint is that it is difficult to connect the logs for the initial query request with the upstream result.  The API `/queries` call has this information correlated together per-query...  I'm providing these metrics here summed up by one aspect only (type, status, upstream, reply, client), but they can't be put together.  The cardinality would be horrible: hundreds of thousands of combinations would generate far too many timeseries for grafana cloud to put up with without charging you up the wazoo for it.  Metrics with that many combinations of labels are not good for a happy Prometheus.

So the next best thing is to bring in the API JSON response in as a log file.  Grafana cloud allows a huge amount of logs short-term (30d with pro, 14d without), so holding the raw queries themsleves is ok.  Then, we can extract fields out of this very easily in Loki on the fly, and correlate queries by multiple aspects.  I could make a graph of query typer *per client*, and even tell you which of those were cached responses, for example.  Because those gigantic responses aren't being saved, they're ok as long as you don't query them too heavily.

This script calls the API for queries in the last whole minute, converts them into timestamped, single-line JSON lines, and appends them to a log file.

**Bottom line: having all this information about every query as a log can let you analyze your queries in ways the metrics can't provide.**

### Installation

(Sorry it's so manual!  Maybe I'll make deb packages...)

* The `pihole6_query_logger` script itself should be placed somewhere like `/usr/local/bin`.  The systemd file I provide assumes it is.
* The `pihole6_query_logger.service` file should go in `/etc/systemd/system` or your host's systemd directory.  Starting this service triggers a single run of the logger.
    * This unit file "`Wants`" the alloy target.  If you're not using Grafana Alloy, remove that line.
* The `pihole6_query_logger.timer` file should also go in that systemd directory.  This calls the service every minute.
* As root, run `systemd enable pihole6_query_logger` to start the timer.  Monitor `/var/log/pihole_api/api.log` for updates in a minute or so to confirm it's working.
* The `pihole_api` file goes in `/etc/logrotate.d/`.  `systemctl restart logrotate` for that to take effect.  This keeps 3 days of the queries before rotating them out so the log doesn't grow out of control.
* Configure your log ingester to use this log!
    * If you're using Alloy, the `config.alloy.snippet` file here is the configuration I use to extract and use the log's timestamp as Loki's timestamp (because the ingest time will be off by up to a minute), and the JSON as the log content (so you can easily parse it with the json parser straightaway).
    * If you use promtail, check out that alloy config anyway.  The loki config.yml is structured differently but the pipeline stages I use here are about what you'd do there.

## Questions/Comments?

Please open a git issue here.  I can't promise a particular reponse time but I'll do my best.
