# Qual é o principal problema de um data lake sem nenhuma organização?
    Vira um pantano de dados, um amontado de arquivos que dificultam o entendimento do que é e pra que serve, tornando a manutenção e controle dificil.

# O que o lakehouse pega do data lake, e o que pega do data warehouse?
    do data lake a alta capacidade de armazenamento, liberdade para armazenar qualquer tipo de arquivo, seja ele Json, CVS, Parquet e etc, a praticidade e velocidade de extração, o carregamento dos arquivos e o baixo custo, ja do data warehouse pega a organização
    
# Na sua opinião, por que uma empresa preferiria um lakehouse a só um warehouse?
    pelo custo, poder armazenar qualquer tipo de arquivo, a capacidade de controle dos dados devido a camada de gerenciamento com qualquer ferramenta que o mercado oferece.

# Pontos de melhorias:


Muito bom. As três respostas mostram que a ideia principal ficou clara. Alguns ajustes para deixá-las com a precisão que se espera numa entrevista:

1. O problema do data lake desorganizado
Certo. Vale acrescentar um ponto técnico: além da bagunça, faltam garantias. Se dois processos escrevem nos mesmos arquivos ao mesmo tempo, ou se uma gravação falha no meio, ninguém sabe qual é a versão correta. Isso abala a confiança nos dados, e dado em que ninguém confia não é usado.

2. O que o lakehouse pega de cada um
A parte do lake está boa: armazenamento barato e de grande capacidade, e aceitar qualquer formato. Só um cuidado com “velocidade de extração”: o lake não é mais rápido para consultar; o que ele facilita é guardar os dados do jeito que chegam, sem precisar organizar antes.

A parte do warehouse ficou genérica. “Organização”, em termos técnicos, significa:

tabelas com schema (colunas e tipos definidos);
transações confiáveis: uma gravação acontece por inteiro ou não acontece. Isso tem um nome que você vai ouvir muito: ACID;
atualizar e apagar linhas com segurança;
histórico de versões das tabelas.

3. Por que um lakehouse
Ótimo, e o seu último ponto é um dos mais importantes: “qualquer ferramenta que o mercado oferece”. O nome técnico disso é formato aberto. Uma tabela Iceberg não pertence a nenhum fornecedor: a mesma tabela pode ser lida pelo Athena, pelo Spark, pelo Trino ou pelo Snowflake. Num warehouse tradicional, os dados ficam presos ao produto (o chamado lock-in). Outro ponto que vale citar: no lakehouse, armazenamento e processamento são separados. Você paga o S3 para guardar e só paga o processamento quando consulta.


# As camadas: raw, trusted e refined

Um lakehouse organiza os dados em etapas, como uma linha de produção: o dado entra bruto e vai sendo tratado até ficar pronto para uso. Cada etapa é uma camada, normalmente uma pasta (ou prefixo) separada no S3.

Raw (bruta)
Os dados exatamente como chegaram da origem, sem nenhuma alteração: os CSVs da Olist, como foram baixados. A regra de ouro: a raw nunca é alterada nem apagada. Ela é a cópia fiel da origem. Se algo der errado numa etapa seguinte, você sempre pode reprocessar a partir dela.

Trusted (confiável)
Os dados limpos e padronizados: tipos corrigidos (datas como datas, valores como números), duplicados removidos, nomes de colunas padronizados, valores inválidos tratados. Ainda no nível de detalhe original: um pedido continua sendo uma linha. Na Olist, seriam as tabelas de pedidos, itens e pagamentos, limpas. É aqui que entram as tabelas Iceberg do seu projeto.

Refined (refinada)
Os dados prontos para o negócio: tabelas que juntam e resumem as da trusted para responder perguntas. Por exemplo, vendas por mês e por estado, ou uma tabela de pedidos já com o valor total e o tempo de entrega calculados. É o que os analistas e os dashboards consomem.

Os mesmos conceitos com outros nomes
Você vai encontrar essas camadas com nomes diferentes, dependendo da empresa: bronze, silver e gold (a chamada arquitetura medalhão, muito usada), ou landing, curated e consumption. A ideia é sempre a mesma.

Por que separar em camadas

