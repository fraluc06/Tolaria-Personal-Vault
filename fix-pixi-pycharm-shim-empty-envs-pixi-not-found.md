---
type: Note
_organized: true
url: https://github.com/pavelzw/pixi-pycharm/issues/74
---
# Fix: pixi-pycharm shim — environment vuoti e pixi non trovato in PyCharm

Lo shim `pixi-pycharm` fa credere a PyCharm di avere a che fare con un
eseguibile `conda` reale. PyCharm chiama `conda env list --json` per
scoprire gli environment disponibili e `conda run ... python` per eseguire
comandi dentro l'environment. Lo shim intercetta queste chiamate e le
traduce in comandi `pixi` equivalenti.

## Sintomi

1. In PyCharm, configurando l'interpreter come **Conda Environment** con il
   shim `pixi-pycharm`, la lista degli environment risulta **vuota**.
2. Dopo aver fixato il primo problema, PyCharm mostra l'errore
   `FileNotFoundError: pixi not found. Is it installed in your PATH?`
   quando cerca di usare l'environment.

***

## Fix 1 — `pixi_envs()` restituisce una lista vuota

### Causa

La funzione `pixi_envs()` nello shim `.pixi/envs/default/libexec/conda`
elenca gli environment filtrandoli per piattaforma.

Versione originale (buggata):

```python
def pixi_envs():
    platform = pixi_json(["info", *std_pixi_args()])["platform"]
    envs = pixi_json(["info", *std_pixi_args()])["environments_info"]
    return [env["prefix"] for env in envs if platform in env["platforms"]]
```

Il filtro `platform in env["platforms"]` assume che `env["platforms"]` sia
una **lista di stringhe**:

```python
["osx-arm64", "linux-64"]
```

Le versioni recenti di pixi restituiscono invece una **lista di dizionari**:

```python
[
    {"name": "osx-arm64", "subdir": "osx-arm64", "virtual_packages": []}
]
```

Il confronto `"osx-arm64" in [{"name": "osx-arm64", ...}]` restituisce
sempre `False` perché confronta una stringa con oggetti dict. Risultato:
la lista `envs` restituita a PyCharm è vuota.

Issue upstream: https://github.com/pavelzw/pixi-pycharm/issues/74

### Fix

Estrarre il campo `name` da ogni dizionario prima di confrontare, gestendo
anche il caso in cui una futura versione di pixi torni al formato stringhe:

```python
def pixi_envs():
    platform = pixi_json(["info", *std_pixi_args()])["platform"]
    envs = pixi_json(["info", *std_pixi_args()])["environments_info"]
    result = []
    for env in envs:
        names = [p if isinstance(p, str) else p["name"] for p in env["platforms"]]
        if platform in names:
            result.append(env["prefix"])
    return result
```

***

## Fix 2 — `conda_run` non trova pixi (installato via Homebrew)

### Causa

La funzione `conda_run()` dello shim lancia `pixi run` per eseguire comandi
dentro l'environment. Usa due fallback per trovare l'eseguibile `pixi`:

1. `os.execlp("pixi", ...)` — cerca `pixi` sul `PATH`
2. `os.execlp(str(DEFAULT_PIXI_EXECUTABLE_PATH), ...)` — cerca in `~/.pixi/bin/pixi`

Manca il fallback per l'installazione via **Homebrew** (`/opt/homebrew/bin/pixi`).
La funzione `pixi()` (usata da `pixi_envs`) ha invece tre fallback: PATH,
Homebrew, e `~/.pixi/bin`.

Quando PyCharm lancia lo shim, il `PATH` non include `/opt/homebrew/bin`, e
`~/.pixi/bin/pixi` non esiste (pixi è installato via Homebrew). Risultato:
`conda_run` non trova pixi e fallisce con `FileNotFoundError`.

### Fix

Aggiungere il check di `PIXI_HOMEBREW_PATH` in `conda_run`, coerentemente
con quanto fa già la funzione `pixi()`:

```python
cmd = ["pixi", "run", *std_pixi_args(), *pixi_env_cli_params(env), *pixi_run_args]
try:
    os.execlp("pixi", *cmd)
except FileNotFoundError as e:
    pixi_path = None
    if PIXI_HOMEBREW_PATH.exists():
        pixi_path = str(PIXI_HOMEBREW_PATH)
    elif DEFAULT_PIXI_EXECUTABLE_PATH.exists():
        pixi_path = str(DEFAULT_PIXI_EXECUTABLE_PATH)
    if pixi_path:
        os.execlp(pixi_path, pixi_path, *cmd[1:])
    msg = "pixi not found. Is it installed in your PATH?"
    raise FileNotFoundError(msg) from e
```

***

## Come applicare i fix

Entrambi i fix modificano lo stesso file.

1. Aprire il file shim:

   ```
   .pixi/envs/default/libexec/conda
   ```

2. **Fix 1** — sostituire la funzione `pixi_envs()` (riga ~73) con la
   versione corretta mostrata sopra.

3. **Fix 2** — sostituire il blocco `try/except` in `conda_run()` (riga
   ~157) con la versione corretta mostrata sopra.

4. Verificare che i fix funzionino:

   ```
   .pixi/envs/default/libexec/conda env list --json
   PATH=/usr/bin:/bin .pixi/envs/default/libexec/conda run -n default python -c "import sys; print(sys.executable)"
   ```

   Il primo comando deve elencare l'ambiente sotto `"envs"`. Il secondo
   deve stampare il path dell'interprete Python (verifica che il fallback
   Homebrew funzioni anche senza `/opt/homebrew/bin` sul PATH).

5. In PyCharm: **Add Python Interpreter → Conda → Using existing
   environment**. Ora la lista contiene l'ambiente pixi e l'interprete
   funziona.

***

## Limiti

- I fix modificano un file dentro `.pixi/`, rigenerato da `pixi install`.
  Se si esegue `rm -rf .pixi && pixi install` le patch vengono perse e
  vanno riapplicate.
- I fix sono locali al workspace corrente; altri workspace pixi vanno
  patchati separatamente.
- Quando upstream rilascerà una nuova versione di `pixi-pycharm` con le
  correzioni (issue #74 ancora aperta), le patch non saranno più
  necessarie.