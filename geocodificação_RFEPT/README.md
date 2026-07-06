# 🗺️🌍 Geocodificador Avançado da Rede Federal (RFEPT)

## 📌 Visão Geral
Este projeto implementa um pipeline de Engenharia de Dados (ETL) focado em inteligência geográfica e planejamento educacional. 
O script processa endereços das unidades de ensino obtidos previamente via *web scraping*, convertendo-os em coordenadas espaciais (Latitude e Longitude), tendo por objetivo o cálculo da abrangência demográfica dos campi.
O código foi desenvolvido para lidar com dados raspados da web, que frequentemente contêm ruídos, formatações inconsistentes e endereços incompletos, aplicando técnicas avançadas de limpeza, fallback de consultas e cache inteligente.

Os dados processados no notebook serão utilizados para alimentar painéis de *Business Intelligence* (como o Looker Studio), subsidiando relatórios de gestão, autoavaliação institucional e diagnósticos de expansão da rede.
Este repositório contém um script Python automatizado para a **geocodificação em massa** das unidades da Rede Federal de Educação Profissional, Científica e Tecnológica (RFEPT). 

## 🚀 Funcionalidades Principais

- **Limpeza e Padronização de Dados:** Normaliza endereços, corrige artefatos comuns de *web scraping* (ex: caracteres especiais, CEPs quebrados) e remove ruídos.
- **Estratégia de Consultas em Camadas (Fallback):** Tenta geocodificar usando 5 níveis de precisão, do mais específico ao mais genérico, garantindo a maior taxa de sucesso possível.
- **Sistema de Cache Inteligente:** Salva os resultados das consultas em um arquivo JSON local (`cache_geocodificacao_rfept.json`). Isso evita reprocessamento, economiza tempo e previne bloqueios por parte da API.
- **Respeito ao Rate Limiting:** Utiliza `RateLimiter` do `geopy` para garantir que as requisições ao Nominatim (OpenStreetMap) respeitem as políticas de uso (1 requisição por segundo).
- **Classificação de Confiança:** Categoriza cada resultado geocodificado (Alta, Média, Baixa, Sem Resultado) e sinaliza automaticamente quais registros precisam de revisão manual.
- **Integração com Google Sheets:** Exporta os dados processados diretamente para uma planilha do Google Sheets para fácil compartilhamento e revisão colaborativa.

---

## ⚙️ Como Funciona (O Pipeline de Geocodificação)

O script opera em um pipeline estruturado em 6 etapas:

### 1. Ingestão e Limpeza
Os dados brutos são lidos do CSV público da RFEPT. Funções de *Regex* e normalização limpam os campos de `Endereco_Padronizado`, `CEP`, `Municipio` e `UF`.

### 2. Geração de Consultas em Camadas
Para cada unidade, o script monta uma lista de tentativas de consulta à API do Nominatim, ordenadas por precisão esperada:
1. **Endereço completo** (Alta precisão)
2. **Endereço + CEP**
3. **CEP + Município + UF**
4. **Nome da Unidade + Município + UF** (Média precisão)
5. **Apenas Município + UF** (Baixa precisão - centroide da cidade)

### 3. Geocodificação com Cache
O script consulta o cache local. Se o resultado não estiver em cache, ele itera pelas camadas de consulta até encontrar uma coordenada válida. O cache é atualizado progressivamente a cada 10 registros para evitar perda de dados em caso de interrupção.

### 4. Classificação de Qualidade
Cada coordenada retornada recebe um *score* de confiança (`geocode_confianca`):
- `alta`: Encontrado via endereço completo ou CEP.
- `media`: Encontrado via CEP+Município ou Nome da Instituição.
- `baixa`: Encontrado apenas pelo Município (retorna o centro da cidade, não o endereço exato).
- `sem_resultado`: Nenhuma camada teve sucesso.

### 5. Exportação Local
Gera um CSV enriquecido (`rfept_unidades_geocodificadas.csv`) contendo todas as colunas originais somadas às coordenadas, status da API, endereço retornado pelo OSM e flags de revisão manual.

### 6. Sincronização com Google Sheets
Autentica via Google Colab e sobe o DataFrame final para uma planilha específica, facilitando a visualização por equipes que não utilizam Python.

---

## 📦 Pré-requisitos e Dependências

Para executar o script localmente (fora do Google Colab), você precisará das seguintes bibliotecas:

```bash
pip install pandas geopy requests gspread gspread_dataframe google-auth google-auth-oauthlib
```

---

### 📊 Dataset Gerado

O resultado final do processamento, contendo as coordenadas (latitude/longitude), status da API, nível de precisão, confiança e flags de revisão manual para todas as unidades da Rede Federal, está disponível para consulta e download no repositório:

🔗 **[rfept_unidades_geocodificadas.csv](https://github.com/SampMark/files/blob/main/rfept_unidades_geocodificadas.csv)**
