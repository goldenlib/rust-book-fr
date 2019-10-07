<!-- TODO bonpatron.com -->

<!--
## To `panic!` or Not to `panic!`
-->

## To `panic!` or Not to `panic!`

<!--
So how do you decide when you should call `panic!` and when you should return
`Result`? When code panics, there’s no way to recover. You could call `panic!`
for any error situation, whether there’s a possible way to recover or not, but
then you’re making the decision on behalf of the code calling your code that a
situation is unrecoverable. When you choose to return a `Result` value, you
give the calling code options rather than making the decision for it. The
calling code could choose to attempt to recover in a way that’s appropriate for
its situation, or it could decide that an `Err` value in this case is
unrecoverable, so it can call `panic!` and turn your recoverable error into an
unrecoverable one. Therefore, returning `Result` is a good default choice when
you’re defining a function that might fail.
-->

Donc, comment décider quand on doit faire un `panic!` et quand nous devons
retourner un `Result` ? Quand un code fait un panic, il n'y a pas de moyen de
continuer l'exécution. Vous pourriez faire appel à `panic!` pour n'importe
quelle situation d'erreur, peux importe s'il est possible de continuer
l'exécution ou non, mais alors vous prenez la décision de tout arrêter à la
place du code appelant. Quand vous choisissez de retourner une valeur `Result`,
vous donnez plus de choix au code appelant plutôt que si vous preniez des
décisions à sa place. Le code appelant peut choisir d'essayer de récupérer
l'erreur de manière appropriée à cette situation, ou il peut décider que dans
ce cas une valeur `Err` est irrécupérable, donc utiliser `panic!` et changer
votre erreur récupérable en erreur irrécupérable. Ainsi, retourner `Result` est
un bon choix par défaut quand vous construisez une fonction qui peut échouer.

<!--
In rare situations, it’s more appropriate to write code that panics instead of
returning a `Result`. Let’s explore why it’s appropriate to panic in examples,
prototype code, and tests. Then we’ll discuss situations in which the compiler
can’t tell that failure is impossible, but you as a human can. The chapter will
conclude with some general guidelines on how to decide whether to panic in
library code.
-->

Dans quelques situations, c'est plus approprié d'écrire du code qui fait un
panic au lieu de retourner un `Result`, mais elles sont moins fréquentes.
Voyons pourquoi il est approprié pourquoi il est plus approprié de faire un
panic dans des exemples, des prototypes, et des tests; ensuite des situations
où vous savez qu'en tant qu'humain qu'une méthode ne peut pas échouer, mais que
le compilateur n'a pas de raison de le faire; et nous allons conclure par
quelques lignes conductrices générales sur comment décider s'il faut paniquer
dans le code des librairies.

<!--
### Examples, Prototype Code, and Tests
-->

### Les exemples, prototypes, et les tests sont les endroits parfaits pour Panic

<!--
When you’re writing an example to illustrate some concept, having robust
error-handling code in the example as well can make the example less clear. In
examples, it’s understood that a call to a method like `unwrap` that could
panic is meant as a placeholder for the way you’d want your application to
handle errors, which can differ based on what the rest of your code is doing.
-->

Quand vous écrivez un exemple pour illustrer un concept, avoir un code de
gestion des erreurs très résilient peut rendre l'exemple moins clair. Dans les
exemples, il est courant d'utiliser une méthode comme `unwrap` qui peut faire
un `panic!` qui remplace le code de gestion de l'erreur que vous utiliseriez
dans votre application, qui peut changer en fonction de ce que le reste de
votre code va faire.

<!--
Similarly, the `unwrap` and `expect` methods are very handy when prototyping,
before you’re ready to decide how to handle errors. They leave clear markers in
your code for when you’re ready to make your program more robust.
-->

De la même manière, les méthodes `unwrap` et `expect` sont très pratiques pour
coder des prototypes, avant de décider comment gérer les erreurs. Ce sont des
indicateurs claires dans votre code pour plus tard quand vous serez prêt à
rendre votre code plus résilient aux échecs.

