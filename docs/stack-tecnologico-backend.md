# Will of the Void – Stack tecnologico del backend

29 ottobre 2026 · Mondir

Il backend è un'applicazione Java 25 con Spring Boot 4.1.2 e PostgreSQL: uno stack richiesto in gran parte dal brief, completato da librerie scelte per sicurezza, test e documentazione delle API.

## Linguaggio e strumenti di base

Java, Spring Boot e GitHub sono imposti dal brief; le altre scelte sono gli standard dell'ecosistema Spring, così tutti e tre i membri trovano documentazione ed esempi senza dover integrare strumenti insoliti.

| Tecnologia | Ruolo | Perché |
| --- | --- | --- |
| Java 25 | Linguaggio della logica lato server | Richiesto dal brief. |
| Spring Boot 4.1.2 | Framework per le API REST e la logica di business | Richiesto dal brief. Configura in automatico server web, database e sicurezza, quindi il tempo va nelle regole del gioco e non nell'infrastruttura. |
| Maven | Build e gestione delle dipendenze | È lo strumento predefinito di Spring Initializr. Permette di tenere il motore di gioco in un modulo separato, come chiede il brief ("componente Java isolato"). |
| Spring Data JPA + Hibernate 6 | Accesso ai dati | I repository generano le query più comuni senza SQL scritto a mano. Hibernate 6 mappa direttamente le colonne `jsonb` di PostgreSQL. |
| Bean Validation (Jakarta Validation) | Validazione dell'input | I vincoli si scrivono come annotazioni sui DTO (`@NotNull`, `@Size`, `@Min`) e Spring rifiuta le richieste non valide prima che arrivino alla logica. |
| Swagger (Springdoc OpenAPI) | Documentazione delle API | Richiesto dal brief. La documentazione si genera dal codice, quindi resta allineata alle API anche quando cambiano, e il team frontend può provare gli endpoint dal browser. |
| JUnit 5 | Test automatici | Richiesto dal brief per il motore di gioco. È integrato in Spring Boot e supporta i test parametrizzati, utili per verificare le 12 relazioni tra fazioni con un solo test. |
| Docker | Ambiente locale | Tutti e tre avviano lo stesso PostgreSQL, alla stessa versione, con un solo comando. Elimina i problemi "sul mio computer funziona". |
| Git, GitHub, GitHub Projects | Versionamento e gestione del lavoro | Richiesti dal brief. GitHub Projects collega i task alle pull request, così l'avanzamento di ogni membro è visibile. |

## Dipendenze del progetto

Ogni dipendenza copre un requisito del brief o una funzionalità decisa dal team; nessuna è presente solo per comodità, tranne Lombok, che si giustifica con il tempo risparmiato.

| Dipendenza | Ruolo | Perché |
| --- | --- | --- |
| Spring Web | Controller REST, rotte HTTP, scambio di JSON | Il portale comunica con il frontend solo tramite API REST, come chiede il brief. Spring Web gestisce rotte, conversione JSON e codici di risposta. |
| Spring Security + JWT | Autenticazione stateless, controllo accessi, ruoli ROLE_USER e ROLE_ADMIN | Il token JWT viaggia con ogni richiesta, quindi il server non conserva sessioni: va bene per un frontend separato e funziona anche per autenticare la connessione WebSocket. I ruoli separano i giocatori dagli amministratori del backoffice, che il brief richiede. |
| Spring Data JPA / Hibernate | Mappatura tra oggetti Java e tabelle | Le 24 tabelle diventano entità Java. Le transazioni e i lock pessimistici servono agli scambi, dove il brief chiede che nessuna carta venga persa o duplicata. |
| Spring Validation (Hibernate Validator) | Validazione delle richieste e vincoli server-side | Il brief chiede che il portale verifichi i vincoli dei mazzi: la validazione lato server è l'unica affidabile, perché quella del frontend si può aggirare. |
| JUnit 5 & AssertJ | Test del motore di gioco | Il brief richiede il motore isolato e coperto da test JUnit. AssertJ rende le verifiche leggibili come frasi, utile per controllare log di partita lunghi e casi limite. |
| Lombok | Meno codice ripetitivo | Genera getter, setter, costruttori e builder: le entità restano corte e leggibili. Va usato da tutti allo stesso modo, deciso all'inizio. |
| Springdoc OpenAPI (Swagger UI) | Documentazione interattiva delle API | È la libreria che produce la documentazione Swagger richiesta dal brief, compatibile con Spring Boot 4.1.2. |

Le funzionalità decise richiedono due dipendenze che la lista non nomina: Spring WebSocket (sfide in tempo reale e amici online) e Flyway (migrazioni versionate dello schema e dati iniziali).

## Database e persistenza

Il brief indica MySQL; il team usa PostgreSQL perché quattro regole del gioco si possono garantire direttamente nel database, invece che solo nel codice.

| Tecnologia | Ruolo | Perché |
| --- | --- | --- |
| PostgreSQL | Database relazionale | Vedi i motivi sotto. Gli stessi lock (`SELECT ... FOR UPDATE`) e le stesse transazioni di MySQL coprono gli scambi senza perdite né duplicazioni. |
| Jackson | Conversione tra oggetti Java e JSON | Serializza il replay della partita (log turno per turno e copia dei mazzi) salvato in colonne `jsonb`, e lo restituisce al frontend così com'è, senza trasformazioni. È già incluso in Spring Web. |

Cosa offre PostgreSQL che lo schema usa davvero:

- **`jsonb`**: il log della partita e la copia dei mazzi si salvano in formato binario, più veloce da leggere e interrogabile se servisse in futuro.
- **Indice univoco su espressioni**: un'amicizia tra A e B non può esistere due volte in direzioni opposte, grazie a un indice su `LEAST` e `GREATEST` dei due utenti.
- **Indice univoco parziale**: un solo torneo in corso per utente, garantito con un indice valido solo sulle righe con stato `IN_CORSO`.
- **`timestamptz`**: le date si salvano con il fuso orario, e le scadenze (Erranti a 7 giorni, scambi a 48 ore, sfide a 30 secondi, protezione dopo gli attacchi) non dipendono dal fuso del server.

PostgreSQL è gratuito e supportato da Spring Boot quanto MySQL; il cambio va comunque concordato con i docenti, visto che il brief nomina MySQL.
