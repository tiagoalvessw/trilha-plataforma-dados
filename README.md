# Trilha de estudos para Plataforma de Dados 

Objetivo: chegar à primeira oportunidade em um time de **plataforma de dados**, com um projeto de portfólio na AWS (pipeline da Olist com Apache Iceberg) que mostre os conhecimentos centrais da área.
 

---

## O que faz um time de plataforma de dados

É o time que **constrói e mantém a infraestrutura** que os outros usam:

- armazenamento (data lake / lakehouse) e formatos de tabela (Iceberg, Parquet);
- catálogo e permissões de acesso;
- ferramentas e padrões de ingestão;
- orquestração dos pipelines;
- confiabilidade, monitoramento e custos;
- infraestrutura versionada e automatizada (infraestrutura como código, CI/CD).
 
---

## Visão geral

# 

| Ordem | Bloco | Peso para plataforma | Status |
|---|---|---|---|
| 0 | Python | essencial | 🟡 |
| 1 | Git e GitHub | essencial | ⬜ |
| 2 | Conceitos de dados e de nuvem (AWS) | essencial | ⬜ |
| 3 | Apache Iceberg, Glue Catalog e Athena | central | ⬜ |
| 4 | Projeto de portfólio: o pipeline da Olist | central | ⬜ |
| 5 | Docker | importante | ⬜ |
| 6 | Airflow | importante | ⬜ |
| 7 | Terraform (infraestrutura como código) | importante | ⬜ |
| — | SQL além do básico (em paralelo, desde já) | obrigatório em entrevista | ⬜ |
| — | Modelagem dimensional (noções) | complementar | ⬜ |
 
---

## 1. Git e GitHub ⬜

**Por que importa:** todo código de plataforma vive em repositórios, passa por revisão e é publicado de forma automatizada. 

**O que estudar**

- Conceitos: repositório, commit, branch, remoto.
- Comandos: `init`, `clone`, `status`, `add`, `commit`, `push`, `pull`, `log`.
- `.gitignore` (nunca subir credenciais, dados grandes ou arquivos temporários).
- Branches e pull requests (o fluxo de revisão de código).
- README bem escrito.
- Noção de CI/CD: o que é e por que existe (aprofundar no bloco 7).
 

---

## 2. Conceitos de dados e de nuvem (AWS) ⬜

**Por que importa:** é o chão de tudo o que uma plataforma faz. Sem isso, as ferramentas viram receita de bolo.

**Conceitos de dados**

- Data lake, data warehouse e lakehouse: diferenças.
- Camadas raw, trusted e refined.
- Formatos: CSV × Parquet (orientado a linhas × colunas, compressão).
- Particionamento: o que é, por que reduz custo e tempo, e como não exagerar.
- Schema, schema-on-read × schema-on-write.
- Carga completa × incremental; noção de CDC.
- Batch × streaming (só a noção).
- Qualidade de dados: o que checar e quando.

**Conceitos de AWS**

- Regiões e zonas de disponibilidade.
- S3: buckets, prefixos, classes de armazenamento, bloqueio de acesso público.
- IAM: usuários, papéis (roles), políticas e o princípio do menor privilégio.
- Custos: como cada serviço cobra, AWS Budgets, boas práticas para não ter sustos.

**Como estudar:** curso gratuito "AWS Cloud Practitioner Essentials" (AWS Skill Builder) para a parte de nuvem; leituras curtas para os conceitos de dados.
 

---

## 3. Apache Iceberg, Glue Catalog e Athena ⬜

**Por que importa:** é o centro do processo seletivo atual (o time usa Iceberg). Em plataforma, espera-se entender como o Iceberg funciona por dentro, não só usar.

**O que estudar**

- Por que formatos de tabela existem (os problemas de usar só Parquet em pastas).
- Arquitetura do Iceberg: catálogo, arquivos de metadados, manifestos e arquivos de dados.
- Snapshots e time travel.
- Evolução de schema e de partição (partição oculta).
- Operações: `INSERT`, `MERGE`, `UPDATE`, `DELETE`.
- Manutenção: compactação de arquivos pequenos, expiração de snapshots, limpeza de arquivos órfãos.
- Glue Data Catalog como catálogo do Iceberg.
- Athena: consultar e criar tabelas Iceberg com SQL; como o custo por dados lidos funciona.
- Comparação com Delta Lake e Hudi (só o panorama).

