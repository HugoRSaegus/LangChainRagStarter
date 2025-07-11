# Contenu du repo

## step_by_step_rag.ipynb
Notebook donnant les étapes élémentaires pour obtenir la base d'un RAG
- Scraping de l'AI Act en Documents LangChain
- Préparation de l'Index Vectoriel Azure
- Appel programmatique à un LLM OpenAI déployé dans un AI Hub Azure
- Création d'un template de prompt
- Création d'un RAG basique avec langgraph
- Exploration de rag_simple.py et rag_chat.py
## globals.py
Variables globales et classes utilisées par les RAGs
## rag_simple.py - [Tutoriel](https://python.langchain.com/docs/tutorials/rag/)
Implémentation d'un RAG basique. 
- get_comiled_rag: Fonction qui retourne un CompiledStateGraph afin d'utiliser notre RAG dans une application streamlit. 
- Ce RAG ajoute à celui du notebook 
    - Une étape d'analyse de la question utilisateur afin d'affiner la requête envoyée à la base vectorielle.
    - Un ajout de citations à la réponse llm. 
## rag_chat.py - [Tutoriel](https://python.langchain.com/docs/tutorials/qa_chat_history/)
- get_comiled_rag: Fonction qui retourne un CompiledStateGraph afin d'utiliser notre RAG dans une application streamlit.
- Ce RAG 
    - Utilise MessageState plutôt qu'une classe d'états comme pour rag_simple.py
    - Passe dans **l'instruction système** le formatage des **citations** dans la réponse générer plutôt que de le faire **après la réponse générée**, ce qui permet de **streamer** la génération dans l'application streamlit.
    - Emploie **MemorySaver** afin de permettre au RAG de se référer aux **précédentes réponses générées**  
    - Définie **l'étape de recherche vectoriel comme un tool** ce qui permet au RAG de **choisir de l'utiliser ou non** plutôt que de forcément y passer comme dans rag_simple.py qui définit cette étape comme un état.

## Application Démonstrative 
### rag_app.py
Application streamlit qui emploie **rag_chat.py**. Les réponses sont streamées et le RAG, l'historique de conversation, est consultable

Lancer l'application avec la commande suivante
```bash 
streamlit run rag_app.py
```
### pages/rag_app_simple.py
page streamlit qui emploie **rag_simple.py**. 


---
# Composants d'un RAG 

<img src="https://codingscape.com/hs-fs/hubfs/llamaindex_rag_overview.png?width=1602&height=780&name=llamaindex_rag_overview.png" width="1000">

## Préparation de l'Index de Données via LangChain
### Données
&emsp;Vos données peuvent avoir une multitude de formats et de sources. Il faut s'assurer que chaque élément a une source associée pour qu'elle puisse être renvoyée avec les réponse du RAG.

Un format typique de données en entrée de RAG serait 

|Titre du document|Lien URL|Contenu|
|-|-|-|
|"Comment Faire un RAG.pdf"|"https://..."|"Pour faire un RAG..."|
|"RAG: Les Bonnes Pratiques.txt"|"https://..."|"Les bonnes pratiques..."|

Dans les notebooks d'exemple on utilisera l'AI act comme exemple de données à intégrer dans le RAG   

### [LangChain Documents](https://python.langchain.com/api_reference/core/documents/langchain_core.documents.base.Document.html)
&emsp;Avant de pouvoir vectoriser et indexer vos données, il faut les convertir en documents LangChain. Un document est un type de donnée qui est composé d'un identifiant, d'un dictionnaire de métadonnées et d'un string de contenu.

### [LangChain Chunking](https://python.langchain.com/api_reference/text_splitters/index.html)

&emsp;Le chunking consiste à diviser les documents en chunks plus petits, qui sont donc plus gérables pour la vectorisation. 

Selon la taille de votre contenu et l'objectif de votre RAG la taille des segments est un paramètre important.

- Petit segment => RAG avec une petite granularité capable de récupérer des détails fins de votre information mais pouvant perdre en visibilité sur le contexte de cette dite information. Idéal pour un RAG factuel, par exemple pour explorer un stock de magasin
- Gros segment => RAG avec une granularité large pouvant prendre en compte le contexte de l'information, résultant en de meilleures performances pour des tâches complexes de raisonnement.

