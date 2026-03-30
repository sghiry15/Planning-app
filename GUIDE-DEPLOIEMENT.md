# Déployer Planify sur Azure App Service — Guide pas à pas

---

## Prérequis

- Un compte Azure actif (portal.azure.com)
- Azure CLI installé (ou utiliser Azure Cloud Shell)
- Un éditeur comme VS Code (optionnel mais recommandé)

---

## Étape 1 — Préparer les fichiers du projet

Votre projet doit contenir ces fichiers :

```
planning-azure/
├── index.html        ← votre application planning
├── server.js         ← serveur Node.js
├── package.json      ← dépendances et scripts
└── web.config        ← configuration IIS pour Azure
```

Ces fichiers sont inclus dans le zip fourni.

---

## Étape 2 — Se connecter à Azure

Ouvrez un terminal et connectez-vous :

```bash
az login
```

Une fenêtre de navigateur s'ouvre pour l'authentification.

Vérifiez votre souscription active :

```bash
az account show --query name -o tsv
```

---

## Étape 3 — Créer le Resource Group

```bash
az group create \
  --name rg-planning \
  --location westeurope
```

Vous pouvez changer `westeurope` par `francecentral` ou `northafrica`
selon vos préférences.

---

## Étape 4 — Créer le App Service Plan

```bash
az appservice plan create \
  --name plan-planning \
  --resource-group rg-planning \
  --sku F1 \
  --is-linux
```

- `F1` = tier gratuit (parfait pour commencer)
- Pour la production, utilisez `B1` (Basic) ou `S1` (Standard)
- `--is-linux` pour un hébergement Linux (plus performant pour Node.js)

---

## Étape 5 — Créer la Web App

```bash
az webapp create \
  --name planify-app \
  --resource-group rg-planning \
  --plan plan-planning \
  --runtime "NODE:18-lts"
```

IMPORTANT : le nom `planify-app` doit être unique sur tout Azure.
Ajoutez un suffixe unique, par exemple : `planify-app-cac`

Votre app sera accessible sur : https://planify-app.azurewebsites.net

---

## Étape 6 — Déployer le code

### Option A — Déploiement par ZIP (le plus rapide)

Créez un zip de votre projet :

```bash
cd planning-azure
zip -r ../planning.zip .
```

Déployez :

```bash
az webapp deploy \
  --resource-group rg-planning \
  --name planify-app \
  --src-path ../planning.zip \
  --type zip
```

### Option B — Déploiement depuis VS Code

1. Installez l'extension "Azure App Service" dans VS Code
2. Ouvrez le dossier du projet
3. Clic droit sur le dossier → "Deploy to Web App..."
4. Sélectionnez votre Web App
5. C'est déployé !

### Option C — Déploiement via Git local

```bash
# Activer le déploiement Git
az webapp deployment source config-local-git \
  --name planify-app \
  --resource-group rg-planning

# Récupérer l'URL Git (notez-la)
az webapp deployment list-publishing-credentials \
  --name planify-app \
  --resource-group rg-planning \
  --query scmUri -o tsv

# Dans votre dossier projet
git init
git add .
git commit -m "Initial deploy"
git remote add azure <URL_GIT_RÉCUPÉRÉE>
git push azure main
```

---

## Étape 7 — Vérifier le déploiement

Ouvrez votre navigateur :

```
https://planify-app.azurewebsites.net
```

Ou via CLI :

```bash
az webapp browse \
  --name planify-app \
  --resource-group rg-planning
```

Pour voir les logs en cas de problème :

```bash
az webapp log tail \
  --name planify-app \
  --resource-group rg-planning
```

---

## Étape 8 — Configurer un domaine personnalisé (optionnel)

Si vous voulez utiliser un sous-domaine de cac.ma :

```bash
# Ajouter le domaine
az webapp config hostname add \
  --webapp-name planify-app \
  --resource-group rg-planning \
  --hostname planning.cac.ma
```

Ajoutez un enregistrement DNS dans votre zone :

```
Type : CNAME
Nom  : planning
Valeur : planify-app.azurewebsites.net
```

---

## Étape 9 — Ajouter un certificat SSL (optionnel)

Comme vous maîtrisez déjà les App Service Certificates :

```bash
# Certificat managé gratuit (le plus simple)
az webapp config ssl create \
  --name planify-app \
  --resource-group rg-planning \
  --hostname planning.cac.ma

# Activer HTTPS uniquement
az webapp update \
  --name planify-app \
  --resource-group rg-planning \
  --https-only true
```

---

## Résumé des coûts

| Tier | Prix/mois | RAM  | Domaine custom | SSL  |
|------|-----------|------|----------------|------|
| F1   | Gratuit   | 1 GB | Non            | Non  |
| B1   | ~13 USD   | 1.75 GB | Oui         | Oui  |
| S1   | ~70 USD   | 1.75 GB | Oui         | Oui  |

Pour cette application statique, le tier F1 suffit pour tester.
Passez en B1 si vous avez besoin d'un domaine custom avec SSL.

---

## Commandes utiles

```bash
# Redémarrer l'app
az webapp restart --name planify-app --resource-group rg-planning

# Voir la config
az webapp config show --name planify-app --resource-group rg-planning

# Supprimer tout (si besoin)
az group delete --name rg-planning --yes
```
