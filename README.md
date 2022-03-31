# Valeo

CHALLENGE : AI POWERED VISUAL INSPECTION FOR MANUFACTURING by VALEO

1)	Introduction : contexte et objectifs

Ce projet a été mis en place pour aider l’entreprise Valeo, un fournisseur en automobile.Cette entreprise a des clients exigeants en termes de qualité, il faut donc fournir des produits sans défauts. De plus, il s’agit également de ne pas jeter des pièces non défectueuses pour éviter un gâchis de ressource. Il est donc primordiale de détecter les défauts avec une précision maximale.

Pour ce faire l’entreprise a mis en place une étape de vérification par l’imagerie. Notre objectif est de mettre en place une méthode d’apprentissage et de reconnaissance de défauts à partir des images fournies par Valeo.

2)	Echantillonnage

On choisit pour des raisons de faisabilité terrain et pour que ce soit applicable à des cas réels chez Valéo, de ne pas rééquilibrer les classes. On garde donc le même taux de produits avec défaut que celui fourni par l’entreprise. Environ 38% des images correspondent à un défaut (62% sans défaut), donc le jeu de données est naturellement relativement équilibré.
Pour des raisons de puissance de machines nous choisissons de construire le modèle de détection sur 20% images fournies. Ceci correspond à 2000 images pour l’échantillon d’apprentissage/validation et 400 images pour l’échantillon test.

L’échantillon d’apprentissage/validation (2000 images) a été divisé en 2. L’échantillon d’apprentissage correspond à 75% de l’échantillon soit 1500 images et l’échantillon de validation à 25% de l’échantillon soient 500 images. 

On construira alors différents modèles de classification entre image avec ou sans défaut qu’on ajustera sur l’échantillon d'apprentissage (1500 images) puis on mesurera leurs performances sur l’échantillon de validation (500 images).
Enfin, le modèle retenu sera évalué sur l’échantillon de test (400 images). 

3)	Pré-traitement des images

On choisit de réduire la taille des images de 2415x2070 pixels à 200x200 pixels afin de limiter les temps de calcul.

Les images sont disponibles en noir et blanc. Nous pourrions les coloriser et obtenir ainsi 3 matrices 200x200 par image. Les algorithmes pré-entraînés (Resnet, Efficient Net) utilisent traditionnellement des images en couleurs comme input. 

Nous avons choisi de ne pas le faire car cela augmenterait les temps de calcul. De plus, nous allons développer notre propre réseau CNN donc cela ne paraît pas nécessaire. Les images sont donc importées en nuances de gris. 

4)	Modèles de référence (Baseline Models)

Baseline model 1 : réseau sans couche cachée

Le modèle initial ne contient que les données d’entrée et la couche de sortie il est constitué d’une étape intermédiaire d'aplatissement de l’image, ceci permet d’utiliser chaque pixel de l’image (200 x 200 = 40 000 pixels) comme une donnée supplémentaire, cela transforme donc une information à deux dimensions (axe horizontal et axe vertical) en une unique dimension. 

La fonction d’activation utilisée est la fonction sigmoïde. Cette dernière retourne des valeurs comprises entre 0 et 1. Cette fonction est adaptée au cas de classification binaire. Lorsque la valeur tend vers 0 alors elle est classée en 0 et vice-versa.

Ce modèle sans couche cachée correspond donc en réalité à une régression logistique éffectuée sur chaque pixel de l’image qui deviennent donc les variables d’entrée. 

Paramètre commun à tous les model (baseline et les suivant) :
●	Optimizer : Adam. Nous utilisons une optimisation par l'algorithme Adam qui est une méthode de gradient descendant basé sur une estimation adaptée de l’ordre des moments. 
●	loss : Binary Cross Entropy (adapté dans les cas de classification binaire)
●	learning rate : 0.0002 (baseline models) et 0.001 (CNN)
●	Le nombre d’epochs est le nombre de fois où le modèle parcourt l’ensemble de la base de données. Nous avons fixé le nombre d’epochs à 40 pour les baselines models (rapides à ajuster) et 200 par la suite.
 
Ce premier modèle a une accuracy de 0.95 (sur l’échantillon de validation).

