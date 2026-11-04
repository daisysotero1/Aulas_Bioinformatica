

# Aula 4 – Montagem de genoma

# Prática síncrona - parte 1

Após avaliar a qualidade dos dados e realizar a trimagem das leituras, o próximo passo em muitos projetos de sequenciamento é a **montagem do genoma** (*genome assembly*).

A montagem consiste em reconstruir sequências maiores (*contigs*) a partir das leituras obtidas no sequenciamento. Como as tecnologias atuais geram milhões de pequenos fragmentos de DNA, utilizamos programas capazes de identificar regiões de sobreposição entre essas leituras para reconstruir a sequência original do genoma.

Nesta aula utilizaremos o **SPAdes** (*St. Petersburg genome assembler*), um dos montadores mais utilizados para genomas bacterianos.

> Execute os comandos no terminal WSL/Linux. Linhas iniciadas por `#` são comentários e não precisam ser executadas.

---

## 1. Voltando para o diretório da Aula 2 - parte 2

Primeiro, volte para o diretório principal da Aula 2 (aula2_parte2):

```bash
cd ..
```
Em seguida veja o que há dentro desse diretório:

```bash
ls -lrth
```

A estrutura da pasta deverá estar semelhante a esta:

```text
(bioinfo) daisy@Daisy:~/Daisy/aula2_parte2$ ls -lrth
total 1.4G

drwxr-xr-x 2 daisy daisy 4.0K Aug 13 14:01 SRR11011985
-rw-r--r-- 1 daisy daisy 675M Aug 13 14:02 SRR11011985_2.fastq
-rw-r--r-- 1 daisy daisy 675M Aug 13 14:02 SRR11011985_1.fastq
drwxr-xr-x 2 daisy daisy 4.0K Aug 13 14:05 1_fastqc_brutos
drwxr-xr-x 2 daisy daisy 4.0K Aug 13 14:08 2_trimmomatic
drwxr-xr-x 2 daisy daisy 4.0K Aug 13 14:11 3_fastqc_trimmed
```

## 2. Criando a pasta da montagem

Crie um diretório para armazenar todos os arquivos produzidos pelo SPAdes:

```bash
mkdir 4_assembly
```

Organizar os resultados em diretórios separados facilita a navegação pelos arquivos e evita misturar resultados de diferentes etapas da análise.

---

## 3. Verificando a instalação do SPAdes

Antes de iniciar a montagem, confirme se o SPAdes está instalado corretamente:

```bash
spades.py -v
```

O resultado esperado é semelhante a:

```text
(bioinfo) daisy@Daisy:~/Daisy/aula2$ spades.py -v
SPAdes genome assembler v4.3.0
(bioinfo) daisy@Daisy:~/Daisy/aula2$
```

> **Importante:** para que o comando funcione, o ambiente virtual `bioinfo` deve estar ativado. Caso o nome `(bioinfo)` não apareça no início da linha de comando, ative o ambiente novamente:

```bash
conda activate bioinfo
```

---

## 4. Modos de execução do SPAdes

Assim como outras ferramentas utilizadas ao longo da disciplina, o SPAdes possui diversos parâmetros que modificam seu funcionamento. Eles podem ser consultados utilizando a opção `--help`:

```bash
spades.py --help
```

Entre as opções mais utilizadas estão:

| Parâmetro | Função |
|-----------|--------|
| `--isolate` | Recomendado para dados de organismos isolados com alta cobertura, como a maioria dos genomas bacterianos. Este será o modo utilizado nesta aula. |
| `--meta` | Utilizado para montagem de dados metagenômicos, provenientes de comunidades de microrganismos. |
| `--rna` | Utilizado para dados de RNA-Seq. |
| `--sc` | Utilizado para dados obtidos por sequenciamento de célula única (*single-cell*). |
| `--only-error-correction` | Executa apenas a etapa de correção de erros das leituras, sem realizar a montagem. |
| `--only-assembler` | Executa apenas a montagem, assumindo que as leituras já foram corrigidas. |
| `--careful` | Reduz o número de mismatches e pequenas inserções/deleções (*indels*) na montagem. **Na versão 4.3.0 do SPAdes, esse parâmetro não pode ser utilizado juntamente com `--isolate`.** |

