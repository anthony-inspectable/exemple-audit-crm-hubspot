# Exemple d'audit CRM Inspectable

Ce dépôt contient **un rapport d'audit CRM complet**, tel qu'il est livré, avec les fichiers qui
l'accompagnent. Rien n'est simulé ni mis en scène : ce sont les livrables réels de l'outil,
produits par une exécution ordinaire.

Il existe pour une raison simple : un audit se juge sur son rendu, pas sur sa description.

---

## Le rapport

[![Première page du rapport d'audit](images/page-01.png)](exemple/rapport-audit-exemple-fr.pdf)

**17 pages, dix sections.** La couverture porte le diagnostic entier, parce qu'un rapport dont il
faut lire douze pages avant de comprendre le problème ne sera pas lu.

Sur cet exemple, l'outil analyse 853 contacts et conclut :

| | |
|---|---|
| Score global | **72,4 / 100**, à surveiller |
| Base réellement prospectable | **73 contacts sur 853** (8,6 %), dont 71 survivent au nettoyage |
| Juridiquement inexploitables en l'état | **441** sur 853 (51,7 %) |
| Leads sans propriétaire | **234** (27,4 %) |
| Leads inactifs à arbitrer | **303** (35,5 %) |

La phrase qui résume l'affaire est en haut de la page : *vous payez pour 853 contacts, 73 sont
exploitables*.

### Le modèle commercial est déclaré, jamais deviné

Cet exemple est audité en **Sales-Led**, c'est-à-dire en prospection sortante active. Ce n'est
pas un détail de paramétrage : c'est ce qui décide de ce qui compte comme un défaut.

> **237 contacts critiques et 0 importants AU SENS DE CE MODÈLE.** Un autre modèle donnerait un
> décompte différent sur le même fichier.

Un contact sans téléphone bloque une équipe qui appelle ; il ne gêne pas un produit en libre-service.
L'outil ne devine donc jamais ce modèle, il le demande, et le rapport dit à voix haute que ses
priorités en découlent. Sans modèle déclaré, cette page invite à le renseigner plutôt que d'inventer
un classement qui aurait l'air savant.

---

## Ce que contient `exemple/`

| Fichier | Ce que c'est |
|---|---|
| `rapport-audit-exemple-fr.pdf` | Le rapport, 17 pages. **Français** |
| `rapport-audit-exemple-en.pdf` | Le même en **anglais**. L'outil est bilingue de bout en bout |
| `classeur-complet-fr.xlsx` | 26 onglets : base préparée, doublons par groupe, décisions à valider, journal des modifications, ce qui reste à corriger à la main |
| `import-mise-a-jour.csv` | Le fichier que le client réimporte dans HubSpot pour appliquer le nettoyage |
| `import-creation.csv` | Sa variante pour alimenter un **autre** portail (migration) |
| `import-associations.csv` | Les liens contact → entreprise, restitués depuis les identifiants HubSpot. Livrable **séparé et facultatif** |
| `mode-emploi-import-fr.txt` | La procédure d'import, pas à pas |

Le dossier `images/` contient les 17 pages en PNG, pour lire le rapport sans le télécharger.

### Deux fichiers d'import, jamais les deux à la fois

C'est le détail qui sépare un livrable utilisable d'un livrable dangereux. Le fichier de **mise à
jour** porte le `Record ID` : HubSpot rattache alors chaque ligne à la fiche existante. Celui de
**création** ne le porte pas, délibérément : sinon HubSpot mettrait à jour au lieu de créer.
Importer le mauvais des deux fabrique un doublon de chaque fiche de la base.

### Les associations : trois niveaux de preuve, un seul écrit

Quand un contact est retiré puis réimporté, son rattachement à l'entreprise se perd, et les
colonnes d'association ne sont pas éditables comme des propriétés. Sur un export réel de 960
contacts, cela représente 955 liens à refaire à la main.

L'export porte pourtant les identifiants d'entreprise que HubSpot a lui-même écrits. Les
réimporter n'est pas créer une association : c'est **rendre** celle qui existait. Sur cet exemple,
**457 liens sont restitués**. Mais tous les cas ne se valent pas, et le fichier le dit :

| Niveau | Situation | Écrit ? |
|---|---|---|
| **A, restitution** | Un seul identifiant d'entreprise. C'est la donnée de HubSpot qu'on lui rend. | **Oui** |
| **B, désambiguïsation** | Plusieurs entreprises liées, et HubSpot déclare la principale. On la retient, et le motif le dit. | **Oui**, tracé comme un choix |
| **B, non tranché** | Plusieurs entreprises, aucune principale déclarée. | **Non**. Choisir au hasard fabriquerait de la donnée fausse |
| **C, inférence** | Un nom d'entreprise, aucun identifiant. | **Jamais** |

Le niveau C est la frontière. Deviner qu'un contact appartient à une société parce que le nom
ressemble, c'est enrichir depuis une source externe, ce qu'un audit s'interdit. Ces cas sont
exposés dans le classeur, pour que le client tranche lui-même, et rien de plus.

---

## Les données de cet exemple sont synthétiques

**Aucune donnée personnelle réelle n'apparaît ici, et ce n'est pas une promesse : c'est vérifiable.**

Le jeu de 853 contacts est produit par un générateur déterministe : même graine, même fichier,
sur n'importe quelle machine. Les anomalies y sont **injectées volontairement** et comptées à l'avance :
3 doublons exacts, 2 doublons approchants, 3 e-mails invalides, 4 désinscriptions, 5 contacts à
purger, 2 téléphones cassés, et ainsi de suite.

Les adresses utilisent le domaine **`.example`**, réservé par la
[RFC 2606](https://www.rfc-editor.org/rfc/rfc2606) : il ne sera jamais délégué, aucun serveur de
messagerie ne peut l'héberger. Aucune des 459 adresses de ces fichiers ne peut donc atteindre qui
que ce soit, ni coïncider avec l'adresse d'une personne réelle. Les seules exceptions sont les
deux fautes de frappe volontairement injectées. L'outil doit démontrer qu'il les repère et les
signale sans jamais les corriger d'office.

Cela rend l'exemple honnête sur deux plans à la fois. Il ne peut divulguer la base de personne. Et
comme la vérité est connue AVANT l'audit, on peut vérifier que l'outil trouve ce qu'il doit trouver,
au lieu de le croire sur parole.

Un exemple bâti sur un vrai export ferait l'inverse : plus flatteur, invérifiable, et il faudrait
vous demander de faire confiance.

---

## Le principe qui gouverne l'outil

**L'audit se fait hors ligne.** Le fichier n'est connecté à aucune API, ne transite par aucun
service tiers, ne part sur aucun serveur. Il est lu sur un poste, traité, et les livrables sont
rendus. Aucune connexion à votre portail HubSpot n'est demandée, donc aucun accès à révoquer.

Trois conséquences pratiques :

- **L'outil propose, vous décidez.** Rien n'est appliqué à votre CRM. Les fichiers d'import restent
  des fichiers tant que vous ne les importez pas.
- **Ce qui n'a pas pu être mesuré est écrit comme tel.** Un rapport qui tait ses angles morts laisse
  croire qu'il n'y en a pas.
- **Rien n'est deviné.** Une association d'entreprise est restituée depuis l'identifiant que HubSpot
  a lui-même exporté, jamais déduite d'un nom qui se ressemble.

---

## Ce que ce dépôt ne contient pas

Le code de l'outil.

Ses **réglages**, en revanche, y sont : l'onglet `Parametres` du classeur publie les seuils, les
pondérations du score et les règles appliquées. C'est volontaire. Un score dont on ne peut pas
vérifier la composition ne vaut pas grand-chose, et ces réglages sont faits pour être discutés.
Trois ans de conservation, par exemple, c'est la recommandation de la CNIL, pas une préférence.

---

## Pour aller plus loin

Le détail des prestations, la méthode et les tarifs : **[inspectable.fr](https://inspectable.fr)**

Anthony Abreu, Inspectable
