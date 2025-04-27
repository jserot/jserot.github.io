---
header-includes:
  - \usepackage{bcprules}
  - \usepackage{alltt}
---

# Un petit voyage en terres sémantiques

## J. Sérot (`jocelyn.serot@uca.fr`) , 15 septembre 2021

## Prologue

Le but de ce document est de clarifier certains concepts et notations liés à la formalisation de la
sémantique d'un langage de programmation. Je me suis en effet aperçu que certains de ces concepts,
que je croyais avoir bien compris et utilisait allègrement, n'étaient en fait pas si évidents. Comme
toujours, le diable est dans les détails.

Le premier déclencheur a été le fait que la sémantique dite naturelle ("big-step operational
semantic"), que j'ai utilisée pour formaliser CAPH et HoCL par exemple, n'est pas toujours le
formalisme le plus adéquat. Pour un langage décrivant des automates[^1] par exemple, raisonner en
termes de _valeurs_ n'est pas naturel et une sémantique fondée plus explicitement sur la notion de
_transitions_ est plus pratique.

Le second a été la difficulté que j'ai eu à appréhender les sémantiques à contexte ("à la
Felleisen"). On peut ignorer ce type de formulation dans le cas des sémantiques à grands pas. Elles
sont au contraire omniprésentes dans les textes traitant des sémantiques à petits pas.

J'ai donc entrepris, comme je l'ai déjà fait dans d'autres domaines, un travail de
"réappropriation". Ce travail est intéressant parce qu'il oblige à se poser les bonnes questions. Il
n'est pas toujours facile tant sont parfois nombreux les raccourcis et omissions dans les textes
pourtant souvent considérés comme "standard".

C'est ce travail de réappropriation / reformulation qui est résumé ici. Rien de neuf donc, seulement
une autre manière (la mienne) d'expliquer (de _m'_ expliquer en fait) les choses.

[^1]: Comme ceux décrits par L. Sylvestre dans son travail de M2 par ex.


## Rappel : sémantique opérationnelle à grands pas

Le point de départ est donc la sémantique opérationnelle à grands pas (SOGP, big-step SOS, sémantique
naturelle).

En voici un exemple, sur un langage très simple -- appelons-le `sum`, défini par la syntaxe abstraite suivante :

```
e ::= n
    | e + e
```

où les classes syntaxiques `n` et `e` désignent respectivement des entiers et des expressions
(limitées ici aux sommes d'entiers).

Quelques exemples d'expressions :

- `1`
- `1+2`
- `(1+2)+3`

Donner une SOGP de `sum` consiste

1. à définir un ensemble $V$ de valeurs (_semantic values_)
2. à spécifier la correspondance (le _mapping_) entre la syntaxe et ces valeurs, c.à.d., au final,
une fonction $f: E \rightarrow V$, où $E$ est l'ensemble des expressions défini par la syntaxe abstraite
ci-dessus[^bsfn].

Ici, on prend, tout naturellement, $V=N$, l'ensemble des entiers naturels.

Et on spécifie $f$, non pas directement sous la forme d'une fonction[^2], mais sous la forme d'un
ensemble de _règles d'inférence_. Ces règles relient chaque construction de la syntaxe à une
valeur. On dit que la sémantique est alors _dirigée par la syntaxe_. 

[^bsfn]: Fonction qui est en général _partielle_ car certains termes n'ont pas de valeur (erreur,
    divergence, ...)

[^2]: On pourrait. C'est ce que l'on fait lorsqu'on écrit directement, en Caml par exemple, un
interpréteur/évaluateur d'expressions.

Il n'y a que deux constructions syntaxiques ici : `n` et `e+e`.

Le première règle s'écrit :

```{=latex}
\infrule[r-num]
{}
{n \Rightarrow n}
```

Elle dit que tout entier s'évalue en lui même. Cela peut paraître tautologique ou redondant mais ce
ne l'est pas vraiment. A gauche du signe `=>` (qui dénote la relation d'évaluation), on a un élément
de syntaxe, à droite une valeur[^3].

[^3]: Certains textes utilisent des astuces typographiques pour distinguer ces deux catégories.

La seconde règle s'écrit :

```{=latex}
\infrule[r-add]
{e_1 \Rightarrow n_1 \quad  e_2 \Rightarrow n_2}
{e_1 + e_2 \Rightarrow n_1 \oplus n_2}
```

On peut la lire comme ceci : "_si_ le terme de gauche (`e1`) s'évalue en un entier `n1` et si le
terme de droite (`e2`) s'évalue en un entier `n2` _alors_[^4] l'expression `e1+e2` s'évalue en un
entier égal à la somme `n1+n2`. On a choisi ici, par souci de précision, d'expliciter la distinction
entre un opérateur au niveau syntaxique (`+` ici, à gauche) et la fonction mathématique
correspondante ($\oplus$ ici, à droite)[^5]. On s'affranchit parfois, lorsque le contexte permet
d'éviter l'ambiguité, de faire cette distinction (_i.e_. on notera simplement `e1+e2 => n1+n2` la
conclusion de la règle précédente).

[^4]: On a mis les termes _si_ et _alors_ en italique pour souligner le caractère déductif de le
régle d'inférence.

[^5]: Dans certains textes, la relation entre un symbole syntaxique et la fonction dénotée est
parfois explicitée via une fonction $\delta$. Par exemple $\delta(+)=\lambda x.\lambda y.x+y$.

Jusque là on est en terrain familier, tout va bien. L'approche suivie se généralise d'ailleurs assez
facilement à des langages plus complexes que `sum`.

Mais, en y refléchissant un peu, on s'aperçoit qu'elle repose sur un certain nombre de ce que l'on
pourrait qualifier de "non-dits". 

