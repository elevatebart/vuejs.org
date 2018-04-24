---
title: Système d'Icônes SVG Dynamique
type: cookbook
order: 4
---

## Exemple de base

<p>Il y a plusieurs façons de créer un système d'icônes SVG. Une méthode qui tire parti des capacités de Vue est de créer des icônes en ligne modifiable en tant que composants. Voici quelques avantages à procéder ainsi :</p>

* Les icônes sont faciles à modifier à la volée
* Elles sont animables
* On peut utiliser des `props` et des `defaults` pour les dimensionner ou les modifier
* Elles sont écrites dans le code HTML, donc aucune requête HTTP n'est nécessaire
* Elles peuvent être rendues accessibles dynamiquement

D'abord on crée un dossier pour stocker toutes nos icônes, puis on les nomme suivant une convention pour pouvoir les appeler facilement.

> components/icons/IconBox.vue
> components/icons/IconCalendar.vue
> components/icons/IconEnvelope.vue

Voici un exemple du système une fois terminé. [https://github.com/sdras/vue-sample-svg-icons/](https://github.com/sdras/vue-sample-svg-icons/)

![Documentation site](https://s3-us-west-2.amazonaws.com/s.cdpn.io/28963/screendocs.jpg 'Docs demo')

Créons un composant icône de base (`IconBase.vue`) disposant d'un `slot`.

```html
<template>
  <svg xmlns="http://www.w3.org/2000/svg"
    :width="width"
    :height="height"
    viewBox="0 0 18 18"
    :aria-labelledby="iconName"
    role="presentation"
  >
    <title :id="iconName" lang="en">{{iconName}} icon</title>
    <g :fill="iconColor">
      <slot />
    </g>
  </svg>
</template>
```

Ce composant peut être utilisé tel quel, le seul paramètre à vérifier sera le `viewBox` qui dépendra de celui de vos icônes. Dans l'icône de base, les paramètres `width`, `height`, `iconColor` et `iconName` sont des props pour pouvoir les piloter selon les besoins. La prop `iconName` sera utilisée comme le contenu de la balise `<titre>` et de l'attribut `id` de notre icône SVG pour assurer l'accessibilité. 

Le `<script>` du composant icône de base ressemble a ça. Remarquons les valeurs par défaut. Notre icône sera toujours affichée avec les mêmes hauteur/largeur partout sauf si on les force à changer. 

```js
export default {
  props: {
    iconName: {
      type: String,
      default: 'box'
    },
    width: {
      type: [Number, String],
      default: 18
    },
    height: {
      type: [Number, String],
      default: 18
    },
    iconColor: {
      type: String,
      default: 'currentColor'
    }
  }
}
```

La valeur `currentColor` de la prop `iconColor` donne aux icônes la couleur - `fill` - du texte qui l'encadre. On peut modifier cette couleur simplement en passant une prop au composant.

Voici un exemple d'utilisation, avec le contenu de `IconWrite.vue` comme path du SVG :

```html
<icon-base icon-name="write"><icon-write /></icon-base>
```

Et si on souhaite afficher l'icône dans d'autres tailles :

```html
<p>
  <!-- on peut lui passer une `width` et une `height` plus petite grace aux props -->
  <icon-base width="12" height="12" icon-name="write"><icon-write /></icon-base>
  <!-- ou ne rien passer du tout et utiliser la width par défaut : 18 -->
  <icon-base icon-name="write"><icon-write /></icon-base>
  <!-- ou même l'agrandir :) -->
  <icon-base width="30" height="30" icon-name="write"><icon-write /></icon-base>
</p>
```

<img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/28963/Screen%20Shot%202018-01-01%20at%204.51.40%20PM.png" width="450" />

## Icônes animables

Mettre des icônes SVG dans des composants devient clé dès que l'on souhaite les animer. D'autant plus quand cette animation se passe lors d'une interaction. Les SVG `inline` sont la forme de SVG la mieux supportée. Voici comment animer une icône lors d'un click :

```html
<template>
  <svg @click="startScissors"
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 100 100"
    width="100"
    height="100"
    aria-labelledby="scissors"
    role="presentation"
    >
    <title id="scissors" lang="en">Scissors Animated Icon</title>
    <path id="bk" fill="#fff" d="M0 0h100v100H0z"/>
    <g ref="leftscissor">
      <path d="M..."/>
      ...
    </g>
    <g ref="rightscissor">
      <path d="M..."/>
      ...
    </g>
  </svg>
</template>
```

```js
import { TweenMax, Sine } from 'gsap'

export default {
  methods: {
    startScissors() {
      this.scissorAnim(this.$refs.rightscissor, 30)
      this.scissorAnim(this.$refs.leftscissor, -30)
    },
    scissorAnim(el, rot) {
      TweenMax.to(el, 0.25, {
        rotation: rot,
        repeat: 3,
        yoyo: true,
        svgOrigin: '50 45',
        ease: Sine.easeInOut
      })
    }
  }
}
```

Nous avons ajouté des attributs `ref` aux groupes de paths pour pouvoir les animer. Comme les deux cotés des ciseaux s'anime en même temps, nous allons écrire une fonction réutilisable a qui nous allons passer des `ref`. Nous pouvons compter sur GreenSock pour gérer la compatibilité de l'animation entre navigateurs notamment le problème de `transform-origin`.

<p data-height="300" data-theme-id="0" data-slug-hash="dJRpgY" data-default-tab="result" data-user="Vue" data-embed-version="2" data-pen-title="Système d'icônes SVG Dynamique : Animation" class="codepen">Voir le Pen <a href="https://codepen.io/team/Vue/pen/dJRpgY/">Système d'icônes SVG Dynamique: Animation</a> by Vue (<a href="https://codepen.io/Vue">@Vue</a>) on <a href="https://codepen.io">CodePen</a>.</p><script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

<p style="margin-top:-30px">Pas trop dur! Et en plus, super facile a modifier à la volée!!</p>

D'autres exemples d'animations sont disponibles dans le dépôt [ici](https://github.com/sdras/vue-sample-svg-icons/)

## Remarques additionnelles

Les designers peuvent changer d'avis. Les exigences du produit peuvent aussi évoluer. Conserver la logique pour l'ensemble des icônes dans un seul composant de base nous permettra de mettre a jour toutes les icônes en modifiant un seul fichier. Même en utilisant un _icon loader_, il existe des situations où recréer ou modifier chaque SVG devient nécessaire si on souhaite la même modification sur tous les icônes. Cette méthode peut vous éviter ces contrariétés et vous faire gagner du temps.

## Quand éviter ce pattern

Ce système d'icônes est particulièrement utile quand vous avez un certain nombre d'icônes qui sont utilisées de différentes façons à travers votre site. Si la même icône est utilisée de nombreuses fois sur la même page (par exemple, une table géante avec une icône de suppression sur chaque ligne), il peut être plus malin de définir un _sprite_ SVG contenant des `<symbol/>` entre des balises `<defs>` et de les référencer dans des balises `<use>`.

## Patterns alternatifs

Quelques autres outils pour aider à gérer le chargement d'icônes SVG :

* [svg-sprite-loader](https://github.com/kisenka/svg-sprite-loader)
* [svgo-loader](https://github.com/rpominov/svgo-loader)

Ces outils compilent les SVG en même temps que le bundle webpack, mais rendent la modification des icônes plus compliquée à l'exécution. En effet, la balise `<use>` réagit bizarrement sous certains navigateurs avec les dessins complexes. Ils génèrent aussi deux propriétés `viewBox` imbriquées avec deux systèmes de coordonnées. Cela peut rendre l'implémentation plus délicate.
