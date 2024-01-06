# NAT

## iptables

Inseriamo delle regole in catene, raggruppate attraverso tabelle mappate in diversi punti dell'analisi dei pacchetti.
Nei nodi di hook possiamo applicare le regole.

Identifichiamo tra tutte le catene due in particolare: post-routing e pre-routing. Quella di output viene usata per il
DNAT sul traffico in uscita dei processi locali.

Ricorda che un'operazione di SNAT (source translation) modifica l'indirizzo IP sorgente di un pacchetto in uscita.
Quindi la regola va aggiunta nella catena di POSTROUTING, collegata all'hook che esce dal multiplexer che collega le
uscite delle due fasi di routing nel diagramma.

## SNAT

Per creare una regola di SNAT con iptables si usa questo comando (modificare a piacere):

```sh
iptables -t nat -A POSTROUTING -j SNAT -o eth1 -s 192.168.1.0/24 --to 1.2.3.4
```

questa regola applica un SNAT a tutti i pacchetti in uscita dall'interfaccia eth1 che provengono dalla rete 192.168.1.0/24
indicata con `-s`. Tutti gli indirizzi vengono tradotti a `1.2.3.4`.

Se vogliamo mascherare (operazione simile al SNAT) tutti gli indirizzi di LAN 1 e farli uscire con `4.3.2.1` usiamo:

```sh
iptables -t nat -A POSTROUTING -j SNAT -o eth1 -s 192.168.1.0/24 --to-source 1.2.3.4
```

Masquerade si usa quando l'indirizzo IP della scheda di rete è assegnato dinamicamente, infatti non dobbiamo specificare il
`--to-source`.

> È importante specificare l'interfaccia di uscita! `-o ethx`

## DNAT

Per esporre un servizio su un'interfaccia di rete bisogna fare DNAT. Si usa il seguente comando:

```sh
iptables -t nat -A PREROUTING -i eth0 -j DNAT -d 4.3.2.1 -p tcp --dport 80 --to-destination 10.108.54.2:80
```

Siamo in catena di prerouting perché stiamo impiegando l'obiettivo di DNAT a pacchetti che provengono dall'esterno, e non
potremmo quindi applicare alcun SNAT.

Per esporre completamente un host omettiamo la parte del protocollo e della porta di destinazione. Non stiamo facendo
port forwarding così.

Per inserire una regola in una certa posizione usiamo l'azione `-I` su una catena.

```
-I POSTROUTING <pos> <coda regola>
```