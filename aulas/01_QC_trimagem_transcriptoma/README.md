# Análise de RNA-seq em *Drosophila melanogaster*

**Análise de qualidade e trimagem em dados de RNA-Seq**

## Objetivo

Esta atividade tem como objetivo realizar uma análise inicial de dados de RNA-seq de *Drosophila melanogaster*, utilizando o sistema WSL Ubuntu.

Ao longo da atividade, serão realizadas as seguintes etapas:

- Download do genoma de referência e do arquivo de anotação;
- Download de reads de RNA-seq paired-end;
- Organização dos arquivos e identificação das condições experimentais;
- Controle de qualidade das reads;
- Trimagem de adaptadores e reads de baixa qualidade;

## Organização dos diretórios

Todos os arquivos da análise serão organizados em um diretório denominado `rna-seq`.

## 1. Preparação do ambiente

O primeiro passo é abrir o terminal do WSL Ubuntu.

Caso ainda não exista um diretório com seu nome, crie-o (substitua 'Daisy' pelo seu nome):

```bash
mkdir Daisy
```

Entre no diretório:

```bash
cd Daisy
```

Crie o diretório principal da análise de RNA-seq:

```bash
mkdir rna-seq
cd rna-seq
```

---

## 2. Download dos dados genômicos de referência

O primeiro passo da análise é obter o genoma de referência e seu respectivo arquivo de anotação.

Para isso, criaremos o diretório `0_referencia`.

```bash
mkdir 0_referencia
cd 0_referencia
```

### 2.1. Obtenção do genoma de referência

Os dados de referência serão obtidos no NCBI (National Center for Biotechnology Information).

Acesse:

https://www.ncbi.nlm.nih.gov/datasets/genome/

Pesquise por *Drosophila melanogaster*. O assembly utilizado nesta atividade será:

**GCF_000001215.4 – Release 6 plus ISO1 MT**

Clique na coluna 'assembly'. Na nova página aberta, acesse clicando na opção FTP (perto de download) e localize o arquivo do genoma:

`GCF_000001215.4_Release_6_plus_ISO1_MT_genomic.fna.gz`

Copie o endereço (link) do arquivo (com botão direito do mouse), volte ao terminal WSL e faça o download utilizando o comando `wget` + endereço do arquivo.

```bash
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/215/GCF_000001215.4_Release_6_plus_ISO1_MT/GCF_000001215.4_Release_6_plus_ISO1_MT_genomic.fna.gz
```

### 2.2. Obtenção do arquivo de anotação

O arquivo GTF contém informações sobre a localização dos genes, transcritos e outras características genômicas. Da mesma forma, faça o download do arquivo de anotação correspondente ao mesmo assembly:

```bash
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/215/GCF_000001215.4_Release_6_plus_ISO1_MT/GCF_000001215.4_Release_6_plus_ISO1_MT_genomic.gtf.gz
```

Verifique os arquivos baixados:

```bash
ls -lh
```

### 2.3. Descompactação dos arquivos

Os arquivos foram baixados no formato `.gz`, que corresponde a arquivos compactados.

Para descompactá-los, utilizaremos o comando `gunzip`:

```bash
gunzip GCF_000001215.4_Release_6_plus_ISO1_MT_genomic.fna.gz

gunzip GCF_000001215.4_Release_6_plus_ISO1_MT_genomic.gtf.gz
```

### 2.4. Renomeação dos arquivos

Os arquivos originais possuem nomes longos, dificultando sua utilização nos comandos das próximas etapas.
Para facilitar a identificação, utilizaremos nomes mais curtos e informativos. Para isso, usaremos 'mv'.
O comando `mv` é utilizado para mover ou renomear arquivos. Neste caso, estamos apenas modificando seus nomes.

```bash
# Renomear o genoma de referência
mv GCF_000001215.4_Release_6_plus_ISO1_MT_genomic.fna Dmel_genome.fasta

# Renomear o arquivo de anotação
mv GCF_000001215.4_Release_6_plus_ISO1_MT_genomic.gtf Dmel_annotation.gtf
```

### 2.5. Verificação dos arquivos

```bash
ls -lh
```

Ao final desta etapa, teremos:

| Arquivo | Conteúdo |
|---|---|
| `Dmel_genome.fasta` | Genoma de referência de *Drosophila melanogaster* |
| `Dmel_annotation.gtf` | Anotação dos genes e suas coordenadas genômicas |

