---
type: Note
_organized: true
---

# Guida: progetti con `uv` e `pixi`

Questa guida mostra come creare un progetto Python con una versione specifica
dell'interprete (es. 3.12), aggiungere pacchetti e gestire un virtual environment
con nome personalizzato. Vengono trattati sia **uv** che **pixi**.

---

## 1. uv

`uv` è un gestore di pacchetti e progetti Python molto veloce, scritto in Rust.
Supporta la gestione delle versioni di Python, dei virtual environment e delle
dipendenze in un unico strumento.

### Installazione di uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Su macOS con Homebrew:

```bash
brew install uv
```

### Verifica

```bash
uv --version
```

### Creazione del progetto con Python 3.12

Per inizializzare un progetto nella directory corrente (senza creare una
sottocartella):

```bash
uv init --python 3.12
```

Oppure, per creare un progetto in una nuova cartella:

```bash
uv init mio_progetto --python 3.12
cd mio_progetto
```

Questo genera:

- `pyproject.toml` (configurazione del progetto)
- `.python-version` (fissa la versione richiesta)
- `main.py` (file di esempio)
- `README.md`

### Specificare la versione di Python

Il flag `--python` accetta diversi formati:

```bash
uv init --python 3.12          # ultima 3.12.x
uv init --python 3.12.3        # patch esatta
uv init --python >=3.11,<3.13  # intervallo
```

Se la versione richiesta non è presente sul sistema, uv la scarica
automaticamente da [python-build-standalone](https://github.com/astral-sh/python-build-standalone).

### Virtual environment con nome personalizzato

Di default uv crea l'ambiente in `.venv`. Per usare un nome diverso, basta
passarlo a `uv venv` (senza punti o percorsi, è sufficiente un nome semplice):

```bash
uv venv uni --python 3.12
```

Questo crea la cartella `uni/` con l'interprete Python 3.12 (se mancante, uv lo
scarica automaticamente).

Per attivarlo:

```bash
source uni/bin/activate
```

Per fare in modo che uv usi l'ambiente con nome personalizzato nelle operazioni
successive (`uv add`, `uv run`, `uv sync`, ...), imposta la variabile d'ambiente
`UV_PROJECT_ENVIRONMENT`:

```bash
export UV_PROJECT_ENVIRONMENT=uni
```

Così `uv add`, `uv sync` e `uv run` installeranno ed eseguiranno dentro `uni/`
invece di `.venv/`. Per rendere la cosa permanente, aggiungi la riga al file di
configurazione della shell (`~/.zshrc`, `~/.bashrc`).

### Aggiungere pacchetti

```bash
uv add requests pandas numpy
```

Esempio reale con pacchetti per testing e tooling:

```bash
uv add radon typeguard pytest ddt pytest-json pytest-profiling \
      spyder-unittest pylsp-mypy spyder-kernels
```

Per dipendenze di sviluppo:

```bash
uv add --dev pytest ruff
```

Per una versione specifica di un pacchetto:

```bash
uv add "requests>=2.31"
uv add "pandas==2.2.0"
```

I pacchetti vengono registrati in `pyproject.toml` sotto `[project]
dependencies` (o `[tool.uv.dev-dependencies]` per quelli di sviluppo) e il lock
file `uv.lock` viene aggiornato. L'installazione avviene nell'ambiente puntato
da `UV_PROJECT_ENVIRONMENT` (oppure `.venv` se la variabile non è impostata).

### Eseguire comandi nel progetto

```bash
uv run python main.py
uv run pytest
```

`uv run` crea o aggiorna automaticamente l'ambiente sincronizzato con il lock
file, senza bisogno di attivarlo manualmente.

### Sincronizzazione

```bash
uv sync
```

Installare tutte le dipendenze di `[project.dependencies]` e
`[tool.uv.dev-dependencies]` nell'ambiente del progetto.

### Integrazione con PyCharm

Al momento **uv è supportato nativamente** da PyCharm: aprendo il progetto,
l'IDE rileva automaticamente il `pyproject.toml` e il venv creato da uv (o
quello puntato da `UV_PROJECT_ENVIRONMENT`) e lo propone come interprete del
progetto, senza passaggi manuali.

### Comandi utili

