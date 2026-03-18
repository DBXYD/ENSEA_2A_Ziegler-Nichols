# TP : Asservissement de vitesse d’une machine à courant continu par PID numérique - Méthode de Ziegler-Nichols

# 1. Objectifs pédagogiques

Ce TP a pour objectif la mise en oeuvre d’un asservissement de vitesse d’une machine à courant continu (MCC) à l’aide d’un microcontrôleur STM32.

Les objectifs sont :

* Analyser une maquette réelle de commande numérique et l'architecture associée
* Comprendre la structure d’une boucle d’asservissement
* Régler expérimentalement un correcteur par la méthode de Ziegler–Nichols
* Implémenter un correcteur PID numérique
* Comprendre le passage du PID continu au PID discret
* Analyser les performances d’un asservissement
* Analyser l'influence de la période d'échantillonnage sur un asservissement numérique
  
---

# 2. Présentation du système expérimental

Le système étudié est constitué de :

* une machine à courant continu (48V, 10A, 500W, 3000 rpm)
* une maquette controleur de moteur composé de :
  * un pont en H de puissance
  * un codeur incrémental
  * un microcontrôleur STM32
* une commande PWM (4 signaux)

Le microcontrôleur réalise :

1. la mesure de vitesse via le codeur
2. la mesure de courant via des sondes à effet hall
3. la limitation de courant afin de ne pas détorioré le matériel
4. le calcul du correcteur PID sur la vitesse (à mettre en place)
5. la génération du signal PWM avec prise en charge des temps morts

Schéma fonctionnel :

![schema](schema_objectif.png)

Pour information, l'ensemble du hardware et du software sont disponibles [ici](https://github.com/DBXYD/Inverter_SQJQ910EL)

---

# 3. Modélisation de la machine à courant continu

Le modèle classique d'une MCC est décrit par les équations suivantes.

## Équation électrique

$$V(t) = Ri(t) + L \frac{di(t)}{dt} + K_e\Omega{}(t)$$

## Équation mécanique

$$J\frac{d\Omega(t)}{dt} + f\Omega(t) = K_ti(t)$$

avec :
* $V$ : tension appliquée
* $R$ : résistance d’induit
* $L$ : inductance d’induit
* $K_e$ : constante de force électromotrice
* $K_t$ : constante de couple
* $J$ : inertie
* $f$ : coefficient de frottement
* $\Omega$ : vitesse de rotation

Dans ce TP on ne réalise pas l’identification complète du moteur. On se place dans une approche expérimentale de réglage du correcteur.

---

# 4. Principe de l’asservissement

La vitesse réelle du moteur est mesurée par le codeur incrémental.

L’erreur est définie par :

$$ε(t) = Ω_{cible}(t) − Ω_{mesuré}(t)$$

Cette erreur sera traitée par un correcteur PID qui calcule la commande appliquée au moteur.

---

# 5. Correcteur PID continu

Le correcteur PID continu s’écrit :

$$u(t) = K_p \epsilon(t) + K_i \int \epsilon(t)  + K_d \frac{d\epsilon(t)}{dt}$$

avec :
* $K_p$ : gain proportionnel
* $K_i$ : gain intégral
* $K_d$ : gain dérivé

Rôle des actions :
* Action proportionnelle : rapidité
* Action intégrale : suppression de l’erreur statique
* Action dérivée : amélioration de la stabilité

---

# 6. Implémentation numérique

Le microcontrôleur calcule la commande à intervalles réguliers.

On note **Te** la période d’échantillonnage, et **Fe** la fréquence d'échantillonnage.

Les variables discrètes sont :
* e[k] : erreur à l’instant k
* u[k] : commande à l’instant k

---

# 7. Discrétisation du PID
Le PID discret peut être écrit sous la forme d'une équation aux différences qui sera à déterminer en fonction des paramètres du filtre établi. A partir de la formule de la transformée bilinéaire, **calculer** l'équation aux différences associé au filtre PID analogique. 

$$ p = \frac{2}{T_e}\frac{1-z^{-1}}{1+z^{-1}} $$

Le résultat sera mise sous la forme canonique à ce que la fonction de transfert d'une PID discret puisse s'écrire sous la forme : 

$$ H(z) = \frac{\sum_0^{N-1} b_k z^{-k}}{1 + \sum_0^{M-1} a_k z^{-k}} $$

**Déterminer : Les coefficients $a_k$ et $b_k$ en fonction de $K_p$, $K_i$, $K_d$ et $Te$**
**Ecriver un script dans le langage de votre choix qui calculera très rapidement les coefficients en fonction des paramètres, vous aurez besoin à plusieures reprises de calculer les $a_k$, $b_k$**



---

# 8. Implémentation dans le code STM32
Le programme principal est fourni. A l'aide d'une interface UART, vous pouvez controler le hacheur et faire tourner le moteur, configurer le PID et afficher divers information. La console vous renverra un certain nombre d'informations comme la limite de courant atteinte, etc...

Les fonctions accessibles sont disponibles à l'aide de la commande **help**. Les fonctions peuvent avoir aucun, un ou plusieurs paramètres. 