La règle `r-add`, en particulier, dit simplement que pour évaluer `e1+e2` il faut évaluer `e1` et `e2` mais
sans préciser dans quel ordre ces évaluations doivent se faire, ni même, d'ailleurs, si il doit y
avoir un tel ordre. Par exemple, pour évaluer `(1+2)+(3+4)`  doit on commencer par évaluer `1+2` ou
`3+4` ? Bien sur, dans ce cas précis, l'ordre n'a pas d'importance. Mais ce n'est pas toujours le
cas. C'est en particulier le cas si l'évaluation d'une (sous-)expression donne lieu à des effets de
bord (modification d'un état global) ou si l'ordre d'évaluation doit reflèter certaines "priorités"
(penser par exemple au langage `arith` obtenu en ajoutant à `sum` un opérateur `*`). 

Ce que dit la SOGP spécifiée par les règles `[r-num]` et `[r-add]` c'est en gros : ces détails ne nous
intéressent pas, seul nous intéresse le résultat "final" de l'évaluation.

Signalons quand même, que quand bien même on adhère à cette vision "à grands pas", la question de l'ordre
d'évaluation resurgit dès lors que l'on cherche à implémenter un interpréteur (évaluateur) pour le
langage en question. Voici par exemple un implementation "classique" d'un interpréteur pour le
langage `sum` en question écrit en `OCaml` :

```ocaml
type expr =
  Number of int
| Plus of expr * expr

let rec eval (e: expr) = match e with
| Number n -> n
| Plus (e1, e2) -> eval e1 + eval e2 
```

Ici, bien que cela puisse échapper à la première lecture[^6] de ce code, l'ordre d'évaluation
découle _de facto_ de l'ordre d'évaluation des arguments d'une fonction en `OCaml`. On peut s'en
convaincre en traçant l'exécution de la fonction `eval` :

[^6]: Et, en fait, même à la première _écriture_  !

```ocaml 
let rec eval' ~level (e: expr) =
  Printf.printf "%seval <- %s\n" (spaces level) (string_of_expr e);
  let v = match e with  
    | Number n ->
       n
    | Plus (e1, e2) ->
       eval ~level:(level+1) e1 + eval ~level:(level+1) e2 in
  Printf.printf "%seval -> %s\n" (spaces level) (string_of_int v);
  v
``` 

et en exécutant, par exemple :

```ocaml
let _ = eval' ~level:0 (Plus (Plus (Number 2, Number 3), Plus (Number 4, Number 5)))
```

ce qui donne :

```
eval <- (2 + 3) + (4 + 5)
  eval <- 4 + 5
    eval <- 5
    eval -> 5
    eval <- 4
    eval -> 4
  eval -> 9
  eval <- 2 + 3
    eval <- 3
    eval -> 3
    eval <- 2
    eval -> 2
  eval -> 5
eval -> 14
- : int = 14
```

où l'on constate que l'ordre d'évaluation est le suivant :

1. `4+5 -> 9`
2. `2+3 -> 5`
3. `9+5 -> 14`

Ce n'est pas surprenant quand on sait qu'en `OCaml` les fonctions évaluent leurs arguments de droite
à gauche mais montre clairement que c'est l'implémentation qui a ici décidé d'une propriété qui restait
"libre" dans la sémantique initiale. Encore une fois, ce n'est pas gênant ici, compte-tenu du
langage en question[^7], mais cela pourrait poser problème
si l'ordre d'évaluation "déduit" de l'implémentation n'était pas celui attendu.

C'est là que la  sémantique opérationnelle _à petit pas_ entre en jeu.

[^7]: N'importe quel ordre d'évaluation conviendrait

## Sémantique opérationnelle à petits pas

L'idée générale des sémantiques opérationnelles à petits pas (SOPP) est de décomposer l'évaluation
d'une expression -- plus généralement l'exécution d'un programme -- en une séquence d'états, chaque
état correspondant à une étape significative de calcul. La notion d' "étape significative" dépend du
langage considéré. Dans notre cas (le langage `sum` introduit à la section précédente), on peut par
exemple considérer qu'une étape correspond à un calcul effectif, c.à.d. au calcul d'une somme.

Dans l'évaluation de l'expression `(1+2)+(4+5)`, par exemple, il y aurait ainsi 3 étapes :

1. le calcul de 1+2, menant à la valeur 3
2. le calcul de 4+5, menant à la valeur 9
3. le calcul de 3+9, menant à la valeur 12

On peut représenter la succession de ces étapes en utilisant la notion classique de _système de
transitions_. L'état initial correspond alors à l'expression à évaluer, les transitions
correspondent aux étapes de calcul, les états intermédiaires représentent les "résultats
intermédiaires", c.à.d. les formes partiellement évaluées de l'expression, et l'état final
représente le résultat de l'évaluation (la _valeur_ associée à l'expression pour la SOGP).

Dans ce cadre, on peut représenter l'évaluation de l'expression `(1+2)+(4+5)`  sous la forme de la
séquence suivante de transitions :

```
(1+2)+(4+5) -> 3+(4+5)
3+(4+5) -> 3+9
3+9 -> 12
```

où chaque ligne correspond à une étape (transition).

Ce séquencement explicite permet cette fois -- et c'est justement le but ! -- de spécifier l'ordre
d'évaluation des sous-expressions.

On peut par exemple imposer -- comme on l'a fait dans l'exemple précédent, sans le dire
explicitement -- de toujours évaluer le terme de gauche d'une somme avant le terme de droite. 
Ou le contraire, ce qui donnerait la séquence suivante pour l'exemple précédent :

```
(1+2)+(4+5) -> (1+2)+9
(1+2)+9 -> 3+9
3+9 -> 12
```

Pour spécifier ces _stratégies d'évaluation_, on va, ici encore, recourir à des règles d'inférences.
Mais ces règles n'établissent plus, ici, une correspondance entre _expressions_[^8] et _valeurs_
mais la transformation d'une expression en une autre correspondant à une étape de calcul. En
d'autres termes, ces régles décrivent un _système de réécriture_[^9] menant, si tout se passe
bien[^10], au résultat final. Chaque réécriture est classiquement appelé _réduction_, dans la mesure
où l'expression produite est en général plus "courte" que celle acceptée. 

Notons tout de suite que la formulation ci-dessus pose _a priori_ un problème : si chaque étape ne
fait que transformer une _expression_ en une autre, comment peut on produire, au final, une _valeur_
?  Notons aussi que le problème ne se posait pas avec un SOGP, qui distingue clairement, justement, 
les _expressions_ des _valeurs_. La solution consiste à regrouper les deux catégories au sein d'une seule
et se doter d'un prédicat permettant de distinguer les secondes.

Dans l'optique d'une formalisation via une SOPP, on définira donc le langage `sum` de la manière
suivante :

```
e ::= n
    | e + e

```

en introduisant le prédicat `is_val` et la fonction `val` de la manière suivante :

```
is_val(n) = true
is_val(_) = false
val(n) = n
```

[^8]: Plus généralement, _termes_.

[^9]: _Term Rewriting System_ (TRS)

[^10]: On précisera cette notion plus tard. Mais disons tout de suite, que les sémantiques à petits
pas permettent de représenter plus finement les situations où le calcul "ne se passe pas bien", en
distinguant par exemple les situations d'erreurs (ex: additionner un entier et un booléen) et les
divergences (calcul qui boucle).


