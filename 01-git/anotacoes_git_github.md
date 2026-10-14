# Antes: o que é o Git, em uma frase

O Git tira “fotos” da sua pasta ao longo do tempo. Cada foto se chama commit e guarda como os arquivos estavam naquele momento, com uma mensagem dizendo o que mudou. Assim, você pode ver o histórico, comparar versões e voltar atrás se algo quebrar. O GitHub é um site que guarda uma cópia desse histórico na internet, onde outras pessoas (e recrutadores) podem ver.

# Passo 1: confirmar a instalação

No terminal, nessa pasta mesmo, rode:
git --version

resultado:
PS C:\Users\PC-Tiago\Documents\Estudo de Python> git --version 
git version 2.54.0.windows.1

# Passo 2: dizer ao Git quem é você
Cada commit guarda o nome e o e-mail de quem o fez. É assim que, num time, todo mundo sabe quem mudou o quê. Essa configuração é feita uma vez só no computador.

# Primeiro, veja se já existe alguma configuração:
git config --global user.name
git config --global user.email

Primeiro, veja se já existe alguma configuração:
PS C:\Users\PC-Tiago\Documents\Estudo de Python> 
PS C:\Users\PC-Tiago\Documents\Estudo de Python> git config --global user.name             
tiagoalvessw
PS C:\Users\PC-Tiago\Documents\Estudo de Python> git config --global user.email
tiagoalvessw@outlook.com
PS C:\Users\PC-Tiago\Documents\Estudo de Python>

Um detalhe extra: sem o --global, a configuração vale só para o repositório em que você está. Isso é útil quando alguém usa um e-mail pessoal nos projetos próprios e o e-mail da empresa nos do trabalho, no mesmo computador.

# Passo 3: transformar a pasta em repositório
git init,  Deve aparecer uma mensagem como “Initialized empty Git repository in...”.

Esse comando cria uma pasta oculta chamada .git dentro da sua pasta. É lá que o Git vai guardar todas as “fotos” (commits). Nada nos seus arquivos muda. A partir daí, o Git passa a observar a pasta.

# Passo 4: dizer ao Git o que ignorar
Antes de qualquer commit, crie no VS Code um arquivo chamado exatamente .gitignore (com o ponto no começo e sem extensão), na raiz da pasta Estudo de Python. Cada linha dele é um padrão de arquivos que o Git deve ignorar. Coloque:

*.csv
__pycache__/

*.csv ignora todos os arquivos que terminam em .csv. O * significa “qualquer nome”. Assim, os dados da Olist ficam só no seu computador.
__pycache__/ ignora uma pasta de arquivos temporários que o Python às vezes cria.

Pense também no anotacoes.md: se quiser que ele fique público junto com o código, deixe como está; se preferir mantê-lo só para você, acrescente uma linha com o nome dele.

# Passo 5: 
rode git status e cole aqui o resultado. Devem aparecer, como “untracked”, o .gitignore, o README.md e a pasta 01-git/.

C:\trilha-plataforma-dados>> git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore
        01-git/
        README.md

nothing added to commit but untracked files present (use "git add" to track)
C:\trilha-plataforma-dados>> 

Funcionou: o repositório existe, e o Git está vendo os três itens. Lendo a saída linha por linha:

On branch master: você está na linha principal do histórico. (Hoje o padrão do GitHub é chamar essa linha de main; vamos ajustar isso quando formos publicar.)
No commits yet: ainda não há nenhuma foto tirada.
Untracked files: arquivos que o Git vê, mas ainda não acompanha.

Como funciona um commit: as três etapas

O Git separa o caminho de um arquivo até a foto em três lugares:

A pasta de trabalho: onde você edita os arquivos normalmente.
A área de preparação (staging): onde você coloca o que quer incluir na próxima foto. É como escolher quais pessoas entram no enquadramento.
O histórico (commit): a foto tirada, guardada com uma mensagem.

O git add leva os arquivos da etapa 1 para a 2, e o git commit tira a foto do que está na etapa 2.

# Passo 6: preparar
git add .

