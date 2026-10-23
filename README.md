# ASD-STE100 — Guia de arquitetura da API

## Estado do repositório

Este repositório ainda não contém uma solução .NET, projetos, código-fonte ou testes. Este README registra a arquitetura planejada. Ele não descreve funcionalidades já implementadas.

O caso de uso de referência é uma API de autenticação para um site. O escopo inicial prevê login, renovação de sessão, logout e consulta do perfil do usuário.

## Objetivo

Organizar a API para que as regras de negócio permaneçam claras, as dependências sigam limites definidos e os comportamentos importantes possam ser testados.

A solução combina:

- DDD para modelar regras e conceitos do domínio.
- CQRS para separar operações que alteram estado das consultas.
- Clean Architecture para manter o núcleo da aplicação independente de detalhes externos.
- Clean Code para facilitar leitura e manutenção.
- Testes unitários, de arquitetura e especificações BDD para verificar comportamentos e limites.
- Princípios de UX/UI para orientar os fluxos e a interface que consome a API.
- ASD-STE100 como referência para documentação técnica escrita em inglês controlado.

## Estrutura planejada

    src/
      Auth.Api/
      Auth.Application/
      Auth.Domain/
      Auth.Infrastructure/

    tests/
      Auth.UnitTests/
      Auth.ArchitectureTests/
      Auth.BddSpecs/

Os nomes acima são uma proposta. O repositório ainda não tem projetos que confirmem essa estrutura.

## Responsabilidade de cada projeto

### Auth.Api

Recebe requisições HTTP e devolve respostas HTTP. Define os endpoints, os contratos de entrada e saída e a configuração de inicialização.

Os endpoints encaminham as ações para Application. Eles não contêm regras de negócio nem acesso direto ao banco de dados.

### Auth.Application

Contém os casos de uso da API. Define Commands, Queries, Handlers, validação e as interfaces necessárias para acessar recursos externos.

Exemplos de Commands: Login, RefreshSession e Logout. Um exemplo de Query: GetUserProfile.

### Auth.Domain

Contém os conceitos e as regras centrais do domínio. Pode conter entidades, objetos de valor, agregados e eventos de domínio quando eles representarem regras reais do produto.

O domínio de autenticação deve proteger invariantes de negócio, como o estado permitido de uma conta ou de uma sessão. Não se deve criar modelos de domínio para cada DTO ou operação simples.

### Auth.Infrastructure

Implementa persistência e integrações externas. Pode conter configurações do Entity Framework Core, repositórios e adaptadores para provedores de identidade.

A escolha entre contas locais e um provedor externo de identidade depende dos requisitos. A solução não deve criar mecanismos próprios de criptografia, armazenamento de senha ou emissão de tokens.

### Auth.UnitTests

Verifica regras do Domain e comportamentos de Application de forma isolada. Esses testes não devem depender de banco de dados, rede ou serviços externos.

O objetivo é detectar regressões nas regras e nos casos de uso com testes pequenos e determinísticos.

### Auth.ArchitectureTests

Verifica automaticamente as dependências entre projetos. O objetivo é impedir que mudanças futuras violem os limites da Clean Architecture.

Regras planejadas:

- Domain não depende de Application, Infrastructure ou API.
- Application depende de Domain.
- Infrastructure implementa abstrações usadas por Application e pode depender de Domain.
- API usa Application. A referência a Infrastructure fica restrita à composição e à configuração de serviços.
- Domain não depende de Entity Framework Core nem de ASP.NET Core.

### Auth.BddSpecs

Contém especificações executáveis em Gherkin, usando Given, When e Then. As especificações descrevem comportamentos do usuário ou do negócio pela fronteira pública da aplicação.

Exemplo:

    Feature: User sign-in

    Scenario: An active user signs in with valid credentials
      Given an active user account exists
      When the user sends valid credentials
      Then the API confirms the user session

BDD complementa os testes unitários. As especificações não devem repetir detalhes internos de Handlers ou classes privadas.
