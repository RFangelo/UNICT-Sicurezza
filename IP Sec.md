La rete internet nasce trascurando l'aspetto della sicurezza, successivamente è nata l'esigenza di fornire agli utenti e alle applicazioni determinate proprietà quali Autenticazione, Segretezza, Integrità... Un modo per garantire queste proprietà è quella di andare a implementare un protocollo di sicurezza al livello di indirizzamento.

IP Sec offre la possibilità di rendere sicure le comunicazioni locali, tra reti locali separate e su internet.

### Benefici di IP Sec
I vantaggi che si hanno dall'utilizzo di IP Sec sono i seguenti:
- Utilizzo di Firewall/Router perimetrali: Possibilità di implementare barriere di sicurezza perimetrali ad una rete locale in modo trasparente all'utente finale, l'utilizzo dei firewall permette di impostare regole di routing e analisi dei pacchetti in ingresso.
- Integrità dei dati
- Autenticazione
- Rifiuto di pacchetti replicati
- Segretezza


### Security Associations
Un concetto chiave nei meccanismi di Autenticazione e Segretezza IP è quella di **Security Association** (SA), essa è una relazione mono-direzionale fra un mittente ed un destinatario che vogliono comunicare in sicurezza, ogni SA è caratterizzata dai suoi parametri di sicurezza necessari per l'implementazione dei protocolli IP Sec. I parametri legati ad una SA sono diversi, i più importanti sono:
- **SPI** (Secure Parameter Index)
	Stringa di bit identificativa per la SA, questo id è trasportato negli header e permette in ricezione di elaborare correttamente il pacchetto in base alle impostazioni e parametri specificati.
- **Indirizzo IP di destinazione**
	Attualmente sono supportati solo Indirizzi unicast
- **Identificatore del protocollo di sicurezza** da utilizzare e la modalità di utilizzo
	Identificazione del protocollo da utilizzare, se ESP o AH, inoltre questi possono essere utilizzati in tunnel o transport mode.

### Modalità di comunicazione
In IP Sec vi sono principalmente due modi per comunicare, ***transport*** e ***tunnel***, i due protocolli AH e ESP supportano entrambe le modalità.

#### Transport
In questa modalità di trasporto viene protetto solo il payload, infatti l'intestazione IP del pacchetto rimane la stessa, viene utilizzato il payload IP per aggiungere un Header IP Sec e di seguito il payload effettivo protetto.
Non modificando l'intestazione IP il pacchetto viene indirizzato sulla rete come definito dal livello superiore.

Questa modalità viene utilizzata per le comunicazioni End To End.
#### Tunnel
Nella modalità tunnel il pacchetto IP viene letteralmente preso ed impacchettato all'interno di un nuovo pacchetto IP con un Header IP Sec.
In questo caso l' indirizzamento non è quello richiesto dal livello superiore, il pacchetto verrà trasportato fino a destinazione, successivamente spacchettato e avverrà l'indirizzamento verso la destinazione originale. Ovviamente dopo che il pacchetto IP originale verrà spacchettato non sarà più protetto (Questo solitamente non è un problema perché si tratta di reti locali protette).

Questa modalità di comunicazione è solitamente usata per comunicazione Site To Site tra due Router che vogliono implementare una rotta statica sicura.

### Servizio Anti-Reply

Ogni pacchetto IP Sec contiene un Sequence Number per mitigare attacchi a replica.

**Lato Mittente**:
Quando viene stabilita una nuova connessione viene inizializzato un contatore a 0, ogni volta che deve essere inviato un messaggio questo contatore viene incrementato e aggiunto all' header IP Sec. Il campo Sequence Number nell'header ha una dimensione di 32 bit, quindi il suo valore massimo sarà 2^32 - 1, quando viene raggiunto il valore massimo dovrà essere negoziata una nuova SA.

**Lato Ricevitore**:
In ricezione viene definita una finestra di ricezione:
- Se si riceve un SN all'interno della finestra e questo è valido si marca quel SN come ricevuto.
- Se si riceve un SN a destra della finestra di ricezione e questo è valido si fa avanzare la finestra fino a quel SN che diventerà la testa della finestra
- Se si riceve un SN contrassegnato come ricevuto lo si scarta
- Se si riceve un SN a sinistra della finestra lo si scarta
- Se in generale un SN non è valido lo si scarta

![[Pasted image 20241208190044.png]]


### Authentication Header (AH) 
AH fornisce il supporto per l'integrità e l'autenticazione dei pacchetti IP ma non fornisce la confidenzialità.
L'autenticazione si basa su un codice MAC, di conseguenza entrambe le parti devono condividere la stessa chiave segreta.

