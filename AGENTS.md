# Gasoil Around Me — instructions du projet

## Objectif produit

Construire une application Android destinée au marché français qui aide l’utilisateur à trouver les stations-service à proximité et à comparer leurs prix. La première version est un prototype gratuit, sans publicité, limité à la France.

L’écran principal doit s’inspirer des trois captures d’écran d’Essence&CO fournies par l’utilisateur :

- une carte interactive en arrière-plan ;
- des marqueurs de stations synchronisés avec la liste ;
- des commandes de sélection du carburant et de tri au-dessus de la liste ;
- un bottom sheet Material 3 déplaçable affichant la station, la ville, la distance à vol d’oiseau, le prix du carburant sélectionné et la fraîcheur du prix ;
- une fiche détaillée affichant tous les prix disponibles, l’adresse, les horaires et services lorsqu’ils sont disponibles, ainsi qu’une action de navigation.

Utiliser les captures comme source d’inspiration pour l’expérience utilisateur, et non comme des ressources à copier. Éviter la publicité intrusive et toute surcharge visuelle inutile.

Les fichiers de référence visuelle sont placés à la racine du projet :

- `1788170251615.jpg` : carte et marqueurs de prix colorés ;
- `Screenshot_2026-08-29-19-02-17-872_com.ripplemotion.android.EssenceLite.jpg` : carte, filtres et liste des stations ;
- `Screenshot_2026-08-29-19-03-19-708_com.ripplemotion.android.EssenceLite.jpg` : fiche détaillée d’une station.

Ces fichiers sont uniquement des références de conception. Ne pas les intégrer dans l’APK et ne pas reproduire les publicités ou textes promotionnels visibles sur les captures. Les logos d’enseignes peuvent être intégrés séparément dans les conditions définies ci-dessous, à partir de ressources propres et correctement sourcées.

## Périmètre confirmé du MVP

- Android uniquement, en Kotlin avec Jetpack Compose et Material 3.
- SDK minimum 26 ; conserver l’identifiant d’application existant `com.francotte.gasoilaroundme`.
- Google Maps SDK for Android pour afficher la carte.
- Localisation au premier plan uniquement. Ne jamais demander la localisation en arrière-plan.
- Si la localisation est indisponible ou refusée, proposer une recherche par ville ou adresse.
- Récupérer directement les stations et les prix depuis l’API officielle du gouvernement français ; aucun backend pour le prototype.
- Calculer localement les distances à vol d’oiseau avec la formule de Haversine. Ne pas appeler de matrice d’itinéraires pour les éléments de la liste.
- Ouvrir une application de navigation installée au moyen d’un intent Android pour guider l’utilisateur vers la station sélectionnée.
- Aucun compte, aucune authentification, aucune publicité, aucun suivi analytique, aucun pistage, aucun paiement et aucune notification push dans le MVP.

## Nomenclature des carburants

Utiliser les choix suivants dans l’interface et les associer explicitement aux champs des données officielles :

- Gazole
- SP95-E5
- E10
- SP98-E5
- E85
- GPL

Ne pas fusionner SP95 et SP98 dans un choix générique `E5`. Conserver localement le carburant sélectionné si cela ne nécessite pas l’ajout d’une infrastructure superflue.

## Services externes de référence

### Stations-service et prix

Utiliser le jeu de données officiel `prix-des-carburants-en-france-flux-instantane-v2` de `data.economie.gouv.fr`, au moyen de l’API Opendatasoft Explore v2.1.

Page du jeu de données :

`https://data.economie.gouv.fr/explore/dataset/prix-des-carburants-en-france-flux-instantane-v2/`

URL de base du point d’accès aux enregistrements :

`https://data.economie.gouv.fr/api/explore/v2.1/catalog/datasets/prix-des-carburants-en-france-flux-instantane-v2/records`

Considérer le schéma reçu comme une entrée réseau non fiable :