Reprocessar: se uma regra de limpeza estiver errada, você corrige e roda de novo a partir da raw, sem precisar buscar os dados na origem.
Rastrear: dá para saber de onde veio cada número de um relatório.
Separar responsabilidades e acessos: o time de plataforma cuida da ingestão e da raw; quem analisa usa a refined, e muitas vezes nem tem acesso à raw.

# Para fixar, com as suas palavras

Por que a raw nunca deve ser alterada? Pense no que aconteceria se a limpeza tivesse um erro e a raw já tivesse sido sobrescrita.
    Ele serve como BKP de ponto de partida, quando algum processo de refinamento da errado, vc pode sempre usar o raw como ponto de partida para recomeçar. 

Em qual camada ficaria cada um destes, na Olist?
a. o arquivo orders.csv como veio do Kaggle; bronze
b. uma tabela com o total vendido por categoria de produto em cada mês; silver
c. a tabela de pedidos com as datas convertidas para o tipo data e sem pedidos duplicados. gold

Um analista pede acesso à raw para fazer um relatório. Que argumento você usaria para sugerir a refined no lugar?
    O raw n pode ser acessado pois ele é o bkp, possui somente os dados brutos sem tratamento e organização, utilizar refined é a melhor opção pois la estão os dados com suas devidas colunas e linhas sem duplicatas, perfeito para utlização e o mais importante caso haja algum problema, n há com o que se preocupar, temos um ponto de partida, seguro.


## As camadas: raw, trusted e refined

Um lakehouse organiza os dados em etapas, como uma linha de produção: o dado entra bruto e vai sendo tratado até ficar pronto para uso. Cada etapa é uma camada, normalmente uma pasta (ou prefixo) separada no S3.

# Raw (bruta)
Os dados exatamente como chegaram da origem, sem nenhuma alteração: os CSVs da Olist, como foram baixados. A regra de ouro: a raw nunca é alterada nem apagada. Ela é a cópia fiel da origem. Se algo der errado numa etapa seguinte, você sempre pode reprocessar a partir dela.

# Trusted (confiável)
Os dados limpos e padronizados: tipos corrigidos (datas como datas, valores como números), duplicados removidos, nomes de colunas padronizados, valores inválidos tratados. Ainda no nível de detalhe original: um pedido continua sendo uma linha. Na Olist, seriam as tabelas de pedidos, itens e pagamentos, limpas. É aqui que entram as tabelas Iceberg do seu projeto.

# Refined (refinada)
Os dados prontos para o negócio: tabelas que juntam e resumem as da trusted para responder perguntas. Por exemplo, vendas por mês e por estado, ou uma tabela de pedidos já com o valor total e o tempo de entrega calculados. É o que os analistas e os dashboards consomem.

Os mesmos conceitos com outros nomes
Você vai encontrar essas camadas com nomes diferentes, dependendo da empresa: bronze, silver e gold (a chamada arquitetura medalhão, muito usada), ou landing, curated e consumption. A ideia é sempre a mesma.

Por que separar em camadas

Reprocessar: se uma regra de limpeza estiver errada, você corrige e roda de novo a partir da raw, sem precisar buscar os dados na origem.
Rastrear: dá para saber de onde veio cada número de um relatório.
Separar responsabilidades e acessos: o time de plataforma cuida da ingestão e da raw; quem analisa usa a refined, e muitas vezes nem tem acesso à raw.

Para fixar, com as suas palavras

Por que a raw nunca deve ser alterada? Pense no que aconteceria se a limpeza tivesse um erro e a raw já tivesse sido sobrescrita.
Em qual camada ficaria cada um destes, na Olist?
a. o arquivo orders.csv como veio do Kaggle;
b. uma tabela com o total vendido por categoria de produto em cada mês;
c. a tabela de pedidos com as datas convertidas para o tipo data e sem pedidos duplicados.
Um analista pede acesso à raw para fazer um relatório. Que argumento você usaria para sugerir a refined no lugar?

1. Por que a raw nunca deve ser alterada? Pense no que aconteceria se a limpeza tivesse um erro e a raw já tivesse sido sobrescrita.
    Ele serve como BKP de ponto de partida, quando algum processo de refinamento da errado, vc pode sempre usar o raw como ponto de partida para recomeçar. 

