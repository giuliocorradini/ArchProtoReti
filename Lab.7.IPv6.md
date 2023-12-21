# IPv6 Linux

Per configurare un'interfaccia con IPv6 usiamo la direttiva

```
auto eth0
iface eth0 inet6 auto
```

abbiamo la nuova keyword `inet6`.

Per assegnare un indirizzo staticamente usiamo:

```
iface eth0 inet6 static
    address 2001:db9::2/56
```

Per usare IPv6 con netcat, dobbiamo usare `ncat` e non `nc`.
