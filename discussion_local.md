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

## Idée 4 — Comment tester proprement une hypothèse de modèle

Philippe a raison : un `ks_2samp` entre deux trajectoires n'a pas de sens. Je reprends la méthodo à zéro, sans m'appuyer sur le passage de `CLAUDE.md`.

### Pourquoi comparer deux trajectoires est faux

Un test à deux échantillons (KS) répond à « ces deux **échantillons i.i.d.** viennent-ils de la même loi ? ». Or les valeurs d'une trajectoire ne sont pas i.i.d. : c'est une série temporelle **autocorrélée**. Trois conséquences.

- « La loi des niveaux d'une trajectoire » est un objet qui dépend du chemin tiré, pas l'estimation d'une loi fixe. Si le processus n'est même pas stationnaire (notre cas, à cause de la descente), il n'existe pas de loi marginale unique à comparer.
- L'hypothèse d'indépendance de KS est violée, donc la p-value analytique qu'il renvoie est fausse : il sur-rejette. C'est exactement ce que j'ai vu, deux tirages du **même** modèle donnant déjà mean-p $\approx 10^{-6}$. Un test qui rejette le vrai modèle ne teste rien.
- En triant les valeurs pour comparer des CDF, on jette l'ordre temporel, là où vit toute la dynamique.

### Ce qu'un modèle affirme réellement

Une EDS $dX_t = a(X_t)\,dt + b(X_t)\,dB_t$ ne dit qu'une chose testable : une fois la dérive retirée, le **bruit est i.i.d. de loi connue**. En discret, les résidus standardisés
$$\varepsilon_i = \frac{\Delta X_i - a(X_i)\,dt}{b(X_i)\,\sqrt{dt}}$$
sont i.i.d. $\mathcal N(0,1)$ sous le modèle. C'est **là** qu'est l'objet i.i.d. dont un test a besoin, pas dans les niveaux. (Pour l'OU, $a(X_i)\,dt$ est la droite de régression $\theta\mu\,dt-\theta\,dt\,X_i$ et $b=\sigma$ ; pour la dérive, $a\,dt=\nu\,dt$.)

### Le test que je ferais — (1) sur les résidus, le cœur du modèle

1. estimer les paramètres (régression) ;
2. former les résidus $\varepsilon_i$ ;
3. tester séparément les **deux** hypothèses du bruit :
   - **loi** : KS à **un** échantillon de $\varepsilon$ contre $\mathcal N(0,1)$ (`scs.kstest`). Là, la p-value est interprétable car $\varepsilon$ est censé être i.i.d. ;
   - **indépendance** : l'autocorrélation des $\varepsilon$ doit être $\approx 0$. Si acf$_1 \approx 0{,}26$, l'hypothèse de bruit blanc tombe, indépendamment des queues.

C'est un vrai test : on confronte les deux affirmations du modèle (loi + indépendance du bruit) aux données. Nuance honnête : les paramètres sont estimés sur les mêmes données, donc le KS est un peu optimiste (biais de Lilliefors). À notre échelle on le signale, ou on le corrige par le bootstrap ci-dessous.

### Le test que je ferais — (2) calibrer une statistique par simulation

Si l'on veut juger une statistique $T$ qui capture une caractéristique (kurtosis des incréments, autocorrélation, variance du plateau, ou même une distance KS), la bonne façon d'avoir une p-value est de construire la **loi de $T$ sous le modèle** (bootstrap paramétrique) :

1. simuler $N$ trajectoires sous le modèle ajusté (même longueur, même $dt$, même $X_0$) ;
2. calculer $T$ sur chacune $\rightarrow$ loi de $T$ sous $H_0$, qui tient compte de l'autocorrélation et de la taille finie ;
3. p-value $=$ proportion des $T$ simulés au moins aussi extrêmes que $T_\text{obs}$.

C'est ainsi qu'une distance KS aurait dû servir : non pas lire la p-value analytique de `ks_2samp`, mais situer le $D$ observé (données vs modèle) dans la distribution de $D$ engendrée **sous** le modèle. Mon « sim-vs-sim » était le début de cette calibration ; il manquait juste d'y placer le $D$ observé.

### Le test que je ferais — (3) le modèle reproduit-il le phénomène