<!--
If a method call fails in a test, you’d want the whole test to fail, even if
that method isn’t the functionality under test. Because `panic!` is how a test
is marked as a failure, calling `unwrap` or `expect` is exactly what should
happen.
-->

Si l'appel à une méthode échoue dans un test, nous voulons que tout le test
échoue, même si cette méthode n'est pas la fonctionnalité que nous testons.
Parce que `panic!` est la manière de le marquer un échec, utiliser `unwrap` ou
`expect` est exactement ce qui est nécessaire.

<!--
### Cases in Which You Have More Information Than the Compiler
-->

### Les cas où vous avez plus d'informations que le compilateur

<!--
It would also be appropriate to call `unwrap` when you have some other logic
that ensures the `Result` will have an `Ok` value, but the logic isn’t
something the compiler understands. You’ll still have a `Result` value that you
need to handle: whatever operation you’re calling still has the possibility of
failing in general, even though it’s logically impossible in your particular
situation. If you can ensure by manually inspecting the code that you’ll never
have an `Err` variant, it’s perfectly acceptable to call `unwrap`. Here’s an
example:
-->

Il peut parfois être approprié d'utiliser `unwrap` quand vous avez un code
logique qui garantie que `Result` aura toujours une valeur de type `Ok`, mais
que c'est une logique que le compilateur ne comprend pas. Vous travaillez
toujours une valeur de type `Result` que vous devez gérer : quel que soit
l'instruction que vous appelez, elle a toujours la possibilité d'échouer,
même si dans ce cas particulier, c'est logiquement impossible. Si vous êtes
sûr en inspectant le code manuellement que vous n'allez jamais obtenir une
variante de `Err`, il est tout à fait acceptable d'utiliser `unwrap`. Voici un
exemple :

