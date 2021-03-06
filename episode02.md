# TP : Programmation orientée objet en PHP (Épisode 2)

## Chapitre 1 : Classes de services
### Retirer toutes les fonctions procédurales
Avoir un gros fichier `functions.php` n'est pas idéal pour gérer un projet et rester organisé. Nous allons voir comment utiliser une classe pour stocker tout ça et donner un autre niveau de sophistication à notre application !

Pour commencer, débarassons-nous de la fonction `battle()` dans `functions.php`.

Prenons par exemple `Ship`. C'est une classe qui va simplement contenir des données, c'est à dire les valeurs des attributs d'un objet `Ship`. Donc, un objet `Ship` contient des données, les restitue en les modifiant parfois un peu, mais ne fait pas grand chose de plus.

La première raison de créer des classes est la suivante : nous avons besoin d'un outil qui permette de rester organisés dans nos différentes données. C'est ce que fait `Ship`.

La seconde grosse raison de faire une classe, c'est parce qu'on a besoin de faire des briques logiques. Par exemple, dans `functions.php`, la fonction `battle()` fonctionne bien : on lui donne deux `Ship`, et après quelques calculs et autres, et un peu de logique pour voir comment les différentes forces et vaisseaux agissent entre eux, elle nous retourne le résultat de ces calculs.

Jusque-là, nous sommes habitués à créer des fonctions de ce genre. À partir de maintenant, le secret de l'orienté objet pour des fonctions logiques/métier, c'est simplement de... Ne plus créer de fonctions n'importe où dans le code comme `battle()`, mais de les mettre dans une classe avec une méthode dedans !

Nous souhaitons faire une classe qui contiendra notre fonction `battle()`, et peut-être d'autres choses. On peut du coup la nommer `BattleManager`. Comme d'habitude pour les classes, stockez la dans un fichier à part nommé `BattleManager.php` ! On peut la mettre dans le dossier `/lib`.

```php
// Fichier: /lib/BattleManager.php
class BattleManager {

}
```

Jusque-là, rien de difficile. Maintenant, comment supprimer notre fonction `battle()` de `functions.php` pour la mettre dans la classe ?

Réponse : en... supprimant notre fonction `battle()` de `functions.php` et en la mettant dans la classe. Tout simplement ! N'oubliez pas de mettre `public` devant `function` néanmoins : cela nous permet d'avoir accès à la méthode en dehors de la classe, et non pas qu'à l'intérieur de la classe.

On a donc **supprimé** la fonction `battle()` de `functions.php`, qui est maintenant dans `BattleManager.php` :

```php
class BattleManager {
    /**
     * L'algorithme de combat super complexe !
     *
     * @return array [winning_ship, losing_ship, used_jedi_powers]
     */
    public function battle(Ship $ship1, $ship1Quantity, Ship $ship2, $ship2Quantity)
    {
        // ...
    }
}
```

C'est réellement le seul changement que nous avons à faire. Mettre la fonction dans une classe, lui rajouter `public`. Pour le reste, les méthodes et les fonctions sont la même chose, rien n'est à changer.

Cependant, nous devons tout de même un peu changer `battle.php` qui ne saura plus où trouver la fonction. Dans `battle.php`, changez : 

```php
$outcome = battle($ship1, $ship1Quantity, $ship2, $ship2Quantity);
```

Par : 
```php
$battleManager = new BattleManager();   // On créée un objet BattleManager
$outcome = $battleManager->battle($ship1, $ship1Quantity, $ship2, $ship2Quantity); // Qui a accès à la méthode battle de sa classe !
```

Et **n'oubliez pas** d'importer notre nouvelle classe pour que cela fonctionne ! Ajoutez-la en haut de `battle.php` :
```php
<?php
session_start();
require __DIR__ . '/functions.php';
require __DIR__ . '/lib/BattleManager.php';
```

Comme on l'a vu à l'épisode 1, il existe une manière d'importer nos classes sans devoir les taper à chaque fois ici, c'est l'autoloading. On le verra plus tard.

Testez votre application, normalement les combats devraient fonctionner comme avant.

