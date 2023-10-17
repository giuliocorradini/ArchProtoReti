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

## Nome dell'host

Bisogna agire su `/etc/hosts`, il file che mappa gli indirizzi ip ai corrispondenti nomi.

Di solito ci interessa una risoluzione diretta, ovvero da nome a indirizzo IP.
Nei pacchetti IP che ci si scambia, i nomi degli host non esistono, e la gestione è completamente locale. Li ritroveremo
invece nel protocollo HTTP.

## tcpdump pt.2

`tcpdump -i eth0`

Ho tolto l'opzione `-e`, quindi non vedo più i frame ma direttamente la parte IP. Inoltre non specificando `-n` vedo che
gli indirizzi IP vengono rimpiazzati dal corrispondente nome dell'host.

Ad esempio `192.168.1.1 > 192.168.1.2` diventa `h1 > h2`. Se ometto l'opzione tengo abilitata l'interpretazione di
alcuni campi dei pacchetti.

## nsswitch.conf

È un file locato in `/etc/nsswitch.conf`, sta per _namespace switch_. È un file di configurazione centrale che governa
come il sistema operativo, centralmente, fa le risoluzioni. Non governa soltanto i nomi dei protocolli di rete.

Contenuto di esempio:

```
passwd:     compat
group:      compat
shadow:     compat
gshadow:    compat

hosts:      files dns
networks:   file

...
```

La direttiva `hosts:      files dns` va letta così: prima risolvo utilizzando il file `/etc/hosts`, poi uso il
protocollo dns per risolvere il nome degli host.

Modificare questi file di configurazione sono metodi efficaci e noti a pochi per disabilitare completamente alcuni
protocolli.

## Managed

Il tick "Show VDE terminal" permette di abilitare una finestra di configurazione dello switch.

FSTP è il fast spanning tree protocol.

`brctl addbr br0`, creo un bridge e posso aggiungere le interfacce.
`brctl addif br0 eth0`, aggiungo l'if eth0 al bridge br0 e poi

`brctl addif br0 eth1`

`brctl addif br0 eth2`

devo attivare le singole interfacce di rete e attivare anche il bridge. Si fa tutto con ifconfig come al solito.

Configurazione automatica del bridge da network interfaces

```
iface br0 inet static
    bridge_ports <ifaces>
```

la direttiva `bridge_ports` è quella fondamentale che stabilisce che l'interfaccia è di tipo bridge.
Si può anche creare un bridge a cui non passiamo nessuna porta ma il parametro 0 così che il bridge venga creato a tempo
di boot senza nessuna interfaccia virtuale associata.

Con una direttiva interface bisogna sempre avere una direttiva `address`. Quando abbiamo un bridge non è scontato che
abbiamo un indirizzo IP associato. Possiamo comunque dargli indirizzo `0.0.0.0` per non assegnare l'indirizzo IP al
bridge.

Ci sono alcuni casi in cui queste direttive non attivano le interfacce associate sotto. Per essere sicuro che
un'interfaccia fisica sia attivata faccio una cosa del genere:

```
auto br0 eth0 eth1 eth1

iface br0 static inet
    bridge_ports eth0 eth1 eth2
    addres 0.0.0.0

iface eth0 static inet
    addres 0.0.0.0

iface eth1 static inet
    addres 0.0.0.0

iface eth2 static inet
    addres 0.0.0.0
```

Associo un indirizzo IP nullo, altrimenti sarebbe un errore. Il comando `ifup -a` cerca di attivare tutti i blocchi del
file di network/interfaces. Però se c'è un errore non si riesce a capire dov'è.
