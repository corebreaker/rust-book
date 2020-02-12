# RUST
- Surcharge d'opérateur
- Programmation système bas niveaux -> peu développer des drivers de périphériques
- Code source du livre: https://github.com/ProgrammingRust
- Le trait `Drop` ne définit pas le comportement de libération de la mémoire, mais celui de l'abandon de possession, l'objet n'a plus de possesseur lorsque la méthode `drop` est appelée

# Install
- Il est conseillé de suivre l'installation par Rustup (https://rustup.rs)
   - TODO: procédure d'installation
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

# Expression
- La méthode `unwrap` d'un résultat (`Result`) peut être utilisées pour pouvoir compiler sans erreur lorsqu'on ne veut pas utilisé le résultat, que ce soit la valeur ou l'errur.
Car Rust nous oblige d'utiliser le résultat, la méthode `wrap` étouffe la valeur de retour, que ce soit une donnée pour un retour de genre `Ok` ou une erreur pour un retour de genre `Err`.
- Tout est expression
Voici l'opérateur ternaire en Java, JS, C, C++:
```java
    int i = x ? y : z;
```

Voici l'équivalent en Rust
```rust
    let i = if x { y } else { z };
```

# Types
- Inférence des types => pas besoin de mettre le type lors de la déclaration

# Erreurs
- TODO: faire une macro try/catch

# Documentation
- Doc en local `rustup doc`

