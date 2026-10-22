## Primeiro conceito: por que formatos de tabela existem

Antes do Iceberg, o jeito comum de ter uma “tabela” num data lake era: a tabela é uma pasta, e os dados são todos os arquivos Parquet que estiverem dentro dela. Para consultar, a ferramenta lista a pasta e lê todos os arquivos que encontrar. Isso funciona, mas tem cinco problemas sérios:

1. Leituras pela metade. Imagine um processo gravando 10 arquivos novos na pasta. Se alguém consultar a tabela no meio da gravação, vai ler os 4 arquivos que já chegaram e nenhum dos outros 6: um resultado incompleto, sem nenhum aviso. Se a gravação falhar no meio, os 4 arquivos ficam lá para sempre, e a tabela fica com dados pela metade.

2. Não dá para atualizar ou apagar uma linha. Arquivos Parquet não são editáveis. Para mudar o status de um pedido, você precisa reescrever o arquivo inteiro em que ele está, ou a partição inteira. É lento, caro e arriscado (e cai de volta no problema 1).

3. Descobrir os arquivos é lento. Para saber o que compõe a tabela, a ferramenta precisa listar a pasta no S3. Com milhares de partições e arquivos, só essa listagem já demora.

4. Mudar a estrutura é arriscado. Renomear ou acrescentar uma coluna pode deixar arquivos antigos e novos incompatíveis.

5. Não há histórico. Se uma carga com erro sobrescreveu os dados, a versão anterior se perdeu.

A virada do Iceberg

O Iceberg muda a definição de tabela: ela não é mais “tudo que está na pasta”. Ela é a lista de arquivos registrada nos metadados. Lembra da analogia do sumário? É exatamente isso: a tabela é o que o sumário diz, e não o que está solto na pasta.

Com isso, cada problema ganha uma solução:

uma gravação acrescenta arquivos na pasta, mas eles só passam a fazer parte da tabela quando o sumário é atualizado, de uma vez só. Quem consulta antes vê a versão anterior inteira; quem consulta depois vê a nova inteira. Nunca metade. (É o “A” do ACID: atômico.)
cada atualização do sumário gera uma versão da tabela, chamada snapshot. Os snapshots anteriores continuam registrados, e é isso que permite o time travel: consultar a tabela como ela estava ontem;
a ferramenta não precisa listar a pasta: ela lê o sumário, que já diz quais arquivos abrir.


Para fixar, com as suas palavras

No modelo “tabela = pasta”, o que acontece se alguém consultar a tabela no meio de uma gravação? E no Iceberg?
    que está lendo, ve os arquivos pela metade, em caso de algum erro na gravação, trava a consulta. ja no iceberg devido os arquivos virarem um snapshot, que está lendo ve o arquivo anterior completo e depois de salvo que fizer uma nova consulta, visualiza o novo arquivo por inteiro e se der algum erro na gravação, para quem visualiza ou vai visualizar nada acontece, pois ira ver o arquivo por inteiro que estava salvo no sumario. p arquivo só entra no sumario quando esta salvo de forma atomica, nunca pela metade.

Qual é a diferença entre as duas definições de tabela: “tudo que está na pasta” e “a lista registrada nos metadados”?
    a diferença é ao fazer uma consulta em uma pasta, a pesquisa se torna arriscada e lenta, pois se houve arquivo alterado ou corrompido afeta a pesquisa. ja na lista de metada dado a consulta fica mais segura uma vez que é realizada pelo snapshot salvo de forma atomica e apontado pelo sumario, isso traz agilidade e confiabilidade.

O que é um snapshot, e o que ele permite fazer?
    é uma foto do schema de coluna montado em um arquivo parquet, ele permite fazer alterações, controle de versionamento


A primeira está muito boa, a segunda acerta as consequências mas não a diferença em si, e a terceira precisa ser refeita. Vamos por partes.

1. Consulta no meio de uma gravação

Certo no essencial, e a última frase está ótima: no Iceberg, os arquivos só entram no sumário quando a gravação termina, nunca pela metade. Dois ajustes:

