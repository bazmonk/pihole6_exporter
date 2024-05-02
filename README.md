# pihole6_exporter - A prometheus-style exporter for Pi-hole v6

This is a basic prometheus exporter for the new API in Pi-hole version 6.

Right now it's very basic, just enough to work.  I'm not sure how much I'll put into polishing it up for general use just yet.

* No config file.
* Assumes the API is available at localhost:443.
* Assumes you don't have the option to require a token from local users selected in Pi-hole.
* Makes no logs.
* All metrics start with "pihole_".

My initial aim is to parse out just the summary and upstream API calls:

* /stats/summary
* /stats/upstreams

...And create a sample grafana dashboard that utilizes all of it.
