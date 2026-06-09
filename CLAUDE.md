# CLAUDE.md — Projet Méthodes Stochastiques

## Contexte du projet

Ce dépôt contient le travail de groupe pour le projet d'évaluation du cours **Méthodes Stochastiques** (SPEIT, 2025-2026). Le projet est à rendre au plus tard le **14 juin 2026 à 23h55** via Moodle, sous la forme de deux fichiers : un rapport PDF et un notebook Jupyter `.ipynb`.

L'objectif est de modéliser de façon stochastique un système proie-prédateur de type **Lotka-Volterra**, dans lequel les proies sont affectées par un virus décrit par un processus stochastique $(X_t)_{t \in [0,\tau]}$. Le projet se découpe en deux modèles distincts, chacun associé à un fichier de données réelles.

**Premier modèle** (fichier `virus4.csv`) : le système est décrit par

$$dV_t = V_t\left(\tfrac{2}{3} - \tfrac{4}{3}P_t + X_t\right)dt, \qquad dP_t = P_t\left(-1 + V_t\right)dt$$

Il s'agit de proposer un modèle pour $(X_t)$, de le valider sur les données, puis d'étudier les probabilités d'extinction des populations (victimes et prédateurs éteints si leur densité descend sous $0.01$).

**Deuxième modèle** (fichier `virus6.csv`) : le bruit est intégré directement dans l'équation de $V_t$ via un terme $\sigma\, dX_t$. Les questions portent sur l'estimation numérique de $\sigma$, la proposition d'une EDS pour $(X_t)$, et la loi de $X_\tau$.

---

## Équipe

- **Philippe LIN** — branche `Philippe`
- **Sayna REN** — branche `Sayna`
- **Xuhong LIU** — branche `Xuhong`

---

## Workflow Git

Chaque membre travaille sur sa branche personnelle et pousse sur `main` uniquement les résultats jugés pertinents et suffisamment propres. L'idée est que `main` reflète l'avancement collectif consolidé, pas les explorations en cours. En pratique :

- les essais, brouillons et codes en cours d'exploration restent sur la branche personnelle ;
- on merge vers `main` quand un résultat est validé, commenté et lisible ;
- un pull request ou un merge direct sont tous les deux acceptables, à condition de ne pas casser ce qui tourne déjà sur `main`.

### Fichiers de travail personnels et `CLAUDE.local.md`

Pour explorer sans toucher aux livrables propres, chaque membre travaille dans des fichiers de brouillon sur sa propre branche :

- `rapport_local.tex` et `notebook_local.ipynb` : copies de travail où l'on fait les essais. On y explore librement, puis on reporte le résultat validé dans `rapport.tex` / `notebook.ipynb`.
- `CLAUDE.local.md` : carnet de bord **personnel**. Claude Code le charge automatiquement dans son contexte à chaque session, en plus de ce `CLAUDE.md` partagé. Chacun crée le sien pour y noter le contexte de l'exploration en cours ; il n'est pas destiné aux autres membres.

