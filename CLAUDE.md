# Psynapse — mémo pour Claude

Ce fichier reprend le brief de passation de la conversation d'origine (état v88), vérifié contre le code le 9 octobre 2026. Les points encore non vérifiés sont marqués « (à vérifier) ». En cas de doute, c'est le code qui fait foi.

## 0. Travailler avec le propriétaire
- Le propriétaire travaille **uniquement depuis son iPhone**. Il est francophone et n'est pas développeur. Réponds en français, simplement, et dis-lui précisément quoi faire après chaque livraison.
- Il surveille sa consommation : fais des modifications ciblées, pas des refontes. Tout ajout doit respecter l'identité visuelle existante.
- Après une mise à jour, il faut fermer puis rouvrir l'app **deux fois**, parce que le service worker sert d'abord l'ancienne version.
- Des fonctions privées existent (§2.12) : **ne pas les supprimer, ne pas les modifier, ne jamais les décrire ni les mentionner**, y compris dans ce fichier, les commits ou les PR.

## 1. Concept
- Psynapse (anciennement Psycho-Quiz) est une PWA de révision de psychologie en français, du L1 au M2 (`YS = ["","L1","L2","L3","M1","M2"]`).
- Contenu : flashcards à répétition espacée, QCM exigeants, mini-jeu chronométré Cortex, expériences de psychologie cognitive, et une couche de jeu (XP, niveaux, succès, série de jours).
- Ton : tutoiement, chaleureux, encourageant, phrases courtes. Ni infantilisant ni froid. Pas d'anglicismes inutiles. On dit seulement Bonjour ou Bonsoir (pas « bon après-midi »).
- Gratuit, sans compte et sans serveur. Dons via Ko-fi.

## 2. Fonctionnalités

### 2.1 Navigation
- Les onglets sont `play` (Révision), `pp` (Cortex), `qcm`, `lab` et `prog` (Progrès).
- `setTab(t)` change `TAB` puis appelle `render()`. La transition directionnelle passe par la classe `dx` et la variable `--sdx`.
- L'en-tête est généré par `topHTML`.

### 2.2 Révision
- On retourne la carte, puis on glisse à droite si on savait, à gauche sinon (boutons `#yes` et `#no`).
- Bonne réponse : `r.iv` vaut 1 la première fois, puis ×2.5 à chaque fois, plafonné à 120 jours. `r.d` est la prochaine date. Si la carte a été ratée dans la séance, l'intervalle repart à 1 jour.
- Mauvaise réponse : la carte est réinsérée environ en 4e position de `Q`, avec `iv=0` et `d=aujourd'hui`.
- `st(i)` renvoie l'état de la carte : `new`, `mas` (`iv>=20`), `ko` (`k>0`) ou `run`. `due(i)` vaut vrai si `P[i].d <= day()`.
- La carte du jour affiche un anneau « x/20 » et la série de jours (`M.streak`, `M.smax`).
- XP : **+5** par bonne réponse, **+2** par mauvaise (vérifié).
- 9 catégories (`CATS`) plus « Tout mélanger ». Le filtre par années passe par `M.ys` et `on(i)`.
- « Réviser un thème » (`thm()`) propose deux modes : S'entraîner, ou Lire en fiche (`fiche()`). Il y a des synthèses `FSY` pour chaque sous-thème non-auteur (62 en v92), et les résumés d'auteurs sont dans `AUTH`.

### 2.3 QCM
- `qzGo()` construit la série : nombre de questions `QN` (20 par défaut) et catégories `QS`.
- Les distracteurs viennent de `TR`, avec un repli automatique `qOpts0` (similarité Jaccard).
- Examen blanc (`QX`) : 45 s par question. Sauvegarde et reprise avec `qzSave` et `qzResume`. Une explication `EXP` s'affiche après chaque correction. On peut aussi lancer un QCM depuis les erreurs (`QERR`).
- XP (vérifié) : +5 par bonne réponse, plus le nombre de questions en fin de série.