No modelo “tabela = pasta”, um erro na gravação não trava a consulta. O problema é pior: os arquivos que chegaram ficam na pasta para sempre, e todas as consultas seguintes os leem, devolvendo resultados errados sem nenhum aviso.
No Iceberg, quem consulta vê a versão anterior da tabela inteira (que pode ter vários arquivos), e não “o arquivo anterior”.

2. A diferença entre as duas definições

Você descreveu bem as consequências (lento e arriscado de um lado, ágil e confiável do outro), mas não disse a diferença em si. Ela está no que é considerado parte da tabela:

no modelo da pasta, a tabela é qualquer arquivo que exista na pasta: se um arquivo pela metade ou esquecido estiver lá, ele entra na consulta;
no Iceberg, a tabela é só o que está registrado nos metadados: um arquivo na pasta que não está no registro é ignorado.

É dessa diferença que nascem todas as consequências que você citou.

3. O que é um snapshot

Aqui houve uma confusão. O snapshot não é uma foto do schema, nem fica montado dentro de um Parquet. Use uma comparação que você já domina: o snapshot é para a tabela o que o commit é para o Git.

Um commit registra como o repositório inteiro estava num momento. Um snapshot registra como a tabela inteira estava num momento: quais arquivos a formavam (e, junto, o schema daquela versão).
Cada git commit cria um commit novo. Cada gravação numa tabela Iceberg cria um snapshot novo.
No Git, você vê uma versão antiga com o git show. No Iceberg, você consulta a tabela como ela estava num snapshot anterior: é o time travel.
E, assim como dá para voltar o código para um commit anterior, dá para voltar a tabela para um snapshot anterior, se uma carga der errado (o chamado rollback).


Refaça a 3 com as suas palavras: o que o snapshot registra, quando ele é criado, e quais as duas coisas que ele permite fazer.
    snpashot registra uma tabela inteira de um determinado momento, fazer dela uma versão; ela é salva quando um commit é realizado; viajem no tempo, alteração de colunas, controle de versão


Quase tudo certo. Dois ajustes finos:

“Quando um commit é realizado”: certo, e com um detalhe curioso: o próprio Iceberg usa a palavra commit. Cada gravação na tabela (um INSERT, um MERGE, um DELETE) termina com um commit da tabela, que é justamente a troca do sumário. Só não confunda com o git commit: são commits de coisas diferentes, com a mesma ideia.
“Alteração de colunas”: essa não vem do snapshot. Mudar as colunas de uma tabela é outro recurso do Iceberg, a evolução de schema, que você vai ver mais adiante. O snapshot permite as outras duas que você citou: viajar no tempo (consultar uma versão antiga) e voltar a uma versão anterior (o rollback, que é o “controle de versão”).

Com isso, o primeiro conceito do bloco 3 está fechado: por que formatos de tabela existem, a tabela como o que os metadados registram, gravações atômicas e snapshots.

O próximo é a estrutura por dentro de uma tabela Iceberg: as camadas do “sumário” (o arquivo de metadados, a lista de manifestos e os manifestos) e o papel do catálogo, que no seu projeto vai ser o Glue Data Catalog. É o tipo de conteúdo que um time de plataforma espera que você saiba explicar.

## A estrutura por dentro de uma tabela Iceberg


Na analogia, eu disse que o “sumário” é organizado em camadas. Agora vamos abrir essas camadas. Uma tabela Iceberg tem cinco peças, de cima para baixo:

Catálogo
   └── arquivo de metadados (metadata.json)
          └── lista de manifestos (um por snapshot)
                 └── manifestos
                        └── arquivos de dados (Parquet)

1. Arquivos de dados
As folhas: os arquivos Parquet com os dados. Ficam no S3, numa pasta data/ dentro da pasta da tabela.

2. Manifestos
Cada manifesto é uma lista de arquivos de dados, com informações sobre cada um: em que partição ele está, quantas linhas tem, e o valor mínimo e máximo de cada coluna. É com essas informações que o Iceberg descarta arquivos que não interessam a uma consulta, sem abri-los.

