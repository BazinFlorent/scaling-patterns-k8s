# HPA Metrics POC

**Démonstrateur d'autoscaling Kubernetes piloté par des métriques applicatives et externes, sans KEDA.**

## Objectif

Démontrer la capacité du *Horizontal Pod Autoscaler* (HPA) à ajuster dynamiquement le nombre de replicas d'une application Kubernetes à partir de **Custom Metrics** et d'**External Metrics**, en complément des métriques traditionnelles CPU et mémoire.

Le POC permet d'expérimenter différents signaux d'autoscaling, de simuler des montées en charge et d'observer les décisions prises par Kubernetes.

## Architecture

```mermaid
flowchart LR
    subgraph A["📦 Application"]
        direction TB
        K6["⚡ k6<br/>Charge HTTP"]
        APP["🖥️ FastAPI<br/>Application instrumentée"]
        MET["📊 /metrics<br/>Compteurs et histogrammes"]
        K6 -->|"Requêtes HTTP"| APP
        APP -->|"Expose"| MET
    end

    subgraph O["🔭 Observabilité"]
        direction TB
        PROM["📈 Prometheus<br/>Collecte et PromQL"]
        AD["⚙️ Prometheus Adapter<br/>RPS par Pod"]
        PROM -->|"Interrogation PromQL"| AD
    end

    subgraph K["☸️ Kubernetes"]
        direction TB
        API["🔌 Custom Metrics API"]
        HPA["📈 HPA<br/>Calcul des replicas"]
        DEP["📦 Deployment / ReplicaSet"]
        API -->|"RPS par Pod"| HPA
        HPA -->|"Replicas désirés"| DEP
    end

    MET -->|"Métriques collectées<br/>en Pull par Prometheus"| PROM
    AD -->|"Implémente Custom Metrics"| API

    style A fill:#e8f2ff,stroke:#3b82f6,color:#172b4d
    style O fill:#e9f8ee,stroke:#27a363,color:#172b4d
    style K fill:#f3edff,stroke:#8055d4,color:#172b4d
```

Le schéma présente le **flux fonctionnel** de gauche à droite : Prometheus lance un `GET /metrics` (Pull), l'Adapter fournit les valeurs à l'API Kubernetes, puis le HPA met à jour le nombre de replicas du Deployment. Le ReplicaSet crée ou supprime alors les Pods. Les appels détaillés sont décrits dans la [documentation d'architecture](docs/03-architecture.md).

**Stack actuelle :** Kubernetes (kind), FastAPI, Prometheus, Prometheus Adapter, Metrics Server, HPA (`autoscaling/v2`) et k6.

> Le diagramme représente la chaîne **Custom Metrics** actuellement opérationnelle. Les variantes External Metrics et OpenTelemetry font partie des évolutions à expérimenter.

## Documentation

| Document | Contenu |
|---|---|
| [Installation](docs/01-installation.md) | Construction et validation reproductibles de l'environnement |
| [Démonstration](docs/02-demonstration.md) | Scénarios de charge, configuration HPA et observations |
| [Architecture](docs/03-architecture.md) | Mécanismes techniques, choix d'architecture et limites |

Les protocoles, mesures et conclusions des prochaines campagnes seront regroupés dans `docs/experiences/` (répertoire à créer).

## État d'avancement

| Jalon | État | Issues associées |
|---|---|---|
| Application FastAPI instrumentée | ✅ Validé | Socle initial |
| Collecte des métriques par Prometheus | ✅ Validé | Socle initial |
| Exposition des Custom Metrics à Kubernetes | ✅ Validé | Socle initial |
| Autoscaling HPA piloté par les RPS | ✅ Validé | Socle initial |
| Calibrage de la capacité applicative | ⬜ À réaliser | [#1](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/1), [#3](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/3) |
| Comparaison avec le HPA CPU | ⬜ À réaliser | [#1](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/1), [#2](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/2) |
| Scénarios de charge reproductibles (k6 / Locust) | 🟡 En cours | [#3](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/3) |
| Autoscaling sur External Metrics | ⬜ À réaliser | [#7](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/7) |
| Instrumentation et collecte alternatives (OpenTelemetry / eBPF) | ⬜ À réaliser | [#4](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/4), [#5](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/5), [#6](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/6), [#9](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/9) |
| Documentation et traçabilité expérimentale | 🟡 En cours | [#8](https://github.com/BazinFlorent/scaling-patterns-k8s/issues/8) |

**Périmètre :** démonstrateur technique local, destiné à l'expérimentation et à la validation de concepts d'architecture.
