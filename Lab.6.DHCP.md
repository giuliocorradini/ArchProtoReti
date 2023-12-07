# DHCP

È un protocollo di livello 4 basato su UDP per configurare la rete, anche i livelli precedenti.
È strano perché sfrutta uno stack non completamente configurato per configurarsi.

DHCP è documentato in RFC2131.

> After obtaining parameters via DHCP, a DHCP client should be able to exhange packets with any other hosts on the
Internet.

DHCP non dovrebbe essere utilizzato per configurare dei router. Non è che non possiamo farlo, ma nello standard non è
pensato non tanto per limiti tecnici ma perché i router sono le dorsali delle nostri reti. Quando configuriamo delle reti
con DHCP introduciamo problematiche e instabilità della rete.

## Architettura

La soluzione più semplcie prevede il server DHCP dove l'host è configurato staticamente, e dei client che eseguono il
protocollo DHCP ma che non sono ancora configurati a livello rete.

## Obiettivi progettuali

Implementare le policy di rete.

Gli host devono riuscire a connettersi alla rete automaticamente senza alcuna informazione preliminare.

Infine l'architettura deve essere scalabile, cioè deve essere adatto sia per reti semplici che per reti che diventano
sempre più complesse: supportare una crescita di carico, complessità e rete.

I server DHCP possono essere distribuiti e replicati per aumentare l'affidabilità della rete.
DHCP può funzionare su più reti e passare attraverso i router.

DHCP permette di centralizzare la definizione della rete e la configurazione dei dispositivi.

## Porte

Il server ascolta su porta UDP 67 (e risponde con quella), mentre il client invia le richiesta con porta sorgente UDP 68.

```
ss -ulpna
```

## PDU

Il pacchetto DHCP contiene diversi indirizzi. `yiaddr` è l'indirizzo proposto dal server al client. `chaddr` contiene
l'indirizzo hardware, quindi nel caso della nostra rete gli indirizzi MAC. Troviamo una ridondanza legata alla
scalabilità, e riesce a inviare informazioni anche tramite reti differenti.

`xid` è il campo di transaction ID. C'è una problematica legata al fatto che tutti i client usano la stessa porta e
abbiamo un problema di distinzione dei flussi di comunicazione.

`op` è il primo campo, lungo 1 byte e specifica il tipo di messaggio.

## Tipologia dei messaggi

Dal punto di vista della direzione, la struttura del pacchetto rimane inalterata, ma il campo `op` prende valori
particolari quando va da client a server (valore 1) e da server a client (valore 2).

DHCPDISCOVER ha broadcast dH2N (0xffffffffffff) perché il client non ha un IP, non sa chi contattare come IP del DHCP.
Anche a livello 3 c'è un indirizzo broadcast (c'è quello con un netid in cui introduciamo uno scope, e quello globale).
Usiamo l'IP di broadcast senza scope perché non sappiamo di che rete facciamo parte (255.255.255.255).

Il source IP invece è 0.0.0.0 (identifica che l'IP non è stato assegnato, lo troviamo anche nei pacchetti che circolano
nella nostra rete oltre alla configurazione).

Il pacchetto di offerta è destinato all'IP che concettualmente verrà assegnato. Potrebbero esserci diverse varianti,
infatti alcune versioni di DHCP stabiliscono che l'offerta sia unicast.

L'offerta unicast ha una funzionalità di test. DHCP è pensato per funzionare nella modalità più resiliente possibile.
Il server deve assumere che alcuni host vogliano configurarsi staticamente.

### REQUEST

È più specifica rispetto al DISCOVER, e intende dire "ho ricevuto questa offerta, posso utilizzarla?". Il server che
riceve la REQUEST risponde con un ACK oppure un NAK esplicito.

Se il client riceve l'ACK e gli va ancora bene, accetta e si considera configurato. Ma se il client per qualche ulteriore
motivo ritiene l'offerta precedente non più accettabile, invia un DHCPDECLINE.

Si fa per questioni di caching che per evitare di mandare richieste duplicate. In alcuni contesti si manda la REQUEST
dal client direttamente: contesti in cui la rete non è molto dinamica.

A questo punto posso creare un tempo di validità: _tempo di lease_, che identifica la durata dell'assegnazione.

### RELEASE

Rappresenta un ulteriore flusso, e il client lo manda per rilasciare l'assegnazione ricevuta. È un pacchetto di
cortesia. Il client se ne va dalla rete e visto che sa che se ne sta andando, il client DHCP invia un release.

Se il server riceva questo pacchetto può riassegnare gli indirizzi ad altri.

### INFORM

Caso particolare in cui l'host vuole solo i parametri di connessione e non contiene l'indirizzo IP.

## Sicurezza

Posso avere rischi di poisoning. Se assumo che gli utenti si comportino bene allora ho solamente un host nella rete e
le richieste che ricevo sono legittime. Se qualcuno vuole fare danni nella mia rete potrebbe offrire lui stesso un server
DHCP.

Ci possono essere race condition e il server che riesce a inviare più velocemente la offer vince. L'xid offre un
meccanismo di sessione e rende più difficile che un server malevolo vinca la race condition. Se la rete è fatta
correttamente, c'è una best practice per cui il DHCP sia raggiungibile su un server a bassa latenza.

## ISC-DHCP Server

Nel file di configurazione `/etc/dhcp/dhcpd.conf` abbiamo i blocchi subnet che sono quelli che ci interessano per la
configurazione.

```
subnet 192.168.0.0 netmask 255.255.255.0 {
    range 192.168.0.10 192.168.0.250;
}
```

i blocchi subnet devono essere reti su cui è presente anche il server, con un indirizzo IP.
Il range di indirizzi è sempre minore di tutta la rete per introdurre un minimo di intelligenza: se una parte è
configurata staticamente non la includo perché non avrebbe senso assegnare quegli indirizzi.

Sulle macchine di Marionnet, che sono un po' vecchie, non c'è systemd quindi devo usare il comando `service`.

```sh
service isc-dhcp-server start
```

ISC sta per Internet Service Consortium ed è l'implementazione di riferimento per il protocollo.

Il client viene lanciato con `dhclient -i <interface>`

I ritardi del protocollo sono dati dai test ARP per vedere che gli indirizzi che sta per proporre non siano già
presenti.

%TODO: provare con un firewall a bloccare la rilevazione di IP assegnati e causare un problema di IP duplicati.

Per specificare un router, all'interno di un blocco subnet specifichiamo `option routers ip;`.

Per ricaricare la configurazione `service isc-dhcp-server restart`. Gli errori vengono comunicati in `/var/log/syslog`.

Dedicare un indirizzo statico attraverso DHCP accentra la configurazione e semplifica la vita all'amminsitratore di
sistema.

## CIDR

Per configurare automaticamente il routing classless dobbiamo definire:

```
option rfc3441-classless-static-routes code 121 = array of unsigned integer 8;
option ms-classless-static-routes code 249 = array of unsigned integer 8;
```

Impostiamo l'invio di due opzioni differenti per questioni di supporto.

## Intersubnet

Per far passare una richiesta attraverso un router è configurare un relay server, che inoltra le richieste broadcast a
livello locale come richieste unicast verso l'authoritative server.

`INTERFACES=""` lista le interfacce su cui mettere in ascolto il relay.
`SERVERS=""` indica a quali indirizzi inoltrare le richieste.

Come fa il server DHCP a dedurre quale subnet block usare? Analizza su quale interfaccia arriva il pacchetto, e avendo
un IP sa in che sottorete si trova.