2. Em qual camada ficaria cada um destes, na Olist?
a. o arquivo orders.csv como veio do Kaggle; bronze
b. uma tabela com o total vendido por categoria de produto em cada mês; silver
c. a tabela de pedidos com as datas convertidas para o tipo data e sem pedidos duplicados. gold

3. Um analista pede acesso à raw para fazer um relatório. Que argumento você usaria para sugerir a refined no lugar?
    O raw n pode ser acessado pois ele é o bkp, possui somente os dados brutos sem tratamento e organização, utilizar refined é a melhor opção pois la estão os dados com suas devidas colunas e linhas sem duplicatas, perfeito para utlização e o mais importante caso haja algum problema, n há com o que se preocupar, temos um ponto de partida, seguro.

# A primeira e a terceira estão no caminho certo, e a segunda teve uma troca importante. Vamos por partes.

1. Por que a raw não é alterada
Certo: ela é o ponto de partida seguro. Um detalhe que torna o argumento mais forte: muitas vezes a origem não guarda os dados para sempre. Uma API pode mostrar só os últimos 30 dias, um sistema pode apagar registros antigos. Nesses casos, a raw é a única cópia que existe do dado original. Se ela for sobrescrita por uma limpeza com erro, não há como recuperar.

2. A classificação
A a está certa: o CSV como veio do Kaggle é raw (bronze). Mas a b e a c ficaram invertidas:

c (pedidos com datas convertidas e sem duplicados) é trusted / silver. É limpeza e padronização, mas cada pedido continua sendo uma linha, com o mesmo nível de detalhe da origem.
b (total vendido por categoria em cada mês) é refined / gold. Os pedidos foram juntados e resumidos para responder uma pergunta de negócio.

Uma regra prática para não confundir: se o dado foi limpo, mas ainda está no nível de detalhe original, é silver. Se foi agregado, combinado ou calculado para uma pergunta de negócio, é gold.

3. O argumento para o analista
A ideia de que a refined está tratada está certa. Mas dois pontos da resposta não se sustentam:

A raw pode ser lida sem risco de alteração, então “é o backup” não é o motivo de negar acesso. O acesso costuma ser restrito por governança: a raw pode ter dados sensíveis ainda não tratados, e controlar quem lê cada camada é parte do trabalho de plataforma.
“Caso haja algum problema, temos um ponto de partida” é a função da raw para o time de dados, e não um argumento para o analista.

Os argumentos mais fortes seriam: na raw há duplicados e tipos errados, então o relatório pode sair com números errados; na refined, as regras de negócio já foram aplicadas do mesmo jeito para todos, então os relatórios de áreas diferentes batem entre si; e consultar a refined costuma ser mais rápido e mais barato, porque ela é menor e já está organizada.


# Para fixar a diferença entre silver e gold

Classifique estes, usando a regra prática:

1. a tabela de pagamentos com o valor convertido de texto para número;
2. o ticket médio por estado do cliente;
3. a tabela de produtos com as categorias traduzidas para português e os nulos tratados;
4. uma tabela com, para cada pedido, o valor total, o tempo de entrega em dias e se atrasou ou não.

Atenção ao 4: ele tem uma linha por pedido, mas pense se foi só limpeza ou se houve cálculo para o negócio.

a tabela de pagamentos com o valor convertido de texto para número; silver
o ticket médio por estado do cliente; gold
a tabela de produtos com as categorias traduzidas para português e os nulos tratados; silver
uma tabela com, para cada pedido, o valor total, o tempo de entrega em dias e se atrasou ou não. gold


Quatro de quatro. E o 4 era a pegadinha: mesmo tendo uma linha por pedido, ele é gold, porque junta várias tabelas (pedidos, itens, pagamentos) e calcula informações de negócio que não existiam na origem, como “atrasou ou não”. A regra se refina assim: o que define a camada não é só o nível de detalhe, mas se houve regra de negócio aplicada. Limpar e padronizar é silver; combinar e calcular para responder perguntas é gold.

