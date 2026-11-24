# Divisão de tarefas

Explicação de cada frente: [FRENTES.md](FRENTES.md). Fluxo de trabalho e cronograma: [CONTRIBUINDO.md](CONTRIBUINDO.md).

## Frente A: Aplicação e orquestração (0,80): responsável: Ricardo
- [ ] Dockerfile próprio (multi-stage, usuário não-root)
- [ ] Rota /health respondendo 200
- [ ] Limite de 1 CPU / 2 GB
- [ ] docker-compose.yml base: rede, restart, variáveis de ambiente
- [ ] .env.example atualizado

## Frente B: Métricas e dashboards (1,00): responsável: Lucas
- [ ] cAdvisor no compose
- [ ] Prometheus: prometheus.yml (alvos e intervalo) + volume
- [ ] Grafana: datasource Prometheus + volume
- [ ] Dashboard próprio: CPU, RAM, rede in/out, estado do container
- [ ] Saber explicar cada query PromQL

## Frente C: Disponibilidade, alertas e testes (1,00): responsável: Wolfgang
- [ ] Uptime Kuma no compose + volume
- [ ] Monitor HTTP (intervalo e timeout definidos)
- [ ] Notificação no Discord (queda e recuperação), sem expor o webhook
- [ ] Cenários 1 e 2 executados, com prints
- [ ] Plano JMeter: 150 threads × 10, execução e análise

## Todos
- [ ] Cada um escreve a sua seção do README e tira os seus prints
- [ ] Ensaio: cada um explica a frente de outro integrante