Baseline model 2 : réseau de neurones avec 2 couches cachées (hidden layers), sans couches de convolution 

Le second modèle de référence est un réseau de neurones sans couches de convolution.

On a tout d'abord réduit la dimension des images. L’opération de Maxpooling permet de réduire la taille des images tout en préservant les caractéristiques importantes. L’image est donc découpée en petites parties, et dans chaque partie on garde la valeur la plus importante. On a fixé pool_size = 8, ce qui permet d’obtenir des images de taille 25x25 contre 200x200 avant cette étape.

Ces nouvelles informations sont à la suite aplaties, on a donc une couche plate de taille 1 x 625 (25*25) . 

On introduit ensuite 2 couches denses comptant 256 et 128 neurones. La fonction d’activation choisie est ‘ReLU’ (Rectified Linear Unit) qui remplace les valeurs négatives d’entrées par des zéros. Nous utiliserons cette fonction pour chaque couche cachée. 

Nous utilisons également une méthode de dropout (rate = 0.1) qui désactive les sorties de neurones aléatoirement à hauteur de 10% (rate = 0.1). Ceci permet d’éviter l’overfitting.

Ce second modèle a également une accuracy de 0.95 (sur l’échantillon de validation).
 


5)	Réseaux de neurones convolutifs (CNN : Convolution Neural Network)

La couche de convolution : 

L’opération de convolution peut-être perçue comme une détection de la présence d’une sous-partie similaire au filtre de convolution définie au signal d’entrée. C’est donc une sorte de détecteur de formes. Ainsi les premières couches convolutionnelles de départ apprennent les formes les plus simples. Plus les couches sont profondes, plus les formes apprises sont complexes. 

Choix du nombre de couches de convolution : 

En pratique, il est difficile de définir le nombre de couches de convolution optimal, mais à partir de la visualisation des images on peut supposer que 2 ou 3 couches ne vont pas suffire car les formes ne sont pas simple (une forme simple peut faire penser aux images de chiffres).

Ainsi nous essayons plusieur valeurs, avec 1 à 4 couches nous obtenons une accuracy de 96%. Avec 5 couches nous obtenons le modèle optimal avec la meilleure accuracy d’une valeur de 0,97 (sur l’échantillon de validation). Les modèles avec 6 couches ont des accuracy moins importantes que celle du modèle à 5 couches.

Autres paramètres du modèle à convolution : 

●	La couche de convolution est définie par différents paramètres. L’un deux est la taille du noyau (kernel). La taille du noyau correspond à la taille de la zone que le filtre de convolution analyse ou ‘scan’ pour apprendre des formes  (cf schéma ci-dessous).  Cette taille est variable. Lorsque la taille de kernel est petite, il est plus facile de détecter des petites formes correspondant à quelques pixels. Nous avons eu de meilleurs résultats pour un kernel size de 2 que de 3. 

●	De façon à limiter le volume d’information et limiter la durée d’apprentissage, il est utile de réduire les dimensions des matrices issues de l’image. Nous avons également comparé deux méthodes de réduction de dimension :
○	La première implique toujours les couches de convolutions Conv2D et le ‘stride’ (ou pas) associé. Le stride modifie le pas de déplacement du kernel sur l’image. Un stride de (1,1) conservera la taille de l’image (à l’exception des bords selon le padding choisi, cf schéma au-dessus). Un stride de (2,2) divisera les dimensions de l’image par 2.
○	La seconde implique d’autres couches appelées ‘Maxpooling2D’. Le Maxpooling consiste à balaier l’image par zone une zone et ne garder que la valeur maximum de chaque zone. Avec pool_size = 2, on réduit les dimensions de l’image par 2, en gardant que la valeur maximale dans chaque carré de taille 2x2.
Nous avons obtenu de meilleurs résultats avec la seconde méthode, en utilisant des couches de MaxPooling pour réduire la dimension (pool_size = 2), plutôt qu’en utilisant un stride de (2,2) dans les couche de convolution Conv2D.

→ Nous avons donc choisi pour les couches Conv2D : kernel_size = 2, stride = (1,1) et padding = ‘same’ (qui permet de ne pas réduire la dimension de l’image en ajoutant des pixels sur les bords). Nous avons aussi choisi des couches de MaxPooling entre chaque couche Conv2D (pool_size = 2) afin de réduire la dimension.


