---
type: Note
_organized: true
---
# Guida: progetti con uv

Creare e gestire un progetto Python specificando la versione dell'interprete (ad esempio 3.12), aggiungere pacchetti e usare un virtual environment con nome personalizzato tramite `uv`.

## Installazione di uv

Su macOS con Homebrew:

```bash
brew install uv
```

## Creazione del progetto senza creare un package

Per inizializzare un progetto nella directory corrente:

```bash
uv init --name mio_progetto --no-package --python 3.12
```

Per crearne uno in una nuova cartella:

```bash
uv init mio_progetto --no-package --python 3.12
cd mio_progetto
```

Il comando genera:

- `pyproject.toml` (configurazione del progetto)
- `.python-version` (versione di Python richiesta)
- `main.py` (script d'esempio)
- `README.md`

Per creare solo il file `pyproject.toml` senza generare gli altri file di supporto, si può usare il flag `--bare`.

### Specificare la versione di Python

Il flag `--python` accetta vari formati:

```bash
uv init --python 3.12          # ultima patch 3.12.x
uv init --python 3.12.13       # Patch specifica
uv init --python >=3.11,<3.13  # intervallo di versioni
```

Se la versione richiesta non è installata sul sistema, `uv` la scarica in automatico da [python-build-standalone](https://github.com/astral-sh/python-build-standalone).

## Virtual environment personalizzato

Di default `uv` crea l'ambiente virtuale nella cartella `.venv`. Per usare un nome diverso, lo si passa direttamente a `uv venv`:

```bash
uv venv uni --python 3.12
```

Questo crea la cartella `uni/` con Python 3.12 (scaricandolo se necessario).

Per attivarlo:

```bash
source uni/bin/activate
```

Per fare in modo che i comandi `uv add`, `uv run` e `uv sync` utilizzino l'ambiente personalizzato anziché `.venv`, si imposta la variabile d'ambiente `UV_PROJECT_ENVIRONMENT`:

```bash
export UV_PROJECT_ENVIRONMENT=uni
```

Per rendere l'impostazione permanente, si può aggiungere questa riga al file di configurazione della shell (`~/.zshrc` o `~/.bashrc`).

## Gestione dei pacchetti

Per installare e aggiungere dipendenze al progetto:

```bash
uv add requests pandas numpy
```

Esempio con pacchetti per un corso di fondamenti di programmazione:

```bash
uv add radon typeguard pytest ddt pytest-json pytest-profiling spyder-unittest pylsp-mypy spyder-kernels
```

I pacchetti vengono registrati in `pyproject.toml` sotto `[project.dependencies]` (o `[tool.uv.dev-dependencies]` per le dipendenze di sviluppo) e il file di blocco `uv.lock` viene aggiornato. Le librerie finiscono nell'ambiente indicato da `UV_PROJECT_ENVIRONMENT` (o `.venv` se non specificato).

## Esecuzione dei comandi

```bash
uv run python main.py
uv run pytest
```

`uv run` sincronizza automaticamente l'ambiente con `uv.lock` ed esegue il comando senza richiedere l'attivazione manuale del virtual environment.

## Sincronizzazione manuale

```bash
uv sync
```

Installa nell'ambiente di progetto tutte le dipendenze elencate in `[project.dependencies]` e `[tool.uv.dev-dependencies]`.

### Esempio di `pyproject.toml`

Un file `pyproject.toml` completo si presenta così:

```toml
[project]
name = "mio_progetto"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "ddt>=1.7.2",
    "pylsp-mypy>=0.7.3",
    "pytest>=9.1.1",
    "pytest-json>=0.4.0",
    "pytest-profiling>=1.8.1",
    "radon>=6.0.1",
    "typeguard>=4.5.2",
]

[build-system]
requires = ["uv_build>=0.12.0,<0.13.0"]
build-backend = "uv_build"

[tool.uv]
package = false
```

---

## Riferimenti

- uv: [https://docs.astral.sh/uv/](https://docs.astral.sh/uv/)
- python-build-standalone: [https://github.com/astral-sh/python-build-standalone](https://github.com/astral-sh/python-build-standalone)
