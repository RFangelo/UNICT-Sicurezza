Indipendentemente dal livello di sicurezza che si riesce a garantire utilizzando avanzati algoritmi crittografici, siano essi simmetrici o asimmetrici, per mantenere un buon livello di sicurezza bisogna gestire correttamente la generazione e distribuzione delle chiavi.

La Distribuzione delle chiavi è una fase critica: bisogna garantire che le chiavi siano scambiate tra le parti senza essere intercettate da terzi (Man In The Middle). Alcuni metodi comuni sono:
- Consegna Diretta
- Terza parte fidata: Un ente centrale genera e distribuisce le chiavi a entrambe le parti
- Crittografia delle chiavi: Se le due parti hanno già comunicato in passato è possibile utilizzare un canale sicuro per la trasmissione di una nuova chiave

Garantire la sicurezza nella distribuzione delle chiavi è complesso, sopratutto in reti di grandi dimensioni. Ecco alcuni punti fondamentali:
- Tempo di vita delle chiavi: Le chiavi di sessione dovrebbero avere un ciclo di vita breve, devono essere aggiornate regolarmente per limitare i rischi
- Sistemi centralizzati: In reti molto grandi, è necessario un sistema gerarchico di centri di distribuzione delle chiavi KDC.

## Standard X.509
Lo standard X.509 è divenuto lo schema universale per strutturare i certificati di chiave publica. Moltissime delle applicazioni di sicurezza di rete tra cui IPSec, SSL, S/MIME utilizza i certificati X.509.

Ciascun certificato contiene la chiave pubblica di un utente ed è firmato con la chiave privata di un autorità di certificazione fidata.
X.509 è uno standard fortemente gerarchico e si basa sull'uso della crittografia a chiave pubblica e delle firme digitali. Lo standard non prevede l'utilizzo di algoritmi crittografici specifici ma ne consiglia alcuni, come RSA.

![[Pasted image 20241215171705.png]]
Questa figura mostra il processo di generazione di un certificato, un documento viene quindi creato (contiene i dati dell'utente e la sua chiave pubblica), viene generato un Hash e questo viene firmato con la chiave privata della Certification Authority.

Un certificato ha diversi campi:
- **Versione**: In base alla versione (1//2//3) cambiano i campi presenti nel certificato e la sua lunghezza
- **Certificate Serial Number**: Ogni certificato emesso dalla CA ha un Serial Number univoco
- **Signature Algorithm Identifier**: Algoritmo e parametri utilizzati per la firma 
- **Issuer Name**: Nome del CA che ha emesso il certificato
- **Period of validity**: Questo campo è formato da due datetime, uno di inizio ed uno di fine
- **Subject Name**: Nome del proprietario del certificato (per chi viene creato)
- **Subject Public Key Info**: Chiave publica, algoritmo utilizzato, parametri
- **Issuer Unique Identifier**: Stringa identificativa della CA, il nome può cambiare questo no
- **Subject Unique Identifier**: Stringa identificativa del proprietario del certificato
- **Extensions**: Campi opzionali che permettono di aggiungere ulteriori informazioni utili per descrivere e limitare l'uso del certificato
- **Signature**: Firma dell'ente di certificazione

![[Pasted image 20241215175107.png]]
La notazione per un certificato di A firmato da CA è:

![[Pasted image 20241215180853.png]]

Lo standard X.509 è fortemente gerarchico, nel caso più semplice A vuole ottenere il certificato di B, quest'ultimo è sotto la gestione della stessa CA di A, quello che avviene è che semplicemente A può verificare il certificato essendo a conoscenza della chiave pubblica della propria CA.
In uno scenario reale abbiamo diverse CA con operazioni di trust che li collegano in una struttura ad albero


![[Pasted image 20241215180940.png]]


Supponiamo adesso che A vuole ottenere il certificato di B, ma non ha modo di verificarne la validità poiché non è a conoscenza della chiave pubblica di Z.
Quindi prima di verificare il certificato di B, A deve ottenere la chiave pubblica di Z:
- A ottiene $Z \ll B \gg$
- A ottiene da Z  $Y \ll Z \gg$
- A ottiene da Y $V \ll Y \gg$
- Una volta ottenuto il certificato di V, A si rende conto che V è una CA comune nella lista di certificati di entrambi
- A ottiene $X \ll W \gg$
- A ottiene $W \ll V \gg$
Adesso è possibile verificare tutti i certificati e ottenere la chiave pubblica di B:

$X \ll W \gg$ | $W \ll V \gg$ | $V \ll Y \gg$ | $Y \ll Z \gg$ | $Z \ll B \gg$

In questo caso abbiamo fatto utilizzo di due tipi di certificati:
- Certificati in avanti, qualsiasi certificato emesso da una CA per un nodo più esterno alla gerarchia (Es. $Z \ll B \gg$ )
- Certificati all'indietro, qualsiasi certificato che permette di risalire la gerarchia, (Es. $X \ll W \gg$)


### Revocazione 
Uno dei campi del certificato X.509 è il periodo di validità oltre il quale non potrà essere utilizzato. Possono nascere diverse situazioni in cui è necessario rendere un certificato non più utilizzabile, ad esempio se viene violata la chiave privata dell'utente o la chiave della CA utilizzata per la firma. Nasce quindi l'esigenza di implementare un metodo di revoca dei certificati già emessi, nelle implementazioni più comuni questo viene effettuato attraverso la definizione di liste di certificati revocati, detti anche Certificate Revocation List (**CRL**), queste vengono prodotte periodicamente dalla CA e sono una lista di tutti i SN associati ai certificati ormai revocati.
Ovviamente una CRL deve essere firmata dalla CA, per mantenere le proprietà di integrità e autenticità come se fosse un certificato. Sarà compito dei vari client come i browser interrogare periodicamente la CA per ottenere la CRL ed eventualmente dismettere i certificati non più validi.

Un altro approccio è quello di Online Certificate Status Protocol (**OCSP**), in questo caso non vi è una generazione una tantum di una lista, essa viene generata ogni volta che viene effettuata una richiesta, quando si chiede lo stato di un certificato la CA risponde con uno dei seguenti tre stati:
- "**good**" IL certificato è valido
- "**revoked**" Il certificato è revocato
- "**unknown**" Non si hanno informazioni sul certificato
La risposta è firmata digitalmente dalla CA.
Perchè OCSP possa essere utilizzato è necessario che il server implementi il protocollo OCSP.