●	Le nombre de filtres (filters) est le dernier paramètre important. Chaque filtre applique une convolution sur les données d’entrée. Le filtre balaye l’image et applique plusieurs fonctions d’activation pour chaque position. Le nombre de filtres dépend de la complexité de l’image, plus l’information à capturer est détaillée plus le nombre de filtres doit être élevé. Au fur et à mesure que les premières couches de convolutions détectent des formes/motifs, les suivantes combinent ces formes de façon à obtenir de nouvelle formes plus complexes et donc plus diverses/nombreuses. Ainsi le nombre de filtres doit-il augmenter lorsque l’on passe des premières couches aux couches plus profondes. Dans notre cas, de façon à limiter le volume d’information, nous nous somme contenter de 4, 8, 12, 16 et 20 filtres respectivement pour les 5 couches.

Le modèle final retenu :

Le modèle sélectionné est donc construit avec 5 couches de convolution. On obtient donc une accuracy de 97% sur l'échantillon de validation. L’ajout de couches de convolution au modèle initial a donc bien permis d’améliorer la performance du modèle. On a désormais 3% d’erreur de classification, contre 5 à 6% pour le réseau sans convolution.

6) Performance du modèle retenu sur l’échantillon test

Pour vérifier que notre modèle est bon, nous avons ajusté le modèle retenu 3 fois séparément (il y a une part de hasard dans l’apprentissage par réseau de neurones, notamment lors de la formation des batchs).

On obtient pour l’échantillon test une accuracy de 97,5% , 97,75% et même 99,75% !

Matrice de confusion pour le cas où accuracy = 97,75% : 

Prédit négatif (0)	Prédit positif(1)
Négatif (0) = pièce défectueuse	VN = 154	FP = 9
Positif(1) = pièce fonctionelle	FN = 0	VP = 237

Le sujet précise que les faux positifs résultent dans des problèmes de qualités (pour le client) plus graves que les faux négatifs (perte d’argent car on jette de bonnes pièces). 

La métrique d’évaluation du modèle à minimiser est donc la suivante : 

C = (FN + lambda * FP)/N avec lambda = 100 donc
C = (FN + 100 * FP)/N

Les faux positifs sont donc 100 fois plus graves que les faux négatifs.
‘Pas de chance’, nos 9 erreurs de classification sont des faux positifs.
On obtient un score C= 100 * 9 / 400 soit C=2,25.

Pour améliorer ce score, on augmente le ‘seuil de reconnaissance des positifs’. Celui-ci est fixé à 0.5 par défaut. En augmentant ce seuil, on va prédire moins de positifs et donc diminuer le nombre de faux positifs. Avec un seuil de 0.9, on obtient la matrice suivante :

Prédit négatif (0)	Prédit positif(1)
Négatif (0) = pièce défectueuse	VN = 154	FP = 2
Positif(1) = pièce fonctionnelle	FN = 6	VP = 237

On a alors C = (6 + 2* 100) / 400 soit C = 0.53

En augmentant le seuil à 0.95, on obtient la matrice suivante :


Prédit négatif (0)	Prédit positif(1)
Négatif (0) = pièce défectueuse	VN = 154	FP = 0
Positif(1) = pièce fonctionnelle	FN = 19	VP = 237

On a alors C = (19 + 0* 100) / 400 soit C = 0.0475.

La valeur de la métrique des ‘premier modèles testés’ dont il est fait allusion dans le sujet est C=0.503, on est donc bien meilleur ! On a alors une accuracy de plus de 95% (381/400) sans avoir aucun faux positif !

7) Conclusion

Le modèle développé répond bien à la demande de la firme. En effet, toutes les pièces avec des défauts sont détectées par imagerie. Il n’y a donc aucun risque qu’un client de Valeo reçoive une pièce défectueuse. Ils gagnent donc la satisfaction client. 

Cependant il est possible que le modèle prédise un défaut qui n’existe pas forcément (5% des cas). Mais cette situation est rare, la précision est bonne (95%). Cette précision va permettre à la firme de réaliser des économies en jetant moins de pièces fonctionnelles qui avaient été détectées défectueuses par la machine. 
















