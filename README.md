# TP : Asservissement de vitesse d’une machine à courant continu par PID numérique - Méthode de Ziegler-Nichols

# 1. Objectifs pédagogiques

Ce TP a pour objectif la mise en oeuvre d’un asservissement de vitesse d’une machine à courant continu (MCC) à l’aide d’un microcontrôleur STM32.

Les objectifs sont :

* Comprendre la structure d’une boucle d’asservissement
* Implémenter un correcteur PID numérique
* Comprendre le passage du PID continu au PID discret
* Régler expérimentalement un correcteur
* Appliquer la méthode de Ziegler–Nichols
* Analyser les performances d’un asservissement
* Analyser l'influence de la période d'échantillonnage sur un asservissement numérique
  
---

# 2. Présentation du système expérimental

Le système étudié est constitué de :

* une machine à courant continu
* un pont en H de puissance
* un codeur incrémental
* un microcontrôleur STM32
* une commande PWM (4 signaux)

Le microcontrôleur réalise :

1. la mesure de vitesse via le codeur
2. la mesure de courant via des sondes à effet hall
3. la limitation de courant avec un PID pré-calculé déjà implémenté
4. le calcul du correcteur PID sur la vitesse
5. la génération du signal PWM

Schéma fonctionnel :
**insérer image ici**

---

# 3. Modélisation de la machine à courant continu

Le modèle classique d'une MCC est décrit par les équations suivantes.

## Équation électrique

V(t) = R i(t) + L di(t)/dt + Ke ω(t)

## Équation mécanique

J dω(t)/dt + f ω(t) = Kt i(t)

avec :
* V : tension appliquée
* R : résistance d’induit
* L : inductance d’induit
* Ke : constante de force électromotrice
* Kt : constante de couple
* J : inertie
* f : coefficient de frottement
* ω : vitesse de rotation

Dans ce TP on ne réalise pas l’identification complète du moteur. On se place dans une approche expérimentale de réglage du correcteur.

---

# 4. Principe de l’asservissement

La vitesse réelle du moteur est mesurée par le codeur incrémental.

L’erreur est définie par :

e(t) = ω_consigne(t) − ω_mesure(t)

Cette erreur est traitée par un correcteur PID qui calcule la commande appliquée au moteur.

---

# 5. Correcteur PID continu

Le correcteur PID continu s’écrit :

u(t) = Kp e(t) + Ki ∫ e(t)dt + Kd de(t)/dt

avec :
* Kp : gain proportionnel
* Ki : gain intégral
* Kd : gain dérivé

Rôle des actions :
* Action proportionnelle : rapidité
* Action intégrale : suppression de l’erreur statique
* Action dérivée : amélioration de la stabilité

---

# 6. Implémentation numérique

Le microcontrôleur calcule la commande à intervalles réguliers.

On note **Te** la période d’échantillonnage.

Les variables discrètes sont :
* e[k] : erreur à l’instant k
* u[k] : commande à l’instant k

---

# 7. Discrétisation du PID
Le PID discret peut être écrit sous la forme :
**u[k] = Kp e[k] + Ki Ts Σ e[i] + Kd (e[k] − e[k−1]) / Ts**

Afin de réduire le temps de calcul dans le microcontrôleur, on utilise une forme récursive appelée équation aux différences.

---

# 8. Équation aux différences du PID
Le correcteur peut être implémenté sous la forme :
u[k] = u[k−1] + a0 e[k] + a1 e[k−1] + a2 e[k−2]

avec
* a0 = Kp + Ki Ts + Kd/Ts
* a1 = −Kp − 2Kd/Ts
* a2 = Kd/Ts

Cette forme est particulièrement adaptée aux microcontrôleurs.

---

# 9. Implémentation dans le code STM32
Le programme principal est fourni. A l'aide d'une interface uart, vous pouvez controler le hacheur et faire tourner le moteur, configurer le PID et afficher les mesures de tension / courant.

Extrait du programme :
```
error = setpoint - speed;
u = u_prev
    + a0 * error
    + a1 * error_prev
    + a2 * error_prev2;
error_prev2 = error_prev;
error_prev = error;
u_prev = u;
```

La commande u est ensuite convertie en rapport cyclique PWM.

---

# 10. Méthode de réglage : Ziegler–Nichols

Dans ce TP on utilise la méthode expérimentale dite de l’oscillation critique.

## Étape 1 : Utiliser un correcteur proportionnel
Configurer le PID avec les paramètres suivants 
* Ki = 0
* Kd = 0

Le correcteur devient un correcteur proportionnel.

---

## Etape 2 : Faire tourner le moteur avec un rapport cyclique de 0.75, V=Vmax/2
Nous allons faire des mesures à un point de fonctionnement spécifique afin de ne pas subir les frottements secs présent lors du démarrage et être long de la vitesse maximale afin de préserver le moteur et le hacheur.

## Étape 2 : Mise en oscillations permanentes
Augmenter progressivement Kp jusqu’à obtenir des oscillations permanentes. Couper les envoies de PWM en cas d'emballement.

On mesure :
* Ku : gain critique
* Tu : période des oscillations

---

# 11. Calcul des paramètres PID

Les paramètres du PID sont donnés par :
* Kp = 0.6 Ku
* Ki = 2Kp / Tu
* Kd = Kp Tu / 8

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
[Digital Controller Tuning - Siemens](https://cache.industry.siemens.com/dl/files/379/51436379/att_93068/v1/AD353-119r2.pdf)