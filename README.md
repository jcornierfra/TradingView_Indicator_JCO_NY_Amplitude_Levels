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

## Marques d'amplitude (greffon)

Greffon intégré (port de l'indicateur **JCO Amplitude v2.2.0**) qui pose automatiquement des marques sur le graphique quand le prix parcourt une amplitude définie. Utile pour identifier visuellement des points contrariens (sortie d'un mouvement) ou des continuations.

![Logique amp1/amp2 continuation](Amplitude_1_2_continuation.png)

### Principe du greffon

Mécanique à deux phases avec **dissociation Long / Short** des amplitudes depuis v1.19 (4 valeurs au lieu de 2) :

- `A1L` / `A1S` — amplitude 1 (reversal) pour Long (marque verte) / Short (marque rouge)
- `A2L` / `A2S` — amplitude 2 (continuation) pour Long / Short

**Phase 1 — Searching (1ère marque du jour)**
Détection du dernier swing valide (selon `Swing left bars` / `Swing right bars`). Les deux directions sont armées en parallèle :

- `bullTrigger = swingLow + A1S` (signal vente / marque rouge → futur trade short)
- `bearTrigger = swingHigh - A1L` (signal achat / marque verte → futur trade long)

Le premier des deux atteint pose une marque et passe en phase 2.

**Phase 2 — Tracking (cycle après la 1ère marque)**
Deux règles évaluées par priorité :

1. **Reversal** (seuil = `A1` de la **nouvelle direction**) : si le prix retrace d'au moins `A1L` (resp. `A1S`) depuis l'extrême atteint, marque posée de couleur opposée, direction inversée.
2. **Continuation** (path-driven, `A1` ou `A2` de la **direction en cours**) : si le prix repart dans le sens de la tendance, marque posée à l'extrême opposé + `A_path`.

Le path (A1 vs A2) est ré-évalué uniquement quand l'extrême opposé fait un nouveau plus bas (ou plus haut). Si le retracement dépasse 30% du mouvement total depuis l'origine → `A1` (pleine amplitude), sinon `A2` (continuation réduite).

### Code couleur (lecture contrarienne)

| Mouvement  | Couleur | Signal       |
|------------|---------|--------------|
| Bas → Haut | Rouge   | Signal vente |
| Haut → Bas | Vert    | Signal achat |

### Spécificités techniques

- **Force la timeframe M1** quelle que soit la TF du chart, via `request.security_lower_tf()`. Permet d'utiliser un chart M5, M15, H1 pour le contexte tout en gardant la finesse de déclenchement M1.
- **Fenêtre d'affichage** configurable, avec **timezone configurable** (mêmes options que la ligne Midnight NY ; défaut : 7h–22h Paris).
- **Reset journalier** à minuit dans la timezone choisie.
- Toggle `Afficher les marques` pour activer/désactiver le greffon (case à cocher en tête du groupe).

### Auto NY (v1.11, dissociation L/S v1.19)

Case à cocher **Auto NY** (activée par défaut depuis v1.12) : les 4 amplitudes du greffon (A1L / A1S / A2L / A2S) sont remplacées automatiquement par les **Amp. Reco** calculées depuis l'historique NY récent. Depuis v1.19, **4 coefficients distincts** sont appliqués directement à la moyenne lissée :

- `A1L = round(0.390 × avg_reco)` — reversal long
- `A1S = round(0.330 × avg_reco)` — reversal short
- `A2L = round(0.420 × avg_reco)` — continuation long
- `A2S = round(0.255 × avg_reco)` — continuation short

Les 4 coefficients sont configurables dans le groupe **"Calcul Amp. Reco"**. Pratique pour adapter automatiquement le greffon à la volatilité récente du marché sans ressaisir les paramètres chaque jour.

### Ajustement pré-NY AM (v1.11)

Case à cocher **Ajuster les amplitudes avant NY AM** (activée par défaut depuis v1.12) + coefficient configurable (défaut **0.5**). Avant l'heure NY AM définie dans le groupe « Session NY AM » (Paris), les **4 amplitudes L/S** (qu'elles soient saisies manuellement ou issues de Auto NY) sont multipliées par ce coef. À 15h30 Paris, on revient automatiquement aux valeurs pleines.

