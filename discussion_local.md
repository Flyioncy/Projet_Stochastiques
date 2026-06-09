# Discussion — journal des idées

> Tableau blanc entre Philippe et Claude. Claude écrit ici ses idées et ses
> raisonnements au fur et à mesure ; on garde une trace lisible de la réflexion,
> séparée du code (notebook) et du rendu propre (rapport). Fichier personnel,
> suivi sur la branche `Philippe`, jamais promu sur `main`.

---

## Idée 1 — Reconstruire $X_t$ à partir des données (modèle `virus4.csv`)

### Ce qu'on attend : passer à l'espérance

Avant de reconstruire quoi que ce soit, regardons l'équilibre du système. Sans virus, les
points fixes sont

$$\dot P = 0 \Rightarrow V^* = 1, \qquad \dot V = 0 \Rightarrow P^* = \tfrac{1}{2},$$

et le système classique tourne autour de $(1, \tfrac12)$. Si l'on suppose que $X_t$ admet
une moyenne finie $\mu = \mathbb{E}[X]$ (plausible, à vérifier), on passe l'équation des
proies à sa moyenne et la nullcline se décale :

$$\tfrac{2}{3} - \tfrac{4}{3}P + \mu = 0 \;\Longrightarrow\; P^* = \tfrac{1}{2} + \tfrac{3}{4}\mu.$$

Comme les données montrent des prédateurs qui s'effondrent bien en dessous de $\tfrac12$, on
s'attend à **$\mu < 0$**. C'est cette prédiction qu'on va vérifier en reconstruisant $X_t$.

### Reconstruire $X_t$ par discrétisation

On part de l'équation des proies, qui porte le virus :

$$dV_t = V_t\left(\tfrac{2}{3} - \tfrac{4}{3}P_t + X_t\right)dt.$$

On discrétise directement entre $t_{i-1}$ et $t_i = t_{i-1}+dt$, à la manière du cours 4 :
on remplace $dV_t$ par l'incrément $V_i - V_{i-1}$ et on évalue le membre de droite au début
du pas.

$$V_i - V_{i-1} \approx V_{i-1}\left(\tfrac{2}{3} - \tfrac{4}{3}P_{i-1} + X_{i-1}\right)dt.$$

Il ne reste qu'à isoler $X_{i-1}$, et tout le membre de droite est observé :

$$\boxed{\,X_{i-1} \approx \frac{V_i - V_{i-1}}{V_{i-1}\,dt} - \tfrac{2}{3} + \tfrac{4}{3}P_{i-1}\,}$$

On reconstruit ainsi toute la trajectoire empirique de $X$, sans hypothèse de modèle. Même
esprit « à la physicienne » que pour $dt$ : on inverse le schéma discrétisé pour isoler
l'inconnue. Ici on obtient une série temporelle (une réalisation du processus) et non une
constante.

### Première manip proposée

Reconstruire $\{X_i\}$ puis l'examiner sous trois angles :

1. $X_t$ en fonction de $t$ — erre-t-il (variance qui gonfle) ou fluctue-t-il autour d'un niveau fixe ?
2. histogramme des incréments $\Delta X_i = X_{i+1}-X_i$ — gaussien i.i.d. ?
3. moyenne et autocorrélation de $X_t$.

C'est gratuit, sans hypothèse de modèle, et ça tranche entre les deux candidats naturels.

| Allure observée | Modèle suggéré |
|---|---|
| $X_t$ erre, $\mathrm{Var}\propto t$ | brownien avec dérive : $dX_t = \mu\,dt + \sigma\,dB_t$ |
| $X_t$ revient vers un niveau | Ornstein-Uhlenbeck : $dX_t = -\theta(X_t-\mu)\,dt + \sigma\,dB_t$ |

Intuition : je penche pour l'**Ornstein-Uhlenbeck**, car l'intensité d'un virus
fluctue mais reste bornée (retour à la moyenne, pas de fuite à l'infini).

### Si c'est de l'OU : estimation immédiate

En discrétisant $dX = -\theta(X-\mu)dt + \sigma\,dB$ :

$$\Delta X_i \approx -\theta\,dt\,(X_i - \mu) + \sigma\sqrt{dt}\,\varepsilon_i.$$

Une simple **régression linéaire de $\Delta X_i$ sur $X_i$** donne tout :
- pente $= -\theta\,dt$ ;
- ordonnée à l'origine $= \theta\mu\,dt$ ;
- écart-type des résidus $= \sigma\sqrt{dt}$.

Trois paramètres, une droite.

### Validation (plus tard)

Une fois le modèle de $X_t$ choisi et estimé, on simule des trajectoires
(Euler-Maruyama), on les réinjecte dans le système couplé $(V,P)$, et on vérifie qu'on
reproduit l'effondrement des prédateurs et la perte de périodicité — pas seulement la
loi de $X_t$ isolée.

### Observations (manip faite dans le notebook propre)

Reconstruction faite et sauvegardée dans `virus4_Xt.csv` (en-tête `t,X`, 1999 points).

Avec **2000 points et une seule trajectoire**, on ne conclut rien, on intuite des tendances.

- **Niveau** : moyenne$(X) \approx -0{,}40$, écart-type $\approx 0{,}23$. Négatif, va dans le
  sens de $\mu < 0$ (prudent, la moyenne mélange la descente et le plateau).
- **$X_t$ vs $t$** : semblerait partir de $\approx 0$, descendre vers $-0{,}6$ vers
  $t\approx 13$, puis fluctuer. Pas stationnaire sur la fenêtre.
- **Incréments $\Delta X$** : moyenne $\approx -3\cdot10^{-4}$ (dérive par pas quasi nulle),
  écart-type $\approx 6{,}4\cdot10^{-3}$.

**Correction de ma sur-interprétation précédente.** L'« autocorrélation » que j'avais tracée
était l'ACF du *niveau* $X_t$, qui n'est pas stationnaire : sa décroissance lente reflète
surtout la tendance, pas une vraie mémoire. À jeter. Plus honnête : regarder les incréments.

- corr$(\Delta X_i, \Delta X_{i-1}) \approx 0{,}26$ → **incréments non indépendants**, donc
  pas un brownien propre (qui les aurait décorrélés).
- régression $\Delta X_i$ sur $X_i$ : pente $\approx -0{,}001$, soit $\theta\,dt$ minuscule
  ($\theta \approx 0{,}07$) → retour à la moyenne **quasi inexistant**, donc pas un OU franc
  non plus.

Bilan honnête : ni brownien net ni OU net. On reste sur « tendance à dériver vers un niveau
négatif », sans trancher la structure fine. Le bloc autocorrélation est remplacé dans le
rapport (local) par la moyenne et la dispersion des incréments.

### Statut

- [x] reconstruire $X_t$ (dans le notebook propre, poussé sur main)
- [x] vérifier le signe de $\mu$ → négatif, conforme (prudemment)
- [x] regarder moyenne + dispersion de $X$ et des incréments $\Delta X$
- [ ] question ouverte : structure fine de $X_t$ (incréments corrélés à $0{,}26$, retour à
  la moyenne très faible) — ne pas sur-conclure avec si peu de données
- [x] notebook : ACF du niveau remplacée par les stats d'incréments (moyenne, écart-type, corr lag-1, régression $\Delta X$ vs $X$) + figure ACF du niveau comparée à des browniens simulés, à titre indicatif