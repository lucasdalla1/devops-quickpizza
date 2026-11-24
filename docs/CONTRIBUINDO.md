# Como contribuir

## Preparar o ambiente

Pré-requisitos: Git, Docker e Docker Compose.

```bash
git clone https://github.com/lucasdalla1/devops-quickpizza.git
cd devops-quickpizza
git switch develop
cp .env.example .env   # preencher os valores localmente
```

> Se a porta 22 (SSH) estiver bloqueada na sua rede, use a URL HTTPS acima e rode `gh auth setup-git` uma vez para autenticar pelo GitHub CLI.

## Fluxo de branches (Git Flow)

| Branch | Uso |
|---|---|
| `main` | Versão entregue. Recebe apenas a release final. |
| `develop` | Integração do trabalho da equipe. |
| `feature/<descricao>` | Uma tarefa por branch, criada a partir da `develop`. |

`main` e `develop` são protegidas: ninguém faz push direto. Toda alteração entra por Pull Request com a aprovação de outro integrante.

## Passo a passo de uma tarefa

```bash
git switch develop
git pull
git switch -c feature/minha-tarefa
# desenvolver e testar
git add <arquivos>
git commit -m "adiciona healthcheck no compose"
git push -u origin feature/minha-tarefa
```

Em seguida, abrir o Pull Request para a `develop` e pedir a revisão de outro integrante.

## Regras

- **Segredos:** webhook, senhas e tokens ficam apenas no `.env` local. Antes de cada commit, confira o `git diff`.
- **Commits:** uma linha, em minúscula, começando pelo verbo: `adiciona ...`, `corrige ...`, `atualiza ...`, `remove ...`.
- **Pull Requests pequenos:** uma tarefa por PR facilita a revisão.
- **Revisão:** quem revisa testa localmente com `docker compose up -d` antes de aprovar.
- **Contrato entre as frentes:** mudanças em nomes de serviço, portas, rotas ou variáveis devem atualizar a tabela em [FRENTES.md](FRENTES.md#contrato-entre-as-frentes).

## Cronograma

| Data | Meta |
|---|---|
| 27/11 | Frente A na `develop`: aplicação rodando via Compose com `/health` |
| 01/12 | Frentes B e C integradas ao Compose via PR |
| 02/12 a 03/12 | Execução dos cenários, evidências, README e ensaio da apresentação |
| 04/12 | Release `v1.0.0` na `main` e apresentação |
