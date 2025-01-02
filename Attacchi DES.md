Il DES, sebbene all'epoca fosse considerato sicuro risulta oggi vulnerabile per vari motivi:
- Spazio delle chiavi ristretto per la potenza computazionale attuale
- Approcci cripto analitici che permettono ulteriormente di ridurre la complessità di rottura
Tra gli approcci di analisi crittografica abbiamo:
- Analisi crittografica differenziale
- Analisi crittografica lineare

## Analisi crittografica differenziale

Questo approccio introdotto intorno al 1990 tende a sfruttare le differenze tra coppie di due plaintext per analizzare le differenze nei rispettivi ciphertext.
Tramite un operazione di XOR viene definito un DELTA in input e si analizza il DELTA in output prodotto, è possibile andare a effettuare delle deduzioni sulle sottochiavi Ki. Tramite questa tecnica è possibile ridurre la complessità della ricerca della chiave da 2^ 55 a 2^ 47.

## Analisi crittografica lineare

Tramite questo approccio è possibile ridurre la complessità dell'attacco addirittura a 2^43.
L'idea è quella di trovare approssimazioni lineari che descrivano la relazione tra plaintext, ciphertext e chiave.
Bisogna conoscere 2^43 testi in chiaro (attacco known plain text), tramite una relazione del tipo:

$$
P[i_1, i_2, \dots, i_a] \oplus C[j_1, j_2, \dots, j_b] = K[k_1, k_2, \dots, k_c]
$$
Se per l'i-esima posizione il risultato del primo membro è per più del 50% 0, ki è probabilmente 0, altrimenti 1.
Tramite queste deduzioni è possibile cercare la chiave su una restrizione dello spazio delle chiavi, la difficoltà è ovviamente data dalla conoscenza di un grosso quantitativo di plaintext. 

