## 11. Análise de expressão diferencial de genes com DESeq2

Após a quantificação das reads associadas a cada gene utilizando o featureCounts, a próxima etapa da análise de RNA-seq é a **análise de expressão diferencial (Differential Gene Expression Analysis)**.

Essa análise permite identificar quais genes apresentam diferenças significativas de expressão entre duas ou mais condições experimentais.

Nesta atividade, utilizaremos o pacote **DESeq2**, desenvolvido para análise de dados de RNA-seq. O DESeq2 utiliza modelos estatísticos baseados na distribuição binomial negativa para identificar diferenças na expressão dos genes entre as condições experimentais.

### 11.1. Preparação dos metadados

Antes de iniciar a análise, precisamos preparar um arquivo de metadados (*metadata*).

Os metadados são informações que descrevem as amostras utilizadas no experimento. Nesse caso, precisamos identificar a qual grupo experimental cada amostra pertence.

Essas informações são utilizadas pelo DESeq2 para estabelecer o delineamento experimental (*experimental design*) e definir quais condições serão comparadas.

Neste exemplo, temos dois grupos experimentais:

- **Control:** grupo controle.
- **Treated:** grupo tratado.

A tabela de metadados deve apresentar uma linha para cada amostra:

| Amostra | Group |
|:---|:---|
| CT_01 | Control |
| CT_02 | Control |
| CT_03 | Control |
| GL_01 | Treated |
| GL_02 | Treated |
| GL_03 | Treated |

Salve essa tabela no formato CSV (MS-DOS), com o nome `dmelanogaster_metadata.csv`.

Em seguida, transfira o arquivo para a pasta `6_featureCounts` do projeto no WSL.

Para abrir a pasta atual do WSL pelo Explorador de Arquivos do Windows, utilize:

```bash
explorer.exe .
```

### 11.2. Organização dos arquivos

No Ubuntu/WSL, dentro do diretório `rna-seq`, teremos a seguinte organização:

```bash
(base) daisy@Daisy:~/Daisy/rna-seq$ ls -l

total 32
drwxr-xr-x 2 daisy daisy 4096 Nov 11 18:40 0_referencia
drwxr-xr-x 2 daisy daisy 4096 Nov 11 18:19 1_reads
drwxr-xr-x 2 daisy daisy 4096 Nov 11 18:20 2_qc_brutos
drwxr-xr-x 2 daisy daisy 4096 Nov 11 18:25 3_reads_trimadas
drwxr-xr-x 2 daisy daisy 4096 Nov 11 18:46 4_qc_trimados
drwxr-xr-x 2 daisy daisy 4096 Nov 11 19:17 5_mapeamento
drwxr-xr-x 2 daisy daisy 4096 Nov 13 17:03 6_featureCounts
```

Dentro do diretório `6_featureCounts`, teremos os seguintes arquivos:

```bash
(base) daisy@Daisy:~/Daisy/rna-seq$ ls 6_featureCounts/

counts.txt
counts.txt.summary
dmelanogaster_counts.csv
dmelanogaster_metadata.csv
featureCounts.log
```

Os arquivos utilizados na análise serão:

- `dmelanogaster_counts.csv`: matriz de contagem contendo o número de reads atribuídas a cada gene em cada amostra.
- `dmelanogaster_metadata.csv`: arquivo contendo os metadados das amostras e seus respectivos grupos experimentais.

### 11.3. Análise de expressão diferencial utilizando DESeq2

Abra o RStudio, cole e execute o código abaixo.

