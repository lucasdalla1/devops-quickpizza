# Requisitos do projeto

Resumo do enunciado do Projeto Final da disciplina DevOps Tools (prof. Renato Sousa Botacim). O documento original fica fora do repositório porque contém o webhook do Discord.

## Objetivo

Montar um pequeno ambiente de produção com containers Docker e ferramentas de observabilidade. O foco não é a complexidade da aplicação, e sim construir, configurar e demonstrar uma infraestrutura capaz de executar e monitorar uma aplicação web.

- Executar uma aplicação web em container Docker
- Orquestrar a aplicação e as ferramentas de monitoramento com Docker Compose
- Coletar métricas dos containers com Prometheus
- Criar dashboards no Grafana com indicadores de infraestrutura
- Monitorar a disponibilidade da aplicação com Uptime Kuma
- Notificar incidentes e recuperação no Discord
- Executar testes controlados de falha e analisar os resultados

## Requisitos técnicos obrigatórios

### 1. Aplicação e Docker
Aplicação definida pelo professor: **QuickPizza** ([grafana/quickpizza](https://github.com/grafana/quickpizza)).

- Criar um Dockerfile para construir a imagem da aplicação
- Recursos do container: **1 CPU e 2 GB**
- Executar a aplicação em container e permitir o acesso pelo navegador
- Usar Docker Compose para subir e gerenciar todos os serviços
- Configurar rede Docker, volumes persistentes, variáveis de ambiente e política de reinicialização quando aplicável

### 2. Coleta de métricas
No mínimo:
- Consumo de CPU do container da aplicação
- Consumo de memória RAM
- Tráfego de rede (entrada e saída), quando disponível nas métricas coletadas
- Estado ou atividade do container

### 3. Dashboards no Grafana
- Grafana com o Prometheus como fonte de dados
- Dashboard **próprio**, com painéis para os indicadores acima e seu comportamento ao longo do tempo
- É permitido usar documentação e dashboards existentes como referência, mas a equipe deve configurar a fonte de dados, entender as consultas e montar os painéis. Importar um dashboard pronto sem adaptação e explicação **não atende** ao requisito.

### 4. Uptime Kuma
- Uptime Kuma em container
- Monitor do tipo HTTP(s) para a aplicação
- Intervalo de verificação e tempo limite de resposta definidos
- Demonstrar o histórico de disponibilidade e as mudanças de estado do monitor

### 5. Notificações no Discord
- Integração do Uptime Kuma com o webhook do Discord fornecido pelo professor
- Uma notificação quando a aplicação ficar indisponível e outra quando voltar
- **O webhook é secreto:** não publicar no repositório, em arquivos versionados, em prints ou na documentação. Usar configuração local/variáveis de ambiente e versionar apenas um `.env.example` sem valores.

## Cenários de teste obrigatórios

Não basta mostrar os serviços funcionando: é preciso comprovar que o monitoramento identifica mudanças e registra os eventos.

### Cenário 1: Indisponibilidade
1. Parar o container da aplicação
2. Aguardar o Uptime Kuma identificar a falha
3. Comprovar o recebimento do alerta no Discord
4. Mostrar o incidente no histórico do Uptime Kuma

### Cenário 2: Recuperação
1. Iniciar novamente o container da aplicação
2. Aguardar o Uptime Kuma identificar a recuperação
3. Comprovar a notificação de recuperação no Discord
4. Mostrar a mudança de estado no histórico

### Cenário 3: Teste de carga com Apache JMeter
1. Configurar um plano de teste no JMeter
2. Usar **150 usuários virtuais (threads)** e **10 requisições** por usuário
3. Executar o teste contra a aplicação em Docker
4. Acompanhar ao mesmo tempo os gráficos de CPU e memória no Grafana
5. Analisar o comportamento da aplicação durante o teste
6. Identificar aumento no tempo de resposta, erros HTTP e indisponibilidade
7. Apresentar e explicar os resultados

Evidências obrigatórias: configuração do plano de teste, resultado da execução, gráficos do Grafana durante a carga e análise dos resultados.

> O enunciado diz "150 (threads) e o 10 número de requisições". A equipe interpretou como 150 threads com 10 repetições cada (1.500 requisições). Confirmar com o professor.

## Entregáveis

| Entregável | Descrição |
|---|---|
| Repositório Git | Código da aplicação, Dockerfile, Docker Compose e arquivos de configuração |
| Configuração do Prometheus | Arquivo com os alvos e intervalos de coleta |
| Dashboard Grafana | Dashboard da equipe com os painéis exigidos (print) |
| Monitor Uptime Kuma | Monitor HTTP configurado para a aplicação (print) |
| Integração Discord | Evidências dos alertas de indisponibilidade e recuperação, sem expor o webhook |
| Documentação | README com pré-requisitos, instruções de execução, arquitetura e explicação dos componentes |
| Evidências | Prints dos dashboards, do monitor e dos testes realizados |

O README deve explicar como iniciar o projeto (referência: `docker compose up -d`).

## Critérios de avaliação (3,0 pontos)

| Critério | Valor |
|---|---|
| Conteinerização da aplicação (Dockerfile e execução) | 0,40 |
| Docker Compose e organização dos serviços | 0,40 |
| Prometheus e cAdvisor: coleta de métricas | 0,45 |
| Grafana: dashboard, painéis e interpretação | 0,55 |
| Uptime Kuma: monitoramento HTTP e histórico | 0,35 |
| Discord: alertas de indisponibilidade e recuperação | 0,35 |
| Demonstração dos cenários de falha e recuperação | 0,30 |
| Documentação, repositório e organização | 0,20 |
| **Total** | **3,00** |

A nota pode considerar a participação e a capacidade de **cada integrante** explicar os componentes, as configurações e os resultados dos testes.

## Orientações gerais

- Todos os serviços em containers Docker, gerenciados pelo Docker Compose
- Dados do Grafana, Prometheus e Uptime Kuma em volumes Docker
- Não versionar senhas, tokens ou webhooks; o repositório tem apenas exemplos de configuração sem segredos
- Testar a inicialização em ambiente limpo ou explicar claramente os pré-requisitos

## Checklist antes da entrega

- [ ] A aplicação inicia em container e pode ser acessada pelo navegador
- [ ] O endpoint `/health` responde HTTP 200 quando a aplicação está funcionando
- [ ] O Docker Compose inicia todos os serviços necessários
- [ ] O Prometheus coleta métricas do cAdvisor
- [ ] O Grafana apresenta os gráficos solicitados
- [ ] O Uptime Kuma monitora a aplicação
- [ ] O Discord recebe alerta de indisponibilidade
- [ ] O Discord recebe alerta de recuperação
- [ ] Os três cenários de teste foram executados
- [ ] O README explica como executar e testar o ambiente
- [ ] O webhook e demais segredos não estão publicados no Git