On peut alors expliciter les règles définissant la (une, en fait) SOPP du langage `sum` :

```{=latex}
\infrule[r1]
{\text{is\_val}(e_1)\  \text{is\_val}(e_2)\  \text{val}(e_1) \oplus \text{val}(e_2) = n}
{e_1 + e_2 \rightarrow n} 

\infrule[r2]
{\text{is\_val}(e_1)\  e_2 \rightarrow e'_2}
{e_1 + e_2 \rightarrow e_1 + e'_2}

\infrule[r3]
{e_1 \rightarrow e'_1}
{e_1 + e_2 \rightarrow e'_1 + e_2 }
```

La règle `r1` dit comment réduire une expression dont les deux termes sont des valeurs (des entiers
donc). Il suffit de faire la somme desdits termes. C'est ainsi que `1+2` se réduit en `3` par
exemple.

La règle `r2` dit comment réduite une expression dont le premier terme (à gauche) est une
valeur. On suppose ici implicitement que `r1` ne s'applique pas, et donc que les règles ont
examinées dans l'ordre `r1`, `r2` puis `r3`. On peut s'affranchir de cette interprétation
"séquentielle" du système de règles donné ci-dessus, en enrichissant les prémisses des règles
`r2` et `r3`, _i.e._ en les réécrivant :

```{=latex}
\infrule[r2]
{\text{is\_val}(e_1)\  \neg\text{is\_val}(e_2)\  e_2 \rightarrow e'_2}
{e_1 + e_2 \rightarrow e_1 + e'_2}

\infrule[r3]
{\neg\text{is\_val}(e_1)\  \neg\text{is\_val}(e_2)\  e_1 \rightarrow e'_1}
{e_1 + e_2 \rightarrow e'_1 + e_2}
```

Ceci dit, l'usage, dans la plupart des textes, est de s'appuyer sur l'interprétation séquentielle,
bien que ce ne soit pas toujours dit[^11].

[^11]: Cette interprétation est par ailleurs tout à fait intuitive pour le programmeur habitué à
implémenter un tel système de règles sous la forme de _pattern matching_, en `OCaml` par exemple,
comme on le verra par la suite.

On peut simplifier l'écriture des règles en s'appuyant sur des conventions de notations. Si on
décide de noter `n` toute expression `e` satisfaisant `is_val(e)` alors le système de règles discuté
ci-dessus se réécrit de manière plus concise (et plus lisible) sous la forme suivante :

```{=latex}
\infrule[r1]
{n_1 \oplus n_2 = n}
{n_1 + n_2 \rightarrow n}

\infrule[r2]
{e_2 \rightarrow e'_2}
{n_1 + e_2 \rightarrow n_1 + e'_2}

\infrule[r3]
{e_1 \rightarrow e'_1}
{e_1 + e_2 \rightarrow e'_1 + e_2}
```


L'évaluation d'une expression consiste alors à enchainer les réductions (via `r1`, `r2` ou `r3`)
_jusqu'à obtenir une valeur_.

$$
e_1 \rightarrow e_2 \rightarrow ... \rightarrow n
$$

où chaque réduction $e_i \rightarrow e_{i+1}$ est une instance d'une des règles `r1`, `r2` ou `r3`.

Par exemple, l'évaluation de l'expression `(1+2)+(4+5)` se décrit par la séquence de réductions
suivante, où on a annoté chaque réduction avec la règle utilisée :

```
(1+2)+(4+5) -> 3+(4+5)       [r3]
3+(4+5) -> 3+9               [r2]
3+9 -> 12                    [r1]
```

Notons qu'avec ce schéma, il est possible qu'une séquence de réduction

- se bloque (_i.e._ il n'existe aucune règle applicable)
- boucle infiniment (_i.e_ on a $e \rightarrow e \rightarrow \ldots$)

Le premier cas correspond à une "erreur" (par exemple, dans un langage avec entiers et booléens,
ajouter `1` et `false`). Le second à une non-terminaison (divergence)[^12b]. 

[^12b]: C'est le rôle des systèmes de _typage_ d'éviter les situations du premier type. Celles du second ne
peuvent pas être évitées dès que le langage décrit est Turing-complet. Le langage `sum`, bien
évidemment, n'est _pas_ Turing-complet.

**Attention** Dans la séquence de réductions illustrée ci-dessus, la règle citée à droite correspond
à la _première_ règle utilisée (celle qui est sélectionnée parce qu'elle "matche" l'expression
examinée). Mais l'instanciation de cette règle peut requérir l'instanciation d'autres règles (en
"sous-main" en quelque sorte). Par exemple, dans l'étape


```
(1+2)+(4+5) -> 3+(4+5)       [r3]
```

on a d'abord sélectionné `r3` (en prenant `e1=1+2` et `e2=4+5`),  _puis_ `r1` (en prenant `n1=1` et
`n2=2`). Cet "empilement" de règles est classiquement représenté sous la forme d'un arbre[^12]. Ici
on le noterait sous la forme suivante :

```
  1 {+} 2 = 3
  ----------- [r1]
    1+2 -> 3
----------------------  [r3]
(1+2)+(4+5) -> 3+(4+5)
```

Le nombre de réductions ne correspond donc pas, en général, au nombre de règles "utilisées"
(instanciées). Ceci apparaît clairement lorsque l'on implémente un évaluateur sur la base de la SOPP
spécifiée ci-dessus, comme illustré par le code `OCaml` suivant :

```ocaml
type expr =
  Number of int
| Plus of expr * expr

let is_value (e: expr) = match e with
  | Number _ -> true
  | _ -> false

exception Stuck of expr
        
let rec reduce (e: expr) = (* Single step reduce relation *)
  match e with  
  | Plus (Number n1, Number n2) -> Number (n1+n2)  (* r1 *)
  | Plus (Number n1 as e1, e2) -> Plus (e1, reduce e2)  (* r2 *)
  | Plus (e1, e2) -> Plus (reduce e1, e2)  (* r3 *)
  | _ -> raise (Stuck e)

let rec eval (e: expr) = (* Multi-step reduce function *)
    if is_value e then e
    else eval (reduce e)
```