O arquivo `Dmel_genome.fasta` será utilizado para indexação e mapeamento das reads com HISAT2.

O arquivo `Dmel_annotation.gtf` será utilizado posteriormente na contagem de reads por gene com FeatureCounts.

Retorne ao diretório principal:

```bash
cd ..
```

---

## 3. Download das reads de RNA-seq

Nesta etapa, serão obtidos os dados de RNA-seq de *Drosophila melanogaster* os quais serão utilizados na análise.

Crie o diretório `1_reads`:

```bash
mkdir 1_reads
cd 1_reads
```

### 3.1. Obtenção dos dados no ENA

Os dados serão obtidos no ENA (European Nucleotide Archive), um banco de dados público que armazena informações de sequenciamento de DNA e RNA.
Acesse: https://www.ebi.ac.uk/ena/

Pesquise pelo número de acesso do projeto: 

**PRJNA613451**

### 3.2. Descrição do projeto

O estudo utilizou RNA-seq para determinar os perfis transcricionais de moscas inteiras (*whole flies*) criadas em duas condições alimentares:

- Dieta holídica enriquecida com glicose;
- Dieta holídica não modificada.

Moscas machos da linhagem `w1118`, recém-emergidas, foram mantidas por 20 dias a 25 °C em uma das duas condições alimentares.
A dieta enriquecida continha 50 g/L de glicose, enquanto a dieta não modificada correspondia à dieta holídica original.
O transcriptoma de moscas inteiras foi sequenciado, permitindo investigar alterações na expressão gênica associadas ao enriquecimento da dieta com glicose.

### 3.3. Organização experimental

O projeto apresenta seis bibliotecas de RNA-seq, distribuídas em dois grupos experimentais, cada um com três réplicas biológicas.

| Condição experimental | Descrição | Réplicas |
|---|---|---|
| Unmodified | Dieta holídica não modificada (controle) | 3 |
| Glucose | Dieta holídica enriquecida com glicose (tratamento) | 3 |

O sequenciamento foi realizado utilizando a estratégia **paired-end**, na qual cada fragmento de cDNA é sequenciado pelas duas extremidades.

Assim, cada amostra possui dois arquivos FASTQ:
- R1: reads obtidas da primeira extremidade;
- R2: reads obtidas da segunda extremidade.

### 3.4. Download das reads

Na página do projeto no ENA:

1. Localize as seis bibliotecas de sequenciamento (SRR).
2. Selecione todos os arquivos FASTQ correspondentes às bibliotecas.
3. Clique na opção verde `Get download script`.
4. Baixe o script disponibilizado pelo ENA.
5. Salve o arquivo com o nome `SRR.sh`.

### 3.5. Transferência do script para o WSL Ubuntu

Para abrir o diretório atual do terminal no Windows, utilize:

```bash
explorer.exe .
```

Copie o arquivo `SRR.sh` para a pasta aberta pelo Windows.

Feche a janela do Explorer e retorne ao terminal do Ubuntu.

Verifique se o arquivo está no diretório correto:

```bash
ls
```

Resultado esperado:

```text
SRR.sh
```

### 3.6. Verificação do script de download

Para visualizar o conteúdo do script, utilize:

```bash
cat SRR.sh
```

O arquivo contém os comandos `wget` necessários para baixar as reads de cada biblioteca.

Exemplo:

```bash
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR113/018/SRR11350618/SRR11350618_1.fastq.gz

wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR113/018/SRR11350618/SRR11350618_2.fastq.gz
```

O parâmetro `-nc` (*no-clobber*) evita que arquivos já existentes sejam sobrescritos durante o download.

### 3.7. Execução do script

Para executar o script e iniciar o download das reads, utilize:

```bash
bash SRR.sh
```

Aguarde até que todos os arquivos sejam baixados.

Verifique os arquivos obtidos:

```bash
ls -lh
```

Os arquivos estarão inicialmente identificados pelos seus códigos de acesso SRR.

---

## 4. Renomeação e organização das reads

Os arquivos FASTQ possuem nomes baseados nos códigos SRR, que não identificam diretamente as condições experimentais.

Para facilitar a organização da análise, os arquivos serão renomeados de acordo com os grupos experimentais.

Utilizaremos as seguintes siglas:

| Sigla | Significado | Condição |
|---|---|---|
| CT | Control | Dieta holídica não modificada |
| GL | Glucose | Dieta holídica enriquecida com glicose |

