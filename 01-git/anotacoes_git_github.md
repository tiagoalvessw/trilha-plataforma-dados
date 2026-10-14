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