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

## Integração entre as frentes: responsável: Lucas
- [ ] Unir no docker-compose.yml os serviços das três frentes (rede, volumes, depends_on)
- [ ] Conferir o contrato entre as frentes (serviço, porta, /health, rede, variáveis)
- [ ] Testar a inicialização em ambiente limpo seguindo apenas o README
- [ ] Executar o cenário 3 junto com a Frente C, acompanhando o Grafana
- [ ] Release final: PR da develop para a main e tag v1.0.0
- [ ] Preparar a máquina da apresentação

Ajustes em uma frente são feitos pelo responsável, na própria branch, para que cada um saiba explicar o que configurou.

## Todos
- [ ] Cada um escreve a sua seção do README e tira os seus prints
- [ ] Ensaio: cada um explica a frente de outro integrante
