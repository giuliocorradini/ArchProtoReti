# NAT

## iptables

Inseriamo delle regole in catene, raggruppate attraverso tabelle mappate in diversi punti dell'analisi dei pacchetti.
Nei nodi di hook possiamo applicare le regole.

Identifichiamo tra tutte le catene due in particolare: post-routing e pre-routing.