A identificação das réplicas será realizada utilizando os números `01`, `02` e `03`.

O sufixo `R1` indica a primeira extremidade sequenciada, enquanto `R2` indica a segunda extremidade.

### 4.1. Renomeação das amostras controle

```bash
# Unmodified - controle

mv SRR11350614_1.fastq.gz CT_01_R1.fastq.gz
mv SRR11350614_2.fastq.gz CT_01_R2.fastq.gz

mv SRR11350615_1.fastq.gz CT_02_R1.fastq.gz
mv SRR11350615_2.fastq.gz CT_02_R2.fastq.gz

mv SRR11350616_1.fastq.gz CT_03_R1.fastq.gz
mv SRR11350616_2.fastq.gz CT_03_R2.fastq.gz
```

### 4.2. Renomeação das amostras tratadas

```bash
# Glucose - tratamento

mv SRR11350617_1.fastq.gz GL_01_R1.fastq.gz
mv SRR11350617_2.fastq.gz GL_01_R2.fastq.gz

mv SRR11350618_1.fastq.gz GL_02_R1.fastq.gz
mv SRR11350618_2.fastq.gz GL_02_R2.fastq.gz

mv SRR11350619_1.fastq.gz GL_03_R1.fastq.gz
mv SRR11350619_2.fastq.gz GL_03_R2.fastq.gz
```

### 4.3. Verificação dos arquivos

```bash
ls -lh
```

Ao final, teremos os seguintes arquivos:

| Amostra | R1 | R2 |
|---|---|---|
| CT_01 | CT_01_R1.fastq.gz | CT_01_R2.fastq.gz |
| CT_02 | CT_02_R1.fastq.gz | CT_02_R2.fastq.gz |
| CT_03 | CT_03_R1.fastq.gz | CT_03_R2.fastq.gz |
| GL_01 | GL_01_R1.fastq.gz | GL_01_R2.fastq.gz |
| GL_02 | GL_02_R1.fastq.gz | GL_02_R2.fastq.gz |
| GL_03 | GL_03_R1.fastq.gz | GL_03_R2.fastq.gz |

A organização dos arquivos facilitará a identificação das amostras nas próximas etapas da análise.

Agora podemos voltar ao diretório anterior para inicio do controle de qualidade

```bash
cd ..
```

---

## 5. Controle de qualidade dos dados brutos de sequenciamento

Nesta etapa, será realizado o controle de qualidade (QC, *Quality Control*) das reads brutas, ou seja, dos arquivos FASTQ obtidos diretamente do ENA, antes de qualquer processamento.

Para isso, utilizaremos a ferramenta **FastQC**, que permite avaliar diferentes características das reads, como:

- Qualidade das bases ao longo das sequências;
- Distribuição dos valores de qualidade (Phred score);
- Distribuição do comprimento das reads;
- Conteúdo de GC;
- Presença de sequências adaptadoras;
- Presença de sequências super-representadas.

### 5.1. Ativação do ambiente de análise

Como as ferramentas foram instaladas no ambiente Conda denominado `rnaseq`, precisamos ativá-lo antes de iniciar a análise.

```bash
conda activate rnaseq
```

O terminal deverá apresentar o nome do ambiente ativo no início da linha:

```text
(rnaseq) daisy@Daisy:~/Daisy/rna-seq$
```

Podemos verificar os diretórios existentes:

```bash
ls
```

Resultado esperado:

```text
0_referencia  1_reads
```

### 5.2. Criação do diretório para armazenar os resultados

Para manter os resultados organizados, criaremos um diretório denominado `2_qc_brutos`, onde serão armazenados os relatórios gerados pelo FastQC.

```bash
mkdir 2_qc_brutos
```

Verifique se o diretório foi criado:

```bash
ls
```

Resultado esperado:

```text
0_referencia  1_reads  2_qc_brutos
```

### 5.3. Execução do FastQC

Primeiramente, podemos verificar se o FastQC está instalado corretamente:

```bash
fastqc --version
```

Para analisar um único arquivo, podemos utilizar o seguinte comando (não executar neste momento):

```bash
fastqc 1_reads/CT_01_R1.fastq.gz -o 2_qc_brutos/
```

**Parâmetros:**

