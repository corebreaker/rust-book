# RUST
- Surcharge d'opérateur
- Programmation système bas niveaux -> peu développer des drivers de périphériques
- Code source du livre: https://github.com/ProgrammingRust
- Le trait `Drop` ne définit pas le comportement de libération de la mémoire, mais celui de l'abandon de possession, l'objet n'a plus de possesseur lorsque la méthode `drop` est appelée


# Installation
- Il est conseillé de suivre l'installation par Rustup (https://rustup.rs)
   - ***TODO:*** procédure d'installation
- Plus d'outils offert par rapport à une installation par les gestionnaires de package (APT, Brew, ...), donc rustup fourni un environnement de dev plus récent et plus complet
- Mise à jour: `rustup update`


# Nouveau projet
Binaire:
- cargo new --bin mon-projet-main
ou
- cargo new mon-projet-main

- Sans GIT: cargo new --vcs-none mon-projet-main

Bibliothèque:
- TODO: commande pour une lib


# Compile
- `assert_debug` peut permettre de faire des test d'ingérité en phase de dev et qui ne seront pas inclut dans la release en production
- La commande `cargo` se lance dans n'importe quel sous-répertoire du projet
- `cargo build --release`


# Documentation
- Docs sur le net: https://www.rust-lang.org/learn
- Doc en local `rustup doc`


# Exercices
- https://github.com/rust-lang/rustlings/


# Présentation des valeurs et la notion objet
....déclaration....
....présentation des traits... 
à l'exception des boucles toutes les instructions sont des expressions


# Variables et possession (ownership)
- Une variable a toujours un seul type, si une expression peut avoir plusieurs valeurs, comme avec le bloc `if`, chaque alternative doit avoir le même type:
```rust
let x = if condition {
    123u8 // Type u8
} else {
    0 // Ce sera obligatoirement du type u8
};
```

- Les variables sont au sens large un emplacement sur la pile (stack), que ce soit une simple variable locale ou globales, un argument de fonction mais aussi dans une variante de l'expression `match`
- Les variables doivent toujours être initialisées avant son utilisation, même si l'initialisation est différée dans un block `if/else` ou un autre type de bloc:
```rust
let my_var;

if condition {
    my_var = 123;
} else {
    my_var = 0;
}
```

Il est évident que l'alternative `else` est nécessaire, sinon la variable `my_var` ne serait pas initialisée.
Bien que cette manière de faire, on préfèrera cette manière de faire qui est plus lisible:
```rust
let my_var = if condition {123} else {0};
```

- Une variable possède sa valeur, et si sa valeur n'est plus accessible, elle est détruite.
- Une variable ne possède qu'une seule valeur à la fois, s'il y a plusieurs valeurs c'est que la valeur possédée par la variable est un conteneur (Tuple, Array, Vecteur, HashMap, etc.), et dans ce cas chaque valeur est possédée par le conteneur.
- A l'inverse, une valeur ne peut être possédée que par une seule variable à la fois, c'est le principe de base de la gestion de la mémoire faite par Rust.

- Il existe une possibilité de gérer la possession pour qu'elle soit distribuée sur plusieurs variable un utilisant:
    - Le type `std::rc::Rc` pour du code en dehors d'un contexte multithread
    - Le type `std::sync::Arc` pour du code utilisé dans un contexte multithread
C'est ce genre de type qui va réellement posséder une valeur, il exploite un compteur de référence pour savoir quand la valeur possédée doit être détruite.

- L'assignation d'une valeur à une variable transfère la possession à la variable:
```
let a = 123; // La variable `a` possède la valeur `123` 
let b = 0; // La variable `v` possède la valeur `0` 

let mut x = a; // la valeur 123 est maintenant possédée par la variable `x`
x = b; // la valeur 0 est maintenant possédée par la variable `x`
```
    - Lorsque la valeur `123` est possédée par la variable `x`, la variable `a` devient non-initialisée, il ne faut donc pas l'utiliser sinon le compilateur vous enverra un message d'erreur de compilation.
    - Lorsque la valeur `0` est possédée par la variable `x`, la variable `b` devient non-initialisée mais aussi la valeur `123` est détruite.

- Pour les types copiants (implémentant le trait Copy), il n'y a pas de transfert de possession; c'est le cas des types primitif (`i8`, `i16`, `u32`, `usize`, etc.),
car la valeur est copiée est c'est la copie que la nouvelle variable va posséder.
- Un type ne peut être un type copiant s'il a besoin d'un traitement pour libérer une valeur, c'est le cas des vecteurs qui ont besoin de libérer l'espace mémoire nécessaire pour stocker chaque valeur qu'il contient,
ou comme un fichier qui a besoin de fermet le fichier avant de détruire l'objet fichier.
- Pour créer un nouveau type copiant (type personalisé), il faut que chaque elément qu'il aggrège ait pour type un type copiant

- Le transfert de possession peut aussi se faire par le passage d'un paramètre de fonction, par retour d'une fonction
**Page 89**

- Le transfert de possession peut se faire dans un bloc `if` et ainsi rendre une variable non-initialisée
**Page 92**

- La possession des valeurs d'une collection est transférée dans la variable d'itération d'une boucle, donc chaque valeur sera détruite à chaque itération.
- Il aussi un tranfert de possession du conteneur dans la boucle, et le conteur est alors détruit à la sortie de la boucle.
- **Page 95** déplacement de value -> changer le type pour permettre d'avoir des valeur vides

- **Todo** variable shadowing: 
let x = 123;
let x = x.to_string();


## Mutatibilité par le mot clé `mut`
- Une variable est par défaut immuable, on ne peut pas la réassigner
- Une valeur est par défaut immuable, on ne peut pas changer son contenu en mémoire
- Pour pouvoir réassigner une variable ou pour pouvoir changer le contenu en mémoire d'une valeur, il faut qu'une valeur soit déclarée mutable et se fait avec le mot clé `mut` que l'on place avant l'identifiant de la variable:
```rust
let mut x = 123; // La variable va pouvoir être réassignée
let mut v = vec![]; // Cette variable va permettre d'appeler des méthodes du vecteur pour le modifier

x = 0; // Ok, car la variable `x` est mutable;
v.push(x); // Ok, car la variable `v` est mutable, on peut donc modifier le contenu du vecteur
```

