# Séance 6 - GenAI et MLOps

Dans cette séance nous allons montrer comment MLFlow peut également s'utiliser pour suivre et évaluer les performances de LLMs.

## Utiliser l'API d'un LLM.

### Informations de connexion

Tout d'abord il vous faut récupérer les informations nécessaires à l'authentification sur le server Azure OpenAI.

Téléchargez ce fichier:  https://drive.google.com/file/d/1yVjfWhSY73vpsHSj6ZKYSWrWJoKMRkjH/view?usp=sharing

Il est chiffré,vous devrez le déchiffrer au moyen de la commande gpg et de la clef qui vous sera fournie.

Il contient _endpoint_ et la _clef_ vous permettant de vous connecter et de vous authentifier. Stockez ces deux valeurs dans des variables d'environnement dans un fichier .env, puis chargez les au moyen de la bibliothèque _dotenv_.

Stockez les également dans une variable d'environnement.

### Connexion à l'API

Créez une fonction `llm_query` qui prends en entrée une question posée et renvoie la réponse fournie par le LLM dont les informations de connexions sont passées en variable d'environnement.

Pour cela, vous devrez:
- Instancier l'objet `openAI.AzureOpenAI` qui gerera la connexion. Consultez sa documentation.
- Utiliser la méthode `chat.completions.create` de cet objet pour requêter le LLM
- Extraire de la réponse le contenu textuel effectif

Consultez les documentations de la classe 

Testez votre fonction sur une question simple.

## Evaluation manuelle des performances du LLM.

Comme pour un modèle classique, nous souhaiton être capable d'évaluer les performances d'un LLM, de le comaprer à un autre, de garantir la non-regression en cas d'update.

Evaluer les performances d'un LLM est une tâche problématique. Il faut bien sur des données de références, c'est à dire des questions et des réponses dont on est sur de la validité, et pertinentes pour le cas d'usage, mais également la capacité à juger de la qualité de la production du LLM comaprée à la réponse de référence.

Une solution évidente consiste à valider manuellement les résultats obtenus. En pratique, quand le nombre de résultats est trop important, une possibilité est de faire un échantillonage pour évaluer la proportion des tâches pour lesquelles les résultats du LLM sont satisfaisant.

Dans cette section nous allons générer un tel fichier permettant de comparer les bonnes réponses et les réponses du LLM et il sera possible de les consulter et de les comparer dans un tableur.

Vous devez:
- Charger le fichier `benchmark.csv` (en utilisant `pandas.read_csv`), qui contient des questions et des réponses correctes. Lien: https://drive.google.com/file/d/1-yM0PGj04rhurJy5CmDRZ4s_ERiqFoUA/view?usp=sharing
- Obtenir les réponses du LLM à chaque question
- Produire un fichier CSV similaire à `benchmark.csv` mais contenant également les réponses du LLM que vous pourrez charger dans un tableur.

## Utiliser MLFlow pour évaluer le LLM.

MLFlow propose depuis la version 3 la possibilité d'évaluer automatiquement un LLM à partir d'un fichier du type `benchmark.csv`. 
L'idée et de juger un LLM _A_ au moyen d'un LLM "juge" _B_ dont le rôle est d'évaluer la proximité entre la réponse fournie par _A_ et la bonne réponse. On fait l'hypothèse raisonnable que le travail de _B_ consistant à juger de la qualité de la réponse de _A_ étant donné la bonne réponse est beaucoup plus simple que de déterminer la bonne réponse, et donc que _B_ peut être considéré comme fiable.

L'utilisation de MLFlow nous permettra également de logger toutes les informations intéressantes et les métriques dans le server MLFlow pour consultation ultérieure.

### Configuration du MLFlow

Tout d'abord vous devez lancer votre server MLFlow en executant en ligne de commande, au même endroit que pour les TP précédent:
```
mlflow server
```

Nous allons demander à MLFlow d'utiliser un LLM "juge", distinct du LLM que nous allons tester, et que nous devons configurer. Cette configuration doit se faire par les variables d'environnement standards OPENAI_API_BASE, OPENAI_API_KEY et OPENAI_API_VERSION.

Les deux dernières sont déjà configurée, pour la première vous devez l'ajouter dans votre fichier `.env` et sa valeur est:

```
OPENAI_API_BASE="<your-endpoint>/openai/deployments/gpt-4.1-nano"
```

Ou `<your_endpoint>` est la valeur de AZURE_OPENAI_ENDPOINT.

### Transformation du dataset Benchmark

MLFlow attend un dataframe structuré de manière bien spécifique pour effectuer ses tests.
Consultez la documentation: https://mlflow.org/docs/latest/api_reference/python_api/mlflow.genai.html (en particulier la documentation de `mlflow.genai.evaluate`) et transformez le dataframe que vous avez chargé depuis `benchmark.csv` en dataframe pandas au bon format.

### Logging MLFlow

Comme vous l'avez fait dans les précédents TP, activez autolog, démarrez un experiment et un run MLFlow. Vous pourrez également logguer à la main des informations qui vous sembleront pertinentes.

### Evaluation MLFlow

Il vous reste à utiliser la fonction `mlflow.genai.evaluate` pour lancer les tests MLFlow. Comme scorers, utilisez au moins `Correctness`, qui correspond à la vérification de cohérence entre la réponse forunie par le LLM et la vraie réponse fournie dans le Benchmark.

Vous pourrez ensuite vous connecter au server MLFlow pour visualiser les métriques liées à vos expériences.

### Comparaison de LLM

Relancez l'ensemble de vos tests avec le déploiement `gpt-4.1-mini` à la place de `gpt-4.1-nano`. Vous pourrez ensuite aller dans l'interface MLFlow pour comparer les performance. __Attention__ changez le modèle à tester, utilisé dans la fonction `predict`, pas celui du LLM juge qui doit rester `gpt-4.1-nano`



    

