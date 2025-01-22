Il problema dell'autenticazione Utente è uno degli aspetti più delicati per un sistema di sicurezza.

**User Authentication**: È il processo di verifica di un identità da parte di un entità, tale processo si divide in due step:
- **Identificazione** presentando un identificativo
- **Verifica** presentazione o generazione di informazioni di autenticazione che corroborano il legame tra entità e identificatore
Questo viene solitamente fatto attraverso le credenziali Username e Password

In generale si possono avere due scenari:
- **Mutual Authentication** i due peer si autenticano entrambi
- **One-Way Authentication** solo una delle due parti si autentica (es. login siti web)

## Needham-Schroeder Protocol
Il protocollo Needham-Schroeder è un meccanismo di auth e distr di chiavi di sessione basato sull'utilizzo di un Key Distribution Center che funge da terza parte fidata.
Il suo scopo è far si che due entità Alice e Bob possano ottenere in modo sicuro una chiave di sessione per poter comunicare in modo riservato e autenticato.

## Kerberos
Kerberos è un servizio di  autenticazione sviluppato dall'MIT, l'obbiettivo di Kerberos è stato di centralizzare l'autenticazione per l'accesso a tutte le risorse di una rete aziendale, prima di un approccio distribuito si doveva andare ad implementare per ogni risorsa di rete un meccanismo di autenticazione sicuro, risulta evidente come un approccio di questo tipo risulta poco scalabile e difficile da mantenere all'aumentare del numero di risorse.

Attualmente esistono due versioni di Kerberos, la v4 e v5, entrambe si basano su Needham-Schroeder.

### Kerberos v4

Si basa su uno schema dove vi sono 4 entità:
- Authentication Server (AS):
	Verifica l'identità dell'utente e rilascia un "ticket" che dimostra l'autenticazione dell'utente sul Authentication Server
- Ticket Granting Server (TGS):
	Se l'utente ha un "ticket" di autenticazione fornito dall'AS il TGS gli fornisce i "ticket" necessari per accedere ai vari servizi e risorse offerti dalla rete.
- Client (C)
	End-point che vuole accedere ad una risorsa sulla rete
- Risorsa (V)
	Servizio/Risorsa accessibile sulla rete sotto autenticazione

Il dialogo si suddivide in **3 FASI**:
- Autenticazione dell'Utente
- Richiesta di accesso ad un servizio
- Accesso al servizio

#### **1. Autenticazione Utente, ottenere ticket granting ticket**

In questa prima fase C contatta AS specificando in chiaro:

**ID<sub>C</sub>** ||  **ID<sub>TGS</sub>** || **TS<sub>1</sub>** 

- Chi sono - **ID<sub>C</sub>**
- Il servizio alla quale voglio accedere (al momento TGS) - **ID<sub>TGS</sub>**
- Quando ho mandato il messaggio (per evitare reply) - **TS<sub>1</sub>**

AS risponde con un messaggio cifrato con la chiave publica di C:

**K<sub>C,TGS</sub>** || **ID<sub>TGS</sub>** ||  **TS<sub>2</sub>** || **Lifetime<sub>2</sub>** ||  **Ticket<sub>TGS</sub>**

- Chiave di sessione da utilizzare nelle comunicazioni tra C e TGS - **K<sub>C,TGS</sub>**
- ID del servizio alla quale puoi accedere con quelle credenziali - **ID<sub>TGS</sub>**
- Istante di generazione del Ticket fornito -  **TS<sub>2</sub>**
- Tempo di validità del Ticket fornito -  **Lifetime<sub>2</sub>**
- Ticket per accedere a TGS -  **Ticket<sub>TGS</sub>**

Il ticket viene sempre crittografato con la chiave publica della risorsa alla quale il client vuole accedere, per il client è una blackbox, si limita a presentarla alla risorsa dopo averla ricevuta. I ticket hanno sempre una forma del tipo:

 **K<sub>sessione</sub>** ||  **ID<sub>Client</sub>** || **AD<sub>Client</sub>** ||  **ID<sub>Risorsa</sub>** || **TS** || **Lifetime**

**AD<sub>Client</sub>** (Authorization Data) questo campo contiene informazioni riguardo ai permessi di quell'utente specifico sulla risorsa alla quale vuole accedere (Privilegi, gruppi di appartenenza, restrizioni, policy).

#### **2. Richiesta accesso ad un servizio, ottenere service granting ticket**

In questa fase C contatta TGS specificando in chiaro:

**ID<sub>V</sub>** ||  **Ticket<sub>TGS</sub>** || **Auth<sub>Client</sub>** 

