# CARNET-DE-CREDI-EN-LIGNE-
# 📒 Carnet de Crédit

Application web de gestion de comptes clients (carnet de crédit) pour petits commerces. Suivi des soldes, historique des transactions, export PDF/Excel et synchronisation en temps réel entre plusieurs postes.

Développée à l'origine à partir d'un fichier Excel (`DEVIS.xlsx`), l'application a évolué vers un tableau de bord web complet, synchronisé via Google Sheets.

---

## ✨ Fonctionnalités

- **Gestion des clients** : ajout, modification (nom / téléphone), recherche, tri par dette ou par ordre alphabétique
- **Suivi des opérations** : achat, montant payé, description libre, mode de paiement (Espèce / Chèque / Virement) avec détails du chèque (porteur, banque, numéro)
- **Historique fusionné** : les anciens mouvements importés de l'Excel d'origine et les nouvelles opérations sont réunis dans un seul relevé chronologique à l'export
- **Export PDF** par client, avec mise en page soignée (jsPDF + AutoTable)
- **Export Excel** par client et export global de tous les clients (ExcelJS / SheetJS)
- **Synchronisation multi-postes en quasi temps réel** via Google Sheets (délai ~6 secondes), avec verrouillage anti-écrasement pendant la saisie
- **Tableau de bord** : KPI (nombre de clients, total des dettes, etc.), badges colorés, sauvegarde/import JSON local

---

## 🗂 Structure du projet

```
├── index.html        # Application complète (HTML + CSS + JS en un seul fichier)
└── Code.gs            # Script backend Google Apps Script (API de synchronisation)
```

L'application est un fichier HTML unique — aucune installation, aucune dépendance côté serveur autre que le script Google Apps Script.

---

## 🚀 Déploiement

### 1. Backend — Google Apps Script

1. Créez une feuille Google Sheets (ou utilisez celle du carnet existant).
2. Menu **Extensions → Apps Script**.
3. Copiez le contenu de [`Code.gs`](./Code.gs) dans l'éditeur.
4. **Déployer → Nouveau déploiement → Application Web** :
   - Exécuter en tant que : **Moi**
   - Qui a accès : **Tout le monde**
5. Copiez l'URL se terminant par `/exec`.

> ⚠️ Après toute modification du script, il faut redéployer une **nouvelle version** (Déployer → Gérer les déploiements → Modifier → Nouvelle version), sinon les changements ne sont pas pris en compte.

### 2. Frontend — Hébergement statique

Le fichier `index.html` peut être hébergé sur n'importe quel service d'hébergement statique :

- **Netlify** (recommandé, gratuit) : glissez-déposez `index.html` sur [app.netlify.com/drop](https://app.netlify.com/drop), ou connectez ce dépôt GitHub pour un déploiement automatique.
- **GitHub Pages** : activez Pages dans les paramètres du dépôt, en pointant vers la racine ou vers `/docs`.
- **Vercel**, ou tout autre hébergeur de fichiers statiques.

### 3. Configuration de la synchronisation

Une fois l'application ouverte (en local ou hébergée) :

1. Cliquez sur **Synchronisation**.
2. Collez l'URL `/exec` obtenue à l'étape 1.
3. **Enregistrer**.

Répétez cette étape sur chaque appareil (ordinateur, téléphone) devant accéder aux mêmes données. Tous les postes utilisant la **même URL** partagent le même carnet de clients.

---

## 🖥️ Utilisation

- **Nouveau client** : bouton en haut de la barre latérale — nom (obligatoire) + téléphone (optionnel).
- **Modifier un client** : icône crayon à côté du numéro de téléphone dans la fiche client (nom et téléphone).
- **Ajouter une opération** : sélectionnez un client, renseignez date, montant acheté, description (optionnelle), montant payé et mode de paiement.
- **Export** : boutons « Télécharger en PDF » / « Télécharger en Excel » dans la fiche client ; export Excel global depuis la barre d'outils.
- **Forcer l'envoi** : en cas de désynchronisation, envoie la copie locale de cet appareil vers la feuille Google Sheets (écrase la feuille — à utiliser avec précaution).

---

## 🔧 Stack technique

| Composant       | Technologie                                  |
|-----------------|-----------------------------------------------|
| Frontend        | HTML / CSS / JavaScript vanilla (fichier unique) |
| Export PDF      | [jsPDF](https://github.com/parallax/jsPDF) + AutoTable |
| Export Excel    | [ExcelJS](https://github.com/exceljs/exceljs) / [SheetJS (xlsx)](https://sheetjs.com/) |
| Backend / API   | Google Apps Script (`doGet` / `doPost`)      |
| Stockage        | Google Sheets (onglet `DATA`)                |
| Sync temps réel | Sondage (`polling`) toutes les 6 secondes, avec anti-cache |

---

## ⚙️ Détails techniques importants

- **Limite de cellule Google Sheets (50 000 caractères)** : les données étant volumineuses, le script `Code.gs` découpe automatiquement le JSON en plusieurs lignes de 40 000 caractères pour rester sous la limite.
- **Anti-cache** : chaque lecture (`GET`) ajoute un paramètre `_ts` et utilise `cache: 'no-store'` pour garantir que chaque poste lit toujours la dernière version, et non une réponse mise en cache par le navigateur.
- **Verrouillage d'écriture** : un indicateur `writeInFlight` empêche le rafraîchissement automatique d'écraser une saisie en cours ou un envoi en attente.
- **Pause pendant la saisie** : le rafraîchissement automatique est suspendu tant que l'utilisateur interagit avec un champ (et pendant 4 secondes après), pour ne jamais effacer une saisie en cours.
- **Rendu conditionnel** : l'écran n'est reconstruit que si les données reçues diffèrent réellement de l'état affiché, ce qui préserve l'état des tableaux dépliés (ex. anciens mouvements) entre deux cycles de synchronisation.
- **Encodage PDF** : la police standard de jsPDF (Helvetica) ne supporte que le Latin-1. Une fonction `pdfTxt()` normalise les espaces insécables et tirets cadratins tout en préservant les caractères accentués français.

---

## 📄 Licence

Projet à usage interne. Adaptez cette section selon vos besoins (MIT, usage privé, etc.).
