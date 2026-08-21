<h1 align="center">Autenticação do Ladesa</h1>

<p align="center">Fornece recursos de login, recuperação de acesso e checagem de identidade de uma forma muito doce 💝.</p>

<div align="center">
  <a href="https://github.com/ladesa-ro/autenticacao/actions/workflows/cd.yml?query=branch%3Amain">
    <img alt="CI Development" src="https://img.shields.io/github/actions/workflow/status/ladesa-ro/autenticacao/cd.yml?style=for-the-badge&logo=githubactions&logoColor=white&label=development&branch=main&labelColor=18181B" />
  </a>
  <a href="https://github.com/ladesa-ro/autenticacao/actions/workflows/cd.yml?query=branch%3Aproduction">
    <img alt="CI Production" src="https://img.shields.io/github/actions/workflow/status/ladesa-ro/autenticacao/cd.yml?style=for-the-badge&logo=githubactions&logoColor=white&label=production&branch=production&labelColor=18181B" />
  </a>
  
</div>

<div align="center">
  <a href="https://github.com/ladesa-ro/autenticacao/actions/workflows/cd.yml?query=branch%3Aproduction">
    <img alt="docs.ladesa" src="https://img.shields.io/badge/DOCS.LADESA-118d3b?style=for-the-badge&logo=readme&logoColor=white&label=Documenta%C3%A7%C3%A3o&labelColor=18181b" />
  </a>
</div>

## Motivação

Os sistemas do Ladesa, assim como muitos aplicativos informáticos modernos, trabalham com a interação entre humanos e máquinas.

Diante disso, surge a demanda de identificar e representar as pessoas em ambientes virtuais para, dentre outras coisas, reconhecer as autorias de ações e delimitar quais operações cada perfil possa realizar em diferentes contextos.

Portanto, é necessário uma solução robusta e confiável que forneça os meios necessários para a correta identificação dos indivíduos, com a qual todo o ecossistema informático possa contar.

## Propósito

Diante da necessidade de que os softwares que conectem às plataformas do Ladesa têm de iniciar e confiar nas sessões dos usuários,
este projeto surge para cuidar do credenciamento e checagem de identidade a essas aplicações tecnológicas.

### Objetivo Geral

### Objetivos Específicos

- Login unificado (Single-Sign On);
- Fedração de Usuários (User Federation);
- Integrador de Identidade e Login Social (Identity Brokering and Social Login);
- Protocolos Padronizados (Standard Protocols).

## Desenvolvimento Local

É muito bom saber que você quer realizar o desenvolvimento do Autenticação do Ladesa. Após checar os requisitos necessários, você será guiado para obter o código-fonte deste sistema e saber, dentre outras coisas, como iniciar o desenvolvimento, subir um servidor local e construir a imagem da aplicação.

### Requisitos

Para o desenvolvimento local, é necessário preparar o seu ambiente de trabalho para mexer com este projeto. A seguir, estão listadas as tecnologias requisitadas.

- [Acesso à Linha de Comando](https://docs.ladesa.com.br/developers/tutorials/os/command-line/);
- [Git](https://docs.ladesa.com.br/developers/tutorials/source-code/git/);
- [Docker](https://docs.ladesa.com.br/developers/tutorials/platforms/containers/docker/);
- GNU Make (documentação inexistente).

> [!TIP]
>
> **Basta clicar nos links acima** para ter acesso às nossas dicas e tutoriais :).

### Obter o código-fonte do projeto

O primeiro passo para trabalhar com o serviço de Autenticação do Ladesa é obter uma cópia dos arquivos deste repositório.

Por meio dos comandos a seguir, você terá em sua máquina de desenvolvimento o acesso ao repositório deste projeto:

```sh
git clone https://github.com/ladesa-ro/autenticacao.git
cd autenticacao
```

### Serviços do [docker-compose.yml](./docker-compose.yml)

| Host               | Endereço                                                                       | Porta Alvo | Descrição               | Plataforma Base                   |
| ------------------ | ------------------------------------------------------------------------------ | ---------- | ----------------------- | --------------------------------- |
| `ladesa-ro-sso`    | `localhost:23032` (mapeamento direto); `sso.ladesa.localhost` (proxy reverso); | `8080`     | Aplicação KeyCloak      | `quay.io/keycloak/keycloak:25.0`  |
| `ladesa-ro-sso-db` | `127.128.5.11:5432`                                                            | `5432`     | Banco de dados postgres | `docker.io/bitnami/postgresql:15` |

### Scripts Make

O projeto conta com um [arquivo make](./Makefile) que comporta scrips destinados ao desenvolvimento da aplicação.

```Makefile
setup:
  # Configura o ambiente de deselvolvimento, como a criação da rede ladesa-net e os arquivos .env
up:
  # Inicia os containers docker
shell:
  # Inicia os containers docker e abre o bash na aplicação keycloak
down:
  # Para todos os containers
logs:
  # Mostra os registros dos containers
```

## Implantação (GitOps)

A implantação em produção é declarada em `gitops/` e reconciliada pelo [Argo CD](https://github.com/ladesa-ro/infrastructure), não por comando imperativo no fim do build.

```
gitops/
  envs/production/applications/sso.yaml   Application observada pelo Argo CD
  apps/sso/                               chart Helm local deste serviço
    Chart.yaml                            declara stakater/application como dependência
    charts/                               a dependência vendorizada, para o build não depender da rede
    values-production.yaml                a configuração de produção
```

São duas camadas de propósito. `envs/` diz **o que** o Argo CD deve observar e com que política de sincronização. `apps/` diz **como** o serviço é montado. Trocar a configuração de produção é editar `values-production.yaml` e abrir um pull request, com revisão e histórico, em vez de mudar uma variável de ambiente pela interface do GitHub.

O repositório `infrastructure` mantém uma `Application` raiz apontando para `gitops/envs/production/applications`, que é o que faz o Argo CD descobrir o que está aqui. A fronteira é essa: a plataforma decide que este repositório é observado, e este repositório decide o que roda.

### Segredos

Nenhum segredo vive aqui. O `Deployment` consome o Secret `ladesa-ro-sso-config` por `envFrom`, e esse Secret é produzido por um `InfisicalSecret` a partir do [Infisical](https://infisical.ladesa.com.br) self-hosted. Trocar uma senha é trocar no Infisical, não neste repositório.

### Por que não há comentário nos arquivos de `gitops/`

O ecossistema Ladesa não permite comentário em arquivo de código, e o motivo é que comentário envelhece sem que ninguém perceba. O contexto que explicaria cada bloco fica aqui e na documentação de arquitetura do `infrastructure`, onde a revisão alcança.