- accepter les prix, coordonnées, dates de mise à jour, horaires et services absents ou nuls ;
- exclure les coordonnées invalides ;
- conserver l’identifiant officiel de la station ;
- représenter explicitement les ruptures temporaires et définitives ;
- afficher la fraîcheur du prix et ne jamais laisser croire qu’un ancien prix est actuel ;
- produire un libellé de station raisonnable à partir des données officielles disponibles, sans inventer une enseigne.

Ne pas scraper de sites commerciaux de prix des carburants et ne pas ajouter une autre source de prix ou de stations sans décision produit explicite.

### Recherche d’une adresse ou d’une ville

Utiliser le service officiel actuel de géocodage de la Géoplateforme/IGN. Ne pas utiliser l’ancien point d’accès `api-adresse.data.gouv.fr`, dont l’arrêt était prévu en janvier 2026.

Temporiser les recherches saisies, imposer une longueur minimale pertinente, limiter les résultats à la France et éviter une requête réseau à chaque caractère.

### Carte

Utiliser uniquement `Maps SDK for Android` pour ce prototype. Ne pas activer ni intégrer Google Places, Routes, Directions, Distance Matrix ou Google Geocoding sans extension explicite du périmètre par l’utilisateur.

## Secrets et sécurité Google Maps

L’utilisateur conserve la clé de développement dans le fichier racine `local.properties` sous la forme `MAPS_API_KEY=...`.

Exigences :

- ne jamais lire, afficher, journaliser, copier, exposer ou versionner la clé ;
- ne jamais placer la clé en clair dans un fichier Kotlin, XML ou Gradle, dans les tests, les captures d’écran ou la documentation ;
- l’injecter dans le manifeste avec Google Maps Platform Secrets Gradle Plugin ;
- utiliser la métadonnée `com.google.android.geo.API_KEY` dans le manifeste ;
- conserver `local.properties` dans `.gitignore` ;
- considérer que la clé Cloud est limitée au package `com.francotte.gasoilaroundme`, à l’empreinte SHA-1 appropriée et à Maps SDK for Android uniquement.

Lors de la publication, utiliser des identifiants séparés et correctement restreints pour la signature de production. Ne pas confondre l’empreinte SHA-1 de développement avec le certificat de production Google Play.

## Localisation et confidentialité

- Demander l’autorisation de localisation dans son contexte d’utilisation, et non immédiatement sans explication.
- Prendre en charge la localisation approximative comme la localisation précise au premier plan.
- L’application doit rester utilisable après un refus grâce à la recherche manuelle.
- Ne pas redemander continuellement l’autorisation après un refus. En cas de refus définitif, proposer l’ouverture des réglages seulement lorsque cela est utile.
- Ne transmettre la position que sous forme de coordonnées nécessaires à la requête directe des stations ; ne pas conserver ni profiler un historique des positions.
- Ne pas demander l’accès aux contacts, à l’état du téléphone, à l’identifiant publicitaire, à la localisation en arrière-plan ou à toute autorisation sans rapport avec le besoin.
- Prévoir des états explicites pour le chargement, l’absence de résultat, le mode hors connexion, le refus de permission et les erreurs du service distant.

## Principes d’architecture

Garder le prototype simple, tout en définissant des frontières claires afin qu’un backend puisse remplacer l’accès direct à l’API ultérieurement :

- `data` : DTO de l’API, sources distantes, analyse des réponses et implémentations des repositories ;
- `domain` : modèles de station et de carburant, contrats des repositories, calcul de Haversine, filtrage et tri ;
- `ui` : écrans et composants, état d’interface immuable et ViewModels ;
- `location` : petite abstraction de la localisation Android ;
- `navigation` : utilitaire d’ouverture d’une application de navigation externe.

Privilégier un flux de données unidirectionnel. Les composables affichent un état et émettent des événements ; le réseau, l’analyse des réponses, le filtrage, la localisation et le tri ne doivent pas être effectués directement dans les composables.

Utiliser les coroutines avec concurrence structurée. Annuler les anciennes recherches d’adresses et les anciens chargements de stations devenus inutiles. Éviter les abstractions prématurées, les frameworks d’injection de dépendances, les bases de données et les modules supplémentaires tant que le prototype n’en a pas besoin. Centraliser les dépendances dans `gradle/libs.versions.toml`.