Nesta aula utilizaremos o parâmetro `--isolate`, pois os dados analisados correspondem ao genoma de uma bactéria isolada (*Lactobacillus fermentum*), exatamente o tipo de dado para o qual esse modo foi desenvolvido.

---

## 5. Executando a montagem (não execute este comando neste momento)

O comando básico para executar o SPAdes é:

```bash
spades.py \
    --isolate \
    -1 2_trimmomatic/SRR11011985_1_paired.fastq \
    -2 2_trimmomatic/SRR11011985_2_paired.fastq \
    -o 4_assembly \
    -t 4 \
    -k auto
```

Entretanto, montagens podem demorar vários minutos ou até horas, dependendo do tamanho do genoma e da quantidade de dados.

Se executarmos o comando normalmente, o terminal permanecerá ocupado durante todo o processamento. Caso o terminal seja fechado, a montagem será interrompida.

Para evitar esse problema, utilizaremos o comando `nohup`, que permite executar programas em segundo plano.

---

## 6. Executando o SPAdes com `nohup` (execute este comando neste momento)

O `nohup` (*no hang up*) permite que um programa continue sendo executado mesmo após o fechamento do terminal.

Além disso, utilizaremos recursos do Linux para salvar todas as mensagens produzidas durante a execução em um arquivo de log.

```bash
nohup spades.py \
    --isolate \
    -1 2_trimmomatic/SRR11011985_1_paired.fastq \
    -2 2_trimmomatic/SRR11011985_2_paired.fastq \
    -o 4_assembly \
    -t 4 \
    -k auto \
    > spades.log 2>&1 &
```

Após executar o comando, o terminal será liberado imediatamente, enquanto a montagem continuará sendo executada em segundo plano.

---

## 7. Entendendo o comando

| Elemento | Função |
|----------|--------|
| `nohup` | Mantém o programa em execução mesmo que o terminal seja fechado. |
| `spades.py` | Executa o montador SPAdes. |
| `--isolate` | Indica ao SPAdes que os dados correspondem ao genoma de um organismo isolado, modo recomendado para a maioria dos genomas bacterianos. |
| `-1` | Arquivo FASTQ contendo as leituras *forward* (R1). |
| `-2` | Arquivo FASTQ contendo as leituras *reverse* (R2). |
| `-o 4_assembly` | Define o diretório onde todos os resultados da montagem serão armazenados. |
| `-t 4` | Utiliza até quatro núcleos de processamento (*threads*). |
| `-k auto` | Permite que o SPAdes escolha automaticamente os tamanhos de *k-mers* mais adequados para o comprimento das leituras. |
| `> spades.log` | Redireciona todas as mensagens exibidas na tela para o arquivo `spades.log`. |
| `2>&1` | Faz com que as mensagens de erro também sejam gravadas no mesmo arquivo de log. |
| `&` | Executa o programa em segundo plano, liberando o terminal para outros comandos. |

Durante a execução, é possível acompanhar o progresso observando o arquivo de log:

```bash
tail -f spades.log
```

Para interromper a visualização do log, pressione:

```text
Ctrl + C
```

Isso **não interrompe** a montagem; apenas fecha a visualização do arquivo.

---

## Organização esperada da pasta `aula2`

