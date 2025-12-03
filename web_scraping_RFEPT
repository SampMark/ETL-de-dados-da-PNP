# 📂 Web Scraping e Consolidação de Dados para Mapeamento da Rede Federal de Educação Profissional e Tecnológica (RFEPT)

Este projeto consiste em um pipeline de Engenharia de Dados em Python (Jupyter Notebook) para extrair, limpar, enriquecer e consolidar todas as unidades da **Rede Federal de Educação Profissional, Científica e Tecnológica (RFEPT)**, inclusive endereços (dados geográficos) das unidades.

O script combina dados "vivos" extraídos do [Portal da Rede Federal no MEC](https://www.gov.br/mec/pt-br/assuntos/ept/rede-federal) com o único _dataset_ disponível da [REDE FEDERAL DE EPCT – Unidades da Rede Federal de EPCT - `CSV`](https://dados.gov.br/dados/conjuntos-dados/rede-federal-de-epct--unidades-da-rede-federal-de-epct) no Portal de Dados Abertos do MEC.
A iniciativa foi gerar um dataset atualizado e padronizado pronto para análise e visualização em ferramentas de BI (como Looker Studio ou Power BI).

## 🎯 Objetivos

1.  **Extração (Scraping):** Coletar dados atualizados de contato, endereço e gestão de todos os Institutos Federais, CEFETs e Colégios vinculados, varrendo as páginas estaduais do portal `gov.br/mec`.
2.  **Padronização:** Normalizar endereços, telefones, e-mails e nomes de campi para garantir consistência.
3.  **Enriquecimento:** Criar Identificadores Únicos (UIDs), classificar tipos de unidade (Campus, Reitoria, Polo) e cruzar com o único _dataset_ disponível, para recuperar colunas como o ano de criação.
4.  **Consolidação:** Gerar uma "fonte da verdade" unificada no Google Sheets e `CSV`.

## 🛠️ Stack Tecnológico

* **Linguagem:** Python 3
* **Coleta de Dados:** `requests` (com retentativas/backoff), `BeautifulSoup4`
* **Processamento de Dados:** `pandas`, `numpy`
* **Manipulação de Texto:** `regex (re)`, `unidecode`
* **Integração:** `gspread` (Google Sheets API), `google-auth`

## 🚀 Fluxo de Execução (Pipeline)

O notebook está dividido nas seguintes seções:

### 1. Configuração e Constantes
Definição de cabeçalhos HTTP, mapeamento de URLs por estado (UF), e dicionários de correção de nomes (`MAPA_CORRECAO`) e siglas (`MAPA_SIGLAS_IFS`) para garantir que "Instituto Federal de São Paulo" seja identificado como "IFSP".

### 2. Motor de Scraping (`parse_state_page`)
Implementa uma lógica robusta que:
* Itera sobre as 27 Unidades Federativas.
* Identifica blocos de HTML correspondentes a Reitorias e Campi.
* Extrai metadados complexos (Reitor, Diretor, Endereço completo) usando Expressões Regulares.
* Utiliza um sistema de *Retry* para lidar com instabilidades no servidor do governo.

### 3. Limpeza e Normalização
* **Endereços:** Criação da coluna `Endereco_Padronizado` otimizada para geocodificação (separando Logradouro, Município, UF e CEP).
* **Contatos:** Limpeza de ruídos em campos de telefone e e-mail.
* **Nomes:** Padronização de maiúsculas/minúsculas e conectivos.

### 4. Criação de Identificador Único (UID)
Geração de uma chave primária sintética (slug) para cada unidade, permitindo o cruzamento de dados e a remoção de duplicatas.
* *Exemplo:* `ifsuldeminas_campus_pouso_alegre`

### 5. Enriquecimento e Merge (Dataset MEC)
Integração com uma planilha legada (`Mapa_RFEPCT_MEC_2022`) para:
* Importar o **Ano de Criação** da unidade.
* Adicionar unidades que existem no mapa oficial mas falharam no scraping (fallback).
* Corrigir divergências de grafia de municípios usando a base oficial como referência.

### 6. Classificação (`Tipo_Unidade`)
Algoritmo de categorização que define a tipologia da unidade com base no nome:
* *Categorias:* Reitoria, Campus, Campus Avançado, Polo de Inovação, Uned, Centro de Referência, etc.

### 7. Exportação
O dataset final processado é exportado automaticamente para uma planilha no Google Sheets (`Web-Scraping-and-merge-RFEPT-map`), substituindo os dados antigos para manter o dashboard atualizado.

## 📊 Estrutura dos Dados (Output)

O DataFrame final contém as seguintes colunas principais:

| Coluna | Descrição |
| :--- | :--- |
| **UID** | Identificador único gerado (Chave Primária). |
| **Sigla_IF** | Sigla da instituição (ex: IFMG, CEFET-RJ). |
| **Nome_IF** | Nome completo da instituição. |
| **Campus_IF** | Nome da unidade ou campus. |
| **Tipo_Unidade** | Classificação (ex: Reitoria, Campus). |
| **UF / Municipio** | Localização geográfica. |
| **Endereco_Padronizado** | Endereço limpo para APIs de mapas. |
| **Ano_Criacao** | Ano de fundação da unidade (via Merge). |
| **Dirigente** | Nome do Reitor(a) ou Diretor(a) Geral. |
| **Fonte** | URL de origem do dado. |

## ⚠️ Tratamento de Erros e Logs
O script possui um sistema de `logging` detalhado que informa:
* Status da conexão com cada página de UF.
* Número de unidades encontradas por estado.
* Alertas de duplicatas removidas ou dados inconsistentes.

---
*Desenvolvido para análise de dados educacionais e gestão da Rede Federal.*
