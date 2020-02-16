# RUST
- Surcharge d'opérateur
- Programmation système bas niveaux -> peu développer des drivers de périphériques
- Code source du livre: https://github.com/ProgrammingRust
- Le trait `Drop` ne définit pas le comportement de libération de la mémoire, mais celui de l'abandon de possession, l'objet n'a plus de possesseur lorsque la méthode `drop` est appelée

# Install
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

# Mut
Un argument `mut` (mutable) ne veut pas dire que la valeur va modifiée après l'appel de la fonction, ce n'est pas un passage par référence
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

Le code 1 compile mais le code 2 compile aussi mais un warning disant que la variable `x` n'a pas besoin d'être mutable ce qui prouve bien qu'elle n'est pas modifiée.

# Compile
- `assert_debug` peut permettre de faire des test d'ingérité en phase de dev et qui ne seront pas inclut dans la release en production
- La commande `cargo` se lance dans n'importe quel sous-répertoire du projet
- `cargo build --release`

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
| Type              | Description               | Exemples                                                           |
|:-----------------:|:-------------------------:|--------------------------------------------------------------------|
| i8, i16, i32, i64 | Entier signé              | -120, 1000, 1_000, 0x10, -0o12, 0b10, 0b_0111_1111, -1i8, 10_i16   |
| u8, u16, u32, u64 | Entier non-signé          | b'X', 5, 2_000, 0x11, 0o10, 0b10, 0b_1111_1111_1111, 5u16, 501_u32 |
| isize, usize      | Mot machine               | 1, 10usize, 1234_isize                                             |
| bool              | Booléen                   | true, false                                                        |
| f32, f64          | Flottant                  | 1., 1e-23, 3.5f32, 1234_f64                                        |
| char              | Caractère                 | 'A', '\x22', '\u{00012c}'                                          |
| &str              | Chaine à taille fixe      | "Hello Alain", "Yes \x32 \t \u{3f}"                                |
| String            | Chaine à taille variable  | "Hello Alain".to_string(), "Yes \x32 \t \u{3f}".to_string()        |
| enum              | Énumération               | enum WeekEnd { Saturday, Sunday }, enum Wait { OnTime, Late(u32) } |
| &T                | Référence partagée        | let z: &i64 = &x;                                                  |
| &mut T            | Référence mutable         | let &mut x                                                         |
| Box<T>            | Boite (pointeur sur heap) | Box::new(1_i32)                                                    |
| [T; n]            | Tableau (Array)           |                                                                    |

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

# Cast
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

# Tuples
- Les parenthèses sont obligatoires, ce qui permet d'éviter la confusion lors des appels de fonctions: `f((1, "Voilà"))` à ne pas confondre avec `f(1, "Voilà")` qui change la signature de la fonction
- Peut avoir une virgule à la fin, c'est même conseillé de la mettre même si en pratique ce n'est jamais fait car cela permet d'ajouter des éléments par copier-coller
- La virgule supplémentaire est nécessaire pour les tuples de taille 1:
    - `(23,)` est un tuple de taille 1 et son type est `(i32,)`
    - `(23)` est un entier encadré de parenthèses et son type est `i32`
- Le type unité est un tuple vide: `()`, sa valeur s'écrit aussi `()`
- La type unité équivaut au type `void` de Java, C/C++ mais à la différence que le type unité en Rust a aussi une valeur
contrairement au type `void` des autres language qui lui n'a pas de valeur et donc pas de littéral pour une valeur qui serait dans ce type `void`
    
# Erreurs
- ***TODO:*** faire une macro try/catch

# Documentation
- Doc en local `rustup doc`

# Macros
***TODO:*** Importation et accès aux fonctions publiées -> ok
Mais pour les macros -> comment ?

# Closures
- Le type à utiliser sur une variable pour les closures est
    - Arguments et type de retour: `Box<dyn Fn({args}) -> {retour}>`
    - Juste les arguments: `Box<dyn Fn({args})>`
    - Juste le type de retour: `Box<dyn Fn() -> {retour}>`
    - Sans argument et retour: `Box<dyn Fn()>`
- Les accollades ne sont pas nécessaires
```rust
let f = |x| x + 1;
```
Sauf si on spécifie le type de retour:
```rust
let f = |x| -> i64 { x + 1 }; // Accolades obligatoires
```

- Les types sont inférés contrairement à une fonction:
```rust
(|x| x + 1)(1234_i64);
(|x: i64| -> i64 { x + 1 })(1234);
(|x: i64| x + 1)(1234);
```
# Variables
- Les variables doivent être initialisées

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

# Méthode
- Les appels de méthodes sur les littéraux doivent être typés car la même méthode aura un traitement différent en fonction du type:
    - `2.pow(4)` est interdit, il faut préciser le type `2i16.pow(4)` car il faut indiquer que l'opération doit se faire sur des entiers différente de `2f64.pow(4)` qui s'effectuée sur des flottant, l'opération est donc bien différente
    - Cela évite aussi une telle écriture: `2.5.pow(4)`, 4 pourrait compris comme un champ de l'objet `2` 
- Les appels de méthodes sont prioritaires devant les opérateurs donc:
    - `-2u16.abs()` vaut `-2u16`, il faut écrire `(-2u16).abs()` pour valoir `2u16`
    
# Pointeur
- 3 types de pointeurs:
    - Référence: `&T`
    - Boite (Box): `Box<T>`
    - Pointeur brut: `*T`
- `&x` emprunte une référence sur `x`, la valeur est une adresse en mémoire sur la pile (stack) ou sur le tas (heap), l'adresse qui pointe sur la valeur de `x`
- `&x` est du type `&T` (comme `&i32`, `&Bobo`, etc.)
- `*p` récupère la valeur pointée par le pointeur `p`, qu'il soit une référence, une boite ou un pointeur brut
- Une référence Rust est un pointeur qui ne sera jamais nul, il est impossible d'assigner une valeur nulle à une référence
- Il n'y a pas de référence nulle car il n'existe pas de littéral comme en `NULL` en C, `null` en Java et JS, `None` en Python, ou `nil` en Go
- Si on veut indiquer l'absence de référence (et donc d'objet ou d'élément donné sous forme de référence), il faut utiliser le type `Option<&T>` (**TODO** mettre un exemple)
- L'existence du type `Option<&T>` évite un mauvaise utilisation d'un pointeur nul et donc les `NullPointerException`
- Les pointeurs bruts peuvent être initiasés dans un bloc safe mais il faut obligatoirement lire la valeur pointée dans un bloc unsafe

# Tableaux
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

