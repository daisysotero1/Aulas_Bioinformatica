
## 8. Alinhamento das reads de RNA-seq ao genoma de referência (mapeamento)

Após a avaliação da qualidade das reads brutas, a trimagem e a avaliação da qualidade pós-trimagem, a próxima etapa da análise de RNA-seq é o **mapeamento (alignment)**.

O mapeamento consiste em comparar as reads obtidas pelo sequenciamento após trimagem (se foi feito) com uma sequência de referência (genoma ou transcriptoma montado), buscando identificar em quais regiões da referência cada read se origina.

No caso de RNA-seq, quando a referência é um genoma, esse processo permite identificar a localização genômica dos fragmentos sequenciados, possibilitando posteriormente determinar quais genes estão sendo expressos e quantificar o número de reads associadas a cada gene.

Nesta atividade, realizaremos o mapeamento contra o genoma de referência de *Drosophila melanogaster*, disponibilizado pelo NCBI (National Center for Biotechnology Information). O assembly utilizado será: **GCF_000001215.4 – Release 6 plus ISO1 MT**

O genoma de referência foi previamente obtido e está armazenado no diretório `0_referencia`, no arquivo `Dmel_genome.fasta`.

### 8.1. Ferramenta utilizada: HISAT2

Para realizar o mapeamento, utilizaremos a ferramenta **HISAT2**, desenvolvida para o alinhamento de reads de RNA-seq vs. genomas de referência.

O HISAT2 é especialmente adequado para análises de RNA-seq por permitir o alinhamento de reads que atravessam junções entre éxons (*splice junctions*). Isso é importante porque o RNA mensageiro maduro apresenta íntrons removidos durante o processamento, e, portanto, uma read pode corresponder a regiões de éxons separados por um íntron no genoma.

Documentação oficial: https://daehwankimlab.github.io/hisat2/

### 8.2. Indexação do genoma de referência

Antes de realizar o mapeamento, precisamos indexar o genoma de referência.

A indexação consiste em preparar a sequência genômica em uma estrutura de arquivos que permite ao HISAT2 localizar e alinhar as reads de maneira mais eficiente, sem precisar pesquisar diretamente em todo o arquivo FASTA a cada alinhamento.

Para isso, utilizaremos a função `hisat2-build`, que cria o índice a partir do genoma de referência.

**Exemplo de sintaxe para indexação (não executar):**

```bash
hisat2-build genoma_referencia.fasta genoma_referencia_index
```

**Parâmetros:**

- `hisat2-build`: executa o programa responsável pela construção do índice;
- `genoma_referencia.fasta`: arquivo FASTA contendo a sequência do genoma de referência;
- `genoma_referencia_index`: prefixo utilizado para nomear os arquivos de índice gerados.

**Importante:** o segundo argumento não corresponde a um arquivo de saída único, mas sim ao prefixo utilizado para nomear os arquivos que compõem o índice.

#### Indexação do genoma de *Drosophila melanogaster*

Antes de executar os comandos, precisamos ativar o ambiente virtual Conda `rnaseq`, no qual as ferramentas de análise foram instaladas.

```bash
conda activate rnaseq
```

O terminal deverá apresentar:

```text
(rnaseq) daisy@Daisy:~/Daisy/rna-seq$
```

Como estamos no diretório principal `rna-seq`, podemos executar:

```bash
hisat2-build 0_referencia/Dmel_genome.fasta 0_referencia/Dmel_genome_index
```

Nesse comando:

- **Input:** `0_referencia/Dmel_genome.fasta` — genoma de referência utilizado para construir o índice.
- **Output:** `0_referencia/Dmel_genome_index` — prefixo dos arquivos de índice que serão gerados.

Observe que tanto o arquivo de entrada quanto os arquivos de saída estão no diretório `0_referencia`. Isso acontece porque especificamos esse caminho nos dois argumentos do comando.

