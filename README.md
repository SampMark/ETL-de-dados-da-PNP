# **Pipeline de Extração, Tratamento e Carregamento para o BigQuery de Dados da Plataforma Nilo Peçanha (PNP)**
<img src="https://novopnp.mec.gov.br/assets/logo-nilo-vertical-white-39a156b6.svg" alt="Logo da Plataforma Nilo Peçanha" width="150"/>

## 📑 Visão Geral

Este projeto consiste em um pipeline de dados automatizado, desenvolvido em um notebook Google Colab, que permite a extração, tratamento e carregamento de dados da **Plataforma Nilo Peçanha (PNP)**. 
O objetivo é transformar e analisar grandes volumes de dados, disponibilizados anualmente em formato `.csv.gz`, em um DataFrame limpo, personalizado, padronizado e pronto para análise. O armazenando do Data Frame Pandas final serão tabelas do **Google BigQuery**, que podem ser conectadas a quaisquer ferramentas de _Business Intelligence_ (BI), como o **Google Looker Studio**, entre outras.

O pipeline é configurável através de uma interface interativa, permitindo que o usuário selecione a tabela de dados conforme a dimensão da PNP, o período de análise e as instituições de ensino desejadas.

## ✨ Funcionalidades Principais

* **Download Automatizado:** Baixa os arquivos de microdados da PNP diretamente da fonte oficial para uma pasta no Google Drive do usuário.
* **Interface Interativa:** Permite a configuração de parâmetros (tabela, ano inicial/final, instituição) de forma amigável, sem necessidade de alterar o código.
* **Análise de Cabeçalhos:** Compara os cabeçalhos dos arquivos CSV de diferentes anos, identificando colunas comuns e inconsistências.
* **Processamento Eficiente de Big Data:** Utiliza a leitura de dados em _chunks_ (pedaços), permitindo processar arquivos grandes sem sobrecarregar a memória.
* **Filtragem e Unificação:** Filtra os dados para uma ou mais instituições específicas e unifica os arquivos de múltiplos anos em um único DataFrame.
* **Limpeza e Padronização:** Aplica um conjunto robusto de regras para limpar, padronizar e enriquecer os dados, com destaque para a normalização de nomes de cursos.
* **Exportação para BigQuery:** Carrega o DataFrame final e tratado diretamente em uma tabela no Google BigQuery, deixando os dados prontos para consultas e visualizações.

## 🚀 Fluxo de Execução do Pipeline

O notebook é organizado em etapas sequenciais. A seguir, uma descrição de cada uma:

### **Etapa 1: Instalação e Autenticação**
* **O que faz:** Instala as bibliotecas Python necessárias (`pandas-gbq`, `gspread`, etc.) e realiza a autenticação do usuário para permitir o acesso ao Google Drive e aos serviços Google Cloud.
* **Ação do Usuário:** Executar a célula e autorizar o acesso à sua Conta Google quando solicitado.

### **Etapa 2: Definição das Funções Principais**
* **O que faz:** Carrega todas as funções que encapsulam a lógica do pipeline (download, descompressão, análise, processamento, etc.).
* **Ação do Usuário:** Apenas executar a célula. Nenhuma modificação é necessária.

### **Etapa 3: Configuração do Processo**
* **O que faz:** Exibe uma interface interativa para o usuário definir os parâmetros da extração.
* **Ação do Usuário:**
    * **Tabela de Dados:** Escolher a base de dados de interesse (ex: `matriculas`, `servidores`).
    * **Ano Inicial e Final:** Definir o período da análise.
    * **Nome da Instituição:** Inserir o nome ou sigla da(s) instituição(ões) a serem filtradas, separados por vírgula (ex: `IFRN, Instituto Federal do Rio Grande do Norte`).

### **Etapa 4: Download e Análise de Cabeçalhos**
* **O que faz:** Com base nos parâmetros da Etapa 3, o script baixa os arquivos `.gz`, os descompacta e lê o cabeçalho de cada `.csv`. Ao final, exibe uma tabela comparativa e sugere uma lista de colunas que são comuns a todos os arquivos do período.
* **Ação do Usuário:** Analisar a tabela de cabeçalhos e **copiar a lista de colunas sugerida (`final_columns_to_keep`)** para usar na próxima etapa.

### **Etapa 5: Processamento e Criação do DataFrame**
* **O que faz:** Utiliza a lista de colunas definida pelo usuário para ler, filtrar (pela instituição selecionada) e unificar todos os arquivos CSV em um único DataFrame Pandas (`df_final`).
* **Ação do Usuário:** **Colar a lista de colunas** copiada da Etapa 4 no local indicado e executar a célula.

### **Etapa 6: Análise Exploratória e Limpeza dos Dados**
* **O que faz:** Aplica uma série de transformações no DataFrame `df_final` para criar uma versão limpa e padronizada, chamada `df_processed`. As principais ações são:
    1.  **Criação da coluna `Cursos`**: Adiciona prefixos como "Licenciatura em", "Tecnologia em" e "FIC" aos nomes dos cursos.
    2.  **Padronização de Nomes de Cursos**: Executa uma limpeza profunda na coluna `Cursos`, removendo prefixos, corrigindo erros e aplicando regras de normalização.
    3.  **Conversão para Booleano**: Trata a coluna `Matrícula Atendida` para que contenha apenas valores `True` ou `False`.
    4.  **Ajustes Categóricos**: Padroniza valores em colunas como `Forma de ingresso`.
    5.  **Renomeação de Colunas**: Renomeia todas as colunas para o padrão `snake_case`, ideal para bancos de dados.
    6.  **Conversão de Tipos**: Ajusta os tipos de dados de cada coluna (inteiro, data, string, etc.).
* **Ação do Usuário:** Executar as células em sequência para aplicar todo o tratamento.

### **Etapa 7: Exportação para o BigQuery**
* **O que faz:** Realiza a última etapa do pipeline, enviando o DataFrame final (`df_processed`) para uma tabela no Google BigQuery.
* **Ação do Usuário:**
    1.  Definir as variáveis `project_id` e `dataset_id` com as informações do seu projeto no Google Cloud.
    2.  Executar a célula para iniciar o upload.

## ⚙️ Como Usar

1.  **Abra o Notebook:** Inicie o arquivo `.ipynb` no ambiente Google Colab.
2.  **Execute a Etapa 1:** Instale as dependências e autentique sua conta Google para acessar o Drive.
3.  **Execute a Etapa 2:** Carregue as funções do pipeline.
4.  **Configure na Etapa 3:** Preencha os campos da interface interativa com a tabela, período e instituição de seu interesse.
5.  **Execute a Etapa 4:** Aguarde o download e a análise dos cabeçalhos. Ao final, copie a lista de colunas (`final_columns_to_keep`).
6.  **Cole na Etapa 5:** Cole a lista de colunas no local indicado e execute a célula para criar o DataFrame bruto (`df_final`).
7.  **Execute a Etapa 6:** Rode todas as células desta etapa para realizar a limpeza e o processamento dos dados.
8.  **Configure e Execute a Etapa 7:** Insira o ID do seu projeto e do seu dataset do BigQuery e execute a célula para exportar a tabela final.

## 🛠 Pré-requisitos

* Uma Conta Google com acesso ao Google Drive.
* Um projeto no **Google Cloud Platform** com a **API do BigQuery** ativada.
* Permissões de Editor (`BigQuery Data Editor` e `BigQuery Job User`) no projeto do Google Cloud.