- `fastqc`: executa a ferramenta de controle de qualidade;
- `1_reads/CT_01_R1.fastq.gz`: indica o arquivo FASTQ que será analisado;
- `-o`: especifica o diretório onde o relatório será salvo;
- `2_qc_brutos/`: diretório de saída.

Entretanto, como temos 12 arquivos FASTQ (seis amostras, cada uma com R1 e R2), podemos automatizar a análise utilizando um **loop**.

Loops são estruturas de programação que permitem executar repetidamente um conjunto de comandos para diferentes arquivos, sem precisar escrever um comando individual para cada amostra.

Utilizaremos o seguinte comando:

```bash
for i in 1_reads/*.fastq.gz; do fastqc -t 4 "$i" --outdir 2_qc_brutos/; done
```

**Entendendo o comando:**

- `for i in`: inicia o loop e define a variável `i`, que representará cada arquivo encontrado;
- `1_reads/*.fastq.gz`: indica que serão selecionados todos os arquivos com extensão `.fastq.gz` existentes no diretório `1_reads`;
- `do`: indica o início dos comandos que serão executados para cada arquivo;
- `fastqc`: executa a análise de qualidade;
- `-t 4`: permite utilizar até quatro threads (núcleos de processamento) durante a execução;
- `"$i"`: representa o arquivo que está sendo processado naquele momento;
- `--outdir 2_qc_brutos/`: define onde os resultados serão armazenados;
- `done`: indica o encerramento do loop.

**Resultados gerados**

Para cada arquivo FASTQ analisado, o FastQC produzirá dois arquivos:

| Arquivo | Conteúdo |
|---|---|
| `.html` | Relatório de qualidade interativo, que pode ser aberto no navegador |
| `.zip` | Arquivo compactado contendo os dados e resultados utilizados na construção do relatório |

Os resultados serão armazenados no diretório `2_qc_brutos`.

Podemos verificar se os relatórios foram gerados:

```bash
ls -lh 2_qc_brutos/
```

### 5.4. Consolidação dos resultados com MultiQC

Como temos vários relatórios individuais, podemos utilizar a ferramenta **MultiQC** para reunir os resultados em um único relatório.

O MultiQC identifica automaticamente os relatórios gerados por ferramentas de bioinformática e organiza suas informações em uma interface interativa.

Primeiramente, verificamos se a ferramenta está instalada:

```bash
multiqc --version
```

Para reunir os relatórios do FastQC, utilizaremos:

```bash
multiqc 2_qc_brutos/ -o 2_qc_brutos/
```

**Parâmetros:**

- `multiqc`: executa a ferramenta MultiQC;
- `2_qc_brutos/`: diretório onde estão os relatórios individuais do FastQC;
- `-o`: define o diretório de saída;
- `2_qc_brutos/`: diretório onde será salvo o relatório consolidado.

O MultiQC produzirá um arquivo denominado `multiqc_report.html`, além de um diretório com os dados utilizados na construção do relatório.

Para visualizar o relatório, entre no diretório:

```bash
cd 2_qc_brutos/
```

Abra a pasta no Windows Explorer:

```bash
explorer.exe .
```

Localize e abra o arquivo `multiqc_report.html` no navegador.

### 5.5. Avaliação da qualidade das reads brutas

Após a análise dos relatórios, devemos avaliar os principais indicadores de qualidade e identificar a necessidade de processamento das reads.

Nesta atividade, observamos a necessidade de remover bases das extremidades das reads e estabelecer um comprimento mínimo de 50 pares de bases (pb) para as sequências mantidas.

Como os valores de qualidade Phred apresentaram resultados satisfatórios, não aplicaremos um filtro rigoroso de qualidade ao longo de toda a sequência. Entretanto, utilizaremos parâmetros de remoção de bases nas extremidades.

A trimagem será realizada com a ferramenta Trimmomatic.

---

## 6. Trimagem das reads

A trimagem (*trimming*) consiste no processamento das reads para remover sequências indesejadas, como bases de baixa qualidade e adaptadores, quando presentes.

Nesta atividade, utilizaremos a ferramenta **Trimmomatic**, que permite realizar esse processamento em dados de sequenciamento paired-end.

### 6.1. Criação do diretório para armazenar as reads trimadas

Primeiramente, precisamos retornar ao diretório principal da análise:

```bash
cd ..
```

Criaremos um diretório denominado `3_reads_trimadas`, onde serão armazenadas as reads após o processamento:

```bash
mkdir 3_reads_trimadas
```