3. Lista de manifestos
Cada snapshot tem a sua: ela diz quais manifestos formam aquela versão da tabela, com um resumo das partições de cada um. Assim, dá para descartar manifestos inteiros antes mesmo de lê-los.

4. Arquivo de metadados (metadata.json)
A capa do sumário. Ele guarda as informações gerais da tabela: o schema, como ela é particionada, a lista de todos os snapshots e qual deles é o atual.

Os manifestos, as listas de manifestos e os metadata.json ficam numa pasta metadata/, ao lado da pasta data/.

5. Catálogo
O catálogo fica fora da pasta da tabela, e tem uma função pequena, mas crucial: para cada tabela, ele guarda o endereço do metadata.json atual. É como o fichário de uma biblioteca: você procura a tabela pelo nome, e ele diz onde está a edição em vigor. No seu projeto, o catálogo vai ser o Glue Data Catalog.

# Como uma consulta percorre as camadas

O Athena pergunta ao catálogo onde está a tabela pedidos, e recebe o endereço do metadata.json atual.
Lê o metadata.json e descobre qual é o snapshot atual.
Lê a lista de manifestos desse snapshot e descarta os manifestos que não interessam.
Lê os manifestos restantes e descarta os arquivos de dados que não interessam (pelos mínimos e máximos).
Só então abre os arquivos Parquet necessários, e deles, só as colunas da consulta.

Repare: em nenhum momento ele lista a pasta no S3.

# Como uma gravação acontece (o commit)

O caminho inverso, de baixo para cima:

1. grava os arquivos Parquet novos;
2. grava um manifesto novo, listando esses arquivos;
3. grava uma lista de manifestos nova (os manifestos antigos que continuam valendo mais o novo);
4. grava um metadata.json novo, com o snapshot novo;
5. pede ao catálogo para trocar o endereço do metadata.json antigo pelo novo.

O passo 5 é o commit: é ele que torna a gravação visível, e o catálogo garante que essa troca é atômica. Antes dele, tudo o que foi gravado existe no S3, mas ninguém enxerga.

# Por que tantas camadas?

Principalmente por dois motivos:

Descartar cedo: cada camada permite eliminar dados antes de ler a seguinte, então as consultas leem muito pouco.
Reaproveitar: um snapshot novo reaproveita os manifestos dos snapshots anteriores que não mudaram, e só acrescenta os novos. Por isso, guardar várias versões da tabela não exige copiar os dados de novo.

# Para fixar, com as suas palavras

1. Qual é a função do catálogo, e por que se diz que o commit acontece nele?
    apontar o endereço do snapshot atual; por que depois do commit concluido é nele que é possivivel visualizar o novo arquivo, atraves do seu endereço.
2. Na ordem em que o Athena percorre: quais são as cinco camadas, de cima para baixo?
    catalogo, lista de metadados. lista de manifesto, manifesto, arquivo
3. Em qual camada ficam os valores mínimo e máximo de cada coluna, por arquivo, e para que eles servem?
    manifesto, elas servem para que o obter a desempenho nas consultas, quando realizada é aqui q ele ignora tudo que n faz sentido e busca somente os dados solicitado pela engine.
4. Uma gravação deu erro no passo 3. O que quem consulta a tabela enxerga, e por quê?
    para quem visualiza, nada acontece por que ele é ignorado. o sumario só assume esse novo snapshot quando ele está commitado.


# Criando regras para os resultados do Athena

Na aba Gerenciamento, clique em Criar regra de ciclo de vida:

Nome: algo como limpa-resultados-athena.
Escopo da regra: escolha Limitar o escopo desta regra usando um ou mais filtros, e em Prefixo escreva athena-results/ (com a barra no final).
Ações: marque só Expirar versões atuais de objetos, com 7 dias.
Salve.

Não precisa marcar a de versões desatualizadas nesta regra: a “limpeza-7-dias” vale para o bucket inteiro, inclusive para o athena-results/, e já remove essas versões depois de 1 dia.

