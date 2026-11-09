**Data:** 21/08

# Aula 2: Introdução ao terminal para análise de sequências biológicas

## Prática síncrona

### Introdução ao Bash

Comandos abordados:

- `pwd`
- `ls`
- `cd`
- `mkdir`
- `cp`
- `mv`
- `rm`
- `cat`
- `head`
- `tail`
- `grep`

### Formato de arquivos

- FASTA

---

# Começando

Durante toda a disciplina utilizaremos o **terminal Linux (Bash)** para organizar arquivos, executar programas e analisar dados biológicos. Nesta primeira prática aprenderemos os principais comandos de navegação e manipulação de arquivos, utilizando um conjunto de proteínas obtido do banco de dados UniProt.

Ao final desta prática você será capaz de:

- navegar entre diretórios;
- criar e organizar pastas;
- copiar, mover e remover arquivos;
- visualizar o conteúdo de arquivos FASTA;
- pesquisar informações utilizando o comando `grep`.

---
> Os comandos abaixo devem ser digitados no terminal. Os trechos iniciados por `#` são comentários explicativos e não precisam ser executados.

## 1. Organizando o diretório da aula

Antes de iniciar qualquer análise, é importante manter os arquivos organizados.

Primeiro, crie um diretório com o seu nome. Todas as atividades da disciplina serão realizadas dentro desse diretório.

Para criar um diretório utilizamos o comando `mkdir`, abreviação de **Make Directory** ("criar diretório").

```bash
mkdir Daisy
```

Substitua **Daisy** pelo seu nome.

---

Agora vamos verificar se o diretório foi criado.

Para listar os arquivos e diretórios utilizamos o comando `ls`, abreviação de **List**.

```bash
ls
```

Você deverá observar algo semelhante a:

```text
Daisy
```

---

Agora entre no diretório recém-criado utilizando o comando `cd`, abreviação de **Change Directory** ("mudar de diretório").

```bash
cd Daisy
```

Para confirmar em qual diretório você está, utilize:

```bash
pwd
```

O comando `pwd` significa **Print Working Directory** e mostra o caminho completo do diretório atual.

---

Vamos verificar se o diretório está vazio.

```bash
ls
```

Como ele acabou de ser criado, nenhuma informação deverá ser exibida.

---

## Outras formas de utilizar o comando `ls`

O comando `ls` possui diversas opções.

### Listagem detalhada

```bash
ls -l
```

Mostra informações detalhadas sobre cada arquivo:

- permissões;
- proprietário;
- tamanho;
- data de modificação.

---

### Mostrar arquivos ocultos

```bash
ls -a
```

Arquivos iniciados por um ponto (`.`) são considerados ocultos no Linux.

---

### Listagem detalhada incluindo arquivos ocultos

```bash
ls -la
```

Combina as opções `-l` e `-a`.

---

### Ordenar por data de modificação

```bash
ls -lrth
```

Onde:

- `-l` → formato longo (*long listing*);
- `-r` → ordem inversa (*reverse*);
- `-t` → ordena pela data de modificação (*time*);
- `-h` → mostra tamanhos em formato legível (*human readable*), como KB, MB e GB.

---

## Criando um diretório para a Aula 2 - parte 1

Dentro do diretório do aluno, vamos criar uma pasta específica para armazenar os arquivos desta aula.

```bash
mkdir aula2_parte1
```

Verifique se ela foi criada.

```bash
ls -lrth
```

---

Agora entre no diretório.

```bash
cd aula2_parte1
```

💡 **Dica**

O terminal possui autocompletar utilizando a tecla **TAB**.

Por exemplo, basta digitar

```bash
cd au
```

e pressionar **TAB**.

O terminal completará automaticamente para

```bash
cd aula2_parte1
```

Esse recurso economiza tempo e evita erros de digitação.

---

## 2. Baixando proteínas do UniProt

Agora utilizaremos um arquivo real contendo proteínas.

