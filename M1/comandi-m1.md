# M1 Esercizi Docker
Nome e cognome: Peschechera Vincenzo
Ambiente: GitHub Codespaces 
Dichiarazione uso IA: Dichiaro di aver usato Gemini AI per l'aiuto alla risoluzione di alcuni esercizi e per riassumere tutti i comandi usati

## Esercizio 1 — Scheda della postazione
* `docker version` --> Mostra la versione del client e del server (demone) di Docker installati sulla postazione.
* `docker info | grep -E "Server Version|Operating System|OSType|Containers|Running|Paused|Stopped|Images"` --> Estrae dal sistema le informazioni dettagliate sul sistema operativo del server e lo stato di conteggio dei container e delle immagini.

---

## Esercizio 2 — Il server web delle prenotazioni sulla porta 8081
* `docker container run -d --name prenotazioni-web-prova -p 8081:80 nginx:alpine` --> Avvia in background (`-d`) un container di prova basato su Nginx, assegnandogli un nome specifico e mappando la porta 8081 dell'host sulla porta 80 del container.
* `docker container ls` --> Elenca i container attualmente in esecuzione per verificare l'avvio corretto.
* `curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8081` --> Invia una richiesta HTTP locale per verificare che il server web risponda correttamente (restituendo il codice HTTP).
* `docker container logs prenotazioni-web-prova` --> Mostra i registri degli eventi generati dal container web.

---

## Esercizio 3 — Percorso completo del ciclo di vita
* `docker container rm -f prenotazioni-web-prova` --> Forza la rimozione immediata di un container esistente per pulire l'ambiente.
* `docker container run -d --name prenotazioni-web-prova -p 8081:80 nginx:alpine` --> Ricrea e avvia il container di prova.
* `docker container inspect --format "{{.State.Status}}" prenotazioni-web-prova` --> Estrae lo stato operativo corrente del container in formato testuale.
* `docker container pause prenotazioni-web-prova` --> Mette in pausa i processi interni del container.
* `docker container unpause` --> Riattiva un container precedentemente in pausa.
* `docker container stop` --> Ferma in modo ordinato il container.
* `docker container start` --> Riavvia un container precedentemente fermo.
* `docker container rm` --> Rimuove definitivamente il container e il suo nome.

---

## Esercizio 4 — Due container dalla stessa immagine, e un terzo che non parte
* `docker container rm -f prenotazioni-web-uffici prenotazioni-web-aule prenotazioni-web-laboratori` --> Rimuove forzatamente i container di prova precedenti per evitare conflitti.
* `docker container run -d --name prenotazioni-web-aule -p 8082:80 nginx:alpine` --> Avvia il primo container sulla porta 8082.
* `docker container run -d --name prenotazioni-web-laboratori -p 8083:80 nginx:alpine` --> Avvia il secondo container sulla porta 8083.
* `docker container run -d --name prenotazioni-web-uffici -p 8082:80 nginx:alpine` --> Tenta di avviare un terzo container sulla stessa porta 8082 già occupata, generando un errore di conflitto.
* `docker container ls -a` --> Mostra tutti i container, inclusi quelli che non sono riusciti ad avviarsi.

---

## Esercizio 5 — Diagnosi di un container che non parte
* `docker container run -d --name prenotazioni-db-prova postgres:16-alpine` --> Tenta di avviare un database PostgreSQL senza specificare la password obbligatoria richiesta dall'immagine.
* `docker container inspect --format "{{.State.ExitCode}}" prenotazioni-db-prova` --> Legge il codice di uscita del container per capire il motivo dell'anomalia.
* `docker container logs prenotazioni-db-prova` --> Leggi i messaggi di errore emessi dal database prima di spegnersi.
* `docker container run -d --name prenotazioni-db-prova -e POSTGRES_USER=prenotazioni -e POSTGRES_PASSWORD=prenotazioni2026 -e POSTGRES_DB=prenotazioni postgres:16-alpine` --> Avvia correttamente il container passando le variabili d'ambiente obbligatorie per l'autenticazione.

---

