L'architettura ARM (Advanced RISC Machine) è una delle architetture più diffuse al mondo, impiegata in una vasta gamma di dispositivi, dai telefoni cellulari ai server.

## Caratteristiche principali

- **Architettura RISC**: ARM si basa sul modello RISC (Reduced Instruction Set Computer), che prevede un set di istruzioni semplice e ottimizzato. Questo approccio mira a migliorare le prestazioni e ridurre il consumo energetico.
- **Efficienza energetica e flessibilità**: L'architettura ARM è rinomata per la sua efficienza energetica e flessibilità, che la rendono ideale per dispositivi mobili, sistemi embedded e server. Questa combinazione di efficienza e adattabilità ha contribuito alla sua ampia diffusione.
- **Pipeline**: Per migliorare il throughput e le prestazioni, i processori ARM utilizzano una pipeline a più stadi. Le versioni più recenti implementano pipeline tipicamente a 3, 5 o 7 stadi.

L'architettura ARM apporta dei miglioramenti sull'architettura RISC per soddisfare le esigenze delle applicazioni embedded. ARM è caratterizzato da:
- Un ampio set di registri uniformi
- Architettura Load Store
- Istruzioni di lunghezza fissa
- In ingresso all'ALU è presente uno shifter per l' implementazioni di operazioni matematiche più complesse.
- Modalità di indirizzamento con auto-incremento e auto-decremento per agevolare l'accesso ad aree di memoria contigue.
- Istruzioni di Load e Store Multiple per massimizzare il throughput dei dati.
- Esecuzione condizionale delle istruzioni per massimizzare il throughput di esecuzione.
- ARM può essere utilizzato sia in **Little Endian** che in **Big Endian**

ARM trova impiego in diversi ambiti e settori come dispositivi Mobili e Embedded, IoT e Server, questo grazie alla flessibilità e all'efficienza energetica.

## Set di registri

I processori ARM utilizzano diversi tipi di registri per memorizzare dati, indirizzi e informazioni sullo stato del processore. I registri sono locazioni di memoria ad alta velocità all'interno del processore, utilizzate per eseguire operazioni e manipolare i dati.
Un processore ARM standard ha 16 registri generali (R0-R15), ognuno a 32 bit, utilizzati per memorizzare temporaneamente dati e indirizzi:
- **R0-R12**: Sono registri multiuso, utilizzabili per vari scopi, come l'archiviazione temporanea di valori durante i calcoli.
- **R13 (Stack Pointer - SP)**: È utilizzato per puntare alla cima dello stack, una struttura dati usata per la gestione di chiamate di funzioni e variabili locali.
- **R14 (Link Register - LR)**: Memorizza l'indirizzo di ritorno quando viene eseguita una subroutine o una chiamata di funzione.
- **R15 (Program Counter - PC)**: Contiene l'indirizzo dell'istruzione corrente da eseguire. Quando il processore è in esecuzione in stato ARM, tutte le istruzioni sono a 32 bit e allineate alla parola. Il valore di PC è memorizzato nei bit [31:2], con i bit [1:0] non definiti (essendo multiplo di 4 sarà sempre *00*)

Registri di Stato (**Program Status Register - PSR**) Oltre ai registri generali, esistono registri di stato che memorizzano informazioni critiche sullo stato attuale del processore:

- **CPSR (Current Program Status Register)**: Contiene flag di stato che riflettono il risultato dell'ultima operazione eseguita dall'ALU e bit che indicano la modalità di esecuzione corrente del processore, abbiamo:
	- **N (Negative flag)**: Indica se l'ultimo risultato di un'operazione aritmetica o logica è stato un valore negativo
	- **Z (Zero flag)**: Indica se l'ultimo risultato è zero
	- **C (Carry flag)**: Indica se c'è stato un carry (o prestito) nelle operazioni aritmetiche, questo flag è utile per operazioni aritmetiche a precisione multipla, dove il riporto o il prestito di un'operazione su una parte dei dati deve essere passato all'operazione successiva
	- **V (Overflow flag)**: Indica un overflow nei calcoli con numeri interi
	- **Modalità di esecuzione**: Il CPSR contiene anche bit che indicano la modalità di esecuzione del processore (User, FIQ, IRQ, ecc.), le modalità di esecuzione definiscono il livello di privilegio e l'accesso alle risorse del sistema.

