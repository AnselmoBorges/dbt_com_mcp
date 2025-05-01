# 🧪 dbt Core + MCP (100 % local)

Playground minimalista para experimentar **dbt Core** e o **MCP Server** da dbt Labs rodando totalmente local, sem dbt Cloud, usando **DuckDB** e arquivos CSV.

### 🚀 Roteiro "tudo-em-um" para levantar **dbt Core + MCP** localmente e plugar no Cursor
*(copie este passo-a-passo para o chat do Cursor e vá executando linha a linha)*

---

## 1. Pré-requisitos

| O que instalar | Como (macOS) |
|----------------|--------------|
| **Python 3.12+** | `brew install python@3.12` <br>ou `brew install pyenv && pyenv install 3.12.3` |
| **Homebrew + curl / git** | já vêm na maioria dos macOS; se não, instale pelo site |

---

## 2. Criar/ativar um virtual-env 3.12 global

```bash
# crie um venv único no $HOME (fica fácil de referenciar)
python3.12 -m venv ~/.venv
source ~/.venv/bin/activate        # ative sempre que for usar o projeto
python -m pip install --upgrade pip
```

---

## 3. Instalar as dependências

```bash
pip install dbt-duckdb             # dbt-core incluído
pip install dbt-mcp                # MCP Server (exige 3.12, já satisfaz)
```

> ✅ Cheque:
> ```bash
> which dbt   # → ~/.venv/bin/dbt
> which mcp   # → ~/.venv/bin/mcp
> ```

---

## 4. Criar um projeto-exemplo e carregar CSVs

```bash
mkdir -p ~/Documentos/repositorios/dbt_com_mcp
cd       ~/Documentos/repositorios/dbt_com_mcp

dbt init mcp_demo --adapter duckdb
mkdir -p mcp_demo/data
cp /caminho/para/seus_csvs/*.csv mcp_demo/data/
dbt --project-dir mcp_demo seed
```

---

## 5. Variáveis de ambiente (use no mesmo terminal que rodará o MCP)

```bash
# caminhos básicos
export DBT_PROJECT_DIR=$PWD/mcp_demo
export DBT_PATH=$HOME/.venv/bin/dbt

# desliga integrações que exigem dbt Cloud
export DISABLE_DISCOVERY=true
export DISABLE_SEMANTIC_LAYER=true

# (opcional) porta alternativa se 3333 já estiver ocupada
export MCP_PORT=3333
```

---

## 6. Descobrir o `main.py` do pacote (one-liner)

```bash
MAIN_PY=$(python -c "import inspect, pathlib, dbt_mcp, sys; print(pathlib.Path(inspect.getfile(dbt_mcp)).with_name('main.py'))")
echo $MAIN_PY   # conferência: .../site-packages/dbt_mcp/main.py
```

---

## 7. Subir o MCP em modo **DEBUG** (mostra tudo)

```bash
MCP_LOG_LEVEL=debug mcp run "$MAIN_PY"
```

Você deve ver linhas de:

```
✓ tool registered: dbt.build
✓ tool registered: dbt.list
Listening on http://127.0.0.1:$MCP_PORT
```

> Se travar >30 s, abra outro terminal:  
> `lsof -i :$MCP_PORT` (confere porta) • `curl -s http://localhost:$MCP_PORT/health`

---

## 8. Testes rápidos

```bash
curl -s http://localhost:$MCP_PORT/health          # {"status":"ok"}

curl -s -X POST http://localhost:$MCP_PORT/invoke \
     -H "Content-Type: application/json" \
     -d '{"tool":"dbt.list","args":["--resource-type","seed"]}'
```

---

## 9. Configurar o Cursor

**Settings → AI Tools → MCP Servers** → "Add":

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

Salve; o painel lateral do Cursor mostrará tools como `dbt.build`, `dbt.list` etc.

---

## 10. Troubleshooting rápido

| Sintoma | Solução |
|---------|---------|
| `No matching distribution for dbt-mcp` | garante que `python --version` é 3.12+ e recrie o venv. |
| Reclama de `DBT_HOST / DBT_TOKEN` | exporte `DISABLE_DISCOVERY=true` e `DISABLE_SEMANTIC_LAYER=true`. |
| Porta ocupada | `export MCP_PORT=3334` e rode de novo. |
| Health-check falha | rode `MCP_LOG_LEVEL=debug …` e veja onde parou; poste as últimas linhas do log. |

---

Pronto! Com esses passos o Cursor conseguirá comandar dbt via MCP totalmente local. Se precisar incluir mais CSVs ou mudar adaptador (ex. Postgres), é só ajustar `pip install dbt-postgres` + `dbt init ...`. Boa diversão! 🎉

## 📜 Licença

MIT — veja `LICENSE`.
