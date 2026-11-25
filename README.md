# DevOps QuickPizza

Projeto final da disciplina DevOps Tools (UVV), prof. Renato Sousa Botacim.

## Equipe
| Integrante | GitHub | Frente |
|---|---|---|
| Lucas | @lucasdalla1 | B: Métricas e dashboards |
| Ricardo | @taliuli10 | A: Aplicação e orquestração |
| Wolfgang | | C: Disponibilidade, alertas e testes |

Organização da equipe: [requisitos do projeto](docs/REQUISITOS.md), [frentes de trabalho](docs/FRENTES.md), [tarefas](docs/TAREFAS.md) e [como contribuir](docs/CONTRIBUINDO.md).

## Visão geral
## Arquitetura
## Pré-requisitos
## Como executar
## Componentes
### Aplicação (QuickPizza)
### Prometheus e cAdvisor

O **cAdvisor** lê do Docker o consumo de cada container e expõe esses dados em `/metrics`. O **Prometheus** busca as métricas periodicamente (modelo *pull*) e guarda o histórico no volume `prometheus_data`.

Configuração em [`prometheus/prometheus.yml`](prometheus/prometheus.yml), com coleta a cada **5 segundos**, para que picos curtos do teste de carga apareçam nos gráficos:

| Job | Alvo | Métricas |
|---|---|---|
| `cadvisor` | `cadvisor:8080` | CPU, memória, rede e estado dos containers |
| `quickpizza` | `quickpizza:3333/metrics` | Requisições por status HTTP e tempo de resposta da aplicação |
| `prometheus` | `localhost:9090` | Saúde do próprio Prometheus |

Os alvos são acessados pelo nome do serviço, pois todos estão na rede Docker `monitoring`. A situação de cada alvo pode ser vista em http://localhost:9090/targets.

**cAdvisor no Docker Desktop (WSL 2):** a configuração padrão do cAdvisor não identifica os containers nesse ambiente. O serviço foi ajustado no `docker-compose.yml`:
- imagem `ghcr.io/google/cadvisor:v0.61.1`, compatível com o armazenamento de imagens do Docker 29 (as versões novas não são mais publicadas no `gcr.io`);
- `pid: host` e `--containerd=/proc/1/root/run/containerd/containerd.sock`, para acessar o containerd que roda dentro da VM do Docker Desktop;
- `mount --bind /proc /rootfs/proc` antes de iniciar, porque o `/` montado a partir do WSL é o da distribuição Linux, e não o da VM; sem isso as métricas de rede por container não aparecem;
- `--disable_metrics=disk`, pois a coleta de disco é incompatível com o Docker 29 e não faz parte dos requisitos.

### Grafana

O Grafana é configurado automaticamente quando sobe (*provisioning*), sem nenhum passo manual:
- [`grafana/provisioning/datasources/prometheus.yml`](grafana/provisioning/datasources/prometheus.yml): cria a fonte de dados Prometheus (`http://prometheus:9090`);
- [`grafana/provisioning/dashboards/dashboards.yml`](grafana/provisioning/dashboards/dashboards.yml): carrega os dashboards da pasta `grafana/dashboards`;
- [`grafana/dashboards/quickpizza.json`](grafana/dashboards/quickpizza.json): dashboard criado pela equipe.

Acesso em http://localhost:3000 com o usuário e a senha definidos no `.env` (`GRAFANA_ADMIN_USER` e `GRAFANA_ADMIN_PASSWORD`). Os dados ficam no volume `grafana_data`.

O dashboard **QuickPizza: infraestrutura e aplicação** tem três seções:

| Painel | Consulta (PromQL) | Leitura |
|---|---|---|
| Uso de CPU | `sum(rate(container_cpu_usage_seconds_total{...}[30s])) * 100` | *Counter* de segundos de CPU convertido em taxa com `rate()`. Como o limite é 1 CPU, 100% = limite atingido |
| Uso de memória | `sum(container_memory_working_set_bytes{...})` | *Gauge*, usado diretamente. A linha tracejada é o limite de 2 GB (`container_spec_memory_limit_bytes`) |
| Tráfego de rede | `sum(rate(container_network_receive_bytes_total{...}[30s]))` e `..._transmit_...` | *Counters* de bytes convertidos em bytes por segundo (entrada e saída) |
| Estado do container | `up{job="quickpizza"}` | 1 (UP) quando a aplicação responde à coleta, 0 (DOWN) quando está fora do ar |
| Requisições por status | `sum by (status) (rate(quickpizza_server_http_requests_total[30s]))` | Requisições por segundo separadas por código HTTP |
| Tempo de resposta | `histogram_quantile(0.95, ...)` e soma / contagem | Percentil 95 e média do tempo de resposta |
| Taxa de erros 5xx | requisições 5xx / total × 100 | Percentual de respostas com erro do servidor |

O filtro `{name=~"(.*-)?quickpizza(-[0-9]+)?"}` seleciona o container da aplicação tanto com o nome criado pelo Compose (`devops-quickpizza-quickpizza-1`) quanto com o nome simples (`quickpizza`), sem incluir os demais serviços.

### Uptime Kuma e Discord
## Cenários de teste
### 1. Indisponibilidade
### 2. Recuperação
### 3. Teste de carga (JMeter)
## Evidências
## Segurança