- Un argument `mut` (mutable) ne veut pas dire que la valeur va modifiée après l'appel de la fonction, ce n'est pas un passage par référence
La fonction
```rust
fn f(mut n: i64) -> i64 {
    n += 1;
    n
}
```

Code 1:
```rust
let x = 1234;

println("Valeur: {}", f(x));
println("Après: {}", x); // Affiche 1234, `x` n'est pas mutable, normal donc
```

Code 2:
```rust
let mut x = 1234;

println("Valeur: {}", f(x));
println("Après: {}", x);
```

Le code 1 compile.
Le code 2 compile aussi mais avec un warning disant que la variable `x` n'a pas besoin d'être mutable ce qui prouve bien qu'elle n'est pas modifiée.


## Types immuables
- Certains types n'ont pas de méthode qui vont modifier leur valeur, il sont immuables.
- Mais la variable peut cependant être ré-assignée, c'est pour cela que le mot clé `mut` et tout de même nécessaire.
- C'est le cas d'une valeur entière, sa valeur intrinsèque ne peut être modifiée mais on peut assigner une autre valeur à la variable qui la possède.


## Variables statiques
- Les variables globales existent en Rust mais elle sont constantes. Elle sont largement déconseillées. Ainsi elle ne sont que très rarement utilisées.
- C'est comme en Python, il n'existe pas de variable globale au niveau du programme, car étant déclarées dans un module, ce sont en fait, des variables «locales» à un module,
c'est pour cela qu'en Rust on les appelles statiques à la place de globales.
- Il faut comprendre qu'une variable globale est commune à toutes les fonctions déclarées dans un module.
- Contrairement aux variables locales, on doit indiquer le type d'une variable globale.
- Les variables globales se déclarent avec:
    - le mot-clé `static`:
```rust
static my_global: u32 = 1234;
```
    - le mot-clé `const`:
```rust
const my_global: u32 = 1234;
```
- Dans tous les cas, le type doit être spécifié dans une déclaration d'une variable globale
- 
 


# Conventions
- Les noms des types sont en camel case majuscule
- Les noms des variables et des fonctions (y compris les méthodes) sont en snake case minuscule


# Expressions
- La méthode `unwrap` d'un résultat (`Result`) peut être utilisées pour pouvoir compiler sans erreur lorsqu'on ne veut pas utilisé le résultat, que ce soit la valeur ou l'errur.
Car Rust nous oblige d'utiliser le résultat, la méthode `wrap` étouffe la valeur de retour, que ce soit une donnée pour un retour de variante `Ok` ou une erreur pour un retour de variante `Err`.
- Tout est expression
Voici l'opérateur ternaire en Java, JS, C, C++:
```java
    int i = x ? y : z;
```

Voici l'équivalent en Rust
```rust
    let i = if x { y } else { z };
```
- Avec les `if` et `match` sous forme d'expressions Rust nous oblige à traiter tous les cas


# Types
**Page 49**
| Type                    | Description               | Exemples                                                           |
|:-----------------------:|:-------------------------:|--------------------------------------------------------------------|
| i8, i16, i32, i64, i128 | Entier signé              | -120, 1000, 1_000, 0x10, -0o12, 0b10, 0b_0111_1111, -1i8, 10_i16   |
| u8, u16, u32, u64, u128 | Entier non-signé          | b'X', 5, 2_000, 0x11, 0o10, 0b10, 0b_1111_1111_1111, 5u16, 501_u32 |
| isize, usize            | Mot machine               | 1, 10usize, 1234_isize                                             |
| bool                    | Booléen                   | true, false                                                        |
| f32, f64                | Flottant                  | 1., 1e-23, 3.5f32, 1234_f64                                        |
| char                    | Caractère                 | 'A', '\x22', '\u{00012c}'                                          |
| &str                    | Chaine à taille fixe      | "Hello Alain", "Yes \x32 \t \u{3f}"                                |
| String                  | Chaine à taille variable  | "Hello Alain".to_string(), "Yes \x32 \t \u{3f}".to_string()        |
| enum                    | Énumération               | enum WeekEnd { Saturday, Sunday }, enum Wait { OnTime, Late(u32) } |
| &T                      | Référence partagée        | let z: &i64 = &x;                                                  |
| &mut T                  | Référence mutable         | let &mut x                                                         |
| Box<T>                  | Boite (pointeur sur heap) | Box::new(1_i32)                                                    |
| [T; n]                  | Tableau (Array)           |                                                                    |