Obs:. se eu n tivesse o “limpeza-7-dias” o que deveria fazer ?

Você criaria as mesmas duas regras, só que a primeira do zero. Ficaria assim:

Regra 1: limpeza geral do bucket

Em Gerenciamento → Criar regra de ciclo de vida:

Nome: algo como limpeza-geral.
Escopo da regra: escolha Aplicar a todos os objetos no bucket. O console pede para você marcar uma caixa confirmando que entende que a regra vale para tudo.
Ações: marque as três que você acabou de configurar:
Excluir permanentemente versões desatualizadas de objetos (1 dia);
Excluir marcadores de exclusão de objeto expirados;
Excluir uploads de várias partes incompletos (1 dia).
E nunca marque “Expirar versões atuais” numa regra que vale para o bucket inteiro, porque ela alcançaria as tabelas.

Regra 2: resultados do Athena

Igual à que você está criando agora: escopo limitado ao prefixo athena-results/, com “Expirar versões atuais de objetos” em 7 dias.

E se o bucket não tivesse versionamento?

Aí as ações de versões desatualizadas e de marcadores de exclusão nem fariam sentido, porque sem versionamento o S3 não guarda versões antigas: um arquivo apagado some na hora. Na regra 1, sobraria só a dos uploads de várias partes incompletos, que vale para qualquer bucket.

A ideia geral, que serve para qualquer bucket de dados: uma regra ampla, que só limpa sobras (versões antigas, marcadores, envios incompletos), e regras com prefixo, para o que é realmente descartável. Assim, nada que esteja em uso é apagado por engano.

Repare na lógica final: as camadas de dados (raw/, trusted/, refined/) ficam protegidas, e só os resultados das consultas, que são descartáveis, expiram. Me avise quando as duas regras estiverem salvas.

# Com o bucket preparado:
 o próximo passo é configurar o Athena para gravar os resultados nesse prefixo, e então criar o primeiro banco de dados no Glue Data Catalog e a primeira tabela Iceberg. 

 Ótimo, o bucket está pronto. Agora vamos à primeira tabela Iceberg, em três partes. Hoje, a ideia é criar a tabela vazia e olhar o que ela gera no S3, para ligar com a estrutura que você acabou de estudar.

# Parte 1: dizer ao Athena onde gravar os resultados

Toda consulta no Athena grava o resultado num arquivo no S3. Antes da primeira consulta, ele precisa saber onde.

1. No console, abra o Athena e entre no Editor de consultas.
2. Se aparecer um aviso pedindo para configurar o local dos resultados, clique nele. Se não aparecer, vá na aba Configurações e clique em Gerenciar.
3. Em Local do resultado da consulta, informe: s3://seu-bucket/athena-results/ (com a barra no final).
4. Salve.

# Parte 2: criar um banco de dados no catálogo

No Athena, um “banco de dados” é só um agrupamento de tabelas dentro do Glue Data Catalog. Ele não guarda dados: só organiza os nomes. No editor, escreva e execute (botão Executar):
    CREATE DATABASE estudo_iceberg;

Parte 2 concluída: a consulta terminou como Concluído, e o estudo_iceberg já aparece selecionado em Banco de dados, no painel da esquerda, com 0 tabelas por enquanto.

Repare no canto direito: “Dados verificados: -”. Esse é o número que o Athena cobra, a quantidade de dados lidos. Comandos que só criam estruturas, como o CREATE DATABASE, não leem dado nenhum, então não custam nada. Vale olhar esse campo a cada consulta, porque é o que vai pesar na sua conta quando você consultar dados de verdade.

# Parte 3: criar a tabela Iceberg

Uma tabela pequena de teste, com três colunas dos pedidos da Olist:

sql
CREATE TABLE estudo_iceberg.pedidos_teste (
  order_id string,
  order_status string,
  order_purchase_timestamp timestamp
)
LOCATION 's3://seu-bucket/trusted/pedidos_teste/'
TBLPROPERTIES ('table_type' = 'ICEBERG');

