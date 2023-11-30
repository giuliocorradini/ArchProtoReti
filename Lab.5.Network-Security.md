# Network Security Concepts

Abbiamo già creato una separazione, una segmentazione o partizionamento delle risorse di sistema dal punto di vista
fisico e logico ma vogliamo segregare.

$$
segmentazione \Rightarrow segregazione
$$

Dobbiamo capire quali sono le best practice per la sicurezza.

## Defense in-depth

Applico delle difese a livello di rete e di host, oppure creo diverse reti protette "a cipolla".

Nel caso delle VLAN abbiamo impiegato dei meccanismi e delle policy per ottenere separazione a livello 2.
Se vogliamo lavorare ai livelli superiori dobbiamo impiegare i firewall, ad esempio agire a livello di routing. L3 e L4.

Quando andiamo a livello applicativo parliamo comunque di firewall ma non dobbiamo dare per scontato alcune tecniche:
_deep packet inspection_ (ispezione in profondità del pacchetto) cioè il firewall ha "cognizione" di come funziona il
protocollo.

Immaginiamo di avere un sistema di email, se il sistema può fare la scansione degli allegati. Anche qua abbiamo un sistema
di deep packet inspection esaminando il payload. Questi nomi identificano un ventaglio di tecniche.

Un _Application Layer Firewall_ implementa completamente un protocollo applicativo e riesce perfettamente a capire cosa
sta succedendo nella rete.

Un'altra tecnica di "sicurezza", che in alcuni casi serve per motivi tecnici, è il NATting. È una tecnica di segmentazione
perché separa lo spazio di indirizzamento delle reti.

## Tipi di approcci

Le policy di sicurezza sono concettualmente definite in Access Control Lists (ACL). Specifico cosa è consentito e cosa
non è consentito.

Ci sono due approcci speculari che sono _implicit negation_ o _implicit allow_, dove il primo preferisce la sicurezza
rispetto all'usabilità mentre il secondo il contrario.

Nel NATting c'è una regola di implicit allow per uscire sulla rete, mentre il contrario è in contesto di implicit negation.

Un firewall può essere implementato sia in hardware che in software, e installati sulla macchina dell'utente oppure su
un router (quindi in rete). Un firewall hardware esegue gli algoritmi di analisi su hardware dedicato implicitamente:
il dispositivo è prodotto per essere efficientemente.

Il firewall è stateless se ogni pacchetto è analizzato indipendentemente, mentre un firewall è stateful se il firewall
deve conservare uno stato per analizzare i pacchetti. Ad esempio voglio consentire pacchetti TCP in ingresso solo per
quegli host che prima inizializzano una connessione.

Esistono logiche stateful potenzialmente più complesse. HTTP è stateless ma può esserci una logica stateful legata
all'uso di cookie.

## Firewall di rete

A volte definiamo i firewall con altri termini.

### Screening router

Quando un border router utilizza regole di firewall viene detto _screening router_.
Di solito opera soltanto a livello 3 e 4 e compie una prima scrematura dei pacchetti.

### Stealth firewall

Analisi L3 e L4, logiche semplici e solitamente stateless. Sono configurati per non essere rilevabili all'interno della
rete. Possiamo considerare questi dispositivi alla stregua di dispositivi L2, perché anche se sono simili a dei router
sono modificati per essere non facilmente rilevabili, con tecniche del tipo:

- non si decrementa il TTL

- non si risponde a richieste IP

Lo blindiamo il più possibile perché è il primo layer di difesa. Non vogliamo lasciare all'attaccante modo di reperire
informazioni sul nostro router. Questo tipo di scelta può avere un impatto sulla complessità della rete.

Il fatto che il firewall si comporti così potrebbe essere legato a "modifiche" che devo mantenere in casa.

### Proxy firewall

Supportano completamente dei protocolli applicativi, deve a volte eseguirlo al posto nostro. Un proxy intercetta le
nostre comunicazioni e realizza le connessioni come se fosse il client stesso.

