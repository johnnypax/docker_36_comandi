# Docker – Cheatsheet comandi base

Oct 16, 2026 · @Giovanni

## Info e sistema

| Comando | Cosa fa |
| --- | --- |
| `docker version` | Versione di client e server |
| `docker info` | Stato del daemon: container, immagini, storage |
| `docker <comando> --help` | Aiuto su un comando specifico |
| `docker login` / `docker logout` | Accesso a un registry (Docker Hub di default) |
| `docker system df` | Spazio su disco usato da immagini, container, volumi |

## Immagini

| Comando | Cosa fa |
| --- | --- |
| `docker pull nginx:latest` | Scarica un'immagine dal registry |
| `docker images` (o `docker image ls`) | Elenca le immagini locali |
| `docker build -t app:1.0 .` | Crea un'immagine dal Dockerfile nella cartella corrente |
| `docker build -f Dockerfile.dev -t app:dev .` | Build con un Dockerfile diverso |
| `docker tag app:1.0 utente/app:1.0` | Assegna un nuovo nome/tag |
| `docker push utente/app:1.0` | Pubblica l'immagine sul registry |
| `docker rmi app:1.0` | Elimina un'immagine |
| `docker history app:1.0` | Mostra i layer dell'immagine |
| `docker search redis` | Cerca immagini su Docker Hub |

## Avviare un container: docker run

Esempio tipico: `docker run -d --name web -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html -e TZ=Europe/Rome --restart unless-stopped nginx`

| Opzione | Significato |
| --- | --- |
| `-d` | Avvio in background (detached) |
| `-it` | Terminale interattivo, es. `docker run -it ubuntu bash` |
| `--name web` | Nome del container |
| `-p 8080:80` | Porta host 8080 → porta container 80 |
| `-v /host/path:/container/path` | Monta una cartella o un volume |
| `-e CHIAVE=valore` | Variabile d'ambiente |
| `--env-file .env` | Variabili da file |
| `--rm` | Elimina il container quando si ferma |
| `--restart unless-stopped` | Riavvio automatico (anche `always`, `on-failure`) |
| `--network mia-rete` | Collega a una rete specifica |
| `-w /app` | Directory di lavoro nel container |

## Gestione container

| Comando | Cosa fa |
| --- | --- |
| `docker ps` | Container in esecuzione |
| `docker ps -a` | Tutti i container, anche fermi |
| `docker stop web` | Ferma un container (SIGTERM, poi SIGKILL) |
| `docker start web` | Riavvia un container fermo |
| `docker restart web` | Stop + start |
| `docker kill web` | Ferma subito (SIGKILL) |
| `docker rm web` | Elimina un container fermo (`-f` per forzare) |
| `docker exec -it web bash` | Apre una shell nel container (`sh` se manca bash) |
| `docker exec web ls /app` | Esegue un singolo comando |
| `docker cp web:/app/log.txt .` | Copia file container → host (e viceversa) |
| `docker rename web web-old` | Rinomina un container |

## Log e debug

| Comando | Cosa fa |
| --- | --- |
| `docker logs web` | Mostra i log del container |
| `docker logs -f --tail 100 web` | Segue i log in tempo reale, ultime 100 righe |
| `docker inspect web` | Dettagli completi in JSON (IP, mount, config) |
| `docker inspect -f '{{.NetworkSettings.IPAddress}}' web` | Estrae un singolo campo |
| `docker stats` | Uso live di CPU, RAM, rete |
| `docker top web` | Processi in esecuzione nel container |
| `docker port web` | Mappatura delle porte |
| `docker diff web` | File modificati rispetto all'immagine |

## Volumi e reti

I volumi conservano i dati oltre la vita del container; le reti permettono ai container di parlarsi per nome.

| Comando | Cosa fa |
| --- | --- |
| `docker volume create dati` | Crea un volume |
| `docker volume ls` | Elenca i volumi |
| `docker volume inspect dati` | Dettagli e percorso sull'host |
| `docker volume rm dati` | Elimina un volume |
| `docker run -v dati:/var/lib/mysql mysql` | Usa un volume con nome |
| `docker network create mia-rete` | Crea una rete bridge |
| `docker network ls` | Elenca le reti |
| `docker network connect mia-rete web` | Collega un container esistente |
| `docker network inspect mia-rete` | Container collegati e configurazione |
| `docker network rm mia-rete` | Elimina una rete |

## Docker Compose

Si usa dalla cartella che contiene `compose.yaml` (o `docker-compose.yml`). La sintassi attuale è `docker compose`, senza trattino.

| Comando | Cosa fa |
| --- | --- |
| `docker compose up -d` | Avvia tutti i servizi in background |
| `docker compose up -d --build` | Ricostruisce le immagini e avvia |
| `docker compose down` | Ferma e rimuove container e reti |
| `docker compose down -v` | Come sopra, eliminando anche i volumi |
| `docker compose ps` | Stato dei servizi |
| `docker compose logs -f app` | Log di un servizio in tempo reale |
| `docker compose exec app sh` | Shell in un servizio attivo |
| `docker compose restart app` | Riavvia un servizio |
| `docker compose pull` | Aggiorna le immagini dei servizi |

Esempio minimo di `compose.yaml`:

```yaml
services:
  app:
    build: .
    ports:
      - "8080:3000"
    environment:
      - DB_HOST=db
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

## Pulizia

Attenzione: i comandi `prune` eliminano senza possibilità di recupero; aggiungi `-f` solo se sai cosa stai togliendo.

| Comando | Cosa elimina |
| --- | --- |
| `docker container prune` | Tutti i container fermi |
| `docker image prune` | Immagini "dangling" (senza tag) |
| `docker image prune -a` | Tutte le immagini non usate da container |
| `docker volume prune` | Volumi non collegati a container |
| `docker network prune` | Reti inutilizzate |
| `docker system prune` | Container fermi, reti, immagini dangling, cache di build |
| `docker system prune -a --volumes` | Pulizia totale, volumi inclusi |
| `docker rm -f $(docker ps -aq)` | Rimuove tutti i container (Linux/macOS) |

## Dockerfile di esempio

Un Dockerfile minimo per un'app Node.js; si costruisce con `docker build -t app .` e si avvia con `docker run -p 3000:3000 app`.

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

| Istruzione | Ruolo |
| --- | --- |
| `FROM` | Immagine di partenza |
| `WORKDIR` | Cartella di lavoro |
| `COPY` / `ADD` | Copia file nell'immagine |
| `RUN` | Esegue un comando in fase di build |
| `ENV` | Variabile d'ambiente |
| `EXPOSE` | Documenta la porta usata |
| `CMD` / `ENTRYPOINT` | Comando eseguito all'avvio del container |

Aggiungi un file `.dockerignore` (es. `node_modules`, `.git`, `.env`) per build più leggere e senza segreti.