Verifique os diretórios existentes:

```bash
ls
```

Resultado esperado:

```text
0_referencia  1_reads  2_qc_brutos  3_reads_trimadas
```

### 6.2. Verificação da instalação do Trimmomatic

Antes de iniciar a trimagem, podemos verificar se a ferramenta está instalada corretamente:

```bash
trimmomatic -version
```

### 6.3. Execução do Trimmomatic

Como temos seis amostras paired-end, utilizaremos novamente um loop para automatizar o processamento de todos os arquivos.

O comando será executado em segundo plano, permitindo que o processo continue mesmo se fecharmos o terminal.

```bash
nohup bash -c 'for i in 1_reads/*_R1.fastq.gz; do
    s=$(basename "$i" _R1.fastq.gz)
    trimmomatic PE -threads 4 \
    "$i" \
    "1_reads/${s}_R2.fastq.gz" \
    "3_reads_trimadas/${s}_R1_paired.fastq.gz" \
    "3_reads_trimadas/${s}_R1_unpaired.fastq.gz" \
    "3_reads_trimadas/${s}_R2_paired.fastq.gz" \
    "3_reads_trimadas/${s}_R2_unpaired.fastq.gz" \
    LEADING:12 TRAILING:3 MINLEN:50
done' > trimmomatic.log 2>&1 < /dev/null &
```

Também é possível executar todo o comando em uma única linha. Para isso, basta remover as barras invertidas (\) utilizadas para indicar a quebra de linha, mantendo os espaços entre os argumentos.

**Entendendo o comando:**

- `nohup`: permite que o processo continue executando mesmo que a sessão do terminal seja encerrada;
- `bash -c`: executa os comandos escritos entre aspas como um script Bash;
- `for i in 1_reads/*_R1.fastq.gz`: inicia um loop que identifica os arquivos R1 de todas as amostras;
- `s=$(basename "$i" _R1.fastq.gz)`: extrai o nome da amostra, removendo o sufixo `_R1.fastq.gz`.

Por exemplo, para o arquivo `CT_01_R1.fastq.gz`, a variável `s` receberá o valor `CT_01`.

**Parâmetros do Trimmomatic:**

- `PE`: indica que os dados são paired-end (single-end seria SE);
- `-threads 4`: permite utilizar quatro threads durante o processamento.

Em seguida, são informados os arquivos de entrada e saída, sempre respeitando a seguinte ordem:

| Posição | Arquivo | Descrição |
|---|---|---|
| 1 | R1 de entrada | Primeira extremidade original |
| 2 | R2 de entrada | Segunda extremidade original |
| 3 | R1_paired | Reads R1 que permaneceram pareadas |
| 4 | R1_unpaired | Reads R1 que perderam o par |
| 5 | R2_paired | Reads R2 que permaneceram pareadas |
| 6 | R2_unpaired | Reads R2 que perderam o par |

**Parâmetros de trimagem utilizados:**

| Parâmetro | Função |
|---|---|
| `LEADING:12` | Remove bases das extremidades iniciais com qualidade Phred inferior a 12 |
| `TRAILING:3` | Remove bases das extremidades finais com qualidade Phred inferior a 3 |
| `MINLEN:50` | Descarta reads com comprimento inferior a 50 bases após a trimagem |

**Importante:** nesta análise, não estamos utilizando um filtro de janela deslizante (*SLIDINGWINDOW*). Esse parâmetro poderia ser adicionado para remover regiões de baixa qualidade ao longo das reads, mas optamos por não utilizá-lo inicialmente, considerando os resultados do controle de qualidade bruto. É possível e recomendado fazer mais de uma analise alterando um parâmetro por vez. 

**Arquivos de saída**

Para cada amostra, o Trimmomatic produzirá quatro arquivos:

| Arquivo | Significado |
|---|---|
| `R1_paired` | Reads R1 que possuem seu par correspondente em R2 |
| `R1_unpaired` | Reads R1 cujo par correspondente foi descartado |
| `R2_paired` | Reads R2 que possuem seu par correspondente em R1 |
| `R2_unpaired` | Reads R2 cujo par correspondente foi descartado |

Os arquivos  contêm os pares de reads que permaneceram após a trimagem e serão utilizados posteriormente nas análises que exigem dados paired-end. Os arquivos  também podem ser aproveitados em análises que aceitam reads single-end, mas não serão utilizados nessa etapa do nosso fluxo.