Repare também que o 3 tem uma sutileza que você acertou: traduzir as categorias é padronização, e não regra de negócio, então fica na silver.

Até aqui, no bloco 2, você tem: warehouse, lake e lakehouse; formato aberto; ACID em linhas gerais; e as camadas. Vale registrar no anotacoes.md a regra prática da silver e da gold.

## CSV × Parquet 
# Como o CSV guarda os dados: por linha

O CSV grava uma linha inteira depois da outra, com todas as colunas juntas:

pedido1, cliente1, delivered, 2017-10-02, ...
pedido2, cliente2, shipped,   2018-07-24, ...

Para saber só os status de todos os pedidos, o programa precisa ler todas as colunas de todas as linhas, porque o status está misturado com o resto. Você viu isso na prática: o seu contagem_status.py lia o arquivo inteiro para usar uma coluna só.

Como o Parquet guarda os dados: por coluna

# O Parquet agrupa os valores de cada coluna juntos:

order_id:     pedido1, pedido2, ...
customer_id:  cliente1, cliente2, ...
order_status: delivered, shipped, ...

Para contar os status, ele lê só o bloco da coluna order_status e ignora as outras sete. Por isso se diz que o Parquet é um formato colunar.

# As vantagens do Parquet

 - Ler só as colunas necessárias. Análises quase sempre usam poucas colunas de tabelas largas.
 - Compressão muito melhor. Valores da mesma coluna são parecidos entre si (a coluna de status repete “delivered” milhares de vezes), e dados parecidos comprimem muito bem. Um Parquet costuma ficar várias vezes menor que o CSV equivalente.
 - Tipos guardados no arquivo. No CSV, tudo é texto, e cada ferramenta precisa adivinhar se “2017-10-02” é data e se “99.90” é número (você viu isso no DictReader, que lê tudo como texto). O Parquet guarda o schema junto: a data já é data.
 - Estatísticas internas. O arquivo guarda, para cada bloco de linhas, o menor e o maior valor de cada coluna. Uma consulta por pedidos de 2018 pode pular blocos inteiros que só têm 2017, sem lê-los.

# Por que isso é dinheiro, no Athena

O Athena cobra pela quantidade de dados lidos em cada consulta. Ler 1 coluna de 8, já comprimida, em vez do CSV inteiro, pode reduzir o volume lido, e portanto o custo, em mais de 90%. E fica mais rápido pelo mesmo motivo.

E o CSV, onde fica?

Ele continua útil: é legível por humanos, universal, e é como muitos dados chegam. Por isso a raw costuma guardar o formato original (CSV, JSON), e a conversão para Parquet acontece na passagem para a trusted. E o Iceberg? Os dados de uma tabela Iceberg ficam, por baixo, justamente em arquivos Parquet. O Iceberg é a camada de tabela por cima deles. 

 - Para fixar, com as suas palavras

Uma consulta no Athena quer só a média do valor pago, numa tabela de pagamentos com 5 colunas. Por que ela fica mais barata em Parquet do que em CSV? Cite pelo menos dois motivos.
    1 - Por que ele vai direto ao ponto, busca somente os dados da duas duas colunas e monta a tabela com o resumo solicitado.
    2 - Custo reduzido em 90%, Athena cobra com dados carregados, invés de ler um arquivo csv inteiro para montar um eschema monto o schema em parquet, com os dados das colunas expecifica, ignorando todo o resto; uma observação é que uma ver que carregado  o dado bruto, criado um arquivo parquet, ele é inteligente por que sabe o que tem e em qual coluna está o dado está, por isso as consultas são rapidas.

Por que faz sentido manter a raw em CSV, e não converter tudo para Parquet logo na chegada?
    é legiivel par humanos e é universal, é como muitos dados chegam, por esse motivo o raw armazena o dado original.

Qual a relação entre o Parquet e o Iceberg?
    o Iceberg fica por baixo do arquivo parquet, ele é o organizador de schema do parquet.



# Bom, a ideia geral está aí. Mas as três respostas têm imprecisões que, numa entrevista, chamariam atenção. Vamos uma por uma.

1. Por que o Parquet fica mais barato