O HISAT2 produzirá oito arquivos de índice, com extensões `.ht2`, que serão utilizados posteriormente durante o mapeamento.

Podemos verificar os arquivos gerados:

```bash
ls 0_referencia/
```

Resultado esperado:

```text
Dmel_annotation.gtf  Dmel_genome_index.1.ht2
Dmel_genome.fasta    Dmel_genome_index.2.ht2
                     Dmel_genome_index.3.ht2
                     Dmel_genome_index.4.ht2
                     Dmel_genome_index.5.ht2
                     Dmel_genome_index.6.ht2
                     Dmel_genome_index.7.ht2
                     Dmel_genome_index.8.ht2
```

**Observação:** a indexação é realizada uma única vez para determinado genoma de referência. Assim, o mesmo índice poderá ser utilizado no mapeamento de todas as amostras, sem necessidade de reconstruí-lo para cada biblioteca.

### 8.3. Criação do diretório para armazenar os resultados do mapeamento

Para manter os resultados organizados, criaremos um diretório denominado `5_mapeamento`, onde serão armazenados os arquivos gerados durante o mapeamento pelo HISAT2.

No diretório principal `rna-seq`, execute:

```bash
mkdir 5_mapeamento
```

Verifique se o diretório foi criado:

```bash
ls
```

Resultado esperado:

```text
0_referencia  1_reads  2_qc_brutos  3_reads_trimadas
4_qc_trimados  5_mapeamento
```

### 8.4. Identificação das reads que serão utilizadas no mapeamento

Antes de executar o HISAT2, precisamos identificar os arquivos FASTQ que serão utilizados como entrada.

Como discutido na etapa de trimagem, os arquivos `paired` contêm os pares de reads que permaneceram associados após o processamento. Já os arquivos `unpaired` contêm reads que perderam seus respectivos pares. Nesta atividade, utilizaremos somente os arquivos `paired`, pois o mapeamento será realizado considerando a estratégia *paired-end*.

Podemos listar esses arquivos utilizando:

```bash
ls 3_reads_trimadas/*_paired.fastq.gz
```

Resultado esperado:

```text
3_reads_trimadas/CT_01_R1_paired.fastq.gz
3_reads_trimadas/CT_01_R2_paired.fastq.gz
3_reads_trimadas/CT_02_R1_paired.fastq.gz
3_reads_trimadas/CT_02_R2_paired.fastq.gz
3_reads_trimadas/CT_03_R1_paired.fastq.gz
3_reads_trimadas/CT_03_R2_paired.fastq.gz
3_reads_trimadas/GL_01_R1_paired.fastq.gz
3_reads_trimadas/GL_01_R2_paired.fastq.gz
3_reads_trimadas/GL_02_R1_paired.fastq.gz
3_reads_trimadas/GL_02_R2_paired.fastq.gz
3_reads_trimadas/GL_03_R1_paired.fastq.gz
3_reads_trimadas/GL_03_R2_paired.fastq.gz
```

Temos, portanto, seis bibliotecas, cada uma representada por dois arquivos FASTQ correspondentes às reads R1 e R2.

### 8.5. Execução do HISAT2

Como temos seis bibliotecas para mapear, utilizaremos novamente um **loop** para automatizar a execução do HISAT2, evitando a necessidade de escrever um comando individual para cada amostra.

O processo será executado em segundo plano, utilizando `nohup`, permitindo que continue mesmo que a sessão do terminal seja encerrada.

Utilizaremos o seguinte comando (pode executar neste momento):

```bash
nohup bash -c '
for r1 in 3_reads_trimadas/*_R1_paired.fastq.gz; do
    sample_name=$(basename "$r1" _R1_paired.fastq.gz)
    r2="3_reads_trimadas/${sample_name}_R2_paired.fastq.gz"

    hisat2 -p 8 -x 0_referencia/Dmel_genome_index \
        -1 "$r1" \
        -2 "$r2" \
        -S "5_mapeamento/${sample_name}.sam"
done
' > hisat2.log 2>&1 < /dev/null &
```

