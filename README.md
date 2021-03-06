# prometheus_pcf
This repository shows how to push a Prometheus binary into Cloud Foundry application.

## manifest.yml
```yaml
---
applications:
- name: prometheus-test
  instances: 1
  buildpack: https://github.com/cloudfoundry/binary-buildpack.git
  command: ./prometheus --config.file=prometheus_pcf.yml --web.listen-address=:8080
  memory: 256M
  random-route: true
```
By default, Cloud Foundry seems to set the port of this type of applications to `8080`.

## prometheus_pcf.yml
```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:8080'] # this field is configured

  - job_name: 'pcf-spring-boot'
    metrics_path: '/actuator/prometheus'
    scheme: https
    static_configs:
    - targets: ['university-unplugging-metrics.cfapps.us10.hana.ondemand.com']
```

## How to push into CF
To push with `manifest.yml` file above, you need to have the `prometheus` binary in your root directory.

Download here: https://prometheus.io/download/
```shell
$ cf push your-app-name
```
OBS: In older versions, Java don't recognize the character `_` (underscore) in urls, so be careful.