Exemple : A1L = 120, A1S = 100, A2L = 127, A2S = 77, coef = 0.5. Avant 15h30 Paris → A1L = 60, A1S = 50, A2L = 63.5, A2S = 38.5. À partir de 15h30 → valeurs pleines restaurées. Permet de coller à la volatilité réduite des séances asiatique et européenne tout en gardant la pleine amplitude pendant la session NY.

### Restart à chaque session (v1.12)

Case à cocher **Restart recherche Amp1 à chaque session** (décochée par défaut). Si activée, l'état tracking est effacé au **début de la fenêtre** (par défaut 7h Paris) et à l'**open NY AM** (15h30 Paris) — le greffon repasse en phase `searching` pour poser une nouvelle marque Amp1.

Les **swings high/low** détectés juste avant l'ouverture sont **conservés** comme référence. Un mouvement amorcé juste avant l'open continue donc de compter dans le calcul Amp1.

Exemple : à 15h29 une bougie haussière de 30 pts forme un swing low à X. À 15h30, l'open NY AM réinitialise la phase à `searching` (swing low toujours = X). Si la bougie d'open monte de 80 pts (high atteint X+110), le trigger `swing + A1S (100)` est atteint → marque rouge posée.

Sans cette option (défaut), la state machine tourne en continu : si elle était déjà en phase tracking avant l'ouverture, elle reste en tracking et pose des marques de continuation/reversal selon la logique habituelle.

### Paramètres principaux

- **A1L** (défaut : **120** pts) — Amp1 Long, reversal pour marque verte (futur trade long)
- **A1S** (défaut : **100** pts) — Amp1 Short, reversal pour marque rouge (futur trade short)
- **A2L** (défaut : **127** pts) — Amp2 Long, continuation pour marque verte
- **A2S** (défaut : **77** pts) — Amp2 Short, continuation pour marque rouge
- **Auto NY** (défaut : **coché** depuis v1.12) — surcharge les 4 amplitudes L/S avec les Amp. Reco calculées par le dashboard
- **Ajuster les amplitudes avant NY AM** + **Coef pre-NY AM** (défauts : coché / 0.5)
- **Prix par point** (défaut : 1.0 pour NQ/MNQ/ES)
- **Méthode détection swing** (défaut : `Durée` depuis v1.15) — choix entre :
  - `Bougies` : pivot défini par **Swing left/right bars** (défaut 3/3, pivot strict sur 7 bougies M1)
  - `Durée` : pivot défini par **Durée avant/après** en minutes (défaut 15/15, **Durée après peut valoir 0** depuis v1.16 pour détection temps réel) — indépendant de la TF du chart, utile sur les charts sub-minute
- **Fenêtre horaire + Timezone** (défaut : 7h–22h Paris)
- **Largeur marque** (défaut : 2 bars), **épaisseur trait**, **couleurs bull/bear** — personnalisation visuelle

---

## Dashboard

Affiché en bas à droite du graphique, **3 colonnes × 9 lignes** (refonte v1.19) :

| Ligne | Colonne 0 (label)    | Colonne 1 (Long / vert)        | Colonne 2 (Short / rouge)     |
|-------|----------------------|--------------------------------|-------------------------------|
| 0     | UP ▲ / DOWN ▼        | —                              | Figé / Dynamique              |
| 1     | « Amp. Pre-NY »      | Valeur Pre-NY (ex. 121 pts)    | —                             |
| 2     | « Amp. Moy. Nd »     | Valeur moy. Nd (ex. 215 pts)   | —                             |
| 3     | « Amp. NY »          | Amp. NY mini (P50, vert)       | Amp. NY maxi (P90, rouge)     |
| 4     | « Amp. Reco »        | « Long » (header vert)         | « Short » (header rouge)      |
| 5     | « A1 »               | A1L (vert)                     | A1S (rouge)                   |
| 6     | « A2 »               | A2L (vert)                     | A2S (rouge)                   |
| 7     | —                    | « No-Go » (header gris)        | « Solder » (header gris)      |
| 8     | « Filtre efficience »| Valeur No-Go (gris/rouge)      | Valeur Solder (gris/rouge)    |

