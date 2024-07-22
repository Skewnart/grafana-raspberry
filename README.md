# WIP

# Grafana / Raspberry

**Home Grafana** (*Grafana / Loki*) implementation for **Raspberry** using **Reverse Proxy** and **HTTPS** to access it through internet.

It includes **basic dashboards** to monitor the Raspberry itself and containers (*Grafana / Prometheus / Node exporter / cadvisor*) allowing you to customize everything.

And also dashboards for sytem logging (*Promtail / Loki*) with default configuration. (Behind need is to log IOT workflows and create some alerts)

## Installation

### SSL Certificate

- Add your subdomain (here called **`YOUR-SUBDOMAIN`**) to your domain's **DNS redirections**, pointing your home's public IP.
- On your router, **forward classic 80 / 443** ports to your Raspberry

In the project directory :

```bash
$ docker compose -f certbot/docker-compose.yml run --service-ports --rm certbot certonly --standalone -d YOUR-SUBDOMAIN
```

*`--standalone`* option will allow certbot to create a standalone webserver (using port 80) just to test and validate subdomain.

Your certificate keys are now available in `/etc/lets-encrypt/live/YOUR-SUBDOMAIN`.
