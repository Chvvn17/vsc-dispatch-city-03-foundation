# Lernjournal – Verteilte Systeme, Containerisierung

Kurs: Verteilte Systeme, Containerisierung | TEKO Schweizerische Fachschule
Autor: Patrick Michel

---

## Block 01 – Docker Refresher

### Worum es geht

Docker ist die Grundlage für alle weiteren Blöcke. Ein **Image** ist eine unveränderliche Vorlage, ein **Container** ist eine laufende Instanz davon. Mit einem **Dockerfile** wird beschrieben, wie ein Image gebaut wird. Eine **Registry** speichert Images. **Docker Compose** verwaltet mehrere Container als eine gemeinsame Umgebung.

### Was gemacht wurde

- Begriffe Image, Container, Dockerfile, Registry, Compose und Kubernetes definiert
- NGINX-Container gestartet, Status geprüft, Shell geöffnet und wieder entfernt
- Docker Compose Mini-Stack mit zwei Services (nginx + redis) gestartet, Netzwerke und Volumes untersucht
- Eigenes Docker-Image mit einer statischen Webseite gebaut und als `docker-refresh-web:1.0.0` getaggt
- Image zusätzlich mit `teko/docker-refresh-web:1.0.0` und `teko/docker-refresh-web:latest` getaggt
- `.dockerignore` erstellt, um unnötige Dateien aus dem Build Context auszuschliessen
- Docker-Debugging-Checkliste erarbeitet

### Warum

Bevor Container in Kubernetes laufen, muss man verstehen wie sie gebaut und betrieben werden. Docker ist das Fundament – ohne dieses Wissen ist Kubernetes schwer verständlich. Das Layer-Caching beim Build und die saubere Trennung von Image-Bau und Container-Betrieb sind zentrale Konzepte die durchgehend genutzt werden.

---

## Block 02 – Kubernetes Einstieg

### Worum es geht

Kubernetes orchestriert Container auf einem oder mehreren Nodes in einem **Cluster**. Die wichtigsten Objekte sind: **Pod** (kleinste Einheit), **Deployment** (verwaltet den gewünschten Zustand), **Service** (stabiler Netzwerkzugriff), **Namespace** (logische Trennung). k3d ermöglicht einen lokalen Kubernetes-Cluster als Docker-Container.

Der Kernmechanismus: Kubernetes vergleicht laufend den **gewünschten Zustand** (z.B. 3 Replicas) mit dem **aktuellen Zustand** und korrigiert Abweichungen automatisch.

### Was gemacht wurde

- k3d, kubectl und kustomize auf Version geprüft
- Cluster `teko-k8s` mit einem Server-Node und zwei Agent-Nodes erstellt
- Namespace `k8s-lab` erstellt und als Standard-Namespace gesetzt
- Deployment `web` mit `nginx:alpine` und 3 Replicas gestartet
- Labels und Selectors verwendet, um Pods zu filtern
- Service erstellt und App per Port-Forward lokal erreichbar gemacht
- Deployment auf 5 Replicas skaliert und Verteilung auf Nodes beobachtet
- Pod manuell gelöscht und beobachtet wie Kubernetes automatisch einen Ersatz erstellt
- `kubectl describe`, `logs` und `get events` zur Diagnose verwendet

### Warum

Kubernetes löst das Problem, Container zuverlässig und skalierbar zu betreiben. Das Self-Healing (automatischer Ersatz bei Pod-Ausfall), die deklarative Konfiguration und die Möglichkeit zur horizontalen Skalierung sind die zentralen Vorteile gegenüber manuellem Container-Management. k3d ermöglicht lokales Lernen mit echten Kubernetes-Objekten.

---

## Block 03 – Deployments, Services, ConfigMaps (Foundation)

### Worum es geht

Statt einer Standard-Applikation wird nun die **Dispatch City**-Anwendung eingesetzt – eine Food-Delivery-Simulation mit einem Dashboard und einer Control-API. Kubernetes-Manifeste werden mit **Kustomize** in Base und Overlays strukturiert. **ConfigMaps** und **Probes** (Readiness/Liveness) steuern das Laufzeitverhalten.

Die Control-API bleibt bewusst bei einer einzigen Replik, da sie einen In-Memory-Zustand hält. Mehrere Replicas würden zu inkonsistenten Ergebnissen führen.

### Was gemacht wurde

