
Il Word Wide Web è ormai un elemento fondamentale della vita comune, l'argomento della sicurezza sul web è diventata sempre più seria, vi sono infatti diverse minacce. Si deve poter garantire le seguenti proprietà:
- **Integrità** dei dati, protezione delle modifiche da terzi e malintenzionati
- **Confidenzialità**, protezione da intercettazioni in rete, furto di dati
- **Denial of Service**, terminazione dei processi a causa di intasamento della macchina con una grande quantità di richieste
- **Autenticazione**, simulazione di utenti illegittimi e falsificazione dei dati.

L'idea alla base di SSL/TLS e' di andare ad implementare dei servizi di sicurezza al di sopra del TCP, l'accesso a tali servizi esposti dovrà essere totalmente trasparente al livello applicativo, SSL/TLS chiamerà a sua volta le primitive TCP.

### SSL (Secure Socket Layer)
L'architettura è basata su un protocollo base al di sopra del TCP e tre protocolli di gestione al livello applicativo, al livello applicativo troviamo anche HTTP che sopra SSL diventa HTTPS. I protocolli in questione sono:

- ***SSL Handshake Protocol***: Si occupa di gestire la fase di negoziazione tra le parti
- ***SSL Change Cipher Spec Protocol***: Notifica il cambio di stato della sessione alla controparte, cambio di sessione significa cambio dei parametri di sicurezza.
- ***SSL Alert Protocol***: Notifica di avvisi riguardo alla sessione.
- ***SSL Record Protocol***: Implementa la comunicazione sicura.


![[Pasted image 20241207173653.png]]


Due concetti fondamentali in SSL sono Sessione e Connessioni:
- **Sessione**: Una sessione si instaura tra client e server, queste sono create dal protocollo di handshake, a livello di sessione vengono concordati dei parametri di sicurezza che saranno comuni ad n connessioni riferiti alla sessione, questo permette di diminuire l'overhead relativo all'handshake. I parametri fondamentali della sessione sono:
	- Algoritmi crittografici negoziati: RSA-AES-SHA256
	- Chiave master dalla quale generare le chiavi di sessione
	- Certificati
- **Connessione**: Una connessione è una forma di trasporto, ogni sessione ha dei parametri univoci come le chiavi di cifratura e checksum crittografico. I parametri fondamentali sono:
	- Chiave di crittografia del client.
	- Chiave di crittografia del server.
	- Chiavi per HMAC.
	- Sequence number

#### Protocollo record SSL
Questo protocollo fornisce i servizi di sicurezza di base per i protocolli di livello superiore:
- **Riservatezza**: Il protocollo di Handshake fornisce due chiavi simmetriche condivise, una per ogni direzione ( client -> server, server -> client). Tramite queste chiavi avviene la cifratura.
- **Integrità**: Il protocollo di handshake fornisce pure due chiavi simmetriche per la generazione di MAC per la verifica dell'integrità.

Dato un messaggio da trasmettere i passi operativi sono frammentazione in più blocchi da 16 KB, compressione (opzionale), aggiunta del MAC, cifratura e aggiunta header SSL.
La compressione deve fornire un blocco di lunghezza inferiore o al peggio non maggiore di 1024 byte.

![[Pasted image 20241207180834.png]]

![[Pasted image 20241207181234.png]]

**Content type, Maj version, Min version = 1 Byte**
**Compressed Length = 2 Byte**

#### SSL Change Spec Protocol
Questo protocollo è uno dei tre che utilizza Record SSL, ed è il più semplice.
Si tratta di un singolo messaggio con un singolo byte con valore 1, viene utilizzato per notificare un cambio di stato della connessione in uso, vi saranno infatti dei nuovi parametri negoziati dal' handshake protocol per quella connessione. I messaggi di questo protocollo sono protetti da confidenzialità e integrità in quanto fanno utilizzo di SSL Record Protocol.

![[Pasted image 20241207182012.png]]


#### SSL Alert Protocol
Questo protocollo viene utilizzato per trasmettere avvisi (al peer) relativi alla connessione SSL. Ogni messaggio di questo protocollo è formato da due byte, uno indica il **livello** e l'altro specifica un codice relativo al tipo di errore o **avviso**.
Il primo byte può assumere i due valori di **warning** o **fatal**:
- Se **fatal** la connessione viene chiusa immediatamente, mentre le altre connessioni della sessione possono continuare, non potranno essere create nuove connessioni per quella sessione e si dovrà effettuare un nuovo Handshake.
Anche i messaggi di questo protocollo sono cifrati e compressi.

