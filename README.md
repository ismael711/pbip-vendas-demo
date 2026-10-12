# Vendas Dashboard — PBIP

> Projeto Power BI em formato **PBIP** (Power BI Project) conectado ao **Databricks SQL** (Spark) via SQL Warehouse. Permite versionamento completo do modelo semântico no Git, incluindo tabelas, colunas e medidas DAX.

---

## Sumário

- [Sobre o projeto](#sobre-o-projeto)
- [Pré-requisitos](#pré-requisitos)
- [Como clonar](#como-clonar)
- [Setup: configurar o Databricks token](#setup-configurar-o-databricks-token)
- [Como publicar no Power BI](#como-publicar-no-power-bi)
- [Estrutura do projeto](#estrutura-do-projeto)
- [Modelo semântico](#modelo-semântico)
- [Troubleshooting](#troubleshooting)

---

## Sobre o projeto

Este repositório contém um dashboard de vendas construído com o formato aberto **PBIP**, que armazena o modelo semântico e o layout do relatório como arquivos de texto puro (JSON). Isso viabiliza:

- **Versionamento via Git** — cada alteração no modelo ou no relatório fica rastreável com `git diff`.
- **Revisão de código (Pull Requests)** — times podem revisar mudanças DAX e de esquema antes do merge.
- **Integração com CI/CD** — possível validar e publicar automaticamente via pipelines.

A fonte de dados é o **Databricks SQL Warehouse** (`power_bi_demo.default.vendas`) acessado pelo conector nativo Spark do Power BI Desktop.

---

## Pré-requisitos

| Ferramenta | Versão mínima | Observação |
|---|---|---|
| [Power BI Desktop](https://powerbi.microsoft.com/pt-br/desktop/) | Junho 2024+ | Suporte a PBIP GA |
| [Git](https://git-scm.com/) | 2.x | Qualquer versão recente |
| Databricks workspace | — | Acesso ao SQL Warehouse configurado |
| Personal Access Token (PAT) | — | Gerado no Databricks |

---

## Como clonar

```bash
# HTTPS
git clone https://github.com/<seu-usuario>/pbip-vendas-demo.git

cd pbip-vendas-demo
```

> **Nota:** substitua `<seu-usuario>` pelo seu usuário ou organização do GitHub.

---

## Setup: configurar o Databricks token

O arquivo [`connections.json`](connections.json) referencia a variável de ambiente `${DATABRICKS_TOKEN}` para a senha da conexão. **Nunca commite o token diretamente no repositório.**

### 1. Gerar um Personal Access Token no Databricks

1. Acesse seu workspace Databricks.
2. Clique no seu perfil (canto superior direito) → **Settings** → **Developer**.
3. Em **Access tokens**, clique em **Generate new token**.
4. Dê um nome descritivo (ex.: `power-bi-desktop`) e defina a validade.
5. Copie o token gerado — ele não será exibido novamente.

### 2. Definir a variável de ambiente localmente

**macOS / Linux:**
```bash
export DATABRICKS_TOKEN="dapi_seu_token_aqui"
```

Para persistir entre sessões, adicione a linha acima ao seu `~/.zshrc` ou `~/.bashrc`.

**Windows (PowerShell):**
```powershell
$env:DATABRICKS_TOKEN = "dapi_seu_token_aqui"
```

Para persistir:
```powershell
[System.Environment]::SetEnvironmentVariable("DATABRICKS_TOKEN", "dapi_seu_token_aqui", "User")
```

### 3. Credenciais no Power BI Desktop

Na primeira abertura do arquivo, o Power BI Desktop solicitará as credenciais do Databricks:

- **Método de autenticação:** Nome de usuário / Senha
- **Nome de usuário:** `token`
- **Senha:** cole o valor do seu PAT Databricks

---

## Como publicar no Power BI

### Método 1 — Power BI Desktop (recomendado para desenvolvimento)

1. Abra o Power BI Desktop.
2. Vá em **Arquivo → Abrir relatório → Procurar relatórios**.
3. Selecione o arquivo `.pbip` na raiz do repositório.
4. Insira as credenciais do Databricks quando solicitado.
5. Clique em **Atualizar** para importar os dados.
6. Quando estiver pronto, clique em **Publicar** e selecione o workspace de destino no Power BI Service.

### Método 2 — Power BI Service via Git Integration (CI/CD)

1. No Power BI Service, abra o workspace de destino.
2. Vá em **Configurações do workspace → Git integration**.
3. Conecte ao repositório GitHub e selecione a branch `main`.
4. Mapeie a pasta raiz do repositório.
5. Clique em **Sync** — o serviço importará o modelo e o relatório automaticamente.

> A integração Git do Power BI Service requer licença **Fabric** ou **Premium Per User (PPU)**.

---

## Estrutura do projeto

```
pbip-vendas-demo/
│
├── definition/
│   └── model.tmd          # Modelo semântico (tabelas, colunas, medidas DAX)
│
├── connections.json        # Configuração da conexão com o Databricks SQL Warehouse
├── pbip.json               # Metadados do projeto PBIP (nome, cultura, compatibilidade)
├── .gitignore
└── README.md
```

| Arquivo | Descrição |
|---|---|
| `definition/model.tmd` | Coração do projeto: define a tabela `vendas`, suas colunas, partição M e as 4 medidas DAX |
| `connections.json` | Endereço do SQL Warehouse e método de autenticação (token via variável de ambiente) |
| `pbip.json` | Nome do projeto, nível de compatibilidade (`1550`) e cultura (`pt-BR`) |

---

## Modelo semântico

### Tabela: `vendas`

| Coluna | Tipo | Resumo padrão |
|---|---|---|
| `id_venda` | Int64 | Nenhum |
| `data_venda` | DateTime (Date) | Nenhum |
| `valor_venda` | Decimal | Soma |
| `regiao` | String | Nenhum |
| `categoria` | String | Nenhum |

### Medidas DAX

| Medida | Expressão | Descrição |
|---|---|---|
| **Total Vendas** | `SUM(vendas[valor_venda])` | Soma total do valor de vendas |
| **Qtd Vendas** | `COUNTROWS(vendas)` | Número de registros de venda |
| **Ticket Médio** | `DIVIDE([Total Vendas], [Qtd Vendas])` | Valor médio por venda (seguro contra divisão por zero) |
| **Vendas 30D** | `CALCULATE([Total Vendas], DATESINPERIOD(..., -30, DAY))` | Total de vendas nos últimos 30 dias |

---

## Troubleshooting

### ❌ Erro de credencial ao abrir o arquivo

**Sintoma:** `DataSource.Error: Datasource credentials are not set.`

**Solução:** No Power BI Desktop, vá em **Arquivo → Opções e configurações → Configurações de fonte de dados**, localize a conexão Databricks e clique em **Editar permissões** para reinserir as credenciais.

---

### ❌ Tabela `vendas` não encontrada

**Sintoma:** `Expression.Error: The key didn't match any rows in the table.`

**Solução:** Verifique se a tabela existe no schema correto no Databricks:

```sql
SHOW TABLES IN power_bi_demo.default;
```

Se estiver em um schema diferente de `default`, edite a expressão M na partição em [`definition/model.tmd`](definition/model.tmd):

```
Schema = Source{[Schema="<seu_schema>"]}[Data],
```

---

### ❌ Token expirado ou inválido

**Sintoma:** `HTTP 401 Unauthorized` ao atualizar dados.

**Solução:** Gere um novo PAT no Databricks, atualize a variável de ambiente `DATABRICKS_TOKEN` e reinsira as credenciais no Power BI Desktop conforme descrito na seção [Setup](#setup-configurar-o-databricks-token).

---

### ❌ Nível de compatibilidade não suportado

**Sintoma:** Aviso sobre `compatibilityLevel: 1550` ao abrir no Desktop.

**Solução:** Atualize o Power BI Desktop para a versão de **junho de 2024** ou mais recente. O nível 1550 requer Desktop ≥ 2.130.

---

### ❌ Medida `Vendas 30D` retorna BLANK

**Sintoma:** A medida não exibe valor em determinados contextos de filtro.

**Causa:** `DATESINPERIOD` usa `LASTDATE(vendas[data_venda])` como âncora. Se o contexto de filtro não contiver datas, a função retorna vazio.

**Solução:** Remova filtros de data do visual ou use `TODAY()` como âncora:

```dax
Vendas 30D =
CALCULATE(
    [Total Vendas],
    DATESINPERIOD(vendas[data_venda], TODAY(), -30, DAY)
)
```
