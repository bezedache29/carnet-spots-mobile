# Carnet de spots — Design system

> Référence visuelle commune à `carnet-spots-web` et `carnet-spots-mobile` (les deux fichiers ont le même contenu, sauf la section 9). À appliquer quand on construira l'interface : rien de tout ça n'est encore dans le code.

## 1. Ambiance

Carnet de voyage personnel pour la vanlife : chaleureux, simple, fiable. Pas « startup tech », pas « outdoor agressif ». L'élément central de l'interface est la **carte** avec des marqueurs colorés selon le statut du spot.

Contraintes qui guident tous les choix :

- **Lisible en plein soleil** sur téléphone : contrastes WCAG AA (≥ 4,5:1 pour le texte), cibles tactiles assez grandes.
- **Mode sombre obligatoire** : utilisé le soir dans le van, il ne doit pas éblouir.
- **Jamais la couleur seule** pour porter une information : chaque statut a aussi une icône (daltonisme).
- **Le hors ligne n'est pas une erreur** : il se signale en gris, calmement.

## 2. Couleurs

Tous les ratios de contraste ci-dessous ont été **recalculés** (formule WCAG), pas recopiés des maquettes : les images générées par l'IA de design affichaient des ratios faux.

### 2.1 Marque et neutres

| Token                | Clair     | Sombre    | Usage                                        |
| -------------------- | --------- | --------- | -------------------------------------------- |
| `primary` (Forêt)    | `#0F5132` | `#29A56B` | Bouton principal, liens, éléments actifs     |
| `secondary` (Lac)    | `#287CA0` | `#38B8F5` | Bouton secondaire, tags, information         |
| `accent` (Soleil)    | `#F59E0B` | `#F59E0B` | Touches de marque (logo, illustrations)      |
| `background`         | `#FFFFFF` | `#0B1220` | Fond de l'application                        |
| `surface`            | `#F8FAFC` | `#1E293B` | Cartes, panneaux, barre de navigation        |
| `surface-2`          | `#F3F4F6` | `#334155` | Surface en retrait (champ, ligne survolée)   |
| `border`             | `#CBD5E1` | `#475569` | Séparateurs, contours de cartes              |
| `border-strong`      | `#64748B` | `#8392A6` | Contour des champs de formulaire (≥ 3:1)     |
| `text`               | `#111827` | `#F8FAFC` | Texte principal                              |
| `text-secondary`     | `#475569` | `#CBD5E1` | Texte secondaire, légendes                   |
| `text-disabled`      | `#94A3B8` | `#64748B` | Éléments désactivés — **jamais** pour du texte à lire (2,6:1) |

Couleurs d'appoint de la palette (illustrations, fonds décoratifs, pas de texte dessus) : Pierre `#E5E7EB`, Ardoise `#334155`, Noir `#111827`.

### 2.2 Statuts des spots

La clé est la valeur renvoyée par l'API (`status` d'un point).

| Statut API      | Libellé     | Clair     | Sombre    | Icône           |
| --------------- | ----------- | --------- | --------- | --------------- |
| `to_test`       | À tester    | `#B8860B` | `#BA880B` | point d'interrogation |
| `validated`     | Validé      | `#2B8172` | `#36A28F` | coche           |
| `to_avoid`      | À éviter    | `#D1411D` | `#E76D4F` | croix           |
| `already_slept` | Déjà dormi  | `#196CF4` | `#5190F7` | lit             |

### 2.3 États

| Token      | Clair     | Sombre    | Usage                                      |
| ---------- | --------- | --------- | ------------------------------------------ |
| `success`  | `#12863D` | `#17A84C` | Synchronisé, enregistré                    |
| `error`    | `#B91C1C` | `#E86A6A` | Échec, champ invalide (422)                |
| `warning`  | `#F59E0B` | `#F59E0B` | Conflit de modification (409), attention   |
| `offline`  | `#64748B` | `#8392A6` | Hors ligne, modifications en attente       |
| `info`     | `#287CA0` | `#38B8F5` | Message neutre                             |
| `focus`    | `#3B82F6` | `#3B82F6` | Anneau de focus clavier                    |