O ponto significa “tudo nesta pasta” (respeitando o .gitignore). Depois rode git status de novo: os arquivos devem aparecer em verde, em “Changes to be committed”.

C:\trilha-plataforma-dados>> git add .
warning: in the working copy of 'README.md', LF will be replaced by CRLF the next time Git touches it
C:\trilha-plataforma-dados>>

Isso é só um aviso, não um erro. O git add funcionou.

O que ele quer dizer: arquivos de texto marcam o fim de cada linha com um caractere invisível, e cada sistema usa um diferente. O Linux e o Mac usam o LF, e o Windows usa o CRLF. O seu README.md foi criado num ambiente Linux (o arquivo que eu gerei), então veio com LF. O Git do Windows está configurado para converter automaticamente para CRLF quando mexer no arquivo, e só está avisando que vai fazer isso.

Para o seu trabalho, não muda nada: o conteúdo do arquivo continua igual. É um aviso que você vai ver com frequência no Windows, e pode ignorar.

Siga com o git status (os arquivos devem aparecer em verde) e depois com o git commit. Cole os resultados.

PS C:\trilha-plataforma-dados> git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   .gitignore
        new file:   01-git/anotacoes_git_github.md
        new file:   README.md

C:\trilha-plataforma-dados> 

# Passo 7: tirar a foto
git commit -m "Inicia a trilha de plataforma de dados"

Para conferir, vamos conhecer um comando novo, que mostra o histórico de commits:
git log --oneline
O --oneline mostra cada commit numa linha só: o código dele e a mensagem.

PS C:\trilha-plataforma-dados> git log --oneline
8cafe2d (HEAD -> main) primeiro commit - trilha de estudos para plataforma de dados
c4230aa (origin/main) Remove caminho local das anotações
6eb8385 Inicia a trilha de plataforma de dados
 


# Após todo o processo, os passos abaixo é para publicar no GitHub

# Passo 1: renomear a linha principal para main
git branch -M main

O seu Git criou a linha principal com o nome antigo, master. O GitHub (e a maioria das empresas) usa main hoje. Renomear agora evita confusão quando você conectar os dois. Confira com git status: a primeira linha deve dizer “On branch main”.

# Passo 2: criar o repositório no GitHub

No site do GitHub, clique no + no canto superior direito e em “New repository”. Preencha:

Repository name: trilha-plataforma-dados
Public, para aparecer no seu perfil.
Não marque as opções de adicionar README, .gitignore ou licença. O seu repositório local já tem os arquivos; se o GitHub criar outros, os dois históricos começam diferentes e dá conflito na hora de enviar.

Clique em “Create repository”. O GitHub vai mostrar uma página com o endereço do repositório, algo como https://github.com/seu-usuario/trilha-plataforma-dados.git.

# Passo 3: ligar o repositório local ao do GitHub
Digite: git remote add origin https://github.com/seu-usuario/trilha-plataforma-dados.git

Troque pelo endereço que o GitHub mostrou. O origin é um apelido para esse endereço, para você não ter que digitá-lo inteiro toda vez. “Remoto” é como o Git chama uma cópia do repositório que fica em outro lugar.

# Passo 4: enviar
git push -u origin main

O push envia os seus commits para o remoto. O -u grava a ligação entre a sua main e a main do GitHub; depois disso, basta digitar git push nas próximas vezes.

Na primeira vez, o Windows deve abrir uma janela pedindo para você entrar na sua conta do GitHub pelo navegador. É normal: é como o Git ganha permissão para enviar.

Ao terminar, abra a página do repositório no GitHub e atualize. O seu README.md deve aparecer na página. Cole aqui o resultado do git push.

# Passo 5: validação dos passos.
E confira com git remote -v.

Com isso, o repositório está publicado só com o README profissional, e o roteiro pessoal fica guardado com você.


# resumo de tudo feito até o momento:

Tudo limpo: um único commit no histórico, publicado, e o remoto com o endereço correto. O roteiro antigo não existe mais no GitHub.

Esse imprevisto acabou ensinando bastante coisa que aparece no dia a dia:

