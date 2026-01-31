# Système de Surveillance de Pression Artérielle (FHIR, Kafka, Elastic Stack, ML)

## Description du Projet
Ce projet implémente une architecture Big Data en temps réel pour la surveillance clinique de la pression artérielle. Il utilise le standard FHIR (Fast Healthcare Interoperability Resources) pour la modélisation des données et combine des règles cliniques strictes avec un modèle de Machine Learning pour détecter les anomalies de santé chez les patients.

## Objectifs principaux :
* Générer des données médicales réalistes au format FHIR.
* Transmettre et traiter les flux de données en temps réel via Kafka.
* Analyser les mesures par seuils médicaux et par prédiction statistique (Régression Logistique).
* Stocker les anomalies dans Elasticsearch pour visualisation dans Kibana.
* Archiver les cas normaux localement au format JSON.

## Architecture Technique
Le système repose sur plusieurs composants conteneurisés et scripts Python :

1.	Générateur (Producer) : Crée des observations FHIR fictives et les envoie dans le topic Kafka blood-pressure.
2.	Pipeline de données (Kafka) : Gère le flux de messages entre le producteur et le consommateur.
3.	Analyseur (Consumer) : Récupère les messages, extrait les données, applique les règles de détection et le modèle ML, puis oriente les données vers le 
stockage approprié.
4.	Stockage & Visualisation : Elasticsearch indexe les alertes (bp-anomalies) et Kibana permet de créer des dashboards de suivi.

## Prérequis
* Docker et Docker Compose
* Python 3.10+
* Bibliothèques Python nécessaires :

```
Bash
pip install kafka-python elasticsearch joblib scikit-learn faker
```

## Installation et Utilisation
1. Lancer l'infrastructure (Docker)
Démarrez les services Kafka, Zookeeper, Elasticsearch et Kibana :

```
Bash

``docker-compose -f kafka-docker-compose.yml up -d``

``docker-compose -f elastic-kibana-docker-compose.yml up -d``
```

2. Entraîner le modèle de Machine Learning
Avant de lancer l'analyse, générez le modèle de classification (Normal vs Anormal) :

```
Bash
``python train_model.py``
```

Ceci créera un fichier bp_logreg.pkl utilisé par le consumer pour prédire la probabilité d'anomalie.

3. Lancer le Consumer (Analyseur)
Le consumer attend les messages de Kafka, les traite et les indexe dans Elasticsearch :

Bash
``python consumer.py``

4. Lancer le Producer (Générateur de données)
Le producer commence à envoyer des mesures de pression artérielle aléatoires :

```
Bash
``python producer.py``
```

## Logique de Détection des Anomalies
Le système utilise deux couches de vérification dans consumer.py :

*	Seuils Cliniques :
 - Systolique : > 140 mmHg ou < 90 mmHg.
 -	Diastolique : > 90 mmHg ou < 60 mmHg.
*	Modèle ML (Régression Logistique) : Calcule une probabilité d'anomalie basée sur l'entraînement préalable du modèle sur des milliers de cas synthétiques.
Catégorisation (Classification FHIR)

Les mesures sont classées selon les standards de santé : Normal, Elevated, Hypertension Stage 1, Hypertension Stage 2, et Hypertensive Crisis.
Visualisation (Kibana)

Une fois le système en marche, accédez à Kibana via http://localhost:5601 :

1.	Créez un Index Pattern sur bp-anomalies.
2.	Configurez des dashboards pour visualiser :
 - Le volume d'anomalies par patient.
 - La répartition des types d'hypertension/hypotension.
 - L'évolution temporelle des crises hypertensives.

## Structure des fichiers
* ``producer.py`` : Génération FHIR et envoi Kafka.
* ``consumer.py`` : Logique d'analyse, ML et indexation ES.
* ``train_model.py`` : Script d'entraînement pour la régression logistique.
* ``kafka-docker-compose.yml`` : Configuration Zookeeper et Kafka.
* ``elastic-kibana-docker-compose.yml`` : Configuration de la stack Elastic.
* ``bp_logreg.pkl`` : Modèle ML sauvegardé (généré après entraînement).