```text
 ls -lrth 4_assembly/
-rw-r--r-- 1 daisy daisy  256 Aug 13 14:19 input_dataset.yaml
-rw-r--r-- 1 daisy daisy 1.4K Aug 13 14:19 params.txt
-rw-r--r-- 1 daisy daisy   67 Aug 13 14:19 dataset.info
-rw-r--r-- 1 daisy daisy 2.4K Aug 13 14:19 run_spades.sh
-rw-r--r-- 1 daisy daisy 4.2K Aug 13 14:19 run_spades.yaml
drwxr-xr-x 4 daisy daisy 4.0K Aug 13 14:21 K21
drwxr-xr-x 4 daisy daisy 4.0K Aug 13 14:22 K33
drwxr-xr-x 4 daisy daisy 4.0K Aug 13 14:25 K55
-rw-r--r-- 1 daisy daisy 1.9M Aug 13 14:25 before_rr.fasta
-rw-r--r-- 1 daisy daisy 1.9M Aug 13 14:25 assembly_graph_after_simplification.gfa
-rw-r--r-- 1 daisy daisy  41K Aug 13 14:25 scaffolds.paths
-rw-r--r-- 1 daisy daisy 1.9M Aug 13 14:25 contigs.fasta
-rw-r--r-- 1 daisy daisy 1.9M Aug 13 14:25 scaffolds.fasta
-rw-r--r-- 1 daisy daisy 1.9M Aug 13 14:25 assembly_graph_with_scaffolds.gfa
-rw-r--r-- 1 daisy daisy  42K Aug 13 14:25 contigs.paths
-rw-r--r-- 1 daisy daisy 3.9M Aug 13 14:25 assembly_graph.fastg
drwxr-xr-x 2 daisy daisy 4.0K Aug 13 14:25 tmp
drwxr-xr-x 2 daisy daisy 4.0K Aug 13 14:25 misc
drwxr-xr-x 2 daisy daisy 4.0K Aug 13 14:25 pipeline_state
-rw-r--r-- 1 daisy daisy 112K Aug 13 14:25 spades.log

```

> A quantidade de arquivos e pastas pode variar dependendo da versão do SPAdes e dos parâmetros utilizados. Os arquivos mais importantes para as próximas etapas serão `contigs.fasta` e `scaffolds.fasta`, que contêm as sequências montadas.

---

## Prática síncrona - Parte 2

Nesta aula, será avaliado o resultado da montagem genômica realizada anteriormente com o programa **SPAdes**. A montagem foi feita a partir das leituras pareadas de *Lactobacillus fermentum* obtidas na corrida **SRR11011985**.

O objetivo é entender os arquivos produzidos pelo SPAdes, verificar características dos contigs e scaffolds e interpretar métricas importantes para avaliar a qualidade da montagem.

> Execute os comandos no terminal WSL/Linux. Linhas iniciadas por `#` são comentários e não precisam ser executadas.

## 1. Acessando o diretório da montagem

Abra o terminal WSL/Linux e entre na pasta principal criada nas aulas anteriores. Substitua `Daisy` pelo nome da pasta que você criou:

```bash
cd Daisy/
```

Entre na pasta da Aula 2:

```bash
cd aula2_parte2/
```

Entre no diretório que contém o resultado da montagem feita com SPAdes:

```bash
cd 4_assembly/
```

O caminho final deverá ser semelhante a este:

```bash
(base) /home/daisy/Daisy/aula2_parte2/4_assembly
```

## 2. Visualizando os arquivos gerados pelo SPAdes

Liste os arquivos presentes no diretório:

```bash
ls
```

Alguns dos arquivos e diretórios gerados podem ser:

```text
K21  assembly_graph.fastg                     before_rr.fasta  dataset.info        params.txt      run_spades.yaml  spades.log
K33  assembly_graph_after_simplification.gfa  contigs.fasta    input_dataset.yaml  pipeline_state  scaffolds.fasta  tmp
K55  assembly_graph_with_scaffolds.gfa        contigs.paths    misc                run_spades.sh   scaffolds.paths
```

### O que significa cada arquivo?

