# Docker Agent — exercices d’évaluation

Ce projet est un exercice réalisé dans le cadre du cours **MBA IA & Data Innovation**. Il explore une question centrale : **comment évaluer qu’un agent choisit le bon outil et l’utilise correctement pour répondre à une demande ?**

L’évaluation ne porte donc pas uniquement sur la réponse finale. Elle vise aussi à vérifier que l’agent sélectionne l’outil adapté, lui transmet les bons paramètres et s’appuie sur le résultat obtenu pour formuler sa réponse.

## Exercices

Trois scénarios sont prévus :

1. **Compter les fichiers** — `file-list-1.json`  
   Demander à l’agent de compter les fichiers dans un dossier et vérifier qu’il utilise l’outil de parcours du répertoire avant de donner un décompte exact.

2. **Gérer une liste de tâches** — `todo-eval-2.json`  
   Demander à l’agent de gérer une todo list et vérifier qu’il utilise les outils appropriés pour effectuer les opérations demandées.

3. **Appeler une API** — `api-call-3.json`  
   Demander à l’agent de récupérer des informations via une API et vérifier qu’il effectue l’appel attendu et exploite correctement la réponse.

Chaque exercice décrit une interaction à évaluer, notamment la demande de l’utilisateur, l’action attendue de l’agent et les critères permettant de juger sa réponse.

## Configuration de l’agent

La configuration de l’agent se trouve dans [`agent.yaml`](agent.yaml). Les scénarios d’évaluation sont regroupés dans le dossier [`evals/`](evals/).

## Exécuter les évaluations

Installez Docker Agent (inclus dans Docker Desktop 4.63 ou version ultérieure) et vérifiez que Docker Desktop est démarré, que Docker Model Runner est activé et que les modèles `qwen3:4b` (agent) et `mistral:latest` (juge) sont disponibles localement (`docker model pull ai/qwen3:4b` et `docker model pull ai/mistral`). Vérifiez les modèles installés avec :

```powershell
docker model ls
```

Vous pouvez aussi utiliser un modèle distant pris en charge par Docker Agent en modifiant `agent.yaml` pour sélectionner son fournisseur et en configurant les identifiants requis par ce fournisseur. Les clés API ne sont pas nécessaires pour les modèles locaux servis par Docker Model Runner.

Les évaluations s’exécutent dans un conteneur isolé. Le fournisseur Docker Model Runner natif (`dmr/...`) essaie de découvrir les modèles avec la commande `docker`, qui n’est pas disponible dans ce conteneur. Le flavor `eval-container` utilise donc l’API OpenAI-compatible de Docker Model Runner à l’adresse `http://model-runner.docker.internal/engines/v1`, accessible depuis les conteneurs Docker Desktop. Cette adresse interne est à utiliser depuis le conteneur ; le réglage TCP `127.0.0.1:12434` concerne les applications exécutées sur l’hôte et n’a pas besoin d’être exposé sur toutes les interfaces réseau.

Depuis PowerShell, placez-vous à la racine du projet :

```powershell
cd C:\Solutions\DockerAgentsTest
```

Le juge utilise le modèle local `mistral:latest`. Comme l’évaluation ne reconnaît pas le fournisseur personnalisé `dmr_http` pour le juge, configurez l’endpoint OpenAI-compatible de Docker Model Runner sur l’hôte et une clé factice (Docker Model Runner n’en exige pas) :

```powershell
$env:OPENAI_BASE_URL = "http://localhost:12434/engines/v1"
$env:OPENAI_API_KEY = "not-needed"
```

Puis lancez les trois scénarios :

```powershell
docker agent eval .\agent.yaml .\evals -c 2 --flavor eval-container --judge-model openai/ai/mistral:latest
```

Si l’erreur `exec: "docker": executable file not found in $PATH` apparaît encore, vérifiez que la commande inclut bien `--flavor eval-container`. Si le modèle agent échoue ensuite au démarrage avec une erreur propre au modèle ou à `llama.cpp`, la connexion à Model Runner fonctionne : testez le modèle séparément avec `docker model run ai/qwen3:4b "Réponds OK"` et vérifiez que Docker Desktop et Docker Model Runner sont à jour.

Docker Agent affiche les résultats dans le terminal et les enregistre dans `evals/results/`. Les évaluations nécessitent que les outils attendus par les scénarios soient configurés et disponibles pour l’agent dans `agent.yaml` : parcours de répertoire, todo et appel API.

Pour l’évaluation du comptage, `working_dir` désigne un dossier isolé sous `evals/working_dirs/`, et non la racine du projet. Le dossier `evals/working_dirs/my-directory/` contient les deux fichiers de test attendus (`file1.txt` et `file2.txt`).

## Documentation

Le projet s’appuie sur [Docker Agent](https://docs.docker.com/ai/docker-agent/), un framework open source permettant de configurer et d’exécuter des agents IA avec leurs instructions et leurs outils. La documentation de [Docker Model Runner](https://docs.docker.com/ai/model-runner/api-reference/) détaille les URL d’accès depuis l’hôte et depuis les conteneurs.
