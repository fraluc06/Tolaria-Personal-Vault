---
type: Note
url: https://orbstack.dev
---
# OrbStack unminimize Ubuntu machine (macOS)

Le macchine Linux su OrbStack usano immagini **"minimized"**: stripped di manuali,\
documentazione e strumenti interattivi per occupare meno spazio su disco.

***

## Sintomo tipico

```bash
ssh francesco@orb
# oppure
orb

francesco@ubuntu:~$ man
-bash: man: command not found
```

***

## Soluzione 1 — Comando `unminimize` (se disponibile)

Nelle immagini Ubuntu standard è presente lo script integrato:

```bash
sudo unminimize
```

- Premi `y` alla richiesta di conferma.
- Scarica e reinstalla tutti i pacchetti rimossi (man pages, doc, strumenti di sistema).
- Il processo richiede alcuni minuti in base alla velocità della connessione.

***

## Soluzione 2 — Ripristino manuale (se `unminimize` non è presente)

Alcune immagini sono così ridotte da non avere nemmeno `unminimize`.\
In questo caso, procedere a mano:

### Step 1 — Rimuovere i blocchi alla documentazione

Ubuntu minimal ha un file che impedisce ad `apt` di installare le pagine man.\
Va eliminato **prima** di installare qualsiasi cosa, altrimenti i manuali\
rimarranno vuoti anche dopo l'installazione.

```bash
sudo rm -f /etc/dpkg/dpkg.cfg.d/excludes
```

### Step 2 — Reinstallare `unminimize` (tentativo)

Lo script fa parte del pacchetto `ubuntu-minimal`. Provare a reinstallarlo:

```bash
sudo apt update
sudo apt install --reinstall ubuntu-minimal
```

Se `/usr/bin/unminimize` compare, eseguirlo. Altrimenti continuare con lo step 3.

### Step 3 — Installare il sistema dei manuali

```bash
sudo apt update
sudo apt install man-db manpages manpages-dev manpages-posix
```

### Step 4 — Installare i tool standard di Ubuntu

Installa il meta-pacchetto che ripristina tutti gli strumenti presenti in una\
Ubuntu "normale":

```bash
sudo apt install ubuntu-standard
```

### Step 5 — Recuperare i manuali dei pacchetti già installati

I pacchetti installati *prima* della rimozione del blocco non hanno i manuali.\
Reinstallare almeno `coreutils` per recuperare `ls`, `cp`, `mv`, ecc.:

```bash
sudo apt install --reinstall coreutils
```

***

## Pacchetti utili da installare subito

| Cosa manca            | Pacchetto da installare |
| --------------------- | ----------------------- |
| `man`                 | `man-db manpages`       |
| `ping`                | `iputils-ping`          |
| `ifconfig`, `netstat` | `net-tools`             |
| `curl`, `wget`        | `curl wget`             |
| `vim`                 | `vim`                   |
| `gcc`, `make`         | `build-essential`       |

Installazione rapida di tutti in un colpo:

```bash
sudo apt install iputils-ping net-tools curl wget vim build-essential
```

***

## Script unico "tutto in uno"

Per non pensarci più, incollare questo script nel terminale della macchina OrbStack:

```bash
#!/bin/bash
set -e

echo "==> Rimozione blocchi documentazione..."
sudo rm -f /etc/dpkg/dpkg.cfg.d/excludes

echo "==> Aggiornamento indice pacchetti..."
sudo apt update

echo "==> Tentativo reinstallazione ubuntu-minimal..."
sudo apt install --reinstall ubuntu-minimal -y

echo "==> Installazione man, strumenti standard e tool comuni..."
sudo apt install -y \
  man-db manpages manpages-dev manpages-posix \
  ubuntu-standard \
  iputils-ping net-tools curl wget vim build-essential

echo "==> Reinstallazione coreutils per recuperare i manuali base..."
sudo apt install --reinstall -y coreutils

# Esegui unminimize se ora è disponibile
if command -v unminimize &> /dev/null; then
  echo "==> Esecuzione unminimize..."
  sudo unminimize
fi

echo ""
echo "Fatto. Prova: man ls"
```

***

## Note

- OrbStack non ha un pulsante "ripristina": il terminale **è** l'interfaccia.
- Per rientrare nella macchina dal Mac: `orb` (default) oppure `orb -m nome_macchina`.
- Per aprire una sessione dal pannello OrbStack: **Linux Machines → icona Terminale**.
- Le app grafiche funzionano via **X11 forwarding** (supportato nativamente da OrbStack): installa l'app (`sudo apt install x11-apps`) e lanciala dal terminale — apparirà come finestra nativa su macOS.