### 2.4 Couleur du texte posé sur une couleur (`on-*`)

Chaque couleur pleine (badge, bouton, bandeau) a sa couleur de texte associée :

- **Mode clair** : texte **blanc** `#FFFFFF`, sauf sur `accent`, `warning`, `to_test` et `focus` → texte **foncé** `#111827`.
- **Mode sombre** : texte **foncé** `#111827` sur toutes les couleurs pleines.

Ratios obtenus (tous ≥ 4,5:1) :

| Fond (clair)                                   | Texte    | Ratio |
| ---------------------------------------------- | -------- | ----- |
| `primary`                                      | blanc    | 9,4   |
| `secondary`, `validated`, `to_avoid`, `already_slept`, `success` | blanc | 4,7 |
| `offline`                                      | blanc    | 4,8   |
| `error`                                        | blanc    | 6,5   |
| `accent` / `warning`                           | foncé    | 8,3   |
| `to_test`                                      | foncé    | 5,5   |
| `focus`                                        | foncé    | 4,8   |

En mode sombre, chaque couleur de statut et d'état atteint ≥ 4,6:1 posée sur `surface`, et ≥ 5,6:1 avec le texte foncé dessus.

## 3. Typographie

| Rôle                                      | Police                | Source                        |
| ----------------------------------------- | --------------------- | ----------------------------- |
| Titres, boutons, éléments importants      | **Plus Jakarta Sans** | Google Fonts, gratuite (OFL)  |
| Texte, interface, légendes                | **Inter**             | Google Fonts, gratuite (OFL)  |

| Token     | Taille / interligne (px) | Graisse  | Police            |
| --------- | ------------------------ | -------- | ----------------- |
| `h1`      | 32 / 40                  | Bold     | Plus Jakarta Sans |
| `h2`      | 24 / 32                  | Semibold | Plus Jakarta Sans |
| `h3`      | 20 / 28                  | Semibold | Plus Jakarta Sans |
| `h4`      | 16 / 24                  | Semibold | Plus Jakarta Sans |
| `body`    | 16 / 24                  | Regular  | Inter             |
| `small`   | 14 / 20                  | Regular  | Inter             |
| `caption` | 12 / 16                  | Regular  | Inter             |
| `label`   | 12 / 16                  | Semibold | Inter             |

## 4. Espacements, arrondis, ombres

**Espacements** (grille de 4 px) : `xs` 4 · `sm` 8 · `md` 12 · `lg` 16 · `xl` 24 · `2xl` 32 · `3xl` 48 · `4xl` 64.

**Arrondis** : `none` 0 · `sm` 4 · `md` 8 · `lg` 12 · `xl` 16 · `2xl` 24 · `full` 9999.

- Cartes : `lg` (12)
- Boutons : `lg` (12)
- Champs de saisie : `lg` (12)
- Badges et tags : `full`

**Ombres** :

| Token | Clair                            | Sombre                          | Usage                     |
| ----- | -------------------------------- | ------------------------------- | ------------------------- |
| `sm`  | `0 1px 2px rgba(0,0,0,0.05)`     | `0 1px 2px rgba(0,0,0,0.4)`     | Cartes de liste           |
| `md`  | `0 4px 6px rgba(0,0,0,0.08)`     | `0 4px 6px rgba(0,0,0,0.5)`     | Pop-up sur la carte       |
| `lg`  | `0 8px 16px rgba(0,0,0,0.10)`    | `0 8px 16px rgba(0,0,0,0.6)`    | Modale, panneau flottant  |

En mode sombre, les ombres se voient peu : on distingue surtout les niveaux par la couleur de surface (`surface` puis `surface-2`).

## 5. Icônes

- Style **trait (outline)**, épaisseur régulière, coins arrondis.
- Taille **24 px**, **20 px** en contexte compact.
- Icônes utiles : pin, maison, loupe, plus, crayon, corbeille, étoile, attention, lune, lit, wifi, wifi barré, réglages, téléchargement, synchronisation, menu, nuage barré (hors ligne).
- Aucune bibliothèque d'icônes n'est choisie : à proposer à Christophe avant installation.

