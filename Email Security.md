La posta elettronica è tra le applicazioni più utilizzate, la sicurezza richiesta nel suo ambito è intesa in:
- Confidenzialità
- Autenticazione
- Integrità dei messaggi
- Non ripudio dell'origine

Tra i protocolli più utilizzati abbiamo sicuramente PGP e S/MIME.

# Pretty Good Privacy
PGP è un servizio che fornisce le proprietà di segretezza e autenticazione, può essere utilizzato per la posta elettronica e per la memorizzazione di file.
PGP fa utilizzo di un pacchetto di algoritmi sicuri come RSA, DSS, 3DES, Diffie-Hellman/ElGamal, SHA-1.
PGP fornisce i seguenti sevizi:
- Autenticazione
- Segretezza
- Compressione
- Compatibilità con la posta elettronica
- Segmentazione

## PGP Operation Authentication
A livello operativo funziona nel seguente modo:
- Il mittente crea un messaggio
- Crea l'HASH SHA-1 del messaggio
- Viene creata la firma cifrando l'HASH con la chiave privata del mittente e questa viene posizionata prima del messaggio
- In ricezione viene verificata la firma decifrando l'hash e creandone uno nuovo a partire dal messaggio, se gli hash coincidono la proprietà di autenticazione è soddisfatta

## PGP Operation Confidentiality
Un altro sevizio di base offerto da PGP  è la riservatezza, viene fornita mediante la crittografia dei messaggi da trasmettere, vengono utilizzati solitamente i seguenti schemi di cifratura:
- 3DES
- IDEA
- CAST-128
Tutti con modalità operativa Cipher Feedback (CFB), la chiave di sessione utilizzata per la cifratura del plaintext viene generata casualmente ed inviata insieme al ciphertext, ovviamente la chiave non può essere inviata in chiaro e pertanto viene cifrata con la chiave RSA pubblica del destinatario.
I passaggi utilizzati in questo processo sono:
- Il mittente genera un messaggio e un numero casuale di 128 bit che verrà utilizzata come chiave di sessione unicamente per quel messaggio
- Il messaggio viene crittografato con uno degli algoritmi compatibili
- La chiave di sessione viene crittografata con la chiave pubblica del destinatario
- In ricezione viene decifrata la chiave di sessione con la chiave privata del destinatario
- Viene decifrato il messaggio utilizzando la chiave di sessione

Le versioni più recenti supportano l'utilizzo di ElGamal per lo scambio delle chiavi simmetriche di sessione.

## PGP Operation Confidentiality & Authentication
Risulta possibile utilizzare i servizi di riservatezza e autenticazione insieme nel seguente modo:
- Il mittente firma il messaggio con la propria chiave privata
- Il mittente cifra il messaggio e la firma con la chiave di sessione generata 
- Il mittente cifra la chiave di sessione con la chiave publica del destinatario

## PGP Compression
DI default PGP comprime il messaggio dopo aver applicato la firma, ma prima della crittografia, questo viene fatto per vari motivi ma principalmente per poter verificare la firma in qualunque momento; Si immagini il caso in cui prima venga applicata la compressione, in ricezione per poter verificare la firma bisognerebbe mantenere una copia del messaggio compresso.
Inoltre la crittografia di un messaggio compresso introduce entropia nel plaintext e rende ancora più difficile l'analisi crittografica.
L'algoritmo di compressione utilizzato è ZIP.

## PGP Email Compatibility
PGP suddivide automaticamente un messaggio troppo grande per una singola mail in segmenti abbastanza piccoli da poter essere inviati.

Un ulteriore problema risolto da PGP è la risoluzione del problema di incompatibilità con i dati binari nei protocolli di posta, infatti questi accettano valori codificabili in ASCII, inoltre un testo ASCII cifrato risulta in un codice binario. PGP risolve questo problema con la conversione radix-64, quello che avviene è una mappatura di 3 ottetti di dati binari in quattro caratteri ASCII, questo comporta un espansione dei dati da inviare del 33%, la compressione riesce comunque a compensare questo overhead.  

## Gestione chiavi in PGP
Una delle funzionalità di PGP è che ogni utente puo avere più coppie di chiavi asimmetriche associate, quindi non c'è una relazione univoca tra utente e chiave da utilizzare per la decifratura. Per la gestione delle chiavi PGP assegna ad ogni chiave un KeyID, sarà uguale ai 64 bit meno significativi della chiave corrispondente. Il keyID viaggia insieme al messaggio per permettere al mittente di selezionare la chiave da utilizzare.
Le chiavi e gli ID devono essere opportunamente archiviati per un uso efficiente, questo viene fatto attraverso l'utilizzo di due strutture dati:
- Private Key Ring
	Contiene le coppie di chiavi pubbliche e private dell'utente, le chiavi sono conservate crittografate con un hash prodotto dall'inserimento di una passphrase.
- Public Key Ring
	Contiene tutte le chiavi pubbliche degli altri utenti indicizzabili per ID utente o per Key ID.

## Rete di Fiducia
Invece di basarsi esclusivamente su un’autorità di certificazione centralizzata (CA), come avviene in altri sistemi, PGP permette ad ogni utente di agire esso stesso come una sorta di “CA” informale. Ciò significa che ciascun utente può “certificare” la chiave pubblica di un altro utente firmandola con la propria chiave privata. In questo modo, la fiducia si può diffondere lungo la rete: se ti fidi della chiave di un amico (perché la conosci personalmente o perché hai ottenuto la sua chiave in modo sicuro), e il tuo amico ha firmato la chiave di una terza persona, allora puoi ereditare un certo grado di fiducia anche nella chiave di questa terza persona, senza averla verificata direttamente.

PGP utilizza dei valori interni per gestire e rappresentare il livello di fiducia:

- **OWNERTRUST:** rappresenta quanto ti fidi di un determinato utente nel ruolo di certificatore di altre chiavi (ossia quanto ritieni affidabile il suo giudizio quando firma chiavi altrui). 
	È una valutazione che assegni tu come utente. Ad esempio:
    - Massima fiducia: completamente certo che quell'utente controlli bene l’identità dei proprietari delle chiavi che firma.
    - Fiducia parziale: sei solo in parte sicuro della bontà delle sue certificazioni.
    - Nessuna fiducia: non ti fidi affatto della sua capacità di convalidare chiavi.
- **Key legitimacy (legittimità della chiave):**
	Indica quanto PGP considera affidabile la corrispondenza tra un ID utente e una chiave pubblica specifica. È un attributo derivato dalle firme e dai valori di OWNERTRUST. Più una chiave è firmata da utenti di cui ti fidi (magari ad alto OWNERTRUST), più la sua legittimità sarà elevata. Questa legittimità è un valore che PGP calcola automaticamente, in base a come le chiavi si firmano tra loro e ai livelli di trust da te assegnati.


# S/MIME