O primeiro motivo está quase certo: o Athena lê só a coluna necessária. Um detalhe: a média do valor pago usa uma coluna, a do valor, e não duas.

O segundo motivo repete o primeiro com outras palavras (“lê só as colunas, por isso custa menos”). E os 90% foram um exemplo do quanto pode reduzir, não um valor fixo. O que faltou foi um motivo diferente. Os outros que vimos:

Compressão: mesmo a coluna lida ocupa menos bytes em Parquet, porque valores parecidos comprimem muito bem. Menos bytes lidos, menos custo.
Estatísticas internas: se a consulta tiver um filtro (por exemplo, só pagamentos acima de 100), o Parquet pode pular blocos inteiros cujo valor máximo é menor que 100.

E uma correção sobre o schema: ele não é montado a cada consulta. Ele é gravado dentro do arquivo Parquet, uma vez, quando o arquivo é criado. A consulta só lê.

2. Por que manter a raw em CSV

Legibilidade e universalidade são motivos secundários. O motivo principal liga com o conceito anterior: a raw é a cópia fiel da origem. Converter para Parquet já é uma transformação: alguém decide que “2017-10-02” é data e que “99.90” é número. Se essa decisão estiver errada e a raw já tiver sido convertida, o original se perdeu. Mantendo o formato de origem na raw, a conversão acontece na passagem para a trusted, e pode ser refeita quantas vezes precisar.

3. Parquet e Iceberg

A ideia de “organizador” está certa, mas a direção ficou invertida: o Iceberg fica por cima dos arquivos Parquet, e não por baixo.

Pense num livro de folhas soltas: os arquivos Parquet são as folhas, com os dados de verdade. O Iceberg é o sumário por cima delas: ele registra quais folhas formam a tabela, qual é o schema e quais versões a tabela já teve. Quando você consulta uma tabela Iceberg, o Athena primeiro lê o sumário para descobrir quais arquivos Parquet abrir, e só então lê os dados.


# Para conferir o entendimento

Responda curto, com as suas palavras:

Cite os três motivos pelos quais uma consulta em Parquet lê menos dados do que em CSV.
    le só a coluna necessária, se a consulta possui filtro, ele vai direto ao ponto e ignora todo o conteudo que n está na busca, arquivos partque consome menos bytes.
Na analogia do livro, o que é a “folha” e o que é o “sumário”? E quem fica por cima de quem?
    parquet e iceberg o sumario

Agora sim, as duas estão certas:

Os três motivos: lê só a coluna necessária (formato colunar), pula blocos que não atendem ao filtro (estatísticas internas) e ocupa menos bytes (compressão).
As folhas são os arquivos Parquet, com os dados, e o sumário é o Iceberg, que fica por cima deles, dizendo quais folhas formam a tabela.


## Particionamento: 
A forma de organizar os arquivos em pastas (como ano=2018/mes=01/) para que uma consulta nem precise abrir os arquivos de períodos que não interessam. É a terceira forma de “ler menos”, junto com o colunar e as estatísticas.

A ideia

Particionar é dividir os arquivos de uma tabela em pastas, de acordo com o valor de uma coluna. O exemplo clássico é por data. Os pedidos da Olist poderiam ficar assim no S3:

s3://bucket/trusted/orders/ano=2017/mes=10/arquivo.parquet
s3://bucket/trusted/orders/ano=2017/mes=11/arquivo.parquet
s3://bucket/trusted/orders/ano=2018/mes=01/arquivo.parquet
...

Esse formato de nome, coluna=valor, é um padrão que as ferramentas (Athena, Spark, Glue) reconhecem automaticamente.

 - Por que isso faz ler menos

Se a consulta pede só os pedidos de janeiro de 2018, o Athena olha os nomes das pastas e abre apenas a pasta ano=2018/mes=01. As outras nem são tocadas. Isso se chama partition pruning (“podar” as partições que não interessam).

Repare na diferença em relação às estatísticas do Parquet: lá, o Athena precisa abrir cada arquivo para descobrir que pode pular blocos; aqui, ele descarta pastas inteiras sem abrir nenhum arquivo, só pelo nome. Somando as três técnicas:

