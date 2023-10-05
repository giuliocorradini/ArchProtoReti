# Comandi

## Configurazione di rete

`ip neigh` mostra i vicini "raggiungibili" ovvero le entry della ARP table senza TTL scaduto.
Dopo un po' da REACHABLE, i record diventano STALE ovvero il record (anche se è ancora nella ARP table) è scaduto.

In base all'implementazione alcuni sistemi potrebbero ottimizzare e mandare una richiesta ARP Unicast se la richiesta è
in STALE.

## Sniffing

Fa il dump dei pacchetti che transitano su un'interfaccia di rete.

```sh
tcpdump -i eth0 -e -n
```

L'opzione `-e` stampa l'header a livello link (livello 2 ISO/OSI). L'opione `-n` inibisce la conversione degli indirizzi
in nomi.

Nell'output di `tcpdump`, un frame Ethernet è così denotato: `source MAC > destination MAC`. C'è anche già un'interpretazione di basso livello del campo type, infatti tcpdump mostra il mapping tra ethertype e protocollo livello 3 corrispondente.

Ping non invia del traffico ARP, ma è il sistema operativo che esegue la logica Ethernet (divisa tra OS e NIC).

## Detached terminal

Permette di eseguire comandi e non dipendere dall'affidabilità dell'emulatore di terminale. Su tmux si può fare entrando in modalità comando con `ctrl + b` e poi si preme `d` per detach.

Per rientrare nel terminale basta fare `tmux attach`.

Per navigare tra split che si creano con `c` si usano i numeri.

## arping

Funziona in modo simile al ping ma sfrutta il protocollo ARP per sondare la rete, il programma accetta anche la NIC.

## Configurazione permanente

Si trova in `/etc/network/interfaces`. Si può dividere su più file  erichiamarli con `source-directory` oppure `source`.

```
iface eth0 inet static
    address 192.168.1.1/24
```

per leggere e configurare un'interfaccia si usa `ifup`. Se modifico, il sistema non riflette automaticamente le nuove
configurazioni. Se tutto va bene dovrei avere l'interfaccia configurata e lo vedo con `ifconfig`.

La direttiva `iface` viene letta a tempo di boot solo se l'interfaccia viene abilitata in autoconfigurazione con la
direttiva `auto <if name>`.
