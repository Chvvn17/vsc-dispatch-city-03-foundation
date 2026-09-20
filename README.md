# Dispatch City

Dieses Repository enthält ein iteratives, cloud-natives Food-Delivery-Projekt, das über mehrere Unterrichtsblöcke hinweg aufgebaut wurde. Ziel ist die Umsetzung einer verteilten Anwendung auf Kubernetes mit Frontend, API, Messaging, Persistenz, Observability und Autoscaling.

Das Projekt zeigt den gesamten Aufbau einer modernen Systemarchitektur: Von der ersten lauffähigen Grundlage über die Integration von RabbitMQ und PostgreSQL bis hin zur Überwachung und automatischen Skalierung mit einem Horizontal Pod Autoscaler.

Das Repository stellt damit kein einzelnes Lab dar, sondern die schrittweise Entwicklung eines kompletten, verteilten Systems. Die einzelnen Blöcke bauen logisch aufeinander auf und ergänzen sich zu einer Gesamtarchitektur aus Infrastruktur, Services, Datenhaltung und Betriebsfähigkeit.

## Projektziel

Die Anwendung modelliert eine kleine Stadt mit Restaurants, Kunden, Kuriere, Bestellungen und Zustandsänderungen. Die visuellen Zustände werden im Dashboard dargestellt, während die Daten und Ereignisse im Hintergrund durch mehrere Services verarbeitet werden.

## Überblick über die Architektur

Die Anwendung besteht aus mehreren Komponenten, die zusammen ein verteiltes System bilden:

- Frontend / Dashboard: Nuxt + PixiJS zeigt die Stadt und den aktuellen Betriebszustand
- Control API: REST-, SSE-, Health- und Metrics-Endpunkte
- Simulations- und Worker-Komponenten: Kunden, Kuriere, Bestellungen und Restaurant-Logik
- Messaging: RabbitMQ übernimmt die Kommunikation zwischen Services
- Persistenz: PostgreSQL speichert Daten und Zustände dauerhaft
- Observability: Metriken, Healthchecks und Visualisierung über Prometheus/Grafana
- Autoscaling: HPA überwacht die CPU-Auslastung und skaliert Pods automatisch

## Systemaufbau

![Software Architektur](/software_architektur.png)

Gesamtarchitektur der Dispatch-City-Anwendung mit Frontend, Control API, Messaging, Persistenz, Observability und automatischer Skalierung durch den Horizontal Pod Autoscaler.

## Verlauf des Projekts

Das Repository wurde über mehrere Blöcke hinweg erweitert:

- Block 03: Basis-Setup, Kubernetes-Foundation und erste lauffähige Anwendung
- Block 05: Messaging mit RabbitMQ
- Block 06: Persistenz mit PostgreSQL und Migrations-/Repository-Schicht
- Block 07: Observability und Autoscaling mit HPA

Damit bildet das Projekt einen durchgängigen Lernpfad von einfacher Containerisierung bis zur skalierbaren cloud-nativen Anwendung.

## HPA-Teil (Block 07)

Ein zentraler Teil des Projekts ist das skalierbare Web-Deployment im HPA-Lab:

- Namespace: `betrieb-lab`
- Deployment: `lab-web`
- MinReplicas: 2
- MaxReplicas: 4
- CPU-Ziel: 50%
- ScaleDown-Stabilisierung: 60s

Konfigurationsdateien:

- `labs/block-07/hpa.yaml`
- `labs/block-07/web.yaml`
- `labs/block-07/load.sh`
- `labs/block-07/load.ps1`

## Lokaler Start

Voraussetzungen:

- Go
- Node.js / npm
- Docker oder Container Runtime
- Kubernetes-Cluster (z. B. k3d)

Beispiel:

```bash
go test -race ./...
cd apps/dashboard && npm install && npm run typecheck && cd ../..
make images load deploy-03
```

Für die HPA-Demo:

```bash
kubectl --context k3d-teko-k8s apply -f labs/block-07/web.yaml
kubectl --context k3d-teko-k8s apply -f labs/block-07/hpa.yaml
kubectl --context k3d-teko-k8s -n betrieb-lab get hpa -w
```

Last erzeugen:

```bash
./labs/block-07/load.sh
```

Oder unter Windows:

```powershell
.\labs\block-07\load.ps1
```

## Erwartetes Verhalten

Bei steigender CPU-Auslastung sollte der HPA automatisch zusätzliche Pods starten, und nach Ende der Last wieder auf den minimalen Wert skalieren. Damit wird das Verhalten eines cloud-native, elastischen Systems im Kubernetes-Cluster nachgewiesen.

## Lernziele des Gesamtprojekts

Das Projekt demonstriert die wichtigsten Schwerpunkte einer modernen cloud-nativen Anwendung:

- Containerisierung und Deployment
- Service-Interaktion über API und Messaging
- Zustandsverwaltung und Persistenz
- Monitoring und Observability
- Resilience und Self-Healing
- Automatisches Horizontal Scaling

## Abschlussbemerkung

Das Repository stellt nicht nur ein einzelnes Lab dar, sondern die komplette Entwicklung eines verteilten, skalierbaren Systems. Der HPA-Teil ist dabei nur einer der letzten Schritte auf dem Weg zu einer echten cloud-nativen Betriebsarchitektur.
