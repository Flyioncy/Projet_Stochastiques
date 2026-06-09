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
- [x] notebook + rapport : finalement ni ACF ni corrélation ; juste un tableau de stats (moyenne, écart-type) sur X et ΔX (choix de Philippe)

---

## Idée 2 — Passer en revue les modèles du cours

**Échelle de bruit adaptée.** L'écart-type brut de $\Delta X$ ($\approx 6\cdot10^{-3}$) est petit
surtout parce que $dt$ l'est. L'échelle indépendante de $dt$ est le coefficient de diffusion
$$\sigma = \frac{\text{écart-type}(\Delta X)}{\sqrt{dt}} \approx 0{,}052,$$
car pour $dX = \dots + \sigma\,dB$ les incréments ont un écart-type $\sigma\sqrt{dt}$. C'est ce
$\sigma$ qu'on affichera plutôt que l'écart-type brut.

**Revue des modèles** (pour $X_t$ : part de $\approx 0$, descend vers $\approx -0{,}6$, plafonne, passe sous 0).

| Modèle | Trait | Verdict |
|---|---|---|
| Brownien géométrique / Black-Scholes | $dX=\mu X\,dt+\sigma X\,dB$, reste $>0$ | écarté (X passe sous 0, min $-0{,}71$) |
| Pont brownien | épinglé aux deux bords | écarté (X ne revient pas au départ) |
| Brownien intégral $\int B\,ds$ | très lisse ($C^1$) | écarté (incréments bruités) |
| Verhulst (logistique) | saturation vers $K$, populations $>0$ | douteux (cadre positif mal adapté) |
| Brownien avec dérive | $X=\mu t+\sigma B$, sans borne | douteux (X plafonne, une dérive non) |
| Ornstein-Uhlenbeck | retour à la moyenne vers $\mu$ | candidat principal ($0\to\mu\approx-0{,}66$) |
| OU stationnaire | déjà dans $\mathcal N(\mu,\sigma^2/2\theta)$ | douteux ($X_0$ loin de $\mu$, régime transitoire) |

**Tests (candidats douteux)** :
- retour à la moyenne : régression $\Delta X$ sur $X$ → $\theta\approx0{,}069$, $\mu\approx-0{,}66$ (rappel faible, léger + pour l'OU) ;
- variance : $\mathrm{Var}(X_{[:n]})$ de $0{,}001$ à $0{,}05$ puis sature → plutôt OU, mais contaminé par la descente ;
- gaussianité des incréments : asymétrie $\approx0$, kurtosis en excès $\approx3{,}3$ (queues épaisses) → ni OU ni brownien gaussien parfait.

**Bilan** : on écarte proprement GBM/Black-Scholes, pont brownien, brownien intégral. OU
transitoire = meilleur candidat, dérive = alternative faible, Verhulst peu naturel. Pas de
tranchage ferme (queues épaisses, une seule trajectoire de 2000 points).

### À faire (Idée 2)
- [ ] rapport : remplacer l'écart-type brut de $\Delta X$ par $\sigma = \text{std}(\Delta X)/\sqrt{dt}$ dans le tableau
- [ ] rapport : brève sous-section « Quel modèle pour $X_t$ ? » avec le tableau de revue
- [ ] notebook : figure de gaussianité des incréments (histogramme vs densité normale)
- [ ] décider avec Philippe ce qui part sur le rapport propre