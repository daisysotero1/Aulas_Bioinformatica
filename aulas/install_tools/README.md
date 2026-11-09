# Instalação de ferramentas para análise de RNA-seq

## 1. Introdução

Para realizar uma análise de RNA-seq, é necessário utilizar diferentes ferramentas de bioinformática, responsáveis por etapas como controle de qualidade, trimagem, mapeamento e contagem de reads.

Neste tutorial, utilizaremos o **Conda** para instalar e organizar as ferramentas necessárias para a análise.

As ferramentas serão instaladas em um ambiente virtual denominado `rnaseq`.

## 2. O que é um ambiente virtual?

Um ambiente virtual é um espaço isolado dentro do sistema operacional, no qual podemos instalar programas, ferramentas e suas dependências sem interferir diretamente nas demais instalações do computador.

Em bioinformática, diferentes análises podem exigir ferramentas com versões específicas de programas ou bibliotecas e por isso a criação de ambientes virtuais tornam-se relevantes.

Ao instalar essas ferramentas em um ambiente virtual, podemos manter suas dependências organizadas e reduzir conflitos entre versões.

### O que é o Conda?

O Conda é um sistema de gerenciamento de pacotes e ambientes virtuais.

Ele permite:

- Criar ambientes independentes;
- Instalar ferramentas de bioinformática;
- Gerenciar dependências;
- Organizar versões de programas;
- Ativar e desativar ambientes conforme a necessidade.

Neste tutorial, utilizaremos o Conda para criar um ambiente específico para as análises de RNA-seq.

## 3. Criando e ativando um ambiente Conda

O primeiro passo é criar o ambiente virtual. Normalmente executamos no terminal (não execute neste momento):

```bash
conda create -n rnaseq
```

O comando significa:

| Elemento | Significado |
|---|---|
| `conda create` | Cria um novo ambiente |
| `-n` | Define o nome do ambiente |
| `rnaseq` | Nome escolhido para o ambiente |

Após a criação, o próximo passo é ativar esse ambiente criado (não execute neste momento):

```bash
conda activate rnaseq
```

Quando o ambiente estiver ativado, o nome `rnaseq` aparecerá no início da linha do terminal.

Exemplo:

```bash
(rnaseq) daisy@Daisy:~$
```

Isso indica que o ambiente está ativo.

## 4. Instalação de ferramentas pelo Conda

Após ativar o ambiente virtual, podemos instalar as ferramentas necessárias para a análise de RNA-seq.

Por exemplo, para instalar o FastQC, utilizamos (não execute neste momento):

```bash
conda install bioconda::fastqc
```

O comando informa ao Conda que desejamos instalar o pacote `fastqc`, disponibilizado no canal Bioconda.

### Como encontrar os comandos de instalação?

Para descobrir como instalar uma ferramenta:

1. Acesse o Google;
2. Pesquise por `conda + nome da ferramenta`;
3. Localize a página oficial do pacote no Anaconda.org;
4. Verifique o comando de instalação disponibilizado;
5. Copie o comando e execute-o no terminal com o ambiente ativado.

Por exemplo (não execute neste momento):

```bash
conda install bioconda::fastqc
```

Após a instalação, o FastQC estará disponível para utilização dentro do ambiente `rnaseq`.

## 5. Instalação de todas as ferramentas utilizando um script Bash (execute neste momento)

Em vez de instalar cada ferramenta individualmente, podemos automatizar todo o processo utilizando um script Bash.
Um script Bash é um arquivo de texto que contém uma sequência de comandos que serão executados pelo interpretador Bash do Linux.
Neste tutorial, utilizaremos um arquivo chamado:

`rnaseq_tools.sh`

O script será responsável por:

1. Configurar o Conda para permitir a ativação de ambientes no Bash;
2. Criar o ambiente virtual `rnaseq`;
3. Instalar as ferramentas necessárias;
4. Ativar o ambiente;
5. Verificar as instalações.