Lendo cada parte:

- estudo_iceberg.pedidos_teste é o banco e o nome da tabela;
- entre parênteses, as colunas e os tipos: aqui o schema é definido na criação, é o schema-on-write que você estudou;
- LOCATION é a pasta no S3 onde a tabela vai viver, na camada trusted;
- TBLPROPERTIES ('table_type' = 'ICEBERG') é o que diz ao Athena que essa tabela é Iceberg, e não uma tabela comum de pasta.

Execute. A tabela deve aparecer no painel da esquerda, dentro do estudo_iceberg.


Parte 3: clique em Limpar (ou abra uma consulta nova no +), escreva o CREATE TABLE da mensagem anterior e execute. Depois, antes de abrir o S3, me mande a sua previsão: dentro de trusted/pedidos_teste/, o que deve aparecer com a tabela ainda vazia?
    é json é o schema criado, o .parquet n está pq ainda n possui os arquivos dentro do bucket, .avro só é criado depois q o snapshot é criado

Excelente leitura, e melhor que a previsão inicial: você corrigiu sozinho olhando o resultado. Confirmando cada ponto:

Só existe a metadata/, com um único metadata.json. É a capa do sumário: ele guarda o schema (as três colunas e os tipos), como a tabela é particionada (não é) e a lista de snapshots, que por enquanto está vazia.
Não existe a data/, porque não há nenhum arquivo de dados. E um detalhe do S3: ele não tem pastas de verdade. Uma “pasta” só aparece no console quando existe algum arquivo cujo nome começa com aquele prefixo. Sem Parquet, não há data/.
Não há arquivos .avro, e a sua explicação está certíssima: os manifestos e as listas de manifestos são gravados em formato Avro, e eles só são criados quando o primeiro snapshot acontece, ou seja, na primeira gravação de dados.

Repare também no nome do arquivo: ele começa com 00000-. Esse número é a versão do metadata.json. A cada commit, o Iceberg grava um novo metadata.json, com o número seguinte, e o catálogo passa a apontar para ele.

Se quiser ver por dentro, baixe esse metadata.json e abra no VS Code. Procure os trechos "schemas" (as colunas) e "current-snapshot-id" (o snapshot atual, que deve estar vazio ou com -1).


A próxima previsão

O próximo passo vai ser inserir três pedidos na tabela, com um INSERT. Antes de rodar, preveja: depois desse INSERT, o que vai aparecer de novo no S3?

Vai surgir a pasta data/? Com quantos arquivos, e de que tipo?
O que muda dentro da metadata/? Quantos metadata.json vão existir, e que arquivos .avro vão aparecer?

    agora sim a pasta data/ vai surgir por que vamos inserir  os arquivos para iniciar as consultas; acredito que após o INSERT será criado um json para o comando, mais um novo json para o arquivo e mais um novo json para o .avro,  com isso vem o primeiro snapshot

Previsão registrada, com uma parte certa: a pasta data/ vai surgir, e vem o primeiro snapshot. Sobre o que aparece na metadata/, vamos conferir com os próprios olhos.

O INSERT

Em uma consulta nova no Athena, execute:

sql
INSERT INTO estudo_iceberg.pedidos_teste VALUES
  ('pedido-001', 'delivered', TIMESTAMP '2018-01-10 10:00:00'),
  ('pedido-002', 'shipped',   TIMESTAMP '2018-01-11 14:30:00'),
  ('pedido-003', 'canceled',  TIMESTAMP '2018-01-12 09:15:00');

Cada linha entre parênteses é um pedido, com os valores na mesma ordem das colunas da tabela. O TIMESTAMP antes da data diz ao Athena que aquele texto deve virar uma data com hora, respeitando o tipo da coluna.

Depois, confira que os dados estão lá:

sql
SELECT * FROM estudo_iceberg.pedidos_teste;


Devem aparecer as três linhas. Olhe também o “Dados verificados” dessa consulta.

A conferência no S3

Abra trusted/pedidos_teste/ de novo e anote:

