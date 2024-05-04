# pihole6_exporter - A prometheus-style exporter for Pi-hole v6

This is a basic prometheus exporter for the new API in Pi-hole version 6.

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
* The prometheus_client Python library (on Pi OS: `sudo apt-get install python3-prometheus-client`)

## Installation

* Copy the exporter itself over to `/usr/local/bin`
* Copy the systemd service file over to `/etc/systemd/system/` (or anywhere systemd will find it)
    * Modify the `Exec=` line with any command line args (like a key) as needed.  Currently there is no config file.  
* `systemctl start pihole6-exporter` to start the exporter.
* `systemctl enable pihole6-exporter` to have it start automatically.

## Metrics Provided

| Metric | Description |
|--------|-------------|
| `pihole_query_by_type`

# HELP pihole_query_by_type Count of queries by type (24h)
# TYPE pihole_query_by_type gauge
pihole_query_by_type
# HELP pihole_query_by_status Count of queries by status over 24h
# TYPE pihole_query_by_status gauge
pihole_query_by_status
# HELP pihole_query_replies Count of replies by type over 24h
# TYPE pihole_query_replies gauge
pihole_query_replies
# HELP pihole_query_count Query counts by category, 24h
# TYPE pihole_query_count gauge
pihole_query_count
# HELP pihole_client_count Total/active client counts
# TYPE pihole_client_count gauge
pihole_client_count
# HELP pihole_domains_being_blocked Number of domains on current blocklist
# TYPE pihole_domains_being_blocked gauge
pihole_domains_being_blocked 128537.0
# HELP pihole_query_upstream_count Total query upstream counts (24h)
# TYPE pihole_query_upstream_count gauge
pihole_query_upstream_count
