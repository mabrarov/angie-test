# Testing [Angie](https://angie.software/angie/)

Prerequisites:

1. Docker 1.12.3+
1. Docker Compose

Start containers:

```bash
docker-compose up -d
```

Add one more backend server:

```bash
docker-compose up -d --scale backend=2 --no-recreate
```

Check output of balancer:

```bash
curl -s http://localhost
```

Output of balancer should look like:

```html
<!DOCTYPE html>
<html lang="en-US">
<head>
<title>Backend</title>
</head>
<body>
hostname
</body>
</html>
```

where `hostname` is the name of backend host which actually served request.

Ensure that if output is requested multiple times then different `hostname` is returned.

Check Prometheus metrics of balancer:

```bash
curl -s http://localhost:81
```

Output of balancer Prometheus endpoint should look like:

```text
...
angie_http_upstreams_peers_state{upstream="backend",peer="172.18.0.3:8080"} 1
angie_http_upstreams_peers_state{upstream="backend",peer="172.18.0.4:8080"} 1
angie_http_upstreams_peers_selected_current{upstream="backend",peer="172.18.0.3:8080"} 0
angie_http_upstreams_peers_selected_current{upstream="backend",peer="172.18.0.4:8080"} 0
angie_http_upstreams_peers_selected_total{upstream="backend",peer="172.18.0.3:8080"} 4
angie_http_upstreams_peers_selected_total{upstream="backend",peer="172.18.0.4:8080"} 1
angie_http_upstreams_peers_responses{upstream="backend",peer="172.18.0.3:8080",code="200"} 4
angie_http_upstreams_peers_responses{upstream="backend",peer="172.18.0.4:8080",code="200"} 1
angie_http_upstreams_peers_data_sent{upstream="backend",peer="172.18.0.3:8080"} 356
angie_http_upstreams_peers_data_sent{upstream="backend",peer="172.18.0.4:8080"} 89
angie_http_upstreams_peers_data_received{upstream="backend",peer="172.18.0.3:8080"} 1012
angie_http_upstreams_peers_data_received{upstream="backend",peer="172.18.0.4:8080"} 253
angie_http_upstreams_peers_health_fails{upstream="backend",peer="172.18.0.3:8080"} 0
angie_http_upstreams_peers_health_fails{upstream="backend",peer="172.18.0.4:8080"} 0
angie_http_upstreams_peers_health_unavailable{upstream="backend",peer="172.18.0.3:8080"} 0
angie_http_upstreams_peers_health_unavailable{upstream="backend",peer="172.18.0.4:8080"} 0
angie_http_upstreams_peers_health_downtime{upstream="backend",peer="172.18.0.3:8080"} 0
angie_http_upstreams_peers_health_downtime{upstream="backend",peer="172.18.0.4:8080"} 0
angie_http_upstreams_keepalive{upstream="backend"} 0
...
```

Stop containers:

```bash
docker-compose stop
```

Cleanup:

```bash
docker-compose down -v -t 0 && \
docker rmi -f abrarov/angie-test-backend && \
docker rmi -f abrarov/angie-test-balancer
```