Técnica	            O que evita ler
particionamento	    pastas inteiras
estatísticas do     Parquet	blocos de linhas dentro de um arquivo
formato colunar	    colunas que a consulta não usa

- Como escolher a coluna da partição

A partição só ajuda se as consultas filtram por ela. Por isso a escolha mais comum é a data: quase toda análise pergunta “neste período”. Duas armadilhas:

Particionar demais. Se você particionar os pedidos por customer_id, que tem quase 100 mil valores diferentes, cria quase 100 mil pastas com um arquivo minúsculo cada. Abrir milhares de arquivos pequenos é mais lento que ler poucos arquivos grandes. Esse é o famoso problema dos arquivos pequenos, muito comum em plataformas de dados.
Particionar por algo que ninguém filtra. As pastas existem, mas toda consulta precisa abrir todas, e a partição só atrapalha.

A regra prática: particione por uma coluna usada nos filtros e com poucos valores diferentes, de forma que cada partição tenha um volume razoável de dados. Para a base da Olist, que é pequena, particionar por mês já seria até exagerado; num e-commerce real, com milhões de pedidos por dia, a partição por dia faria sentido.

E o Iceberg?
O Iceberg melhora o particionamento de duas formas, que você vai estudar no bloco 3:

Partição oculta: você define que a tabela é particionada pelo mês da data da compra, e quem consulta só filtra pela data normal. O Iceberg descobre sozinho quais partições usar, sem a pessoa precisar saber como as pastas estão organizadas.
Evolução de partição: se o volume crescer e você quiser mudar de mês para dia, dá para mudar a partição da tabela sem reescrever os dados antigos.


Uma tabela de pagamentos está particionada por mês. Uma consulta pede o total pago em março de 2018. O que o Athena lê, e o que ele não lê?
    Athena lê o mes e o ano espefico, ignora todo o resto

Um colega sugere particionar a tabela de pedidos por order_id. O que você responderia?
    Não é uma boa opção por que cada linha seria uma partição e se tiver 1000 linhas, seria 1000 partições.

As consultas de um time quase sempre filtram por estado do cliente (são 27). E quase nunca por data. Qual coluna faria mais sentido para a partição, e por quê?
    id cliente , por que o filtro é pelo estado do mesmo, então as consultas serão mais objetivas, ignorando todo o restos, buscando só a informação necessária.. 


 A. Uma tabela de acessos a um site recebe 10 milhões de linhas por dia. Colunas: id_usuario, data_acesso, pagina e pais. As consultas quase sempre pedem “os acessos dos últimos 7 dias”.
     coluna pagina por que com ela fica mais facil fazer a consulta solicitada. 

B. Uma tabela de vendas tem 50 milhões de linhas. Os analistas quase sempre filtram por categoria_produto (são 15 categorias) e raramente por data.
    categoria_produto por ter menos comparação só 15.

C. Uma tabela de cadastro de vendedores tem 3 mil linhas e é consultada poucas vezes por semana. Colunas: id_vendedor, cidade, estado.   
    estado é a ideal pelo menos motivo da resposta anterior, por ter menos comparação só 27.

A B está certa. A A e a C mostram um padrão que vale corrigir agora: você está escolhendo pela condição “poucos valores” e deixando de lado a primeira, “o que as consultas filtram”. Vamos refazer olhando as duas, nesta ordem.

A. Os acessos ao site

Primeira pergunta: o que as consultas filtram? O enunciado diz: “os acessos dos últimos 7 dias”. O filtro é por data. A coluna pagina não aparece no filtro, então uma partição por página não seria aproveitada: a consulta dos últimos 7 dias abriria todas as pastas de todas as páginas.

Segunda pergunta: o volume por partição é razoável? Com partição por dia, cada pasta teria 10 milhões de linhas, um volume ótimo. E a consulta dos últimos 7 dias abriria só 7 pastas.

B. As vendas

Certo: categoria_produto. Só complete a justificativa com a primeira condição: além de ter só 15 valores, é a coluna que os analistas filtram.

C. O cadastro de vendedores