Na pasta data/: quantos arquivos, e com qual extensão?
Na pasta metadata/: quantos arquivos .metadata.json existem agora, e como começam os nomes deles?
Quantos arquivos .avro apareceram? Um deles começa com snap-: pela estrutura que você estudou, qual das camadas você acha que ele é?

Na data/
Um único arquivo .parquet, com os três pedidos. Ele está dentro de uma subpasta com nome aleatório (mDq9Fg/): o Athena faz isso de propósito, para espalhar os arquivos por prefixos diferentes no S3, o que ajuda o desempenho quando a tabela tem muitos arquivos. Não tem nada a ver com partição.

Na metadata/
Comparando com a sua previsão:

um metadata.json novo, que começa com 00001-: a versão seguinte ao 00000-. Foi o commit do INSERT. Confira se o 00000- continua lá também: o Iceberg não apaga o anterior na hora.
dois arquivos .avro, e não .json: o ...-m0.avro e o snap-...avro.

Ou seja, não foi criado um arquivo “para o comando”: o INSERT gerou exatamente uma peça de cada camada, de baixo para cima, como na gravação que você estudou: o Parquet, o manifesto, a lista de manifestos e o novo metadata.json.

Falta a pergunta 3: olhando os dois .avro, qual deles é o manifesto e qual é a lista de manifestos? Duas pistas: um deles tem no nome o número do snapshot (aquele número grande), e o outro termina com m0.

Quase. Você acertou que o snap-... está ligado ao snapshot, mas os papéis dos dois ficaram trocados:

snap-8145742500398239650-...avro é a lista de manifestos. Cada snapshot tem a sua, e por isso o nome dela leva o número do snapshot: 8145742500398239650.
...-m0.avro é o manifesto: a lista dos arquivos de dados. É ele que aponta para o Parquet da pasta data/, com os mínimos e máximos de cada coluna. O m0 indica que é o primeiro manifesto gerado naquela gravação.

E repare num detalhe que confirma a ligação: o nome da lista de manifestos termina com e7726987-797f-..., o mesmo código que aparece no nome do manifesto. É ela apontando para ele.

E o snapshot em si? Ele não é um arquivo. O snapshot é um registro dentro do metadata.json: um item na lista de snapshots, com o número 8145742500398239650, a data, e o endereço da sua lista de manifestos. Se você abrir o 00001-...metadata.json no VS Code e procurar por esse número, vai encontrar esse registro, e o "current-snapshot-id" agora apontando para ele.



# A estrutura::

s3://<bucket>/trusted/pedidos_teste/
├── data/
│   └── <prefixo-aleatório>/<arquivo>.parquet           ← os dados
└── metadata/
    ├── 00000-<id>.metadata.json                         ← versão 0 (tabela criada, vazia)
    ├── 00001-<id>.metadata.json                         ← versão 1 (após o INSERT)
    ├── snap-<id-do-snapshot>-1-<id>.avro                ← lista de manifestos
    └── <id>-m0.avro                                     ← manifesto

# Na pratica:

catálogo (Glue)
  → 00001-....metadata.json   (snapshot atual: 8145742500398239650)
    → snap-8145742500398239650-....avro   (lista de manifestos)
      → e7726987-...-m0.avro   (manifesto)
        → data/mDq9Fg/....parquet   (os 3 pedidos)


# Vendo o time travel funcionando: 
fazer uma segunda gravação, consultar a tabela como ela estava antes, e ver no S3 o que a segunda gravação reaproveitou.

Vamos ver o time travel funcionando, e conferir no S3 o reaproveitamento das camadas. Lembre: no Athena, um comando por execução.

# Passo 1: a previsão

Você vai inserir mais um pedido. Antes de rodar, preveja o que muda no S3:

quantos arquivos Parquet novos?
 1
quantos manifestos (-m0.avro) novos?
 2
quantas listas de manifestos (snap-...) novas?
 2
qual vai ser o número do próximo metadata.json?
 00002