**Como praticar:** dentro do projeto (bloco 4), nos passos 4 e 5.

---

## 4. Projeto de portfólio: o pipeline da Olist ⬜

**Por que importa:** é a prova prática de tudo. Em entrevista, um projeto ponta a ponta bem explicado vale mais que uma lista de ferramentas.

| Passo | O que fazer | Blocos que usa |
|---|---|---|
| 1 | Preparar o ambiente local, repositório e AWS CLI com usuário IAM | 1, 2 |
| 2 | Criar o bucket e as camadas (raw, trusted, refined) | 2 |
| 3 | Script de ingestão em Python (boto3) enviando os CSVs para a raw | Python, 2 |
| 4 | Catalogar a raw no Athena | 3 |
| 5 | Criar a trusted em Iceberg via Athena (tipos, duplicados, padronização) | 3 |
| 6 | Reescrever uma transformação como job PySpark no Glue, gravando Iceberg | Python, 3 |
| 7 | Montar a camada refined (tabelas de negócio) | SQL, modelagem |
| 8 | Orquestrar, checar qualidade e documentar (arquitetura, decisões, custo) | 6 ou Step Functions, 1 |

**Extensões com cara de plataforma (depois do básico funcionando)**

- Demonstrar manutenção de tabelas Iceberg (compactação, expiração de snapshots).
- Demonstrar evolução de schema e time travel no README.
- Criar a infraestrutura do projeto com Terraform (bloco 7).
- Validações automáticas no GitHub a cada commit (CI).

**Entregável:** repositório `olist-datalake-aws` com README contendo diagrama, decisões técnicas e custo aproximado.

---

## 5. Docker ⬜

**Por que importa:** plataformas rodam muitos serviços em contêineres; é a base para Airflow local, testes e ambientes reproduzíveis.

**O que estudar:** imagem × contêiner, Dockerfile, `docker run`, volumes, variáveis de ambiente, `docker compose`.

**Como praticar:** empacotar o script de ingestão num contêiner.

---

## 6. Airflow ⬜

**Por que importa:** é a ferramenta de orquestração mais pedida em vagas de engenharia e plataforma de dados.

**O que estudar:** DAGs, tarefas e dependências, agendamento, retentativas, sensores, variáveis e conexões, boas práticas (tarefas idempotentes).

**Como praticar:** rodar o Airflow localmente com Docker e orquestrar os passos do pipeline da Olist. (Evitar o Airflow gerenciado da AWS para estudo, por custo.)

---

## 7. Terraform (infraestrutura como código) ⬜

**Por que importa:** em plataforma, a infraestrutura é criada por código versionado, revisado e reproduzível, não por cliques no console.

**O que estudar:** providers, resources, variables, outputs, state, `plan` e `apply`, módulos. Noção de CI/CD aplicando Terraform.

**Como praticar:** recriar com Terraform o bucket, as permissões e o banco do Glue do projeto.

---

## SQL além do básico (em paralelo) ⬜

**Por que importa:** É usado para consultar, validar e depurar.

**O que estudar**

- JOINs (inner, left) e armadilhas de duplicação de linhas.
- `GROUP BY` com `HAVING`.
- Subqueries e CTEs (`WITH`).
- Funções de janela: `ROW_NUMBER`, `RANK`, `LAG`/`LEAD`, somas acumuladas.
- `CASE WHEN`, tratamento de nulos (`COALESCE`).
- Noções de desempenho: por que filtrar cedo e ler só as colunas necessárias.

**Como praticar:** na base da Olist no PostgreSQL que já existe, em sessões curtas.

---

## Modelagem dimensional (noções) ⬜

**Por que importa:** menos central para plataforma, mas aparece na camada refined e em conversas com times de análise.

**O que estudar:** fatos e dimensões, modelo estrela, granularidade.

---

## Como vou acompanhar

- Atualizar o status de cada bloco ao concluir.
- Para cada bloco, registrar em anotacoes os conceitos com as minhas palavras, entender antes de usar, explicar com as minhas palavras e anotar onde travei.