Vous avez maintenant 2 raisons de créer des classes :
1. Stocker des données de façon claire et descriptive. Une sorte d'array superpuissant avec des attributs bien définis et des méthodes qui nous donnent accès à toutes sortes de fonctionnalités sur nos données. En général, on comme ces classes représentent nos données, les modélisent, on appelle ça un modèle (ou **Model**).

2. Écrire des fonctions pour faire quelque chose dans votre application (ici, une bataille). L'avantage, c'est de pouvoir trier nos fonctions en différentes classes, par exemple, dans une classe `BattleManager`, on peut avoir des fonctions qui font le combat, génèrent une liste d'opposants, tirent un tableau des meilleurs scores...

3. Ou bien on peut même faire un `ScoreManager` qui serait une classe qui gèrerait spécifiquement les scores ! C'est à vous de voir comment vous voulez organiser votre projet. En général, on met dans une classe toutes les fonctions qui se rapportent au nom de la classe (`BattleManager` : fonctions autour d'une bataille, `ScoreManager` : fonctions autour du calcul des scores). Utilisez des noms parlants !

Et comment on s'en sert, de ces classes ?

Comme on l'a vu : on créée un objet depuis la classe (`$battleManager = new BattleManager()`), puis on utilise cet objet pour appeler la fonction. Voyez `BattleManager` comme le rayon "boîte à outils" de Bricorama, et `$battleManager = new BattleManager()` UNE boîte à outils, la votre, dans laquelle vous picorez quelques outils quand il y en a besoin (`$battleManager->battle()` pour l'outil `battle()` !).

Ce deuxième type de classes pleines de fonctions, on appelle ça des **Services**.