| Arquivo ou diretório | Descrição |
|---|---|
| `contigs.fasta` | Arquivo FASTA com os contigs finais da montagem. |
| `scaffolds.fasta` | Arquivo FASTA com os scaffolds finais. |
| `K21`, `K33` e `K55` | Diretórios referentes aos valores de k-mer usados pelo SPAdes durante a montagem. |
| `spades.log` | Arquivo de registro com todas as etapas executadas pelo SPAdes. |
| `params.txt` | Arquivo com os parâmetros utilizados na montagem. |
| `run_spades.sh` | Script com o comando utilizado para executar o SPAdes. |
| `run_spades.yaml` | Arquivo de configuração da execução do SPAdes. |
| `assembly_graph.fastg` | Grafo da montagem no formato FastG. |
| `assembly_graph_after_simplification.gfa` | Grafo da montagem após etapas de simplificação. |
| `assembly_graph_with_scaffolds.gfa` | Grafo da montagem contendo informações dos scaffolds. |
| `contigs.paths` | Caminhos dos contigs no grafo de montagem. |
| `scaffolds.paths` | Caminhos dos scaffolds no grafo de montagem. |
| `before_rr.fasta` | Sequências geradas antes da etapa de resolução de repetições. |
| `tmp` | Diretório temporário utilizado pelo SPAdes. |
| `misc` | Diretório com arquivos auxiliares da montagem. |

---

## 3. Contigs e scaffolds

Inicialmente, vamos avaliar os arquivos `contigs.fasta` e `scaffolds.fasta`.

### Contando os contigs

Use o comando abaixo para contar o número de sequências (contigs) no arquivo FASTA:

```bash
grep -c '>' contigs.fasta
```

Resultado:

```text
397
```

O comando `grep` procura um padrão dentro de um arquivo.

- `grep` procura um texto ou padrão;
- `-c` conta o número de linhas encontradas;
- `'>'` procura cabeçalhos de sequências FASTA;
- `contigs.fasta` é o arquivo analisado.

Em arquivos FASTA, cada sequência começa com o símbolo `>`. Portanto, o resultado `397` indica que a montagem possui **397 contigs**.

### Contando os scaffolds

Agora, conte as sequências presentes no arquivo de scaffolds:

```bash
grep -c '>' scaffolds.fasta
```

Resultado:

```text
391
```

Esse resultado indica que foram obtidos **391 scaffolds**.

### Diferença entre contigs e scaffolds

Um **contig** é uma sequência contínua montada diretamente a partir da sobreposição entre as leituras de sequenciamento. Ele não possui lacunas internas.

Um **scaffold** é uma estrutura maior, formada quando dois ou mais contigs são conectados utilizando informações das leituras paired-end. Quando não é possível determinar exatamente a sequência entre dois contigs, a região desconhecida é representada por letras `N`.

Neste resultado, existem 397 contigs e 391 scaffolds. A quantidade de scaffolds é menor porque alguns contigs foram conectados durante a etapa de scaffolding.

---

## 4. Verificando os parâmetros utilizados na montagem

Para visualizar os parâmetros utilizados pelo SPAdes, execute:

```bash
cat params.txt
```

Exemplo de resultado:

```text
Command line: /home/daisy/anaconda3/envs/bioinfo/bin/spades.py --isolate -1 /home/daisy/Daisy/aula2/2_trimmomatic/SRR11011985_1_paired.fastq -2 /home/daisy/Daisy/aula2/2_trimmomatic/SRR11011985_2_paired.fastq -o /home/daisy/Daisy/aula2/4_assembly -t 4 -k auto

SPAdes version: 4.3.0
Python version: 3.11.15
Output dir: /home/daisy/Daisy/aula2/4_assembly

Dataset parameters:
Isolate mode
Library type: paired-end
orientation: fr

Assembly parameters:
k: automatic selection based on read length
Repeat resolution is enabled
Coverage cutoff is turned OFF

Other parameters:
Threads: 4
Memory limit (in Gb): 9
```

### Interpretação dos principais parâmetros