Aqui a resposta é não particionar. Com 3 mil linhas, a tabela inteira cabe em um único arquivo pequeno, que o Athena lê quase instantaneamente. Particionando por estado, você criaria 27 arquivos ainda menores, de cerca de 100 linhas cada, e cairia no problema dos arquivos pequenos: mais arquivos para abrir, mais lento, sem ganho nenhum. Além disso, o enunciado nem diz que as consultas filtram por estado.    


## Carga completa × carga incremental


Toda ingestão responde a uma pergunta: a cada execução, eu trago tudo de novo ou só o que mudou?

# Carga completa (full load)
A cada execução, você copia a origem inteira e substitui o que tinha antes.

Vantagem: é simples, e o resultado é sempre um retrato exato da origem naquele momento. Se algo foi apagado na origem, também some no destino.
Desvantagem: custa caro e demora quando a origem é grande. Copiar 700 milhões de transações todo dia para pegar as 2 milhões novas é desperdício.
Quando usar: tabelas pequenas ou que mudam por inteiro, como o cadastro de 3 mil vendedores do exercício. Ou a primeira carga de qualquer tabela.

# Carga incremental
A cada execução, você traz só o que entrou ou mudou desde a última vez.

Vantagem: muito mais rápida e barata em tabelas grandes.
Desvantagem: é mais complexa. Você precisa saber o que mudou, e precisa tratar alterações em linhas que já existem.
Quando usar: tabelas grandes que crescem com o tempo, como a de transações de cartão.

# Como saber o que mudou: a coluna de controle

O jeito mais comum é usar uma coluna de data na origem, como data_criacao ou data_atualizacao. O processo guarda a maior data que já trouxe, e na execução seguinte pede só as linhas com data maior que ela. Esse marcador se chama watermark (marca d’água).

Exemplo: ontem a carga trouxe pedidos até 2018-08-29 23:59. Hoje, ela pede só os pedidos depois desse momento.

# O problema das linhas que mudam

Um pedido da Olist nasce como “created”, depois vira “approved”, “shipped” e “delivered”. Se a carga de ontem trouxe o pedido como “shipped”, e hoje ele mudou para “delivered”, a carga incremental de hoje traz o pedido de novo. O destino não pode simplesmente acrescentar essa linha: ficaria com o mesmo pedido duas vezes, com status diferentes.

O certo é: se o pedido já existe, atualiza; se não existe, insere. Essa operação se chama upsert (update + insert), e em SQL é feita com o comando MERGE. Um data lake comum, só com arquivos Parquet, não consegue fazer isso com segurança. O Iceberg consegue, e esse é um dos principais motivos de ele existir.

# E o CDC?

Uma forma mais avançada de incremental: em vez de perguntar “o que tem data maior que X”, o processo lê o registro interno de alterações do banco de origem, que anota cada inserção, atualização e exclusão. Chama-se CDC (change data capture, captura de mudanças). Na AWS, o serviço DMS faz isso. Para agora, basta saber que existe e que resolve o caso das exclusões, que a coluna de controle não enxerga.

# Exercicio

Para cada tabela, qual tipo de carga você usaria, e por quê?
a. uma tabela de 50 linhas com as cotações das moedas, que muda inteira todo dia;
    usaria a carga completa, por ter poucas linhas, e por ser simples, é um retrato exato do momento. 

b. uma tabela de pedidos com 200 milhões de linhas, que recebe 300 mil pedidos novos por dia.
    usaria a incremental por ser mais rapido e barato em tabelas grandes.

Na carga incremental de pedidos, por que não basta acrescentar as linhas novas no destino? Use o exemplo do status.
    se um pedido ontem é a caminho e hoje é entregue, a carga incremental traz um novo pedido; isso é um problema pois ele ficara com o mesmo pedido 2x, porém status diferentes.

Uma carga incremental usa a coluna data_criacao como watermark. Um pedido antigo, criado há um mês, teve o status alterado hoje. Ele entra na carga de hoje? Que coluna resolveria isso?
    a coluna data_criacao, o processo guarda a maior data  atual, se houve uma alteração, ele vai novamente guardar essa nova data com a alteração do status.

A 1 e a 2 estão certas. A 3 tem o ponto mais importante do assunto, então vale olhar com calma.

