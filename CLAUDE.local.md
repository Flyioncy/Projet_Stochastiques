# CLAUDE.local.md — notes de travail personnelles (Philippe)

Chargé automatiquement à chaque session, en plus du `CLAUDE.md` partagé. Carnet de bord
privé pour la branche `Philippe`, commité comme sauvegarde mais **jamais promu sur `main`**
(comme `rapport_local.tex`, `notebook_local.ipynb`, `discussion_local.md`).

## Convention de travail

- `discussion_local.md` = tableau blanc (journal des idées, raisonnements détaillés).
- On explore dans `notebook_local.ipynb` / `rapport_local.tex` ; on reporte le validé dans
  `notebook.ipynb` / `rapport.tex` et on ne pousse sur `main` que sur décision explicite.
- Estimation/numérique : discrétisation directe (cours 4) ; Itô (cours 6/7) seulement s'il
  y a un terme brownien. Conventions de rédaction et de méthode dans le `CLAUDE.md` partagé.

## Pièges techniques (pour mes prochaines sessions)

- **Ne jamais passer du LaTeX dans un heredoc Python via Bash** : `\t \b \f \n \a \v` sont
  interprétés (ex. `\theta`→TAB, `\begin`→backspace) et corrompent le fichier. Écrire le
  contenu dans un fichier temporaire avec l'outil Write (sûr), puis splicer en Python qui
  lit ce fichier. Pour un marqueur de recherche, le prendre **sans backslash**.
- L'outil `Edit` échoue parfois sur ces fichiers (vue désynchronisée) : relire juste avant,
  ou passer par Write/splice.

## Question 1 (`virus4.csv`) — TERMINÉE

Système : $dV_t = V_t(\tfrac23 - \tfrac43 P_t + X_t)dt$, $dP_t = P_t(-1+V_t)dt$.

Démarche et résultats (tout dans `rapport.tex`/`notebook.ipynb`, désormais sur `main`) :

1. **Pas de temps** récupéré via l'équation (sans bruit) des prédateurs : $dt \approx 0{,}015$, $\tau = 30$.
2. **Reconstruction déterministe** de $X_t$ en inversant l'équation des proies :
   $X_{i-1} \approx \tfrac{V_i - V_{i-1}}{V_{i-1}\,dt} - \tfrac23 + \tfrac43 P_{i-1}$. Sauvegardée dans `virus4_Xt.csv`.
   Moyenne $\approx -0{,}40$, écart-type $\approx 0{,}23$. (Argument d'équilibre : $P^*=\tfrac12+\tfrac34\,m$ avec $m=\mathbb E[X]<0$, cohérent avec l'effondrement des prédateurs.)
3. **Loi des différences** $\Delta X$ : non gaussienne (KS, p $\approx 6\cdot10^{-14}$ ; queues lourdes, kurtosis $\approx 3{,}3$).
4. **Revue des modèles du cours** : GBM/Black-Scholes (positif), pont brownien (épinglé), brownien intégral (trop lisse), Verhulst (positif) écartés a priori. Restent **brownien avec dérive** et **Ornstein-Uhlenbeck**.
5. **Tests par simulation** (KS entre $X$ simulé et reconstruit, simulation par brownien `cumsum` ; OU par solution exacte via intégrale d'Itô) :
   - dérive $\nu\approx-0{,}02$, $\sigma\approx0{,}05$ → p-value moyenne $\sim10^{-9}$.
   - OU $\theta\approx0{,}07$, $\mu\approx-0{,}66$, $\sigma\approx0{,}05$ → p-value moyenne $\sim10^{-7}$.
   - **Les deux rejetés** (p $\ll 0{,}05$). Cohérent avec les différences non gaussiennes.

**Conclusion Q1 : aucun modèle gaussien du cours ne convient. Le modèle de $X_t$ reste à trouver.**

Leçon de méthode (acquise avec Philippe, voir `CLAUDE.md`) : KS très sur-puissant sur ces
trajectoires autocorrélées (même sim-vs-sim donne p$\approx$0), mais on s'en tient à
l'interprétation simple de la p-value (cours/TP) ; pas de stat annexe ni de Monte-Carlo
sur une statistique de test.

## Prochaines étapes

- **Soit** proposer un nouveau modèle pour $X_t$ hors cadre gaussien (différences à queues lourdes).
- **Soit** attaquer les **probabilités d'extinction** (densité sous $0{,}01$) : là, le Monte-Carlo / loi des grands nombres sera dans son vrai contexte.
- Question 2 (`virus6.csv`) pas encore commencée : bruit $\sigma\,dX_t$ dans l'équation de $V$, estimer $\sigma$, proposer une EDS pour $X_t$, loi de $X_\tau$.

## État dépôt (au 10 juin 2026)

- `main` : Q1 validée (rejet des modèles gaussiens) — `rapport.tex`, `notebook.ipynb`, `CLAUDE.md`, figures `virus4_*`.
- `Philippe` : idem + fichiers `_local` d'exploration.