<!--
```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1".parse().unwrap();
```
-->

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1".parse().unwrap();
```

<!--
We’re creating an `IpAddr` instance by parsing a hardcoded string. We can see
that `127.0.0.1` is a valid IP address, so it’s acceptable to use `unwrap`
here. However, having a hardcoded, valid string doesn’t change the return type
of the `parse` method: we still get a `Result` value, and the compiler will
still make us handle the `Result` as if the `Err` variant is a possibility
because the compiler isn’t smart enough to see that this string is always a
valid IP address. If the IP address string came from a user rather than being
hardcoded into the program and therefore *did* have a possibility of failure,
we’d definitely want to handle the `Result` in a more robust way instead.
-->

Nous créons une instance de `IpAddr` en parsant une chaine de caractères pure.
Nous savons que `127.0.0.1` est une adresse IP valide, donc il est convenable
d'utiliser `unwrap` ici. Toutefois, avoir une chaine de caractères valide,
codée en dur ne change jamais le type de retour de la méthode `parse` : nous
obtenons toujours une valeur de type `Result`, et le compilateur va nous faire
gérer le `Result` comme si la variante `Err` est toujours probable car le
compilateur n'est pas encore suffisamment intelligent pour analyser que cette
chaine de caractères est toujours une adresse IP valide. Si le texte de
l'adresse IP provient de l'utilisateur plutôt que d'être codé en dur dans le
programme, et fait en sorte qu'il y a désormais une possibilité d'erreur, nous
devrions désormais gérer le `Result` de manière plus résiliente.

<!--
### Guidelines for Error Handling
-->

### Recommandations pour gérer les erreurs

<!--
It’s advisable to have your code panic when it’s possible that your code
could end up in a bad state. In this context, a *bad state* is when some
assumption, guarantee, contract, or invariant has been broken, such as when
invalid values, contradictory values, or missing values are passed to your
code—plus one or more of the following:
-->

Il est recommandé de faire un `panic!` dans votre code lorsqu'il s'exécute dans
de mauvaises conditions. Dans ce sens, les mauvaises conditions sont lorsque un
postulat, une garantie, un contrat ou une invariante a été rompue, comme des
valeurs invalides, contradictoires ou manquantes fournis à votre code, par un
ou plusieurs des éléments suivants :

<!--
* The bad state is not something that’s *expected* to happen occasionally.
* Your code after this point needs to rely on not being in this bad state.
* There’s not a good way to encode this information in the types you use.
-->

* Ces mauvaises conditions ne sont pas *prévues* pour surgir de temps en temps.
* Après cette instruction, votre code a besoin de ne pas être dans ces
mauvaises conditions.
* Il n'y a pas de bonne façon de coder ces informations dans les types que vous
utilisez.

<!--
If someone calls your code and passes in values that don’t make sense, the best
choice might be to call `panic!` and alert the person using your library to the
bug in their code so they can fix it during development. Similarly, `panic!` is
often appropriate if you’re calling external code that is out of your control
and it returns an invalid state that you have no way of fixing.
-->

Si quelqu'un utilise votre code et lui fournit des valeurs qui n'ont pas de
sens, la meilleure des choses à faire et de faire un `panic!` et avertir
le développeur de votre librairie du bogue dans leur code afin qu'il le résoud
pendant la phase de développement. De la même manière, `panic!` est parfois
approprié si vous appelez un code externe dont vous n'avez pas la main dessus,
et qu'il retourne de mauvaises conditions que vous ne pouvez pas corriger.

<!--
However, when failure is expected, it’s more appropriate to return a `Result`
than to make a `panic!` call. Examples include a parser being given malformed
data or an HTTP request returning a status that indicates you have hit a rate
limit. In these cases, returning a `Result` indicates that failure is an
expected possibility that the calling code must decide how to handle.
-->

Lorsque les conditions sont mauvaises, mais qui est prévu que cela arrive peu
importe la façon dont vous écrivez votre code, il est plus approprié de
retourner un `Result` plutôt que faire appel à `panic!`. Il peut s'agir par
exemple d'un parseur qui reçoit des données erronées, ou une requête HTTP
qui renvoie un statut qui indique que vous avez atteint une limite de débit.
Dans ce cas, vous devriez que cet échec est une possibilité en retournant un
`Result` pour propager ces mauvaises conditions vers le haut pour que le code
appelant puisse décider quoi faire pour gérer le problème. Faire un `panic!` ne
serait pas la manière la plus appropriée ici pour gérer ces problèmes.

<!--
When your code performs operations on values, your code should verify the
values are valid first and panic if the values aren’t valid. This is mostly for
safety reasons: attempting to operate on invalid data can expose your code to
vulnerabilities. This is the main reason the standard library will call
`panic!` if you attempt an out-of-bounds memory access: trying to access memory
that doesn’t belong to the current data structure is a common security problem.
Functions often have *contracts*: their behavior is only guaranteed if the
inputs meet particular requirements. Panicking when the contract is violated
makes sense because a contract violation always indicates a caller-side bug and
it’s not a kind of error you want the calling code to have to explicitly
handle. In fact, there’s no reasonable way for calling code to recover; the
calling *programmers* need to fix the code. Contracts for a function,
especially when a violation will cause a panic, should be explained in the API
documentation for the function.
-->

Lorsque votre code effectue des opérations sur des valeurs, votre code devrait
d'abord vérifier que ces valeurs sont valides, et faire un `panic!` si les
valeurs ne sont sont pas correctes. C'est pour essentiellement des raisons de
sécurité : tenter de travailler avec des données invalides peut exposer votre
code à des vulnérabilités. C'est la raison principale pour laquelle la
librairie standard va faire un `panic!` si vous essayez d'accéder d'accéder à
la mémoire en dehors des limites : essayer d'accéder à de la mémoire qui n'a
pas de rapport avec la structure des données actuelle est un problème de
sécurité courant. Les fonctions ont parfois des *contrats* : leur comportement
est garanti uniquement si les données d'entrée remplissent des conditions
particulières. Faire un panic quand le contrat est violé est sensé, car une
violation de contrat indique toujours un bogue du côté de l'appelant, et ce
n'est le genre d'erreur que vous voulez que le code appelant gère
explicitement. En fait, il n'y a aucun moyen raisonnable de récupérer le code
d'appel : le *développeur* du code appelant doit corriger le code. Les contrats
de fonction, en particulier quand une violation va faire un panic, doivent être
expliqués dans la documentation de l'API pour la fonction.

<!--
However, having lots of error checks in all of your functions would be verbose
and annoying. Fortunately, you can use Rust’s type system (and thus the type
checking the compiler does) to do many of the checks for you. If your function
has a particular type as a parameter, you can proceed with your code’s logic
knowing that the compiler has already ensured you have a valid value. For
example, if you have a type rather than an `Option`, your program expects to
have *something* rather than *nothing*. Your code then doesn’t have to handle
two cases for the `Some` and `None` variants: it will only have one case for
definitely having a value. Code trying to pass nothing to your function won’t
even compile, so your function doesn’t have to check for that case at runtime.
Another example is using an unsigned integer type such as `u32`, which ensures
the parameter is never negative.
-->

Cependant, avoir beaucoup de vérifications d'erreurs dans toutes vos fonctions
risque d'être verbeux et ennuyeux. Heureusement, vous pouvez utiliser le
système de type de Rust (et donc la vérification de type du compilateur) pour
faire une partie des vérifications à votre place. Si votre fonction un
paramètre d'un type particulier, vous pouvez continuer à écrire votre code en
sachant que le compilateur s'est déjà assuré que vous avez une valeur valide.
Par exemple, si vous avez un type de valeur plutôt qu'un `Option`, votre
programme s'attend d'avoir *autre chose* plutôt que *rien*. Votre code n'a
donc pas à gérer les deux cas de variantes `Some` et `None` : il n'aura qu'un
seul cas pour avoir une valeur. Du code qui essaye de ne rien fournir à votre
fonction ne compilera même pas, donc votre fonction n'a pas besoin de vérifier
ce cas-ci lors de l'exécution. Un autre exemple est d'utiliser un type unsigned
integer comme `u32`, qui garantit que le paramètre n'est jamais négatif.

<!--
### Creating Custom Types for Validation
-->

### Créer des types personnalisés pour la Validation

<!--
Let’s take the idea of using Rust’s type system to ensure we have a valid value
one step further and look at creating a custom type for validation. Recall the
guessing game in Chapter 2 in which our code asked the user to guess a number
between 1 and 100. We never validated that the user’s guess was between those
numbers before checking it against our secret number; we only validated that
the guess was positive. In this case, the consequences were not very dire: our
output of “Too high” or “Too low” would still be correct. But it would be a
useful enhancement to guide the user toward valid guesses and have different
behavior when a user guesses a number that’s out of range versus when a user
types, for example, letters instead.
-->

Allons plus loin dans l'idée d'utiliser le système de types de Rust pour
assurer d'avoir une valeur valide en créant un type personnalisé pour
validation. Souvenez-vous du jeu du plus ou du moins du Chapitre 2 où notre
code demandait à l'utilisateur de deviner un nombre entre 1 et 100. Nous
n'avons jamais validé que le nombre fourni par l'utilisateur était entre ces
nombres avant de le comparer à notre nombre secret; nous avons seulement validé
que le nombre était positif. Dans ce cas, les conséquences ne sont pas très
graves : notre résultat "Too high" ou "Too low" sera toujours correct. Ce
serait une amélioration utile pour guider les suppositions de l'utilisateur
vers des valeurs valides et d'avoir différents comportements quand un
utilisateur propose un nombre en dehors des limites versus quand un utilisateur
renseigne, par exemple, des lettres à la place.

<!--
One way to do this would be to parse the guess as an `i32` instead of only a
`u32` to allow potentially negative numbers, and then add a check for the
number being in range, like so:
-->

Une façon de faire cela serait de faire un parse du nombre renseigné en un
`i32` plutôt que seulement un `u32` pour permettre des potentiels nombres
négatifs, et ensuite vérifier que le nombre est dans la plage autorisée, comme
ceci :

<!--
```rust,ignore
loop {
    // --snip--

    let guess: i32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    if guess < 1 || guess > 100 {
        println!("The secret number will be between 1 and 100.");
        continue;
    }

    match guess.cmp(&secret_number) {
    // --snip--
}
```
-->

```rust,ignore
loop {
    // --snip--

    let guess: i32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    if guess < 1 || guess > 100 {
        println!("The secret number will be between 1 and 100.");
        continue;
    }

    match guess.cmp(&secret_number) {
    // --snip--
}
```

<!--
The `if` expression checks whether our value is out of range, tells the user
about the problem, and calls `continue` to start the next iteration of the loop
and ask for another guess. After the `if` expression, we can proceed with the
comparisons between `guess` and the secret number knowing that `guess` is
between 1 and 100.
-->

L'expression du `if` vérifie si nous valeur est en dehors des limites, et
informe l'utilisateur du problème, et utilise `continue` pour passer à la
prochaine itération de la boucle et demander un nouveau nombre à deviner. Après
l'expression `if`, nous pouvons continuer avec la comparaison entre `guess` et
le nombre secret en sachant que `guess` est entre 1 et 100.

<!--
However, this is not an ideal solution: if it was absolutely critical that the
program only operated on values between 1 and 100, and it had many functions
with this requirement, having a check like this in every function would be
tedious (and might impact performance).
-->

Cependant, ce n'est pas une solution idéale : si c'était absolument critique
que le programme travaille avec des valeurs entre 1 et 100, et qu'il aurait de
nombreuses fonctions qui exige cela, cela pourrait être fastidieux (et cela
impacterait potentiellement la performance) de faire une validation comme
celle-ci dans chaque fonction.

<!--
Instead, we can make a new type and put the validations in a function to create
an instance of the type rather than repeating the validations everywhere. That
way, it’s safe for functions to use the new type in their signatures and
confidently use the values they receive. Listing 9-10 shows one way to define a
`Guess` type that will only create an instance of `Guess` if the `new` function
receives a value between 1 and 100.
-->

A la place, nous pourrions construire un nouveau type et y intégrer les
validations dans une fonction pour créer une instance de ce type plutôt que de
répliquer les validations partout. De cette manière, c'est plus sûr pour les
fonctions d'utiliser le nouveau type dans leurs signatures et d'utiliser avec
confiance les valeurs qu'ils reçoivent. L'entrée 9-9 montre une façon de
définir un type `Guess` qui ne créera une instance de `Guess` uniquement si la
fonction `new` reçoit une valeur entre 1 et 100 :

<!--
```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```
-->

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```

<!--
<span class="caption">Listing 9-10: A `Guess` type that will only continue with
values between 1 and 100</span>
-->

<span class="caption">Entrée 9-9: un type `Guess` qui ne va continuer
uniquement si la valeur est entre 1 et 100</span>

<!--
First, we define a struct named `Guess` that has a field named `value` that
holds an `i32`. This is where the number will be stored.
-->

Premièrement, nous définissons un struct qui s'appelle `Guess` qui a un champ
`value` qui stocke un `u32`. C'est là que le nombre sera stocké.

<!--
Then we implement an associated function named `new` on `Guess` that creates
instances of `Guess` values. The `new` function is defined to have one
parameter named `value` of type `i32` and to return a `Guess`. The code in the
body of the `new` function tests `value` to make sure it’s between 1 and 100.
If `value` doesn’t pass this test, we make a `panic!` call, which will alert
the programmer who is writing the calling code that they have a bug they need
to fix, because creating a `Guess` with a `value` outside this range would
violate the contract that `Guess::new` is relying on. The conditions in which
`Guess::new` might panic should be discussed in its public-facing API
documentation; we’ll cover documentation conventions indicating the possibility
of a `panic!` in the API documentation that you create in Chapter 14. If
`value` does pass the test, we create a new `Guess` with its `value` field set
to the `value` parameter and return the `Guess`.
-->

Ensuite, nous ajoutons une fonction associée `new` sur `Guess` qui crée des
instances de `Guess`. La fonction `new` est conçue pour avoir un paramètre
`value` de type `u32` et de retourner un `Guess`. Le code dans le corps de la
fonction `new` vérifie `value` pour s'assurer que c'est bien entre 1 et 100.
Si `value` échoue à ce test, nous faisons un `panic!`, qui alertera le
développeur qui écrit le code appelant qu'il a un bogue qu'il a besoin de
régler, car créer un `Guess` avec un `value` en dehors de cette plage va violer
le contrat sur lequel `Guess::new` s'appuie. Les conditions dans lesquels
`Guess::new` va faire un panic devrait être explicité dans sa documentation sur
l'API ouverte au public; nous verrons les conventions de documentation pour
indiquer un `panic!` lorsque vous créerez votre documentation d'API au
Chapitre 14. Si `value` réussit le test, nous allons créer un nouveau `Guess`
avec son champ `value` assigné au paramètre `value` et retourner le `Guess`.

<!--
Next, we implement a method named `value` that borrows `self`, doesn’t have any
other parameters, and returns an `i32`. This kind of method is sometimes called
a *getter*, because its purpose is to get some data from its fields and return
it. This public method is necessary because the `value` field of the `Guess`
struct is private. It’s important that the `value` field be private so code
using the `Guess` struct is not allowed to set `value` directly: code outside
the module *must* use the `Guess::new` function to create an instance of
`Guess`, thereby ensuring there’s no way for a `Guess` to have a `value` that
hasn’t been checked by the conditions in the `Guess::new` function.
-->

Ensuite, nous implémentons une méthode `value` qui emprunte `self`, qui n'a
aucun autre paramètre, et retourne un `u32`. C'est un type de méthode qui est
parfois appelé un *getter*, car sa fonction est de récupérer (NdT: get) des
données des champs et de les retourner. Cette méthode publique est nécessaire,
car le champ `value` du struct `Guess` est privé. C'est important que le champ
`value` soit privé pour que le code qui utilise la struct `Guess` ne puisse pas
assigner `value` directement : le code en dehors du module *doit* utiliser la
fonction `Guess::new` pour créer une instance de `Guess`, qui s'assure qu'il
n'y a pas de façon pour un `Guess` d'avoir un `value` qui n'a pas été vérifié
par les fonctions dans la fonction `Guess:new`.

<!--
A function that has a parameter or returns only numbers between 1 and 100 could
then declare in its signature that it takes or returns a `Guess` rather than an
`i32` and wouldn’t need to do any additional checks in its body.
-->

Une fonction qui a un paramètre ou qui retourne des nombres uniquement entre 1
et 100 peut ensuite utiliser dans sa signature qu'elle prend en paramètre ou
retourne un `Guess` plutôt qu'un `u32` et n'aura pas besoin de faire des
vérifications supplémentaires dans son code.

<!--
## Summary
-->

## Résumé

<!--
Rust’s error handling features are designed to help you write more robust code.
The `panic!` macro signals that your program is in a state it can’t handle and
lets you tell the process to stop instead of trying to proceed with invalid or
incorrect values. The `Result` enum uses Rust’s type system to indicate that
operations might fail in a way that your code could recover from. You can use
`Result` to tell code that calls your code that it needs to handle potential
success or failure as well. Using `panic!` and `Result` in the appropriate
situations will make your code more reliable in the face of inevitable problems.
-->

Les fonctionnalités de gestion d'erreurs de Rust sont conçues pour vous aider à
écrire du code plus résilient. La macro `panic!` signale que votre programme
fonctionne dans de mauvaises conditions qu'il ne peut pas gérer et vous permet
de dire au processus de s'arrêter au lieu d'essayer de continuer avec des
valeurs invalides ou incorrectes. Le enum `Result` utilise le système de type
de Rust pour avertir que l'opération peut échouer d'une manière que votre code
pourrait corriger. Vous pouvez utiliser `Result` pour dire au code qui appelle
votre code qu'il a besoin de gérer le résultat et aussi les potentielles
erreurs. Utiliser `panic!` et `Result` de manière appropriée va faire de votre
code plus fiable face à des problèmes inévitables.

<!--
Now that you’ve seen useful ways that the standard library uses generics with
the `Option` and `Result` enums, we’ll talk about how generics work and how you
can use them in your code.
-->

Maintenant que vous avez vu les pratiques utiles que la librairie standard
utilise avec les enums `Option` et `Result`, nous allons voir au chapitre
suivant comment les génériques fonctionnent et comment vous pouvez les utiliser
dans votre code.