1. Certo nas duas. Na a, um motivo extra reforça a escolha: como a tabela muda inteira todo dia, uma carga incremental traria tudo de qualquer jeito. Na b, dá para mostrar com números: trazer 300 mil linhas em vez de 200 milhões.

2. Certo. Só um ajuste de palavra: a carga não traz um pedido novo, traz o mesmo pedido de novo, com o status atualizado. E a solução é o upsert (MERGE): se o pedido já existe no destino, atualiza; se não, insere.

3. Aqui está a armadilha. A data_criacao não muda quando o status muda: ela registra quando o pedido nasceu, e esse momento nunca se altera. Acompanhe:

Momento	O que acontece
há um mês	o pedido é criado, com data_criacao = há um mês
ontem	a carga incremental roda e guarda o watermark = ontem
hoje	o status do pedido muda, mas a data_criacao continua “há um mês”
hoje, na carga	o processo pede “pedidos com data_criacao depois de ontem”. Esse pedido tem data de um mês atrás, então fica de fora

Ou seja: com a data_criacao, a carga só enxerga pedidos novos, e nunca as alterações em pedidos antigos. O destino ficaria com o status desatualizado para sempre.

O que resolve é uma coluna que muda toda vez que a linha é alterada: normalmente chamada data_atualizacao (ou updated_at). Quando o status muda hoje, ela passa a valer hoje, e o pedido entra na carga.    

## Schema-on-write × schema-on-read


“Schema” é a estrutura de uma tabela: quais colunas existem e de que tipo é cada uma. A diferença entre os dois modelos está em quando essa estrutura é aplicada aos dados.

Schema-on-write (estrutura na escrita)
É o modelo dos bancos de dados e dos warehouses, como o seu PostgreSQL da Olist. Antes de gravar, você cria a tabela com as colunas e os tipos. Na hora de inserir, o banco confere cada linha: se uma data vier como “abc”, ele recusa.

Vantagem: o que está gravado é sempre válido. Quem consulta confia na estrutura.
Desvantagem: é rígido. Antes de guardar qualquer dado, alguém precisa definir a estrutura, e dados fora do padrão são barrados.

Schema-on-read (estrutura na leitura)
É o modelo do data lake. Os arquivos são gravados como chegam, sem conferência. A estrutura só é aplicada na hora de ler: você descreve como interpretar o arquivo (quais colunas, de que tipo), e a ferramenta aplica essa descrição ao consultar.

Vantagem: é flexível. Você guarda qualquer coisa agora e decide depois como ler.
Desvantagem: os problemas só aparecem na leitura. Se uma linha tiver “abc” no lugar da data, a consulta é que vai falhar ou devolver vazio.

A ligação com a AWS

É assim que o Athena consulta os CSVs da raw sem eles terem sido “carregados” em um banco: você registra no Glue Data Catalog uma tabela que descreve os arquivos (onde estão no S3, quais colunas, de que tipo), e o Athena aplica essa descrição a cada consulta. Os arquivos continuam lá, intocados. Esse foi o passo 4 do roteiro do projeto: “catalogar a raw”.

E repare em como isso se encaixa nas camadas: a raw é schema-on-read (guarda o que chega, sem barrar nada), e a passagem para a trusted é onde os tipos são conferidos e corrigidos. As tabelas Iceberg da trusted funcionam como schema-on-write: o schema faz parte da tabela, e a gravação respeita os tipos.

Qual a diferença entre os dois modelos, em uma frase?
    um é rigido e outro flexivel
Um CSV chega na raw com uma linha em que o valor pago é “N/A”. O que acontece em cada modelo?
    no Schema-on-write ele recusa, Schema-on-read você descreve como interpretar

Por que faz sentido a raw ser schema-on-read e a trusted ser schema-on-write?
    schema-on-read guarda o que chega, sem barrar nada, schema-on-write o schema faz parte da tabela, e a gravação respeita os tipos
    
Por que a raw combina com schema-on-read e a trusted com schema-on-write? Use as duas perguntas-guia da mensagem anterior (a regra de ouro da raw, e o que quem usa a trusted precisa).
    por ela prioriza a preservação total dos dados ja a write presenta a consistencia e leitura dos dados.