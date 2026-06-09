# CLAUDE.local.md — notes de travail personnelles (Philippe)

Ce fichier est **chargé automatiquement** dans le contexte de Claude à chaque session,
en plus du `CLAUDE.md` partagé. C'est mon carnet de bord privé pour l'exploration sur la
branche `Philippe`. Il est commité sur ma branche (sauvegarde) mais **n'est jamais promu
sur `main`** — comme `rapport_local.tex` et `notebook_local.ipynb`.

## Convention de travail

- `discussion_local.md` sert de **tableau blanc** entre Philippe et moi. À chaque fois
  que je dois lui expliquer une idée ou un raisonnement, je l'écris dans ce fichier
  (proprement, avec les formules) plutôt que seulement dans le chat. Je le tiens à jour
  au fil de la réflexion, et je mets aussi ce `CLAUDE.local.md` à jour en conséquence.
- On explore dans `notebook_local.ipynb` et `rapport_local.tex` (suivis sur la branche, jamais sur `main`).
- Les figures d'exploration vont dans `figures_local/` (ignoré), pour ne pas écraser
  les figures « propres » de `figures/`.
- Quand un résultat est **validé**, je le reporte dans `notebook.ipynb` / `rapport.tex`
  (les livrables propres), et on ne pousse sur `main` que sur décision explicite.
- Méthode par défaut pour passer au numérique ou estimer depuis les données :
  **discrétisation directe** (cours 4), $dX_t \rightsquigarrow X_i - X_{i-1}$. On ne sort
  le calcul différentiel d'Itô (cours 6/7) que s'il y a un terme brownien. Méthodes
  reportées dans le `CLAUDE.md` partagé.

## Question 1 — modélisation de $(X_t)$ (modèle `virus4.csv`)

> Raisonnement détaillé et à jour dans `discussion_local.md` (Idée 1). Résumé ci-dessous.

Système :
$$dV_t = V_t\left(\tfrac{2}{3} - \tfrac{4}{3}P_t + X_t\right)dt, \qquad dP_t = P_t(-1 + V_t)\,dt.$$

Point de départ retenu : **reconstruire la trajectoire empirique de $X_t$** en inversant
l'équation des proies (même esprit que la récupération de $dt$ via l'équation des
prédateurs). L'équation de $V$ n'a pas de terme brownien, donc on discrétise directement
(cours 4) sans passer par Itô :
$$V_i - V_{i-1} \approx V_{i-1}\left(\tfrac23 - \tfrac43 P_{i-1} + X_{i-1}\right)dt
\;\Longrightarrow\;
X_{i-1} \approx \frac{V_i - V_{i-1}}{V_{i-1}\,dt} - \tfrac23 + \tfrac43 P_{i-1}.$$
Estimable à partir des données seules, sans hypothèse de modèle.

Pistes de modèle à départager une fois $X_t$ tracé :
- mouvement brownien avec dérive $dX = \mu\,dt + \sigma\,dB$ (X qui erre, variance ∝ t) ;
- Ornstein-Uhlenbeck $dX = -\theta(X-\mu)\,dt + \sigma\,dB$ (stationnaire, retour à la moyenne).

Intuition physique : un $X_t$ de moyenne négative abaisse le point d'équilibre des
prédateurs ($P^\* = \tfrac12 + \tfrac34\,\mathbb{E}[X]$), ce qui expliquerait leur
effondrement observé. À vérifier numériquement.