Dans ce code, la fonction `reduce` implémente une étape de réduction, le _pattern matching_ sur
l'argument `e` exprimant la sélection de la règle à appliquer[^13]. La fonction `eval` implémente le
processus d'évaluation proprement dit, c.à.d. l'enchainement des réductions jusqu'à obtention d'une
valeur (ou blocage, signalé par la levée de l'exception `Stuck`). 

[^12]: Arbre de dérivation, _derivation tree_

[^13]: Le fait qu'une clause de ce _pattern matching_ ne soit examinée que si les précédentes n'ont
pas été sélectionnées correspond à interpréter _séquentiellement_ la collection de règles, comme
indiqué plus haut.

Ci suit un exemple d'exécution d'une version instrumentée de ce code (avec traçage des appels) sur
l'expression `(1+2)+(4+5)`.

```
<- (1 + 2) + (4 + 5)
  <- 1 + 2
  -> 3
-> 3 + (4 + 5)
** (1 + 2) + (4 + 5) --> 3 + (4 + 5)
<- 3 + (4 + 5)
  <- 4 + 5
  -> 9
-> 3 + 9
** 3 + (4 + 5) --> 3 + 9
<- 3 + 9
-> 12
** 3 + 9 --> 12
- : expr = Number 12

```

Les lignes commençant par `**` correspondent aux étapes de
réduction. Les paires de lignes `<-` et `->` à l'entrée et à la sortie de la fonction `reduce`. On
retrouve bien les trois étapes de réduction avec, pour chaque étape, les appels recursifs à `reduce`
correspondant à l'arbre de dérivation(s) correspondant.


Il est aisé de voir comment la forme des règles de réduction permet de contrôler l'ordre -- et plus
généralement -- la stratégie d'évaluation.

Pour forcer une évaluation "de droite à gauche" des expressions du langage `num`, par exemple, il
suffit de remplacer les règles `r2` et `r3` par les suivantes :


```{=latex}
\infrule[r2']
{e_1 \rightarrow e'_1}
{e_1 + n_2 \rightarrow e'_1 + n_2}

\infrule[r3']
{e_2 \rightarrow e'_2}
{e_1 + e_2 \rightarrow e_1 + e'_2}
```


L'expression `(1+2)+(4+5)` se réduit alors ainsi :

```
(1+2)+(4+5) -> (1+2)+9       [r3']
(1+2)+9 -> 3+9               [r2']
3+9 -> 12                    [r1]
```

### Application au lamba-calcul

Les idées précédentes s'appliquent bien évidemment au lambda calcul[^lc1].

Ci-suit donc, une SOPP, archi-classique, du lambda calcul avec constantes entières, défini par :

```
t ::= x
    | n 
    | t t
    | \x. t

v ::= n
    | \x. t
```

Les classes `t` et `v` décrivent respectivement les termes et les valeurs du langage. Comme
on l'a vu ci-dessus, les secondes sont un sous-ensemble des premiers. Les méta-variables `x` et `n`
designent respectivement les variables et les constantes entières.

Une sémantique classique est donnée par le jeu de règles suivant:

```{=latex}
\infrule[beta-v]
{}
{(\lambda x. t)\ v \rightarrow t[x \leftarrow v]}

\infrule[app-r]
{t_2 \rightarrow t_2'}
{v_1\ t_2 \rightarrow v_1\ t_2'}

\infrule[app-l]
{t_1 \rightarrow t_1'}
{t_1\ t_2 \rightarrow t_1'\ t_2}
```

La règle `[beta-v]` correspond à l'application d'une abstraction à une valeur. Le résultat est
simplement le terme de l'abstraction dans lequel toutes les occurrences de la variable liée (`x`)
ont été substituées par la valeur de l'argument[^subst].

Les deux autres règles disent comment on réduit une application lorsque le terme de gauche
n'est pas, justement, une abstraction. Si c'est une valeur (`app-r`), on réduit le terme de
droite. Sinon (`app-l`), on réduit ce terme de gauche. 

Avec ce schéma, voici comment la séquence de réductions associée au terme  `(\x.\y.y x) ((\x.x) 1))
(\x.x)` :

```
** ((\x.(\y.(y x))) ((\x.x) 1)) (\x.x) --> ((\x.(\y.(y x))) 1) (\x.x)
** ((\x.(\y.(y x))) 1) (\x.x) --> (\y.(y 1)) (\x.x)
** (\y.(y 1)) (\x.x) --> (\x.x) 1
** (\x.x) 1 --> 1
```

On notera que les règles données ici conduisent toujours à évaluer tous les arguments d'une fonction
(abstraction) avant d'appeler cette fonction (_i.e._ de substituer au sein de terme définissant). 
Cette stratégie est dite d'évaluation par valeur (_call by value_). 

Mais on peut aussi, et c'est tout l'intérêt d'un SOPP, spécifier d'autres stratégies
d'évaluation. Avec une stratégie d'évaluation par nom (_call by name_), on substitue directement,
sans évaluer les arguments. Soit :

```{=latex}
\infrule[beta-n]
{}
{(\lambda x. t)\ t' \rightarrow t[x \leftarrow t']}

\infrule[app-n]
{t_1 \rightarrow t_1'}
{t_1\ t_2 \rightarrow t_1'\ t_2}
```

Ce qui donne, pour l'exemple précédent :

```
** ((\x.(\y.(y x))) ((\x.x) 1)) (\x.x) --> (\y.(y ((\x.x) 1))) (\x.x)
** (\y.(y ((\x.x) 1))) (\x.x) --> (\x.x) ((\x.x) 1)
** (\x.x) ((\x.x) 1) --> (\x.x) 1
** (\x.x) 1 --> 1
```

L'intérêt d'une stratégie CBN, on le rappelle, est de donner un valeur à certains termes qui
divergent en CBV. Par exemple

```
** (\x.1) ((\x.(x x)) (\x.(x x))) -CBV-> (\x.1) ((\x.(x x)) (\x.(x x)))
** (\x.1) ((\x.(x x)) (\x.(x x))) -CBV-> (\x.1) ((\x.(x x)) (\x.(x x)))
** ...
```

mais 

```
** (\x.1) ((\x.(x x)) (\x.(x x))) -CBN-> 1
```

Son inconvénient, on le rappelle aussi, est de parfois conduire à évaluer de multiples fois un même argument.

[^lc1]: En fait, la plupart ont été _développées pour_ le lambda calcul. 
[^subst]: On fait l'hypothèse ici, pour simplifier, que l'opération de substitution ne génère pas de
    capture de noms.

### Redex et contextes

Dans les SOPPs décrites jusque là, on peut distinguer deux types de règles :
- celles qui effectuent un "calcul" (au sens large) proprement dit
- celles qui disent comment choisir un sous-terme (une sous-expression) et comment utiliser le
résultat de la réduction de ce sous-terme pour réduire le terme englobant[^ecrules].

[^ecrules]: Chez Harper, ces deux catégories sont nommées _instruction rules_ et _search rules_
    resp.
    
Pour le langage `sum`, par exemple, la règle `r1` appartient à la première catégorie et les règles
`r2` et `r3` à la seconde.

Pour le lambda calcul avec évaluation CBV présenté à la section précédente, ce sont les règles
`beta-v` d'une part et `app-l` et `app-r` d'autre part qui jouent ce rôle (`beta-n` et `app-n`
respectivement pour le lambda calcul avec CBN). 

Pour les règles de la seconde catégorie, le sous-terme que l'on choisit de réduire est classiquement
appelé _redex_. Lorsque l'on représente une séquence de réductions, il est utile de désigner, à
chaque étape, ce redex. On peut le faire, en pratique, en soulignant par exemple, le sous-terme
correspondant. On peut alors représenter, toujours par exemple, l'évaluation de l'expression 
 `(1+2)+(4+5)` du langage `sum` avec une stratégie d'évaluation de gauche à droite de la manière
 suivante :

```
    (1 + 2) + (4 + 5)
     ^^^^^
->  3 + (4 + 5)
         ^^^^^
-> 3 + 9
-> 12
```

Avec une stratégie d'évaluation de droite à gauche, on aurait

```
    (1 + 2) + (4 + 5)
               ^^^^^
->  (1 + 2) + 9
     ^^^^^
-> 3 + 9
-> 12
```

Sous cette forme, les redex restent toutefois une notion "externe", qui ne fait pas partie du
langage proprement dit, mais _découle_ des règles d'inférence définissant la sémantique dudit
langage.

La notion de **contexte d'évaluation** permet de formaliser la notion de _redex_ en l'intégrant à la
définition de la sémantique (et au autorisant, au passage, une formulation plus concise cette
définition).

## Contextes d'évaluation

Un _contexte d'évaluation_ (_contexte_ dans la suite pour simplifier), est une expression
dans laquelle on a remplacé une sous-expression par un "trou". Toute expression peut alors être vue
comme le "remplissage" par une sous-expression. 

Par exemple, en notant, classiquement, `[]` le "trou" dans un contexte, l'expression `1+(2+3)` du
langage `sum` peut être vue comme 

- le remplissage du contexte `1+[]` par l'expression `2+3` ou
- le remplissage du contexte `[]+(2+3)` par l'expression `1` ou encore
- le remplissage du contexte `1+(2+[])` par l'expression `3`

Cette opération de "remplissage" est typiquement notée comme une application. On écrira donc, par
exemple : `1+(2+3) = (1+[]){2+3}`[^ctxapp]

Muni de cette notion, réduire une sous-expression `e1` d'une expression `e` en `e2` peut alors se
formuler de la manière suivante :

1. décomposer `e` en un contexte `C` et la sous-expression `e1`
2. réduire `e1` en `e2` 
3. replacer `e2` dans `C`

Formellement 

```{=latex}
\infrule[]
{e=C\{e_1\}\quad  e_1 \rightarrow e_2\quad  e'=C\{e_2\}}
{e \rightarrow e'}
```

A priori, cette formulation pose néanmoins un problème : on a vu plus haut, qu'étant donné une
expression `e`, il y a en général plusieurs manières de la décomposer sous la forme d'un contexte
`C` appliqué à une sous expression `e1`.  L'idée consiste alors à choisir l'ensemble des contextes
autorisés de telle sorte que la décomposition soit toujours unique. Et c'est précisément ce choix
qui va définir la stratégie d'évaluation car il va fixer, à chaque étape la sous-expression `e1` --
le _redex_ autrement dit -- qui va être réduite. Pour que ce choix soit unique, on impose par
ailleurs que la sous-expression `e1` soit "immédiatement calculable", c.à.d qu'elle se réduise via
une règle de la première catégorie[^immred]. Dans le cas du langage `sum`, par exemple, ces expressions sont
celles de la forme `n1+n2`. Si on convient de noter "$\mapsto$" la relation de réduction
immédiate[^immred2], la régle précédente se ré-écrit donc plus précisément 

```{=latex}
\infrule[rctx]
{e=C\{e_1\}\quad  e_1 \mapsto e_2\quad  e'=C\{e_2\}}
{e \rightarrow e'}
```

Ce qui se lit : "si l'expression `e` se décompose sous la forme d'un contexte `C` appliqué à une
sous-expression `e1` et si cette sous-expression se réduit immédiatement en `e2`, appliquer `C` à
`e2` donne le résultat de la réduction de `e`.

Reste à formaliser la relation `e=C{e'}`, c.à.d le processus de décomposition d'une expression en un
contexte et une sous expression.

Pour cela, il faut définir inductivement ce qu'est un contexte, en gardant à l'esprit que cette
définition doit _de facto_ traduire la stratégie d'évaluation.

Prenons l'exemple du langage `sum`. 

Les deux règles à "traduire" sont les règles `r2` et `r3`, soit[^r1-sum]

```{=latex}
\infrule[r2]
{e_2 \rightarrow e'_2}
{n_1 + e_2 \rightarrow n_1 + e'_2}

\infrule[r3]
{e_1 \rightarrow e'_1}
{e_1 + e_2 \rightarrow e'_1 + e_2}
```

Pour `r1`, les contextes correspondants seront de la forme `n+C` où `n` est un entier et `C` un
(autre) contexte. Autrement dit quand on décompose une expression dont le membre de gauche est un
entier, le _redex_ est forcément à droite (dans une stratégie d'évaluation de gauche à droite, bien
sur). Lorsque la sous-expression de droite est immédiatement réductible, le contexte `C` est un
simple trou (conformément à la relation `rctx` qui dit que dans ce cas, il suffira de remplir `C`
avec le résultat de la réduction de cette sous-expression pour obtenir le résultat). Le cas
contraire, il faudra aller chercher "plus profond" une telle sous-expression. Le contexte `C`
traduira alors cette décomposition hiérarchique. 

Pour `r2`, les contextes seront de la forme `C+e` où `e` est cette fois une expression, ce qui
signifie que la décomposition d'une expression, lorsque le premier terme n'est _pas_ un entier,
commence par ce premier terme.

On résume tout ceci, dans le cas d'une stratégie d'évaluation de gauche à droite donc, en
définissant inductivement les contextes de la manière suivante[^ctxsum] :

```
C ::= []
    | n + C
    | C + e
```

et la relation de décomposition `e=C{e'}` de la manière suivante :

```{=latex}
\infrule[ctx-i]
{e = n_1+n_2}
{e = []\{e\}}

\infrule[ctx-r]
{e = n_1+e_2\quad  e_2 = C_2\{e'\}}
{e = (n_1+C_2)\{e'\}}

\infrule[ctx-l]
{e = e_1+e_2\quad  e_1 = C_1\{e'\}}
{e = (C_2+e_2)\{e'\}}
```

Ce que l'on peut décrire de manière plus "constructive" avec la fonction de décomposition récursive suivante, qui prend une
expression `e` et retourne une paire formée d'un contexte `C` et d'une expression `e'` tels que
`e=C{e'}` 


```{=latex}
$$
D(e)=
\begin{cases}
([],\ e) \qquad \text{si}\ e=n1+n2\\
(n1+C_2,\ e') \qquad \text{où}\ C_2,e' = D(e_2)  \qquad \text{si}\ e=n1+n2\\
(C_1+e_2,\ e') \qquad \text{où}\ C_1,e' = D(e_1)  \qquad \text{si}\ e=e1+n2\\
\end{cases}
$$
```
Quelques exemples :

- `D(1+2) = ([],1+2)`
- `D(1+(2+3)) = (1+C2,e2)` avec `C2,e2=D(2+3)=([],2+3)`, soit `(1+[],2+3)` 
- `D((1+2)+3) = (C1+3,e1)` avec `C1,e1=D(1+2)=([],1+2)`, soit `([]+3,1+2)`
- `D((1+2)+(3+4)) = (C1+(3+4),e1` avec `C1,e1=D(1+2)=([],1+2)`, soit `([]+(3+4),1+2)`
- `D((1+(2+(3+4))) = (1+C2,e2)` avec `C2,e2=D(2+(3+4))=(2+[],3+4)`, soit `(1+(2+[]),3+4)`

Comme on le voit, la décomposition d'une expression en un contexte traduit exactement la notion de
_redex_ évoquée plus haut. La relation

`1+(2+(3+4)) = (1+(2+[]){3+4}`

par exemple correspond à la première partie de la réduction notée plus haut 

```
1+(2+(3+4)) -> 1+(2+7)
      ^^^
```

La relation de "remplissage" `C{.}` se définit elle trivialement par induction sur le contexte. Dans
l'exemple précédent :

```{=latex}
\infrule[]
{}
{[]\{e\} = e}

\infrule[]
{C\{e\}=e'}
{(n_1+C)\{e\} = n_1+e'}

\infrule[]
{C\{e\}=e'}
{(C+e_2)\{e\} = e'+e_2}
```

Muni de ces définitions, on peut alors résumer la sémantique de `sum` sous la forme des deux règles
suivantes :

```{=latex}
\infrule[r0]
{}
{n_1 + n_2 \mapsto n_1 \oplus n_2}

\infrule[r1]
{e=C\{e_1\} \quad  e_1 \mapsto e_2  \quad e'=C\{e_2\}}
{e \rightarrow e'}
```

Ce que l'on écrit souvent de manière encore plus concise sous la forme suivante :

```{=latex}
\infrule[r0]
{}
{n_1 + n_2 \mapsto n_1 \oplus n_2}

\infrule[r1]
{e \mapsto e'}
{C\{e\} \rightarrow C\{e'\}}
```


Cette formulation a l'avantage de bien séparer les deux catégories de règles évoquées au début de
cette partie, à savoir
- les règles effectuant du "calcul proprement dit" (celles dites de "réduction
immédiate", `R0` ici) d'une part,
- les règles relatives à la sélection des _redex_, exprimées ici via la notion de contexte, d'autre
  part (`R1` ici).

Dit autrement, les règles du premier type disent _quelles sont_ les expressions immédiatement
réductibles, celles du second type _où on peut les trouver_.
  
Elle suppose bien entendu que soit données la définition des contextes `C` et de la décomposition d'une
expression en un contexte et une sous-expression (cf plus haut). 

Rappelons-ici que les règles `R0` et `R1` ne décrivent qu'une étape élémentaire de réduction. Le
processus d'évaluation d'une expression, qui enchaîne ces étapes jusqu'à obtention d'une valeur (ou
blocage ou divergence) est inchangé.

Ci-suit un exemple d'évaluation via le mécanisme de contextes dans le cas du langage `sum` avec une
stratégie d'évalution de gauche à droite. On a noté ici pour chaque étape la
décomposition en contexte et sous-expression :

```
(1+2)+(4+5) -> 3+(4+5)     [C=([]+(4+5)) / e=1+2 |-> e'=3]
3+(4+5) -> 3+9             [C=(3+[])   / e=4+5 |-> e'=9]
3+9 -> 12                  [C=[]       / e=3+9 |-> e'=12]
```

Un autre exemple, pour lequel le redex doit être cherché "plus profond" 

```
(1+(2+(3+4)) -> 1+(2+7)    [C=(1+(2+[])) / e=3+5 |-> e'=7]
1+(2+7) -> 1+9             [C=(1+[])   / e=2+7|-> e'=9]
1+9 -> 10                  [C=[]       / e=1+9 |-> e'=10]
```

Et un dernier, illustrant bien la stratégie d'évaluation de gauche à droite



Point essentiel -- et c'est un des intérêt des formulations à base de contextes --, la stratégie
d'évaluation ne dépend désormais que la définition des contextes. Il suffit donc de changer cette
définition pour changer la stratégie d'évaluation. Dans l'exemple précédent, si l'on veut passer
d'une stratégie d'évaluation de droite à gauche à une stratégie de droite à gauche il suffit (sans
changer la définition des contextes ici) de modifier les règles de décomposition de la manière
suivante :

```{=latex}
\infrule[ctx-r']
{e = e_1+n_2  \quad e_1 = C_1\{e'\} }
{e = (C_1+n_2)\{e'\}}

\infrule[ctx-l']
{e = e_1+e_2  \quad e_2 = C_2\{e'\}}
{e = (e_1+C_2)\{e'\}}
```

Avec cette redéfinition, l'évaluation de l'exemple précédent donne cette fois :

```
(1+2)+(4+5) -> (1+2)+9     [C=(1+2+[]) / e=4+5 |-> e'=9]
(1+2)+9 -> 3+9             [C=([]+9)   / e=1+2 |-> e'=3]
3+9 -> 12                  [C=[]       / e=3+9 |-> e'=12]
```

[^ctxapp]: Certains auteurs notent `C[e] ` l'application d'un contexte `C` à une expression
    `e`. C'est assez malheureux car cela crée une confusion avec l'usage des crochets droits (`[`,
    `]`) pour dénoter les trous.

[^immred]: Selon les textes, cette notion de "réduction immédiate" est nommée de manière très
    diverse. Harper parle ainsi d'"instructions", Felleisen de "basic notions of reduction" et Leroy
    de "head reduction".

[^immred2]: `~~>` chez Harper, `-e->` chez Leroy, `|->` aussi chez Felleisen

[^r1-sum]: La règle `r1` correspond elle à la notion de réduction immédiate évoquée ci-dessus.

[^ctxsum]: Où `n` et `e` sont issus de la syntaxe des termes et désignent donc respectivement, des entiers et des expressions.

### Application au $\lambda$-calcul

On reprend l'exemple du $\lambda$-calcul avec constantes entières introduit plus haut.
Les règles `app-l` et `app-r` peuvent se ré-écrire sous la forme


```{=latex}
\infrule[lc0]
{}
{(\lambda x. t)\ v \mapsto t[x \leftarrow v]}

\infrule[lc1]
{e \mapsto e'}
{C\{e\} \rightarrow C\{e'\}}
```

où les contextes sont définis cette fois, dans le cas d'une évaluation avec appel par valeur, de la manière suivante

```
C ::= []
    | v C
    | C t
```

## PLT-Redex

Les formulation SOPP utilisant les contextes se mécanisent très bien avec l'outil
[PLT-Redex](https://redex.racket-lang.org).
Voici par exemple un codage du langage `sum` en PLT-Redex. La première ligne de la définition décrit
la syntaxe (abstraite) des _expressions_, la seconde celle des _valeurs_. La catégorie _number_ est
prédéfinie et correspond aux entiers.

~~~scheme
#lang racket
(require redex)

(define-language sum
  (e integer (e + e))
  (v integer))
~~~

Il est alors possible, et c'est le principal intérêt de l'outil, de coder la sémantique du langage,
afin de visualiser et de vérifier des séquences de réduction. 
Pour cela, on commence par étendre le langage en introduisant la définition des _contextes
d'évaluation_

~~~scheme
(define-extended-language sum' sum)
  (C hole
     (integer + C)
     (C + e)))
~~~

définition qui correspond littéralement à la définition inductive donnée plus haut.

On peut alors traduire directement les règles réduction `r0` et `r1` données plus haut
en définissant 

```scheme
(define red
   (reduction-relation
    sum'
    (-->
     (in-hole C (v_1 + v_2))
     (in-hole C ,(+ (term v_1) (term v_2))))))
```

Il n'y a ici, dans la définition de la relation de réduction `red`, qu'une règle (`-->`). Cette
règle associe un motif (_pattern_) à un terme. Motif et terme sont ici construits avec la fonction
`in-hole`. Cette fonction correspond à la fonction de remplissage de contexte introduite plus haut. Etant donné
un contexte `C` et un terme `t`, elle construit le terme obtenu en insérant `t` dans le trou du
contexte `C`[^fremp]. La règle en question se lit donc de la manière suivante : si une expression `e` s'écrit
`C{e1}`, où `e1` est de la forme `v1 + v2`, alors on peut la ré-écrire (réduire) sous la forme
`C{v1+v2}`. Les règles `r0` et `r1`, que l'on rappelle ci-dessous, ne disaient pas autre chose.
On notera qu'on n'a _pas_ eu à spécifier explicitement comment une expression de décompose en un
contexte et un redex. La donnée de la syntaxe des contextes dans la définition du langage
suffit. 

```{=latex}
\infrule[r0]
{}
{n_1 + n_2 \mapsto n_1 \oplus n_2}

\infrule[r1]
{e \mapsto e'}
{C\{e\} \rightarrow C\{e'\}}
```

[^fremp]: Autrement dit, avec les notations introduites plus haut, elle calcule `C{t}`. 

Muni de ces définitions, on peut demander à PLT-Redex de tracer un séquence de réduction.
Par exemple, l'évaluation de

```
(traces red (term ((1 + 2) + (4 + 5))))
```

produit l'affichage reproduite sur la Fig. 1. 

![Exemple de réduction avec l'outil PLT-Redex](./figs/redex-red.png "Exemple de réduction avec l'outil PLT-Redex")

On peut ainsi "jouer" avec la sémantique et, le cas échéant, la corriger.

### Test

On peut aussi tester plus explicitement la relation de réduction, en lui donnant un terme en entrée et le résultat
attendu :

```scheme
(test-->
 red
 (term (1 + 2))
 (term 3))
> ok
(test-->
 red
 (term (1 + 2))
 (term 4)
> FAILED: expected: 4 actual: 3

```

Mais aussi de tester la relation d'_évaluation_ (c.à.d. la fermeture transitive de la relation de
réduction) :

```scheme
(test-->>
 red
 (term ((1 + 2) + 3))
 (term 6))
> ok
```

La fonction `redex-check` de PLT-Redex peut aussi faire du _random testing_.  La syntaxe d'appel est

```scheme
(redex-check <lang> <pat> <pred>
```

Elle génère aléatoirement des termes du langage `lang` conforme au motif `pattern` et vérifie que,
pour chacun de ses termes, le prédicat `pred` retourne `vrai`. Par exemple, supposons que l'on
veuille vérifier[^tauto] 
que le résultat de l'évaluation d'une expression[^eval]
de `sum` soit toujours un entier.

On commence pour cela par écrire la fonction d'évaluation

```scheme
(define (eval e)
  (car (apply-reduction-relation* red e)))

```

La fonction `apply-reduction-relation*` retourne la liste de tous les termes obtenus via la
fermeture transitive de la relation
de réduction[^appred1]. On sait ici que cette liste est toujours réduite à un singleton; on prend
donc son premier élément[^eval2].

[^appred1]: La fonction `apply-reduction-relation` renvoie, elle, la liste des termes obtenus par
une seule application de la relation de réduction.

[^eval2]: On peut se passer de la fonction `apply-reduction-relation` en définissant soi-même
`(define (step e) (if (value? e) e (car (apply-reduction-relation red e))))` et 
`(define (eval e) (if (value? e) e (eval (step e))))`.

Puis, on appelle la fonction `redex-check` :

```scheme
(define (integer? e) (redex-match? sum integer e))
(redex-check sum e (integer? (eval (term e))))
```

ce qui donne ici :

```
> redex-check: no counterexamples in 1000 attempts
```

[^tauto]: Bien que ce soit évident ici !

[^eval]: C.à.d. le terme final d'une séquence de réductions

### Le $\lambda$-calcul avec constantes entières en PLT-Redex

Le langage introduit plus haut se définit de la manière suivante en PLT-Redex

```{=latex}
\begin{alltt}
(define-language \(\lambda\)c
  (t (t t) x v)
  (v integer (\(\lambda\) x t))
  (x (variable-except \(\lambda\)))
  #:binding-forms
  (\(\lambda\) x t #:refers-to x)) 
\end{alltt}
```

La clause `binding-forms` permet de gérer proprement les substitutions (en évitant les
captures). Comme dans la définition initiale, on
a bien distingué les _valeurs_ (`v`) au sein des termes (`t`). 

La sémantique avec appel par valeur (CBV) se définit en enrichissant ce langage avec une la syntaxe
suivante pour les _contextes_

```{=latex}
\begin{alltt}
(define-extended-language \(\lambda\)-cbv \(\lambda\)c
  (C hole (v C) (C t)))
\end{alltt}
```

soit 

```
C ::= []
    | v C
    | C t
```

et en définissant la relation de réduction de la manière suivante

```{=latex}
\begin{alltt}
(define \(\beta\)-cbv
   (reduction-relation
    \(\lambda\)-cbv
    (-\->
     (in-hole C ((\(\lambda\) x t) v))
     (in-hole C (substitute t x v))
    "\(\beta\)")))
\end{alltt}
```

Ce qui peut se lire de la manière suivante : si le terme à réduire peut s'écrire `C{t1}`, où `t1` est
un terme immédiatement réductible (de la forme $(\lambda\ x\ t)\ v$ donc, alors sa réduction est simplement
`C{t2}`, où `t2` est obtenu en substituant la variable `x` par la valeur `v` dans le terme `t`. 

Par exemple $e_1=(\lambda\ x\ x)\ ((\lambda\ x\ x)\ 1)$ se réduit en $1$ parce que 

- $(\lambda\ x\ x)\ ((\lambda\ x\ x)\ 1)$ s'écrit (se décompose en) $C_1\{e_2\}$ avec $C_1=(\lambda\ x\
  x)\ []$ et $e_2=(\lambda\ x\ x)\ 1$

- $e_2=(\lambda\ x\ x)\ 1$ se réduit (immédiatement) en 1

- donc $e_1$ se réduit en $C_1\{1\} = ((\lambda\ x\ x)\ [])\{1\} = (\lambda\ x\ x)\ 1 = e_3$

- $e_3 = (\lambda\ x\ x)\ 1$ s'écrit (se décompose en) $C_2\{e_4\}$ avec $C_2=[]$ et $e_4=(\lambda\ x\
  x)\ 1$

- $e_4=(\lambda\ x\ x)\ 1$ se réduit (immédiatement) en $1$

- donc $e_3$ se réduit en $C_2\{1\} = []\{1\} = 1$

Ce que confirme la trace calculée par PLT-Redex sur la figure 2.

```{=latex}
\begin{alltt}
(traces \(\beta\)-cbv (term ((\(\lambda\) x x) ((\(\lambda\) x x) 1))))
\end{alltt}
```

Les figures 3 et 4 donnent respectivement les traces de réduction des termes

$((\lambda\ x (\lambda\ y\ (y\ x))) ((\lambda\ x\ x)\ 1)) (\lambda\ x\ x)))$

et 

$(\lambda\ x\ 1) ((\lambda\ x\ (x\ x)) (\lambda\ x\ (x\ x)))))$