| Comando                      | Descrizione                                  |
| ---------------------------- | -------------------------------------------- |
| `uv init <nome> --python X`  | Crea un nuovo progetto                       |
| `uv venv <percorso> --python X` | Crea un venv con nome/path personalizzato |
| `uv add <pkg>`               | Aggiunge una dipendenza                      |
| `uv add --dev <pkg>`         | Aggiunge una dipendenza di sviluppo          |
| `uv remove <pkg>`            | Rimuove una dipendenza                       |
| `uv sync`                    | Sincronizza l'ambiente con il lock file      |
| `uv run <cmd>`              | Esegue un comando nell'ambiente del progetto |
| `uv lock`                    | Rigenera il lock file                        |
| `uv tree`                    | Mostra l'albero delle dipendenze             |
| `uv python list`             | Elenca le versioni di Python disponibili     |
| `uv python install 3.12`     | Installa Python 3.12                         |

---

## 2. pixi

`pixi` è un gestore di progetti multi-linguaggio basato sull'ecosistema
[conda-forge](https://conda-forge.org). Oltre a Python, può gestire dipendenze C/C++,
R, Rust e altro, il che lo rende utile quando servono librerie con parti native
complesse (es. GDAL, OpenCV, CUDA).

### Installazione di pixi

```bash
curl -LSs https://pixi.sh/install.sh | sh
```

Su macOS con Homebrew:

```bash
brew install pixi
```

### Verifica

```bash
pixi --version
```

### Creazione del progetto

Per inizializzare un progetto nella directory corrente (senza creare una
sottocartella), passando `.` come percorso e specificando il canale:

```bash
pixi init . --channel conda-forge
```

Oppure, per creare un progetto in una nuova cartella:

```bash
pixi init mio_progetto --channel conda-forge
cd mio_progetto
```

Questo genera un file `pixi.toml` (usato al posto di `pyproject.toml`) con la
sezione `channels = ["conda-forge"]` già impostata.

### Specificare la versione di Python

In pixi la versione di Python è una normale dipendenza del canale `conda-forge`.
Si aggiunge con:

```bash
pixi add python=3.12
```

Oppure si può scrivere direttamente nel `pixi.toml`:

```toml
[project]
name = "mio_progetto"
version = "0.1.0"
description = "Progetto di esempio"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-64", "osx-arm64", "win-64"]

[tasks]
start = "python main.py"

[dependencies]
python = "3.12"
```

### Aggiungere pacchetti

Si possono aggiungere più pacchetti in un solo comando, compresi Python e le
dipendenze in un colpo solo (pixi capisce che `python=3.12` è il interprete):

```bash
pixi add python=3.12 radon typeguard pytest ddt pytest-json pytest-profiling \
          spyder-unittest pylsp-mypy spyder-kernels
```

Per una versione specifica:

```bash
pixi add "python=3.12"
pixi add "pandas=2.2.0"
pixi add "requests>=2.31"
```

Per integrare pixi con IDE/ambienti specifici, esistono pacchetti dedicati su
conda-forge, ad esempio:

```bash
pixi add pixi-pycharm      # integrazione con PyCharm
```

Dipendenze di sviluppo (sezione `[feature.dev.dependencies]`):

```bash
pixi add --feature dev pytest ruff
```

Per rendere `dev` una feature attiva di default:

```bash
pixi add --feature dev --environments default
```

### Dipendenze PyPI (non disponibili su conda-forge)

Pixi supporta anche pacchetti PyPI in aggiunta a quelli conda:

```bash
pixi add --pypi some-pypi-only-package
```

Genera una sezione `[pypi-dependencies]` nel `pixi.toml`.

### Ambiente/virtual environment

Pixi gestisce automaticamente gli ambienti in `.pixi/envs/default` e non
richiede (nè raccomanda) l'attivazione manuale: i comandi vanno eseguiti con
`pixi run`.

Tuttavia è possibile creare **ambienti multipli** con nomi specifici. Esempio
nel `pixi.toml`:

```toml
[environments]
default = ["py312"]
py310 = ["py310"]
test = ["test"]

[feature.py310.dependencies]
python = "3.10"

[feature.py312.dependencies]
python = "3.12"

[feature.test.dependencies]
pytest = "*"
```

In questo modo:

- `pixi run python ...` usa l'ambiente `default`
- `pixi run -e py310 python ...` usa l'ambiente `py310`
- `pixi run -e test pytest` usa l'ambiente `test`

Per elencare gli ambienti:

```bash
pixi info
```

Per attivare un ambiente manualmente (shell):

```bash
pixi shell            # ambiente default
pixi shell -e py310   # ambiente py310
```

### Eseguire comandi

```bash
pixi run python main.py
pixi run start        # se definito in [tasks]
```

`pixi run` crea/sincronizza automaticamente l'ambiente prima di eseguire.

### Sincronizzazione

L'ambiente viene sincronizzato automaticamente da `pixi run` e `pixi shell`.
Per forzarne la (ri)creazione:

```bash
pixi install
```

### Integrazione con PyCharm

A differenza di uv, **pixi non è (ancora) supportato nativamente** come tipo di
ambiente da PyCharm. Per usare l'ambiente pixi dentro PyCharm occorre:

1. Aggiungere il pacchetto dedicato, che crea un **shim** di `conda`:
   ```bash
   pixi add pixi-pycharm
   ```
2. In PyCharm, quando si aggiunge un interpreter, scegliere **Conda** come tipo
   di ambiente.
3. Come **conda executable** indicare lo shim generato da `pixi-pycharm`,
   situato nella cartella `libexec` del venv di pixi:
   ```
   .pixi/envs/default/libexec/conda        # macOS/Linux
   .pixi\envs\default\libexec\conda.exe    # Windows
   ```
   PyCharm lo usa come se fosse un vero binario `conda` e può così elencare e
   selezionare l'ambiente pixi come interpreter del progetto.

### Comandi utili

| Comando                     | Descrizione                                          |
| --------------------------- | ---------------------------------------------------- |
| `pixi init <nome>`         | Crea un nuovo progetto (con `pixi.toml`)            |
| `pixi add <pkg>`           | Aggiunge una dipendenza                               |
| `pixi add --pypi <pkg>`    | Aggiunge un pacchetto PyPI                            |
| `pixi add --feature F <pkg>` | Aggiunge una dipendenza nella feature `F`          |
| `pixi remove <pkg>`        | Rimuove una dipendenza                                |
| `pixi install`             | Crea/aggiorna gli ambienti                           |
| `pixi run <cmd>`          | Esegue un comando nell'ambiente                       |
| `pixi run -e <env> <cmd>` | Esegue un comando in un ambiente specifico            |
| `pixi shell`              | Attiva l'ambiente nella shell corrente                |
| `pixi shell -e <env>`     | Attiva un ambiente specifico                          |
| `pixi info`               | Mostra info progetto e ambienti                       |
| `pixi list`               | Elenca i pacchetti installati                        |
| `pixi global add <pkg>`  | Installa un pacchetto globalmente                    |

---

## 3. Confronto rapido

| Aspetto                       | uv                              | pixi                              |
| ----------------------------- | ------------------------------- | --------------------------------- |
| Ecosistema                    | PyPI                            | conda-forge + PyPI                |
| Versioni di Python gestite    | Sì (download automatico)        | Sì (modulo `python` su conda)     |
| File di progetto              | `pyproject.toml`                | `pixi.toml`                       |
| Lock file                     | `uv.lock`                       | `pixi.lock`                       |
| Velocità                      | Molto alta (Rust)               | Alta                               |
| Pacchetti C/C++ nativi        | Via wheel PyPI                  | Direttamente da conda-forge       |
| Ambienti multipli             | Tramite variabile `UV_PROJECT_ENVIRONMENT` o venv espliciti | Nativo (`[environments]` + `-e`) |
| Idoneo per                    | App Python pure, web, data science con wheel buone | Data science pesante, GIS, ML, progetti multi-linguaggio |

---

## 4. Flussi di lavoro completi (esempio)

### uv

```bash
uv init --python 3.12          # progetto nella directory corrente
uv venv uni --python 3.12      # venv con nome personalizzato "uni"
export UV_PROJECT_ENVIRONMENT=uni   # da quel momento uv usa "uni"
uv add radon typeguard pytest ddt pytest-json pytest-profiling \
      spyder-unittest pylsp-mypy spyder-kernels
uv sync
uv run python main.py
uv run pytest
```

### pixi

```bash
pixi init . --channel conda-forge     # progetto nella directory corrente
pixi add python=3.12 radon typeguard pytest ddt pytest-json pytest-profiling \
          spyder-unittest pylsp-mypy spyder-kernels
pixi add pixi-pycharm                  # integrazione con PyCharm
pixi run python main.py
pixi shell                             # attiva l'ambiente nella shell corrente
```

Esempio di `pixi.toml` risultante:

```toml
[project]
name = "mio_progetto"
version = "0.1.0"
description = "Demo con pixi e Python 3.12"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-64", "osx-arm64", "win-64"]

[tasks]
start = "python main.py"
test = "pytest"

[dependencies]
python = "3.12"
radon = "*"
typeguard = "*"
pytest = "*"
ddt = "*"
pytest-json = "*"
pytest-profiling = "*"
spyder-unittest = "*"
pylsp-mypy = "*"
spyder-kernels = "*"
pixi-pycharm = "*"
```

---

## Riferimenti

- uv: https://docs.astral.sh/uv/
- pixi: https://pixi.sh/latest/
- conda-forge: https://conda-forge.org/
- python-build-standalone: https://github.com/astral-sh/python-build-standalone