O histórico guarda tudo. Apagar ou editar um arquivo num commit novo não remove as versões antigas. Por isso, o cuidado com o que entra no repositório vem antes do primeiro commit (é para isso que serve o .gitignore).
Local e remoto são cópias separadas. Uma edição feita no site do GitHub não aparece no seu computador até um git pull.
Comandos novos: git pull, git remote -v, git remote set-url, git branch -M e a configuração init.defaultBranch.


# Teste para verificar o historico fic

O ponto foi entender que o histórico vive na pasta .git e que cada commit é uma foto permanente: enquanto ela não era apagada, os commits antigos voltavam a cada envio. Um exercício curto, só local, sem GitHub, para ver isso com os próprios olhos:

Crie uma pasta teste-git fora da trilha, rode git init e crie um arquivo segredo.txt com uma frase qualquer. Faça um commit.
Apague a frase do arquivo, salve e faça outro commit.
Rode git log --oneline e copie o código do primeiro commit.
Rode git show codigo-do-primeiro-commit (trocando pelo código).

Repare no que o git show mostra, e me diga o que concluiu. Depois, pode apagar a pasta teste-git inteira.
 
PS C:\Users\PC-Tiago\Documents\teste-git> git add .
PS C:\Users\PC-Tiago\Documents\teste-git> 
PS C:\Users\PC-Tiago\Documents\teste-git> git commit -m "Adiciona a frase secreta"
[master 6e9c919] Adiciona a frase secreta
 1 file changed, 1 insertion(+)
PS C:\Users\PC-Tiago\Documents\teste-git> git add .                               
PS C:\Users\PC-Tiago\Documents\teste-git> 
PS C:\Users\PC-Tiago\Documents\teste-git> git commit -m "Remove a frase secreta"  
[master 22662b9] Remove a frase secreta
 1 file changed, 1 deletion(-)
PS C:\Users\PC-Tiago\Documents\teste-git> git log --oneline
22662b9 (HEAD -> master) Remove a frase secreta
6e9c919 Adiciona a frase secreta
d822aeb teste ver isso com os próprios olhos
PS C:\Users\PC-Tiago\Documents\teste-git> git show 6e9c919
commit 6e9c919a9659990c787bd638ae16a09f85a6c97a
Author: tiagoalvessw <tiagoalvessw@outlook.com>
Date:   Wed Oct 14 10:49:30 2026 -0300

    Adiciona a frase secreta

diff --git a/segredo.txt b/segredo.txt
index e69de29..b16d477 100644
--- a/segredo.txt
+++ b/segredo.txt
@@ -0,0 +1 @@
+segredo
\ No newline at end of file
PS C:\Users\PC-Tiago\Documents\teste-git> 



Apague a frase, salve, e faça git add . e git commit -m "Remove a frase secreta".
Rode git log --oneline: devem aparecer três commits. Pegue o código do commit “Adiciona a frase secreta”.
Rode git show com esse código.

 PS C:\Users\PC-Tiago\Documents\teste-git> git add .
PS C:\Users\PC-Tiago\Documents\teste-git> 
PS C:\Users\PC-Tiago\Documents\teste-git> git commit -m "Adiciona a frase secreta"
[master 6e9c919] Adiciona a frase secreta
 1 file changed, 1 insertion(+)
PS C:\Users\PC-Tiago\Documents\teste-git> git add .                               
PS C:\Users\PC-Tiago\Documents\teste-git> 
PS C:\Users\PC-Tiago\Documents\teste-git> git commit -m "Remove a frase secreta"  
[master 22662b9] Remove a frase secreta
 1 file changed, 1 deletion(-)
PS C:\Users\PC-Tiago\Documents\teste-git> git log --oneline
22662b9 (HEAD -> master) Remove a frase secreta
6e9c919 Adiciona a frase secreta
d822aeb teste ver isso com os próprios olhos
PS C:\Users\PC-Tiago\Documents\teste-git> git show
commit 22662b93859fbd498fbbf48eed2749d271564982 (HEAD -> master)
Author: tiagoalvessw <tiagoalvessw@outlook.com>
Date:   Wed Oct 14 10:50:25 2026 -0300

    Remove a frase secreta

