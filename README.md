# JCO NY Amplitude Levels

![JCO NY Amplitude Levels](screenshot.png)

Indicateur TradingView Pine Script v6 pour le scalp contrarien sur NQ Futures (Nasdaq 100).

Il trace les niveaux de Fibonacci basés sur le range Pre-NY (ouverture CME Globex → ouverture NY AM), avec dashboard d'amplitude intégré incluant une recommandation d'amplitude par niveau de risque basée sur les statistiques NQ 2024–2026.

---

## Principe

Chaque nuit, le NQ trace un range entre l'ouverture de la session CME Globex (22h UTC) et l'ouverture de la session NY AM (15h30 Paris par défaut). Ce range Pre-NY constitue la base des niveaux Fibonacci affichés en pré-session.

L'indicateur fonctionne en **deux modes** :

- **Mode Pre-NY (avant 15h30 Paris)** : les niveaux Fibonacci sont **dynamiques** — ils suivent le High/Low du Pre-NY en temps réel, visibles à partir de 7h Paris pour préparer la session.
- **Mode Post-NY (à partir de 15h30)** : les niveaux Fibo disparaissent et sont remplacés par 3 lignes horizontales tracées sur la **journée TradingView complète** :
  - Day High (orange)
  - Day Low (orange)
  - Day 50% (vert) — milieu du range du jour

Les Day High/Low sont mis à jour en temps réel pendant la session NY si de nouveaux extrêmes sont atteints.

L'ordre chronologique des extrêmes (High avant Low ou Low avant High) détermine la direction et l'orientation du Fibonacci en mode Pre-NY.

---

## Niveaux tracés (Mode Pre-NY)

### Retracements (contre-tendance)

| Niveau | Couleur par défaut | Description                                    |
|--------|--------------------|------------------------------------------------|
| 0%     | Orange             | Extrémité récente (départ des retracements)    |
| 38.2%  | Rouge              | Premier retracement                            |
| 50.0%  | Vert foncé         | Retracement médian                             |
| 61.8%  | Bleu               | OTE+ (Optimal Trade Entry)                     |
| 78.6%  | Violet             | OTE-                                           |
| 100%   | Orange             | Extrémité ancienne (retracement complet)       |

### Extensions sous 0% (continuation tendance Pre-NY)

| Niveau | Couleur par défaut | Style par défaut | Position (DOWN)                   |
|--------|--------------------|------------------|-----------------------------------|
| -27.2% | Gris               | Tirets           | Low − 0.272 × range (sous le Low) |
| -61.8% | Gris               | Tirets           | Low − 0.618 × range               |
| -100%  | Gris               | Pointillés       | Low − 1.000 × range               |

### Extensions au-dessus de 100% (cassure contre Pre-NY)

| Niveau | Couleur par défaut | Style par défaut | Position (DOWN)                          |
|--------|--------------------|------------------|------------------------------------------|
| 127.2% | Gris               | Tirets           | High + 0.272 × range (au-dessus du High) |
| 161.8% | Gris               | Tirets           | High + 0.618 × range                     |
| 200.0% | Gris               | Pointillés       | High + 1.000 × range                     |

L'échelle de pourcentage est mathématiquement cohérente : 0% = extrême récent, 100% = extrême ancien, valeurs négatives sous 0%, valeurs > 100% au-dessus.

---

## Niveaux tracés (Mode Post-NY)

À partir de 15h30 Paris, ces 3 lignes remplacent les Fibo et restent visibles jusqu'à la fin de la session NY PM :

| Ligne     | Couleur par défaut | Description                                       |
|-----------|--------------------|---------------------------------------------------|
| Day High  | Orange             | Plus haut de la journée TradingView complète      |
| Day Low   | Orange             | Plus bas de la journée TradingView complète       |
| Day 50%   | Vert foncé         | Milieu du range du jour : (High + Low) / 2        |

---

## Midnight NY Open

Ligne horizontale au prix d'ouverture de la barre 0h00 NY (configurable). Utile comme référence de gap overnight.

- Timezone configurable (New York, Paris, Londres, Tokyo, etc.)
- Heure et minute configurables
- Style, couleur et épaisseur configurables

---

## Dashboard

Affiché en bas à droite du graphique. **3 modes** disponibles via la liste déroulante "Mode d'affichage" :

### Mode Complet

| Ligne            | Colonnes                                          | Description                                        |
|------------------|---------------------------------------------------|----------------------------------------------------|
| Direction Pre-NY | UP / DOWN — Jour — Dynamique/Figé                 | Direction, jour de la semaine, état                |
| Amplitude Pre-NY | ex. 121 pts                                       | Range total High–Low                               |
| Faible P90       | 10% dép. — Ratio — Amp. NY — **Amp. Reco**        | 10% des jours dépassent cette amplitude            |
| Modéré P75       | 25% dép. — Ratio — Amp. NY — **Amp. Reco**        | 25% des jours dépassent cette amplitude            |
| Fort P50         | 50% dép. — Ratio — Amp. NY — **Amp. Reco**        | 50% des jours dépassent cette amplitude            |

### Mode Simplifié

Affichage compact en 2 colonnes, sans légendes — pour utilisateurs avancés qui veulent économiser l'espace à droite du graphique :

