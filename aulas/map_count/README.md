
## 8. Alinhamento das reads de RNA-seq ao genoma de referência (mapeamento)

Após a avaliação da qualidade das reads brutas, a trimagem e a avaliação da qualidade pós-trimagem, a próxima etapa da análise de RNA-seq é o **mapeamento (alignment)**.

O mapeamento consiste em comparar as reads obtidas pelo sequenciamento com uma sequência de referência, buscando identificar em quais regiões do genoma cada read se origina.

No caso de RNA-seq, esse processo permite identificar a localização genômica dos fragmentos sequenciados, possibilitando posteriormente determinar quais genes estão sendo expressos e quantificar o número de reads associadas a cada gene.

Nesta atividade, realizaremos o mapeamento contra o genoma de referência de *Drosophila melanogaster*, disponibilizado pelo NCBI (National Center for Biotechnology Information).

O assembly utilizado será:

**GCF_000001215.4 – Release 6 plus ISO1 MT**

O genoma de referência foi previamente obtido na etapa 2 e está armazenado no diretório `0_referencia`, no arquivo `Dmel_genome.fasta`.

### 8.1. Ferramenta utilizada: HISAT2

Para realizar o mapeamento, utilizaremos a ferramenta **HISAT2**, desenvolvida para o alinhamento de reads de RNA-seq e DNA-seq contra genomas de referência.

O HISAT2 é especialmente adequado para análises de RNA-seq por permitir o alinhamento de reads que atravessam junções entre éxons (*splice junctions*). Isso é importante porque o RNA mensageiro maduro apresenta íntrons removidos durante o processamento, e, portanto, uma read pode corresponder a regiões de éxons separados por um íntron no genoma.

Documentação oficial: https://daehwankimlab.github.io/hisat2/

### 8.2. Indexação do genoma de referência

Antes de realizar o mapeamento, precisamos indexar o genoma de referência.

A indexação consiste em preparar a sequência genômica em uma estrutura de arquivos que permite ao HISAT2 localizar e alinhar as reads de maneira mais eficiente, sem precisar pesquisar diretamente em todo o arquivo FASTA a cada alinhamento.

Para isso, utilizaremos o programa `hisat2-build`, que cria o índice a partir do genoma de referência.

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

Os arquivos `Dmel_genome_index.1.ht2` até `Dmel_genome_index.8.ht2` correspondem às partes que compõem o índice do genoma.

**Observação:** a indexação é realizada uma única vez para determinado genoma de referência. Assim, o mesmo índice poderá ser utilizado no mapeamento de todas as amostras, sem necessidade de reconstruí-lo para cada biblioteca.

### 8.3. Criação do diretório para armazenar os resultados do mapeamento

Para manter os resultados organizados, criaremos um diretório denominado `5_mapeamento`, onde serão armazenados os arquivos gerados pelo HISAT2.

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

Como discutido na etapa de trimagem, os arquivos `paired` contêm os pares de reads que permaneceram associados após o processamento. Já os arquivos `unpaired` contêm reads que perderam seus respectivos pares.

Nesta atividade, utilizaremos somente os arquivos `paired`, pois o mapeamento será realizado considerando a estratégia *paired-end*.

Podemos listar esses arquivos utilizando:

```bash
ls 3_reads_trimadas/*paired*
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

Utilizaremos o seguinte comando:

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

O arquivo SAM (*Sequence Alignment/Map*) é um formato de texto que armazena informações sobre o alinhamento das reads contra o genoma de referência.

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
