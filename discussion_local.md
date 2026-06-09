# Discussion — journal des idées

> Tableau blanc entre Philippe et Claude. Claude écrit ici ses idées et ses
> raisonnements au fur et à mesure ; on garde une trace lisible de la réflexion,
> séparée du code (notebook) et du rendu propre (rapport). Fichier personnel,
> suivi sur la branche `Philippe`, jamais promu sur `main`.

---

## Idée 1 — Reconstruire $X_t$ à partir des données (modèle `virus4.csv`)

### Le geste de départ : isoler $X_t$ comme on a isolé $dt$

Pour récupérer $dt$, on avait utilisé l'équation **propre** (sans bruit) des prédateurs.
Ici on fait le geste inverse, sur l'équation des proies, qui est justement celle qui
porte le virus :

$$dV_t = V_t\left(\tfrac{2}{3} - \tfrac{4}{3}P_t + X_t\right)dt.$$

L'équation de $V$ ne dépend pas de $B_t$ (aucun terme $dB_t$), donc inutile de sortir
l'artillerie d'Itô. On **discrétise directement** entre $t_{i-1}$ et $t_i = t_{i-1}+dt$,
à la manière du cours 4 : on remplace $dV_t$ par l'incrément $V_i - V_{i-1}$ et on évalue
le membre de droite au début du pas.

$$V_i - V_{i-1} \approx V_{i-1}\left(\tfrac{2}{3} - \tfrac{4}{3}P_{i-1} + X_{i-1}\right)dt.$$

Il ne reste qu'à isoler $X_{i-1}$, et tout le membre de droite est observé :

$$\boxed{\,X_{i-1} \approx \frac{V_i - V_{i-1}}{V_{i-1}\,dt} - \tfrac{2}{3} + \tfrac{4}{3}P_{i-1}\,}$$

On reconstruit ainsi toute la trajectoire empirique de $X$. Même esprit « à la
physicienne » que pour $dt$ : on inverse le schéma discrétisé pour isoler l'inconnue. La
différence, c'est qu'ici on obtient une série temporelle (une réalisation du processus)
et non une constante.

### Ce qu'on attend (argument d'équilibre)

Sans virus, les points fixes sont :

$$\dot P = 0 \Rightarrow V^* = 1, \qquad \dot V = 0 \Rightarrow P^* = \tfrac{1}{2},$$

et le système classique tourne autour de $(1, \tfrac12)$. Si $X_t$ a une moyenne
$\mu = \mathbb{E}[X]$, la nullcline des proies se décale :

$$\tfrac{2}{3} - \tfrac{4}{3}P + \mu = 0 \;\Longrightarrow\; P^* = \tfrac{1}{2} + \tfrac{3}{4}\mu.$$

Donc un $X_t$ de **moyenne négative** abaisse l'équilibre des prédateurs, ce qui colle
avec leur effondrement observé. **Prédiction à vérifier : $\mu < 0$.**

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

- **Moyenne** $\approx -0{,}40$ (écart-type $\approx 0{,}23$). Négative, comme prédit par
  l'argument d'équilibre. ✓
- **$X_t$ vs $t$** : part de $\approx 0$, décroît régulièrement jusque vers $-0{,}6$ vers
  $t\approx 13$, puis se stabilise en fluctuant. Donc **non stationnaire** sur la fenêtre —
  l'intensité du virus se renforce au cours du temps, ce qui colle avec l'effondrement
  progressif des prédateurs.
- **$\Delta X_t$ vs $t$** : bruit centré, amplitude à peu près constante ($\sim\pm0{,}02$),
  pas de tendance. Les incréments, eux, ont l'air stationnaires.
- **Autocorrélation** : décroissance **lente et quasi linéaire** (encore $\approx 0{,}2$ au
  décalage $7{,}5$). Signature de mémoire longue / marche aléatoire, pas d'un retour rapide
  à la moyenne.

Lecture : ça penche vers une **structure de type brownien** (marche aléatoire, peut-être
avec dérive) plutôt qu'un OU à retour rapide. Un OU à retour très lent reste possible (son
ACF $e^{-\theta\,\Delta t}$ avec $\theta$ petit ressemble aussi à une droite). Pas encore
tranché.

### Statut

- [x] reconstruire $X_t$ dans le notebook (fait dans le notebook **propre**, poussé sur main)
- [x] regarder les trois diagnostics ($X_t$, $\Delta X_t$, ACF + moyenne)
- [x] vérifier le signe de $\mu$ → négatif, conforme
- [ ] décider brownien (avec dérive ?) vs OU lent — prochaine étape : régression
  $\Delta X_i$ sur $X_i$ et/ou croissance de la variance