La frontière du repository doit exposer un modèle applicatif stable et ne pas laisser les DTO Opendatasoft remonter jusqu’à l’interface. Cette frontière permettra une future migration vers un backend.

## Exigences d’interface et d’interaction

- Concevoir l’interface bord à bord en respectant les marges système.
- Ne pas masquer l’attribution cartographique ni la marque Google.
- Le bottom sheet ne doit pas recouvrir définitivement les commandes essentielles de la carte.
- Les sélections de carburant et de tri doivent être évidentes et accessibles.
- Tri par défaut : prix valide le plus bas pour le carburant sélectionné.
- Autre tri : station la plus proche selon la distance à vol d’oiseau.
- Distinguer clairement un prix absent, une rupture déclarée, un prix ancien et une erreur réseau.
- La sélection d’un marqueur doit révéler ou sélectionner la ligne correspondante ; la sélection d’une ligne doit mettre en évidence son marqueur.
- Afficher le logo de l’enseigne sur la carte, dans la liste et dans la fiche lorsqu’elle est identifiée de façon suffisamment fiable ; utiliser un pictogramme de pompe générique dans les autres cas.
- Utiliser des textes français et le formatage français des nombres, par exemple `1,789 €/L` et `3,2 km`.
- Fournir des descriptions de contenu et des zones tactiles adaptées à l’accessibilité.

### Marqueurs de prix sur la carte

S’inspirer particulièrement de `1788170251615.jpg` pour les éléments affichés au-dessus de la carte. Chaque station possédant un prix valide pour le carburant sélectionné doit disposer d’un marqueur compact composé :

- d’un encart coloré à coins arrondis contenant le prix en grand, par exemple `1,990 €` ;
- d’une petite pointe ancrée précisément sur les coordonnées de la station ;
- du logo de l’enseigne au-dessus ou à côté du prix lorsqu’il est disponible, sinon d’un pictogramme générique de pompe ;
- d’un état sélectionné visuellement distinct, sans dépendre uniquement de la couleur.

Prévoir un catalogue interne d’enseignes permettant d’associer un identifiant stable à un nom normalisé, à un logo local et à un libellé accessible. Ce catalogue peut notamment couvrir les principales enseignes françaises telles que TotalEnergies, Esso, BP, Avia et les stations de la grande distribution.

N’utiliser que des logos provenant d’une source officielle ou d’un kit média dont l’usage dans l’application a été vérifié. Conserver la provenance et les éventuelles conditions d’utilisation de chaque ressource. Ne pas extraire les logos des captures d’écran et ne pas télécharger arbitrairement des images issues d’un moteur de recherche.

L’enseigne ne doit jamais être déduite du prix, des couleurs du marqueur ou d’une ressemblance approximative. La résolution doit suivre une règle explicite et testable à partir des données disponibles ou d’une table de correspondance contrôlée. En cas d’absence ou d’ambiguïté, afficher le pictogramme générique et le meilleur libellé officiel disponible plutôt qu’un logo potentiellement erroné.

La couleur traduit la position relative du prix parmi les stations actuellement comparables pour le carburant sélectionné :

- vert : prix parmi les moins chers ;
- orange : prix intermédiaire ;
- rouge : prix parmi les plus élevés ;
- gris : prix manquant, indisponible, ancien au-delà du seuil retenu ou carburant en rupture, si ce marqueur doit malgré tout rester visible.

Le calcul des catégories doit être déterministe, documenté dans le code et robuste avec peu de résultats ou des prix identiques. Utiliser de préférence des quantiles sur les prix valides actuellement affichés, avec un comportement explicite pour moins de trois prix. Recalculer les catégories lorsque le carburant, la zone ou les résultats changent.

La couleur est une aide visuelle, jamais l’unique information : le prix reste écrit dans l’encart et les états indisponibles disposent d’un libellé ou pictogramme compréhensible. Choisir des couleurs suffisamment contrastées, testables en thème clair et sombre, et compatibles autant que possible avec les déficiences de perception des couleurs.