Ces fichiers personnels peuvent être commités et poussés **sur la branche personnelle** (c'est utile comme sauvegarde), mais ils **ne doivent jamais arriver sur `main`**. Sur `main`, on ne fait remonter que les livrables propres (`rapport.tex`, `notebook.ipynb`, `figures/`, `CLAUDE.md`). Concrètement, plutôt que de merger toute la branche — ce qui emporterait les brouillons personnels — on reporte les résultats validés dans les fichiers propres puis on ne promeut que ceux-là, par exemple :

```
git checkout main
git checkout <branche> -- rapport.tex notebook.ipynb figures/
git commit
```

Ainsi chacun garde sur sa branche un Claude « au courant » de son exploration via son `CLAUDE.local.md`, sans que ces notes ni les brouillons ne polluent `main`.

---

## Répartition entre le notebook et le rapport

Les deux livrables ont des rôles **distincts et complémentaires** ; il ne faut pas dupliquer le contenu de l'un dans l'autre.

- **Le notebook (`notebook.ipynb`) est le moteur de calcul.** Il contient le code qui charge les données, estime les paramètres, lance les simulations et **régénère les figures**. Le texte y est réduit au strict minimum : un titre de section court par étape, et éventuellement une phrase de transition pour savoir ce que fait le bloc de code qui suit. Pas de paragraphes d'explication, pas de discussion des résultats, pas de contexte mathématique développé — tout cela va dans le rapport. Quelqu'un qui lit le notebook doit comprendre **ce que fait le code et comment relancer les simulations**, pas la démarche scientifique.

- **Le rapport (`rapport.tex` → `rapport.pdf`) porte tout le contexte et la démarche.** C'est là que l'on explique le problème, que l'on motive chaque choix de modélisation, que l'on présente et commente les figures (extraites du notebook), et que l'on déroule les justifications mathématiques. C'est le document qui se lit seul.

En résumé : si une phrase explique *pourquoi* on fait quelque chose ou *ce que cela signifie*, elle va dans le rapport. Si elle décrit *comment* le code procède, elle peut rester en commentaire dans le notebook.

---

## Philosophie du rapport

Le rapport est un **journal de bord scientifique**, pas une démonstration de résultats parfaits. Ce qui compte pour la note, c'est la démarche, l'honnêteté et l'originalité. Il faut donc documenter aussi bien les pistes qui ont abouti que celles qui ont échoué, en expliquant pourquoi on les a essayées et ce qu'on en a conclu.

Concrètement, pour chaque tentative pertinente, le rapport doit expliquer le raisonnement qui a motivé le choix, présenter les résultats obtenus (graphiques, statistiques, tests), et discuter de ce que cela implique, que ce soit concluant ou non.

**Sur l'utilisation de l'IA :** l'enseignant autorise l'IA à condition qu'elle soit déclarée. On ne va pas pour autant signaler chaque ligne générée, ce qui rendrait le rapport illisible. À chaque push sur `main`, le membre indiquera dans son message de commit (ou en commentaire dans le rapport) si la contribution contient des éléments produits avec l'IA qui méritent d'être mentionnés explicitement dans le rapport. Ce sera à chacun de juger ce qui est réellement significatif — utiliser Claude pour écrire une boucle `for` ne mérite pas une note de bas de page.

---

## Format du rapport

Le rapport est rédigé en **LaTeX**. Le fichier source est un `.tex` compilable en PDF. Quelques conventions adoptées pour garder un style cohérent entre les sections rédigées par des membres différents :

- Les formules mathématiques sont numérotées dès qu'on s'y réfère dans le texte.
- Chaque figure générée par le code doit apparaître dans le rapport avec une légende descriptive.
- La rédaction est en français, dans un registre scientifique mais accessible — on évite les phrases trop sèches et on prend le temps d'expliquer ce qu'on fait et pourquoi.
- Le rapport inclut une section par grande étape de travail, avec des sous-sections si nécessaire, et une conclusion générale.
- Les sections et sous-sections sont **non numérotées** (`\section*`, `\subsection*`).
- Le rapport ne comporte pas de résumé/abstract. Il s'ouvre simplement par une courte phrase en italique rappelant que tous les résultats sont reproductibles dans le notebook, où l'on trouve aussi les codes.
- Chaque paragraphe du source `.tex` est écrit **sur une seule ligne** (pas de retour à la ligne manuel à l'intérieur d'un paragraphe).

Sur le ton et la ponctuation :

- Le ton reste **personnel et mesuré**, plutôt qu'affirmatif et impersonnel. On préfère « une idée a été de… », « nous avons eu l'idée de… », « il nous a semblé… » à des formulations péremptoires comme « on peut montrer que… ».
- On **limite les deux-points** `:`, qui hachent le texte. La plupart du temps, une reformulation de la phrase est préférable.
- On **évite d'empiler les virgules**. Les conjonctions de subordination (`puisque`, `tandis que`, `alors que`, `lorsque`, `parce que`, `si bien que`…) rendent souvent mieux le lien logique. Cela dit, on garde des phrases qui ne sont pas trop longues.
- En particulier, **on ne met pas de virgule devant une conjonction de subordination** placée en milieu de phrase ; la subordonnée s'enchaîne directement (« … aucun bruit puisque l'on a… » plutôt que « …, puisque… »). La virgule ne se justifie que lorsque la subordonnée est placée en tête de phrase.
- La mise en page est **légèrement aérée** : interligne un peu augmenté et espacement entre les paragraphes (`\linespread{1.05}` et `\setlength{\parskip}{0.6em}` dans le préambule), avec quelques sauts de ligne pour détacher les équations du texte.

---

## Style de code

Le style s'inspire directement des corrections du cours. Les conventions à respecter dans le notebook :

- Les bibliothèques principales sont `numpy`, `matplotlib.pyplot` et `scipy.stats`, importées avec les alias standards `np`, `plt`, `scs`.
- Les variables sont nommées de façon explicite et cohérente avec les notations mathématiques du sujet (`V`, `P`, `Xt`, `tau`, etc.).
- Chaque cellule de code fait une seule chose. Les grandes simulations sont découpées en cellules distinctes.
- Les commentaires dans le code sont en français, concis, et expliquent le *pourquoi* plutôt que le *quoi*.
- Pour les simulations, on utilise `scipy.stats` (`.rvs`, `.cdf`, `.ppf`, `.pdf`, `.ks_1samp`, etc.) plutôt que `numpy.random` pour rester cohérent avec le cours.
- Les graphiques sont soignés : axes labelisés, titre si nécessaire, `density=True` pour les histogrammes comparés à une densité théorique.

Exemple de bloc typique tiré des corrections :

```python
import numpy as np
import matplotlib.pyplot as plt
import scipy.stats as scs

n = 10000  # Nombre de simulations

loi = scs.norm(loc=0, scale=1)
X = loi.rvs(size=n)

t = np.linspace(min(X), max(X), 300)
plt.hist(X, bins=50, density=True)
plt.plot(t, loi.pdf(t))
plt.xlabel("x")
plt.ylabel("densité")
plt.title("Comparaison simulation / loi théorique")
plt.show()
```

---

## Structure du dépôt

```
Projet_Stochastiques/
├── CLAUDE.md          # Ce fichier (partagé, sur main)
├── CLAUDE.local.md    # Carnet de bord personnel (par branche, jamais sur main)
├── Projet.pdf         # Sujet officiel
├── virus4.csv         # Données premier modèle
├── virus6.csv         # Données deuxième modèle
├── rapport.tex        # Source LaTeX du rapport (livrable propre)
├── rapport_local.tex  # Brouillon de travail (par branche, jamais sur main)
├── notebook.ipynb     # Jupyter notebook principal (livrable propre)
├── notebook_local.ipynb # Brouillon de travail (par branche, jamais sur main)
└── figures/           # Figures générées par le notebook
```

---

## Résumé des livrables

- `rapport.pdf` (généré depuis `rapport.tex`) : journal de bord rédigé, qui porte le contexte, la démarche, les figures commentées et les justifications mathématiques. C'est le document qui se lit seul.
- `notebook.ipynb` : code propre, commenté, reproductible et **peu bavard** — il charge les données, lance les simulations et régénère les figures du rapport, sans dupliquer les explications de fond.