1. Acesse [UniProt](https://www.uniprot.org/).
2. Na barra de pesquisa, procure por um organismo, por exemplo *Escherichia coli*.
3. Filtre ou selecione as proteínas **revisadas** (*reviewed*).
4. Clique em **Download** e escolha o formato **FASTA**. Em seguida marque a opção de compactar (*compressed*) e baixe o arquivo.

Após o download, abra a pasta atual do terminal WSL utilizando:

```bash
explorer.exe .
```

O ponto (`.`) representa o diretório atual.

Esse comando facilita copiar ou mover arquivos entre o Windows e o WSL.

Após copiar o arquivo para o diretório da aula, utilize:

```bash
ls
```

Você deverá observar algo semelhante a:

```text
uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta.gz
uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta.gz:Zone.Identifier
```

O arquivo `:Zone.Identifier` pode aparecer quando um arquivo é baixado pelo Windows. Ele contém apenas informações de segurança do sistema operacional e **não faz parte do arquivo FASTA**.

Podemos removê-lo com segurança utilizando:

```bash
rm uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta.gz:Zone.Identifier
```

Confirme que restou apenas o arquivo FASTA compactado.

```bash
ls
```

---

## 3. Descompactando o arquivo

O arquivo foi baixado compactado (`.gz`).

Para descompactá-lo utilizamos o comando `gunzip` (**GNU unzip**).

```bash
gunzip uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta.gz
```

Esse comando remove o arquivo compactado e mantém apenas a versão descompactada.

Caso deseje manter também a versão compactada, utilize:

```bash
gzip -dk uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta.gz
```

onde:

- `-d` → descompacta (*decompress*);
- `-k` → mantém (*keep*) o arquivo original.

---

## 4. Entendendo e visualizando um arquivo FASTA

O formato FASTA armazena sequências biológicas. Cada proteína normalmente possui:

- uma linha de cabeçalho, iniciada por `>`;
- uma ou mais linhas com a sequência de aminoácidos.

Exemplo:

```text
>sp|P0A7V8|RS2_ECOLI 30S ribosomal protein S2 OS=Escherichia coli
MARGKKIGYS...
```

No cabeçalho, `sp` indica uma entrada revisada do UniProt/Swiss-Prot. A sequência abaixo dele é composta pelos códigos de uma letra dos aminoácidos.

Use `cat` (*concatenate*) para imprimir o arquivo inteiro:

```bash
cat uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta
```

Em arquivos grandes, prefira `head` (*cabeça*) e `tail` (*cauda*), que mostram, por padrão, as 10 primeiras e as 10 últimas linhas:

```bash
head uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta
tail uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta
```

Para escolher a quantidade de linhas, use `-n`:

```bash
head -n 20 uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta
tail -n 20 uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta
```

## 5. Renomeando, movendo e removendo arquivos

Um nome curto torna os comandos mais fáceis de ler. `mv` vem de *move* e serve tanto para renomear como para mover arquivos.
Para renomear:

```bash
mv uniprotkb_Escherichia_coli_AND_reviewed_2026_07_28.fasta ecoli_reviewed.fasta
ls
```

Para mover um arquivo, informe uma pasta como destino:

```bash
mkdir nova_pasta
mv ecoli_reviewed.fasta nova_pasta/
ls
```

Entre na pasta e confirme que o arquivo foi movido:

```bash
cd nova_pasta
ls
```

`..` representa o diretório imediatamente anterior (a pasta-pai). Portanto, este comando devolve o arquivo à pasta `aula1`:

```bash
mv ecoli_reviewed.fasta ../
```

E este comando retorna para a pasta-pai:

```bash
cd ..
ls
```

Para voltar dois níveis, use `../..`; para três, `../../..`.

Para remover a pasta vazia criada no exemplo:

```bash
rm -r nova_pasta
```

`rm` vem de *remove*. A opção `-r` significa *recursive*: ela percorre a pasta e remove seu conteúdo junto com a própria pasta. Use-a com cuidado, pois a remoção pelo terminal normalmente não vai para a Lixeira.

## 6. Buscando informações com `grep`

`grep` procura linhas que correspondem a um padrão de texto. A forma geral é:

```bash
grep "padrão" arquivo
```

Em um FASTA, cada proteína começa em uma linha de cabeçalho iniciada por `>`. Para mostrar todos os cabeçalhos:

```bash
grep "^>" ecoli_reviewed.fasta
```

O símbolo `^` significa “início da linha”; assim, `^>` busca somente linhas cujo primeiro caractere é `>`.

### Contar proteínas

Como há um cabeçalho por proteína, conte os cabeçalhos com `-c`, de *count* (contar):

```bash
grep -c "^>" ecoli_reviewed.fasta
```

### Procurar uma proteína pelo nome

```bash
grep "polymerase" ecoli_reviewed.fasta
```

Por padrão, a busca diferencia letras maiúsculas e minúsculas. Use `-i` para ignorar essa diferença (*ignore case*):

```bash
grep -i "polymerase" ecoli_reviewed.fasta
```

### Exibir contexto ao redor de uma ocorrência

As sequências vêm após o cabeçalho. `-A` (*after*) mostra linhas **depois** da correspondência; `-B` (*before*) mostra linhas **antes** dela:

```bash
grep -A 1 "rpoB" ecoli_reviewed.fasta
grep -A 10 "rpoB" ecoli_reviewed.fasta
grep -B 1 -A 2 "ATPase" ecoli_reviewed.fasta
```

> Atenção: proteínas longas podem ocupar muitas linhas. `-A 10` não garante a sequência completa; ele apenas mostra as 10 linhas seguintes.

### Procurar mais de um padrão

`-E` ativa expressões regulares estendidas. Nesse modo, `|` significa “ou”:

```bash
grep -E "ATPase|polymerase|helicase" ecoli_reviewed.fasta
```

Assim, o comando mostra linhas que contêm **ATPase**, ou **polymerase** ou **helicase**.

### Mostrar linhas que não contêm um termo

`-v` vem de *invert match*: inverte a seleção e exibe as linhas que **não** correspondem ao padrão, ou seja, mostra as linhas que NÃO contêm a palavra.

```bash
grep -v "hypothetical" ecoli_reviewed.fasta
```

Em FASTA, esse comando também mostrará linhas de sequência, pois elas normalmente não possuem a palavra pesquisada. Para trabalhar apenas com cabeçalhos, faça primeiro a seleção de linhas iniciadas em `>`.

```bash
grep "^>" ecoli_reviewed.fasta | grep -v "hypothetical"
```

## 7. Pesquisando vários genes a partir de uma lista

Crie um arquivo de padrões com `nano`, um editor de texto do terminal:

```bash
nano genes.txt
```

Digite um termo por linha:

```text
rpoB
gyrA
dnaK
```

Para salvar no `nano`, pressione `Ctrl + X`, depois `Y` e `Enter`. Confira o arquivo:

```bash
ls
cat genes.txt
```

Use `grep -f` para ler os padrões de um arquivo (*file*):

```bash
grep -f genes.txt ecoli_reviewed.fasta
```

Para contar as linhas retornadas, use o operador `|`, chamado *pipe*. Ele envia a saída do comando à esquerda para o comando à direita. `wc` significa *word count* e `-l` conta linhas:

```bash
grep -f genes.txt ecoli_reviewed.fasta | wc -l
```

> Esse total representa o número de linhas encontradas, não necessariamente o número de proteínas, pois uma proteína pode ocupar várias linhas no arquivo FASTA.
> Para contar apenas as proteínas correspondentes aos padrões de genes.txt, primeiro filtramos os cabeçalhos FASTA, que começam com >, e depois contamos essas linhas:
```bash
grep -f genes.txt ecoli_reviewed.fasta | grep "^>" | wc -l
```
- grep -f genes.txt → procura os padrões listados em genes.txt;
- grep "^>" → mantém apenas os cabeçalhos das proteínas;
- wc -l → conta quantos cabeçalhos foram encontrados.

## 8. Exemplos finais de filtragem

Contar cabeçalhos que incluem “hypothetical”:

```bash
grep "^>" ecoli_reviewed.fasta | grep -ic "hypothetical"
```

O primeiro `grep` limita a saída aos cabeçalhos; o segundo procura o termo e `-c` conta as linhas. A opção `-i` torna a busca independente de maiúsculas e minúsculas.

Entradas revisadas do UniProt usam o prefixo `sp|`:

```bash
grep "^>sp|" ecoli_reviewed.fasta
grep -c "^>sp|" ecoli_reviewed.fasta
```

Entradas não revisadas usam `tr|` (TrEMBL):

```bash
grep "^>tr|" ecoli_reviewed.fasta
grep -c "^>tr|" ecoli_reviewed.fasta
```

Se o arquivo foi baixado com o filtro **reviewed**, é esperado que a busca por `tr|` retorne zero resultados.

## Comandos básicos com SeqKit

### Visualizar as primeiras sequências

```bash
seqkit head -n 5 ecoli_reviewed.fasta
```

### Obter estatísticas do FASTA
```bash
seqkit stats ecoli_reviewed.fasta
```

### Contar o número de proteínas
```bash
seqkit fx2tab -n ecoli_reviewed.fasta | wc -l
```

### Selecionar proteínas com pelo menos 500 aminoácidos
```bash
seqkit seq -m 500 ecoli_reviewed.fasta
```

### Selecionar proteínas com pelo menos (no mínimo 'm') 500 aminoácidos e salvar em novo arquivo
```bash
seqkit seq -m 500 ecoli_reviewed.fasta > proteinas_500aa.fasta
```

## Qual comando para contar proteínas do novo arquivo filtrado 'proteinas_500aa.fasta'?

### Selecionar proteínas com no máximo ('M') 100 aminoácidos
```bash
seqkit seq -M 100 ecoli_reviewed.fasta > proteinas_ate100aa.fasta
```

### Selecionar proteínas no mínimo 101 aa + no máximo 499 aa
```bash
seqkit seq -m 101 -M 499 ecoli_reviewed.fasta > proteinas_101_499aa.fasta
```

### Remover sequências duplicadas
```bash
seqkit rmdup ecoli_reviewed.fasta
```







