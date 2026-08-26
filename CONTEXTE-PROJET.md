# Carnet de spots — Mobile (React Native CLI bare)

> Document de référence pour Claude Code sur ce repo. Ce repo est l'app **terrain**, conçue pour fonctionner intégralement **sans réseau** une fois sur place.

## 1. Contexte & besoin du projet

App personnelle (usage solo en V1) pour marquer des points GPS précis (parkings, spots de nuit, emplacements repérés) même sans adresse existante (ex. en pleine forêt), avec une note libre, pour les retrouver et les consulter une fois sur le terrain sans réseau.

Ce repo couvre la phase **terrain** (dans le fourgon, téléphone, souvent sans réseau) : consultation des points déjà posés, navigation GPS vers un point, mise à jour du statut sur place (validé, à éviter, déjà dormi). **Le offline n'est pas une fonctionnalité secondaire ici, c'est la contrainte structurante de tout le repo.**

## 2. Rôle de ce repo dans l'architecture globale

**3 repos séparés**, une seule API comme source de vérité :

- `carnet-spots-api` (Laravel) — source de vérité, contactée par ce repo uniquement en amont (sync avant départ) et en aval (rejeu des mutations au retour du réseau)
- `carnet-spots-web` (React + Vite) — usage préparation, pas concerné par ce repo
- `carnet-spots-mobile` (ce repo, React Native CLI bare) — usage terrain

## 3. Décisions techniques actées pour ce repo

| Sujet           | Décision                                              | Raison                                                                                                               |
| --------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Framework       | **React Native CLI bare — pas Expo**                  | Choix de dev tranché et non négociable ; contrôle natif total nécessaire pour MapLibre Native                        |
| Plateforme      | **Android uniquement**                                | Pas de Mac/iPhone disponible, ciblé Google Pixel 9a                                                                  |
| Distribution    | Sideload (APK direct)                                 | Pas de Play Store en V1                                                                                              |
| Carte           | **MapLibre Native**                                   | Cohérence avec MapLibre GL JS côté web ; support natif du offline via fichiers locaux                                |
| Tuiles offline  | **`.pmtiles` (Protomaps)**, lu en local via `file://` | Le tile server officiel OSM interdit le téléchargement en masse/automatisé pour usage offline — voir section 5.1     |
| Données offline | **SQLite local** + file de mutations                  | Le offline ne concerne pas que la carte, mais aussi la lecture/écriture des points sur le terrain — voir section 5.2 |
| Auth            | Token **Laravel Sanctum** côté API, stocké localement | Écosystème Laravel natif                                                                                             |

## 4. Modèle de données consommé (répliqué localement en SQLite)

Ce repo réplique localement les ressources exposées par `carnet-spots-api` pour permettre lecture/écriture offline :

- **`points`** : `id`, `owner_id`, `title`, `latitude`, `longitude`, `note`, `status` (`a_tester` / `valide` / `a_eviter` / `deja_dormi`), `visited_at`, `visibility`
- **`tags`** : `id`, `owner_id`, `name` (unique par owner)
- **`point_tag`** : relation many-to-many
- **`photos`** (V2, hors scope V1)

## 5. Stratégie offline — cœur de ce repo

### 5.1 Tuiles cartographiques — Protomaps / `.pmtiles`

- OpenStreetMap reste la source de données ; seule l'infrastructure de distribution change (pas le tile server officiel OSM, non autorisé pour du cache offline en masse).
- Avant chaque road trip : génération d'un fichier `.pmtiles` régional (extraction ciblée sur la zone, pas la planète), téléchargé depuis `carnet-spots-api` (au wifi) et stocké sur l'appareil.
- **MapLibre Native lit le fichier en local via le protocole `file://`** — aucun serveur de tuiles requis à l'exécution, donc zéro dépendance réseau une fois sur le terrain.
- Attribution ODbL (`© OpenStreetMap contributors`) — gérée par défaut par les styles MapLibre standards.
- Gratuit : le fichier vit directement sur le device, pas d'hébergement cloud nécessaire pour ce usage solo.

### 5.2 Données applicatives — SQLite local + file de mutations

- Couche de stockage local (SQLite, ex. via `op-sqlite` ou équivalent compatible RN bare).
- **Sync descendante** : au wifi avant le départ, les points pertinents (zone du road trip) sont téléchargés depuis l'API et stockés localement.
- **Lecture/écriture 100% locale** sur le terrain — l'app ne dépend jamais du réseau pour fonctionner en mode consultation/mise à jour.
- **File de mutations (mutation queue)** : chaque changement fait hors ligne (statut, note) est stocké localement avec horodatage, puis rejoué vers `carnet-spots-api` dès que le réseau est retrouvé.
- Stratégie de conflit à définir précisément avant implémentation (probable : _last-write-wins_ horodaté, vu l'usage solo — pas de vrai risque de conflit concurrent multi-device en V1, mais à documenter explicitement dans le code).

## 6. Fonctionnalités attendues pour ce repo (roadmap V1)

- Carte GPS (MapLibre Native + `.pmtiles` local)
- Ajout de point par tap sur la carte ou par position actuelle
- Liste des points triée par distance
- Consultation/mise à jour de statut **100% offline**
- Sync avant départ (téléchargement points + `.pmtiles` de la zone)
- Rejeu de la file de mutations au retour du réseau

## 7. Hors scope V1 (à ne pas anticiper dans le code maintenant)

- Inscription multi-utilisateurs, policies d'accès croisé, modération des points publics
- Landing page / blog (hors scope, autre repo)
- Photos attachées aux points
- Notifications ("point à proximité")
- iOS (pas de Mac disponible)