Le modèle de $X$ sert à simuler $(V,P)$ et à étudier l'extinction. Test complémentaire de bon sens : injecter $X$ simulé dans le système $(V,P)$ et vérifier par Monte-Carlo que les trajectoires reproduisent les faits observés (effondrement des prédateurs, niveau du plateau, ordre de grandeur des temps et probabilités d'extinction). C'est « le modèle reproduit-il ce qu'on voit », complémentaire du test sur le bruit.

### Conséquence pour nos rapports

Le test sur les niveaux (rapport principal) et le `ks_2samp` sur incréments (cette note) sont à refaire dans ce cadre : KS à un échantillon des **résidus** $+$ autocorrélation, et p-value calibrée par bootstrap si l'on garde une distance. À décider avec Philippe avant de toucher aux rapports.

---

## Idée 4 bis — Précisions (questions de Philippe)

### « Es-tu déjà sous l'hypothèse d'un OU ? » → oui, et c'est voulu

Le test des résidus **dépend du modèle qu'on teste**. On en teste un à la fois (un par sous-section). Pour fabriquer un résidu il faut connaître la dérive $a(X)$ et la diffusion $b(X)$ du modèle, donc on est forcément « sous » une hypothèse précise.

- dérive : $a(X)\,dt = \nu\,dt$, $b = \sigma$ ;
- OU : $a(X)\,dt = \theta(\mu - X)\,dt$, $b = \sigma$.

Le résidu, c'est la part de $\Delta X_i$ que le modèle **n'explique pas**, ramenée à l'échelle du bruit. Le construire, c'est déjà supposer le modèle ; le tester, c'est tester ce modèle-là.

### « Pourquoi comparer les résidus à une $\mathcal N(0,1)$ ? » → la normale n'est pas un choix, elle est imposée

Le modèle discret s'écrit
$$\Delta X_i = a(X_i)\,dt + b(X_i)\,\Delta B_i, \qquad \Delta B_i = B_{t_{i+1}} - B_{t_i}.$$

Par **définition** du mouvement brownien, ses incréments valent $\Delta B_i \sim \mathcal N(0, dt)$, indépendants. C'est l'hypothèse brownienne elle-même, pas une hypothèse en plus. En isolant le bruit et en le ramenant à l'échelle $\sqrt{dt}$,
$$\varepsilon_i = \frac{\Delta X_i - a(X_i)\,dt}{b(X_i)\,\sqrt{dt}} = \frac{\Delta B_i}{\sqrt{dt}} \sim \mathcal N(0,1), \quad \text{i.i.d.}$$

Le « $0$ » et le « $1$ » ne sont donc pas arbitraires : c'est $\mathcal N(0,dt)$ standardisée par $\sqrt{dt}$. Si le vrai bruit du virus n'est pas gaussien (ou pas indépendant), les $\varepsilon_i$ ne seront pas des $\mathcal N(0,1)$ i.i.d., et c'est exactement ce qu'on veut détecter.

### « En quoi ça confirme le modèle ? » → ça ne confirme rien, ça échoue (ou non) à le rejeter

Un test ne valide jamais. Ici on confronte les **deux** affirmations du bruit brownien, séparément.

1. **Indépendance** : l'autocorrélation des $\varepsilon_i$ doit être $\approx 0$. Si acf$_1 \approx 0{,}26$, le bruit blanc tombe et le modèle n'est pas retenu, sans même regarder la loi.
2. **Loi** : si l'indépendance tient, KS **à un échantillon** des $\varepsilon_i$ contre $\mathcal N(0,1)$. Une p-value faible $\to$ on ne retient pas ; une p-value qui n'est pas faible $\to$ rien ne s'oppose au modèle.

C'est le KS de goodness-of-fit qu'on connaît, mais appliqué au **bon objet** : un seul échantillon (les résidus) comparé à une loi **fixée d'avance** $\mathcal N(0,1)$, et non deux trajectoires aléatoires l'une contre l'autre. C'est là toute la différence avec le `ks_2samp` cassé — sous le modèle, les $\varepsilon_i$ sont réellement i.i.d., donc les hypothèses du KS sont satisfaites et sa p-value veut dire quelque chose.

