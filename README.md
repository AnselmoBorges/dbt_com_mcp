# 🧪 dbt Core + MCP (100 % local)

Playground minimalista para experimentar **dbt Core** e o **MCP Server** da dbt Labs rodando totalmente local, sem dbt Cloud, usando **DuckDB** e arquivos CSV.

---

## 🚀 Pré-requisitos

| Ferramenta | Versão mínima | Instalação rápida |
|------------|---------------|-------------------|
| Python | **3.12** | `brew install python@3.12` (macOS) ou `pyenv install 3.12.3` |
| Bash + curl | n/a | Já vêm no macOS / Linux / WSL |

> O pacote `dbt-mcp` exige `requires-python >= 3.12`.

---

## ⚙️ Setup passo-a-passo

```bash
# Clonar (ou apenas trabalhar nesta pasta se já existir)
git clone https://github.com/<seu_org>/dbt_com_mcp.git
cd dbt_com_mcp

# 1 — virtualenv (usando Python 3.12)
python3.12 -m venv ~/.venv            # preferiu global
source ~/.venv/bin/activate
python -m pip install --upgrade pip

# 2 — dependências
pip install dbt-duckdb dbt-mcp        # dbt-core incluído

# 3 — projeto-exemplo (se ainda não existir)
dbt init mcp_demo --adapter duckdb
mkdir -p mcp_demo/data
cp caminho/para/*.csv mcp_demo/data/
dbt --project-dir mcp_demo seed
```

---

## 🛡️  Rodar o MCP em modo **100 % local**

```bash
# Paths obrigatórios
export DBT_PROJECT_DIR=$PWD/mcp_demo
export DBT_PATH=$HOME/.venv/bin/dbt   # ajuste se o venv estiver noutro lugar

# Desligar features que exigem dbt Cloud
export DISABLE_DISCOVERY=true
export DISABLE_SEMANTIC_LAYER=true

# Localizar o main.py do pacote e subir
MAIN_PY=$(python - <<'PY'
import inspect, pathlib, dbt_mcp
print(pathlib.Path(inspect.getfile(dbt_mcp)).with_name("main.py"))
PY)

mcp run "$MAIN_PY"
# → Listening on http://127.0.0.1:3333
```

### Health-check

```bash
curl http://localhost:3333/health          # {"status":"ok"}
```

### Exemplo de invocação de tool

```bash
curl -X POST http://localhost:3333/invoke      -H "Content-Type: application/json"      -d '{"tool":"dbt.list","args":["--resource-type","seed"]}'
```

---

## 🖥️ Integração com o **Cursor**

Abra **Settings → AI Tools → MCP Servers** e adicione:

```jsonc
{
  "dbt-mcp-local": {
    "command": "/Users/<user>/.venv/bin/mcp",
    "args": [
      "run",
      "/Users/<user>/.venv/lib/python3.12/site-packages/dbt_mcp/main.py"
    ],
    "env": {
      "DBT_PROJECT_DIR": "/Users/<user>/Documentos/repositorios/dbt_com_mcp/mcp_demo",
      "DBT_PATH": "/Users/<user>/.venv/bin/dbt",
      "DISABLE_DISCOVERY": "true",
      "DISABLE_SEMANTIC_LAYER": "true"
    }
  }
}
```

*Salve → o painel lateral do Cursor exibirá ferramentas como `dbt.build`, `dbt.list`, `discovery.get_all_models` (esta última estará desativada, pois desligamos Discovery).*

---

## 💡 Comandos úteis

| Descrição | Comando |
|-----------|---------|
| Build completo | `curl -X POST localhost:3333/invoke -d '{"tool":"dbt.build"}'` |
| Listar seeds | `curl -X POST localhost:3333/invoke -d '{"tool":"dbt.list","args":["--resource-type","seed"]}'` |
| Health check | `curl localhost:3333/health` |

---

## 🛠️ Troubleshooting

| Sintoma | Causa & Solução |
|---------|-----------------|
| **`No matching distribution for dbt-mcp`** | O Python do venv não é 3.12 → refaça com `python3.12 -m venv`. |
| **`Errors found in configuration: DBT_HOST ...`** | Faltou `DISABLE_DISCOVERY=true` e/ou `DISABLE_SEMANTIC_LAYER=true`. |
| Porta em uso | Já existe outro MCP. Pare com `Ctrl-C` ou `kill` o processo, ou defina `MCP_PORT=3334`. |

---

## 📜 Licença

MIT — veja `LICENSE`.