- `spades.py`: programa utilizado para realizar a montagem.
- `--isolate`: modo recomendado para a montagem de genomas de isolados, como bactérias.
- `-1`: arquivo contendo as leituras forward, ou seja, as leituras R1.
- `-2`: arquivo contendo as leituras reverse, ou seja, as leituras R2.
- `-o`: diretório onde os resultados da montagem foram salvos.
- `-t 4`: utilização de até quatro núcleos de processamento.
- `-k auto`: seleção automática dos valores de k-mer pelo SPAdes.
- `paired-end`: informa que os dados são provenientes de sequenciamento pareado.
- `orientation: fr`: indica que as leituras estão na orientação forward-reverse.
- `Repeat resolution is enabled`: a resolução de regiões repetitivas foi ativada.
- `Memory limit: 9`: limite máximo de memória RAM utilizado na análise.

---

## 5. Verificando os k-mers utilizados pelo SPAdes

Para verificar os valores de k-mer empregados na montagem, execute:

```bash
grep -i "K21\|K33\|K55" spades.log
```

O resultado mostrará informações semelhantes a:

```text
===== K21 started.
===== K21 finished.
===== K33 started.
===== K33 finished.
===== K55 started.
===== K55 finished.
```

O **k-mer** é uma sequência de DNA com tamanho `k`. Durante a montagem, o SPAdes quebra as leituras em fragmentos menores e utiliza a sobreposição desses fragmentos para reconstruir o genoma.

Neste caso, foram usados os valores:

- `K21`: fragmentos de 21 nucleotídeos;
- `K33`: fragmentos de 33 nucleotídeos;
- `K55`: fragmentos de 55 nucleotídeos.

O SPAdes utiliza diferentes tamanhos de k-mer porque cada valor apresenta vantagens diferentes. Valores menores podem ajudar a montar regiões com baixa cobertura, enquanto valores maiores podem ajudar a resolver regiões repetitivas.

---

## 6. Visualizando os cabeçalhos dos scaffolds

Para observar os cabeçalhos das sequências presentes no arquivo `scaffolds.fasta`, utilize:

```bash
grep '>' scaffolds.fasta | head
```

Exemplo de resultado:

```text
>NODE_1_length_114875_cov_50.046804
>NODE_2_length_110180_cov_57.786007
>NODE_3_length_78937_cov_41.270226
>NODE_4_length_78411_cov_50.849227
>NODE_5_length_74250_cov_33.067956
>NODE_6_length_72856_cov_68.161976
>NODE_7_length_65612_cov_37.985555
>NODE_8_length_60999_cov_56.700233
>NODE_9_length_58562_cov_43.728785
>NODE_10_length_58320_cov_68.465631
```

Cada cabeçalho possui informações importantes sobre o scaffold.

Exemplo:

```text
>NODE_1_length_114875_cov_50.046804
```

| Parte do cabeçalho | Significado |
|---|---|
| `NODE_1` | Identificador da sequência. |
| `length_114875` | Comprimento do scaffold: 114.875 pares de bases. |
| `cov_50.046804` | Cobertura média estimada: aproximadamente 50 vezes. |

A cobertura indica, aproximadamente, quantas vezes cada base da sequência foi observada nas leituras. Em geral, uma boa cobertura aumenta a confiança na montagem, embora valores muito altos ou muito baixos também devam ser investigados.

---

## 7. Ativando o ambiente virtual

Antes de utilizar as ferramentas de análise, ative o ambiente virtual criado nas aulas anteriores:

```bash
conda activate bioinfo
```

Após a ativação, o terminal deverá mostrar algo semelhante a:

```bash
(bioinfo) daisy@Daisy:~/Daisy/aula2_parte2/4_assembly$
```

---

## 8. Estatísticas com SeqKit

Primeiro, verifique se o programa SeqKit está disponível:

```bash
seqkit
```

Se o programa estiver instalado corretamente, serão apresentadas suas opções de uso.

### Estatísticas básicas

Execute:

```bash
seqkit stats contigs.fasta scaffolds.fasta
```

Resultado:

```text
file             format  type  num_seqs    sum_len  min_len  avg_len  max_len
contigs.fasta    FASTA   DNA        397  1,875,984       56  4,725.4  114,875
scaffolds.fasta  FASTA   DNA        391  1,876,502       56  4,799.2  114,875
```

