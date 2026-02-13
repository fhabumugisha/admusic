# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AD Music is a JSON data repository serving as the backend for an event management system. It uses [My JSON Server](https://my-json-server.typicode.com/) (typicode) to expose `db.json` as a REST API without any build step or server code.

- **Live API:** `https://my-json-server.typicode.com/fhabumugisha/admusic`
- **GitHub Pages (images):** `https://fhabumugisha.github.io/admusic/`

## Running Locally

```bash
npx json-server db.json
```

No package.json, no build tools, no dependencies to install.

## Architecture

The entire backend is a single `db.json` file with two top-level keys:

- **`contact`** — Single object with company info and social media links (TikTok, Instagram, Facebook, Twitter)
- **`events`** — Array of event objects

JSON Server auto-generates REST endpoints: `GET /contact`, `GET /events`, `GET /events/:id`, etc.

## Event Schema

| Field | Type | Required | Notes |
|---|---|---|---|
| `id` | number | yes | Sequential integer |
| `title` | string | yes | Event name |
| `location` | string | yes | Venue and address |
| `date` | ISO 8601 | yes | Event date |
| `dateDisplay` | string | yes | Human-readable date shown in UI |
| `cover` | URL | yes | GitHub Pages URL to image in repo root |
| `admusicEvent` | boolean | yes | `true` if organized by AD Music |
| `description` | string | no | Event details |
| `heureDebut` | ISO 8601 | no | Start time |
| `heureFin` | ISO 8601 | no | End time |
| `booklink` | URL | no | Ticket/booking link |

## Image Convention

- **Format du nom** : `DDMMYYYY.jpeg` (toujours DDMMYYYY, toujours `.jpeg`)
- Stockées à la racine du repo
- **URL** : `https://fhabumugisha.github.io/admusic/DDMMYYYY.jpeg`

> Note : certaines anciennes images utilisent `YYYYMMDD.jpg`. Pour les nouveaux événements, toujours utiliser `DDMMYYYY.jpeg`.

## Format dateDisplay standardisé

Le champ `dateDisplay` est affiché tel quel dans l'app mobile. Format standardisé :

```
Jour DD mois YYYY à HH:MM
```

**Règles :**
- Jour de la semaine en français, majuscule initiale : Lundi, Mardi, Mercredi, Jeudi, Vendredi, Samedi, Dimanche
- Jour du mois sur 2 chiffres : 01, 06, 14, 31
- Mois en français en minuscules : janvier, février, mars, avril, mai, juin, juillet, août, septembre, octobre, novembre, décembre
- Année sur 4 chiffres
- `à HH:MM` uniquement si l'heure de début est connue

**Exemples :**
- `Samedi 06 décembre 2025 à 21:00`
- `Vendredi 31 octobre 2025 à 22:00`
- `Dimanche 12 février 2023` (pas d'heure connue)

## Ajouter un événement depuis un flyer (Workflow Claude Code)

### Étapes

1. **Placer l'image du flyer** dans la racine du repo avec le nom `DDMMYYYY.jpeg` (date de l'événement)
2. **Demander à Claude Code** : *"Lis le flyer `DDMMYYYY.jpeg` et ajoute l'événement dans db.json"*
3. **Claude Code** va :
   - Lire l'image et extraire les informations (titre, lieu, date, horaires, description, lien de réservation)
   - Déterminer le prochain `id` séquentiel (max existant + 1)
   - Générer l'objet JSON complet
   - Construire l'URL cover : `https://fhabumugisha.github.io/admusic/DDMMYYYY.jpeg`
   - Formater `dateDisplay` selon le format standardisé
   - Présenter les données extraites pour vérification humaine
   - Signaler les champs manquants ou incertains
4. **Vérifier** les données extraites (surtout date, lieu et admusicEvent)
5. **Claude Code** insère l'événement dans `db.json`
6. **Commit et push**

### Informations à extraire du flyer

| Information | Où chercher sur le flyer |
|---|---|
| Titre | Texte principal / nom de l'événement |
| Lieu | Nom de la salle + adresse complète |
| Date | Date affichée |
| Horaires | Heure d'ouverture des portes / début |
| Description | Détails, artistes, DJ, dress code |
| admusicEvent | Présence du logo ou mention "AD Music" |

### Template JSON

```json
{
  "admusicEvent": true,
  "id": 17,
  "title": "TITRE DE L'ÉVÉNEMENT",
  "description": "Description détaillée de l'événement",
  "location": "NOM DE LA SALLE, ADRESSE, CODE POSTAL VILLE",
  "date": "2025-12-06T23:59:59.000Z",
  "dateDisplay": "Samedi 06 décembre 2025 à 21:00",
  "heureDebut": "2025-12-06T21:00:00.000Z",
  "heureFin": "2025-12-07T05:00:00.000Z",
  "cover": "https://fhabumugisha.github.io/admusic/06122025.jpeg",
  "booklink": "https://linktr.ee/intladmusic"
}
```

### Règles pour chaque champ

| Champ | Règle |
|---|---|
| `id` | Max existant + 1 |
| `title` | Titre principal du flyer, en majuscules |
| `location` | Adresse complète en majuscules si disponible |
| `date` | Date de l'événement au format `YYYY-MM-DDT23:59:59.000Z` |
| `dateDisplay` | Format standardisé (voir section ci-dessus) |
| `heureDebut` | ISO 8601 UTC, ex: `2025-12-06T21:00:00.000Z`. Si pas visible, demander |
| `heureFin` | Généralement le lendemain à 05:00 UTC |
| `cover` | `https://fhabumugisha.github.io/admusic/DDMMYYYY.jpeg` |
| `booklink` | Toujours `https://linktr.ee/intladmusic` |
| `admusicEvent` | `true` si logo/mention AD Music visible, sinon demander |
| `description` | Optionnel. Détails : artistes, DJ, dress code, infos pratiques |