Também é possível executar todo o comando em uma única linha, removendo as quebras de linha utilizadas para facilitar a leitura.

**Entendendo o comando:**

- `nohup`: permite que o processo continue executando mesmo que a sessão do terminal seja encerrada;
- `bash -c`: executa os comandos escritos entre aspas como um script Bash;
- `for r1 in 3_reads_trimadas/*_R1_paired.fastq.gz`: inicia um loop que identifica todos os arquivos R1 pareados no diretório `3_reads_trimadas`;
- `sample_name=$(basename "$r1" _R1_paired.fastq.gz)`: extrai o nome da amostra, removendo o sufixo `_R1_paired.fastq.gz`.

Por exemplo, para o arquivo `CT_01_R1_paired.fastq.gz`, a variável `sample_name` receberá o valor `CT_01`.

- `r2="3_reads_trimadas/${sample_name}_R2_paired.fastq.gz"`: utiliza o nome da amostra para localizar automaticamente o arquivo R2 correspondente.

Assim, para a amostra `CT_01`, o loop identificará os seguintes arquivos:

```text
R1: 3_reads_trimadas/CT_01_R1_paired.fastq.gz
R2: 3_reads_trimadas/CT_01_R2_paired.fastq.gz
```

**Parâmetros do HISAT2:**

| Parâmetro | Função |
|---|---|
| `hisat2` | Executa a ferramenta de mapeamento |
| `-p 8` | Utiliza oito threads durante o processamento |
| `-x` | Indica o prefixo dos arquivos de índice do genoma de referência |
| `-1` | Especifica o arquivo FASTQ contendo as reads R1 |
| `-2` | Especifica o arquivo FASTQ contendo as reads R2 |
| `-S` | Especifica o arquivo de saída no formato SAM |

**Sobre o formato SAM**

O arquivo SAM (*Sequence Alignment/Map*) é um formato de texto que armazena informações sobre o alinhamento das reads vs. o genoma de referência.

Entre as informações armazenadas estão:
- Identificação de cada read;
- Localização genômica em que ocorreu o alinhamento;
- Orientação do alinhamento;
- Número de bases alinhadas e não alinhadas;
- Informações sobre a correspondência entre a read e a sequência de referência.

Cada biblioteca terá um arquivo SAM contendo os resultados do alinhamento de suas duas reads (R1 e R2).

Portanto, não teremos um arquivo SAM para R1 e outro para R2. As duas extremidades de cada fragmento serão processadas conjuntamente pelo HISAT2 e seus alinhamentos serão registrados no mesmo arquivo SAM da respectiva biblioteca.

**Arquivos de entrada e saída:**

| Amostra | Arquivos de entrada | Arquivo de saída |
|---|---|---|
| CT_01 | R1 + R2 | `CT_01.sam` |
| CT_02 | R1 + R2 | `CT_02.sam` |
| CT_03 | R1 + R2 | `CT_03.sam` |
| GL_01 | R1 + R2 | `GL_01.sam` |
| GL_02 | R1 + R2 | `GL_02.sam` |
| GL_03 | R1 + R2 | `GL_03.sam` |

**Sobre a execução em segundo plano**

Ao final do comando, temos:

- `> hisat2.log`: direciona a saída padrão para um arquivo de registro denominado `hisat2.log`;
- `2>&1`: direciona também as mensagens de erro para esse mesmo arquivo;
- `< /dev/null`: impede que o processo dependa de entradas do terminal;
- `&`: coloca o processo em segundo plano.

Assim, as mensagens de execução e possíveis erros serão registrados no arquivo `hisat2.log`, localizado no diretório principal `rna-seq`.

Para acompanhar o andamento do mapeamento:

```bash
tail -f hisat2.log
```

Para sair da visualização do log sem interromper o processo, pressione `Ctrl + C`.