| Ligne | Colonne 0                | Colonne 1                   |
|-------|--------------------------|-----------------------------|
| 0     | UP ▲ / DOWN ▼            | Figé / Dynamique            |
| 1     | « Amp. Pre-NY » (label)  | Valeur Pre-NY (ex. 121 pts) |
| 2     | « Amp. Reco » (header)   | « Amp. NY » (header)        |
| 3     | Amp. Reco Faible (vert)  | Amp. NY Faible (vert)       |
| 4     | Amp. Reco Modéré (jaune) | Amp. NY Modéré (jaune)      |
| 5     | Amp. Reco Fort (rouge)   | Amp. NY Fort (rouge)        |

### Mode Masquer

Cache complètement le dashboard.

---

Les ratios et l'amplitude recommandée sont calculés par jour de la semaine à partir des percentiles P90/P75/P50 issus des statistiques NQ (2 jan. 2024 – 2 avr. 2026, 581 jours).

**Formule amplitude recommandée** : `round(0.1965 × ratio × Amplitude Pre-NY)` — 0.1965 étant la moyenne des 4 distances inter-niveaux Fibo (0→38.2→50→61.8→78.6%).

---

## Paramètres

### Session NY AM

- **Heure debut NY AM (Paris)** : heure d'ouverture NY en heure de Paris (défaut : 15)
- **Minute debut NY AM** : minute (défaut : 30)

### Paramètres Dashboard

- **Mode d'affichage** : liste déroulante — `Masquer` / `Complet` / `Simplifié`

### Retracements

Chaque niveau (0%, 100%, 38.2%, 50%, 61.8%, 78.6%) est configurable indépendamment :

- Afficher/masquer
- Couleur
- Épaisseur (1–4)
- Style (solid / dashed / dotted)

### Extensions

**Toggles maîtres** (en haut du groupe) :

- **Afficher extensions sous 0%** : masque/affiche d'un coup les 3 lignes -27.2% / -61.8% / -100%
- **Afficher extensions au-dessus 100%** : masque/affiche d'un coup les 3 lignes 127.2% / 161.8% / 200%

Chaque niveau individuel reste configurable indépendamment (afficher, couleur, épaisseur, style).

### Mode Post-NY

Chaque ligne (Day High, Day Low, Day 50%) est configurable indépendamment :

- Afficher/masquer
- Couleur, épaisseur, style

### Paramètres Midnight NY Open

- Afficher/masquer la ligne
- Couleur, épaisseur, style
- Timezone (New York, Chicago, Los Angeles, London, Paris, Berlin, Tokyo, Hong Kong, Sydney, UTC)
- Heure (0–23) et Minute (0–59)

---

## Installation

1. Ouvrir TradingView → Pine Script Editor
2. Coller le contenu de `Indicator_JCO_NY_Amplitude_Levels.pine`
3. Cliquer sur **Ajouter au graphique**
4. Appliquer sur un graphique NQ Futures (CME) en timeframe 1m à 15m

---

## Compatibilité

- **Symbole recommandé** : NQ1! / MNQ1! (CME Globex)
- **Timeframes recommandés** : 1m, 2m, 5m, 15m
- **Pine Script** : v6

---

## Changelog

### v1.7 - 2026-05-08

- Mode simplifié : passe de 1 colonne à 2 colonnes (Amp. Reco à gauche, Amp. NY à droite)
- Mode simplifié : ajout de l'état Figé/Dynamique en haut à droite, et séparation du label « Amp. Pre-NY » (col 0) et de sa valeur (col 1)

### v1.6 - 2026-05-06

- Dashboard : case à cocher remplacée par liste déroulante (Masquer / Complet / Simplifié)
- Mode simplifié : affichage compact en 1 colonne sans légendes (direction, amplitude Pre-NY, header « Amp. Reco » et 3 amplitudes recommandées)
- Toggles maîtres pour afficher/masquer rapidement les groupes d'extensions (sous 0% et au-dessus 100%)
- Couleurs unifiées : vert `#3fb950` pour tous les éléments verts ; jaune assombri `#c99a2c`

### v1.5 - 2026-05-05

- Renommage des extensions sous 0% : `127.2%`/`161.8%`/`200%` deviennent `-27.2%`/`-61.8%`/`-100%` (échelle de pourcentage cohérente)
- Ajout de 3 nouvelles extensions au-dessus du 100% : `127.2%`/`161.8%`/`200%` (miroir des extensions sous 0%, marquent une cassure contre la tendance Pre-NY)

### v1.4 - 2026-04-28

- Mode Post-NY : à partir de 15h30, les niveaux Fibo disparaissent et sont remplacés par 3 lignes horizontales (Day High, Day Low, Day 50%)
- Day High/Low calculés sur la journée TradingView complète, mis à jour en temps réel si la session NY casse les extrêmes du Pre-NY
- Nouveau groupe d'inputs "Mode Post-NY" pour configurer ces 3 lignes

### v1.3 - 2026-04-28

- Suppression des lignes guides High/Low (redondantes avec les niveaux 0% et 100% en orange)

### v1.2 - 2026-04-21

- Dashboard : amplitude recommandée par niveau de risque (Faible P90 / Modéré P75 / Fort P50)
- Dashboard : ratio et amplitude NY estimée affichés par niveau, basés sur les stats NQ 2024–2026
- Dashboard : jour de la semaine détecté automatiquement

### v1.1 - 2026-04-09

- Midnight NY label automatically offset when it overlaps a Fibo level

### v1.0 - 2026-04-07

- Initial release

---

## Licence

[Mozilla Public License 2.0](https://mozilla.org/MPL/2.0/)

© jcornier — [GitHub](https://github.com/jcornierfra/TradingView_Indicator_JCO_NY_Amplitude_Levels)
