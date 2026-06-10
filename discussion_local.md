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

---

## Idée 3 — Recherche d'un modèle pour $X_t$ (rapport séparé `rapport_recherche_local.tex`)

Suite du rapport principal, où l'OU et la dérive avaient été écartés par un KS sur les niveaux. On creuse.

### Le test sur les niveaux ne discrimine pas

En comparant une trajectoire d'OU à une autre du **même** OU, le KS sur les niveaux donne une p-value moyenne $\approx 10^{-6}$. Le test écarterait donc même le bon modèle : sur des trajectoires longues et corrélées, il est sur-puissant. Une p-value faible sur les niveaux ne désigne pas un mauvais modèle. On bascule sur la **loi des incréments** $\Delta X$, qui est ce qu'un modèle pour $X_t$ fixe réellement.

### Les incréments natifs ne suivent aucune loi simple

- gaussien : mean-p $\approx 10^{-7}$ (queues lourdes, excès de kurtosis $\approx 3{,}3$) ;
- Laplace, mélange normal-Gamma (calés sur la kurtosis) : mean-p $\approx 10^{-3}$, mieux mais loin de $0{,}05$ ;
- en plus des queues, les incréments sont **corrélés** (acf1 $\approx 0{,}26$) → structure de haute fréquence qu'aucune loi i.i.d. ne reproduit. (On évite Student et la « dérivée seconde », pistes déjà prises par un autre groupe.)

### Coarse-graining = la piste qui marche

Si la structure fine est du bruit de haute fréquence (probablement la dérivation discrète de $V$ dans la reconstruction), agréger les incréments doit les gaussianiser (somme → normale) et les décorréler. On sous-échantillonne $X$ un point sur $k$ :

- la kurtosis s'effondre vers $0$, l'autocorrélation aussi ;
- dès $dt_k \approx 0{,}03$, des incréments **gaussiens** passent le KS (mean-p $> 0{,}05$).

Au pas $dt_k \approx 0{,}045$ ($k=3$) : kurtosis $\approx 0{,}9$, acf $\approx 0$, normalité non écartée (ks_1samp $\approx 0{,}84$). On réestime l'OU à ce pas : $\theta \approx 0{,}08$, $\mu \approx -0{,}63$, $\sigma \approx 0{,}06$. **KS sur les incréments grossiers : OU mean-p $\approx 0{,}70$, gaussien $\approx 0{,}60$** → bien au-dessus de $0{,}05$.

### Bilan et limite

Piste plausible trouvée : **OU au pas grossier**. Limites honnêtes : écart-type stationnaire $\sigma/\sqrt{2\theta} \approx 0{,}15$ vs plateau observé $\approx 0{,}055$ ; descente exponentielle qui n'épouse que grossièrement la forme en S des données. Suite naturelle : une tendance en S (croissance logistique de l'effet du virus, $-X$ qui monte de $0$ à $\approx 0{,}58$).

### Statut (Idée 3)
- [x] rapport séparé `rapport_recherche_local.tex` + notebook `notebook_recherche_local.ipynb` + figures `figures_local/`
- [x] piste avec mean-p $> 0{,}05$ (OU au pas grossier)
- [ ] à voir avec Philippe : merge dans le rapport principal ? creuser la tendance logistique en S ?

---

## Idée 4 — Comment tester une hypothèse de modèle

Un `ks_2samp` entre deux trajectoires n'a pas de sens : les valeurs d'une trajectoire ne sont pas i.i.d. (série autocorrélée), donc comparer deux « distributions de niveaux » ne teste rien — deux tirages du même modèle donnent déjà une p-value $\approx 10^{-6}$ (le test sur-rejette).

Ce qu'un modèle $dX = a(X)\,dt + \sigma\,dB$ affirme de testable : une fois la dérive retirée, le bruit est blanc et gaussien. Ses résidus $\varepsilon_i = \Delta X_i - a(X_i)\,dt$ doivent être indépendants et de loi $\mathcal N(0,\sigma^2 dt)$ (la gaussienne est imposée par le brownien, pas choisie). On teste donc le modèle par ses résidus, sur deux points : leur **loi** (histogramme vs gaussienne ; KS à un échantillon, on ne lit que la p-value, jamais la distance $D$) et leur **indépendance** (corrélation entre résidus consécutifs $\approx 0$). Si $\sigma$ est constant, inutile de normaliser à variance $1$, la forme suffit ; la normalisation par $b(X_i)\sqrt{dt}$ ne sert que si la diffusion dépend de $X$.

Un test ne confirme jamais, il échoue (ou non) à rejeter. Si les deux conditions tiennent, le modèle est cohérent et on passe au test phénoménologique : injecter $X$ simulé dans $(V,P)$ et vérifier par Monte-Carlo l'effondrement des prédateurs, le plateau et les probabilités d'extinction. (Un Monte-Carlo directement sur des caractéristiques de $X$ est possible mais plus faible, car $X$ est intégré dans une dynamique non-linéaire et l'extinction se joue sur $(V,P)$.)

Application (modèle virus4) : au pas natif, les résidus de l'OU comme de la dérive s'écartent nettement d'une gaussienne (KS $p \approx 10^{-14}$) et restent corrélés ($\approx 0{,}26$), donc ni l'un ni l'autre n'est retenu. Corrigé dans `rapport_local`.

---

## Idée 5 — Partie 2 (modèle virus6, terme $\sigma\,dX$)

Modèle : $dV = V(\tfrac23-\tfrac43 P+X)\,dt + \sigma\,dX$, $dP=P(-1+V)\,dt$. Prédateurs éteints (P sous $0{,}01$ vers $t=28$). $dt\approx0{,}0154$, $\tau\approx30{,}8$.

**Q1 ($\sigma$).** Sous-identifié : la seule variation quadratique vient de $\sigma\,dX$, $[V]_\tau=\sigma^2[X]_\tau$, donc les données ne donnent que le produit $\sigma\cdot s$ ($s$ = diffusion du virus). On isole la partie martingale par les différences secondes (le lisse est tué), corrigées par virus4 ($\sigma=0$) : $\sigma s\approx10^{-3}$. Avec $s\approx0{,}05$ (Partie 1, même virus) $\to$ **$\sigma\approx0{,}02$**.

**Q2 (EDS).** Virus reconstruit (Z, méthode P1) : marche descendante de $0$ à $\approx-1$, sans plateau $\to$ pas d'OU, on propose un **brownien avec dérive** $dX=\nu\,dt+s\,dB$, $\nu\approx-0{,}036$. Résidus : $p\approx10^{-24}$, corr $\approx0{,}26$ (mêmes réserves qu'en P1, bruit pas exactement blanc/gaussien).

**Q3 (loi de $X_\tau$).** EDS linéaire $\to X_\tau=\nu\tau+sB_\tau$ gaussienne : $X_\tau\sim\mathcal N(\nu\tau,\,s^2\tau)\approx\mathcal N(-1{,}1,\,0{,}28^2)$. Moyenne $\nu\tau\approx-1{,}1$ = valeur reconstruite. Justif : intégrale d'Itô d'un intégrand déterministe contre $B$ = gaussienne.

Fichiers : `rapport_partie2_local.tex`, `notebook_partie2_local.ipynb`, `figures_local/p2_*.png`. Local, non promu.
Limite assumée : $\sigma$ sous-identifié sans la Partie 1 ; estimation fragile (baseline virus4, $s$ de P1).