I campi dell'intestazione sono:
- **Next Header**: Identifica il tipo di intestazione, ovvero il pacchetto che si deve infilare all'interno del payload del pacchetto IP standard
- **Payload Length** Dimensione dell header AH espressa in Word da 32 bit - 2 (supponendo auth data da 96bit/3 word, payload length varrà 4)
- **RESERVED** per implementazioni future
- **Security Parameter Index** legata al SA
- **Sequence Number** per difendersi da attacchi Reply
- **Authentication Data** MAC che permette di implementare il servizio di *integrity check value*

	![[Pasted image 20241208183209.png]]

Il campo Authentication Data è un codice MAC del tipo HMAC-MD5-96 o HMAC-SHA-1-96, gli input della funzione di Hashing sono:
- Se in mod Transport IP header + AH header + Payload del livello superiore
- Se in mod Tunnel AH header + Intero pacchetto IP inglobato
### Encapsulation Security Payload (ESP) 

Questo protocollo fornisce principalmente confidenzialità, ma può anche fornire autenticazione e integrità.

I campi dell'header ESP sono:
- **Security Parameter Index**
- **Sequence Number**
- **Payload Data** In ESP il payload insieme al padding hanno una dimensione variabile, l'utilizzo del padding permette di ottenere una dimensione del blocco da cifrare compatibile con l'algoritmo di cifratura, inoltre permette di celare informazioni relative alla dimensione effettiva del dato e rendere più difficile l'analisi del traffico.
- **Padding** Ha dimensione variabile, la sua dimensione dipende strettamente dal payload, inoltre deve terminare con il secondo ottetto di una parola di 4 byte (vedi figura). Può avere una dimensione massima di 255 byte, in questo modo è possibile celare la dimensione effettiva dei pacchetti ed impedire l’analisi del traffico. Un ulteriore utilità data dal padding è quella di rendere il plaintext di una determinata dimensione per essere dato in ingresso ad uno schema di cifratura.
- **Pad Length** Specifica la dimensione del padding.
- **Next Header** Specifica il tipo di contenuto in Payload Data.
- **Authentication Data** Campo opzionale e di dimensione variabile (deve essere comunque una dimensione multipla di 32), contiene un integrity check value calcolato su tutti i campi meno Authentication Data.


![[Pasted image 20250104163312.png]]
### Gestione delle chiavi in IP Sec
In IP Sec due applicazioni che vogliono comunicare hanno bisogno di due Security Association, ogni SA ha bisogno di una chiave di cifratura ed una di Autenticazione, pertanto:
- 2 chiavi per la SA (A -> B) / Key-Cipher & Key-Auth
- 2 chiavi per la SA (B -> A) / Key-Cipher & Key-Auth

La gestione di queste chiavi è definita dallo Standard IKE (Internet Key Exchange), sono molti i protocolli, quelli più di rilevanza sono sicuramente:
- Oakley: Protocollo di scambio delle chiavi basato su Diffie-Hellman
- ISAKMP: Protocollo di gestione delle chiavi, Oakley si basa su ISAKMP ad un livello più basso


#### Oakley
Oakley è un miglioramento del classico schema Diffie-Hellman per lo scambio di chiavi, con l'obbiettivo di mitigare attacchi quali il clogging o man-in-the-middle. Oakley introduce le seguenti principali novità:
- **Meccanismo di cookies** per contrastare il clogging:
	L'attacco a clogging è un tipo di DoS, l'idea è quella di inondare un server di richieste ed esaurire le sue risorse computazionali.
	Il server prima di impegnarsi a livello computazionale invia un cookie, il client conferma la richiesta inviando il valore ricevuto, se era un attaccante che ha falsificato L'IP di origine non avrà modo di ricevere il messaggio contenente il valore e non potrà confermarsi.
- **Utilizzo di gruppi** per la definizione dei parametri globali di Diffie-Hellman
- **Utilizzo di Nonce** per mitigare gli attacchi a replay
- **Autenticazione** dello scambio per prevenire attacchi Man-In-The-Middle:
	Oakley integra uno strato di Autenticazione a Diffie-Hellman per autenticare A e B, può essere implementato attraverso l'utilizzo di certificati o coppie di chiavi asimmetriche (le varie modalità di autenticazione sono fornite da IKE).


#### ISAKMP
ISAKMP (Internet Security Association and Key Management Protocol) è un protocollo di supporto che definisce il framework per la gestione, la negoziazione e la manutenzione delle Security Association (SA), quindi questo protocollo stabilisce come due entita devono comunicare tra di loro per:
- Attivare una SA
- Negoziare i parametri di connessione e sicurezza
- Modificare parametri esistenti
- Cancellare o terminare una SA

L'header ISAKMP contiene una serie di campi:
- **Initiator Cookie**
- **Responder Cookie**
- **Next Payload** indica quale payload segue immediatamente nel messaggio
- **Major Version**
- **Minor Version**
- **Exchange Type** indica il tipo di operazione/ scambio avviene tra i due peer
- **Flags**
- **Message ID** rappresenta un codice univoco per il messaggio
- **Lenght** lunghezza totale del messaggio, Header + Payload


![[Pasted image 20250104181548.png]]
