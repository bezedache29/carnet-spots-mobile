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

- **`points`** : `id`, `owner_id`, `title`, `latitude`, `longitude`, `note`, `status` (`to_test` / `validated` / `to_avoid` / `already_slept`), `visited_at`, `visibility`
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

Le contrat côté API est déjà implémenté et documenté dans le README de `carnet-spots-api` (section 5.2) : c'est la référence en cas de doute.

- Couche de stockage local (SQLite, ex. via `op-sqlite` ou équivalent compatible RN bare).
- **Lecture/écriture 100% locale** sur le terrain — l'app ne dépend jamais du réseau pour fonctionner en mode consultation/mise à jour.

**Sync descendante** (`GET /api/v1/sync`) :

- On télécharge **tous** les points et tags de l'utilisateur, pas seulement la zone du road trip : quelques centaines de points pèsent quelques Ko, et un point déplacé hors d'un filtre de zone n'arriverait jamais sur le téléphone.
- Premier appel sans paramètre : tout est renvoyé. Les suivants envoient `since` = le `server_time` reçu à l'appel précédent, et ne reçoivent que ce qui a changé.
- Réponse : `server_time`, `points`, `tags`, et `deleted.points` / `deleted.tags` (des `{id, uuid}` à supprimer localement). Un même élément peut revenir dans deux syncs successives : toujours appliquer en _upsert_.
- Les points référencent leurs tags **par id** : garder les tags dans leur propre table, ne jamais copier le nom d'un tag dans un point. Pour un tag présent dans `deleted.tags`, le retirer soi-même des points locaux.

**File de mutations** (rejeu au retour du réseau) :

- Chaque changement fait hors ligne est mis en file, puis rejoué **un par un, dans l'ordre**, via les routes CRUD standard (`points`, `tags`). Un point qui utilise un tag créé hors ligne doit attendre la création de ce tag.
- **Créations** : le téléphone génère un `uuid`. Rejouer une création déjà reçue renvoie l'élément existant (200) au lieu d'un doublon.
- **Modifications** : envoyer `base_updated_at` = l'`updated_at` de la version modifiée sur le téléphone. Après chaque modification acceptée, remplacer `base_updated_at` dans **les autres changements en file du même élément** par le nouvel `updated_at` reçu, sinon la deuxième modification hors ligne d'un même point déclenche un faux conflit.
- **Suppressions** : _soft delete_ côté API. Supprimer un élément déjà supprimé renvoie 204.
- Que faire de chaque réponse :
  - **2xx** : terminé, retirer de la file.
  - **409** : conflit, l'élément a changé sur le serveur entre-temps. Rien n'est enregistré, la réponse contient la version actuelle : demander à l'utilisateur laquelle garder.
  - **404** : l'élément a été supprimé sur le serveur, la suppression gagne. Abandonner le changement et supprimer l'élément local.
  - **422** : le changement ne passera jamais tel quel (ex. nom de tag déjà pris). Le montrer à l'utilisateur pour qu'il le corrige ou l'abandonne. Les changements qui en dépendent attendent avec lui.
  - **Erreur réseau / 5xx** : garder en file et réessayer plus tard.
- Seules les dates du serveur sont comparées : l'horloge du téléphone n'a aucune importance.

## 6. Fonctionnalités attendues pour ce repo (roadmap V1)

- Carte GPS (MapLibre Native + `.pmtiles` local)
- Ajout de point par tap sur la carte ou par position actuelle
- Liste des points triée par distance
- Consultation/mise à jour de statut **100% offline**
- Sync avant départ (téléchargement de tous les points + `.pmtiles` de la zone)
- Rejeu de la file de mutations au retour du réseau

## 7. Hors scope V1 (à ne pas anticiper dans le code maintenant)

- Inscription multi-utilisateurs, policies d'accès croisé, modération des points publics
- Landing page / blog (hors scope, autre repo)
- Photos attachées aux points
- Notifications ("point à proximité")
- iOS (pas de Mac disponible)