### 2.4 Cortex (`PP`, `PQ`)
- Il faut répondre en un mot à une question par catégorie, pour remplir un camembert 3D (`pie3()`).
- Le format de `PQ` est `[cat, année(1-5), question, [réponses acceptées]]`. Il y a **693 entrées** (v92). Répartition par année : 138, 215, 152, 111, 77 ; de 74 à 89 questions par catégorie.
- La correction est tolérante (`pOk`) : accents ignorés, distance de Levenshtein, préfixe.
- Une erreur coûte +10 s. Chaque réponse met le chrono en pause jusqu'au bouton « Continuer ». Bonne réponse : +10 XP. Fin de partie : +50 XP et confettis.
- Records : `ppHist` et `M.pr`. Sauvegarde et reprise : `ppSave`, `ppResume`, `M.ppg`.
- Sur écran tactile (`VKB`), un clavier maison remplace le clavier iOS (`vkBoard`, `vkKeys`, `vkP`). `ppFit()` dimensionne le camembert, qui est masqué si la place manque (iPhone SE). Le fond `synLoop` est mis en pause quand le clavier est affiché.

### 2.5 Labo (`LABS`, résultats dans `M.lab`)
- 10 expériences : `stroop`, `corsi`, `drm`, `cb` (cécité au changement), `pd` (dilemme du prisonnier), `gn` (Go/No-Go), `hk` (Hick), `rm` (rotation mentale), `sp` (empan de chiffres), `ig` (Iowa Gambling).
- Chaque expérience terminée donne +10 XP, via `labEnd()` (vérifié).

### 2.6 XP, niveaux, succès
- 20 niveaux (`LVN`). `lvXP(l)` = somme de (100 + 50·(k−1)) pour k < l. `lvUp()` affiche la montée de niveau.
- 19 succès (`ACH`, vérifié), évalués par `achStats()`, avec la pastille `achToast`.
- Le succès « Correcteur » a été retiré volontairement. Les textes disent « des succès », jamais un nombre fixe.

### 2.7 Progrès (`progHTML()`)
- Vue d'ensemble, barre de niveau cliquable, maîtrise par catégorie et « Mes cartes ».
- Le graphique « cette semaine » a été supprimé volontairement.

### 2.8 Mes erreurs (`errPg()`)
- Les erreurs viennent de `M.er[i]` (cartes et QCM) et de `M.ep` (Cortex).
- Une erreur disparaît dès qu'on répond juste (`errMark`, `errPQ`, compteur `M.erf`).

### 2.9 Cartes perso (`ccS()`, `M.cc`)
- Leurs indices sont ≥ 10000.
- Partage par lien `URL0 + "#cartes=" + code` (préfixe `PQ1:`). Import via `ccImp`.

### 2.10 Histoire de la psychologie (`hist()`)
- Frise de 122 auteurs (`AUTH`) et 40 jalons (`EVT`), avec 12 courants (`HC`), filtres et recherche. `histCards(n)` ouvre une fiche, `hist(2)` revient à la frise.
- Les dates ont été écrites de mémoire. Seuls Kamin et Latané ont été vérifiés. Dates discutables : Seligman 1967, Lewin 1939, Erickson 1957, Ferenczi 1913.

### 2.11 Profils, transfert, réglages
- `PR={cur,list}` : prénoms de 16 caractères maximum.
- Premier lancement : prénom, genre (`SX`), année (`M.yr`), acceptation des CGU (`M.cgu`).
- Transfert : `xfer()`, avec un code `PQP1:` + base64 (`xCode`, `xData`). Un rappel de sauvegarde (`bkCard()`) apparaît après 30 XP si la dernière sauvegarde a plus de 7 jours.
- Ko-fi : `DON` = https://ko-fi.com/psynapsegame. Contact : `MAIL` = psynapse.help@gmail.com. `report(i)` prépare un signalement.
- Tutoriel de 8 pages (`TUT`, `tuto()`), affiché tant que `M.tu < 3`.
- Réglages (`opts()`) : `M.ex` (date du partiel), `M.snd`, `M.rm`, années, transfert, CGU.
- Sons via `snd`. Haptique via `tick()` et `hap()` (switch iOS 18+, jamais testé sur un vrai iPhone).

### 2.12 Fonctions privées — NE PAS TOUCHER
- Les fonctions `ql…` (`qlOk`, `qlCard`, `qlFlip`, `qlBurst`, `qlDay`, `qlDec`, `qlHash`, `qlKey`) et leurs appels dans `thmS` et `exSet`. Leurs textes et leurs déclencheurs sont volontairement encodés. Ne rien changer, ne jamais les décoder ni les afficher en clair, et ne rien documenter de plus à leur sujet.

