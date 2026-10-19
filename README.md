# DataAccess

Aplicação de console em C# voltada ao estudo prático de acesso a dados relacionais com SQL Server. O projeto explora operações CRUD, consultas parametrizadas, procedures, leitura de views, mapeamento de relacionamentos e transações usando uma camada de acesso leve e explícita.

## Visão geral

O código utiliza um modelo simples de domínio com categorias, cursos e trilhas de carreira. As operações são executadas diretamente sobre o banco por meio do Dapper, mantendo o SQL visível e permitindo observar com clareza como os dados são projetados, materializados e agrupados na aplicação.

O entry point em [`Program.cs`](./Program.cs) funciona como um catálogo executável de cenários de acesso a dados. A chamada ativa no momento é a demonstração de uma transação de inserção; os demais exemplos podem ser habilitados conforme o cenário desejado.

## Stack tecnológica

| Tecnologia | Papel no projeto |
| --- | --- |
| **C#** | Linguagem principal, com nullable reference types e implicit usings habilitados para reduzir estados inválidos e boilerplate. |
| **.NET 10** | Runtime e SDK da aplicação de console, definido em [`DataAccess.csproj`](./DataAccess.csproj). |
| **Dapper 2.1.89** | Micro-ORM usado para executar SQL, procedures e múltiplos result sets, além de mapear linhas para os modelos de domínio sem ocultar o SQL. |
| **Microsoft.Data.SqlClient 7.1.1** | Driver oficial para conexão e comunicação com o SQL Server. |
| **SQL Server** | Banco relacional utilizado pelos exemplos de categorias, cursos, carreiras e estudantes. |
| **System.Security.Cryptography.Pkcs 10.0.13** | Dependência declarada para recursos de criptografia PKCS disponíveis no ecossistema .NET. |

### Decisões técnicas

- **SQL explícito com Dapper:** favorece controle sobre joins, filtros, projeções e procedures, em vez de introduzir uma abstração pesada de ORM.
- **Parâmetros nomeados:** os comandos usam parâmetros do Dapper (`@Id`, `@Title`, `@CategoryId`), evitando interpolação direta de valores em SQL e reduzindo a superfície para injeção.
- **Mapeamento de relações:** `Query<TFirst, TSecond, TReturn>` é usado para materializar relações um-para-um e um-para-muitos, incluindo a composição de `Career` com seus `CareerItem`.
- **Transações explícitas:** a operação de inserção demonstra `BeginTransaction`, execução vinculada à transação e `Commit`, com possibilidade de rollback no mesmo fluxo.
- **Consultas compostas:** `QueryMultiple` lê categorias e cursos em uma única ida ao banco quando os dois conjuntos são necessários.

## Funcionalidades demonstradas

- Criação, atualização, exclusão e listagem de categorias.
- Busca individual com `QuerySingleOrDefault`.
- Inserção em lote e recuperação de identificador com `ExecuteScalar`.
- Execução e leitura de stored procedures.
- Leitura de view SQL (`vwCourses`).
- Consultas com `LIKE` e listas usando `IN @Id`.
- Mapeamento de relações um-para-um e um-para-muitos.
- Leitura de múltiplos result sets.
- Inserção transacional com commit explícito.

## Escopo atual

Este repositório é um projeto backend/console de acesso a dados, não uma aplicação frontend ou uma experiência de creative development no browser. Portanto, não há atualmente:

- renderização 3D ou bibliotecas WebGL;
- animações baseadas em scroll;
- vídeo usado como textura;
- gerenciamento de estado de interface;
- interações em tempo real no navegador.

Esses recursos exigiriam uma camada frontend específica e não são inferidos a partir das dependências ou imports presentes no projeto.

## Pré-requisitos

- [.NET SDK 10](https://dotnet.microsoft.com/download/dotnet/10.0)
- SQL Server acessível localmente ou em uma instância configurada
- Banco `balta` com as tabelas, views e procedures referenciadas pelos exemplos
- Permissão de leitura e escrita no banco para executar os cenários ativos

## Instalação e execução

```bash
git clone https://github.com/Thyagovz/DataAccess.git
cd DataAccess
dotnet restore
dotnet build
dotnet run
```

Antes de executar, configure a connection string em [`Program.cs`](./Program.cs) para o seu ambiente. Não mantenha credenciais reais versionadas; prefira variáveis de ambiente, secret management ou configuração local fora do repositório em qualquer uso além de estudo.

Para testar outro cenário, descomente a chamada correspondente no método `Main` e execute novamente:

```bash
dotnet run
```

Os scripts SQL de apoio estão em [`exec.sql`](./exec.sql) e [`procedure.sql`](./procedure.sql).

## Observações

O projeto serve como laboratório compacto para fundamentos de persistência com .NET e SQL Server. As implementações podem evoluir para separar configuração, conexão, repositórios e testes automatizados à medida que o escopo crescer.

### Atenção sobre credenciais

> **Observação de estudo:** a credencial que esteve exposta em uma versão anterior foi deixada
> propositalmente para fins didáticos. Ela não deve ser reutilizada, compartilhada ou tratada como
> um padrão aceitável para desenvolvimento profissional. O valor não é reproduzido neste documento.

Mesmo em um projeto de estudos, um desenvolvedor não deve versionar senhas, tokens ou strings de
conexão com credenciais reais. A referência brasileira principal é a **Lei Geral de Proteção de
Dados (LGPD), Lei nº 13.709/2018**, especialmente:

- **Art. 46:** exige medidas de segurança, técnicas e administrativas, aptas a proteger os dados
  pessoais contra acesso não autorizado e situações acidentais ou ilícitas;
- **Art. 47:** mantém o dever de garantir a segurança da informação mesmo após o seu tratamento;
- **Arts. 48 e 49:** podem trazer consequências adicionais quando houver incidente de segurança e
  falhas na estrutura de tratamento.

A aplicação concreta depende do contexto e não substitui orientação jurídica. Além da LGPD, devem
ser observadas políticas da organização, contratos e regras de segurança aplicáveis. Se uma
credencial real for publicada, ela deve ser revogada ou substituída imediatamente, mesmo que o
commit seja apagado depois.

### Melhorias registradas no último commit

No commit `371f299` (`Replace hardcoded credentials with environment variables`), foram aplicadas
as seguintes boas práticas:

- remoção da string de conexão com credencial diretamente do `Program.cs`;
- leitura da configuração pela variável de ambiente `DB_CONNECTION_STRING`;
- inclusão de `Properties/launchSettings.json` no `.gitignore`, evitando o versionamento acidental
  de configurações locais;
- abertura da conexão uma única vez no fluxo principal, permitindo seu uso pelos cenários
  selecionados;
- manutenção da consulta parametrizada no cenário ativo, sem interpolar valores diretamente no
  SQL;
- desativação da chamada de transação enquanto o cenário de leitura individual está selecionado.

Como próximos passos, a aplicação deve validar explicitamente se `DB_CONNECTION_STRING` foi
configurada, evitar registrar a string em logs e usar um gerenciador de segredos ou um mecanismo
equivalente em ambientes compartilhados e de produção. Variáveis de ambiente reduzem o risco de
versionamento, mas não substituem rotação de credenciais, menor privilégio e controle de acesso.

## Contato

Consulte o perfil do autor no [Thyagovz](https://github.com/Thyagovz) para acompanhar a evolução do projeto ou abrir uma discussão.