Toutes les fonctions commençant par *set* écrivent une valeur.
Toutes les fonctions commençant par *get* récupèrent une valeur.

Il faut se référer à l'aide suivante :

| Fonction          | Description                             | Arguments                                     |
|-------------------|-----------------------------------------|-----------------------------------------------|
| setLed            | Allume la led RGB                       | setLED <uint8 R> <uint8 G> <uint8 B>          | 
| setDuty           | Règle le rapport cyclique à $\alpha$    | setDuty cycle <$\alpha$>, $0 < \alpha < 1$    |
| setMotorOn        | | |
| setMotorOff       | | |
| setLoopOpen       | | |
| setLoopClose      | | |
| setCoeff          | | |

**LISTE DES FONCTIONS A COMPLETER**

---

# 9. Cablage de la maquette

A compléter

Demander à afficher à l'oscilloscope
* Sonde numérique
  * Voies 1, 2 : Encodeur numérique,
  * Voies 3, 4, 5 et 6 : PWM générées,
* Entrée analogiques :
  * Voie 1 : Mesure de tension aux bornes du moteur à travers les sondes différentielles,
  * Voie 2 : Mesure de la vitesse du moteur à l'aide de la sonde tachymétrique,
  * Voie 3 : Mesure du courant du canal *U* à l'aide d'une sonde grip-fils sur la maquette,

Brancher également la sonde tachymétrique sur un voltmètre de table pour avoir une vue constante sur la vitesse de rotation de la machine.

**Régler les unités et les gains afin de lire directement toutes les valeurs sur l'oscilloscope**
**Régler le trigger dans un premier temps sur la voie 3 numérique**

---

# 10. Méthode de réglage : Ziegler–Nichols

Dans ce TP on utilise la méthode expérimentale dite de l’oscillation critique.

## Étape 1 : Utiliser un correcteur proportionnel
Configurer le PID avec les paramètres suivants 
* $K_d = 0$, la valeur changera au cours du temps
* $K_i = 0$, la valeur reste nulle dans cette partie
* $K_d = 0$, la valeur reste nulle dans cette partie

Le correcteur devient un correcteur proportionnel.

---

## Etape 2 : Faire tourner le moteur avec un rapport cyclique de 0.75, V=Vmax/2=24V
En boucle ouverte, mesurer le point de fonctionnement du moteur. Nous allons faire des mesures à un point de fonctionnement spécifique afin de ne pas subir les frottements secs présent lors du démarrage et être loin de la vitesse maximale afin de préserver le moteur et le hacheur.
**Mesurer l'erreur de la tachymétrique à partir de la valeur mesurer de l'encodeur**
**Mesurer les frottements 

## Étape 3 : Mise en oscillations permanentes
Réitéré plusieures fois les mesures en augmentant progressivement le gain $K_p$. Augmenter progressivement Kp jusqu’à obtenir des oscillations quasi-permanentes. Couper les envoies de PWM avec l'interrupteur prévu à cette effet en cas d'emballement.

On mesure :
* $K_u$ : gain critique
* $T_u$ : période des oscillations

---

# 11. Calcul des paramètres PID

Les paramètres du PID sont donnés par :
* $K_p = 0.6 K_u$
* $K_i = 2K_p / T_u$
* $K_d = K_p T_u / 8$

# 12. Transformation du PID analogique au PID numérique


---

# 12. Manipulation expérimentale
1. Charger le programme sur le STM32
2. Observer la vitesse moteur
3. Appliquer une consigne de vitesse
4. Augmenter progressivement Kp
5. Identifier les oscillations permanentes
6. Mesurer la période Tu
7. Calculer les paramètres PID
8. Implémenter les valeurs dans le programme
9. Observer la réponse du système

---

# 13. Analyse des performances

Pour chaque réglage, relever :

* temps de montée
* dépassement
* erreur statique
* stabilité

Tracer la réponse pour :
* Correcteur P
* Correcteur PI
* Correcteur PID

---

# 14. Questions
1. Quel est l’effet de l’augmentation du gain proportionnel ?
2. Pourquoi l’action intégrale permet-elle de supprimer l’erreur statique ?
3. Quel est l’effet de l’action dérivée sur la stabilité ?
4. Quels sont les inconvénients de la méthode de Ziegler–Nichols ?
5. Comment le choix de la période d’échantillonnage influence-t-il les performances du correcteur ?

---

# 15. Conclusion
Ce TP a permis de mettre en œuvre un asservissement numérique de vitesse d’une machine à courant continu. L’utilisation d’un microcontrôleur STM32 permet de réaliser facilement un correcteur PID numérique. La méthode de Ziegler–Nichols fournit un premier réglage qui peut ensuite être affiné expérimentalement afin d’améliorer les performances du système.


## Source
* [Digital Controller Tuning - Siemens](https://cache.industry.siemens.com/dl/files/379/51436379/att_93068/v1/AD353-119r2.pdf)
* [text](http://tom.poub.free.fr/blog/XUFO/Docs/correction6.pdf)