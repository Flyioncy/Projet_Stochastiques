# CLAUDE.local.md — notes de travail personnelles (Philippe)

Ce fichier est **chargé automatiquement** dans le contexte de Claude à chaque session,
en plus du `CLAUDE.md` partagé. C'est mon carnet de bord privé pour l'exploration sur la
branche `Philippe`. Il est commité sur ma branche (sauvegarde) mais **n'est jamais promu
sur `main`** — comme `rapport_local.tex` et `notebook_local.ipynb`.

## Convention de travail

- On explore dans `notebook_local.ipynb` et `rapport_local.tex` (suivis sur la branche, jamais sur `main`).
- Les figures d'exploration vont dans `figures_local/` (ignoré), pour ne pas écraser
  les figures « propres » de `figures/`.
- Quand un résultat est **validé**, je le reporte dans `notebook.ipynb` / `rapport.tex`
  (les livrables propres), et on ne pousse sur `main` que sur décision explicite.

## Question 1 — modélisation de $(X_t)$ (modèle `virus4.csv`)

Système :
$$dV_t = V_t\left(\tfrac{2}{3} - \tfrac{4}{3}P_t + X_t\right)dt, \qquad dP_t = P_t(-1 + V_t)\,dt.$$

Point de départ retenu : **reconstruire la trajectoire empirique de $X_t$** en inversant
l'équation des proies (même esprit que la récupération de $dt$ via l'équation des
prédateurs). Comme $d\ln V_t = (\tfrac23 - \tfrac43 P_t + X_t)\,dt$, on a
$$X_t = \frac{d\ln V_t}{dt} - \tfrac23 + \tfrac43 P_t,$$
estimable par différences finies à partir des données seules, sans hypothèse de modèle.

Pistes de modèle à départager une fois $X_t$ tracé :
- mouvement brownien avec dérive $dX = \mu\,dt + \sigma\,dB$ (X qui erre, variance ∝ t) ;
- Ornstein-Uhlenbeck $dX = -\theta(X-\mu)\,dt + \sigma\,dB$ (stationnaire, retour à la moyenne).

Intuition physique : un $X_t$ de moyenne négative abaisse le point d'équilibre des
prédateurs ($P^\* = \tfrac12 + \tfrac34\,\mathbb{E}[X]$), ce qui expliquerait leur
effondrement observé. À vérifier numériquement.