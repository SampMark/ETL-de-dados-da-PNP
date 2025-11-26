# ETL: Unidades da Rede Federal de EPCT

> Pipeline leve para extrair, tratar e carregar a lista de instituições e campi da Rede Federal (EPCT) a partir do Portal de Dados Abertos do MEC (serviço OData “PDA_SETEC”). Saída pronta para análise e para publicação no Google Sheets.

## Objetivo

* **Extrair** CSV oficial da Rede Federal (OData Olinda MEC).
* **Tratar** textos (mojibake, acentuação, Unicode), padronizar colunas e enriquecer com **Endereço** e **Regiões**.
* **Carregar** o DataFrame em uma aba do Google Sheets.

## Download
* **[DOWNLOAD REDE FEDERAL DE EPCT – Unidades da Rede Federal de EPCT](https://raw.githubusercontent.com/SampMark/files/refs/heads/main/REDE%20FEDERAL%20DE%20EPCT%20%E2%80%93%20Unidades%20da%20Rede%20Federal%20de%20EPCT%20-%20Mapa_RFEPCT_MEC_2022.csv)**: arquivo `CSV` tratado com a listagem das instituições da Rede Federal de EPCT, data de autorização de funcionamento - 2022
  
| # | Column       | Non-Null Count | Dtype  |
|---|---------------|----------------|--------|
| 0 | UID           | 644 non-null   | object |
| 1 | Ano_Criacao   | 644 non-null   | int64  |
| 2 | Sigla_IF      | 644 non-null   | object |
| 3 | Nome_IF       | 644 non-null   | object |
| 4 | Campus_IF     | 644 non-null   | object |
| 5 | Regioes       | 644 non-null   | object |
| 6 | UF            | 644 non-null   | object |
| 7 | Município     | 644 non-null   | object |
| 8 | Cidade_UF     | 644 non-null   | object |

**dtypes:** int64(1), object(8)  
**memory usage:** 45.4+ KB

## Stack

* Python 3.10+
* `pandas`, `requests`
* `gspread`, `google-auth` (exportação p/ Sheets)
* Ambiente: local ou Google Colab

## Fonte dos dados

* Conjunto: Rede Federal de Educação Profissional, Científica e Tecnológica (dados.gov.br)
* Endpoint padrão (OData Olinda): `.../servico/PDA_SETEC/versao/v1/odata/RedeFederal_Lista_Instituicoes_RFEPCT_2022?$format=text/csv`

## Principais funções

### `load_mec_unidades(url: str = URL, save_local: bool = True, show_preview: bool = True) -> pd.DataFrame`

Extrai e prepara o DataFrame.

* **Download** com `requests` (timeout, `raise_for_status`).
* **Leitura robusta de CSV**: autodetecção de separador (`sep=None`, `engine='python'`).
* **Codificação**: ordem de tentativas `utf-8-sig` → `utf-8` → `cp1252` → `latin-1` com detecção de **BOM**.
* **Higienização de texto**:

  * Correção de **mojibake** (recodificação `latin1→utf-8` quando detectado).
  * Normalização Unicode **NFC**.
  * Remoção do caractere inválido `ÿ` quando presente.
* **Enriquecimento e padronização**:

  * `remove_campus_suffixes`: remove sufixos como “`- IFXX`/`- CPII`” do final do nome do campus.
  * `create_address_column`: cria **`Endereço`** combinando `Município/UF`.
  * `add_regioes_column`: mapeia **UF → Região** (Norte, Nordeste, Centro-Oeste, Sudeste, Sul).
  * `rename` de colunas para nomes finais.

**Colunas finais (ordem):**
`Ano_Criação`, `Sigla_IF`, `Nome_IF`, `Campus_IF`, `Regiões`, `Município`, `UF`, `Endereço`.

## Uso rápido

### Google Colab

```python
# Autentique e rode tudo
from github_etl_unidades_da_rede_federal_de_epct import load_mec_unidades

df = load_mec_unidades(show_preview=True)
```

### Local (sem Sheets)

```bash
pip install pandas requests
python -c "from github_etl_unidades_da_rede_federal_de_epct import load_mec_unidades; print(load_mec_unidades(show_preview=False).head())"
```

## Exportação para Google Sheets

O script inclui uma etapa opcional para publicar o DataFrame em uma planilha.

1. **Configurar** no código:

   * `ID_PLANILHA = "<id_da_planilha>"`
   * `NOME_ABA    = "Mapa_RFEPCT"` (criada se não existir)
2. **Autenticar**:

   * Em **Colab**: `from google.colab import auth; auth.authenticate_user()` (interativo).
   * Em ambiente local, prefira **Service Account** (não incluso por padrão).
3. **Executar** a célula de exportação:

   * Abre a planilha por chave
   * Cria/limpa a aba
   * Converte o DataFrame → lista de listas e faz `worksheet.update('A1', dados)`

> Permissões: a conta autenticada precisa ser **Editora** na planilha.

## Instalação

```bash
pip install pandas requests gspread google-auth
```

Para Colab, as dependências de Google geralmente já estão presentes.

## Validações e logs

* Dimensões do DataFrame e `df.info()` opcionais (`show_preview=True`).
* Mensagens claras em falhas de rede e de API do Sheets.

## Decisões técnicas

* **Leitura tolerante** a CSVs com `;` e codificações mistas do portal.
* **Unicode NFC** para consistência de acentos em todo o DataFrame.
* **Mapeamento estático UF → Região** para consultas e agregações rápidas.

## Limitações conhecidas

* URL de 2022 como padrão; para anos adicionais, alterar `URL`.
* Exportação via conta interativa do Colab; uso em produção recomenda `Service Account` e variáveis de ambiente para credenciais e IDs.

## Roadmap curto

* Parametrizar ano e endpoint (`argparse`).
* Testes unitários para utilitários de texto e mapeamentos.
* Validação de esquema (colunas esperadas) e `pyproject.toml`.
* Modo **dry-run** para exportação.

## Licença MIT
