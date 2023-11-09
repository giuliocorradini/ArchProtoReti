# VLAN

In qualche modo estendono le LAN. La V sta per virtual. Cercano di rispondere a un problema del mondo reale: quello di
disaccoppiare la gestione logica delle reti con la topologia fisica.

Posso usare uno stesso switch per due reti logiche all'interno della stessa rete fisica, e dal punto di vista funzionale
la reti sono analoghe a prima. Ho un problema però dal punto di vista della sicurezza: le reti condividono lo stesso
dominio di broadcast Ethernet.

Le VLAN aggiungono dei vincoli per soddisfare un requisito di sicurezza, e non funzionale, quindi non aggiungono nulla
al funzioanmento della rete.

Lo switch deve supportare la divisione dei domini di broadcast. L'assegnazione della VLAN può avvenire a diversi livelli
e in base alle proprietà del pacchetto: porta di ingresso, mac address del frame, livello 3 o 4.

Possiamo configurare due tipi di collegamenti sui bridge: access link e trunk link.

- Una VLAN access link prevede che il nodo terminale collegato alla rete non è consapevole che esiste la VLAN, non c'è
alcuna configurazione sul **nodo finale** per far funzionare la VLAN; che è del tutto trasparente.

- Trunk VLAN, l'host o il router deve avere una configurazione aggiuntiva per sapere che sta comunicando con la VLAN.

- Hybrid quando ci sono entrambe: un dispositivo partecipa in più reti in modalità access e trunk mista.

Mi servono due tipi di link perché quando collego un nodo su più reti posso creare al massimo un tipo di collegamento
access link.

Sulle VLAN trunk il traffico viaggia taggato, ovvero c'è un campo aggiunto nell'header del frame Ethernet.
Lo standard è 802.1q.

Nell'intestazione 802.1q c'è un campo di 12 bit riservato al tagging delle VLAN (valori da 0 a 4096).

## Scenario

Ho il router con un singolo collegamento fisico.

Per creare più reti logiche sullo stesso dominio di broadcast assegno più indirizzi IP alla stessa interfaccia.
Nei sistemi operativi moderni posso specificarlo nativamente, ma con Network Interfaces di Debian devo usare `post-up`
con un comando.

Se lancio un ARPing potrei vedere un redirect host del router (192.168.1.254) che mi dice di fare una comunicazione H2N
diretta per comunicare più velocemente.

Nella tabella di routing di h1 ho una nuova route per `192.168.2.1/32` attraverso `192.168.2.1` (diretto).
Traceroute mi mostra un hop diretto.

Voglio creare reti IP separate per avere una separazione maggiore. Vorrei ad esempio far passare tutto attraverso un
router che potrebbe avere un firewall.

```sh
route add -net 192.168.2.0 dev eth0
```

indica che la destinazione sia raggiungibile direttamente a livello network.

```sh
route add -net 192.168.2.0 gw 192.168.1.254 dev eth0
```

specifico il gateway.

Se in network interfaces specifichiamo il nome dell'interfaccia `eth0:0` andiamo a creare un alias (alias 0).
Attenzione: non è un'interfaccia di rete virtuale.

Uno switch con un terminale è detto _managed_ e posso conifgurare alcuni aspetti, tra cui le VLAN.

## Configurazione VDE

```sh
vlan/create 10
vlan/create 20
```

creo le due VLAN.

Per impostare una VLAN per una certa porta uso `port/setvlan 1 10` mentre per configurare in modalità trunk:
`vlan/addport 10 2`

## Tagged e untagged

Sono sinonimi gergali per trunk e access.

Il progetto da implementare è

```
# LAN1 - 192.168.1.0/24		-> VLAN 10
# LAN2 - 192.168.2.0/24		-> VLAN 20
# Le VLAN sono associate a degli ID liberi, ma spesso vengono scelti nomi
# assonanti.

# Gli identificatori delle VLAN sono informazioni tecniche precise, spesso
# inviate anche su Internet

VLAN10
	H1.eth0 S1.1	access link 10
	ROUTER.eth0 S1.2 trunk 10

VLAN20

	H2.eth0 S1.3	access link 20
	ROUTER.eth0 S1.2 trunk 20
```

Per creare interfacce trunk e access su Linux specifichiamo i nomi delle interfacce con il `.`, ad esempio `eth0.10`.
Queste sono vere **interfacce virtuali**, è un nuovo oggetto.

```
auto eth0.10 eth0.20

iface eth0.10 inet static
    address 192.168.1.254/24

iface eth0.20 inet static
    address 192.168.2.254/24
```

attenzione a dire che la rete funziona perché le VLAN riguardano un requisito non funzionale. Un tool che usa un
protocollo di livello 3 non è sufficiente per testare le VLAN.

```sh
arping 192.168.2.2 -i eth0.20
arping 192.168.2.2 -i eth0.10
```

mi permette di verificare il corretto funzionamento.

## Interfaccia di rete virtuale

Cosa significa? Vuol dire che il nome di quell'oggetto può essere utilizzato in tutti i software che richiedono il nome
di un'interfaccia.

Posso mettermi in ascolto su un'interfaccia con `tcpdump` oppure forzare un `arping` su un'interfaccia di rete virtuale.

Se faccio un `tcpdump` vedo dei pacchetti Ethernet classici, non colgo che questa è un'intefaccia virtuale. Sono su un
trunk che dovrebbe mostrarmi un Ethertype 0x8100 con il tag.

Per vedere il tag mi metto in ascolto su eth0.

Proviamo a generare traffico con arping forzando l'interfaccia fisica di basso livello.

```sh
arping -i eth0 192.168.1.2
```

il primo errore è che manca un IP sorgente. Se specifico invece `-0i` ho una richiesta ARP fuori standard perché come
IP sorgente ho 0.0.0.0, ma va bene. Vedrò traffico solo su `eth0` e non su `eth0.10`.
