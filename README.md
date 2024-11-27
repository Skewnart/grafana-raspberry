# WIP

# Grafana / Raspberry

**Home Grafana** (*Grafana / Loki*) implementation for **Raspberry** using **Reverse Proxy** and **HTTPS** to access it through internet.

It includes **basic dashboards** to monitor the Raspberry itself and containers (*Grafana / Prometheus / NodeExporter / cAdvisor*) allowing you to customize everything. And also dashboards for sytem logging (*Promtail / Loki*) with default configuration. (Behind need is to log IOT workflows and create some alerts, but these will be in another repo)

**Nginx** is separated in order to use yours if it is already (or later) configured for other web servers. You might want to use Nginx separately from the Grafana stack.

Finally, **certbot** is also separated because it's only used to *add* or *renew* certificate, and then stops.

## Installation steps

These steps will be performed in order : 

1. **SSL Certificate** (router configuration and SSL (outside server configuration))
2. **Monitoring (Grafana stack)** (inside server configuration)
3. **Nginx (Reverse proxy)** (link between inside and outside)

## Installation

### SSL Certificate

- Add your subdomain (here called **`YOUR-SUBDOMAIN`**) to your domain's **DNS redirections**, pointing your home's public IP.
- On your router, **forward classic 80 / 443** ports to your Raspberry

In the project directory :

```bash
cd certbot && ./add-domain && cd -
```

*`--standalone`* option will allow certbot to create a standalone webserver (using port 80) just to test and validate subdomain.

Your certificate keys are now available in `/etc/letsencrypt/live/YOUR-SUBDOMAIN`.

### Monitoring

The monitoring network will be in common with Nginx. Because both have separate docker-compose, the network is declared separately.

```bash
./start-network
```

Associated Monitoring docker-compose launches *all the Grafana stack* :

- **cAdvisor**      (expose containers metrics)
- **NodeExporter**  (expose system metrics)
- **Prometheus**    (pull and stores metrics for Grafana)
- **Promtail**      (tails logs and pushes to Loki)
- **Loki**          (store logs for Grafana)
- **Grafana**       (dashboards)

Change owner of Prometheus and Grafana directories :

```bash
sudo chown -R 472:472 monitoring/grafana/ \
&& sudo chown -R 65534:65534 monitoring/prometheus/ \
&& sudo chown -R 10001:10001 monitoring/loki/loki-data/
```

Now update your domain in the SSL volumes for Grafana in the associated docker-compose.

In the following command, like in the SSL part, replace **YOUR-SUBDOMAIN** with yours :

```bash
sed -i "s/subdomain/YOUR-SUBDOMAIN/g" ./monitoring/docker-compose.yml
```

At this time, you can launch associated docker-compose, just not access it through internet.
(unless you forward Grafana port (3000) in your router of course... But don't do it now, we're almost done 😉)

```bash
$ docker compose -f monitoring/docker-compose.yml up -d

[+] Running 6/6
 ✔ Container node-exporter  Started		1.3s
 ✔ Container loki           Started		1.0s
 ✔ Container cadvisor       Started		1.2s
 ✔ Container prometheus     Started		1.6s
 ✔ Container promtail       Started		1.1s
 ✔ Container grafana        Started		1.8s

$ curl localhost:3000

<a href="/login">Found</a>.

$ docker compose -f monitoring/docker-compose.yml down
```

Note it down : First time you access the Grafana interface, **username / passord** is **"admin" / "admin"**. You will be asked to change it.

### Nginx

Because the network is the same between docker-compose, you don't have to recreate it.

For the same reason as before, change subdomain by yours (here **YOUR-SUBDOMAIN**) in the *Grafana Nginx* configuration file :

```bash
sed -i "s/subdomain/YOUR-SUBDOMAIN/g" ./nginx/conf.d/grafana.conf
```

The configuration file is designed to respond to **80** and **443** ports :

- **Port 80 :** redirect to 443 port
- **Port 443 :** linked to SSL certificate, redirect to Grafana port 3000 upstream.

With network still up, launch both docker compose :

```bash
./start-containers
```

You are ready now, go to your **favorite internet browser** and access to your **brand new on-premise Grafana**.

## The end

To start it : `./start`

To stop it : `./stop`

To renew your certificate :

```bash
cd certbot && ./renew && cd -
```