- **Lignes 0-3** : direction Pre-NY, amplitude du jour, moyenne historique, estimations NY.
- **Lignes 4-6** : amplitudes recommandées du greffon, dissociées Long / Short (v1.19).
- **Lignes 7-8** : filtre efficience Kaufman (v1.19). Valeur **live** avant l'heure gate, **figée** ensuite. Couleur **rouge** dès que la valeur ≥ seuil configuré, **gris** sinon.

Le dashboard se masque entièrement avec le toggle `Afficher le dashboard` (groupe **Dashboard**).

---

### Méthode de calcul de l'Amp. Reco (v1.9)

L'objectif est d'obtenir une amplitude recommandée **stable et raisonnable** pour le scalping, qui ne s'envole pas brutalement après une journée explosive (les fins de journée sont rarement aussi extrêmes que les débuts).

**Étapes du calcul** :

1. **Suivi automatique des amplitudes NY récentes** : à chaque session NY (15h30 → 22h Paris), l'indicateur calcule `Amp NY = High - Low` et stocke la valeur dans un historique glissant des N derniers jours de trading (N configurable, défaut **5**).

2. **Estimation NY pour aujourd'hui** : on prend la plus grande estimation statistique, soit **Amp. NY P90** (ratio P90 du jour × Amp. Pre-NY), correspondant au risque faible.

3. **Moyenne lissée** (diviseur ajusté en v1.18) :

   ```text
   avg = (Amp. NY P90 + somme des amplitudes NY chargées) / (nb_jours_chargés + 1)
   ```

   Le diviseur utilise le nombre **réel** de jours présents dans l'historique (au plus N), pas N figé. Évite la sous-estimation en phase de warmup.

4. **Quatre Amp. Reco** (dissociation L/S v1.19) avec **4 coefficients distincts** :

   ```text
   A1L = round(0.390 × avg)   # Long Amp1 (reversal long)
   A1S = round(0.330 × avg)   # Short Amp1 (reversal short)
   A2L = round(0.420 × avg)   # Long Amp2 (continuation long)
   A2S = round(0.255 × avg)   # Short Amp2 (continuation short)
   ```

**Pourquoi cette méthode ?**

- **Lissage** : la somme des N derniers jours intègre la dynamique récente du marché et amortit les valeurs extrêmes.
- **Cohérence avec les statistiques** : l'estimation P90 (Amp. NY maxi) sert d'ancrage statistique haut.
- **Dissociation Long / Short** (v1.19) : sur NQ, le côté short est statistiquement plus volatil que le côté long — les 4 coefs permettent de coller à cette asymétrie sans ressaisir les valeurs chaque jour.

Les ratios statistiques par jour de la semaine sont issus de l'analyse historique NQ (2 jan. 2024 – 2 avr. 2026, 581 jours de trading).

---

## Paramètres

### Session NY AM

- **Heure debut NY AM (Paris)** : heure d'ouverture NY en heure de Paris (défaut : 15)
- **Minute debut NY AM** : minute (défaut : 30)

### Paramètres Dashboard

- **Afficher le dashboard** : toggle bool (défaut **coché**) — affiche / masque le dashboard.

### Calcul Amp. Reco

Configurable depuis le groupe **"Calcul Amp. Reco"** :

- **Nb jours pour la somme** (défaut **5**) : nombre N de sessions NY précédentes utilisées dans la moyenne. Le diviseur utilise la taille réelle de l'historique + 1 (les jours chargés + l'estimation P90 du jour).
- **Coef A1L (Long Amp1)** (défaut **0.390**)
- **Coef A1S (Short Amp1)** (défaut **0.330**)
- **Coef A2L (Long Amp2)** (défaut **0.420**)
- **Coef A2S (Short Amp2)** (défaut **0.255**)