Ristabilire le connessioni è obbligatorio se usiamo connessioni cifrate, perché i dati sono cifrati a livello di
payload sia alla rete che al firewall.

## Defense-in-depth

Indica che usiamo più tecniche, dispositivi contemporaneamente e a diversi livelli della rete.

### Bastion host

È un altro termine per identificare un proxy firewall. Usiamo il termine quando il proxy firewall opera obbligando che
un host interno debba passare per un proxy interno per comunicare all'esterno della rete.

Lo screening router blocca il traffico, tranne quello del bastian host che fa deep packet inspection.

L'approcio è defense-in-depth perché i dispositivi operano in modo coordinato.

### DMZ

_Demilitarized zone_: identifica il concetto di una parte di rete più esposta all'esterno. La zona è demilitarizzata
perché è sottoposta a meno controlli.

Chi governa e configura la DMZ si assume abbia più consapevolezza di quello che succede.

La DMZ ha fiducia completa, mentre la rete interna ha una fiducia più limitata dal punto di vista della sicurezza.

## Approach

Un approccio famoso è avere un external screening router con regole lasche, mentre un internal screening router per
applicare regole aggiuntive ai miei utenti (sempre a L3 e L4), ma ho anche un bastion host ovvero un proxy firewall per
uscire sulla rete in sicurezza.

Questo approccio è fondamentale per avere delle buone performance. Più saliamo di livello più risorse sono richieste,
per la complessità dei protocolli.

## iptables

Si può usare per fare regole di NATting, ma il suo funzionamento è quello di firewall.

Per realizzare dei firewall agisco nella tabella _filter_ che ha accesso a tre catene principali: input, output e forward.
Hanno nomi diversi da quelle del NAT ma lavorano su aspetti simili.

- input: riguarda tutti quanti i pacchetti prima che arrivino a un processo locale (non è la catena di pre-routing);

- output:  definisce tutti pacchetti generati da un processo locale;

- forward: identifica pacchetti che escono dalla logica di routing verso l'interfaccia di uscita, si usa per i router.

Se configuro un firewall a livello di host non ha senso lavorare a livello di forward. Input e output servono a livello
di host.

### Policy

Si specifica con `-P` per una catena. Di solito è ACCEPT, però potrei voler impostare DROP.

```sh
iptables -P INPUT ACCEPT
iptables -P INPUT DROP
```

Se uso UDP, una policy DROP in INPUT permette comunque di mandare dei pacchetti ma non mi permette di riceverli. Con TCP
non riesco nemmeno a instaurare una connessione.

L'opzione `-m state --state` è legata alla gestione dello stato. In generale indica un modulo di iptables per fare
analisi di pacchetti un po' più avanzata. Il modulo state indica di eseguire analisi stateful.

`-j` indica l'azione da eseguire su match della regola.

```sh
iptables -A INPUT -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
```

questo accetta connessioni TCP in entrata su porta 80.

Voglio che il server non possa creare connessioni nuove, ma solo rispondere a quelle accettate.

```sh
iptables -A OUTPUT -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

notare come abbiamo rimosso NEW e cambiato dport in sportà.

## Persistence

Possiamo salvare le regole di iptables sia tramite le direttive post-up/post-down in network interfaces oppureusare
`iptables-save` e `iptables-restore`, salvando tutto in `/etc/iptables/rules.v4`.

I due comandi operano su stdout e stdin, semplicemente li salviamo all'interno della directory `/etc/iptables`. I file
di configurazione vengono letti a tempo di boot.

Per creare diversi checkpoint possiamo redirezionare l'output su file in posizioni non particolari.

```sh
iptables-restore < /etc/iptables/rules.v4
```

Per visualizzare cosa sta succedendo:

```sh
iptables -nvL
```

Possiamo avere tra le possibilità di state: NEW, ESTABLISHED e RELATED.