Au passage, cela redonne proprement mon résultat de coarse-graining : au pas natif les résidus sont corrélés (0,26) et à queues lourdes $\to$ OU non retenu ; au pas grossier ils sont décorrélés et gaussiens (KS un échantillon $p \approx 0{,}84$) $\to$ OU non rejeté. Même conclusion, mais sur une base correcte.

### Test 2 → je le retire

J'avais proposé de **calibrer la distance $D$** du KS par simulation. Tu as raison de rappeler qu'on ne parle pas de $D$ : ce test reposait entièrement sur $D$ comme statistique, donc **je l'abandonne**. Le seul KS qu'on garde est celui du test 1 (un échantillon sur les résidus), dont on ne lit que la **p-value**. Si un jour on veut une p-value pour l'indépendance, on calibrerait par simulation une grandeur **interprétable** (l'autocorrélation des résidus), jamais $D$ — mais l'inspection directe de l'autocorrélation suffit déjà.

### Bilan du protocole

Deux tests, complémentaires, sans $D$ ni `ks_2samp` entre trajectoires.

- **(A) sur le bruit** : résidus du modèle $\to$ autocorrélation $\approx 0$ (indépendance) puis KS à un échantillon contre $\mathcal N(0,1)$ (loi). On ne lit que la p-value.
- **(B) sur le phénomène** : injecter $X$ simulé dans $(V,P)$ et vérifier par Monte-Carlo que l'effondrement des prédateurs, le plateau et les probabilités d'extinction sont reproduits.

---

## Idée 4 ter — Précisions (variance des résidus, indépendance, Monte-Carlo sur X)

### Pas besoin de ramener à une variance 1 (Philippe a raison)

Pour l'OU et la dérive, la diffusion $b=\sigma$ est **constante**, donc $r_i = \Delta X_i - a(X_i)\,dt = \sigma\,\Delta B_i \sim \mathcal N(0,\sigma^2 dt)$. Diviser par $\sigma\sqrt{dt}$ n'est qu'un changement d'échelle qui ne touche pas la **forme** : tester que l'histogramme des $r_i$ est gaussien revient au même. La normalisation à $\mathcal N(0,1)$ est seulement cosmétique ici.

Elle ne devient **nécessaire** que si la diffusion dépend de $X$ (brownien géométrique, $b=\sigma X$) : chaque $r_i$ a alors une variance $b(X_i)^2 dt$ différente, et il faut diviser par $b(X_i)\sqrt{dt}$ pour rendre les résidus identiquement distribués avant de les regrouper en un seul histogramme.

### La porte « cohérent » a deux battants

Histogramme gaussien $+$ autocorrélation $\approx 0$. Le premier teste la **loi** du bruit, le second son **indépendance**. Les deux sont nécessaires. Si l'un tombe, le modèle n'est pas retenu ; si les deux tiennent, le modèle est cohérent et on passe au test (B). (Natif : les deux tombent ; pas grossier : les deux tiennent.)

### Monte-Carlo sur les trajectoires de $X$ ? oui, mais plus faible

On peut simuler $N$ trajectoires de $X$ sous le modèle et comparer, mais **sur des caractéristiques interprétables** (niveau du plateau, temps de descente, amplitude), jamais la loi des niveaux (le piège du `ks_2samp`) ni $D$. On situe la valeur observée dans la distribution simulée.

Ce n'est pas équivalent au Monte-Carlo sur $(V,P)$, pour deux raisons.

- $X$ entre dans $dV = V(\tfrac23 - \tfrac43 P + X)\,dt$ et y est **intégré dans le temps**. Deux modèles de $X$ de même loi marginale mais de structure temporelle différente (autocorrélation, calendrier de la descente) donnent des $(V,P)$ très différents. Le test sur $(V,P)$ voit donc le comportement cumulé, pas seulement la marginale de $X$.
- L'extinction est définie sur $(V,P)$ (densité sous $0{,}01$), et $(V,P)$ est la donnée observée propre, tandis que $X$ est la reconstruction bruitée.

Donc Monte-Carlo sur $X$ = vérification complémentaire légère mais faible ; Monte-Carlo sur $(V,P)$ = le test pertinent. Les deux sont liés (puisque $X \Leftrightarrow (V,P)$ via la dynamique) mais pas interchangeables.