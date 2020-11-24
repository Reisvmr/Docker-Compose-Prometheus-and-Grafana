Prometheus-Grafana
========

A monitoring solution for Docker hosts and containers with [Prometheus](https://prometheus.io/), [Grafana](http://grafana.org/), [cAdvisor](https://github.com/google/cadvisor),
[NodeExporter](https://github.com/prometheus/node_exporter) and alerting with [AlertManager](https://github.com/prometheus/alertmanager).  

This is a forked repository. So, you may want to visit the original repo at [stefanprodan
/
dockprom](https://github.com/stefanprodan/dockprom)

Additional info: [Docker - Prometheus and Grafana](https://bogotobogo.com/DevOps/Docker/Docker_Prometheus_Grafana.php)

## Install

### Create .env:
```
ADMIN_USER=admin  
ADMIN_PASSWORD=admin
```

### Clone this repository on your Docker host, cd into test directory and run compose up:

```
git clone https://github.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana.git
cd Docker-Compose-Prometheus-and-Grafana
docker-compose up -d
```

## Prerequisites:

* Docker Engine >= 1.13
* Docker Compose >= 1.11

## Containers:

* Prometheus (metrics database) `http://<host-ip>:9090`
* Prometheus-Pushgateway (push acceptor for ephemeral and batch jobs) `http://<host-ip>:9091`
* AlertManager (alerts management) `http://<host-ip>:9093`
* Grafana (visualize metrics) `http://<host-ip>:3000`
* NodeExporter (host metrics collector)
* cAdvisor (containers metrics collector)
* Caddy (reverse proxy and basic auth provider for prometheus and alertmanager)

## Setup Grafana

Navegue para `http: // <host-ip>: 3000` e faça login com o usuário *** admin *** senha *** admin ***. Você pode alterar as credenciais no arquivo de composição ou fornecendo as variáveis ​​de ambiente `ADMIN_USER` e` ADMIN_PASSWORD` via arquivo .env na composição. O arquivo de configuração pode ser adicionado diretamente na parte grafana assim

```
grafana:
  image: grafana/grafana:5.2.4
  env_file:
    - config

```
e o formato do arquivo de configuração deve ter este conteúdo
```
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=changeme
GF_USERS_ALLOW_SIGN_UP=false
```
Se você quiser alterar a senha, você deve remover esta entrada, caso contrário, a alteração não terá efeito
```
- grafana_data:/var/lib/grafana
```

Grafana é pré-configurado com painéis e Prometheus como fonte de dados padrão:

* Name: Prometheus
* Type: Prometheus
* Url: http://prometheus:9090
* Access: proxy

***Docker Host Dashboard***

![Host](https://raw.githubusercontent.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/master/screens/Grafana_Docker_Host.png)

O Docker Host Dashboard mostra as principais métricas para monitorar o uso de recursos do seu servidor:

* Tempo de atividade do servidor, porcentagem de ociosidade da CPU, número de núcleos da CPU, memória disponível, troca e armazenamento
* Gráfico de média de carga do sistema, rodando e bloqueado por gráfico de processos de IO, gráfico de interrupções
* Gráfico de uso da CPU por modo (convidado, inativo, iowait, irq, nice, softirq, roubar, sistema, usuário)
* Gráfico de uso de memória por distribuição (usado, livre, buffers, em cache)
* Gráfico de uso de IO (leia Bps, leia Bps e tempo IO)
* Gráfico de uso da rede por dispositivo (Bps de entrada, Bps de saída)
* Trocar gráficos de uso e atividade

Para armazenamento e particularmente gráfico de armazenamento gratuito, você deve especificar o fstype na solicitação de gráfico grafana.
Você pode encontrá-lo em `grafana/dashboards/docker_host.json`, na linha 480:

      "expr": "sum(node_filesystem_free_bytes{fstype=\"btrfs\"})",

trabalhar em BTRFS, então eu preciso mudar `aufs` para` btrfs`.

Você pode encontrar o valor correto para o seu sistema no Prometheus `http: // <host-ip>: 9090` iniciando esta solicitação:

      node_filesystem_free_bytes
***Docker Containers Dashboard***

![Containers](https://raw.githubusercontent.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/master/screens/Grafana_Docker_Containers.png)

O Docker Containers Dashboard mostra as principais métricas para monitorar contêineres em execução:

* Total de carga de CPU de contêineres, uso de memória e armazenamento
* Execução de gráfico de contêineres, gráfico de carga do sistema, gráfico de uso de IO
* Gráfico de uso da CPU do contêiner
* Gráfico de uso de memória do contêiner
* Gráfico de uso de memória em cache do contêiner
* Gráfico de uso de entrada da rede de contêineres
* Gráfico de uso de saída da rede de contêiner

Observe que este painel não mostra os contêineres que fazem parte da pilha de monitoramento.

***Monitor Services Dashboard***

![Monitor Services](https://raw.githubusercontent.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/master/screens/Grafana_Prometheus.png)
![Monitor Services](https://raw.githubusercontent.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/master/screens/Grafana_Prometheus2.png)
![Monitor Services](https://raw.githubusercontent.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/master/screens/Grafana_Prometheus3.png)

O Monitor Services Dashboard mostra as principais métricas para monitorar os contêineres que compõem a pilha de monitoramento:

* Tempo de atividade do contêiner do Prometheus, monitoramento do uso da memória total da pilha, blocos e séries de memória de armazenamento local do Prometheus
* Gráfico de uso da CPU do contêiner
* Gráfico de uso de memória do contêiner
* Pedaços de Prometheus para persistir e gráficos de urgência de persistência
* Prometheus agrupa operações e gráficos de duração de pontos de verificação
* Taxa de ingestão de amostras de Prometheus, raspagens de destino e gráficos de duração de raspagem
* Gráfico de solicitações HTTP do Prometheus
* Gráfico de alertas do Prometheus
## Define alerts

Três grupos de alerta foram configurados dentro do [alert.rules](https://github.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/blob/master/prometheus/alert.rules) 

Arquivo de configuração:

* Monitoring services alerts [targets](https://github.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/blob/master/prometheus/alert.rules#L2-L11)
* Docker Host alerts [host](https://github.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/blob/master/prometheus/alert.rules#L13-L40)
* Docker Containers alerts [containers](https://github.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/blob/master/prometheus/alert.rules#L42-L69)

Você pode modificar as regras de alerta e recarregá-las fazendo uma chamada HTTP POST para Prometheus:

```
curl -X POST http://admin:admin@<host-ip>:9090/-/reload
```

***Monitoring services alerts***

Acione um alerta se algum dos alvos de monitoramento (node-exporter e cAdvisor) ficar inativo por mais de 30 segundos:

```yaml
- alert: monitor_service_down
    expr: up == 0
    for: 30s
    labels:
      severity: critical
    annotations:
      summary: "Monitor service non-operational"
      description: "Service {{ $labels.instance }} is down."
```

***Docker Host alerts***

Acione um alerta se a CPU do host Docker estiver sob alta carga por mais de 30 segundos:

```yaml
- alert: high_cpu_load
    expr: node_load1 > 1.5
    for: 30s
    labels:
      severity: warning
    annotations:
      summary: "Server under high load"
      description: "Docker host is under high load, the avg load 1m is at {{ $value}}. Reported by instance {{ $labels.instance }} of job {{ $labels.job }}."
```

Modifique o limite de carga com base em seus núcleos de CPU.

Acione um alerta se a memória do host Docker estiver quase cheia:

```yaml
- alert: high_memory_load
    expr: (sum(node_memory_MemTotal_bytes) - sum(node_memory_MemFree_bytes + node_memory_Buffers_bytes + node_memory_Cached_bytes) ) / sum(node_memory_MemTotal_bytes) * 100 > 85
    for: 30s
    labels:
      severity: warning
    annotations:
      summary: "Server memory is almost full"
      description: "Docker host memory usage is {{ humanize $value}}%. Reported by instance {{ $labels.instance }} of job {{ $labels.job }}."
```

Acione um alerta se o armazenamento do host Docker estiver quase cheio:

```yaml
- alert: high_storage_load
    expr: (node_filesystem_size_bytes{fstype="aufs"} - node_filesystem_free_bytes{fstype="aufs"}) / node_filesystem_size_bytes{fstype="aufs"}  * 100 > 85
    for: 30s
    labels:
      severity: warning
    annotations:
      summary: "Server storage is almost full"
      description: "Docker host storage usage is {{ humanize $value}}%. Reported by instance {{ $labels.instance }} of job {{ $labels.job }}."
```

***Docker Containers alerts***

Acione um alerta se um contêiner ficar inativo por mais de 30 segundos:

```yaml
- alert: jenkins_down
    expr: absent(container_memory_usage_bytes{name="jenkins"})
    for: 30s
    labels:
      severity: critical
    annotations:
      summary: "Jenkins down"
      description: "Jenkins container is down for more than 30 seconds."
```

Acione um alerta se um contêiner estiver usando mais de 10% do total de núcleos de CPU por mais de 30 segundos:

```yaml
- alert: jenkins_high_cpu
    expr: sum(rate(container_cpu_usage_seconds_total{name="jenkins"}[1m])) / count(node_cpu_seconds_total{mode="system"}) * 100 > 10
    for: 30s
    labels:
      severity: warning
    annotations:
      summary: "Jenkins high CPU usage"
      description: "Jenkins CPU usage is {{ humanize $value}}%."
```

Acione um alerta se um contêiner estiver usando mais de 1,2 GB de RAM por mais de 30 segundos:

```yaml
- alert: jenkins_high_memory
    expr: sum(container_memory_usage_bytes{name="jenkins"}) > 1200000000
    for: 30s
    labels:
      severity: warning
    annotations:
      summary: "Jenkins high memory usage"
      description: "Jenkins memory consumption is at {{ humanize $value}}."
```

## Setup alerting

O serviço AlertManager é responsável pelo tratamento dos alertas enviados pelo servidor Prometheus.
O AlertManager pode enviar notificações por e-mail, Pushover, Slack, HipChat ou qualquer outro sistema que exponha uma interface de webhook.
Uma lista completa de integrações pode ser encontrada[here](https://prometheus.io/docs/alerting/configuration).

Você pode ver e silenciar notificações acessando `http://<host-ip>:9093`.

Os receptores de notificação podem ser configurados em [alertmanager/config.yml](https://github.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/blob/master/alertmanager/config.yml) file.

Para receber alertas via Slack, você precisa fazer uma integração personalizada, escolhendo *** entrantes web hooks *** na página do aplicativo de equipe do Slack.
Você pode encontrar mais detalhes sobre como configurar a integração com o Slack [here](http://www.robustperception.io/using-slack-with-the-alertmanager/).

Copie o URL do Slack Webhook no campo *** api_url *** e especifique um canal *** do Slack.

```yaml
route:
    receiver: 'slack'

receivers:
    - name: 'slack'
      slack_configs:
          - send_resolved: true
            text: "{{ .CommonAnnotations.description }}"
            username: 'Prometheus'
            channel: '#<channel>'
            api_url: 'https://hooks.slack.com/services/<webhook-id>'
```

![Slack Notifications](https://raw.githubusercontent.com/Einsteinish/Docker-Compose-Prometheus-and-Grafana/master/screens/Slack_Notifications.png)

## Sending metrics to the Pushgateway

The [pushgateway](https://github.com/prometheus/pushgateway) is used to collect data from batch jobs or from services.

To push data, simply execute:

    echo "some_metric 3.14" | curl --data-binary @- http://user:password@localhost:9091/metrics/job/some_job

Substitua a parte `user:password` pelo seu usuário e senha definidos na configuração inicial (padrão:`admin:admin`).

## Updating Grafana to v5.2.2

[Nas versões Grafana> = 5.1 o id do usuário grafana foi alterado](http://docs.grafana.org/installation/docker/#migration-from-a-previous-version-of-the-docker-container-to-5-1-or-later).Infelizmente, isso significa que os arquivos criados antes do 5.1 não terão as permissões corretas para as versões posteriores.

| Version |   User  | User ID |
|:-------:|:-------:|:-------:|
|  < 5.1  | grafana |   104   |
|  \>= 5.1 | grafana |   472   |

Existem duas soluções possíveis para este problema.
- Alterar propriedade de 104 para 472
- Inicie o contêiner atualizado como usuário 104

##### Specifying a user in docker-compose.yml


Para alterar a propriedade dos arquivos, execute o seu container grafana como root e modifique as permissões.

Primeiro execute um `docker-compose down` e então modifique seu docker-compose.yml para incluir a opção` user:root`:

```
  grafana:
    image: grafana/grafana:5.2.2
    container_name: grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/datasources:/etc/grafana/datasources
      - ./grafana/dashboards:/etc/grafana/dashboards
      - ./grafana/setup.sh:/setup.sh
    entrypoint: /setup.sh
    user: root
    environment:
      - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped
    expose:
      - 3000
    networks:
      - monitor-net
    labels:
      org.label-schema.group: "monitoring"
```

Execute um `docker-compose up -d` e emita os seguintes comandos:

```
docker exec -it --user root grafana bash
# no contêiner que você acabou de iniciar:
chown -R root:root /etc/grafana && \
chmod -R a+r /etc/grafana && \
chown -R grafana:grafana /var/lib/grafana && \
chown -R grafana:grafana /usr/share/grafana
```

Para executar o contêiner grafana como `user:104`, altere seu` docker-compose.yml` assim:

```
  grafana:
    image: grafana/grafana:5.2.2
    container_name: grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/datasources:/etc/grafana/datasources
      - ./grafana/dashboards:/etc/grafana/dashboards
      - ./grafana/setup.sh:/setup.sh
    entrypoint: /setup.sh
    user: "104"
    environment:
      - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: unless-stopped
    expose:
      - 3000
    networks:
      - monitor-net
    labels:
      org.label-schema.group: "monitoring"
```