Os arquivos `paired` contêm os pares de reads (R1 e R2) que passaram pelos critérios de trimagem e serão utilizados nas análises paired-end. Já os arquivos `unpaired` contêm reads que passaram pelos critérios de trimagem, mas cuja read correspondente do mesmo fragmento foi descartada por não atender a algum dos critérios, como o comprimento mínimo. Portanto, são reads que ficaram sem seu par.

**Entendendo a execução em segundo plano**

Ao final do comando, temos:

- `> trimmomatic.log`: direciona a saída padrão para um arquivo de registro denominado `trimmomatic.log`;
- `2>&1`: direciona também as mensagens de erro para esse mesmo arquivo;
- `< /dev/null`: impede que o processo dependa de entradas do terminal;
- `&`: coloca o processo em segundo plano.

Assim, as mensagens de execução e possíveis erros serão registrados no arquivo `trimmomatic.log`, localizado no diretório principal `rna-seq`.

Para acompanhar o andamento da trimagem:

```bash
tail -f trimmomatic.log
```

---

## 7. Controle de qualidade das reads trimadas

Após a trimagem, precisamos realizar novamente o controle de qualidade para verificar se o processamento foi adequado e avaliar as características das reads resultantes.

Essa etapa é importante para comparar os resultados obtidos antes e depois da trimagem, verificando, por exemplo, se houve melhora na qualidade das extremidades, se o comprimento das reads foi preservado conforme os parâmetros estabelecidos e se foi perdido quantidade considerável de reads.

### 7.1. Criação do diretório para armazenar os resultados

No diretório principal `rna-seq`, criaremos um novo diretório denominado `4_qc_trimados`:

```bash
mkdir 4_qc_trimados
```

Verifique os diretórios existentes:

```bash
ls
```

Resultado esperado:

```text
0_referencia  1_reads  2_qc_brutos  3_reads_trimadas  4_qc_trimados
```

### 7.2. Execução do FastQC nas reads trimadas

Utilizaremos novamente o FastQC para avaliar os arquivos resultantes da trimagem.

Como temos quatro arquivos de saída por amostra, serão analisados todos os arquivos `.gz` presentes no diretório `3_reads_trimadas`.

```bash
for i in 3_reads_trimadas/*.fastq.gz; do fastqc -t 4 "$i" --outdir 4_qc_trimados/ done
```

Assim como na análise das reads brutas, o FastQC produzirá dois arquivos para cada FASTQ analisado:

- `.html`: relatório interativo de qualidade;
- `.zip`: arquivo compactado contendo os dados utilizados na geração do relatório.

### 7.3. Consolidação dos resultados com MultiQC

Para facilitar a comparação entre as amostras, utilizaremos novamente o MultiQC para reunir os relatórios individuais do FastQC em um único relatório.

```bash
multiqc 4_qc_trimados/ -o 4_qc_trimados/
```

O relatório consolidado será salvo no diretório `4_qc_trimados`, com o nome `multiqc_report.html`.

Para visualizar o resultado:

```bash
cd 4_qc_trimados/
explorer.exe .
```

Abra o arquivo `multiqc_report.html` no navegador.

### 7.4. Avaliação da qualidade após a trimagem

Após a execução do FastQC e do MultiQC, devemos avaliar os resultados e compará-los com aqueles obtidos antes da trimagem.

Alguns aspectos importantes a serem observados são:

- **Qualidade das bases:** verificar se houve melhora nos valores de qualidade ao longo das reads;
- **Comprimento das reads:** verificar a distribuição dos comprimentos após a remoção das bases;
- **Conteúdo de GC:** avaliar se a distribuição permaneceu semelhante à observada nos dados brutos;
- **Sequências adaptadoras:** verificar se houve redução ou remoção das sequências adaptadoras, caso estivessem presentes;
- **Número de reads:** avaliar quantas reads foram mantidas, descartadas ou perderam seus pares durante a trimagem.

É importante lembrar que nem todos os indicadores do FastQC precisam apresentar o resultado *PASS*. Alguns módulos podem apresentar *WARNING* ou *FAIL* mesmo em dados adequados para análise, dependendo das características do material biológico e da biblioteca de sequenciamento.

Ao final dessa etapa, teremos os arquivos de reads processadas e seus respectivos relatórios de qualidade, que serão utilizados para dar continuidade à análise de RNA-seq.