## 6. Marqueurs de carte

- Forme de **goutte (pin)** remplie de la couleur du statut (section 2.2), avec l'**icône du statut en blanc** au centre (texte foncé pour `to_test` en clair).
- Contour blanc fin pour se détacher du fond de carte OpenStreetMap (routes, forêts, eau).
- **Ma position** : point bleu `focus` `#3B82F6` cerclé de blanc, avec un halo translucide.
- En mode sombre, utiliser les couleurs sombres des statuts.

## 7. Composants

| Composant              | Règles                                                                                          |
| ---------------------- | ----------------------------------------------------------------------------------------------- |
| Bouton principal       | Fond `primary`, texte `on-primary`, arrondi `lg`                                                 |
| Bouton secondaire      | Contour `primary`, fond transparent, texte `primary`                                            |
| Bouton texte           | Pas de fond ni de contour, texte `primary`                                                      |
| Bouton désactivé       | Fond `surface-2`, texte `text-disabled`                                                         |
| Champ de saisie        | Fond `background`, contour `border-strong`, arrondi `lg`, contour `focus` au focus, `error` si invalide |
| Badge de statut        | Fond couleur du statut, icône + libellé, texte `on-*`, arrondi `full`                           |
| Tag                    | Fond `secondary`, texte `on-secondary`, arrondi `full`                                          |
| Carte de spot (liste)  | Fond `surface`, ombre `sm`. Titre, distance + coordonnées, badge de statut, tags, extrait de note, date d'ajout. **Variante sans photo par défaut** (vignette illustration neutre) |
| Bandeau hors ligne     | Fond `offline`, icône nuage barré, « Vous êtes hors ligne — vos modifications seront synchronisées dès que vous aurez de nouveau un réseau » |
| Indicateur de synchro  | « N modifications en attente · Dernière synchro : HH:MM » (`offline`) ou « Synchronisé » (`success`) |
| Conflit (409)          | Encadré `warning` : « Ce spot a été modifié ailleurs » + boutons « Garder ma version » / « Garder l'autre » |
| Région de carte        | Nom de la région, taille téléchargée / totale, barre de progression `primary`, bouton annuler / supprimer |

## 8. Hors périmètre V1

Vus dans les maquettes, mais **à ne pas construire** en V1 (l'API ne les gère pas) :

- Favoris (étoile sur les spots).
- Champs « Accès » et « Équipements » sur la fiche d'un spot.
- Photos des spots (V2) : d'où la carte de spot sans photo par défaut.

**Logo** : pas encore choisi (pistes : carnet + pin, ou paysage avec chemin). Utiliser un emplacement provisoire.

## 9. Mise en œuvre sur ce repo (mobile — React Native CLI, Android)

- Regrouper les tokens dans un **objet de thème** (une version claire, une version sombre), choisi selon `useColorScheme()` de React Native, avec un réglage manuel possible plus tard.
- Les composants n'utilisent **que les tokens** du thème, jamais un code hex en dur.
- **Lisibilité en plein soleil** : cibles tactiles d'au moins 48 × 48 dp, texte courant jamais sous `body` (16) sur les écrans de terrain, boutons d'action principaux bien visibles (bouton `+` central de la barre de navigation).
- **Barre de navigation** : Carte · Liste · `+` (ajouter un spot) · Cartes hors ligne · Profil.
- **MapLibre Native** : les couleurs des marqueurs viennent des tokens de statut. Le style de carte doit avoir une variante sombre pour le mode sombre.
- **Hors ligne** : le bandeau hors ligne et l'indicateur de synchro (section 7) sont des éléments permanents de l'interface, pas des erreurs. Les états de la file de mutations s'affichent ainsi :
  - réponse 409 → écran de conflit (`warning`) ;
  - réponse 422 → modification à corriger ou à abandonner (`error`) ;
  - en attente de réseau → `offline`.
- **Polices** : Plus Jakarta Sans et Inter en fichiers `.ttf` dans les assets du projet, liés avec `react-native.config.js` + `npx react-native-asset`. Demander l'accord de Christophe avant d'ajouter un outil ou un paquet pour ça.