Voir [ici](https://www.machinelearningplus.com/gen-ai/optimizing-rag-chunk-size-your-definitive-guide-to-better-retrieval-accuracy/#:~:text=Optimal%20chunk%20size%20for%20RAG%20systems%20typically%20ranges,tokens%29%20provide%20better%20context%20for%20complex%20reasoning%20tasks.) pour un deep dive dans la configuration de vos chunks.

### Vectorisation ([Text Embedding](https://www.geeksforgeeks.org/nlp/what-is-text-embedding/))
&emsp;La vectorisation est l'étape où l'on transforme l'ensemble des chunks de texte en vecteurs dans un espace à haute dimension où des mots ou phrases similaires se trouvent à proximité les unes des autres.

Ce process est fait via des modèles de vectorisation comme BERT ou ada2.

### Indexation
&emsp;L'indexation consiste à organiser les vecteurs pour permettre une recherche efficace.

## Application RAG
### Question Utilisateur
&emsp;Comme avec un chatbot classique, l'utilisateur va commencer par poser une question. Celle-ci va alors être traitée par 2 composants.
### Chercheur d'Index
&emsp;Le chercheur va utiliser la question utilisateur pour récupérer les informations de la base vectorielle indexée. Il peut reformuler ladite question pour obtenir de meilleurs résultats de l'index.
<p style="line-height: 0;">Il formatera alors les documents récupérés dans une sortie souvent appelée contexte qui sera ensuite utilisée par le générateur de réponse.</p>

### Générateur de Réponse
&emsp;Le générateur de réponse est le plus souvent un modèle LLM classique à qui on donne la question utilisateur ainsi que le contexte récupéré de l'index. 
<p style="line-height: 0;">La question peut alors être formatée pour inclure des liens cliquables vers les documents passés en référence.</p>

## Les concepts LangChain employés
- Vector Stores
    - [Document](https://python.langchain.com/api_reference/core/documents/langchain_core.documents.base.Document.html)
    - [DocumentLoader](https://python.langchain.com/docs/integrations/document_loaders/)
    - [AzureAISearch](https://python.langchain.com/docs/integrations/vectorstores/azuresearch/)
    - [AzureAISearchRetriever](https://python.langchain.com/docs/integrations/retrievers/azure_ai_search/)
    - [AzureEmbedding](https://python.langchain.com/docs/integrations/text_embedding/azureopenai/#setup)
    - [Azure OData Filtering](https://learn.microsoft.com/en-us/azure/search/search-query-odata-filter)
- ChatCompletion
    - [AzureChatOpenAI](https://python.langchain.com/api_reference/openai/chat_models/langchain_openai.chat_models.azure.AzureChatOpenAI.html)
    - [Streaming Answers](https://python.langchain.com/docs/how_to/chat_streaming/)
- RAG App
    - [StateGraph](https://pypi.org/project/langgraph/)
    - [PromptTemplate](https://python.langchain.com/api_reference/core/prompts.html)
    - [Chat Interface with Message Chaining](https://python.langchain.com/docs/tutorials/qa_chat_history/)
    - [Document References](https://python.langchain.com/docs/how_to/qa_citations/)
    - [Structuring LLM Outputs](https://python.langchain.com/docs/how_to/structured_output/)
    - [Agent Tools](https://python.langchain.com/docs/how_to/custom_tools/)

## Ressources Azures

Bien qu'il existe des alternatives gratuites, les ressources nécessaires à la création d'un RAG sont accessibles via Azure. Pour y avoir accès, contactez Julien Ayral-**julien.ayral@saegus.com**. Expliquez-lui dans votre mail le projet qui justifie la demande de ressource. 

Le groupe de ressources auquel vous devez être assigné est **SAEGUS_CHATGPT** et les rôles qui vous procureront les droits nécessaires sont **Contributeur Cognitive Services** et **Collaborateur**.

Une fois vos accès obtenus, vous devriez les activer quotidiennement[ici](https://portal.azure.com/?feature.msaljs=true#view/Microsoft_Azure_PIMCommon/ActivationMenuBlade/~/azurerbac/provider/azurerbac).



Les LLM tels que les GPT de **AzureOpenAI** et les **modéles de Vectorisation de données** nous sont accessibles via [Azure AI Foundry](https://ai.azure.com/).

Sur la page d'accueil, vous pouvez **créer une Ressource de hub IA**. Avec celle-ci, vous seriez alors en mesure de **déployer des LLM et des modèles de vectorisation** que vous pourriez alors utiliser par **appel API**.

Pour la création d'index vectoriel, il vous suffit de récupérer la **clef API** d'une ressource Azure AI Search préexistante.  

Pour pouvoir exécuter le code des notebooks démonstratifs, compléter le fichier **.env** avec les informations relatives.

### Variables pour OpenAI
- AZURE_OPENAI_API_ENDPOINT
- AZURE_OPENAI_API_VERSION
- AZURE_OPENAI_EMBEDDING_MODEL
- AZURE_OPENAI_DEPLOYMENT_ID (nom du modèle llm à utiliser pour répondre aux questions)
- AZURE_OPENAI_API_KEY

### Variables pour Azure AI Search
- SEARCH_ENDPOINT = "https://<nom_du_service>.search.windows.net" (https://ai102srch1716175.search.windows.net est le endpoint de la ressouce utilisé pour ce tutoriel)
- SEARCH_KEY ([clef API de la ressource](https://portal.azure.com/#@saegus.com/resource/subscriptions/1091ae17-09a6-4df2-875a-14e064254ff7/resourceGroups/SAEGUS_CHATGPT/providers/Microsoft.Search/searchServices/ai102srch1716175/Keys))
- SEARCH_INDEX = **lang_rag_index_2** 