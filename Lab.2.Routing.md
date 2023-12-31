# Panoramica sulle interconnessioni alle reti e accesso a Internet
Cosa succede sono interessato a mettere in comunicazioni reti e non più host?

Dalla *Federal Networking Council*
> Internet refers to the **global information system**...

## Organizzazione interna di Internet
Ha una architettura *lascamente* (debolmente, non strettamente) gerarchica: è una rete gerarchica in cui alcuni livelli potrebbero mancare.

Gli host sono connessi tramite ISP - Internet Service Provider - locali.
Gil ISP locali sono connessi a **ISP regionali**.
Gli ISP regionali sono connessi a **ISP internazionali**: **NBP - National Backbone Provider** o **NSP - National Service Provider**

## Gestione dell'infrastruttura
**POP** - Point of Presence -> infrastrutture con cui in ISP gestisce localmente il collegamento (di privati o pubblici) alla propria rete.

**AS** - Autonomous System -> regioni in cui sono aggregati i router. Insieme di reti, indirizzi IP e router sotto il controllo della stessa organizzazione o consorzio.

Con una vista top-down Internet è un'aggregazione di Autonomous Systems.

#Vedi UUNET backbones MCI

I NBP gestiscono anche i cavi intercontinentali e oceanici. Questi AS vengono solitamente mappati ad entità legali. Livello legale, gestionale, tecnico.

#Vedi GARR - Gruppo di Armonizzazione della Rete di Ricerca

Tutte le reti all'interno di uno stesso AS non sono disconnesse, quindi per comunicare tra loro non passano attraverso AS differenti.

TIM, entità legale, può avere in gestione più AS, ad esempio per interconnettere le reti tra più stati.

#Vedi www.submarinecablemap.com

## Protocolli PPP
Gli *accessi residenziali* cablati sono prevalentemente **point-to-point**

Backbone LAN: rete locale che sta *in mezzo* alla rete globale

Modalità di accesso:
- Dial-up - analogico ma deprecato
- ISDN - digitale ma deprecato
- xDSL - analogico
- Fibra ottica - digitale
Ma anche via radio:
- reti wireless
- FWA

### Dial-up
Utilizza la rete telefonica e richiede che venga stabilito un circuito
Modem dial-up dalla banda (teorica) di 56Kbps
### Integrated Services Digital Network
Linea digitale commutata

Velocità massima di 128Kbps

### x Digital Subscriber Line
Da ADSL a VDSL2

Si modula un segnale sul doppino telefonica.

### FFTx
Da FTTH a FTTC/FTTB (ultimo miglio in rame)

### Fixed Wireless Access
Per zone a scarsa densità abitativa

Fixed in contrapposizione alle stazioni wireless mobili (*mobile*)

Il collegamento può avere banda **dedicata** o **condivisa** con gli altri utenti -> nel secondo caso diminuisce il costo del servizio

### Frame PPP
- flag: delimitatori del frame PPP
- address: non utilizzato
- control: non utilizzato, per utilizzi futuri
- protocol: protocollo di livello superiore codificato nel payload
- info: payload
- control: crc o simili

# Internet
## Packet vs circuit switching
La logica di funzionamento, quindi la complessità, l'intelligenza della rete diciamo risiede nei dispositivi intermedi nelle reti *circuit switching*, nei dispositivi terminali nelle reti *packet switching*

Nella rete Internet sono gli host a gestire la complessità. I nodi terminali devono gestire poco traffico, quindi possono permettersi di introdurre latenze e simili. Nodi intermedi più semplici facilitano la scalabilità.

## Viste a diversi livelli di astrazione
- **Internet** è una rete trasparente tra client e server.
- Dal punto di vista organizzativo è un insieme di oltre 5000 AS su diverse scale -> non monolitico ma decentrato.
- Organizzazione gerarchica dei nomi, non integrata in IP -> DNS, protocollo ortogonale a IP

## Principi funzionali di progetto
L'affidabilità non è un first-class citizen in Ethernet.

**Survivability**: se tra due host esiste un qualsiasi percorso, la comunicazione deve poter avvenire.
**Forma a clessidra**: minime assunzioni sui mezzi di trasporto sottostanti e funzionamento per tutti i tipi di applicazioni di rete.
**Stateless**: l'intelligenza è mantenuta ai bordi della rete; riduce la complessità -> no stato distribuito perché i nodi intermedi sono stateless
**Net neutrality**: vede un acceso dibattito negli USA. I gestori della rete non possono attuare scelte di natura economica o politica sull'inoltro dei pacchetti. Qualsiasi ISP non deve scegliere quali pacchetti inoltrare (o simili) sulla base del suo contenuto.