### Interpretação das colunas

| Coluna | Significado |
|---|---|
| `file` | Nome do arquivo analisado. |
| `format` | Formato do arquivo; neste caso, FASTA. |
| `type` | Tipo de sequência; neste caso, DNA. |
| `num_seqs` | Número de sequências presentes no arquivo. |
| `sum_len` | Soma do tamanho de todas as sequências. |
| `min_len` | Tamanho da menor sequência. |
| `avg_len` | Tamanho médio das sequências. |
| `max_len` | Tamanho da maior sequência. |

Os contigs possuem tamanho total de **1.875.984 pb**, enquanto os scaffolds possuem tamanho total de **1.876.502 pb**.

Os scaffolds possuem um tamanho total ligeiramente maior porque podem ser introduzidos gaps, representados por bases desconhecidas (N), entre contigs. Essas regiões contribuem para o tamanho total do scaffold, mesmo sem representar uma sequência determinada.

### Estatísticas detalhadas (-a = addition)

Execute:

```bash
seqkit stats -a contigs.fasta scaffolds.fasta
```

Resultado:

```text
file             format  type  num_seqs    sum_len  min_len  avg_len  max_len    Q1   Q2     Q3  sum_gap     N50  N50_num  Q20(%)  Q30(%)  AvgQual  GC(%)  sum_n
contigs.fasta    FASTA   DNA        397  1,875,984       56  4,725.4  114,875    96  116    416        0  40,956       14       0       0        0  52.35      0
scaffolds.fasta  FASTA   DNA        391  1,876,502       56  4,799.2  114,875  94.5  114  371.5      600  43,280       14       0       0        0  52.33    600
```

### Interpretação das métricas adicionais

| Coluna | Significado |
|---|---|
| `Q1` | Primeiro quartil do tamanho das sequências. |
| `Q2` | Mediana do tamanho das sequências. |
| `Q3` | Terceiro quartil do tamanho das sequências. |
| `sum_gap` | Número total de bases presentes em lacunas. |
| `N50` | Comprimento em que 50% da montagem está em sequências desse tamanho ou maiores. |
| `N50_num` | Número de sequências necessárias para alcançar 50% do tamanho total da montagem. |
| `GC(%)` | Porcentagem de bases G e C no genoma. |
| `sum_n` | Número total de letras `N` presentes no arquivo. |

O valor de **N50** dos contigs foi de **40.956 pb**, enquanto o N50 dos scaffolds foi de **43.280 pb**.

Isso indica que os scaffolds são, em geral, mais longos que os contigs, pois alguns contigs foram conectados durante o processo de montagem.

O conteúdo GC foi de aproximadamente **52,3%**, valor compatível com um genoma bacteriano.

As colunas `Q20(%)`, `Q30(%)` e `AvgQual` são mais relevantes para arquivos FASTQ. Como os arquivos analisados são FASTA, essas métricas aparecem como zero.

---

## 9. Visualizando os identificadores com SeqKit

Para visualizar somente os cabeçalhos das sequências no arquivo de scaffolds, execute:

```bash
seqkit fx2tab -n -i scaffolds.fasta
```

| Comando | Significado |
|---|---|
| `seqkit` | Ferramenta para manipulação e análise de arquivos FASTA/FASTQ. |
| `stats` | Calcula estatísticas das sequências (número de sequências, tamanho total, tamanho médio, N50, GC%, etc.). |
| `fx2tab` | Converte arquivos FASTA/FASTQ para formato de tabela. |
| `-n` | Mostra apenas o nome/ID das sequências (header). |
| `-i` | Mostra o identificador das sequências. |

Para visualizar apenas os dez primeiros cabeçalhos (mesmo comando acima + 'head'):

```bash
seqkit fx2tab -n -i scaffolds.fasta | head
```

- `head`: exibe apenas as primeiras linhas.

---

## 10. Contando as letras N nos scaffolds

