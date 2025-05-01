# dbt + MCP (local) Playground

Pequeno laboratório para brincar com **dbt Core** e o **MCP Server** da dbt Labs,
rodando 100 % local sobre DuckDB.

---

## 🚀 Prerequisites

| Ferramenta | Versão mínima | Instalação rápida |
|------------|--------------|-------------------|
| Python | **3.12** | `brew install python@3.12` <br>ou `pyenv install 3.12.3` |
| Bash / curl | n/a | já vêm no macOS / WSL / Linux |

> O pacote `dbt-mcp` requer `>=3.12`; versões 3.10/3.11 não instalam.

---

## ⚙️ Setup

```bash
git clone https://github.com/<seu_org>/dbt_com_mcp.git
cd dbt_com_mcp

# 1. Ambiente virtual
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip

# 2. Dependências
pip install dbt-duckdb dbt-mcp   # inclui dbt-core

# 3. Projeto-exemplo
dbt init mcp_demo --adapter duckdb
mkdir -p mcp_demo/data && cp caminho/para/*.csv mcp_demo/data/
dbt --project-dir mcp_demo seed

# 4. Variáveis de ambiente
export DBT_PROJECT_DIR=$PWD/mcp_demo
export DBT_PATH=$PWD/.venv/bin/dbt