### 2.13 Ajouts v90
- **Confusions fréquentes** : `CF` (paires `{c, a, b, da, db, t, q:[[énoncé, 0|1]×4]}`), écrans `cfList()`, `cfShow(k)`, quiz `cfGo(k)`/`cfQ`/`cfA`/`cfN`. Meilleur score par paire dans `M.cf[k]`, +2 XP par bonne réponse. Ajouter une paire = l'ajouter à la fin de `CF`.
- **Labo → cartes** : `LBL` associe chaque expérience à des sous-thèmes et à une regex ; `lbRel(id)` donne les cartes, `lbRev(id)` lance la révision. Bouton `.lbrv` en fin d'expérience.
- **Mise en page compacte** : `.grid.cpg` (tuiles de l'accueil Révision) et `.qcg2.cpq` (pastilles du QCM), avec une variante ≤ 360 px.
- **Progrès** : le calendrier d'activité et la courbe d'XP ne s'affichent qu'avec des données. **Cortex** : les 3 tuiles de règles disparaissent après la première partie.
- **Animations** : la barre d'XP de l'en-tête était déjà animée par le wrapper `_rnd2` de `render`. Ajout : `cvBounce()` fait rebondir le compteur de séance (`#xp`) à chaque bonne réponse (`.cvb`, coupé par reduced-motion et `M.rm`).

### 2.14 Ajouts v91
- **Couverture** : 77 cartes sur des notions qui manquaient totalement (repérées en confrontant la banque à environ 670 notions classiques), plus 40 questions Cortex. Avant d'ajouter une carte, vérifier l'absence de doublon avec une recherche sur `nrm(sous-thème + question + réponse)`.
- **Labo** : 32 listes DRM (`DRL`) et 46 mots distracteurs (`DRP`), 7 règles Go/No-Go (`GNR`), 10 formes pour la cécité au changement (`CBS`, indices calculés avec `CBS.length`), 14 caractères pour la rotation mentale (`RML`, sans axe de symétrie miroir), 10 stratégies au dilemme du prisonnier (`PDS`). `lbPick(clé, n)` évite de retomber sur une liste, une règle ou un adversaire récent (historique dans `M.lr`).

### 2.15 Ajouts v92
- **Cas cliniques** : `VG` (`{c, y, t, v, q, o:[4 options, la bonne en premier], e}`), écrans `vgList()`, `vgShow(k)`, `vgA(j)`, `vgRnd()`. Les options sont mélangées à l'affichage. Filtre par années via `M.ys`. Cas réussis dans `M.vg[k]`, +5 XP la première fois.
- **Atlas du cerveau** : `ATR` (13 zones : 6 lobes et structures en mode 0, 7 aires corticales en mode 1), `atSvg(mode, surbrillance, états)`, quiz `atGo`/`atQ`/`atA` (toucher la zone) / `atF` (retrouver la fonction). Record dans `M.atl`, +2 XP par bonne réponse. Les lobes sont découpés par `clip-path` sur le contour `ATO`.
- **Confusions** : 24 paires (6 ajoutées).
- **Synthèses** : `FSY` couvre maintenant tous les sous-thèmes non-auteurs (62).
- **Clavier Cortex** : plus de `backdrop-filter` sur `.vkb` (recalcul du flou coûteux sur iOS à chaque touche), calque isolé, pas de transition sur les touches, bulle positionnée en `transform` et géométrie lue avant les écritures (`vkGeo`).
- **Labo** : couleurs alignées sur la palette des catégories (Stroop `SCA`, cécité `CBC`, Go/No-Go `GNR`) ; boutons du dilemme en style « verre » vert et rouge.
- **Frise** : dates relues ; Ekman (mort en 2025) et Rosenthal (mort en 2024) corrigés.

## 3. Architecture

### 3.1 Fichiers
- Le dépôt est servi par GitHub Pages. Il contient :
  - `index.html` : tout le HTML, le CSS et le JS, environ 1,56 Mo et 2 982 lignes, dont des lignes de données énormes ;
  - `sw.js` ;
  - `manifest.webmanifest` ;
  - les icônes et 10 écrans de démarrage `splash-*.png`.
- En pratique, seuls `index.html` et `sw.js` changent.
- **Ne jamais lire ni réécrire `index.html` en entier.** Utiliser Grep et des remplacements exacts, par exemple un script qui vérifie avec `assert` le nombre d'occurrences.

### 3.2 Globales clés
- `A` = `#app` et `B` = `document.body` (vérifié).
- `D` = cartes. `BASE` = 1 554 (v91). `TR` = mauvaises réponses et `EXP` = explications ; les deux ont une entrée par carte, et sont **clés sur le texte exact de la question**.
- `PQ` = banque Cortex, `PCS` = ordre des catégories, `PP` = partie Cortex en cours.
- `CATS` et `cat(id)` = catégories. `PC` = pseudo-catégorie des cartes perso.
- `P` = progression, `M` = méta, `PR` = profils, `K` et `MK` = clés localStorage.
- `Q` = file de révision, `ft` = réponses de la séance, `QZ`, `QS`, `QN`, `QX`, `QERR` = QCM.
- Fonctions utiles : `render`, `setTab`, `playHTML`, `qcmHTML`, `ppHTML`, `labHTML`, `progHTML`, `start`, `ids(f)`, `on(i)`, `st(i)`, `due(i)`, `day()`, `save()`, `loadProf(n)`, `xpAdd(n)`, `lvl()`, `lvUp()`, `achChk()`, `sheet(html)`, `pop(msg)`, `ic(name)`, `esc()`, `nrm()`.
- `fx(q, nq, na, nf)` corrige une carte. Elle est définie localement dans deux IIFE après les données (portée locale voulue, ce n'est pas un doublon à fusionner).

### 3.3 Banque de questions
- `var D=[...]` est suivi de nombreux blocs `D=D.concat([...])`.
- Carte : `[catId, "Sous-thème", "Question", "Réponse", année]`.
  - `catId` ∈ cog, dev, soc, neu, npsy, cli, app, met, aut.
  - Année de 1 à 5 ; si elle est absente, la carte compte comme L2.
- Répartition (v91) : L1 292, L2 546, L3 289, M1 239, M2 188. Il y a 184 sous-thèmes, dont 122 auteurs pour `aut` (v91 a créé « Émotion » en cog, « Psychologie légale » et « Psychologie environnementale » en app). Le dernier bloc `D=D.concat` (v91, notions absentes) est juste avant `var BASE=D.length` ; le dernier bloc `PQ=PQ.concat` doit rester le dernier.

### 3.4 Stockage (localStorage)
- `psy-prof` : `{cur, list}`.
- `psy-v2:<prénom>` : `P = {i: {s, o, k, l, iv, d}}`.
- `psy-meta:<prénom>` : `M`, avec notamment xp, streak, smax, last, h, ys, yr, cgu, tu, ex, snd, rm, ach, er, ep, erf, cc, lab, pr, pg, ppg, qzg, qh, bkd.
- `psy-a2hs`.
- La migration des anciennes clés sans prénom se fait dans `loadProf`.
- L'ancien défi du jour (`dfCard`, `dfGo`…, `DF`, `rng`) a été supprimé en v90. La clé `M.dd` peut encore exister chez des utilisateurs : sans effet.

## 4. Conventions

### 4.1 Style visuel
- Fond encre sombre, accents or (`#F2C14E`), panneaux « verre », icônes ligne.
- Valeurs effectives : `--bg #130F25`, `--panel #1D1838`, `--panel2 #262043`, `--ink #F4F2FF`, `--mut #A9A3CC`, `--acc #7B5CFF`. Le premier `:root` a d'autres valeurs, mais elles sont écrasées plus loin.
- Polices : Plus Jakarta Sans pour l'interface, Fraunces pour les titres et les questions.
- Couleurs des catégories : toujours passer par `cat(id)`, et elles doivent rester distinctes. Cognition #7E86FF, Dév #FF8A3D, Sociale #FF63B0, Neuro #3DD97A, Neuropsy #25E0D0, Clinique #4FA8FF, Appliquée #B5E04A, Méthodo #C77BFF, Auteurs #FF5468.
- **Pas de mode clair** : refusé par le propriétaire. Respecter `prefers-reduced-motion` et `M.rm`.
- Cartes de menu (`.qa2 button`) : toutes identiques. Ordre : Réviser un thème, Confusions fréquentes, Cas cliniques, Atlas du cerveau, Histoire, Mes cartes, Mes erreurs.
- Fenêtres : `sheet()` ou `.sh`. Messages courts : `pop()`.
- Le CSS est en couches : plus loin dans le fichier = prioritaire, avec beaucoup de `!important`.
  - Insérer le nouveau CSS juste avant `button:focus-visible{`, et le nouveau JS juste avant `function lvTier(k){` (une seule occurrence de chacun, vérifié).
  - Préfixer les nouvelles classes (par exemple `mm…`, `tl…`) : beaucoup de classes courtes existent déjà.

### 4.2 Écrire des cartes
- Toujours renseigner la catégorie, le sous-thème et l'année. **Ajouter uniquement à la fin de `D`.**
- Ajouter 3 mauvaises réponses dans `TR`, **vraiment difficiles et plausibles**. Règle permanente : la bonne réponse ne doit pas dépasser environ 1,25 fois la longueur de la plus longue mauvaise réponse.
- Ajouter une explication courte dans `EXP`.
- Cortex : la réponse tient en un mot, avec ses variantes acceptées.

### 4.3 Cache — à chaque modification de `index.html`
- Incrémenter `const C='psynapse-vNN'` dans `sw.js` (actuellement v92).
- La stratégie est « réseau d'abord ». GoatCounter est ignoré. Les chemins doivent rester **relatifs**.

## 5. Hébergement
- L'URL est https://psynapsegame.github.io/Psynapse/ (= `URL0`). Le dépôt est `psynapsegame/Psynapse`, branche `main`.
- Changer de domaine fait perdre la progression locale : il faut alors passer par le code de transfert.
- GoatCounter (`psynapse.goatcounter.com`) compte les visites et des événements `qcm-…` et `revision-…`.

## 6. Décisions déjà prises
- Pas d'app native pour l'instant. La publication sur les stores est à rediscuter.
- Pas de compte ni de serveur, d'où le code de transfert. Ko-fi plutôt que de la publicité.
- Refusé ou retiré : mode clair, plan de révision automatique, défi du jour, graphique hebdomadaire, succès « Correcteur », « codes de triche ».
- Le clavier maison de Cortex existe parce que le clavier iOS redimensionnait la page.
- Une explication est obligatoire pour chaque carte.

## 7. Points fragiles
- **La progression est indexée par position** (`P[i]`, `M.er`, `M.ep`, index de `PQ`). Ne jamais insérer, supprimer ni réordonner : ajouter uniquement à la fin.
- Pour modifier le texte d'une question, passer par `fx(...)`, pour garder `TR` et `EXP` liés.
- Safari iOS :
  - éviter `preserve-3d` et `backface-visibility` ;
  - `navigator.vibrate` est absent ;
  - `:has()` demande iOS 15.4 ou plus ;
  - clavier Cortex et haptique jamais testés sur un vrai iPhone.
- Tests : Chromium avec Playwright, aux tailles iPhone SE (320×568), iPhone 13 (390×664), iPad et PC.
  - Vérifier qu'il n'y a aucune erreur JS ni débordement horizontal.
  - Parcours : profil → tuto → révision → thème → carte perso → QCM → Cortex → Labo → Progrès → réglages.
- Les contenus de `FSY`, `AUTH` et `EVT` ont été rédigés par un modèle et seulement en partie vérifiés.

## 8. Idées prévues (par priorité)
1. Audit d'accessibilité : aria-label, contrastes, focus, cibles de 44 px, reduced-motion.
2. ~~Confusions fréquentes~~ : fait en v90 (18 paires). On peut en ajouter dans `CF`.
3. Relecture de `FSY` et des dates de `AUTH` et `EVT`.
4. Enrichir les cartes et Cortex, toujours avec `TR` et `EXP` (v90 : +19 cartes de neuropsychologie L1-L2, +20 questions Cortex M1-M2).
5. Validation sur un vrai iPhone : clavier Cortex, haptique, frise, PWA.
6. Cortex sur très petit écran : afficher un camembert réduit ?
7. Faire vivre la section Histoire au-delà de la frise.
8. Publication sur les stores : à rediscuter.
9. Découper le fichier unique : seulement si le propriétaire le demande.