Éviter que les marqueurs rendent la carte illisible :

- à faible niveau de zoom, regrouper les stations proches ou réduire la densité affichée ;
- à un niveau de zoom utile, afficher les encarts de prix complets ;
- maintenir la station sélectionnée au premier plan ;
- ne pas masquer la position de l’utilisateur, les commandes de recentrage ni l’attribution Google ;
- assurer la cohérence entre la couleur du marqueur et les informations de la ligne correspondante dans le bottom sheet.

Les marqueurs doivent être générés à partir de composants ou ressources internes et mis en cache par combinaison utile de prix, couleur et état. Ne pas recréer inutilement tous les bitmaps à chaque recomposition Compose ou mouvement de caméra.

## Réseau et résilience

- Ne déclarer que la permission Internet habituelle et les permissions de localisation au premier plan nécessaires.
- Utiliser exclusivement HTTPS.
- Appliquer des délais de connexion et de lecture raisonnables.
- Limiter les champs et le nombre d’enregistrements demandés lorsque l’API officielle le permet.
- Ne pas charger toute la France à chaque déplacement de la caméra.
- Temporiser les actualisations liées à la carte et éviter les requêtes concurrentes en double.
- Conserver en mémoire le dernier résultat valide lors d’un échec temporaire d’actualisation lorsque cela est raisonnable.
- Toute erreur d’API doit produire un état récupérable avec une action de nouvelle tentative, et non un plantage.

## Exigences de test

Ajouter au minimum des tests unitaires déterministes pour :

- l’association entre les carburants et les champs officiels ;
- l’analyse des données absentes, nulles ou malformées ;
- le calcul de la distance de Haversine ;
- le filtrage par rayon ;
- le tri par prix et par distance ;
- la gestion des ruptures et des prix absents ;
- la normalisation des enseignes, l’association des logos et le fallback générique ;
- l’annulation ou le remplacement des anciennes requêtes lorsqu’ils sont pertinents.

Ajouter des tests Compose ou d’interface ciblés pour les transitions d’état importantes lorsque cela est raisonnable. Les tests unitaires ne doivent pas dépendre des services gouvernementaux, IGN ou Google réels. Garder les données de test petites et représentatives.

Avant de livrer des changements, exécuter les vérifications les plus ciblées puis, lorsque cela est possible sous Windows :

```powershell
.\gradlew testDebugUnitTest
.\gradlew assembleDebug
```

Ne jamais afficher `local.properties` dans les diagnostics ou les sorties de test.

## Définition d’un premier prototype utilisable

Le prototype est utilisable lorsqu’une nouvelle installation peut :

1. expliquer puis demander la localisation au premier plan ;
2. centrer la carte à partir d’une position approximative ou précise acceptée ;
3. proposer une recherche française par ville ou adresse après un refus ou un échec de localisation ;
4. récupérer les stations officielles proches du point choisi ;
5. basculer entre les six carburants ;
6. afficher des marqueurs et une liste synchronisée dans le bottom sheet ;
7. trier les résultats valides par prix ou par distance à vol d’oiseau ;
8. signaler honnêtement les prix absents, anciens ou indisponibles ;
9. ouvrir une application de navigation externe pour une station ;
10. gérer correctement les erreurs réseau ou d’API sans exposer de secret.

## Hors périmètre sauf demande explicite

- backend, import planifié, base de données ou agrégation de données de stations ;
- authentification et comptes utilisateur ;
- favoris synchronisés entre appareils ;
- alertes de prix ou notifications ;
- historique et graphiques de prix ;
- vérification communautaire des prix ;
- publicités, abonnements, achats ou suivi analytique ;
- localisation en arrière-plan ou suivi d’un trajet ;
- distance routière et durée de trajet pour chaque station ;
- versions iOS ou web et marchés hors de France.

Si une future demande entre en conflit avec ces contraintes, expliquer ses conséquences produit ou financières avant de modifier l’architecture.