Chaque coefficient est appliqué directement à la moyenne `avg_reco` pour donner la valeur correspondante en points.

### Niveaux (toggle maître)

- **Afficher toutes les lignes** : case à cocher unique qui masque/affiche d'un coup **toutes** les lignes des niveaux (les 6 retracements, les 6 extensions, et les 3 Day lines Post-NY). La ligne Midnight NY garde son toggle indépendant.

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

### Paramètres greffon Marques d'amplitude

- **Afficher les marques** : toggle global du greffon
- **Auto NY** (défaut **coché** depuis v1.12) : surcharge les 4 amplitudes L/S avec les Amp. Reco du dashboard
- **A1L** / **A1S** / **A2L** / **A2S** (défauts **120 / 100 / 127 / 77** pts depuis v1.19) — ignorés si Auto NY est coché
- **Ajuster les amplitudes avant NY AM** (défaut **coché** depuis v1.12) + **Coef pre-NY AM** (défaut 0.5)
- **Restart recherche Amp1 à chaque session** (défaut décoché) — relance une recherche fraîche au début de la fenêtre et à l'open NY AM
- **Prix par point** : 1.0 pour NQ/MNQ/ES
- **Méthode détection swing** (défaut **`Durée`** depuis v1.15) + paramètres associés :
  - Mode `Bougies` : **Swing left/right bars** (défaut **3/3** depuis v1.13)
  - Mode `Durée` : **Durée avant/après** en minutes (défaut **15/15**, nouveau v1.14 ; **Durée après ∈ [0, 120]** depuis v1.16 — 0 = pivot temps réel sur la bougie courante)
- **Fenêtre horaire** : début/fin (défaut 7h–22h) + **Timezone** (mêmes options que Midnight NY, défaut Paris)
- **Largeur marque** (défaut 2 bars), **épaisseur trait, couleurs bull/bear**

### Paramètres Efficience NY (v1.19)

Configurable depuis le groupe **"Efficience NY"** :

- **Activer le filtre d'efficience** (défaut **coché**) — active le calcul, le dashboard et les 2 alertes.
- **Seuil No-Go** (défaut **0.55**) — au-dessus de ce seuil à l'heure No-Go, le marché est jugé trop directionnel pour une entrée contrarienne.
- **Seuil Solder** (défaut **0.40**) — au-dessus de ce seuil à l'heure Solder, envisager de solder une position contrarienne ouverte.
- **Heure No-Go** / **Minute** (défaut **15h45** Paris)
- **Heure Solder** / **Minute** (défaut **16h15** Paris)

