# JCO NY Amplitude Levels

![JCO NY Amplitude Levels](screenshot.png)

Indicateur TradingView Pine Script v6 pour le scalp contrarien sur NQ Futures (Nasdaq 100).

Il trace les niveaux de Fibonacci basés sur le range Pre-NY (ouverture CME Globex → ouverture NY AM), avec dashboard d'amplitude intégré incluant une recommandation d'amplitude par niveau de risque basée sur les statistiques NQ 2024–2026.

---

## Principe

Chaque nuit, le NQ trace un range entre l'ouverture de la session CME Globex (22h UTC) et l'ouverture de la session NY AM (15h30 Paris par défaut). Ce range Pre-NY constitue la base de tous les niveaux Fibonacci.

- **Avant 15h30 Paris** : les niveaux sont **dynamiques** — ils suivent le High/Low en temps réel
- **A partir de 15h30** : les niveaux sont **figés** — snapshot complet du range Pre-NY
- **Visible à partir de 7h Paris** pour préparer la session

L'ordre chronologique des extrêmes (High avant Low ou Low avant High) détermine la direction et l'orientation du Fibonacci.

---

## Niveaux tracés

### Retracements (contre-tendance)

| Niveau | Couleur par défaut | Description                                    |
|--------|--------------------|------------------------------------------------|
| 0%     | Orange             | Extrémité récente (départ des retracements)    |
| 38.2%  | Rouge              | Premier retracement                            |
| 50.0%  | Vert foncé         | Retracement médian                             |
| 61.8%  | Bleu               | OTE+ (Optimal Trade Entry)                     |
| 78.6%  | Violet             | OTE-                                           |
| 100%   | Orange             | Extrémité ancienne (retracement complet)       |

### Extensions (continuation de tendance)

| Niveau  | Couleur par défaut | Style par défaut |
|---------|--------------------|------------------|
| 127.2%  | Gris               | Tirets           |
| 161.8%  | Gris               | Tirets           |
| 200.0%  | Gris               | Pointillés       |

---

## Midnight NY Open

Ligne horizontale au prix d'ouverture de la barre 0h00 NY (configurable). Utile comme référence de gap overnight.

- Timezone configurable (New York, Paris, Londres, Tokyo, etc.)
- Heure et minute configurables
- Style, couleur et épaisseur configurables

---

## Dashboard

Affiché en bas à droite du graphique :

| Ligne            | Colonnes                                          | Description                                        |
|------------------|---------------------------------------------------|----------------------------------------------------|
| Direction Pre-NY | UP / DOWN — Jour — Dynamique/Figé                 | Direction, jour de la semaine, état                |
| Amplitude Pre-NY | ex. 121 pts                                       | Range total High–Low                               |
| Faible P90       | 10% dép. — Ratio — Amp. NY — **Amp. Reco**        | 10% des jours dépassent cette amplitude            |
| Modéré P75       | 25% dép. — Ratio — Amp. NY — **Amp. Reco**        | 25% des jours dépassent cette amplitude            |
| Fort P50         | 50% dép. — Ratio — Amp. NY — **Amp. Reco**        | 50% des jours dépassent cette amplitude            |

Les ratios et l'amplitude recommandée sont calculés par jour de la semaine à partir des percentiles P90/P75/P50 issus des statistiques NQ (2 jan. 2024 – 2 avr. 2026, 581 jours).

**Formule amplitude recommandée** : `round(0.1965 × ratio × Amplitude Pre-NY)` — 0.1965 étant la moyenne des 4 distances inter-niveaux Fibo (0→38.2→50→61.8→78.6%).

---

## Paramètres

### Session NY AM

- **Heure debut NY AM (Paris)** : heure d'ouverture NY en heure de Paris (défaut : 15)
- **Minute debut NY AM** : minute (défaut : 30)

### Paramètres Dashboard

- **Afficher le dashboard** : afficher/masquer le dashboard

### Retracements

Chaque niveau (0%, 100%, 38.2%, 50%, 61.8%, 78.6%) est configurable indépendamment :

- Afficher/masquer
- Couleur
- Épaisseur (1–4)
- Style (solid / dashed / dotted)

### Extensions

Chaque niveau (127.2%, 161.8%, 200%) est configurable indépendamment.

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