```r
############################################################
# ANÁLISE DE EXPRESSÃO DIFERENCIAL DE GENES COM DESeq2
############################################################

# Objetivo:
# Identificar genes diferencialmente expressos (DEGs) entre
# dois grupos experimentais, utilizando dados de RNA-seq.

# O DESeq2 utiliza modelos estatísticos baseados na
# distribuição binomial negativa para identificar diferenças
# na expressão dos genes entre as condições experimentais.


############################################################
# 1. INSTALAÇÃO E CARREGAMENTO DOS PACOTES
############################################################

# Verificar se os pacotes já estão instalados
"DESeq2" %in% rownames(installed.packages())
"tidyverse" %in% rownames(installed.packages())

# Instalar o BiocManager, caso ainda não esteja instalado
if (!requireNamespace("BiocManager", quietly = TRUE)) {
  install.packages("BiocManager")
}

# Instalar o DESeq2 pelo Bioconductor
if (!requireNamespace("DESeq2", quietly = TRUE)) {
  BiocManager::install("DESeq2")
}

# Instalar o tidyverse pelo CRAN
if (!requireNamespace("tidyverse", quietly = TRUE)) {
  install.packages("tidyverse")
}

# Carregar os pacotes
library(DESeq2)
library(tidyverse)


############################################################
# 2. DEFINIR O DIRETÓRIO DE TRABALHO
############################################################

# Verificar o diretório de trabalho atual
getwd()

# Definir o diretório onde estão os arquivos de entrada.
# O caminho abaixo corresponde ao caminho do Ubuntu/WSL.
setwd("//wsl$/Ubuntu/home/daisy/Daisy/rna-seq/6_featureCounts")

# Listar os arquivos disponíveis no diretório
dir()


############################################################
# 3. IMPORTAR A MATRIZ DE CONTAGEM E OS METADADOS
############################################################

# A matriz de contagem contém o número de reads atribuídas
# a cada gene em cada amostra.

# Linhas: genes
# Colunas: amostras

# IMPORTANTE: o DESeq2 deve receber contagens brutas
# (inteiras), e não valores normalizados, TPM ou FPKM.

counts_data <- read.csv2(
  "dmelanogaster_counts.csv",
  header = TRUE,
  row.names = 1,
  check.names = FALSE
)

# Visualizar as primeiras linhas da matriz
head(counts_data)

# Verificar as dimensões da matriz:
# número de genes x número de amostras
dim(counts_data)


# IMPORTAR OS METADADOS

# Os metadados contêm informações sobre cada amostra,
# incluindo o grupo experimental ao qual ela pertence.

# Cada linha representa uma amostra.
# A coluna Group identifica o grupo experimental.

metadata <- read.csv2(
  "dmelanogaster_metadata.csv",
  header = TRUE,
  row.names = 1,
  check.names = FALSE
)

# Visualizar os metadados
head(metadata)

# Verificar a quantidade de amostras e variáveis
dim(metadata)

# Converter Group em fator.
# Essa variável representa as condições experimentais
# que serão comparadas pelo DESeq2.
class(metadata$Group)
# "character"
metadata$Group <- factor(metadata$Group)
class(metadata$Group)
# Conferir ordem dos grupos experimentais
levels(metadata$Group)
# Verificar a quantidade de amostras em cada grupo
table(metadata$Group)


############################################################
# 4. CONFERIR A CORRESPONDÊNCIA ENTRE AMOSTRAS
############################################################

# É fundamental garantir que as colunas da matriz de
# contagem correspondam às linhas dos metadados.

# Além disso, as amostras devem estar exatamente na mesma
# ordem nos dois arquivos.

all(colnames(counts_data) == rownames(metadata))

# O resultado deve ser TRUE.
# Caso seja FALSE, é necessário corrigir a ordem das amostras
# antes de continuar a análise.


############################################################
# 5. CONSTRUIR O OBJETO DESeqDataSet
############################################################

# O DESeqDataSet é o objeto principal utilizado pelo DESeq2.

# Ele reúne:
# - A matriz de contagem dos genes;
# - Os metadados das amostras;
# - O delineamento experimental (design).

# O design = ~ Group indica que queremos avaliar diferenças
# na expressão gênica associadas ao grupo experimental.

deseq_dataset <- DESeqDataSetFromMatrix(
  countData = counts_data,
  colData = metadata,
  design = ~ Group
)

# Visualizar o objeto criado
deseq_dataset

# Conferir a matriz de contagem dentro do objeto
#Tabela original
counts(deseq_dataset)[1:6, 1:4]


############################################################
# 6. FILTRAR GENES COM BAIXA CONTAGEM
############################################################

# Genes com pouquíssimas reads em todas as amostras
# geralmente fornecem pouca informação para a análise
# estatística e podem aumentar o custo computacional.

# Neste exemplo, serão mantidos apenas genes cuja soma
# das contagens em todas as amostras seja >= 10.

# ATENÇÃO: o filtro considera a soma das reads de cada gene
# entre todas as amostras, e não a contagem individual
# em cada amostra.

deseq_dataset <- deseq_dataset[
  rowSums(counts(deseq_dataset)) >= 10,
]

# Conferir as dimensões após a filtragem
dim(deseq_dataset)


############################################################
# 7. DEFINIR O GRUPO DE REFERÊNCIA
############################################################

# O grupo de referência é utilizado como base para calcular
# as diferenças de expressão entre as condições.

# Neste exemplo, Control será o grupo de referência.

# Assim, o log2FoldChange será calculado considerando
# a expressão do outro grupo em relação ao Control.

deseq_dataset$Group <- relevel(
  deseq_dataset$Group,
  ref = "Control"
)

# Conferir os níveis do fator
levels(deseq_dataset$Group)

# O primeiro nível deve ser Control.


############################################################
# 8. EXECUTAR A ANÁLISE COM DESeq2
############################################################

# A função DESeq() executa as principais etapas da análise:

# 1. Estima os fatores de tamanho (size factors), utilizados
#    para normalizar as diferenças de profundidade de
#    sequenciamento entre as amostras.

# 2. Estima a dispersão dos genes, considerando a variabilidade
#    biológica observada entre as amostras.

# 3. Ajusta modelos estatísticos baseados na distribuição
#    binomial negativa para cada gene.

# 4. Realiza os testes estatísticos para identificar
#    diferenças de expressão entre os grupos.

deseq_dataset <- DESeq(deseq_dataset)

# Visualizar o objeto após a análise
deseq_dataset


############################################################
# 9. OBTER (nao é normalizar) AS CONTAGENS NORMALIZADAS
############################################################

# As contagens brutas podem variar entre amostras devido
# a diferenças na profundidade de sequenciamento.

# O DESeq2 estima fatores de tamanho (size factors) para
# corrigir essas diferenças.

# A normalização permite comparar os níveis de expressão
# entre amostras de maneira mais adequada.

# IMPORTANTE:
# A normalização não transforma os dados em TPM ou FPKM.
# São contagens ajustadas pelos fatores de tamanho.

normalized_counts <- counts(
  deseq_dataset,
  normalized = TRUE
)

# Visualizar as primeiras linhas

dim(normalized_counts)

head(normalized_counts)
head(counts_data) #comparar com a tabela original

# Salvar as contagens normalizadas
# write.csv(
#   as.data.frame(normalized_counts),
#   "dmelanogaster_contagens_normalizadas.csv",
#   row.names = TRUE
# )


############################################################
# 10. ANÁLISE EXPLORATÓRIA: PCA
############################################################

# O PCA (Principal Component Analysis) é uma análise
# exploratória utilizada para avaliar a distribuição
# e a similaridade entre as amostras.

# Permite verificar:
# - Se as amostras do mesmo grupo apresentam comportamento
#   semelhante;
# - Se os grupos experimentais apresentam separação;
# - Se existem amostras discrepantes (possíveis outliers).

# O PCA é uma análise exploratória e não identifica,
# por si só, genes diferencialmente expressos.

# Por que não utilizar diretamente as contagens brutas?

# As contagens apresentam diferenças de profundidade de
# sequenciamento e variância dependente da média.
# Por isso, utilizamos uma transformação estabilizadora
# da variância (VST), após a normalização.

#VST é uma transformação dos dados já normalizados, com o
#objetivo de torná-los mais adequados para análises exploratórias.

vsd <- vst(
  deseq_dataset, #resultado do DESEq2
  blind = FALSE
)

# Obter a matriz de expressão transformada
vst_counts <- assay(vsd)

# Visualizar as primeiras linhas
head(vst_counts)


# REALIZAR O PCA

# A função plotPCA seleciona automaticamente os 500 genes
# mais variáveis para realizar a análise.

# As amostras precisam estar nas linhas e os genes
# nas colunas. A função já realiza essa organização.

pca_data <- plotPCA(
  vsd,
  intgroup = "Group",
  ntop = 500,
  returnData = TRUE
)

# Obter a porcentagem de variância explicada
# pelos dois primeiros componentes principais.

percentVar <- round(
  100 * attr(pca_data, "percentVar"),
  1
)

# Visualizar os resultados do PCA
head(pca_data)


# CONSTRUIR O GRÁFICO DO PCA
# CONSTRUIR O GRÁFICO DO PCA
# CONSTRUIR O GRÁFICO DO PCA

pca_plot <- ggplot(
  pca_data,
  aes(
    x = PC1,
    y = PC2,
    color = Group
  )
) +

  # Pontos representando as amostras
  geom_point(
    size = 4,
    alpha = 0.9
  ) +

  # Identificação dos eixos e legenda
  labs(
    title = "Principal Component Analysis",
    x = paste0("PC1 (", percentVar[1], "%)"),
    y = paste0("PC2 (", percentVar[2], "%)"),
    color = "Treatment"
  ) +

  # Tema visual
  theme_classic(base_size = 14) +

  theme(
    plot.title = element_text(
      size = 16,
      face = "bold",
      hjust = 0.5
    ),

    axis.title = element_text(
      size = 13,
      face = "bold"
    ),

    axis.text = element_text(
      size = 11,
      color = "black"
    ),

    axis.line = element_line(
      linewidth = 0.7,
      color = "black"
    ),

    legend.title = element_text(
      face = "bold",
      size = 12
    ),

    legend.text = element_text(size = 11),

    legend.position = "bottom",

    plot.margin = margin(15, 15, 15, 15)
  )

# Exibir o gráfico
print(pca_plot)

# Salvar o gráfico em alta resolução
# ggsave(
#   "dmelanogaster_PCA.png",
#   plot = pca_plot,
#   width = 8,
#   height = 6,
#   dpi = 300
# )


############################################################
# 11. OBTER OS RESULTADOS DA EXPRESSÃO DIFERENCIAL
############################################################

# A função results() extrai os resultados dos testes
# estatísticos realizados pelo DESeq2.

# Como Control foi definido como grupo de referência,
# a comparação será realizada entre o outro grupo
# experimental e Control.

# alpha = 0.05 define o nível de significância utilizado
# pelo DESeq2 para o procedimento de filtragem independente.

resultados <- results(
  deseq_dataset, #resultado do DESeq2
  alpha = 0.05
)

# Visualizar os resultados
resultados

# Converter para data.frame, se necessário
resultados_df <- as.data.frame(resultados)
resultados_df

#write.csv(resultados_df,"resultados_DESeq2.csv",row.names = TRUE)

############################################################
# 12. ENTENDER AS COLUNAS DOS RESULTADOS
############################################################

# Cada linha representa um gene analisado.

# As principais colunas são:

# baseMean:
# Média das contagens normalizadas do gene considerando
# todas as amostras.

# log2FoldChange:
# Indica a magnitude e a direção da diferença de expressão
# entre os grupos experimentais.

# Valores positivos: maior expressão no grupo experimental
# em relação ao Control.

# Valores negativos: menor expressão no grupo experimental
# em relação ao Control.

# lfcSE:
# Erro padrão (standard error) da estimativa do log2FoldChange.
# Indica a incerteza associada à estimativa.

# stat:
# Estatística do teste de Wald, calculada a partir da
# estimativa do log2FoldChange e seu erro padrão.

# pvalue:
# Valor de p não ajustado, utilizado para avaliar a
# significância estatística de cada gene.

# padj:
# Valor de p ajustado para múltiplos testes pelo método
# de Benjamini-Hochberg (controle da taxa de falsas
# descobertas - FDR).

# A coluna padj é utilizada para selecionar os genes
# diferencialmente expressos, pois milhares de genes
# são testados simultaneamente.


############################################################
# 13. RESUMO DOS RESULTADOS
############################################################

# A função summary() apresenta um resumo da análise,
# incluindo a quantidade de genes com resultados
# significativos e a direção da alteração de expressão.

# O resumo considera o padj e por padrão, o limiar de
#log2FoldChange é zero.

summary(resultados)

# O argumento alpha = 0.05 foi definido anteriormente
# em results(), de modo que o resumo utiliza esse limiar.


############################################################
# 14. SELECIONAR OS GENES DIFERENCIALMENTE EXPRESSOS (DEGs)
############################################################

# Para selecionar os DEGs, serão utilizados dois critérios:

# 1. padj < 0.05:
#    Indica significância estatística após correção
#    para múltiplos testes.

# 2. |log2FoldChange| >= 1:
#    Indica uma variação de expressão de pelo menos
#    duas vezes (fold change >= 2 ou <= 0.5).

# O valor absoluto (abs) permite selecionar tanto os genes
# com expressão aumentada quanto aqueles com expressão
# reduzida.

# IMPORTANTE:
# log2FoldChange >= 1  -> expressão pelo menos 2 vezes maior.
# log2FoldChange <= -1 -> expressão pelo menos 2 vezes menor.

DEGs <- subset(
  resultados_df,
  !is.na(padj) &
    padj < 0.05 &
    abs(log2FoldChange) >= 1
)

# Visualizar os DEGs
head(DEGs)

# Quantidade total de genes diferencialmente expressos
nrow(DEGs)


############################################################
# 15. SEPARAR GENES AUMENTADOS E REDUZIDOS
############################################################

# GENES AUMENTADOS (UPREGULATED)

# Genes que apresentam expressão pelo menos duas vezes
# maior no grupo experimental em relação ao Control.

# Critérios:
# padj < 0.05
# log2FoldChange >= 1

DEGs_up <- subset(
  resultados_df,
  !is.na(padj) &
    padj < 0.05 &
    log2FoldChange >= 1
)


# GENES REDUZIDOS (DOWNREGULATED)

# Genes que apresentam expressão pelo menos duas vezes
# menor no grupo experimental em relação ao Control.

# Critérios:
# padj < 0.05
# log2FoldChange <= -1

DEGs_down <- subset(
  resultados_df,
  !is.na(padj) &
    padj < 0.05 &
    log2FoldChange <= -1
)


############################################################
# 16. CONTABILIZAR OS GENES DIFERENCIALMENTE EXPRESSOS
############################################################

# Quantidade total de DEGs
nrow(DEGs)

# Quantidade de genes aumentados
nrow(DEGs_up)

# Quantidade de genes reduzidos
nrow(DEGs_down)

# Conferir se a soma dos genes aumentados e reduzidos
# corresponde ao total de DEGs.

nrow(DEGs_up) + nrow(DEGs_down)


############################################################
# 17. EXPORTAR OS RESULTADOS
############################################################

# Salvar os resultados em arquivos CSV.

# Os arquivos serão salvos no diretório de trabalho atual,
# que pode ser consultado utilizando getwd().

# row.names = TRUE mantém os identificadores dos genes
# que estão armazenados nos nomes das linhas.

# Todos os genes diferencialmente expressos
write.csv(
  DEGs,
  "DEGs.csv",
  row.names = TRUE
)

# Genes com expressão aumentada
#write.csv(
# DEGs_up,
# "DEGs_up.csv",
# row.names = TRUE
#)

# Genes com expressão reduzida
#write.csv(
# DEGs_down,
# "DEGs_down.csv",
# row.names = TRUE
#)


############################################################
# FIM DA ANÁLISE
############################################################
```
A interpretação dos resultados considera principalmente:

- **log2FoldChange positivo:** genes com maior expressão no grupo Treated em relação ao Control.
- **log2FoldChange negativo:** genes com menor expressão no grupo Treated em relação ao Control.
- **padj menor que 0,05:** genes com diferenças estatisticamente significativas após correção para múltiplos testes.
- **|log2FoldChange| maior ou igual a 1:** genes com variação de expressão de pelo menos duas vezes.

Os genes identificados podem posteriormente ser utilizados em análises funcionais, como enriquecimento de Gene Ontology (GO), análise de vias metabólicas (KEGG) e construção de redes de interação gênica.

### 11.4. Visualização dos resultados por volcano plot - Em python

Ao final da análise, teremos o arquivo `DEGs.csv`, contendo os genes diferencialmente expressos entre os grupos experimentais. Esse arquivo será input para o volcano plot.