- GitHub-Repository (`vsc-dispatch-city-03-foundation`) als eigenes Repo aufgesetzt, Tag `v1.0.0` verwendet
- Cluster `teko-k8s` aus Block 2 wiederverwendet
- Rollen von `dashboard` und `control-api` erklärt
- Zwei Docker-Images gebaut (`food-delivery-control-api:local`, `food-delivery-dashboard:local`) und in den k3d-Cluster importiert
- `imagePullPolicy: IfNotPresent` erklärt
- Kustomize-Overlay `block-03-standalone` gerendert und deployed
- Service-Selector → Pod-Labels sowie `targetPort` → Container-Port-Verbindung nachvollzogen
- ConfigMap `simulation-config` mit `APP_MODE` und `TICK_MS` verwendet
- Readiness-Probe (`/health/ready`) und Liveness-Probe (`/health/live`) untersucht
- `TICK_MS` von 500 auf 2000 geändert, `rollout restart` durchgeführt und neuen Wert in Logs bestätigt
- Dashboard und Control-API per Port-Forward lokal erreichbar gemacht, Simulation beobachtet
- EndpointSlice, DNS-Auflösung und Architekturgrenze dokumentiert

### Warum

Kustomize erlaubt es, Kubernetes-Manifeste DRY (Don't Repeat Yourself) zu halten – gemeinsame Konfiguration in der Base, umgebungsspezifische Anpassungen in Overlays. ConfigMaps entkoppeln Konfiguration vom Image-Bau. Probes stellen sicher, dass Kubernetes nur dann Traffic an einen Pod sendet, wenn dieser wirklich bereit ist.

---

## Block 04 – Ingress und externe Zugriffe

### Worum es geht

Ein **Ingress** ist ein gemeinsamer HTTP-Einstiegspunkt, der eingehende Anfragen anhand von Pfaden an verschiedene Services weiterleitet. Der **Ingress Controller** (hier: Traefik) ist dafür verantwortlich, diese Regeln umzusetzen. Statt mehrerer Port-Forwards genügt ein einziger Port.

### Was gemacht wurde

- Block-4-Repository geklont und Installationsskript auf dem Projektverzeichnis ausgeführt
- Overlay `block-04-ingress` mit `kubectl kustomize` gerendert: Ingress-Klasse `traefik`, Dashboard-Replicas 2 identifiziert
- Beide Images gebaut, in den k3d-Cluster importiert und Overlay deployed
- Traefik per Port-Forward auf Port 8080 geöffnet
- Alle drei Routen (`/`, `/api/v1/snapshot`, `/health/ready`) über dieselbe Basisadresse aufgerufen – alle HTTP 200
- Load Balancing sichtbar gemacht: 20 Anfragen mit `Connection: close` gesendet, zwei unterschiedliche Dashboard-Pod-Namen in der Antwort beobachtet
- EndpointSlice des Dashboard-Services auf zwei IP-Adressen bestätigt
- Pod gelöscht und beobachtet wie der Dienst während des Ersatzes erreichbar bleibt (Zero-Downtime)

### Warum

In einer produktionsnahen Umgebung ist ein zentraler Ingress unerlässlich: Er vereinfacht das Routing, ermöglicht TLS-Terminierung an einem Ort und macht Load Balancing transparent sichtbar. Das Zero-Downtime-Verhalten durch Deployments ist ein Kernvorteil von Kubernetes.

---

## Block 05 – Messaging mit RabbitMQ

### Worum es geht

Im verteilten Betrieb (`APP_MODE=distributed`) kommunizieren die Dispatch-City-Komponenten nicht mehr direkt, sondern über **RabbitMQ** als Message Broker. Ein **Exchange** nimmt Nachrichten entgegen und leitet sie anhand von Routing Keys an **Queues** weiter. Verschiedene Worker subscriben auf ihre jeweiligen Queues.

Drei zentrale Messaging-Patterns wurden demonstriert:
- **Competing Consumers**: Mehrere Instanzen eines Workers teilen sich eine Queue
- **Dead Letter Queue (DLQ)**: Nicht verarbeitbare Nachrichten werden isoliert statt verloren
- **Event-driven Architecture**: Producer und Consumer sind entkoppelt

### Was gemacht wurde

**Aufgabe 1 – Overlay erkunden**
- Block-5-Repository geklont und Installationsskript ausgeführt
- Overlay `block-05-messaging` gerendert: `APP_MODE=distributed`, RabbitMQ-Image `rabbitmq:4.3.5-management-alpine`, vier Worker-Typen (customer-simulator, courier-simulator, restaurant-worker ×3, order-worker) identifiziert

**Aufgabe 2 – System deployen**
- Images gebaut und in k3d importiert (CRLF-Problem in Shell-Skripten mit `sed -i 's/\r//'` behoben)
- Overlay deployed, auf RabbitMQ und alle Workloads gewartet
- PVC `data-rabbitmq-0` mit Status `Bound` (1Gi) bestätigt
- Problem behoben: RabbitMQ-Liveness-Probe lief in Timeout (`rabbitmq-diagnostics` benötigt >1s) → `timeoutSeconds: 10` gesetzt

**Aufgabe 3 – Eventfluss beobachten**
- Traefik (Port 8080) und RabbitMQ Management UI (Port 15672) per Port-Forward geöffnet
- Dashboard im DISTRIBUTED-Modus bestätigt
- Eventfluss verfolgt: customer-simulator → Exchange `food.events` → order-worker → restaurant-Queue → restaurant-pizza → courier-Queue → courier-simulator → Lieferung abgeschlossen

**Aufgabe 4 – Competing Consumers**
- `restaurant-pizza` auf 0 Replicas skaliert (kein Consumer aktiv)
- 8 Events mit `lab-publish.sh valid 8` publiziert → Queue `restaurant.restaurant-pizza` zeigte 9 messages_ready (8 + 1 vom customer-simulator)
- `restaurant-pizza` auf 2 Replicas skaliert → beide Pods bauten den Rückstau gemeinsam ab → Queue: 0 messages_ready, 2 consumers

**Aufgabe 5 – Dead Letter Queue**
- DLQ geleert: `rabbitmqctl purge_queue food.dead`
- Ungültige Nachricht mit `lab-publish.sh invalid` publiziert
- Queue `food.dead` zeigte 3 Einträge: beide aktiven restaurant-pizza-Pods (2 Replicas) lehnten die Nachricht je einmal ab, zusätzlich 1 Eintrag vom order-worker
- `restaurant-pizza` auf 1 Replica zurückgesetzt

### Warum

RabbitMQ entkoppelt Produzenten und Konsumenten: Ein Producer muss nicht wissen, wer seine Nachrichten verarbeitet oder ob ein Consumer gerade verfügbar ist. Das ermöglicht:
- **Skalierung ohne Codeänderung**: Mehr Consumer-Pods = mehr Durchsatz
- **Fehlertoleranz**: Nachrichten bleiben in der Queue, bis sie verarbeitet werden
- **Isolation fehlerhafter Nachrichten**: DLQ verhindert, dass eine ungültige Nachricht den Worker in eine Endlosschleife treibt

---

## Block 06 – Helm, Operator und CloudNativePG

### Worum es geht

Stateful Workloads wie PostgreSQL benötigen mehr als ein einfaches Deployment: Daten müssen auf einem **PersistentVolume** überleben, und bei einem Ausfall muss automatisch ein neuer Primary gewählt werden. Der **CloudNativePG-Operator** übernimmt diese Aufgabe – er überwacht `Cluster`-Ressourcen und verwaltet Primary, Replica, Replikation und Failover selbstständig.

**Helm** wird genutzt, um den Operator als versioniertes Release im Cluster zu installieren. Statt roher Manifeste beschreibt ein **Chart** alle benötigten Kubernetes-Objekte, und eine `values.yaml` erlaubt umgebungsspezifische Anpassungen ohne den Chart-Code zu verändern.

Zentrale Konzepte:
- **Operator-Pattern**: Ein Controller beobachtet Custom Resources (`kind: Cluster`) und bringt den Cluster in den gewünschten Zustand
- **Idempotenz**: Doppelt zugestellte Nachrichten führen nicht zu doppelten Datenbankeinträgen, weil `processed_events` und Projektion in derselben Transaktion geschrieben werden
- **Failover**: Der `-rw`-Service zeigt immer auf den aktuellen Primary – Pod-Namen sind nach einem Failover irrelevant

### Was gemacht wurde

**Aufgabe 1 – Wunschzustand lesen**
- Block-6-Repository geklont (`v1.1.0`), Installationsskript ausgeführt, Overlay `block-06-persistence` ins Projekt kopiert
- Overlay mit `kubectl kustomize` gerendert und vier Prognosen begründet:
  - Instanzen: 2 (`spec.instances`)
  - Image: `ghcr.io/cloudnative-pg/postgresql:18.4-system-trixie` (`spec.imageName`)
  - Write-Service: `food-delivery-db-rw` (aus `Secret/database-url → stringData.url`)
  - Pod-Ersteller: CloudNativePG-Operator (`kind: Cluster`)
- `spec.affinity.podAntiAffinityType: required` festgestellt – beide DB-Pods müssen auf verschiedenen Nodes laufen

**Aufgabe 2 – Chart prüfen und Operator installieren**
- Helm via `winget` installiert, CNPG-Repo hinzugefügt (`helm repo add cnpg`)
- `helm show chart` und `helm show values` für Version `0.29.0` ausgeführt
- `platform/cloudnative-pg/values-course.yaml` mit den Defaults verglichen: Ressourcen explizit gesetzt (Default war `{}`), Monitoring deaktiviert
- Operator mit `./platform/cloudnative-pg/install.ps1` installiert
- Release `cnpg` in `cnpg-system` als `deployed` bestätigt, CRD `clusters.postgresql.cnpg.io` vorhanden

**Aufgabe 3 – Datenbank deployen und Service-Routing erklären**
- Images gebaut und in k3d importiert, Overlay `block-06-persistence` deployed
- Auf `cluster/food-delivery-db condition met` gewartet (beide Instanzen Ready)
- Service-Routing untersucht: `food-delivery-db-rw` → Primary (`food-delivery-db-1`, IP `10.42.2.44`), `food-delivery-db-ro` → Replica, `food-delivery-db-r` → alle Instanzen
- `database-migrate`-Job: Status `Completed`

**Aufgabe 4 – Persistenz nachweisen**
- Bestellung im Dashboard notiert, dann `order-worker` und `control-api` neu gerollt (`rollout restart`)
- Nach dem Rollout: Bestellung noch im Dashboard sichtbar, in PostgreSQL via `SELECT id,status FROM orders` bestätigt
- Erkenntnis: Persistenz liegt im PVC, nicht im Pod; die Applikation rekonstruiert die Anzeige beim Start aus der DB

**Aufgabe 5 – Idempotenz mit DB-Evidenz belegen**
- Prognose: beide Zähler müssen 1 ergeben, obwohl dieselbe Nachricht zweimal publiziert wird
- `./scripts/lab-idempotency.ps1` ausgeführt: `EVENT_ID` und `ORDER_ID` notiert
- `SELECT count(*) FROM processed_events WHERE event_id=...` → 1
- `SELECT count(*) FROM orders WHERE id=...` → 1
- Begründung: `processed_events` und Projektion werden in einer atomaren Transaktion gespeichert – ein Crash zwischen zwei getrennten Writes würde Idempotenz brechen

**Aufgabe 6 – Failover beobachten und messen**
- Vorher-Zustand erfasst: Primary `food-delivery-db-1` auf Node `k3d-teko-k8s-server-0`, `-rw`-Endpoint `10.42.2.44`
- Watch auf Pods (`-L cnpg.io/instanceRole`) und EndpointSlice gleichzeitig gestartet
- `food-delivery-db-1` mit `--grace-period=0 --force` gelöscht
- Beobachtung: `food-delivery-db-2` übernahm Primary-Rolle in ~2s, EndpointSlice wechselte von `10.42.2.44` auf `10.42.0.52`
- `food-delivery-db-1` wurde vom Operator als neue Replica neu gestartet
- Bestellung aus Aufgabe 4 nach Failover noch vorhanden ✅
- Fazit: Failover beweist Hochverfügbarkeit bei Pod-Ausfall, beweist aber **nicht** Backup/RPO – ein PVC-Verlust würde Datenverlust bedeuten

### Warum

PostgreSQL als StatefulSet ohne Operator zu betreiben ist fehleranfällig: Failover, WAL-Replikation und Primary-Promotion müssten manuell konfiguriert werden. Der CloudNativePG-Operator abstrahiert diese Komplexität hinter einer einzigen `Cluster`-Ressource. Helm stellt sicher, dass der Operator versioniert und reproduzierbar installiert wird – dasselbe Chart mit derselben `values.yaml` ergibt immer dasselbe Ergebnis. Die Kombination aus Operator, PersistentVolumes und Idempotenz-Pattern ergibt eine produktionsnahe Datenschicht, die Pod-Neustarts und Node-Ausfälle übersteht.

---

## Block 07 – Observability, Skalierung und HPA

### Worum es geht

Nach Messaging und Persistenz kommt der letzte entscheidende Schritt: **Beobachtbarkeit und automatisches Skalieren**. Ein System ist erst produktionsreif, wenn es nicht nur Daten verarbeitet, sondern auch **Metriken**, **Berechtigungs-/Health-Status**, **Fehlerzustände** und **Lastverhalten** sichtbar macht. Der **Horizontal Pod Autoscaler (HPA)** nutzt CPU- oder andere Metriken, um Replicas automatisch zu erhöhen oder zu reduzieren.

Die Kombination aus Prometheus, Grafana, Kubernetes-Metriken und HPA zeigt, wie ein System auf Last reagieren und dabei stabil bleiben kann.

### Was gemacht wurde

- Observer/Monitoring-Stack für das Cluster installiert und konfiguriert
- Metrics-Endpoints der Control-API untersucht
- Prometheus und Grafana als Beobachtungswerkzeuge genutzt
- `cluster-observer` und `grafana-dashboard` in den Overlays eingebunden
- HPA-Konfiguration für das `lab-web`-Deployment erstellt
- `minReplicas: 2`, `maxReplicas: 4`, `averageUtilization: 50` gesetzt
- `scaleDown.stabilizationWindowSeconds: 60` ergänzt, damit nicht zu schnell wieder skaliert wird
- Deployment `lab-web` mit einem nginx-Container und CPU-Requests/Limits definiert
- Lasttest mit `load.sh` bzw. `load.ps1` ausgelöst
- HPA-Verhalten im Watchmodus beobachtet: Pods stiegen unter Last an und fielen nach der Stabilisierung wieder ab
- Verfügbarkeit von Pods, CPU-Auslastung und Skalierungsreaktion dokumentiert

### Warum

Observability ist die Grundlage für zuverlässige Operations: Ohne Metriken kann man weder Lastspitzen erkennen noch das Verhalten eines Systems in realen Bedingungen beurteilen. Der HPA macht aus der Beobachtung eine automatische Reaktion: Bei hoher CPU-Auslastung werden mehr Pods gestartet, bei niedriger Last wieder reduziert. So wird die Infrastruktur elastisch und resilient.

---

## Abschluss: Was ich im Kurs gelernt habe

### Die wichtigsten Lerninhalte

- Docker und Containerisierung sind die Basis für jede moderne Anwendung
- Kubernetes orchestriert Container zuverlässig, automatisch und skalierbar
- Services, Deployments, Probes und Namespaces sind zentrale Bausteine für stabilen Betrieb
- Ingress und Load Balancing erlauben kontrollierten externen Zugriff
- Messaging mit RabbitMQ entkoppelt Producer und Consumer und verbessert Skalierung und Fehlertoleranz
- PostgreSQL mit CloudNativePG liefert Persistenz, Failover und Datenintegrität für Stateful Workloads
- Monitoring und Metriken machen das System messbar und verständlich
- HPA verbindet Metriken mit automatischer Reaktion auf Last

### Die Kernidee des Projekts

Das Projekt zeigt, wie aus einer einfachen Anwendung ein komplettes cloud-natives System wird:

- Frontend stellt Informationen dar
- API verarbeitet Anfragen
- Simulatoren und Worker erzeugen und verarbeiten Events
- Broker und Datenbank sichern Kommunikations- und Persistenzschicht
- Monitoring und Autoscaling sorgen für Betriebssicherheit und Elastizität

### Was besonders wichtig war

Die zentrale Erkenntnis war: Ein modernes verteiltes System ist nicht nur Software, sondern ein Zusammenspiel aus Architektur, Infrastruktur, Laufzeit-Phänomenen und Betriebsprozessen. Kubernetes ist dabei nicht nur ein Deployment-Tool, sondern ein komplettes Orchestrierungssystem, das Selbstheilung, Skalierung und robuste Betriebssicherheit ermöglicht.

---

## Abschlussbewertung

Die Lernreise über alle Kurstage war eine komplette Einführung in moderne, containerisierte Systemarchitektur:

- Grundlagen: Docker und Container
- Orchestrierung: Kubernetes und k3d
- Service-Architektur: Deployments, Services, Ingress
- Event-Driven Design: RabbitMQ
- Persistenz: PostgreSQL + CloudNativePG
- Betrieb: Observability, Metrics, HPA

Zusammen bilden diese Bausteine die Grundlage einer echten cloud-nativen Anwendung – genau so, wie sie in produktiven Systemen eingesetzt wird.

### Gesamtfazit

Die Entwicklung des Projekts zeigt, wie aus einer einfachen Anwendung über mehrere Lernschritte ein verteiltes, cloud-natives System entsteht. Die einzelnen Blöcke – von der Basisarchitektur über Messaging und Datenhaltung bis hin zur Skalierung und Überwachung – ergänzen sich zu einem durchgängigen Betriebskonzept. Der HPA-Teil ist dabei nur ein letzter, wichtiger Schritt in dieser Gesamtarchitektur: Er demonstriert, wie ein System unter Last automatisch reagieren und seine Ressourcen dynamisch anpassen kann.