- **SPSR (Saved Program Status Register)**: Memorizza lo stato del CPSR quando si passa da una modalità a un'altra, ad esempio, da User mode a una modalità privilegiata come Supervisor o IRQ. Ogni modalità privilegiata (eccetto la modalità di sistema) ha un SPSR associato. Questo registro viene utilizzato per salvare lo stato del CPSR quando viene inserita la modalità privilegiata, in modo che lo stato utente possa essere ripristinato completamente quando il processo utente viene ripreso

ARM supporta diverse **modalità operative** per il processore, ognuna con diversi privilegi e accesso alle risorse. Le modalità di esecuzione si distinguono in privilegiate e non privilegiate.

**Modalità non privilegiate**:
- **User mode**: È la modalità utilizzata per eseguire applicazioni di livello utente. In questa modalità, il processore ha accesso limitato alle risorse hardware e non può eseguire operazioni critiche

**Privilegiate**:
- **Supervisor Mode (SVC)**: Utilizzata dal sistema operativo per eseguire operazioni privilegiate. Il processore passa a questa modalità quando viene chiamata un'istruzione di sistema/system call.
- **FIQ Mode (Fast Interrupt Request)**: Una modalità privilegiata per la gestione di interrupt veloci e a bassa latenza, utilizzata per eventi che richiedono una risposta immediata
- **IRQ Mode (Interrupt Request)**: Una modalità privilegiata utilizzata per gestire le interrupt hardware standard.
- **Abort Mode**: Utilizzata quando il processore rileva un errore di memoria o altre eccezioni gravi.
- **Undefined Mode**: Attivata quando il processore cerca di eseguire un'istruzione non riconosciuta o non supportata.
- **System Mode**: Una modalità privilegiata simile a Supervisor, usata per eseguire codice di sistema.

Inoltre è presente il concetto di Banked Register, in generale in ARM sono presenti ben **37 registri**:
- 17 registri standard di cui 16 General Purpose.
- 20 registri nascosti al programma e gestiti dall'hardware come copie dei registri standard (banked register).

![[Pasted image 20250122181833.png]]
Questi banked register vengono utilizzati quando il processore si trova in modalità di esecuzione diversa rispetto a System e User.
Per ogni modalità si ha un set di Banked register a disposizione, ad esempio per **FIQ**:
- [R8_fiq : R14_fiq] registri general purpose + Stack Pointer + Link register
- [SPSR_fiq] Copia del Current Program Status Register che può essere tranquillamente modificato in modo da non sporcare il CPSR della modalità Utente.

L'utilizzo di registri banked permette di velocizzare il cambio di contesto dei registri al passaggio da una modalità di esecuzione ad un altra, si noti infatti come FIQ abbia una quantità di registri banked più elevato, questo perché rispetto alle altre modalità deve gestire molto più velocemente la routine.

## Set di Istruzioni

Il set di istruzioni ARM nasce per essere semplice compatto e altamente efficiente, vi sono diversi set di istruzioni:
- **ARM a 32 bit**
- **Thumb a 16 bit**
- **AArch64 a 64 bit**


Il set di istruzioni **Thumb** è una versione compatta delle istruzioni ARM Standard a 32 bit, esso è molto utilizzato nei sistemi embedded in quanto hanno una migliore efficienza energetica a discapito delle prestazioni. DI seguito le caratteristiche fondamentali che differenziano l'instruction set Thumb:
- Istruzioni a 16 bit, thumb-2 usa una combinazione di 16 e 32 bit
- 8 registri R0 a R7
- Operazioni di barrel shift non sono integrate nell'ALU, sono presenti delle istruzioni addette allo scopo
- Non sono presenti istruzioni di Load e Store multiple in quanto non codificabili con soli  16 bit

Con l'architettura ARMv8, ARM ha introdotto il set di istruzioni a 64-bit, noto come **AArch64**, questo set è progettato per sfruttare le caratteristiche dei sistemi a 64 bit, fornendo accesso a un maggiore spazio di indirizzamento e a registri più ampi.

Sono inoltre disponibili le estensioni **NEON\SIMD**, queste estensioni forniscono un set di istruzioni dedicati all'elaborazione parallela dei dati. Le operazioni NEON eseguono operazioni matematiche e logiche su vettori di dati così da parallelizzarne l'esecuzione.

Essendo di base un architettura RISC tutte le istruzioni lavorano mediante l'utilizzo di registri e per tale motivo sono disponibili delle operazioni di load/store. Tutte le istruzioni ARM sono tali da:
- Possedere tre operandi
- Possono essere eseguite in modo condizionale
Inoltre le operazioni di load e store possono agire su registri multipli, e si ricordi che le ALU possono eseguire lo shift destro e sinistro di un operando in ingresso.


