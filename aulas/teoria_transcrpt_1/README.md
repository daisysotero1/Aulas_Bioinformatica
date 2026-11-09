# Estudo Dirigido – Introdução à Transcriptômica e RNA-seq

## Objetivo

Compreender os fundamentos da transcriptômica e das principais etapas de uma análise de RNA-seq, desde a obtenção dos dados de sequenciamento até a identificação e interpretação de genes diferencialmente expressos.

A atividade tem como objetivo desenvolver a capacidade de interpretar conceitos, compreender as estratégias de análise bioinformática e relacionar os resultados obtidos com processos biológicos.
---

## Referências para estudo

### Leituras essenciais

**1. A survey of best practices for RNA-seq data analysis**

Artigo de revisão que apresenta recomendações e boas práticas para diferentes etapas de uma análise de RNA-seq.

Link: https://link.springer.com/article/10.1186/s13059-016-0881-8

**2. A Beginner’s Guide to Analysis of RNA Sequencing Data**

Artigo voltado à introdução da análise de dados de RNA-seq, abordando conceitos e etapas fundamentais para a compreensão de experimentos transcriptômicos.

Link: https://pmc.ncbi.nlm.nih.gov/articles/PMC6096346/

### Leitura opcional – Aprofundamento

**RNA Sequencing Data: Hitchhiker's Guide to Expression Analysis**

Link: https://www.annualreviews.org/content/journals/10.1146/annurev-biodatasci-072018-021255

---

## Atividade assíncrona

### De acordo com a aula, e leitura e interpretação de artigos científicos, responda às seguintes questões:

**1.** O que é o transcriptoma de uma célula ou organismo? Por que ele pode variar entre diferentes tecidos ou condições ambientais?

**2.** Qual é a principal diferença entre RNA-seq e técnicas tradicionais de análise da expressão gênica, como o microarray?

**3.** Durante um experimento de RNA-seq, por que é necessário realizar a extração e a avaliação da qualidade do RNA antes do sequenciamento? O que poderia acontecer se o RNA estivesse degradado?

**4.** Explique a diferença entre o alinhamento de reads a um genoma de referência e a montagem de um transcriptoma *de novo*. Em quais situações cada abordagem pode ser utilizada?

**5.** O que são reads em um experimento de RNA-seq? Qual é a importância de avaliar sua qualidade antes de realizar o alinhamento?

**6.** Por que a expressão gênica não pode ser comparada diretamente entre amostras utilizando apenas o número bruto de reads de cada gene? Explique a importância da normalização.

**7.** Qual é a diferença entre TPM e contagens brutas (*raw counts*)? Por que métodos como DESeq2 utilizam contagens brutas para análise de expressão diferencial?

**8.** O que significa identificar genes diferencialmente expressos entre duas condições experimentais? Explique o significado de um log₂ Fold Change positivo e de um log₂ Fold Change negativo.

**9.** Por que a utilização de réplicas biológicas é importante em experimentos de RNA-seq? O que poderia acontecer se um experimento fosse realizado com apenas uma amostra por condição?

**10.** Após identificar genes diferencialmente expressos, por que é importante realizar uma análise de enriquecimento funcional, como GO ou KEGG? Como essa análise pode contribuir para a interpretação biológica dos resultados?

---

## Orientações para a atividade

- Utilize linguagem científica clara e objetiva.
- Sempre que possível, relacione os conceitos apresentados nos artigos e em aula com as etapas de uma análise de RNA-seq.
- As respostas devem demonstrar compreensão dos conceitos, evitando apenas a reprodução literal de trechos dos artigos.