Les 2 alertes (`Efficience NO-GO` et `Efficience SOLDER`) se configurent via le menu **Add alert → Condition → JCO NY Amplitude Levels → [nom de l'alerte]** dans TradingView. Une seule émission par jour par alerte.

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

### v1.19 - 2026-06-12

- **Greffon — Dissociation Long / Short des amplitudes** : les 2 inputs `Amp1` et `Amp2` sont remplacés par **4 inputs** :
  - `A1L` (Long Amp1, défaut **120**) — seuil reversal pour les marques vertes (futur trade long)
  - `A1S` (Short Amp1, défaut **100**) — seuil reversal pour les marques rouges (futur trade short)
  - `A2L` (Long Amp2, défaut **127**) — seuil continuation pour les marques vertes
  - `A2S` (Short Amp2, défaut **77**) — seuil continuation pour les marques rouges
- **Auto NY** : les 2 coefficients passent à **4** — `reco_coef_a1l` (0.390), `reco_coef_a1s` (0.330), `reco_coef_a2l` (0.420), `reco_coef_a2s` (0.255), appliqués directement à `avg_reco` (plus de facteur 0.30 intermédiaire).
- **Convention de mapping** :
  - Direction `"up"` (mouvement bull, marque rouge / signal vente) = futur trade **SHORT** → utilise A1S / A2S
  - Direction `"down"` (mouvement bear, marque verte / signal achat) = futur trade **LONG** → utilise A1L / A2L
- Le **reversal** utilise A1 de la **nouvelle direction** (celle de la marque à venir), la **continuation** utilise A1 ou A2 de la direction en cours selon le path-driven à 30%.

- **Dashboard — Refonte 3 colonnes** : passage à **3 colonnes** (`label | Long | Short`) sur **9 lignes**. Affichage compact des 4 valeurs A1L / A1S / A2L / A2S sur 2 lignes (A1 et A2). Suppression de l'ancien mode "Complet" (seul "Simplifie" subsiste, plus la liste déroulante : un simple toggle Afficher / Masquer).

- **Filtre Efficience NY (Kaufman) + 2 alertes** : nouveau groupe d'inputs avec :
  - 2 seuils configurables (`Seuil No-Go` défaut **0.55**, `Seuil Solder` défaut **0.40**)
  - 2 horaires gate configurables (`No-Go` défaut **15h45** Paris, `Solder` défaut **16h15** Paris)
  - Toggle d'activation (`Activer le filtre d'efficience`)

  Calcul : `eff = |close - o0| / sum(|close - prevClose|)` avec `o0` = open de la 1ère M1 de la session NY (15h30 Paris). Bornée [0, 1]. Reset journalier à minuit timezone amp_tz_id.

  **Dashboard** : 2 nouvelles lignes en bas (header `No-Go / Solder`, puis valeurs). Valeur figée à l'heure gate, **rouge** dès que le seuil est franchi, gris sinon.

  **Alertes** : 2 `alertcondition` — `Efficience NO-GO` (déclenchée à 15h45 si valeur ≥ seuil No-Go) et `Efficience SOLDER` (déclenchée à 16h15 si valeur ≥ seuil Solder). Une seule émission par jour grâce au tracking `eff_prev_*_done`.

- **Efficience NY — live preview sur charts > M1** : `request.security_lower_tf` livre les M1 batches à la clôture de chaque barre du chart. Sans correctif, le dashboard attendait la fin de la 1ère barre après 15h30 (5 min sur M5, 15 min sur M15...). Deux mécanismes ajoutés :
  - **Fallback anchor** : si la barre realtime franchit 15h30 et qu'aucune M1 n'a encore ancré, on ancre avec l'open de la barre courante (identique à l'open de la M1 contenant 15h30 sur tout chart aligné).
  - **Live preview** : recalcul de `eff_s.current` en ajoutant le close courant du chart comme tick supplémentaire au-dessus du path committé. La prochaine M1 batch écrase proprement le path par-dessus.

### v1.18 - 2026-06-05

- **Dashboard / Amp. Reco — diviseur ajusté** : la moyenne lissée divisait par `N + 1` figé (paramètre `Nb jours pour la somme`), alors que la somme `sum_5d_ny` était calculée sur la taille **réelle** de l'historique. En phase de warmup ou si moins de N jours étaient disponibles, la moyenne était **sous-estimée**.
- Le diviseur utilise maintenant `array.size(ny_amp_hist) + 1` (le `+1` correspond à l'amplitude NY estimée P90 du jour, ajoutée au numérateur).
- Sur historique plein (`hist_size == N`) le résultat est **inchangé** — le fix vise la cohérence avec la version Tradovate de l'indicateur (commit `3e9f539`, v2.3.0), qui tourne souvent avec moins de jours chargés.

### v1.17 - 2026-06-05

- **Greffon — fix Phase 1 searching en tendance forte** : la variable swing unique (`swingPrice` + `swingType`) était écrasée à chaque nouveau pivot détecté. En mode `Durée` avec `Durée après = 0`, chaque nouveau plus bas / plus haut local devenait un swing. En tendance baissière forte qui ne produit que des swing low successifs, `swingType` restait figé à `"low"` et le **bear trigger n'était jamais armé**. Si un reset de session (open NY AM, début fenêtre, minuit) tombait pendant la tendance, la state machine restait coincée en searching jusqu'à la fin de la tendance.
- **Fix** : le type `AmpState` stocke maintenant `swingLow` et `swingHigh` **indépendamment**. Phase 1 searching arme les **deux directions en parallèle** :
  - `bullTrigger = swingLow + Amp1` (signal achat / marque rouge)
  - `bearTrigger = swingHigh - Amp1` (signal vente / marque verte)
- Quel que soit l'écrasement répété d'un côté, l'autre côté reste mémorisé et un trigger Amp1 peut toujours se déclencher.
- Le tracking (continuations Amp2, reversals Amp1, ré-évaluation du path à 30 %) est **strictement inchangé**.

### v1.16 - 2026-05-21

- **Greffon — `Durée après` peut valoir 0** (`minval` passe de 1 à 0). Avec 0 minute à droite, le pivot peut être la **bougie courante elle-même** : pas d'attente de confirmation, le swing est détecté en temps réel et la marque Amp1 peut se déclencher sur la même bougie. Pratique pour le scalping où la réactivité prime.
- **Trade-off** : avec 0 à droite, les swings deviennent plus agressifs (chaque nouveau plus bas / plus haut local devient un swing). Si trop de re-marquages parasites, repasser à 1-3 min.

### v1.15 - 2026-05-20

- **Greffon — fix des marques "flottantes"** : une marque n'est plus dessinée que si `triggerPrice` est dans `[low, high]` de la bougie courante. Cela résout les cas où la state machine "rattrape" un seuil déjà franchi (typiquement au démarrage de la fenêtre ou sur un gap d'ouverture). La state machine continue d'avancer normalement, seul le **dessin** est filtré. Port du correctif équivalent côté Tradovate (`maybePushMarker`).
- **Greffon — défaut "Méthode détection swing"** : passe de `Bougies` à **`Durée`**. Le critère temporel reste pertinent quelle que soit la TF du chart (M1, S30, S15, etc.).

### v1.14 - 2026-05-20

- **Greffon — détection swing en mode "Durée"** : nouveau paramètre `Méthode détection swing` avec deux modes au choix :
  - `Bougies` (défaut) : pivot défini par N bougies à gauche / droite (paramètres `Swing left/right bars`, défaut 3/3) — comportement v1.13 inchangé.
  - `Durée` : pivot défini par une **durée en minutes** à gauche / droite (paramètres `Durée avant/après`, défaut 15/15) — **indépendant de la TF du chart**.
- Le mode `Durée` est utile sur les charts **sub-minute** (S30, S15, S1) où le greffon retombe sur la TF du chart au lieu de M1. La durée garantit un critère temporel cohérent peu importe la granularité (15 min = 15 bars M1 = 30 bars S30 = 60 bars S15).
- Tous les modes de tracking (continuations Amp2, reversals Amp1, ré-évaluation du path à 30 %) restent inchangés.

### v1.13 - 2026-05-20

- **Greffon — détection de swing** : `Swing left bars` et `Swing right bars` passent de **1/1 à 3/3** par défaut. Le pivot doit désormais être le strict min/max sur **7 bougies M1** (3 à gauche + pivot + 3 à droite) au lieu de 3. Moins de pollution par les micro-swings, `swingPrice` plus stable entre les vrais mouvements.
- Impacte uniquement la **Phase 1 searching** (première marque de la journée et, si l'option *Restart* est cochée, première marque de chaque session). Le tracking (continuations Amp2, reversals Amp1) est strictement inchangé — il utilise les extrêmes dynamiques (`refExtreme`, `oppExtreme`, etc.), pas les swings.

### v1.12 - 2026-05-20

- **Greffon — fix de cohérence** : la state machine tourne désormais 24h/24. Auparavant, modifier l'heure de début de fenêtre (`Debut h`) changeait les marques affichées plus tard dans la journée, car la state machine ne démarrait qu'à l'ouverture et les phases (searching/tracking) divergeaient. Désormais, seul le **dessin** des marques est filtré par la fenêtre horaire — la logique interne est cohérente quelle que soit l'heure de début.
- **Greffon — détection swing 24h/24** : la recherche de swing low/high tournait auparavant uniquement en phase `searching`. Elle tourne désormais en continu (y compris pendant le tracking). Le dernier swing détecté est toujours disponible comme référence.
- **Greffon — conservation du swing aux resets** : `swingPrice` / `swingTime` / `swingType` ne sont plus effacés ni au reset journalier (minuit) ni au reset de session. Un mouvement amorcé juste avant une ouverture continue de compter dans le calcul Amp1.
- **Greffon — nouvelle option `Restart recherche Amp1 à chaque session`** (décoché par défaut). Si activée, l'état tracking est effacé au début de la fenêtre et à l'open NY AM → nouvelle marque Amp1 fraîche, en gardant le dernier swing comme référence.
- **Defaults greffon** : `Auto NY` et `Ajuster les amplitudes avant NY AM` sont désormais **activés par défaut**.

### v1.11 - 2026-05-20

- **Greffon** : timezone configurable pour la fenêtre horaire (liste déroulante, mêmes options que la ligne Midnight NY). Défaut : Paris.
- **Greffon** : case à cocher **Auto NY** qui surcharge `Amp1` / `Amp2` avec les **Amp. Reco** du dashboard (`Amp1` ← reco faible / vert, `Amp2` ← reco fort / rouge).
- **Greffon** : case à cocher **Ajuster les amplitudes avant NY AM** + coefficient configurable (défaut **0.5**). Avant l'heure NY AM définie en tête d'indicateur (Paris), `Amp1` et `Amp2` sont multipliés par ce coef. Permet d'adapter le greffon à la volatilité réduite des séances asiatiques et européennes.
- **Calcul Amp. Reco** : coef. risque faible passe de **0.25 à 0.30** par défaut.
- **Greffon** : valeurs par défaut affinées — **Amp1 = 100 pts** (était 80), **Début fenêtre = 7h** (était 8h), **Largeur marque = 2 bars** (était 1.4).

### v1.10 - 2026-05-26

- **Dashboard** : inversion des amplitudes NY estimées affichées.
  - En haut (vert, risque Faible) : **Amp. NY mini** (P50)
  - En bas (rouge, risque Fort) : **Amp. NY maxi** (P90)
- Logique : petite amplitude = petit risque, grande amplitude = gros risque.
- Le ratio et le % dépassement suivent le swap (50% en haut, 10% en bas).
- Le calcul interne de la moyenne est inchangé (toujours basé sur P90 pour la stabilité).

### v1.9 - 2026-05-25

- **Refonte du calcul de l'Amp. Reco** : remplacement du coefficient Fibo (0.1965) par une moyenne lissée sur la session NY récente.
- **Tracking automatique** des amplitudes NY (15h30–22h Paris) sur les N derniers jours de trading (N configurable, défaut 5).
- **Nouvelle formule unique** :

  ```text
  avg = (Amp.NY estimée P90 + somme des N amplitudes NY récentes) / (N + 1)
  Amp. Reco Faible = round(coef_faible × avg)  [défaut 0.25]
  Amp. Reco Fort   = round(coef_fort   × avg)  [défaut 0.20]
  ```

- Nouveau groupe d'inputs **"Calcul Amp. Reco"** pour configurer N et les 2 coefficients.
- Dashboard : suppression de la ligne Modéré P75 (calcul unique sur P90).
- Dashboard : ajout d'une ligne « Amp. Moy. Nd » sous « Amplitude Pre-NY » qui affiche la moyenne simple des N dernières amplitudes NY (référence informative).
- **Avantage vs ancien calcul** : valeurs moins extrêmes, plus stables jour après jour, car la moyenne intègre l'historique récent en plus de l'estimation statistique.

### v1.8 - 2026-05-17

- Ajout du greffon **Marques d'amplitude** (port de l'indicateur JCO Amplitude v2.2.0) : détection swing M1 + machine d'états deux phases qui pose des marques aux points d'amplitude `amp1` (reversal) et `amp2` (continuation). Force la timeframe M1 via `request.security_lower_tf()` quelle que soit la TF du chart.
- Nouveau toggle maître **Afficher toutes les lignes** pour masquer d'un coup toutes les lignes des niveaux (Fibo + Day lines). La ligne Midnight NY garde son toggle indépendant.

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