E a pergunta mais interessante: a lista de manifestos nova vai apontar só para o manifesto novo, ou também para o antigo? Por quê?
    vai apontar para as duas, a lista antiga serve como complemento da nova, na pratica esse novo arquivo só compara com o antigo qual o foi a alteração realizada, utlizando a versão antiga só aplica a atualização, a partir dai o sumario passa para a nova versão do snapshot.

Passo 2: a segunda gravação

sql
INSERT INTO estudo_iceberg.pedidos_teste VALUES
  ('pedido-004', 'delivered', TIMESTAMP '2018-01-13 16:45:00');

Depois, um SELECT * FROM estudo_iceberg.pedidos_teste; (separado) deve mostrar quatro pedidos.

Passo 3: ver o histórico de snapshots

Toda tabela Iceberg no Athena tem tabelas de metadados escondidas, que você consulta acrescentando um sufixo ao nome. A de snapshots é assim:

sql
SELECT * FROM "estudo_iceberg"."pedidos_teste$snapshots";

As aspas são necessárias por causa do $. Deve aparecer uma linha para cada snapshot, com o número dele (snapshot_id), a data (committed_at) e o tipo de operação (operation).

Passo 4: viajar no tempo

Consulte a tabela como ela estava no primeiro snapshot, usando o número que você já conhece:

sql
SELECT * FROM estudo_iceberg.pedidos_teste
FOR VERSION AS OF 8145742500398239650;

Devem aparecer só os três pedidos originais, mesmo que a tabela hoje tenha quatro.

Passo 5: conferir no S3
Abra a data/ e a metadata/ de novo e compare com a sua previsão do passo 1.


# Passo 2: a segunda gravação

INSERT INTO estudo_iceberg.pedidos_teste VALUES
  ('pedido-004', 'delivered', TIMESTAMP '2018-01-13 16:45:00');


Depois, um SELECT * FROM estudo_iceberg.pedidos_teste; (separado) deve mostrar quatro pedidos.

passo 2
#
order_id
order_status
order_purchase_timestamp
1
pedido-004
delivered
2018-01-13 16:45:00.000000
2
pedido-001
delivered
2018-01-10 10:00:00.000000
3
pedido-002
shipped
2018-01-11 14:30:00.000000
4
pedido-003
canceled
2018-01-12 09:15:00.000000



# Passo 3: ver o histórico de snapshots

Toda tabela Iceberg no Athena tem tabelas de metadados escondidas, que você consulta acrescentando um sufixo ao nome. A de snapshots é assim:

SELECT * FROM "estudo_iceberg"."pedidos_teste$snapshots";

As aspas são necessárias por causa do $. Deve aparecer uma linha para cada snapshot, com o número dele (snapshot_id), a data (committed_at) e o tipo de operação (operation).


1
2026-10-22 14:45:51.711 UTC
8145742500398239650
append
s3://seu-bucket/trusted/pedidos_teste/metadata/snap-8145742500398239650-1-e7726987-797f-44a7-9c95-813c4f6a9813.avro
{changed-partition-count=1, added-data-files=1, total-equality-deletes=0, added-records=3, trino_query_id=20261022_144551_00052_itcm8, total-position-deletes=0, added-files-size=632, total-delete-files=0, total-files-size=632, total-records=3, total-data-files=1}
2
2026-10-22 15:17:12.758 UTC
2856609270296008671
8145742500398239650
append
s3://seu-bucket/trusted/pedidos_teste/metadata/snap-2856609270296008671-1-f908276b-17a6-4492-a7f2-044fe412a4a1.avro
{changed-partition-count=1, added-data-files=1, total-equality-deletes=0, added-records=1, trino_query_id=20261022_151711_00034_vv8jj, total-position-deletes=0, added-files-size=629, total-delete-files=0, total-files-size=1261, total-records=4, total-data-files=2}


# Passo 4: viajar no tempo
Consulte a tabela como ela estava no primeiro snapshot, usando o número que você já conhece:
SELECT * FROM estudo_iceberg.pedidos_teste
FOR VERSION AS OF 8145742500398239650;