### 8.6. Sobre a orientação da biblioteca: parâmetro `--rna-strandness`

O HISAT2 também permite especificar se a biblioteca de RNA-seq foi construída utilizando um protocolo orientado (*stranded*) ou não orientado (*unstranded*).

Essa informação é importante porque, em bibliotecas *stranded*, é possível preservar a informação sobre a fita de DNA a partir da qual o RNA foi originalmente transcrito.

Para isso, utiliza-se o parâmetro `--rna-strandness`.

Os valores aceitos dependem da estratégia de sequenciamento:

| Tipo de biblioteca | Parâmetro | Significado |
|---|---|---|
| Single-end | `F` | A read corresponde ao sentido do transcrito |
| Single-end | `R` | A read corresponde à sequência reversamente complementar ao transcrito |
| Paired-end | `FR` | Orientação correspondente a uma biblioteca *forward-stranded* |
| Paired-end | `RF` | Orientação correspondente a uma biblioteca *reverse-stranded* |

**Importante:** os valores `F` e `R` não devem ser utilizados isoladamente para bibliotecas paired-end. Nesse caso, devem ser utilizados `FR` ou `RF`, conforme o protocolo de preparação da biblioteca.

Nesta atividade, **não utilizaremos o parâmetro `--rna-strandness`**, pois não temos a informação necessária para determinar se as bibliotecas utilizadas foram construídas por um protocolo *forward-stranded* ou *reverse-stranded*.

Assim, o HISAT2 realizará o alinhamento sem considerar uma orientação específica da biblioteca (*unstranded*), que é o comportamento padrão da ferramenta.

**Observação:** quando trabalhamos com dados de RNA-seq provenientes de bibliotecas *stranded*, é recomendado consultar as informações do protocolo de preparação da biblioteca e utilizar o parâmetro correspondente. Isso permite preservar a informação sobre a orientação dos transcritos, o que pode ser importante especialmente em regiões genômicas que apresentam genes sobrepostos ou transcritos em fitas opostas.

### 8.7. Verificação dos arquivos gerados

Após a conclusão do mapeamento, podemos verificar os arquivos gerados no diretório `5_mapeamento`:

```bash
ls -lh 5_mapeamento/
```

Ao final, esperamos encontrar seis arquivos SAM, um para cada biblioteca:

```text
CT_01.sam
CT_02.sam
CT_03.sam
GL_01.sam
GL_02.sam
GL_03.sam
```

Cada arquivo conterá os alinhamentos das reads R1 e R2 da respectiva amostra.

Os arquivos SAM podem ocupar bastante espaço em disco, pois armazenam informações detalhadas sobre os alinhamentos. Posteriormente, eles poderão ser convertidos para o formato BAM (*Binary Alignment/Map*), que é uma versão binária e compactada do SAM.

**Ao final desta etapa, teremos realizado o mapeamento das seis bibliotecas de RNA-seq contra o genoma de referência de *Drosophila melanogaster*.** Os arquivos gerados serão utilizados nas próximas etapas da análise, incluindo a conversão, ordenação e indexação dos alinhamentos e, posteriormente, a contagem das reads por gene utilizando o FeatureCounts.


## 9. Convertendo de SAM para BAM e ordenando os alinhamentos

Após realizar o mapeamento das reads contra o genoma de referência utilizando o HISAT2, temos como resultado arquivos no formato SAM (*Sequence Alignment/Map*).

Embora o formato SAM armazene todas as informações dos alinhamentos, ele é um arquivo de texto e, por isso, pode ocupar bastante espaço em disco. Além disso, seu processamento pode ser menos eficiente quando comparado a formatos binários.

Para otimizar o armazenamento e facilitar as etapas seguintes da análise, converteremos os arquivos SAM para o formato BAM (*Binary Alignment/Map*).

O formato BAM armazena as mesmas informações do SAM, mas em formato binário compactado, reduzindo significativamente o tamanho dos arquivos. Além disso, permite operações mais eficientes de leitura e processamento.