Para contar quantas letras `N` estão presentes no arquivo de scaffolds, execute:

```bash
grep -o "N" scaffolds.fasta | wc -l
```

O resultado esperado é:

```text
600
```

### Explicação do comando

- `grep -o "N" scaffolds.fasta`: procura cada ocorrência da letra `N` no arquivo;
- `-o`: faz com que cada ocorrência encontrada seja mostrada em uma linha;
- `|`: envia o resultado de um comando para o próximo;
- `wc -l`: conta o número de linhas recebidas.

As letras `N` representam bases desconhecidas ou lacunas entre contigs conectados em um scaffold.

Neste caso, foram encontradas **600 bases N** distribuídas nos scaffolds.

---

## 11. Estatísticas com Assembly-stats

Verifique se o programa está disponível:

```bash
assembly-stats
```

Caso esteja instalado, serão exibidas as opções de uso.

Agora, execute a análise do arquivo de scaffolds:

```bash
assembly-stats scaffolds.fasta
```

Resultado:

```text
stats for scaffolds.fasta
sum = 1876502, n = 391, ave = 4799.24, largest = 114875
N50 = 43280, n = 14
N60 = 31904, n = 19
N70 = 24750, n = 25
N80 = 18574, n = 34
N90 = 10788, n = 47
N100 = 56, n = 391
N_count = 600
Gaps = 6
```

### Interpretação do resultado

| Métrica | Resultado | Interpretação |
|---|---:|---|
| `sum` | 1.876.502 pb | Tamanho total da montagem. |
| `n` | 391 | Número total de scaffolds. |
| `ave` | 4.799,24 pb | Tamanho médio dos scaffolds. |
| `largest` | 114.875 pb | Tamanho do maior scaffold. |
| `N50` | 43.280 pb | Metade do genoma montado está em scaffolds com tamanho igual ou maior que esse valor. |
| `N60` | 31.904 pb | 60% da montagem está em scaffolds com tamanho igual ou maior que esse valor. |
| `N70` | 24.750 pb | 70% da montagem está em scaffolds com tamanho igual ou maior que esse valor. |
| `N80` | 18.574 pb | 80% da montagem está em scaffolds com tamanho igual ou maior que esse valor. |
| `N90` | 10.788 pb | 90% da montagem está em scaffolds com tamanho igual ou maior que esse valor. |
| `N100` | 56 pb | Menor scaffold presente na montagem. |
| `N_count` | 600 | Número total de bases desconhecidas, representadas por `N`. |
| `Gaps` | 6 | Número de lacunas presentes nos scaffolds. |

---

## Conclusões finais

A montagem do genoma de *Lactobacillus fermentum* foi gerada pelo SPAdes a partir de leituras paired-end previamente tratadas.

Foram obtidos **397 contigs** e **391 scaffolds**, indicando que alguns contigs foram conectados com o uso das informações das leituras pareadas.

O tamanho total da montagem foi de aproximadamente **1,88 Mb**, com conteúdo GC de cerca de **52,3%**. O maior scaffold apresentou **114.875 pb**.

O valor de **N50 de 43.280 pb** indica que uma parcela importante da montagem está concentrada em scaffolds relativamente longos. Quanto maior o N50, menor tende a ser a fragmentação da montagem, embora essa métrica deva ser interpretada junto a outras informações.

---

## Perguntas de revisão

1. O que é um contig e scaffold?

2. O que representa a letra `N` em um scaffold?

3. Quantos contigs e scaffolds foram encontrados nesta montagem?

4. Qual foi o valor de N50 dos scaffolds?

5. Qual foi o tamanho total da montagem?

6. Qual foi o conteúdo GC aproximado da montagem?

7. Para que serve o arquivo `params.txt` e `spades.log`?

8. O que são k-mers?

9. Quais valores de k-mer foram usados pelo SPAdes nesta montagem?

10. Qual comando pode ser utilizado para contar o número de sequências em um arquivo FASTA?



