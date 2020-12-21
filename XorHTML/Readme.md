# XORHTML
## Element Initial :

Une page HTML (détail qui sera important) qui serait "chiffrée" via du XOR.

## Raisonement :

Si vous connaissez le XOR, vous n'êtes sûrement pas sans savoir que c'est une opération dite commutative et associative. Concrètement, cela veut dire que : 

A^B = B^A (Commutativité) et A^(B^C) =  (A^B)^C (Associativité) où "^" représente évidemment le XOR.


La deuxième propriété est essentielle pour l'attaque qui va suivre, qui se nomme l'attaque par clair-connu. 
En effet, le XOR étant une opération associative, si nous disposons d'un bout du plaintext ainsi que du documment chiffré, nous pouvons récupérer la clé par la même opération (ou un morceau de celle ci, de longueur égale à la partie connue du texte).

## Application et marche à suivre : 

Dans un soucis de simplicité et de clarté, rendons nous sur https://gchq.github.io/CyberChef/.
Ouvrons le fichier "index.html.enc" dans cyberchef et déposons la méthode "XOR" dans recipe.

Mettons la clé (key) en Latin1 ou UTF-8, puis réflechissons.
 Nous savons que le document est un document HTML, il y a donc forcément l'en-tête HTML sur ce document. 

Cet en-tête, le voici : "\<!DOCTYPE html>".

Testons cela sur CyberChef.

 ![alt text](https://github.com/Lime667/B00Kctf/blob/master/XorHTML/cyberchef.png) 

<b>Output : </b>"V3RY53CUR3DKEY!" 

Désormais, l'étape finale consiste à remplacer l'en-tête dans "key" par "V3RY53CUR3DKEY!" et le fichier sera entièrement déchiffrer.

L'associativité du XOR est sa plus grande faiblesse, les enfants, n'utilisez pas le XOR. 

FLAG : BC{5h0r7_K3y+Kn0wN_c1pH3r=Fl4G!!}