![[Pasted image 20241207183115.png]]

#### SSL Handshake Protocol
Si occupa di eseguire la procedura di Handshake tra le entità peer, i messaggi scambiati dal client e dal server contengono 3 campi:
- **Type** (1 Byte): Codice per specificare il tipo di messaggio tra i 10 possibili.
- **Length** (3 Byte): Lunghezza del messaggio espressa in byte.
- **Content** (>= 0 Byte): Parametri specifici del tipo di messaggio.

![[Pasted image 20241208115321.png]]

Questo protocollo consente al server e al client di autenticarsi a vicenda e negoziare chiavi e algoritmi crittografici.

Questo protocollo viene utilizzato prima della trasmissione di tutti i dati del livello applicativo. In generale l'handshake si svolge in 4 fasi:
- **Fase 1** Stabilire Funzionalità di sicurezza: In questa fase il client invia in chiaro al server un messaggio di  **client_hello** dove presenta:
	- RANDOM: Prevenzione al reply attack e aumento entropia
	- ID SESSION: Valore diverso da 0 significa che si tratta di una sessione esistente, 0 significa nuova sessione e il server risponde con un nuovo ID.
	- CIPHER SUITE
	- COMPRESSION METHOD
	Il Server risponde con un messaggio di server_hello, contenente gli stessi parametri e gli algoritmi di cifratura - compressione selezionati tra quelli proposti dal client. 

	![[Pasted image 20241208122153.png]]

- **Fase 2** Autenticazione server: 
	- In questa fase avviene l'**autenticazione** del server tramite certificato X.509 (opzionale se si usa Diffie-Hellman - ci sono possibili setting che permettono l'anonimato su SSL/TLS, ovviamente si è vulnerabili a man in the middle)
	- Se si usa **Diffie-Hellman** viene mandato un messaggio di Server Key Exchange, se vengono scambiati i certificati non è richiesto, si utilizzera **RSA**
	- Se necessario il certificato del client viene mandato un messaggio di **certificate_request** al client
	- Viene conclusa la fase di autenticazione del server con l'invio di un messaggio **server_done**
	![[Pasted image 20241208130822.png]]
	
- **Fase 3** Autenticazione client e scambio chiavi: 
	- In questa fase avviene l'**autenticazione** del client tramite certificato X.509 se richiesto (Se precedentemente richiesto)
	- Se si usa **Diffie-Hellman** viene utilizzato il messaggio client_key_exchange per comunicare la chiave publica, in altri casi come RSA, questo messaggio conterrà la pre-master secret cifrata con la chiave publica del server
	- Viene conclusa la fase di autenticazione del client con l'invio di un messaggio **certificate_verify**, in questo modo il server può testare la validità del certificato del client
	![[Pasted image 20241208132433.png]]

- **Fase 4** Fine: Il client ed in seguito il server mandano due messaggi.
	- **change_cipher_spec** comunica che i prossimi messaggi faranno utilizzo della cipher suite appena negoziata
	- **finished** rappresenta il primo messaggio cifrato e conferma la validità dell'handshake
	
	![[Pasted image 20241208132530.png]]



	 ##### Calcoli Crittografici e key
	 Il flusso di generazione delle chiavi / secrets a grandi linee è il seguente:
	 - Nella prima fase vengono scambiati due random client/server
	 - Nella seconda fase e terza viene generato una pre-master key, utilizzando DiffieHellman o RSA (in questo caso la pre-master viene scelta dal client e inviata al server cifrata con la sua chiave publica)
	 - Viene generata la Master-Key attraverso una funzione di generazione, questa prende in ingresso pre-master ed i due valori random della prima fase.
	 - Per ogni nuova connessione le chiavi simmetriche vengono generate a partire dal Master Secret e dai nuovi random client e server.
	

### TLS

TLS è il successore di SSL, offre miglioramenti in termini di sicurezza, prestazioni e flessibilità.

##### Maggiore Sicurezza
Utilizzo di HMAC rispetto a MAC, AES e altri rispetto a 3DES

##### Migliore Efficienza
Introduzione del session resumption con i session ticket invece di session id.
I session ticket contengono tutti i parametri della connessione firmati, questo aumenta sia dal punto di vista della sicurezza, ma anche dal punto di vista dell efficienza... Prima infatti il server doveva tenere una cache con tutti questi parametri di connessione per ogni sessione, adesso il client si presenta con questo ticket contenente tutte le informazioni necessarie.