### 5.1. Criando o arquivo Bash

No terminal do WSL Ubuntu, execute:

```bash
nano rnaseq_tools.sh
```

O comando `nano` abre um editor de texto no próprio terminal. Caso o arquivo ainda não exista, ele será criado.

Copie o conteúdo abaixo e cole dentro do editor Nano.

```bash
#!/bin/bash

# ============================================================
# Configuração do ambiente para análise de RNA-Seq
# Ambiente: rnaseq
# ============================================================

# Interrompe o script em caso de erro
set -euo pipefail

# ------------------------------------------------------------
# 1. Nome do ambiente
# ------------------------------------------------------------

ENV_NAME="rnaseq"

# ------------------------------------------------------------
# 2. Configurar o Conda para permitir a ativação no Bash
# ------------------------------------------------------------

source "$(conda info --base)/etc/profile.d/conda.sh"

# ------------------------------------------------------------
# 3. Criar o ambiente Conda e instalar as ferramentas
# ------------------------------------------------------------

echo "Criando o ambiente ${ENV_NAME}..."

conda create -y -n "${ENV_NAME}" \
    -c conda-forge \
    -c bioconda \
    --strict-channel-priority \
    fastqc \
    multiqc \
    trimmomatic \
    hisat2 \
    samtools \
    subread

# ------------------------------------------------------------
# 4. Ativar o ambiente
# ------------------------------------------------------------

echo "Ativando o ambiente ${ENV_NAME}..."

conda activate "${ENV_NAME}"

# ------------------------------------------------------------
# 5. Verificar a instalação das ferramentas
# ------------------------------------------------------------

echo "Verificando as ferramentas instaladas..."

echo "=========================================="
echo " VERIFICANDO FERRAMENTAS DO RNA-SEQ"
echo "=========================================="
echo

echo "[1] FastQC"
fastqc --version
echo

echo "[2] MultiQC"
multiqc --version
echo

echo "[3] Trimmomatic"
trimmomatic -version
echo

echo "[4] HISAT2"
hisat2 --version 2>&1 | head -n 1
echo

echo "[5] SAMtools"
samtools --version | head -n 1
echo

echo "[6] featureCounts"
featureCounts -v
echo

# ------------------------------------------------------------
# 6. Finalização
# ------------------------------------------------------------

echo "============================================================"
echo "Ambiente ${ENV_NAME} criado e ferramentas instaladas!"
echo "============================================================"
```

Após inserir todo o conteúdo:

1. Pressione `Ctrl + X` para sair;
2. O Nano perguntará se deseja salvar as modificações;
3. Digite `Y` (Yes);
4. Pressione `Enter` para confirmar o nome do arquivo.

O arquivo `rnaseq_tools.sh` será salvo no diretório atual.

Utilize o comando `ls` para visualizar os arquivos do diretório:

```bash
ls
```

Exemplo:

```text
rnaseq_tools.sh
```

Para visualizar o conteúdo do script, utilize:

```bash
cat rnaseq_tools.sh
```

O comando `cat` permite exibir o conteúdo de arquivos de texto diretamente no terminal.

## 6. Executando o script de instalação

Após salvar o arquivo, podemos executar todos os comandos de instalação de uma única vez.

Utilize:

```bash
bash rnaseq_tools.sh
```

O Bash executará os comandos presentes no arquivo, seguindo a ordem definida no script.

**Atenção:** A instalação pode demorar alguns minutos, dependendo da velocidade da conexão com a internet e das dependências necessárias.

O parâmetro `-y` utilizado no comando `conda create` permite aceitar automaticamente as confirmações solicitadas pelo Conda durante a criação do ambiente.

## 7. Verificando e utilizando o ambiente

Após a execução do script, podemos ativar manualmente o ambiente para utilizar as ferramentas.

```bash
conda activate rnaseq
```

Para listar os pacotes instalados no ambiente:

```bash
conda list
```

Para desativar o ambiente:

```bash
conda deactivate
```