- Le préfixe `i` pour `i8`, `isize`, etc. spécifie un type entier signé
- Le préfixe `u` pour `u8`, `usize`, etc. spécifie un type entier non-signé
- Pour rappel, le mot machine est un entier de la taille d'une adresse memoire, c'est la valeur d'un pointeur. Un pointeur est de la même taille qu'une valeur de type `isize` ou `usize`.
Donc la taille du mot machine (et donc des types `isize` et `usize`) va dépendre de l'espace d'adressage du processeur, 32 bits ou 64 bits.
- Pour un littéral flottant, la partie entière est obligatoire, `.123` n'est pas permis, il faut écrire `0.123`.
- Le type `char` est sur 32 bits
- `'A'` est un littéral de type `char` mais `b'A'` est un littéral de type u8 (octet)
- Les buffers mémoire, les flux, les données de fichiers sont de type `[u8]` (tableau d'octets)
- Une chaine de caractères est codée en UTF-8, ce n'est donc pas une liste de valeur de type char comme `[char]`, c'est une liste d'octets donc du type tableau d'octet: `[u8]`
- Inférence des types => pas besoin de mettre le type lors de la déclaration
```rust
let x = 123_i32;
// Equivaut à:
let x: i32 = 123;
```
- On évite les confusions sur les valeurs comme c'est le cas en C/C++ où une même valeur peut avoir plusieurs significations.
En C, la valeur zéro dans une variable peut signifier:
    - L'entier numérique 0
    - Un pointeur NULL
    - Le booléen FALSE
    - Un accès réussi à un fichier ou à une base de données qui a une signification inverse du FALSE

Le simple zéro peut donc indiqué une réussite (requête en base de données correctement exécutée) ou un échec (booléen FALSE), il y a confusion sur la signification d'une valeur et donc d'une variable.
En Rust, les variables, les types et les variables sont plus clair. Pour les accès aux ressources (fichiers, base de données), les valeurs retournées sont claires par le type enuméré `Result`:
    - `Ok` en cas de réussite
    - `Err` en cas d'échec
- Les littéraux numériques peuvent être suffixés par le type: `123i64`, `456usize`
- Les littéraux numériques peuvent avoir des underscores pour une meilleure présentation et lecture des nombres: 
    - `1_000` au lieu de `1000`
    - `0b_1111_1111` au lieu de `0b11111111` pour la notation binaire
    - `0x12_3d_56_af` ou `0x_123d_56af` au lieu de `0x123d56af` pour la notation hexadécimale
    - `1_i64` au lieu de `1i64` pour le suffixe de type


## Cast
- **Page 53/54**
- `false as i32` -> `0` (`bool as T` avec `T` un type numérique est autorisé)
- `0 as bool` -> interdit (`T as bool` avec `T` un type numérique est autorisé)
- `'A' as i32` -> `65` (`char as T` avec `T` un type numérique est autorisé)
- `65u8 as char` -> `'A'` (`u8 as char` est autorisé)
- `65u16 as char` -> interdit (`T as char` avec `T` un type numérique autre que `u8` est interdit pour éviter les plages unicode interdite)
- La plage unicode 0xD7FF-0xDFFF est interdite, c'est pour cela qu'on ne peut pas convertir une valeur de type `u16` en un caractère unicode sans cette interdiction Rust devrait vérifier la valeur pendant l'éxecution.
Ce qui contredit la volonté de Rust qui se veut plus performant.
- Pour convertir une valeur de type `u2` en caractère il faut utiliser la méthode `std::char::from_u32` qui retourne le type `Option<char>` et indique donc par la valeur `None` qu'une valeur ne peut pas être convertie
- Pas de conversion implicite
- Une inférence de type n'est pas une conversion
```rust
let a = 12i16;      //La variable `a` aura le type `i16` par inférence de type, ce n'est pas une conversion.
let b: i64 = 1_i32; // Interdit: car Rust ne fait pas de conversion implicite
let c: i64 = 1;     // Le littéral «1» aura le type `i64` par inférence de type
let d = 123;        // La variable `d` aura le type `i32` par inférence de type et parce que le type entier par défaut est `i32`
let e = 12.3;       // La variable `e` aura le type `f64` par inférence de type et parce que le type entier par défaut est `f64`
```
- le module `std::convert` sert à faire des conversions automatiques (**TODO**: exemple)


## Pointeur
- Il existe 3 types de pointeurs:
    - Référence ou emprunt (borow), il en existe de 2 type
        - La référence partagée, pour un accès en lecture seule: `&T`
        - La référence mutable, pour un accès en lecture/modification: `&mut T`
    - Boite (Box): `Box<T>`
    - Pointeur brut: `*T`
- `&x` emprunte une référence sur `x`, la valeur est une adresse en mémoire sur la pile (stack) ou sur le tas (heap), l'adresse qui pointe sur la valeur de `x`
- `&x` est du type `&T` (comme `&i32`, `&Bobo`, etc.)
- La référence partagée est un type copiant (trait `Copy`)
```rust
let a = &x;
let b = a; // Il n'y a pas transfert de possession, la référence est copiée, et c'est la copie que possède la variable `b`
let c = a; // La référence est de nouveau copié, ceci serait impossible avec un type non-copiant
```
- Une référence sur une valeurs mutable (simplifiée en nommant cela référence mutable) permet d'emprunter une référence sur une valeur mutable, on initialise une telle variable par l'opérateur `&mut` 
```rust
let mut v = vec![];
let r = &mut v;
```
- Sans mettre le mot clé `mut` sur l'opération `&mut`, ainsi avec juste `&`, cela ferait juste un emprunt immuable, et le déréférencement `*r` ne permettrai pas de modifier la valeur même si la valeur référencée, elle est modifiable.
- Une référence mutable est de type `&mut T` et ce n'est pas un type copiant, il y a donc un transfert de possession à la réassignation d'une variable
- `*p` récupère la valeur pointée par le pointeur `p`, qu'il soit une référence, une boite (`Box<T>`) ou un pointeur brut
- Une référence Rust est un pointeur qui ne sera jamais nul, il est impossible d'assigner une valeur nulle à une référence.
- Il n'y a pas de référence nulle car il n'existe pas de constante pointeur nul comme `NULL` en C, `null` en Java et JS, `None` en Python, ou `nil` en Go.
- On peut cependant assigner un zéro à un pointeur brut dans du code non-sûr (unsafe) mais ce pointeur brut ne pourra être transformé en référence que grâce à du code unsafe donc impossible à utiliser dans du code Rust sûr. 
- Si on veut indiquer l'absence de référence (et donc d'objet ou d'élément donné sous forme de référence), il faut utiliser le type `Option<&T>`:
```rust
let x: Option<&i32> = None;
```

- L'existence du type `Option<&T>` évite un mauvaise utilisation d'un pointeur nul et donc les `NullPointerException`, en Rust il n'exite pas d'équivalent au `NullPointerException`, et ça ne se produit jamais tout comme le "Segmentation Fault".
- Si un éxecutable produit à partir d'un code en Rust produit un "Segmentation Fault", que cela vient de code non-Rust ou du code unsafe.
- Les pointeurs bruts peuvent être initialisés dans un bloc safe mais il faut obligatoirement lire la valeur pointée dans un bloc unsafe.
- Les pointeurs bruts sont de type `*T` (*i32, *f64, etc.) et représente un pointeur du langage C,
il en a les même propriétés, même sur l'arithmétique du pointeur en C (qu'on puisse décaler l'addresse pointer en ajoutant un nombre entier au pointeur).


## Tableaux
- `[T; n]`: Un tableau de taille fixe est alloué sur la pile (stack) par le compilateur, la taille ne peut donc pas être modifiée contrairement à un vecteur qui a une taille variable
- `Vec<T>`: Un vecteur est une liste de taille variable, l'allocation se fait dynamique au runtime sur le tas (heap)
- `&[T]`: Une tranche (slice) est un pointeur sur un morceau de tableau, c'est pour cela que c'est déclaré comme une référence sur un tableau sans taile car la taille n'est pas connue à la compilation
- Tous les type de tableaux ont la méthode `len()`
- Lors de la récupération d'une valeur, l'index est verifié, on ne peut pas accéder à un tableau avec un index en dehors de ses limites
- Un index doit être du type `usize`
- Une initialisation du tableau peut se faire de 2 façons:
    - `[1, 2, 100]` un tablau de type `[i32; 3]`
    - `[10u16; 5]` un tableau de type `[u16; 5]` équivalent à l'initialisation `[10u16, 10u16, 10u16, 10u16, 10u16]`
- Comme pour n'importe quelle déclaration, toutes les valeurs d'un tableau doit être initialisées:
    - `let x: [u8; 5] = [1, 2, 3, 4, 5];` est autorisé
    - `let x: [u8; 5] = [0; 5];` est autorisé
    - `let x: [u8; 5] = [0, 5];` est interdit, il manque à initialiser les 3 dernières valeurs, seule 2 valeurs sur 5 sont initialisées

## Textes (Strings)
- **Page 57/58**: séquence d'échappement
- La chaine standart peut contenir les séquences d'échappement:
    - Codes ASCII: `\x00`
    - Codes Unicode: `\U{000000}`
- Une chaine peut être définie sur plusieurs lignes:
```rust
let s = "Bonjour Monsieur,
je voudrais vous demanders cela,
Merci.";
```

Un backslash à la fin de la ligne évide d'ajouter un retour la ligne «\n» dans la chaine:
```rust
let s1 = "A
B
C";

let s2 = "A\
B\
C";

assert_eq!(s1, "A\nB\nC");
assert_eq!(s2, "ABC");
```
- Définition d'une chaine brute: `r""`
La chaine brute désactive l'interprétation des séquences d'échappement comme de retour à la ligne `\n`: `assert_eq(r"\", "\\")`
Les chaines brutes sont utiles pour les expressions régulières:
```rust
let number_pattern = Regex::new(r"\d+(\.\d+)*");
```
et pour les chemins de fichiers Windows:
```rust
let filename = r"C:\Program Files\Gorillas";
```
- Avec les chaines brutes, le seul moyen de mettre des guillements dans la chaine est de mettre un délimiteur composé d'un certain nombre de caractères dièse (`#`) avant le guillemet de début
et avec le même nombre de dièses après le guillemet de fermeture: `r#"texte: "bonjour"."#`


## Tuples
- Les parenthèses sont obligatoires, ce qui permet d'éviter la confusion lors des appels de fonctions: `f((1, "Voilà"))` à ne pas confondre avec `f(1, "Voilà")` qui change la signature de la fonction
- Peut avoir une virgule à la fin, c'est même conseillé de la mettre même si en pratique ce n'est jamais fait car cela permet d'ajouter des éléments par copier-coller
- La virgule supplémentaire est nécessaire pour les tuples de taille 1:
    - `(23,)` est un tuple de taille 1 et son type est `(i32,)`
    - `(23)` est un entier encadré de parenthèses et son type est `i32`
- Le type unité est un tuple vide: `()`, sa valeur s'écrit aussi `()`
- La type unité équivaut au type `void` de Java, C/C++ mais à la différence que le type unité en Rust a aussi une valeur
contrairement au type `void` des autres language qui lui n'a pas de valeur et donc pas de littéral pour une valeur qui serait dans ce type `void`


## Erreurs
- ***TODO:*** faire une macro try/catch
- Le type `Option<T>` est vraiment utile et synthétique pour vérifier une valeur vide en retour d'une opération.
Par exemple C, il faut passer un pointeur en paramètre de la fonction et vérifier la valeur de retour, il y a donc 2 informations:
```C
bool popValue(List *lst, int *resultValue) {
    int size = getListSize(lst);

    if ( size == 0 ) {
        return false;
    }

    // Valeur retournée
    *resultValue = getListIntValue(lst, size - 1);
    removeListValueAt(lst, size - 1);

    return true;
}

int showInfo(Table *t, int defaultValue) {
    // Obligation de déclarer une variable pour recevoir le résultat
    int resultat;

    // Il faut vérifier l'existence
    if getValue(t, "info", &resultat)
        printf("Found Info: %d\n", resultat);
    else 
        resultat = defaultValue;

    return resultat;
}
```

En rust, le même programme est plus simple car la même valeur va simplement être interprétée de 2 façons suivant que la liste est vide ou non:
```rust
fn popValue(&mut lst: &mut Vec<i64>) -> Option<i64> {
    let size = lst.len();

    if size == 0 {
        return None;
    }

    Ok(lst.remove(size - 1))
}

fn showInfo(&mut lst: &mut Vec<i64>, defaultValue: i64) -> i64 {
    match popValue(lst) {
        Ok(resultat) => {
            println!("Found Info: {}", resultat);

            resultat
        },
        None => defaultValue
    }
}
```

# Macros
***TODO:*** Importation et accès aux fonctions publiées -> ok
Mais pour les macros -> comment ?


# Blocs
On peut utiliser un bloc pour limiter la durée de vie d'une variable:
```rust
use std::fs::File;
use std::io::Write;

fn main() {
    let filename = "app.txt";
    
    {
        let f = File::create(filename);

        writeln!(&f, "Salut !").unwrap();
        writeln!(&f, "Gars !").unwrap();
        // Ici le fichier est fermé et la variable f est supprimée
    }

    // Comme le fichier est fermé, on a plus besoin de la variable `f`
}
```
Cela éviter de poluer le code avec des variables qu'on utilise plus.


# Fonctions
## Fonction sans environnement
Une fonction sans environnement (ou plus simplement, une fonction) doit obligatoirement spécifier les types dans sa signature
```rust
fn my_inc(i: i32) -> i32 {
    if i < 0 {
        return (-i) + 1;
    }
    
    i + 1
}
```

La dernière valeur est une valeur retournée, donc la fonction précédente est equivalente à:
```rust
fn my_inc(i: i32) -> i32 {
    if i < 0 {
        return (-i) + 1;
    }
    
    return i + 1; // Ici, l'instruction `return` est déconseillée, la valeur seule en dernier est préférée
}
```

L'instruction `return` est obligatoire pour les ruptures du flow d'exécution, mais il faut éviter son utilisation autant que possible pour améliorer la lisibilité du code.
Ainsi, la version suivante de la fonction est préférable:
```rust
fn my_inc(i: i32) -> i32 {
    if i < 0 {
        (-i) + 1
    } else { 
        i + 1
    }
}
```

On peut encore factoriser, mais cette version rend le code beaucoup plus syntétique mais moins souple:
```rust
fn my_inc(i: i32) -> i32 {
    (if i < 0 {-i} else {i}) + 1
}
```

Cette version a pourtant l'avantage d'être très lisible.

Les paramètres peuvent être modifiable en ajoutant un `mut`:
```rust
fn my_inc(mut i: i32) -> i32 {
    if (i < 0) {
        i = -i;
    }

    i + 1
}
```

Ce qui évite aussi d'utiliser une instruction `return`.

Attention, le paramètre n'étant pas une référence mutable (`&'mut`), la modification ne sera pas conservée après l'appel de la fonction:
```rust
let x = -3;
let r = my_inc(x);

assert_eq(x, -3);
assert_eq(r, 4);
```

Ceci serait différent si la fonction était définie ainsi:
```rust
fn my_inc_mut(i: &mut i32) -> i32 {
    if (i < 0) {
        *i = -*i;
    }

    *i + 1
}
```

Ici la valeur accepte un passage par référence et donc la valeur sera modifiée après l'appel:
```rust
let mut x = -3;
let r = my_inc_mut(&mut x);

assert!(x == 3);
assert!(r == 4);
```

Pour terminer sur cette section, il est utile de dire que tout ce qui est vrai pour ce genre de fonction est vrai pour une méthode,
car une méthode est une fonction sans environnement comme par exemple la méthode `Vec::push`.


## Elements imbriqués dans une fonction
Quasiment tout élémént qu'on trouve au premier d'imbrication d'un fichier peut être inclus dans une fonction:
```rust
fn nest_all() {
    use std::collections::HashSet;
    
    struct X(HashSet<i32>);
    
    impl X {
        fn new() -> Self {
            let mut s = HashSet::new();
            
            s.insert(123);
            Self(s)
        }
    }
    
    fn do_nested() {
        let r = X::new();
        
        println!("Val = {:?}", r.0);
    }
    
    do_nested();
}
```

## Type "fonction"
Le nom de la fonction sert de variable, une fonction est une valeur de type `fn`, la fonction `my_inc` est type `fn(i32) -> i32`, le type est quelque sorte sa signature.
Ainsi une variable peut recevoir une fonction comme valeur:
```rust
let a_func = my_inc;
```

De même, une fonction peut être passée en paramètre:
```rust
fn call_func(f: fn(i32) -> i32) {
    println!("Result of Call: {}", f(1234));
}
```


## Fonctions génériques
Comme tout type, une fonction peut avoir des types en paramètre par la définition de fonctions génériques.
```rust
use std::ops::Add;

fn my_generic_add<T: Add<Output=T>>(a: T, b: T) -> T {
    a + b
}
```

Ici c'est la définition d'une fonction générique qui permet de faire l'addition de 2 valeurs pour autant que ces valeurs implémentent l'addition.
En effet, on demande à la fonction d'accepter uniquement des valeurs qui implementent le trait `std::ops::Add<Output=T>`. C'est ce trait qui est d'ailleurs utilisé pour permettre à une valeur d'utiliser l'operateur `+`.
C'est pour cela qu'on peut utiliser l'opérateur `+` (avec `a + b`).
Sans cette restriction, on aurrait pas pu utiliser cette expression `a + b` car le compilateur Rust va nous obliger à utiliser ce genre de variable pour pouvoir utiliser cet opérateur.

Rust va inférer les types ainsi on peut appeler la fonction ainsi:
```rust
let v1 = my_generic_add(1, 2); // T == i32
let v2 = my_generic_add(1.5, 2.5); // T == f64
```

Mais on peut spécifier le type:
```rust
let v1 = my_generic_add::<usize>(1, 2); // T == usize
```

Le nom de la fonction est en fait: `my_generic_add::<T>`.


Cette version est, somme toute pas trop générique car elle ne peut accepter que des valeurs d'un même type pour une valeur retounée aussi du même type:
```rust
let ex1 = my_generic_add(1, 2); // 2 entiers i32 et retourne un entier i32
let ex2 = my_generic_add(1.5, 2.5); // 2 flottants f64 et retourne un flottant 1234_f64
let ex3 = my_generic_add(1, 2.5); // Erreur, les 2 arguments sont de types différents
```

Rust est à ce niveau, suffisament riche pour définir une fonction encore plus générique:
```rust
use std::ops::Add;

fn my_hyper_generic_add<R, B, A: Add<B, Output=R>>(a: A, b: B) -> R {
    a + b
}
```

Ici, la fonction `my_hyper_generic_add` peut potentionnellement accepter 2 types différents comme argument et avec une addition qui retourne un 3ème type. Mais ceci ne fonctionne pas.

Le compilateur indique alors `no implementation for '{integer} + {float}'`. Effectivent le problème est que le type entier (i32) n'implémente qu'une addition dont les 2 opérandes sont du même type.
Pour que cela soit exploitable, il faudrait un type qui implemente ce genre d'addition hétérogène comme ceci:
```rust
struct X(i32);

impl Add<f64> for X {
    type Output = u16;

    fn add(self, rhs: f64) -> u16 {
        (self.0 as u16) + (rhs as u16)
    }
}

let ex4 = my_hyper_generic_add(X(1), 2.5); // Cela fonctionne et la variable `ex4` est de type u16
```

Malgré cela, si on l'appelle avec des arguments de même type, cela fonctionne car cette version est tout de même moins restrictive:
```rust
let ex5 = my_hyper_generic_add(1, 2); // 2 entiers i32 et retourne un entier i32
```

Simplement, les type `A`, `B` et `R` auront le même type.
Pour régler le problème avec les type primitifs, on peut "légèrement" changer la fonction pour intégrer les conversions de types comme cela:
```rust
use num_traits::AsPrimitive;

fn my_hyper_generic_add<R: Copy+'static, A: Add<Output=A>+AsPrimitive<R>+Copy+'static, B: AsPrimitive<A>>(a: A, b: B) -> R {
    (a + b.as_()).as_()
}

let ex6: u16 = my_hyper_generic_add(1, 2.5); // Cela fonctionne, maintenant !
```

Cela fonctionne car même on rend homogène les types sur l'addition en faisant des conversions de type sur les valeurs.
Par contre l'entête de la fonction devient horrible. Il y a un moyen de la réécrire pour la rendre un peu plus lisible comme cela:
```rust
fn my_hyper_generic_add<A, B, R>(a: A, b: B) -> R
    where R: Copy + 'static,
          A: Add<Output=A> + AsPrimitive<R> + Copy + 'static,
          B: AsPrimitive<A> {
    (a + b.as_()).as_()
}
```

Les clauses `where` sont fréquentes puisque Rust demande beaucoup de description sur les types.
Le système de typage de Rust permet beaucoup de richesse et de flexibilité mais cela implique aussi beaucoup de rigueur.


## Fonction avec environnement: Closures
- Le type à utiliser sur une variable pour les closures est
    - Arguments et type de retour: `Box<dyn Fn({args}) -> {retour}>`
    - Juste les arguments: `Box<dyn Fn({args})>`
    - Juste le type de retour: `Box<dyn Fn() -> {retour}>`
    - Sans argument et retour: `Box<dyn Fn()>`
- Les accollades ne sont pas nécessaires si on ne spécifie pas le type de retour, elle sont obligatoire sinon, même si le corps de la closure ne contient que la valeur retournée.
```rust
let f = |x| x + 1;
```
Sauf si on spécifie le type de retour:
```rust
let f = |x| -> i64 { x + 1 }; // Accolades obligatoires
```

- Les types sont inférés contrairement à une fonction (sans environnement), il n'est pas nécessaire de spécifier les types si le compilateur peut inférer les types:
```rust
(|x| x + 1)(1234_i64);
(|x: i64| -> i64 { x + 1 })(1234);
(|x: i64| x + 1)(1234);
```

Sinon le compilateur demandera de spécifier le type s'il ne peut pas l'inférer.

Bien sûr, une closure n'est pas du même ordre qu'une `lambda` en Python, une closure peut contenir un bloc d'instruction.
```rust
let local_inc = |x| {
    println!("Param = {}", x);
    
    x + 1
};
```

Ainsi une closure est une fonction un peut spéciale, car elle diffère d'une simple fonction comme on verra plus bas.


### Capture de l'environnement
La principale différence entre une fonction (dite fonction sans environnement) et une closure (qui est justement une fonction avec environnement), c'est qu'une closure peut capturer son environnement.
Cela veut dire que le corps d'une closure va pouvoir utiliser une variable qui en-dehors de sa définition:
```rust
let extern_var = 1000;
let my_add = |x| extern_var + x;

assert!(my_add(200) == 1200);
```

La capture de variable est la terminologie employée dans ce genre de situation car la variable, ou plus exactement sa possession va être transférée dans l'environnement d'exécution de la closure (environnement lors de l'appel).


### Transfert de possession
Sur l'exemple précédent tout ce passe bien car la variable `extern_var` a un type primitif, ici elle est de type `i32`. Et ces types implementent le trait `Copy`.
Mais en réalité, la possession de la valeur doit être transférée de la variable externe `extern_var` dans la closure.
Ici, le compilateur n'est pas content:
```rust
let extern_var = "Prefix:".to_string();
let my_concat = |x| extern_var + x;

println!("External variable: {}", extern_var); // Erreur ici
assert!(my_concat(200) == 1200);
```

Le `println!()` utilise la variable `extern_var` alors que la possession a été tranférée dans la closure, même si la closure est utilisée après le `println!()`, la possession a bien été transférée au moment de la définition de la closure.
Pour palier au problème, il suffit de copier la valeur soit avant la définition de la closure:
```rust
let extern_var = "Prefix:".to_string();
let var_copy = extern_var.clone();
let my_concat = |x| extern_var.clone() + x;

println!("External variable: {}", var_copy); // Ici, il n'y a plus d'erreur
assert!(my_concat(200) == 1200);
```

Soit dans la définition de la closure:
```rust
let extern_var = "Prefix:".to_string();
let my_concat = |x| extern_var.clone() + x;

println!("External variable: {}", extern_var); // Il n'y plus d'erreur non plus
assert!(my_concat(200) == 1200);
```

En fait, la méthode `clone` dans la closure, va utiliser une référence sur la variable `extern_var`. La possession n'est alors plus du tout transférée, il y a un emprunt. Ce qui évite aussi le problème de compilation.
On peut donc éviter le transfert de possession via une référence, car au lieu du transfert de possession, il y a emprunt;
```rust
let extern_var = "Fred".to_string();
let my_eq = |x| (&extern_var) == x;

println!("External variable: {}", extern_var);
println!("Is equals: {}", my_eq("Fred"));
```


## Type "fonction" et closures
Une closure a un type particulier. Par exemple, pour la précédente closure la variable `my_eq` a un type assez étrange: `[fn(String) -> String; String]`.
Le dernier type `String` est en fait le type de la variable externe `extern_var`. En fait, le compilateur va contruire un type spécial pour chaque closure qui embarque l'environnement externe.
D'ailleurs le compilateur, n'affiche pas réellement cela, il affiche plûtot ce genre d'informations comme son type closure: `[closure@src/main.rs:11:14: 11:61 extern_var:_]`.

Autant dire que ce genre de type ne peut pas être spécifié ou déclaré, d'ailleurs aucune construction du language Rust ne le permet. Et chaque closure a son type, vous ne pourrez faire un truc du genre:
```rust
let extern_var = "Fred".to_string();
let a = |x| (&extern_var) == x;
let mut b = |x| (&extern_var) == x;

b = a;
```

Même si les 2 closures font la même chose et sont écrites de manière exactement identique, le type de la variable `a` est pourtant différent du type de la variable `b` et ceci car en fait les 2 closures sont différente.
C'est un peut comme le type embarquait aussi la position de la déclaration. Et c'est vraiment le cas, cela explique pourquoi le compilateur nous affiche cette bizarrerie comme type de la closure.

Il est donc impossible de déclarer une variable avec un type closure. Il n'est même pas possible d'assigner une closure à une variable avec un type, aucun type ne pouvant décrire une closure
Mais une closure étant une fonction, il existe un moyen de déclarer une variable avec un type pouvant recevoir une closure mais uniquement en paramètre ou en retour d'autres fonctions:
```rust
// Cette fonction accepte une fonction de signature `fn(String) -> String`
// donc elle accepte aussi les closures avec cette signatue
fn do_something_on_string<F: Fn(&str) -> String>(f: F) {
    println!("The resulted string: `{}`", f("Fred")); 
}

fn to_upper(s: &str) -> String {
    s.to_uppercase()
}

fn main() {
    do_something_on_string(to_upper);
    do_something_on_string(|x| "Prefix/".to_string() + x);
}
```

En fait, ce type `Fn` est un trait d'où l'écriture de la signature de la fonction `do_something_on_string`.
Chaque type créé par le compilateur pour chaque closure va implémenter ce trait `Fn`. De même, une fonction (sans environnement) implémente ce trait `Fn`.
Un peu comme si l'implémentation était définie ainsi:
```rust
impl Fn(String) for fn(String) {}

impl Fn(String) -> String for fn(String) -> String {}
```

Même si c'est syntaxiquement juste, bien évidemment, c'est le compilateur que fait cette implémentation en interne, car il ne peut exister un fichier qui implement une infinité de signature.


## Interration d'une closure avec son environnement
### Modification de l'environnement
Heureusement pour nous, les closures ne se limitent pas à lire leur environnement, elles peuvent aussi le modifier.
Mais pour que Rust puisse suivre la gestion des modifications, il faut un nouveau type de closure qui signale que la closure va modifier quelque chose.
Le type `FnMut` va décrire ce genre de closure:
```rust
fn do_something_on_string<F: FnMut(&str) -> String>(mut f: F) {
    println!("The resulted string: `{}`", f("Fred")); 
}

let mut registry = vec![];

do_something_on_string(|s| {
    let r = s.to_string();
    
    registry.push(r.clone());
    r
});

println!("The list: {:?}", registry);
```

Il est à noté qu'un paramètre de fonction de type `FnMut` doit être déclaré mutable (`mut`), comme ici le paramètre `f` de la fonction `do_something_on_string` (à la première ligne).
C'est ainsi en Rust, car lorsque la closure est appelée dans la fonction `do_something_on_string` par `f("Fred")`, l'appel est fait par une référence mutable (`&mut FnMut`) au même titre que l'appel de méthode `push` du vecteur (`Vec<String>`).


### Capturer des variables qui seront détruites
Il arrive qu'une valeur de variable qui va capturée dans une closure soit détruite après l'appel de la closure.
Cela arrive car il y a transfert de possession et c'est toujours à la charge du possesseur de faire le ménage.

En fait, ce n'est pas la closure qui va posséder la variable mais le contexte lorsque la closure va être appelée, ce sera donc l'environnement d'éxecution du corps de la closure.
Ce qui fait qu'à chaque appel de la closure, la valeur de la variable capturée sera forcément détruite.
On voit donc le problème une valeur n'est détruite qu'une seule fois, il faut donc que la closure ne soit appelée qu'une seule fois, si elle était appelée une deuxième fois, ce serait un désastre, car on liberera de la mémoire déjà libérée.

Rust, va donc faire en sorte que ce genre de closure qui capture la possession de la valeur d'une variable ne soit exécutée plusieurs fois,
le compilateur va refuser les appels multiples de ce genre de closure, on ne pourra même pas l'appeler dans une boucle.

Ce type de closure a donc un nouveau type, qui dit que la closure doit être appelée qu'une fois car une variable est possédée par la closure.
Ce type est `FnOnce`.

Prenons cet exemple:
```rust
fn show_result<F: FnOnce(&str) -> String>(f: F) {
    println!("Resulted string: {}", f("Fred"))
}

fn main() {
    let prefix = "prefix/".to_string();

    show_result(|s| prefix + s);
}
```

La possession de la valeur de la variable `prefix` dans la fonction `main` a été transféré à la closure. Et lorsqu'elle est exécutée dans la fonction `show_result`, la valeur sera détruite.
C'est pour cela que la closure ne peut être appelée plusieurs fois.


### Hiérarchie des types de fonctions
Les 3 traits de fonctions `Fn`, `FnMut`, et `FnOnce` ont une hiérarchie ce qui va faire que:
- On va pouvoir passer une fonction implémentant `Fn` ou `FnMut` à une fonction accetant un paramètre de type `FnOnce`
- On va pouvoir passer une fonction implémentant `Fn` à une fonction accetant un paramètre de type `FnMut`

Ceci est possible car `FnOnce` a une exécution plus restreinte que `Fn` et `FnMut` puisque une fonction `FnOnce` ne peut obligatoirement être qu'une seule fois (ça ne dérange pas une `Fn` ou `FnMut` d'être exécuté qu'une seule fois).
De même, `FnMut` est plus restreinte que `Fn`
puisque `FnMut` peut modifier son environnement mais comme une fonction `Fn` ne modifie rien elle peut très bien être exécutée dans les conditions de protection offertes par en environnement modifiable.

Contrairement à ce qu'on croit, une lecture seule n'est pas une restriction, c'est la mutation qui va restreindre un traitement car il faut protéger les accès afin d'éviter des modifications simultanées d'une donnée.
C'est pour cela que dans un environnement protégé de modifications simultanées d'une données, peut très bien se voir exécuter un bout de code qui ne modifie rien.

Ainsi:
```rust
fn do_something_readonly<F: Fn(i32)>(f: F) {
    do_something_mut(f);
    do_something_once(f);
}

fn do_something_mut<F: FnMut(i32)>(f: F) {
    do_something_once(f);
}

fn do_something_once<F: FnOnce(i32)>(f: F) {
}
```

On peut noté que dans la fonction `do_something_readonly` le paramètre `f` sera forcément appelé plusieurs fois, mais pas de problème car la fonction `f` peut le faire,
l'appel `do_something_once(f)` dit juste que la fonction `f` ne sera appelé qu'une seule fois à cause de son paramètre `FnOnce`, ainsi on peut très bien avoir ceci:
```rust
fn do_something_readonly<F: Fn(i32)>(f: F) {
    do_something_once(f);
    do_something_once(f);
    do_something_once(f);
}
```

Par contre ceci serait interdit:
```rust
fn do_other<F: FnOnce(i32)>(f: F) {
    do_something_once(f);
    do_something_once(f); // Interdit car la fonction `f` a déjà été appelé
    do_something_once(f); // Pareil ici
}
```

Car ce coup-ci, le paramètre est une fonction `FnOnce`.
La vérification est faite par le compilateur et non pas à l'éxecution et puisque cette vérification n'est pas faire au runtime, le code compilé est optimal.

En résumant cette hiérarchie, c'est comme si les traits étaient définis ainsi:
```rust
trait FnOnce(i32): FnMut(i32) {}

trait FnMut(i32): Fn(i32) {}

// Pour les fonctions sans environnement, elles implementent le trait `Fn`
impl Fn(i32) for fn(i32) {}
```

Attention, les types `i32` indiqués ici sont à titre d'exemple et pour simplifier car il existe une infinité de signature de fonction, il faut voir plus cela en terme générique (`Fn(A, B, C, ...) -> R`).

On peut alors dire que `FnOnce` est une spécialisation de `FnMut`, reformulé, `FnMut` est parent de `FnOnce`,
et donc qu'une fonction `FnMut` est une fonction `FnOnce`, ce qui fait qu'une fonction `FnMut` peut être appelée comme une fonction `FnOnce`, d'où l'appel de `f` dans `do_something_mut`.

De même dire aussi que `FnMut` est une spécialisation de `Fn`, reformulé `Fn` est parent de `FnMut`,
et donc qu'une fonction `Fn` est une fonction `FnMut`, ce qui fait qu'une fonction `Fn` peut être appelée comme une fonction `FnMut`, d'où l'appel de `f` dans `do_something_readonly`.

Quant aux fonction sans environnement, les fonctions de type `fn`, elle implementent le trait `Fn` puisque dans environnement, elle sont des fonction dont l'appel ne modifient pas leur environnement.

Voici un diagramme, qui résume cette hiérarchie.
[![Hiérarchie des types de fonction](https://github.com/corebreaker/rust-book/blob/master/images/rust-book-001.png?raw=true)](https://github.com/corebreaker/rust-book/blob/master/images/rust-book-001.png?raw=true)


## Une fonction imbriquée n'est pas une closure
Même si une closure semble être une fonction imbriquée dans une autre, il y a 2 différence fondamentale.
1. Une fonction (sous-entendu sans environnement), même imbriquée n'enbarque pas l'environnement externe à sa déclaration.
2. 2 fonctions (sans environnement) avec la même signature, même imbriquées, auront le même type `fn`, alors on l'a dire 2 closures auront chacune leur propre type.


# Multithread
- Rust va interdire:
    - que 2 threads modifient une même donnée
    - qu'une valeur qui n'est plus utilisée ou nettoyée ne soit modifiée
Tout ce ceci évite à 100% les conflits de données
- Rust garantit une programmation multithread 100% sûre (safe):
    - Même si la partie qui crée les threads sont dans une lib (comme les framework web), toute les développements qui utilise cette lib ne seront pas obliger à assurer la concurence d'accès aux données.
    Rust le fait naturellement, ce n'est pas à ajouter.
    - Pas de besoin de faire une version thread-safe d'un objet, en Rust tout est naturellement thread-safe.
    En Java à des tableaux associatifs (`java.util.HashMap`) comme Vector qui ne sont pas thread-safe.
    Oracle a dû introduire une version thread-safe pour leur collections, pour les tableaux associatifs il faut utiliser `java.util.concurrent.ConcurrentHashMap`.
    Les 2 versions (thread-safe et classique) co-existent dans la bibliothèque standard de Java.
    En Rust, il est impossible de développer quelque chose qui n'est pas thead-safe.
    

# Méthode
- Les appels de méthodes sur les littéraux doivent être typés car la même méthode aura un traitement différent en fonction du type:
    - `2.pow(4)` est interdit, il faut préciser le type `2i16.pow(4)` car il faut indiquer que l'opération doit se faire sur des entiers différente de `2f64.pow(4)` qui s'effectuée sur des flottant, l'opération est donc bien différente
    - Cela évite aussi une telle écriture: `2.5.pow(4)`, 4 pourrait compris comme un champ de l'objet `2` 
- Les appels de méthodes sont prioritaires devant les opérateurs donc:
    - `-2u16.abs()` vaut `-2u16`, il faut écrire `(-2u16).abs()` pour valoir `2u16`


# Fonction main
fn main() -> Result<T, E> {
}

>>>>>>>
apachebench
temps accès base de données 5ms
2000000-5000000 connexion > 
  - ecriture de fichier
  - while 1000 itérations
> état de mémoire
> limite écroulement
