# Frentes de trabalho

O projeto monta um pequeno ambiente de produção: a aplicação roda em container, um sistema coleta as métricas dela, outro verifica se ela está no ar e a equipe é avisada no Discord quando ela cai ou volta.

O trabalho foi dividido em três frentes, que correspondem às camadas desse ambiente:

```
            ┌──────────── Frente C ─────────────┐
 JMeter ──► │ Uptime Kuma ──(caiu/voltou)──► Discord
   │        └────────│──────────────────────────┘
   │                 │ GET /health a cada X s
   ▼                 ▼
┌──── Frente A ──────────┐      ┌──────── Frente B ─────────────┐
│ QuickPizza (1 CPU/2GB) │◄─────│ cAdvisor ◄── Prometheus ◄── Grafana
│ docker-compose         │ lê   │ (métricas)   (coleta)    (gráficos)
└────────────────────────┘      └───────────────────────────────┘
```

| Frente | Responsável | Critérios de avaliação | Pontos |
|---|---|---|---|
| A: Aplicação e orquestração | Ricardo | Conteinerização + Docker Compose | 0,80 |
| B: Métricas e dashboards | Lucas | Prometheus/cAdvisor + Grafana | 1,00 |
| C: Disponibilidade, alertas e testes | Wolfgang | Uptime Kuma + Discord + Cenários | 1,00 |
| Todos | equipe | Documentação, repositório e organização | 0,20 |

A nota considera a capacidade de **cada integrante** explicar os componentes. Todos devem entender as três frentes, não só a própria.

---

## Frente A: Aplicação e orquestração

**Objetivo:** executar o QuickPizza em um container construído pela equipe e organizar todos os serviços em um `docker-compose.yml`.

**Entregas**
- Dockerfile próprio para o QuickPizza
- Rota `/health` respondendo HTTP 200
- Limite de recursos do container da aplicação: 1 CPU e 2 GB
- `docker-compose.yml` base com rede, política de reinicialização e variáveis de ambiente
- `.env.example` com os nomes das variáveis, sem valores secretos

**Conceitos-chave**
- **Multi-stage build (Aula 13):** o QuickPizza é um backend em Go com frontend em Svelte. As etapas pesadas (Node e Go) compilam; a imagem final leva apenas o binário.
- **Usuário não-root e `.dockerignore` (Aula 13):** imagem menor e mais segura.
- **Limites de recursos:** no Compose, em `deploy.resources.limits`. Pode ser comprovado com `docker stats`.
- **Rede Docker (Aula 12):** dentro do Compose, os serviços se comunicam pelo nome do serviço, não por `localhost`.
- **Política de reinicialização:** `unless-stopped` reinicia o container se ele falhar, mas respeita um `docker stop` manual, o que é necessário no Cenário 1.

**Como validar**
- `docker compose up -d` sobe a aplicação
- A aplicação abre no navegador
- `curl -i http://localhost:3333/health` retorna `200 OK`

**Pontos de atenção**
- O QuickPizza já possui `/healthz` e `/ready`, mas o checklist do professor exige `/health`.
- Esta frente destrava as outras duas: a aplicação precisa estar na `develop` o quanto antes.

---

## Frente B: Métricas e dashboards

**Objetivo:** coletar o consumo de recursos dos containers e apresentá-lo em um dashboard criado pela equipe.

**Entregas**
- cAdvisor no Compose
- Prometheus com `prometheus.yml` (alvos e intervalo de coleta) e volume persistente
- Grafana com o Prometheus como fonte de dados e volume persistente
- Dashboard próprio com: CPU, memória RAM, tráfego de rede (entrada e saída) e estado do container da aplicação

**Conceitos-chave**
- **Fluxo dos dados:** o cAdvisor lê do Docker o consumo de cada container e expõe em `/metrics`; o Prometheus busca esses dados periodicamente (modelo *pull*, Aula 32); o Grafana consulta o Prometheus com PromQL (Aula 33).
- **Counter × Gauge:**
  - CPU e rede são *counters* (só crescem). É preciso usar `rate()` para obter o consumo por segundo.
  - Memória é *gauge* (sobe e desce) e pode ser usada diretamente.
- **Métricas do cAdvisor:** `container_cpu_usage_seconds_total`, `container_memory_working_set_bytes`, `container_network_receive_bytes_total`, `container_network_transmit_bytes_total`, `container_last_seen`.
- **Interpretação:** com limite de 1 CPU, um `rate` de CPU igual a 0,5 significa 50% do limite.

**Como validar**
- Em `Status → Targets` do Prometheus, todos os alvos aparecem como `UP`
- Os painéis do Grafana mudam quando a aplicação recebe carga

**Pontos de atenção**
- Importar um dashboard pronto sem adaptação e explicação **não atende** ao requisito.
- No Docker Desktop, o cAdvisor pode não exibir o nome dos containers. Testar isso primeiro.
- O QuickPizza também expõe métricas próprias em `/metrics` (requisições e latência), úteis para analisar o teste de carga.

---

## Frente C: Disponibilidade, alertas e testes

**Objetivo:** comprovar que o monitoramento detecta falhas, avisa a equipe e mostrar o comportamento da aplicação sob carga.

**Entregas**
- Uptime Kuma no Compose com volume persistente
- Monitor HTTP para a aplicação, com intervalo e tempo limite definidos
- Notificação no Discord para indisponibilidade e para recuperação
- Cenários 1 e 2 executados e registrados
- Plano de teste do JMeter (150 usuários × 10 repetições), execução e análise

**Conceitos-chave**
- **Monitor HTTP:** dentro da rede Docker, o Uptime Kuma acessa a aplicação pelo nome do serviço (ex.: `http://quickpizza:3333/health`).
- **Intervalo, timeout e retries:** definem quanto tempo o Uptime Kuma leva para declarar a aplicação fora do ar. A equipe deve saber calcular e explicar esse tempo.
- **Teste de carga:** *Thread Group* com 150 threads e 10 repetições (1.500 requisições), um *HTTP Request* e *listeners* para os resultados. Executar em modo não gráfico para não distorcer o teste.
- **Análise:** tempo médio e percentil 95 de resposta, percentual de erros e vazão (requisições/s), cruzados com CPU e memória no Grafana.

**Cenários**
1. **Indisponibilidade:** `docker compose stop <serviço da app>` → alerta no Discord → incidente no histórico do Uptime Kuma
2. **Recuperação:** `docker compose start <serviço da app>` → notificação de recuperação no Discord → mudança de estado no histórico
3. **Carga:** executar o JMeter acompanhando o Grafana e explicar o comportamento observado

**Pontos de atenção**
- O webhook do Discord é **secreto**: fica apenas no `.env` local. Nunca em arquivo versionado, documentação ou print.

---

## Contrato entre as frentes

Valores combinados que as três frentes usam. Se algum mudar, avise a equipe e atualize esta tabela no mesmo PR.

| Item | Valor | Usado por |
|---|---|---|
| Nome do serviço da aplicação | `quickpizza` | B (consultas), C (URL do monitor) |
| Porta da aplicação | `3333` | Todas |
| Rota de saúde | `/health` | C |
| Rede Docker | `monitoring` | Todas |
| Variáveis do `.env` | `DISCORD_WEBHOOK_URL`, `GRAFANA_ADMIN_USER`, `GRAFANA_ADMIN_PASSWORD` | B, C |
