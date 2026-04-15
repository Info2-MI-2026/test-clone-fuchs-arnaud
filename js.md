# Introduction à JavaScript

> Cours orienté pratique — vous connaissez déjà Python et les bases du web. Ce document met l'accent sur ce qui est **différent**, **piégeux**, ou 

---

## 1. Déclaration de variables

Oubliez `var`. Il existe pour des raisons historiques mais son comportement est source de bugs.

```js
let   compteur = 0        // variable modifiable, portée au bloc
const PI       = 3.14     // constante, portée au bloc
```

### Règle simple : toujours `const` par défaut, `let` si on a besoin de réassigner.

```js
const nom = "Alice"       // ok, pas réassigné
let   score = 0
score = score + 10        // ok

const items = []
items.push("a")           // ok — on ne réassigne pas items, on le mute
```

**Différence avec Python :**
- En Python tout est variable. En JS, `const` ne protège pas le contenu d'un objet/tableau, juste la référence.
- La portée est au **bloc** `{}`, pas à la fonction (contrairement à `var`).

---

## 2. Types de base

```js
// Primitifs
let n  = 42
let f  = 3.14
let s  = "bonjour"        // ou 'bonjour' ou `bonjour`
let b  = true
let u  = undefined        // variable déclarée mais sans valeur
let nu = null             // absence intentionnelle de valeur

// Objets
let obj    = { nom: "Alice", age: 30 }
let tab    = [1, 2, 3]
let fn     = function() {}
```

### Pièges à éviter

```js
typeof null        // "object"  ← bug historique de JS, pas logique
typeof undefined   // "undefined"
NaN === NaN        // false  ← utilisez Number.isNaN()

// Comparaison : toujours === (strict), jamais == (avec coercition)
0 == "0"    // true  ← DANGEREUX
0 === "0"   // false ← correct
null == undefined  // true avec ==, false avec ===
```

---

## 3. Fonctions — 3 syntaxes à connaître

### Déclaration classique (hoistée)

```js
function addition(a, b) {
  return a + b
}
```

La déclaration est **hoistée** : la fonction existe dès le début du scope même si elle est écrite après son appel. À éviter en pratique pour garder du code lisible.

### Expression de fonction

```js
const addition = function(a, b) {
  return a + b
}
```

### Fonction fléchée (arrow function) — à préférer dans Vue

```js
const addition = (a, b) => a + b          // retour implicite si une seule expression
const carre    = x => x * x               // parenthèses optionnelles si un seul param
const saluer   = () => "Bonjour !"        // sans param
const complexe = (x) => {                 // avec bloc : return obligatoire
  const double = x * 2
  return double + 1
}
```

**Différence importante avec `function` : `this`**

Les arrow functions n'ont pas leur propre `this`. Elles héritent du `this` du contexte englobant. C'est généralement ce qu'on veut dans Vue, et c'est pourquoi Vue les privilégie.

```js
// Problème avec function classique dans un objet
const obj = {
  valeur: 42,
  getDouble: function() {
    const helper = function() {
      return this.valeur * 2  // this est undefined ici !
    }
    return helper()
  }
}

// Résolu avec arrow function
const obj2 = {
  valeur: 42,
  getDouble() {
    const helper = () => this.valeur * 2  // this est bien obj2
    return helper()
  }
}
```

---

## 4. Objets et déstructuration

```js
const user = { nom: "Alice", age: 30, ville: "Paris" }

// Accès
console.log(user.nom)          // "Alice"
console.log(user["nom"])       // "Alice"
console.log(user.inexistant)   // undefined (pas d'erreur !)

// Déstructuration — très courant dans Vue
const { nom, age } = user
console.log(nom)               // "Alice"

// Avec alias
const { nom: prenom } = user
console.log(prenom)            // "Alice"

// Avec valeur par défaut
const { pays = "France" } = user
console.log(pays)              // "France"
```

### Spread operator `...`

```js
const base   = { nom: "Alice", age: 30 }
const etendu = { ...base, ville: "Paris" }   // copie + ajout

const a = [1, 2, 3]
const b = [...a, 4, 5]                        // [1, 2, 3, 4, 5]
```

---

## 5. Tableaux et méthodes fonctionnelles

Ce sont les méthodes les plus utilisées avec Vue. Elles retournent un **nouveau tableau** sans modifier l'original.

```js
const nombres = [1, 2, 3, 4, 5]

// map — transformer chaque élément
const doubles = nombres.map(n => n * 2)          // [2, 4, 6, 8, 10]

// filter — garder les éléments qui satisfont la condition
const pairs = nombres.filter(n => n % 2 === 0)   // [2, 4]

// find — premier élément qui correspond
const premier = nombres.find(n => n > 3)         // 4

// some / every — au moins un / tous
nombres.some(n => n > 4)    // true
nombres.every(n => n > 0)   // true

// forEach — itérer sans retour (comme un for, mais ne pas l'utiliser pour transformer)
nombres.forEach(n => console.log(n))
```

