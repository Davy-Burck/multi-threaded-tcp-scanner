# Rapport d'Audit Réseau & Bonnes Pratiques de Scan TCP

## 1. Objet du Projet & Contexte Technique
* **Outil développé :** Scanner de ports TCP multi-threadé en Python (`src/network_scanner.py`) utilisant le module `socket`, `ThreadPoolExecutor` et `argparse`, avec capture de bannières (*banner grabbing*) et export structuré au format JSON (`data/report.json`).
* **Objectif :** Identifier les ports TCP ouverts sur une cible donnée, détecter les services potentiellement actifs et produire un compte rendu exploitable.

## 2. Pré-Engagement, Périmètre & Normes de Référence
Avant d'exécuter un scan réseau — même avec un outil développé soi-même — le respect d'un cadre strict est indispensable (inspiré des phases de **pré-engagement** et de **collecte d'informations / analyse de vulnérabilité** de la méthodologie **PTES**) :
* **Autorisation & périmètre (Scope) :** Le scan ne doit être exécuté que sur des adresses IP, plages ou noms de domaine explicitement autorisés (laboratoire local, environnement de test, périmètre validé par écrit). Tout scan hors périmètre expose à des risques juridiques et opérationnels.
* **Règles d'engagement & impact réseau :**
  * **Débit / parallélisme (`-w`) :** Un nombre élevé de *threads* peut saturer un service ou un lien réseau peu dimensionné (risque de déni de service involontaire).
  * **Fenêtre de tir et discrétion :** Un scan TCP complet ou une sollicitation de bannière génère du trafic observable par les pare-feu, IDS/IPS et journaux système.
  * **Traçabilité :** Chaque exécution doit être datée, justifiée et rattachée à un périmètre précis.

## 3. Méthodologie d'Exécution & Limites Techniques
* **Paramétrage contrôlé :** Ciblage précis via `-t <IP_CIBLE>`, `-p <PORTS>`, `-w <THREADS>` et `-o <FICHIER_EXPORT>`, avec gestion des *timeouts* et des ports filtrés ou fermés.
* **Fermeture propre des ressources :** Utilisation de gestionnaires de contexte (`with`) sur les sockets TCP afin d'éviter l'épuisement des descripteurs de fichiers lors de scans répétés.
* **Limites d'interprétation :**
  * Un port ouvert indique qu'un service écoute sur ce port TCP, pas nécessairement qu'il est vulnérable.
  * Le *banner grabbing* reste indicatif : une bannière peut être absente, incomplète ou volontairement modifiée/masquée, et ne remplace pas une vérification approfondie.
* **Confidentialité des sorties (`data/` & `.gitignore`) :** Les rapports générés (`data/report.json`) peuvent contenir des informations sensibles sur l'infrastructure scannée. Leur exclusion du dépôt Git via `.gitignore` protège ces données.

## 4. Structure du Rapport d'Audit & Modèle de Synthèse
Un rapport de scan professionnel ne se limite pas à une liste de ports ouverts : il relie **le constat technique**, **le risque potentiel** et **la mesure corrective** :

| Priorité | Constat technique (ex. issu de `data/report.json`) | Risque / Impact potentiel | Recommandation / Remédiation | Statut |
| :--- | :--- | :--- | :--- | :--- |
| **Haute** | Port d'administration ou service sensible exposé (ex. : SSH, base de données, service non chiffré) | Accès non autorisé, exposition de surface d'attaque ou interception | Restreindre l'accès (pare-feu, VPN, liste blanche IP) et durcir l'authentification | À adapter |
| **Moyenne** | Bannière de service exposant une version logicielle datée ou verbeuse | Facilitation du ciblage d'une vulnérabilité connue ou divulgation d'informations | Mettre à jour le service et limiter la divulgation inutile de versions | À adapter |
| **Faible** | Port ouvert légitime ou comportement ambigu (port filtré / *timeout*) | Risque faible ou besoin de confirmation complémentaire | Vérifier la nécessité du service et ajuster les paramètres de sonde (*timeout* / plage) | Validé |
