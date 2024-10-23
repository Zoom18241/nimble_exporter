# Nimble Exporter

Nimble Exporter collects and exposes Secure Reliable Transport (SRT) metrics from Nimble Server to Prometheus. It can collect the server connection stats and SRT receiver information.

## Table of Contents
- [Installation](#installation)
	- [Local installation & Docker](#local_installation_&_docker)
	- [Kubernetes Deployment](#kubernetes_deployment)
- [Usage](#usage)
- [Prometheus Configuration](#prometheus_configuration)
- [Grafana Dashboard](#grafana_dashboard)

## Installation

### Local Installation & Docker

Using Go:
```
git clone https://github.com/deckhouse/nimble_exporter
cd nimble_exporter
go build -o nimble_exporter main.go
```
Using Docker:
```
docker build github.com/deckhouse/nimble_exporter
docker run -d -p 9017:9017 nimble_exporter
```
### Kubernetes Deployment

Deploy Nimble Exporter to your Kubernetes cluster using the following configuration:
```
      - name: srt-exporter
        image: {{ $.Values.werf.image.exporter }}
        command:
        - /nimble_exporter
        - -auth_salt=590
        - -auth_hash=xxxx
        ports:
        - containerPort: 9017
          name: exporter
          protocol: TCP
        resources:
          {{ toYaml $.Values.resources.exporter |  nindent 10 }}
```
This config assumes that you are using Helm for templating, and the exporter image and resources are configured in your **Values.yaml** file.  

To deploy the Helm chart use the following command:
```helm install nimble-exporter ./chart-directory/```

## Usage

To run:
```
./nimble_exporter -listen ":9017" -nimble "http://127.0.0.1:8082" -auth_salt "your_nimble_salt" -auth_hash "your_nimble_hash"
```

The following command-line flags are supported:

-	-listen: The port which the exporter will listen to for Nimble data (default: :9017).
-	-nimble: The URL of the Nimble server (default: http://127.0.0.1:8082).
-	-auth_salt: Nimble authentication salt.
-	-auth_hash: Nimble authentication hash.

Metrics are available at the /metrics endpoint, which Prometheus can scrape for monitoring.

## Prometheus Configuration

To scrape metrics from the Nimble Exporter, add the following job to your Prometheus configuration:

```yaml
scrape_configs:
  - job_name: 'nimble_exporter'
    static_configs:
      - targets: ['localhost:9017']
```

## Grafana Dashboard

This repo includes a pre-configured [Grafana dashboard](https://github.com/deckhouse/nimble_exporter/tree/main/monitoring/grafana-dashboards) to help visualize the exported data. The dashboard provides insights into:

- System performance (CPU, RAM, network).
- SRT receiver metrics (latency, bandwidth, retransmissions).
- Cache sizes and server health.

You can import one of the included JSON files into your Grafana instance.