diff --git a/segredo.txt b/segredo.txt
index b16d477..e69de29 100644
--- a/segredo.txt
+++ b/segredo.txt
@@ -1 +0,0 @@
-segredo
\ No newline at end of file
PS C:\Users\PC-Tiago\Documents\teste-git>


Agora o exercício mostrou exatamente o ponto. Você rodou git show sem código, e nesse caso ele mostra o commit mais recente, o “Remove a frase secreta”. E repare no que aparece no final:

-segredo

No diff, as linhas com - são as que foram removidas, e as com + são as que foram adicionadas. Ou seja: o próprio commit que apaga o segredo mostra o segredo. Para remover a frase, o Git registra qual era a frase removida.

Se você rodar git show 6e9c919, vai ver o outro lado: o commit “Adiciona a frase secreta”, com a linha +segredo.

A conclusão é a mesma do roteiro na trilha: o arquivo hoje está vazio, mas a frase continua acessível no histórico de duas formas, no commit que a adicionou e no commit que a removeu. Enquanto existir a pasta .git com esses commits, o segredo existe.

Isso tem uma consequência muito prática para o seu projeto na AWS: se uma chave de acesso da AWS for parar num commit, mesmo que você apague no commit seguinte, ela deve ser considerada vazada. O certo, nesse caso, é desativar a chave no console da AWS e criar outra. Por isso as credenciais nunca ficam no código: elas ficam na configuração do AWS CLI, fora do repositório.

=============================================
###### Branches e pull requests ######
=============================================

O que é uma branch

Até agora, todos os seus commits foram numa linha só, a main. Uma branch é uma linha paralela: você cria uma cópia do ponto atual, trabalha nela à vontade, e a main continua intacta. Quando o trabalho está pronto, você junta a branch de volta na main. Juntar se chama merge.

Por que isso importa em times de plataforma: a main é a versão “oficial”, às vezes a que está rodando em produção. Ninguém mexe nela diretamente. Cada mudança nasce numa branch.

O que é um pull request (PR)

É o pedido, feito no GitHub, para juntar uma branch na main. No PR, outras pessoas leem as mudanças, comentam e aprovam, e só então o merge acontece. É assim que o código é revisado nas empresas. Quando você trabalha sozinho, o PR serve como registro organizado de cada mudança, e é um ótimo treino do fluxo real.

O fluxo completo
criar branch → editar → commit → enviar a branch → abrir PR → merge → atualizar a main local

Parte 1: criar a branch e trabalhar nela

Na pasta da trilha:
Criar a branch e entrar nela:
   git switch -c anotacoes-branches

resultado
PS C:\Users\PC-Tiago\Documents\trilha-plataforma-dados>    git switch -c anotacoes-branches
Switched to a new branch 'anotacoes-branches'   

O switch troca de branch, e o -c cria uma nova com esse nome. Confira com git branch: ele lista as branches, e a atual aparece com um *.

PS C:\Users\PC-Tiago\Documents\trilha-plataforma-dados> git branch
* anotacoes-branches
  main
PS C:\Users\PC-Tiago\Documents\trilha-plataforma-dados> 


Editar: no anotacoes_git_github.md, escreva o que aprendeu sobre branches e PR até aqui, com as suas palavras. Salve.
Commit, como sempre: git add . e git commit -m "...", com uma mensagem que descreva a mudança.
Enviar a branch para o GitHub:
   git push -u origin anotacoes-branches

resultado:

PS C:\Users\PC-Tiago\Documents\trilha-plataforma-dados\01-git> git push -u origin anotacoes-branches
Enumerating objects: 11, done.
Counting objects: 100% (11/11), done.
Delta compression using up to 24 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (8/8), 2.72 KiB | 928.00 KiB/s, done.
Total 8 (delta 2), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (2/2), completed with 1 local object.
remote:
remote: Create a pull request for 'anotacoes-branches' on GitHub by visiting:
remote:      https://github.com/tiagoalvessw/trilha-plataforma-dados/pull/new/anotacoes-branches
remote:
To https://github.com/tiagoalvessw/trilha-plataforma-dados.git
 * [new branch]      anotacoes-branches -> anotacoes-branches
