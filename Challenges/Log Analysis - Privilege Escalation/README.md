# Log Analysis — Privilege Escalation

## Panoramica

- **Piattaforma:** [Blue Team Labs Online](https://blueteamlabs.online/)
- **Categoria:** Log Analysis
- **Argomento:** Analisi delle attività dell'attaccante ed escalation dei privilegi
- **Stato:** Completata

## Obiettivo

Analizzare i log forniti dalla challenge per ricostruire le attività dell'attaccante, individuare i comandi eseguiti e identificare la configurazione errata sfruttata per ottenere privilegi di root.

## Metodologia di analisi

Ho esaminato i comandi registrati nei log per ricostruire le attività dell'attaccante e individuare le informazioni richieste dalla challenge.

Le prime domande riguardavano elementi direttamente riconoscibili nei comandi, come la directory dell'utente `daniel` e l'esecuzione dello strumento di analisi di rete `tcpdump`.

L'ultima domanda ha richiesto un approfondimento ulteriore, perché l'attaccante aveva eseguito numerosi comandi per esplorare il sistema e cercare possibili modalità di escalation dei privilegi.

## Analisi dell'escalation dei privilegi

### 1. Ricognizione del sistema

Nei log sono presenti diversi comandi utilizzati per esplorare il server, tra cui:

- `whoami` e `id`, per verificare l'identità e i privilegi dell'utente corrente.
- `ls -la` e `pwd`, per esaminare directory e posizione nel filesystem.
- `ps -ef` e `ps -aux`, per visualizzare i processi in esecuzione.
- `sudo -l`, per verificare i privilegi disponibili tramite sudo.
- `tcpdump`, per identificare l'esecuzione di uno strumento di analisi del traffico di rete.

### 2. Ricerca di possibili vie di escalation

L'attaccante ha cercato eseguibili come Perl, Python e compilatori C, oltre a controllare configurazioni, servizi, processi e permessi del sistema.

Un comando particolarmente rilevante è:

```bash
find / -type f -user root -perm -4000 2>/dev/null
```

Questo comando cerca file regolari appartenenti a `root` con il bit SUID impostato, una configurazione che può consentire a un programma di essere eseguito con i privilegi effettivi del proprietario.

### 3. Sfruttamento del binario Python

Dopo la ricerca dei file SUID, nei log compare il comando:

```bash
./usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

Il comando utilizza Python per avviare una shell `/bin/sh` con l'opzione `-p`, che in questo contesto serve a preservare i privilegi effettivi.
La sequenza è coerente con lo sfruttamento di una configurazione SUID insicura sul binario Python per ottenere una shell con privilegi di root.

### 4. Risposta alla domanda finale

**Domanda:** quale configurazione errata del binario `python` è stata sfruttata per ottenere accesso con privilegi di root?

**Risposta:** SUID.

La risposta è supportata dalla ricerca dei file SUID appartenenti a `root` e dal successivo utilizzo di Python per avviare una shell preservando i privilegi effettivi.
Per confermare il significato di questa sequenza ho approfondito la documentazione tecnica e utilizzato un assistente AI come supporto alla comprensione dei comandi.

## Lezioni apprese

Questa challenge mi ha aiutato a distinguere le attività di ricognizione dalle tecniche di escalation dei privilegi.
La parte più impegnativa è stata distinguere i diversi comandi eseguiti dall'attaccante e individuare quale configurazione avesse consentito di ottenere privilegi di root.
In particolare, ho approfondito il significato del bit SUID, il suo potenziale impatto sulla sicurezza e l'importanza di correlare più comandi per interpretare correttamente le attività registrate nei log.