## Chapitre 2 : Une armée de classes services
Il nous reste un petit souci : si on regarde bien `BattleManager::battle()` (c'est le petit nom donné à la méthode `battle()` située dans la classe `BattleManager`), on se rend compte qu'il nous reste une fonction procédurale qui est appelée dedans : `usedSpatiodriveBoosters()` (la fonction qui teste si le vaisseau a utilisé son booster Spatiodrive).

Comme on préfèrerait tout mettre dans notre BattleManager, allons-y, déplaçons la fonction de `functions.php` à notre classe de services `BattleManager` !

```php
class BattleManager {
    // ...
    public function usedSpatiodriveBoosters(Ship $ship)
    {
        $spatiodriveBoostersProbability = $ship->getSpatiodriveBooster() / 100;
        // On tire un nombre entre 1 et 100, s'il est inférieur à la probabilité du booster Spatiodrive, alors on retourne true :
        return mt_rand(1, 100) <= ($spatiodriveBoostersProbability * 100);
    }
}
```

Tout simplement : comme pour `battle()`, on copie-colle la fonction, on rajoute `public` devant, on la supprime de `functions.php` et c'est parti.

Attention tout de même, car pour l'instant on a un bug, la fonction n'existe plus (eh oui, c'est une méthode maintenant). Pour corriger cela, il suffit d'aller aux endroits où la fonction est appelée (spoiler: dans `BattleManager::battle`), et remplacer le code suivant :

```php
if (usedSpatiodriveBoosters($ship1)) {
    $ship2Health = 0;
    $ship1UsedSpatiodriveBoosters = true;
    // Si le spatiodrive booster a été utilisé, on break la boucle while
    break;
}
if (usedSpatiodriveBoosters($ship2)) {
    $ship1Health = 0;
    $ship1UsedSpatiodriveBoosters = true;
    // Si le spatiodrive booster a été utilisé, on break la boucle while
    break;
}
```

Par le code suivant :
```php
if ($this->usedSpatiodriveBoosters($ship1)) {
    $ship2Health = 0;
    $ship1UsedSpatiodriveBoosters = true;
    // Si le spatiodrive booster a été utilisé, on break la boucle while
    break;
}
if ($this->usedSpatiodriveBoosters($ship2)) {
    $ship1Health = 0;
    $ship1UsedSpatiodriveBoosters = true;
    // Si le spatiodrive booster a été utilisé, on break la boucle while
    break;
}
```

Et voilà, ça marche !

Mais, pourquoi `$this` ici ? Eh bien parce que `usedSpatiodriveBoosters()` est une fonction de la même classe, et que c'est un objet de la classe, `$battleManager`, qui appelle la méthode. Pour rappel, `$this`, dans une classe, ça représente l'objet lui-même (donc avec les attributs et méthodes de sa classe). Donc pour dire "accède à ta propre méthode qui s'apelle usedSpatiodriveBoosters()", on dit : `$this->usedSpatidoriveBoosters()`.

### Rendre la méthode privée
On se rend compte que cette méthode, `BattleManager::usedSpatiodriveBoosters()`, n'est utilisée que dans la classe elle-même et ne sera jamais utilisée à l'extérieur (en effet, elle ne sert à rien à part dans la méthode `BattleManager::battle()`).

On a vu qu'une classe service, ou une classe bourrée de fonctions, c'était une boîte à outils. Imaginez qu'un autre développeur prenne une copie de cette boîte à outils et, pas très dégourdi, ne sache pas quels outils sont utilisables directement, lesquels sont utilisables qu'en interne à la classe...

Par exemple : un fer à souder. C'est un outil `public`, on s'en sert de partout. Mais l'étain, ce fil qui nous permet de souder, ne nous sert à rien à l'extérieur de la boîte à outils, il ne sert QUE au fer à souder. On peut lui mettre un autocollant `private` pour que le stagiaire bricoleur ne se trompe pas et ne l'utilise pas !

Ici, c'est exactement pareil. `BattleManager::battle()`, c'est notre outil public, on peut s'en servir quand on en a besoin. `BattleManager::usedSpatiodriveBoosters()`, c'est un outil privé, il ne sert qu'à `battle()` et on ne doit pas avoir à s'en servir à l'extérieur de la boîte à outils.

Du coup, passons cette méthode sur `private` !

```php
class BattleManager {
    // ...
    private function usedSpatiodriveBoosters(Ship $ship)
    {
        $spatiodriveBoostersProbability = $ship->getSpatiodriveBooster() / 100;
        // On tire un nombre entre 1 et 100, s'il est inférieur à la probabilité du booster Spatiodrive, alors on retourne true :
        return mt_rand(1, 100) <= ($spatiodriveBoostersProbability * 100);
    }
}
```

Pour le reste, cette fois, rien n'est à changer dans notre code (ouf) !

### Quelques notes

**NOTE :** Quand on déplace des morceaux de code un peu partout, qu'on restructure un projet, qu'on renomme des choses, qu'on créée des classes etc., mais que rien ne change fondamentalement dans la logique de notre application, on appelle ça du **refactoring**, ou **refactoriser** (ou encore **réusiner**, mais personne ne dit ça, soyons sérieux, on est dans l'espace).

**NOTE 2:** Alors, j'ai bien compris cette histoire de private/public, mais pourquoi faire ? Est-ce que ça n'est pas mieux de tout laisser en public, et de ne me servir que de ce dont j'ai besoin, comme ça si jamais j'ai une méthode qui aurait du être private peut me servir à l'extérieur, eh bien je peux m'en servir tout de suite ?

Par exemple, pour ma boîte à outils : si jamais je ne compte pas la prêter. Mon étain, je sais que je ne peux m'en servir qu'avec mon fer à souder, pas besoin de mettre une étiquette pour s'en rappeler. Et puis, si jamais j'ai besoin de m'en servir malgré tout comme un fil de fer par exemple, je pourrais, au moins.

Mauvaise idée ! L'avantage de laisser `private` ce qui doit être privé, c'est que, le jour où vous faites une modification dans une méthode `private` par exemple, vous saurez que les changements n'affecteront que la classe et pas l'extérieur, puisque seule la classe peut utiliser cette méthode. Pour la boîte à outils : vous aviez changé de marque d'étain, et tout a capoté. Si c'est en `private`, vous savez que les dégâts ne seront que dans la boîte à outils. Si ça n'était pas private, vous n'aurez aucune idée de la portée des dégâts, où est-ce que vous aviez utilisé l'étain ailleurs dans le code, etc !

### Un nouveau service, ShipLoader
Pour les plus maniaques d'entre vous, quelque chose doit être en train de vous titiller : il reste UNE fonction dans `functions.php`. Le fichier est nommé au PLURIEL, et il n'y a qu'UNE fonction.

On va y remédier.

Il nous reste `getShips()` dans ce fichier, dans quel classe pourrions-nous l'insérer ?
- Dans `Ship` : bof, car la classe représente un seul Ship à la fois. Et puis c'est un Model : ça modélise un seul vaisseau. Ce serait comme si, imaginons un quartier avec 10 fois le même immeuble. Dans les plans d'architecture d'un immeuble, vous mettiez aussi les données de tout le quartier. Pas très efficace comme rangement !
- Dans `BattleManager`: cette classe ne gère que les batailles. Importer les protagonistes dedans n'est pas très cohérent non plus. Pour notre boîte à outils, c'est comme si j'avais la liste des autres modèles de boîte à outils de Bricorama dans ma boîte à outils. Pas très utile !

On va du coup créer un nouveau service : `ShipLoader` qui contiendra notre fonction `getShips()`.

Comme d'habitude : coupez-collez `getShips()` de `functions.php` et mettez la fonction dans la classe.

```php
// Fichier: /lib/ShipLoader.php
class ShipLoader {
    public function getShips() {

        $ships = [];

        $ship1 = new Ship();
        $ship1->setName("Jedi Starfighter");
        $ship1->setWeaponPower(5);
        $ship1->setSpatiodriveBooster(15);
        $ship1->setStrength(30);
        $ships['starfighter'] = $ship1;

        $ship2 = new Ship();
        $ship2->setName("X-Wing Fighter");
        $ship2->setWeaponPower(2);
        $ship2->setSpatiodriveBooster(2);
        $ship2->setStrength(70);
        $ships['x_wing_fighter'] = $ship2;

        $ship3 = new Ship();
        $ship3->setName("Super Star Destroyer");
        $ship3->setWeaponPower (10);
        $ship3->setSpatiodriveBooster(0);
        $ship3->setStrength(500);
        $ships['super_star_destroyer'] = $ship3;

        $ship4 = new Ship();
        $ship4->setName("RZ1 A-Wing Interceptor");
        $ship4->setWeaponPower(4);
        $ship4->setSpatiodriveBooster(4);
        $ship4->setStrength(50);
        $ships['rz1_a_wing_interceptor'] = $ship4;

        return $ships;
    } 
}
```

### Un peu de nettoyage dans les require
Oulà, il faut qu'on fasse un peu de ménage dans les classes et fichiers que l'on requiert de partout.

Pour commencer, `functions.php` ne me sert qu'à importer toutes mes classes. Le **seul** code présent dans ce fichier est dorénavant :
```php
// Fichier: functions.php
require_once __DIR__.'/lib/Ship.php';
require_once __DIR__.'/lib/BattleManager.php';
require_once __DIR__.'/lib/ShipLoader.php';
```

Ensuite, `index.php` : j'ai besoin de mes classes pour accéder à ma liste de vaisseaux. J'importe `functions.php` qui les possède :
```php
require __DIR__.'/functions.php';
$shipLoader = new ShipLoader();
$ships = $shipLoader->getShips();

// Suite du code...
```

Et idem pour `battle.php` :
```php
<?php
require __DIR__.'/functions.php';

$shipLoader = new ShipLoader();
$ships = $shipLoader->getShips();

// Suite du code...
```

Et voilà, le code est déjà plus propre ! Finalement, `functions.php` ne me sert plus qu'à importer mes classes. D'ailleurs, comme je n'ai pas le droit de déclarer deux fois une même classe, j'ai utilisé des `require_once` : c'est à dire que si jamais le fichier a déjà été importé, et qu'il est de nouveau demandé, on ne l'importera pas.

Testez ! Normalement, tout fonctionne comme avant. Ça y est ! Notre code est complètement codé en POO ! Finies les fonctions, place aux méthodes ! D'ailleurs, pour fêter ça, on va même renommer notre fichier `functions.php` en... `bootstrap.php`. Pensez à renommer également dans les require de  `battle.php` et `index.php` !

**NOTE :** Rien à voir avec la librairie Twitter Bootstrap ! Bootstrap, en anglais, veut dire "amorcer". C'est le fichier d'amorce de notre projet, tout simplement.

D'ailleurs, comme c'est l'amorce du projet et qu'on s'en sert de partout, on peut aussi mettre le `session_start()` à l'intérieur, et le retirer des autres fichiers. Voilà un beau nettoyage de code ! (Enfin... Refactoring).

## Chapitre 3 : Affiner le résultat d'une bataille avec une classe
Le cas le plus fréquent où l'on sait que l'on a besoin d'une classe, c'est lorsque l'on se trouve avec un array de données. En l'occurrence, il nous reste un array à traiter : celui du résultat de la bataille, dans `BattleManager::battle()`.

Rapellez-vous :
1. Dans `BattleManager`, j'ai du code comme ça :
   ```php
   return array(
            'winning_ship' => $winningShip,
            'losing_ship' => $losingShip,
            'used_spatiodrive_boosters' => $usedSpatiodriveBoosters,
        );
    ```
2. Dans `battle.php`, j'ai du code comme :
   ```php
    <p class="text-center">
        <?php if ($outcome['winning_ship'] == null) : ?>
            Les deux opposants se sont détruits lors de leur bataille épique.
        <?php else : ?>
            Le groupe de <?php echo $outcome['winning_ship']->getName(); ?>
            <?php if ($outcome['used_spatiodrive_boosters']) : ?>
                a utilisé son booster Spatiodrive pour détruire ladversaire !
            <?php else : ?>
                a été plus puissant et a détruit le groupe de <?php echo $outcome['losing_ship']->getName() ?>s
            <?php endif; ?>
        <?php endif; ?>
    </p>
    ```
Terrifiant ! Avec les arrays, on n'a aucune assurance des données retournées. Comment être parfaitement certains que notre variable `$outcome` contienne exactement un `winning_ship`, un `losing_ship`...

En utilisant une classe bien sûr !

### Créer la classe modèle BattleResult
Créons un Model (car c'est une représentation de données) dans `/lib`, nommé `BattleResult.php`, contenant la classe `BattleResult`.

Cette classe contiendra le résultat d'une bataille, c'est à dire pour l'instant les données suivantes : `winning_ship`, `losing_ship`, `used_spatiodrive_boosters`.

```php
// Fichier: /lib/BattleResult.php
class BattleResult {
    private $usedSpatiodriveBoosters;
    private $winningShip;
    private $losingShip;
}
```

Jetez un oeil à `Ship.php` et souvenez vous : comment avons-nous fait pour fournir notre classe en données ? En fait, on a utilisé deux stratégies : le **constructeur**, qui nous oblige à insérer des données à la création d'un objet, et les **setters**, plus flexibles.

Ici, utilisons la stratégie du **constructeur** pour **toutes** nos données : en effet, un objet `BattleResult`, donc un résultat de bataille, n'a de sens que si on connaît le vaisseau gagnant, le perdant, et si le booster a été utilisé !

```php
class BattleResult
{
    private $usedSpatiodriveBoosters;
    private $winningShip;
    private $losingShip;
    public function __construct($usedSpatiodriveBoosters, $winningShip, $losingShip)
    {
        $this->usedSpatiodriveBoosters = $usedSpatiodriveBoosters;
        $this->winningShip = $winningShip;
        $this->losingShip = $losingShip;
    }
}
```

Et hop, on a une belle classe `BattleResult` de prête avec de petits résultats de bataille en devenir.

Pour rendre les choses plus propres, on sait à l'avance les types qu'auront les paramètres de notre constructeur. Indiquons-les pour être sûrs de recevoir les bonnes données dans un résultat de bataille !

```php
class BattleResult {
    // ...
    public function __construct(bool $usedSpatiodriveBoosters, Ship $winningShip, Ship $losingShip) {
        // ...
    }
}
```

Maintenant, changeons le reste du code.

Commençons avec `BattleManager`, notre fichier qui retourne un résultat de bataille en array :


Remplacez ce code :
```php
class BattleManager
{
    // ...
    public function battle(Ship $ship1, $ship1Quantity, Ship $ship2, $ship2Quantity)
    {
        // ...
        // Tout en bas de la fonction ...
        return array(
            'winning_ship' => $winningShip,
            'losing_ship' => $losingShip,
            'used_spatiodrive_boosters' => $usedSpatiodriveBoosters,
        );
    }
}
```

Par ce code : 
```php
class BattleManager
{
    // ...
    public function battle(Ship $ship1, $ship1Quantity, Ship $ship2, $ship2Quantity)
    {
        // ...
        // Tout en bas de la fonction ...
        return new BattleResult($usedSpatiodriveBoosters, $winningShip, $losingShip);
    }
}
```

Et n'oubliez pas de rajouter dans `bootstrap.php` notre nouvelle classe : 
```php
// ...
require_once __DIR__.'/lib/BattleResult.php';
```

Enfin, quel fichier se sert de ce `BattleResult` déjà ? `battle.php` bien sûr ! Rapellez-vous cette ligne qui allait nous chercher le résultat de bataille pour le stocker dans une variable `$outcome` :
```php
$outcome = $battleManager->battle($ship1, $ship1Quantity, $ship2, $ship2Quantity);
```

Et pour la réutiliser ainsi :
```php
<?php if ($outcome['winning_ship']) : ?>
    <?php echo $outcome['winning_ship']->getName(); ?>
```

L'idée de tout ce que nous faisons, c'est de se débarasser de cet array.
Avant tout, si on suit la pile d'exécution, voyons ce qu'on devrait trouver dans `$outcome` :
1. `$outcome` est égal à `$battleManager->battle()`
2. `->battle()`, dans `BattleManager`, retourne un `new BattleResult()`
3. Donc, dans `$outcome`, on aura un `new BattleResult()`.
4. Je vérifie ce qu'il y a dans un objet `BattleResult` : c'est un objet qui a les attributs suivants : `usedSpatiodriveBoosters`, `winningShip`, `losingShip`.

Et voilà ! En POO, prenez le temps de prendre du recul sur votre code et remonter petit à petit les étapes d'exécution du code, sinon on se perd vite.

On comprend donc que `$outcome` est un objet de type `BattleResult`, et qu'il possède les attributs `$outcome->usedSpatiodriveBoosters`, `$outcome->winningShip`, `$outcome->losingShip`.


### Créer des getters

Avant toute chose, rapellez-vous que ces attributs sont en private ! Il va falloir créer des getters pour chacun d'eux au sein de `BattleResult` : `getUsedSpatiodriveBoosters()`, `getWinningShip()` et `getLosingShip()`. D'ailleurs, `getUsedSpatiodriveBoosters()` semble un peu moche et long à dire : vous pouvez pourquoi pas créer un getter nommé `wereBoostersUsed()` (qui veut dire "est-ce que les boosters ont été utilisé ?").

Parfait ! Il n'y a plus qu'à remplacer dans `battle.php` les arrays par cet objet `$outcome`. Par exemple : `$outcome['winning_ship']` deviendra `$outcome->getWinningShip()` et ainsi de suite.


### Commenter notre model BattleResult avec PHPDoc

N'oubliez pas de commenter votre code petit à petit, voici un exemple d'un `BattleResult.php` proprement documenté :

```php
class BattleResult
{
    private $usedSpatiodriveBoosters;
    private $winningShip;
    private $losingShip;

    /**
     * @param bool $usedSpatiodriveBoosters
     * @param Ship $winningShip
     * @param Ship $losingShip
     */
    public function __construct(bool $usedSpatiodriveBoosters, Ship $winningShip, Ship $losingShip)
    {
        $this->usedSpatiodriveBoosters = $usedSpatiodriveBoosters;
        $this->winningShip = $winningShip;
        $this->losingShip = $losingShip;
    }

    /**
     * @return Ship
     */
    public function getWinningShip() : Ship {
        return $this->winningShip;
    }

    /**
     * @return Ship
     */
    public function getLosingShip() : Ship {
        return $this->losingShip;
    }

    /**
     * @return bool
     */
    public function whereBoostersUsed() : bool {
        return $this->usedSpatiodriveBoosters;
    }
}
```

## Chapitre 4 : Déclarations de type optionnelles et méthodes sémantiques


## Chapitre 5 : Les objets sont passés par référence
## Chapitre 6 : Récupérer des objets depuis une base de données
## Chapitre 7 : Gérer l'ID d'un objet
## Chapitre 8 : Ne faire qu'une connexion à la BDD avec un attribut
## Chapitre 9 : Best Practice: Configuration centralisée
## Chapitre 10 : Best Practice: Connexion centralisée
## Chapitre 11 : Service Container
## Chapitre 12 : Container: Forcer des objets uniques
## Chapitre 13 : Les containers à la rescouse