branch 'anotacoes-branches' set up to track 'origin/anotacoes-branches'.
PS C:\Users\PC-Tiago\Documents\trilha-plataforma-dados\01-git> 

Enviado! E repare que o próprio GitHub já respondeu com um convite: “Create a pull request for ‘anotacoes-branches’”, com o link pronto. Esse é o fluxo normal: você envia a branch, e o GitHub sugere abrir o PR.

Parte 2: o pull request

Abra o link que apareceu na saída (.../pull/new/anotacoes-branches).
Confira a direção: no topo, deve aparecer base: main ← compare: anotacoes-branches. Lê-se “juntar a anotacoes-branches na main”.
Título e descrição: o título é uma frase curta sobre a mudança (por exemplo, “Adiciona anotações sobre branches e pull requests”). Na descrição, explique em duas ou três linhas o que foi feito. Em times, é aqui que você conta ao revisor o que ele vai encontrar.
Clique em Create pull request.
Revise as mudanças: abra a aba Files changed. Você vai ver o mesmo diff do git show: linhas com + em verde (adicionadas) e com - em vermelho (removidas). É essa tela que um revisor lê. Repare que aparecem os dois commits: o do teste do histórico (que estava só na sua main local) e o das anotações.
Faça o merge: volte à aba Conversation, clique em Merge pull request e em Confirm merge.
O GitHub vai oferecer Delete branch. Pode apagar: a branch já cumpriu o papel, e os commits dela agora estão na main.

Parte 3: atualizar o seu computador

O merge aconteceu no GitHub, então a sua main local ainda não sabe dele. No terminal:

git switch main
git pull
git branch -d anotacoes-branches
o switch main volta para a linha principal;
o pull traz o merge do GitHub;
o branch -d apaga a branch local, que já foi juntada.

No final, rode git log --oneline e cole o resultado.

PS C:\Users\PC-Tiago\Documents\trilha-plataforma-dados\01-git> git log --oneline 
91ec1ef (HEAD -> main, origin/main, origin/HEAD) Merge pull request #1 from tiagoalvessw/anotacoes-branches
17dac6c (origin/anotacoes-branches) Aprendendo sobre ###### Branches e pull requests ######
3e63cf4 Acompanhando historico de alterações de arquivo - Teste para verificar o historico fica
7171337 Inicia a trilha de plataforma de dados
PS C:\Users\PC-Tiago\Documents\trilha-plataforma-dados\01-git> 

Fluxo completo! Lendo o histórico de baixo para cima:

7171337: o primeiro commit da trilha;
3e63cf4 e 17dac6c: os dois commits que vieram pela branch;
91ec1ef: o merge commit, “Merge pull request #1”. É o commit que o GitHub cria ao juntar a branch na main. O “#1” é o número do seu primeiro pull request.

E o (HEAD -> main, origin/main) na mesma linha mostra que a sua main local e a do GitHub estão iguais.

Um detalhe: ainda aparece origin/anotacoes-branches. É uma lembrança local de como a branch estava no GitHub. Se você apagou a branch no site, essa lembrança ficou desatualizada. Para limpar:

git fetch --prune

Resultado:
PS C:\Users\PC-Tiago\Documents\trilha-plataforma-dados\01-git> git fetch --prune
From https://github.com/tiagoalvessw/trilha-plataforma-dados
 - [deleted]         (none)     -> origin/anotacoes-branches
PS C:\Users\PC-Tiago\Documents\trilha-plataforma-dados\01-git> 

O fetch busca as novidades do GitHub sem mexer nos seus arquivos, e o --prune remove as lembranças de branches que não existem mais lá.

Uma sugestão de estilo: as mensagens de commit ficam no histórico público, então vale mantê-las limpas e descritivas, sem os ######. Por exemplo, “Adiciona anotações sobre branches e pull requests”. É um detalhe que recrutadores notam ao olhar um repositório.

Com isso, o essencial do bloco de Git está feito: commits, .gitignore, histórico, remoto, push e pull, branches, pull request e merge. Você também passou por problemas reais, como o histórico com informação indesejada e o repositório com nome errado, que ensinam mais do que um tutorial sem imprevistos.