Também realizaremos a **ordenação dos alinhamentos (*sorting*)**, organizando as reads de acordo com suas posições no genoma de referência (ordenação por coordenadas genômicas). Essa organização é importante para facilitar o processamento e a utilização dos arquivos em ferramentas posteriores.

### 9.1. Ferramenta utilizada: SAMtools

Para realizar essas operações, utilizaremos o **SAMtools**, um conjunto de ferramentas desenvolvido para manipular arquivos de alinhamento nos formatos SAM, BAM e CRAM.

Entre suas funcionalidades estão a conversão de formatos, ordenação dos alinhamentos, indexação dos arquivos BAM e extração de informações dos alinhamentos.

Documentação oficial: [SAMtools](https://www.htslib.org/)

**Comando genérico para conversão e ordenação (não executar):**

```bash
samtools view -@ 6 -b arquivo.sam | samtools sort -@ 6 -o arquivo_sorted.bam
```

**Entendendo os parâmetros:**

| Parâmetro | Função |
|---|---|
| `samtools view` | Permite visualizar, filtrar e converter arquivos de alinhamento. |
| `-@ 6` | Utiliza seis threads adicionais para as operações de compressão/descompressão. |
| `-b` | Indica que a saída deverá ser gerada no formato BAM. |
| `arquivo.sam` | Arquivo SAM de entrada. |
| `\|` | Encaminha a saída do primeiro comando diretamente para o segundo, sem a necessidade de gerar um arquivo intermediário. |
| `samtools sort` | Realiza a ordenação dos alinhamentos. |
| `-o` | Especifica o nome e o caminho do arquivo de saída. |
| `arquivo_sorted.bam` | Arquivo BAM ordenado por coordenadas genômicas. |

**Importante:** a combinação dos dois comandos permite converter e ordenar os alinhamentos em uma única operação. Primeiro, o `samtools view` converte o SAM para BAM. Em seguida, o `samtools sort` recebe esse resultado diretamente pelo pipeline (`|`) e realiza a ordenação, gerando o arquivo BAM final.

### 9.2. Conversão e ordenação dos arquivos de *Drosophila melanogaster*

Como temos seis bibliotecas mapeadas, utilizaremos novamente um **loop** para automatizar a conversão e a ordenação dos arquivos, evitando a necessidade de executar um comando individual para cada amostra.

No diretório principal `rna-seq`, execute:

```bash
for sam_file in 5_mapeamento/*.sam; do
    sample_name=$(basename "$sam_file" .sam)

    echo "Processando amostra: $sample_name"

    samtools view -@ 6 -b "$sam_file" | \
    samtools sort -@ 6 -o "5_mapeamento/${sample_name}_sorted.bam"

    echo "Concluído: $sample_name"
    echo "-----------------------------"
done

echo "Todas as amostras foram processadas!"
```

**Entendendo o comando:**

- `for sam_file in 5_mapeamento/*.sam`: inicia um loop que identifica todos os arquivos com extensão `.sam` no diretório `5_mapeamento`;
- `sample_name=$(basename "$sam_file" .sam)`: extrai o nome da amostra, removendo a extensão `.sam`;
- `echo`: exibe mensagens no terminal para acompanhar o processamento;
- `samtools view -@ 6 -b`: converte o arquivo SAM para BAM;
- `samtools sort -@ 6`: recebe o BAM pelo pipeline e ordena os alinhamentos;
- `-o "5_mapeamento/${sample_name}_sorted.bam"`: define o nome e o caminho do arquivo BAM de saída.

### 9.3. Verificação dos arquivos gerados

Após a conclusão da conversão, podemos verificar os arquivos presentes no diretório `5_mapeamento`:

```bash
ls -lh 5_mapeamento/
```

Resultado esperado:

```text
CT_01.sam         CT_02.sam         CT_03.sam
GL_01.sam         GL_02.sam         GL_03.sam

CT_01_sorted.bam  CT_02_sorted.bam  CT_03_sorted.bam
GL_01_sorted.bam  GL_02_sorted.bam  GL_03_sorted.bam
```

Também podemos verificar a integridade básica dos arquivos BAM utilizando:

```bash
for bam in 5_mapeamento/*_sorted.bam; do
    samtools quickcheck -v "$bam"
done
```

O `samtools quickcheck` verifica aspectos básicos da integridade dos arquivos BAM, incluindo o cabeçalho e o final do arquivo. Se não houver mensagens de erro, os arquivos passaram por essa verificação.

## 10. Contagem das reads por gene utilizando featureCounts

Após realizar o mapeamento das reads e organizar os alinhamentos no formato BAM, a próxima etapa da análise de RNA-seq consiste em **quantificar a expressão gênica**, determinando o número de reads associadas a cada gene.

Para isso, utilizaremos a ferramenta **featureCounts**, desenvolvida para realizar a contagem de reads alinhadas a regiões genômicas de interesse (*features*), como genes ou éxons. A ferramenta utiliza dois arquivos principais: os arquivos BAM, que contêm os alinhamentos das reads, e um arquivo de anotação genômica, no formato GTF  que informa a localização e a identificação dos genes no genoma de referência. A partir dessas informações, o featureCounts identifica quais reads se sobrepõem às regiões anotadas e realiza a contagem correspondente a cada gene.

Documentação oficial: [featureCounts](https://subread.sourceforge.net/featureCounts.html)

### 10.1. Criação do diretório para armazenar os resultados

Primeiramente, criaremos um diretório denominado `6_featureCounts`, onde serão armazenados os arquivos gerados durante a etapa de quantificação.

No diretório principal `rna-seq`, execute:

```bash
mkdir -p 6_featureCounts
```

Verifique se o diretório foi criado:

```bash
ls
```

Resultado esperado:

```text
0_referencia
1_reads
2_qc_brutos
3_reads_trimadas
4_qc_trimados
5_mapeamento
6_featureCounts
```

### 10.2. Entendendo o comando do featureCounts

Antes de executar a contagem das reads, vamos compreender a estrutura do comando.

**Exemplo de sintaxe para uma amostra (não executar):**

```bash
featureCounts -T 4 -a input_coordenadas.gtf -o output_tabela.txt arquivo_mapeamento_sorted.bam
```

**Parâmetros:**

| Parâmetro | Função |
|---|---|
| `featureCounts` | Executa a ferramenta responsável pela contagem das reads. |
| `-T 4` | Utiliza quatro threads durante o processamento. |
| `-a` | Especifica o arquivo de anotação genômica no formato GTF ou GFF. |
| `input_coordenadas.gtf` | Arquivo contendo as coordenadas e informações dos genes no genoma de referência. |
| `-o` | Especifica o nome e o caminho do arquivo de saída. |
| `output_tabela.txt` | Arquivo de saída contendo a tabela de contagem. |
| `arquivo_mapeamento_sorted.bam` | Arquivo BAM contendo os alinhamentos das reads. |

**Importante:** o arquivo de anotação genômica deve corresponder ao mesmo genoma de referência utilizado durante o mapeamento. Isso garante que as coordenadas utilizadas para identificar os genes sejam compatíveis com os alinhamentos armazenados nos arquivos BAM.

No nosso caso, utilizaremos o arquivo `Dmel_annotation.gtf`, correspondente ao genoma de referência de *Drosophila melanogaster*.

### 10.3. Contagem das reads das seis bibliotecas

Como temos seis bibliotecas, podemos fornecer todos os arquivos BAM ao featureCounts utilizando o caractere curinga `*`. Assim, a ferramenta processará todas as amostras em uma única execução, gerando uma tabela de contagem com os valores correspondentes a cada biblioteca.

**Atenção:** como nossas bibliotecas são *paired-end*, precisamos informar essa característica ao featureCounts. Para contar cada par de reads como um único fragmento sequenciado, utilizaremos os parâmetros `-p --countReadPairs`, disponíveis nas versões recentes da ferramenta.

Portanto, o comando que utilizaremos será:

```bash
nohup featureCounts \
    -T 6 \
    -p --countReadPairs \
    -a 0_referencia/Dmel_annotation.gtf \
    -o 6_featureCounts/counts.txt \
    5_mapeamento/*_sorted.bam \
    > 6_featureCounts/featureCounts.log 2>&1 < /dev/null &
```

**Entendendo os parâmetros adicionais:**

| Parâmetro | Função |
|---|---|
| `-p` | Informa que os arquivos de entrada contêm reads *paired-end*. |
| `--countReadPairs` | Conta cada par de reads como um único fragmento, em vez de contabilizar as duas extremidades separadamente. |
| `nohup` | Permite que o processo continue executando mesmo que a sessão do terminal seja encerrada. |
| `> 6_featureCounts/featureCounts.log` | Direciona a saída padrão para um arquivo de log. |
| `2>&1` | Direciona também as mensagens de erro para o mesmo arquivo de log. |
| `< /dev/null` | Impede que o processo dependa de entradas do terminal. |
| `&` | Coloca o processo em segundo plano. |

**Observação:** a opção `--countReadPairs` depende da versão do featureCounts instalada. Caso necessário, verifique a versão e as opções disponíveis utilizando:

```bash
featureCounts -v
featureCounts -h
```

### 10.4. Acompanhamento da execução

Como a contagem será realizada para seis bibliotecas, o processamento poderá levar alguns minutos, dependendo do tamanho dos arquivos BAM e dos recursos computacionais disponíveis.

Como utilizamos `nohup`, podemos acompanhar o andamento da execução por meio do arquivo de log:

```bash
tail -f 6_featureCounts/featureCounts.log
```

Para sair da visualização do log sem interromper o processamento, pressione `Ctrl + C`.

### 10.5. Verificação dos resultados

Após a conclusão da contagem, podemos verificar os arquivos gerados:

```bash
ls -lh 6_featureCounts/
```

Resultado esperado:

```text
counts.txt
counts.txt.summary
featureCounts.log
```

Os arquivos gerados possuem as seguintes finalidades:

- **`counts.txt`:** tabela contendo os genes identificados na anotação e o número de fragmentos associados a cada gene em cada amostra. Essa tabela será utilizada nas próximas etapas da análise de expressão diferencial.
- **`counts.txt.summary`:** arquivo contendo um resumo da contagem, com o número de fragmentos atribuídos e não atribuídos às regiões anotadas, incluindo as possíveis razões para a não atribuição.
- **`featureCounts.log`:** arquivo de registro contendo as mensagens geradas durante a execução da ferramenta.

### 10.6. Organização da matriz de contagem para a análise de expressão diferencial

Após a execução do featureCounts, o arquivo `counts.txt`, localizado no diretório `6_featureCounts/`, contém a matriz de contagem que será utilizada na análise de expressão diferencial.

Nessa matriz, cada linha representa um gene, enquanto cada coluna representa uma amostra (CT_01, CT_02, CT_03, GL_01, GL_02 e GL_03), contendo o número de fragmentos atribuídos a cada gene. Entretanto, antes de utilizar essa matriz nas próximas etapas, precisamos realizar algumas modificações para adequar o arquivo ao formato de entrada das ferramentas de análise de expressão diferencial, como DESeq2.

**Estrutura original do arquivo `counts.txt`:**

<table>
<tr>
<td colspan="12"># Program:featureCounts v2.1.1; Command:"featureCounts" "-T" "6" "-p" "--countReadPairs" "-a" "0_referencia/Dmel_annotation.gtf" "-o" "6_featureCounts/counts.txt" "5_mapeamento/CT_01_sorted.bam" "5_mapeamento/CT_02_sorted.bam" "5_mapeamento/CT_03_sorted.bam" "5_mapeamento/GL_01_sorted.bam" "5_mapeamento/GL_02_sorted.bam" "5_mapeamento/GL_03_sorted.bam"</td>
</tr>
<tr>
<th>Geneid</th>
<th>Chr</th>
<th>Start</th>
<th>End</th>
<th>Strand</th>
<th>Length</th>
<th>5_mapeamento/CT_01_sorted.bam</th>
<th>5_mapeamento/CT_02_sorted.bam</th>
<th>5_mapeamento/CT_03_sorted.bam</th>
<th>5_mapeamento/GL_01_sorted.bam</th>
<th>5_mapeamento/GL_02_sorted.bam</th>
<th>5_mapeamento/GL_03_sorted.bam</th>
</tr>
<tr>
<td>Dmel_CR40469</td>
<td>NC_004354.4</td>
<td>122493</td>
<td>122706</td>
<td>+</td>
<td>214</td>
<td>9</td>
<td>24</td>
<td>17</td>
<td>56</td>
<td>54</td>
<td>40</td>
</tr>
<tr>
<td>Dmel_CG17636</td>
<td>NC_004354.4</td>
<td>...</td>
<td>...</td>
<td>-</td>
<td>2036</td>
<td>1102</td>
<td>1121</td>
<td>1379</td>
<td>1387</td>
<td>1497</td>
<td>1398</td>
</tr>
</table>


O arquivo apresenta algumas informações adicionais, como a versão do programa, o comando utilizado e as características das regiões genômicas (cromossomo, posição inicial, posição final, fita e comprimento). Essas informações não são necessárias para a matriz de contagem que será utilizada na análise. Para organizar a matriz, realizaremos as seguintes modificações:

1. **Salvar uma cópia** do arquivo original com o nome `counts_2.csv`, preservando o arquivo `counts.txt` original.
2. **Excluir a primeira linha**, que contém informações sobre a execução do programa.
3. **Excluir as colunas 2 a 6**, que correspondem às informações de anotação genômica (`Chr`, `Start`, `End`, `Strand` e `Length`).
4. **Renomear as colunas das amostras**, removendo o caminho dos arquivos BAM `5_mapeamento/` e o sufixo `_sorted.bam`. Por exemplo, `5_mapeamento/CT_01_sorted.bam` passará a ser `CT_01`.
5. **Salvar o arquivo final** no formato CSV.

Após essas modificações, teremos uma matriz contendo apenas os identificadores dos genes e suas respectivas contagens em cada amostra.

**Estrutura final esperada (`counts_2.csv`):**

| Geneid | CT_01 | CT_02 | CT_03 | GL_01 | GL_02 | GL_03 |
|---|---:|---:|---:|---:|---:|---:|
| Dmel_CR40469 | 9 | 24 | 17 | 56 | 54 | 40 |
| Dmel_CG17636 | 1102 | 1121 | 1379 | 1387 | 1497 | 1398 |

Essa matriz simplificada será utilizada como entrada para a próxima etapa da análise.

### 10.7. Análise de expressão diferencial de genes

Com a matriz de contagem organizada, podemos iniciar a análise de expressão diferencial entre os grupos experimentais.

Nessa etapa, serão identificados os genes que apresentam diferenças significativas nos níveis de expressão entre as condições experimentais CT e GL.

Para isso, utilizaremos o pacote **DESeq2**, disponível no ambiente R/Bioconductor. Essa ferramenta utiliza modelos estatísticos baseados na distribuição binomial negativa para analisar os dados de contagem, considerando a variabilidade entre as réplicas biológicas e normalizando as diferenças na profundidade de sequenciamento entre as amostras.

A análise permitirá identificar genes diferencialmente expressos (DEGs), considerando critérios estatísticos e de magnitude de variação da expressão.