## Storia
1. Anni '60 - sviluppo della teoria sui sistemi packet switching e primi esperimenti
2. 1969 - Nasce **ARPAnet**
3. Inizio ani '90 - si dismette ARPAnet, nasce WWW (Tim Barners Lee)

#Completa 
#Rivedi storia Internet

# Instradamento dei pacchetti
Ci interessa la comunicazione tra più reti, non come più dispositivi della stessa rete comunicano tra loro.

In Ethernet non ci interessa dove si trova il dispositivo con cui comunicare, perché un messaggio broadcast arriva a tutti.

In Internet non è possibile, bisogna evitare di inondare la rete con schifezze: problema di instradamento.

## Router
> Dispositivo di LV3 che risolve il problema di instradamento dei pacchetti nella rete da un host a qualsiasi altro host sulla base dell'indirizzo IP di destinazione.

## Routing IP
Un pacchetto che compie un cammino da HostA a HostB passando per Internet (per molte reti in generale). I due host si possono trovare all'interno di reti diverse tra loro.

1. capisco se la dst appartiene alla mia rete
2. se non appartiene, a chi lo mando? ****
3. Il router è un dispositivo che riceve un pacchetto non indirizzato a lui ma ad un altro dispositivo: nessun router ha la visione completa del cammino scelto dal pacchetto. La src decide solo il first hop; ogni router decide il **next hop**

Questo approccio può non funzionare sempre bene -> protocollo best effort.

Possono esserci problemi di loop, configurazione, malfunzionamenti.
Il primo router si dice **source router** o **first hop router**. Ogni router decide a sua volta il **next hop router** che appartenga alla stessa rete.
Il router di arrivo è **destination router**.

La src invia al src router il frame con IP dst appartenente ad una rete diversa.
> Il router si distingue dagli host perché accetta pacchetti non indirizzati a lui a LV3

I pacchetti Internet contengono un TTL -> ad ogni salto decresce di 1.
Per evitare pacchetti IP fantasma che rimangono per sempre nella rete.

### Sottoproblemi
**Sottoproblema 1**: ad ogni pacchetto in ingresso determinare #Completa
**Sottoproblema 2**: formazione della tabella di routing
## IP Forwarding
>Meccanismo di inoltro con cui un router trasferisce un datagram dalla in if alla out if. L'out if corrisponde a quella if sulla rete connessa a livello h2n al next hop.

Viene usato l'indirizzo IP di destinazione presente nell'header del datagramma.

## Tabella di routing
Ogni host e ogni router hanno una tabella di routing in cui ciascuna riga è composta da: *destinazione - next hop*, inoltre è presente una entry che definisce il next hop di default per ogni rete non matchata, chiamata *default router* o *default gateway* (regola di fallback).

La rete di appartenenza di un indirizzo IP la posso dedurre grazie alla struttura gerarchica degli indirizzi.

La tabella di routing può essere statica o dinamica. In contesti AS è solitamente dinamica, per le LAN va bene una tabella statica.

Se la tabella di routing è vuota o non ha la destinazione richiesta: **network unreachable**
Non ha logiche di fallback come lo switch che inoltra a tutte le if.

LV2: host unreachable, LV3: network unreachable

Host unreachable viene sollevato (interpretazione non contenuto nel protocollo) quando il timeout di un'ARP request scade.

Le dimensioni delle tabelle di routing potrebbero essere un limite allo sviluppo di Internet. Teoricamente i router dovrebbero mantenere una entry per ogni rete raggiungibile su Internet.

Nel mondo reale di usano regole di **aggregazione** (regola aggregata) -> tecniche di aggregazione per fare in modo che una regola catturi più reti

#Vedi: OSPF, BGP, #Completa 

# Architettura di router
Componenti:
- porta di ingresso
- commutatore
- processore di routing
- porta di uscita

#Completa con disegno
## Porte di ingresso
Funzioni di livello 1, 2, 3
```
-> line termination -> data link processing & filtering -> logica di accodamento
```

Se la logica di switching è lenta può succedere le l'accodamento fallisca perché la coda è piena.

## Componenti di switching
>Spostamento del pacchetto dalla porta di ingresso a quella di uscita

Architettura:
- switch -> basata su memoria, costosa
- bus -> economica, può far comunicare solo una coppia di porte alla volta
- crossbar -> rete di multiplexer, limite sottostante alla velocità massima di un'architettura di questo tipo

Può essere realizzata in hardware o in software.
#completa con disegno
## Porta di uscita
Fa accodamento: può riempirsi se la velocità di invio è molto lenta, non le permette di svuotarsi. Congestione.

#completa architettura

Il LV3 non se ne occupa. Solitamente le code sono FIFO.

# Indirizzi IP
Un indirizzo IP è un indirizzo dalla dimensione di 32 bit, ovvero 4 byte, in notazione huma-readable separati da punto.