## Esercizio 6 — La tabella dei docenti
* `docker container run -d --name prenotazioni-db -e POSTGRES_USER=prenotazioni -e POSTGRES_PASSWORD=prenotazioni2026 -e POSTGRES_DB=prenotazioni postgres:16-alpine` --> Avvia il container del database principale senza esporre porte all'esterno.
* `until docker container exec prenotazioni-db pg_isready -U prenotazioni; do sleep 1; done` --> Attende in modo automatizzato che il servizio database sia pronto a ricevere connessioni.
* `docker container exec prenotazioni-db psql -U prenotazioni -d prenotazioni -c "CREATE TABLE docente (...);"` --> Esegue un comando SQL all'interno del container per creare la tabella dei docenti.
* `docker container exec prenotazioni-db psql -U prenotazioni -d prenotazioni -c "INSERT INTO docente ...;"` --> Inserisce i record di prova nella tabella.
* `docker container exec prenotazioni-db psql -U prenotazioni -d prenotazioni -c "SELECT * FROM docente ...;"` --> Interroga il database per visualizzare i dati inseriti.

---

## Esercizio 7 — Leggere l'ispezione
* `docker container inspect prenotazioni-web-aule` --> Estrae l'intero documento di configurazione in formato JSON relativo al container.
* `docker container inspect --format "{{.State.Status}}" prenotazioni-web-aule` --> Estrae selettivamente lo stato corrente.
* `docker container inspect --format "{{.State.StartedAt}}" prenotazioni-web-aule` --> Estrae la data e l'ora esatta di avvio.
* `docker container inspect --format "{{.Config.Image}}" prenotazioni-web-aule` --> Estrae il nome dell'immagine di partenza.
* `docker container inspect --format "{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}" prenotazioni-web-aule` --> Estrae l'indirizzo IP assegnato nella rete Docker interna.

---

## Esercizio 8 — Che cosa libera davvero la pulizia
* `docker system df` --> Mostra un riepilogo dello spazio su disco occupato da immagini, container e volumi Docker.
* `docker container prune -f` --> Rimuove automaticamente tutti i container che si trovano attualmente nello stato di arresto (stopped).
* `docker image prune -f` --> Rimuove le immagini orfane o non taggate (dangling) per liberare spazio.

---

## Esercizio 9 — Dimostrare che i dati non sopravvivono
* `docker container exec prenotazioni-db psql -U prenotazioni -d prenotazioni -c "CREATE TABLE prenotazione (...);"` --> Crea una tabella di test all'interno del database del container.
* `docker container exec prenotazioni-db psql -U prenotazioni -d prenotazioni -c "INSERT INTO prenotazione ...;"` --> Inserisce dati di esempio.
* `docker container rm -f prenotazioni-db` --> Distrugge completamente il container e la sua memoria volatile.
* `docker container run -d --name prenotazioni-db ... postgres:16-alpine` --> Ricrea il container da zero con la stessa immagine.
* `docker container exec prenotazioni-db psql -U prenotazioni -d prenotazioni -c "SELECT * FROM prenotazione;"` --> Verifica che la tabella e i dati siano andati persi a causa della mancanza di volumi di persistenza.

---

## Esercizio 10 — Chiusura ordinata e uccisione immediata
* `docker container stop prenotazioni-db` --> Invia un segnale di arresto controllato (graceful) al container, permettendogli di salvare i dati in corso.
* `docker container kill prenotazioni-db` --> Tronca immediatamente l'esecuzione del processo del container senza attendere la chiusura pulita (codice di uscita 137).
* `docker container logs --tail 10 --timestamps prenotazioni-db` --> Analizza gli ultimi registri temporizzati per verificare la modalità di spegnimento subita dal servizio.

---

## Esercizio 11 — Scheda tecnica di un container
* `docker container run -d --name prenotazioni-db -e POSTGRES_USER=prenotazioni -e POSTGRES_PASSWORD=prenotazioni2026 -e POSTGRES_DB=prenotazioni postgres:16-alpine` --> Comando di avvio documentato per il container di produzione/servizio.
* `docker container stop prenotazioni-db` --> Comando di arresto pulito del servizio.
* `docker container rm prenotazioni-db` --> Comando di rimozione definitiva del container documentato.