Dans le premier cas, on retrouve bien la séquence donnée initialement (modulo le renommage de la
variable `y` dans le troisième terme (opéré par la fonction de substitution). 

Dans le second cas, on observe bien que la réduction boucle sur elle-même.

![Exemple de réduction avec l'outil PLT-Redex](./figs/lcbv1-red.png "Exemple de réduction avec l'outil PLT-Redex")

![Exemple de réduction avec l'outil PLT-Redex](./figs/lcbv2-red.png "Exemple de réduction avec l'outil PLT-Redex")

![Exemple de réduction avec l'outil PLT-Redex](./figs/lcbv3-red.png "Exemple de réduction avec l'outil PLT-Redex")


Pour changer la stratégie d'évaluation de CBV (call by value) à CBN (call by name), il suffit de
changer la définition des contextes et de la fonction de réduction 

```{=latex}
\begin{alltt}
(define-extended-language \(\lambda\)-cbn \(\lambda\)c
  (C hole (C t)))

(define \(\beta\)-cbn
   (reduction-relation
    \(\lambda\)-cbn
    (-\->
     (in-hole C ((\(\lambda\) x t_1) t_2))
     (in-hole C (substitute t_1 x t_2))
    "\(\beta\)")))
\end{alltt}
```

On obtient alors cette fois les traces des figures 5 et 5 pour les termes

$((\lambda\ x (\lambda\ y\ (y\ x))) ((\lambda\ x\ x)\ 1)) (\lambda\ x\ x)))$

et 

$(\lambda\ x\ 1) ((\lambda\ x\ (x\ x)) (\lambda\ x\ (x\ x)))))$

![Exemple de réduction avec l'outil PLT-Redex](./figs/lcbn1-red.png "Exemple de réduction avec l'outil PLT-Redex")

![Exemple de réduction avec l'outil PLT-Redex](./figs/lcbn2-red.png "Exemple de réduction avec l'outil PLT-Redex")

## Bibliographie

- R. Garcia lectures on Operational Semantics : [big step
semantics](https://www.cs.ubc.ca/~rxg/cpsc509/04-big-step.pdf), [small step
semantics](https://www.cs.ubc.ca/~rxg/cpsc509/05-small-step.pdf)

  -- https://www.cs.ubc.ca/~rxg/cpsc509/05-reduction.pdf

- R. Harper. [Practical Foundations of Programming Languages](https://www.cs.cmu.edu/~rwh/pfpl/2nded.pdf)

- X. Leroy. [Functional Programming Languages. Part 1. Interpreters and operational
  semantics](https://xavierleroy.org/mpri/2-4/semantics.2up.pdf)

- M. Felleisen and R. Hieb. [The Revised Report on the Syntactic Theories of Sequential Control and
State](https://www2.ccs.neu.edu/racket/pubs/tcs92-fh.pdf)

## Révisions
- 16/06/24. Ajout de la section PLT-Redex