- **Auth<sub>Client</sub>** Contiene:

**ID<sub>C</sub>** ||  **AD<sub>C</sub>** || **TS** 

TGS risponde con un messaggio cifrato con la chiave publica di C:

**K<sub>C,V</sub>** || **ID<sub>V</sub>** ||  **TS<sub>4</sub>** || **Ticket<sub>V</sub>**

- Chiave di sessione da utilizzare nelle comunicazioni tra C e TGS - **K<sub>C,TGS</sub>**
- ID del servizio alla quale puoi accedere con quelle credenziali - **ID<sub>TGS</sub>**
- Istante di generazione del Ticket fornito -  **TS<sub>2</sub>**
- Tempo di validità del Ticket fornito -  **Lifetime<sub>2</sub>**
- Ticket per accedere a TGS -  **Ticket<sub>TGS</sub>**


#### **3. Scambio di autenticazione client server, ottenere accesso al servizio

Il client C vuole ottenere i permessi per accedere alla risorsa, C invia a V:

 **Ticket<sub>V</sub>** || **Auth<sub>Client</sub>**

- Service granting ticket ottenuto nella seconda fase cifrato con la chiave publica di V - **Ticket<sub>V</sub>**
- Autenticatore per rafforzare l'autenticità, contiene un timestamp diverso rispetto allo step precedente, cifrato con la chiave di sessione **K<sub>C,V</sub>**  - **Auth<sub>Client</sub>**

l TGS risponde con un messaggio cifrato con la chiave di sessione **K<sub>C,V</sub>** contenente il **TS**<sub>5</sub> incrementato contenuto nel Autenticatore.

**TS<sub>5</sub>+1**


### Kerberos Realm

In kerberos v5 viene introdotto il concetto di Realm, ovvero un insieme di entità che definisce il perimetro amministrativo di un KDC. Abbiamo un insieme di risorse che condividono lo stesso KDC, ogni risorsa è identificata da **nome-realm/nome-entità**, diversi realm possono essere joinati da relazioni di trusting, inoltre si possono definire strutture di realm gerarchici per la gestione di reti private complesse.
Supponendo che un client voglia accedere ad una risorsa di un altro realm, tra il realm A ed il realm B esiste una relazione di trust, quello che avviene è il seguente:
- C ottiene TGT da A/AS
- C ottiene SGT1 da A/TGS
- C ottiene SGT2 da B/TGS
- C ottiene l'accesso a V presentando SGT2

Quindi l'autenticazione avviene sempre con l'authentication server del dominio locale, bisogna ottenere un SGT per accedere al TGS del dominio esterno.


Kerberos v5
Anche se è nata negli anni 90 è la versione ancora utilizzata, nasce per sopperire ad alcune criticità di Kerberos v4, nello specifico:
- Introduzione ad IPv6
- Soppiantato DES a favore di una compatibilità con diversi algoritmi di cifratura tra cui AES e 3DES
- Aggiunta di nonce per difendersi da Reply Attack
- Introduzione ai Realm, e la comunicazione tra diversi realm
- Gestione più accurata dei ticket attraverso l'aggiunta del parametro times, il quale specifica il numero di volte in cui è possibile utilizzare il ticket
- Introduzione di un parametro option nei messaggi, per specificare opzioni aggiuntive

Maggiori differenze e migliorie nello scambio di messaggi:
- Introduzione del campo **Options** nelle richieste del client, viene introdotto pure il campo **Flag** nei ticket per permettere nuove funzionalità
- Introduzione del campo **Realm**
- Sostituzione del timestamp con un valore di nonce
- Migliorie di efficienza nella cifratura dei messaggi, adesso solo i campi necessari vengono cifrati, per esempio nel messaggio di risposta al client non si cifra un ticket che risulta essere già cifrato, come non si cifrano i campi **Realm<sub>C</sub>** e **ID<sub>C</sub>**
- Introduzione del campo **Times**, indica quante volte è possibile utilizzare il ticket prima che si consumi, può essere richiesto un valore di utilizzo per l'emissione del ticket
- Nell'ultima fase viene inviata alla risorsa l'authenticator, in questo caso oltre a contenere info di autenticazione per il cient, conterrà valori come:
	- **Subkey**: Il client può richiedere esplicitamente l'utilizzo di una chiave di sessione, qualora non specificato continurà ad essere utilizzata la **K<sub>C,V</sub>**
	- **Numero di sequenza Seq#** Campo opzionale per specificare un numero dalla quale iniziare a contare