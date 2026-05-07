# Setup Instructions - Cabinet Dentaire Chatbot

## 1. Import du Workflow

1. Aller sur `app.n8n.cloud`
2. Menu → **Workflows** → **+** → **Import from JSON**
3. Coller le contenu de `n8n_workflow_cabinet_dentaire.json`
4. Cliquer **Import**

---

## 2. Configurer les Credentials (obligatoire)

### 2a. Google Sheets
- Ouvrir le node **Google Sheets - Lire FAQ**
- Cliquer sur **Credentials** → **Create New**
- Choisir **Google Sheets OAuth2**
- Suivre l'authentification Google
- **IMPORTANT:** Remplacer `REMPLACER_PAR_VOTRE_SPREADSHEET_ID` par l'ID de votre Google Sheet
  - L'ID se trouve dans l'URL: `https://docs.google.com/spreadsheets/d/**{SPREADSHEET_ID}**/edit`

### 2b. Anthropic (Claude AI)
- Ouvrir le node **Anthropic API - Claude AI**
- Cliquer sur **Credentials** → **Create New**
- Choisir **HTTP Header Auth**
- Name: `x-api-key`
- Value: Votre clé API (`sk-ant-...`)

### 2c. Google Calendar
- Ouvrir le node **Google Calendar - Créer RDV**
- Cliquer sur **Credentials** → **Create New**
- Choisir **Google Calendar OAuth2**
- Suivre l'authentification Google

### 2d. Gmail
- Ouvrir le node **Gmail - Confirmation Patient**
- Cliquer sur **Credentials** → **Create New**
- Choisir **Gmail OAuth2**
- Se connecter avec `renaissanceitech@gmail.com`

### 2e. Slack
- Créer une Slack App sur https://api.slack.com/apps
- Ajouter les permissions: `chat:write`, `chat:write.public`
- Copier le **Bot Token** (`xoxb-...`)
- Dans le node **Slack - Notification Équipe** → Credentials → paste le token
- S'assurer que le channel `#appointments` existe dans le workspace

---

## 3. Structure Google Sheets requise

Votre spreadsheet doit avoir ces 3 colonnes exactes en ligne 1 (headers):

| A | B | C |
|---|---|---|
| Catégorie | Question | Réponse |
| Horaires | Vous ouvrez quand? | Lun-Ven 9h-18h, Sam 9h-12h |
| ... | ... | ... |

---

## 4. Tester le Workflow

### Test 1 - FAQ Simple
```
Message: "Vous ouvrez le samedi?"
Attendu: Réponse sur les horaires du samedi
Nodes actifs: Chat → Sheets → Code → Claude → Code → IF (false)
```

### Test 2 - Réservation Complète
```
Message: "Je veux un détartrage vendredi à 14h, je m'appelle Jean Martin, 06-12-34-56-78"
Attendu:
  - Réponse de confirmation
  - Événement Google Calendar créé
  - Email envoyé à renaissanceitech@gmail.com
  - Message Slack dans #appointments
Nodes actifs: Tous les 9 nodes
```

### Test 3 - Urgence
```
Message: "J'ai très mal aux dents urgence!"
Attendu: "Appelez 06-98-70-77-21 IMMÉDIATEMENT"
Nodes actifs: Chat → Sheets → Code → Claude → Code → IF (false)
```

---

## 5. Activer le Workflow

1. Toggle **Active** = **ON**
2. Copier l'URL du Chat Trigger
3. Partager avec les patients

---

## Architecture des 9 Nodes

```
Chat Trigger
    ↓
Google Sheets - Lire FAQ
    ↓
Code - Préparer Prompt Claude
    ↓
Anthropic API - Claude AI
    ↓
Code - Extraire Réservation
    ↓
IF - Réservation Détectée?
    ↓ (OUI)           ↓ (NON)
┌───┬───┬───┐      [Fin - réponse chat]
Cal Gml Slk
```