**Règle :** préférer `map`/`filter`/`find` à `forEach` + mutation. Cela facilite le travail avec la réactivité de Vue.

---

## 6. Valeurs truthy / falsy

Contrairement à Python, JS a ses propres règles pour ce qui est considéré comme faux :

| Valeur        | Booléen |
|---------------|---------|
| `false`       | false   |
| `0`           | false   |
| `""`          | false   |
| `null`        | false   |
| `undefined`   | false   |
| `NaN`         | false   |
| tout le reste | true    |

```js
// Utilisé très souvent dans Vue pour l'affichage conditionnel
if (user.nom) { /* nom existe et n'est pas vide */ }
```

### Opérateurs pratiques

```js
// Nullish coalescing — valeur par défaut uniquement si null/undefined
const nom = user.nom ?? "Inconnu"

// Optional chaining — évite l'erreur si l'objet est null
const ville = user.adresse?.ville     // undefined si adresse n'existe pas
const zip   = user.adresse?.code?.zip
```

---

## 7. Asynchrone — Promises et async/await

JS est **single-thread** mais non-bloquant. Les opérations longues (réseau, fichiers) sont asynchrones.

### Promise

```js
fetch("https://api.example.com/data")
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(erreur => console.error(erreur))
```

### async/await — syntaxe préférée

```js
async function chargerDonnees() {
  try {
    const response = await fetch("https://api.example.com/data")
    const data     = await response.json()
    return data
  } catch (erreur) {
    console.error("Erreur :", erreur)
  }
}
```

```js
// Avec arrow function (le plus courant dans Vue)
const chargerDonnees = async () => {
  const response = await fetch("/api/users")
  return response.json()
}
```

**Points importants :**
- `await` ne peut s'utiliser qu'à l'intérieur d'une fonction `async`
- Une fonction `async` retourne toujours une Promise
- En cas d'erreur, la Promise est rejetée — toujours gérer avec `try/catch`

---

## 8. Modules — import / export

Dans Vue, tout le code est organisé en modules.

```js
// utils.js — nommés
export const PI = 3.14
export function addition(a, b) { return a + b }
export const multiplier = (a, b) => a * b

// utils.js — export par défaut (un seul par fichier)
export default function principal() { /* ... */ }
```

```js
// main.js — import
import principal              from "./utils.js"      // défaut, le nom principal est arbitraire, c'est défaut qui est important
import { PI, addition }      from "./utils.js"      // nommés
import { addition as add }   from "./utils.js"      // avec alias
import * as utils             from "./utils.js"      // tout
```

Dans Vue, vous verrez constamment :

```js
import { ref, computed, onMounted } from "vue"
```

---

## 9. Ce qu'il faut retenir pour Vue

| Concept JS             | Pourquoi c'est important dans Vue              |
|------------------------|------------------------------------------------|
| `const` / `let`        | Réactivité — Vue observe les changements       |
| Arrow functions        | `this` cohérent dans les composants            |
| Déstructuration        | `const { ref } = Vue`, props, composables      |
| `...spread`            | Copie d'objets réactifs sans muter             |
| `map` / `filter`       | Rendu de listes (`v-for`), données calculées   |
| `?.` et `??`           | Données parfois absentes avant chargement      |
| `async/await`          | Appels API dans `onMounted`, actions           |
| `import` / `export`    | Structure des composants `.vue`                |

---

## 10. Bonnes pratiques à adopter dès maintenant

1. **`const` par défaut, `let` si nécessaire, jamais `var`**
2. **Toujours `===`, jamais `==`**
3. **Arrow functions pour les callbacks et les méthodes dans Vue**
4. **Ne pas muter un tableau ou objet directement** — créer un nouveau avec `...spread` ou `map`/`filter`
5. **Toujours gérer les erreurs async** avec `try/catch`
6. **Nommer explicitement** : pas de `data`, `obj`, `temp` — des noms qui décrivent l'intention
7. **Un fichier = un module** avec une responsabilité claire

---

## Annexe — Comparaison Python / JS rapide

| Python                     | JavaScript                        |
|----------------------------|-----------------------------------|
| `def f(x):`                | `const f = x => ...`              |
| `[x*2 for x in l]`        | `l.map(x => x * 2)`               |
| `[x for x in l if x > 0]` | `l.filter(x => x > 0)`            |
| `None`                     | `null` ou `undefined`             |
| `dict`                     | objet `{}`                        |
| `f-string f"Bonjour {x}"` | template literal `` `Bonjour ${x}` `` |
| `async def` / `await`      | `async function` / `await`        |
| `import module`            | `import { fn } from "./module"`   |