Ogni byte ha un valore compreso tra 0 e 255.

Interpretazione gerarchica degli indirizzi IP (**struttura gerarchica**), contrapposta all'interpretazione piana dell'indirizzo MAC. A sinistra stanno i valori più significativi. Ogni indirizzo è costituito dalla coppia:
```
<netid, hostid> # <prefisso di rete, indirizzo dell'host>
```

Gli indirizzi IP hanno **lunghezza fissa**. Esistono sistemi di indirizzamento dalla lunghezza variabile.

Cosa definisce la linea di demarcazione tra prefisso di rete e hostid?
- rappresentazione storica -> *classful*: classi di indirizzi
- rappresentazione moderna -> *classless*: subnet mask
## Classful representation
- 3 classi per indirizzamento di host: A, B, C
- 1 classe multicast: D
- 1 classe riservata: E

Classi:
- **Classe A**: primo bit (msb) a 0. 0 - 127. 16M host
- **Classe B**: prefisso *10*. 128 - 191. 65k host
- **Classe C**: prefisso *110*. 192 - 223. 256 host
- **Classe D**: prefisso *1110*. 224 - 239
- **Classe E**: prefisso *11111*. 240 - 255
Ma man che cresce la classe aumenta la dimensione del numero di reti.

Trucco: scala di 1k ad ogni aggiunta di 10 alla potenza del due.

Ho a disposizione: $2^{bit\_liberi} - 2$ device. Tolgo broadcast (tutti a 1) e indirizzo di rete (tutti a 0, usato nei vecchi protocolli per identificare la rete)
## Classless
Permette architetture più flessibili, maggior controllo granulare.
Si usa la notazione **CIDR** - Classless Inter-domain Routing.

L'insieme dei bit del netid è indicato da un numero **n**:
```
a.b.c.d/n
```

Questo numero viene chiamato *netmask*.
Implicitamente la netmask definisce una classe.

Unimore possiede gli indirizzi: `155.185.0.0/16`

### Notazione estesa
4 byte separati da un punto. Permetterebbero di assegnare bit di HostID sparsi (non contigui), per convenzione non viene fatto.

**Network addr**: `NetID & Netmask`
**Broadcast addr**: `NetID | Netmask`

Gli indirizzi IP assegnabili sulla rete sono confinati da NetworkAddr e BroadcastAddr. Sono indirizzi da scartare.

Il numero massimo di host è dato dalla formula: $2^{(32-N)}-2$ dove $N$ è il numero di bit della netmask.

#Esercizi slide 13 -> calcolo
## Indirizzi IP speciali
- Network address: hostid tutto a zero -> denota la rete. Es: 192.168.1.0/24, 127.0.0.0/8
- Directed broadcast address: hostid tutto a 1 -> permette il broadcast a tutti gli host della rete
- Limited broadcast address: tutti i bit a 1 -> broadcast della rete fisica locale (limited perché mai inoltrato dai router). Qualsiasi host connesso alla rete locale accetterà il pacchetto
- Nessun indirizzo IP: tutti i bit a 0 -> usato per il boot o per configurazioni particolari
- Loopback o localhost: tutti gli indirizzi della rete 127.0.0.0/8, sicuramente 127.0.0.1

#Attenzione 192.168.1.0/23 è assegnabile, non di rete

Il limited broadcast address serve per evitare flooding di pacchetti. Viene usato per mandare datagram che debbano arrivare a tutti gli host che a LV2 appartengono allo stesso dominio di broadcast, ma possono appartenere a reti logiche LV3 diverse.

Il directed broadcast invia anche a livello logico il pacchetto solo agli host della stessa rete.

Broadcast IP implica broadcast MAC

Nulla mi vieta di avere più reti logiche sulla stessa rete h2n.

Nel protocollo DHCP i pacchetti di broadcast vengono usati per richiedere un nuovo IP, dato che non si conosce l'indirizzo del server.

## Non routable IP addresses
- classe A: `10.0.0.0/8`
- classe B: `172.16.0.0/12`
- classe C: `192.128.0.0/16`

Sono classi di indirizzi privati, risolti a livello locale e sicuramente non duplicati.

Se si assegna un indirizzo non privato ad un host della propria rete si crea ambiguità su chi debba essere raggiungo.

# Subnetting e supernetting
Unimore ha a disposizione *155.185.0.0/16*. Gli indirizzi assegnabili sono tra *155.185.0.1* e *155.185.255.254*

Il subnetting gerarchico modifica l'interpretazione di una rete locale 


Per convenzione gli ultimi indirizzi IP vengono solitamente assegnati ai router della rete. Gli host partono dal basso, i router dell'alto.

#Nota differenza tra *Network unreachable* e *Host unreachable*. Nel primo caso non genero neanche traffico.

#TODO revise