order_id
order_status
order_purchase_timestamp
1
pedido-001
delivered
2018-01-10 10:00:00.000000
2
pedido-002
shipped
2018-01-11 14:30:00.000000
3
pedido-003
canceled
2018-01-12 09:15:00.000000


Passo 5: conferir no S3

Abra a data/ e a metadata/ de novo e compare com a sua previsão do passo 1.

trusted/
pedidos_teste/
data/

CDUlBQ/
mDq9Fg/

trusted/
pedidos_teste/
metadata/
 
 00000-1b6d4b17-bb2d-4940-ab11-7d3f4a825d89.metadata.json
json
22 Oct 2026 11:28:24 AM -03
1.2 KB
Padrão
00001-6c5fcee4-405a-482f-bd5e-dde3d18a37b3.metadata.json
json
22 Oct 2026 11:45:52 AM -03
2.3 KB
Padrão
00002-9322297e-bf4f-4a35-a886-25f509532a3d.metadata.json
json
22 Oct 2026 12:17:13 PM -03
3.3 KB
Padrão
e7726987-797f-44a7-9c95-813c4f6a9813-m0.avro
avro
22 Oct 2026 11:45:52 AM -03
6.7 KB
Padrão
f908276b-17a6-4492-a7f2-044fe412a4a1-m0.avro
avro
22 Oct 2026 12:17:13 PM -03
6.7 KB
Padrão
snap-2856609270296008671-1-f908276b-17a6-4492-a7f2-044fe412a4a1.avro
avro
22 Oct 2026 12:17:13 PM -03
4.2 KB
Padrão
snap-8145742500398239650-1-e7726987-797f-44a7-9c95-813c4f6a9813.avro
avro
22 Oct 2026 11:45:52 AM -03
4.2 KB
Padrão



# O time travel
A consulta FOR VERSION AS OF 8145742500398239650 devolveu só os três pedidos originais, enquanto a tabela atual tem quatro. Você consultou a tabela como ela estava antes da segunda gravação. É exatamente o que o snapshot permite.

# O histórico de snapshots
Repare na segunda linha: além do número próprio (2856609270296008671), ela tem uma coluna com 8145742500398239650, o número do snapshot anterior. É o snapshot pai, igualzinho ao Git, em que cada commit sabe qual veio antes. E o resumo da segunda linha mostra o reaproveitamento em números: added-data-files=1 (um arquivo novo), mas total-data-files=2 (a tabela tem dois). O arquivo antigo não foi copiado: continuou valendo.

# O S3, comparado com a sua previsão

Peça	Previsão	Novos de verdade
Parquet	1	1 (a subpasta CDUlBQ/)
manifesto (-m0.avro)	2	1 (o f908276b-...)
lista de manifestos (snap-...)	2	1 (o snap-2856609270...)
metadata.json	00002	00002

Cada gravação gerou uma peça nova de cada camada, e nada mais. Agora, a pergunta de antes, com a resposta que os arquivos mostram:

A lista de manifestos nova (snap-2856...) aponta para os dois manifestos: o novo (com o pedido 4) e o antigo (com os pedidos 1 a 3). O manifesto antigo é reaproveitado, sem cópia e sem comparação nenhuma: ele simplesmente continua na lista, porque os arquivos que ele descreve continuam fazendo parte da tabela.
A lista de manifestos antiga (snap-8145...) não é usada pelo snapshot novo. Ela fica guardada para o histórico: é por ela que o time travel encontrou a versão com três pedidos.

Ou seja, na sua explicação, a parte “aponta para os dois” está certa, mas quem é reaproveitado é o manifesto antigo, e não a lista antiga, e não há nenhuma comparação: só reaproveitamento.

E um último detalhe: os metadata.json estão crescendo (1,2 KB → 2,3 KB → 3,3 KB), porque cada versão carrega a lista de todos os snapshots. É um dos motivos de existir a manutenção que expira snapshots antigos.

Até aqui, no bloco 3, você viu na prática: a criação da tabela, a gravação atômica, os snapshots, o reaproveitamento das camadas e o time travel.  