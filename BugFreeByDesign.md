
class: center, middle
# Bug Free, By Design
A la poursuite des bugs, au delà des tests

.footnote[.red.bold[@johan_alps], Developer]


---
# Design

--
.left[![Center-aligned image](/impossible_design.png)]
--
Fais plus attention! 

---
# Agenda
1. Poka Yoke - améliorer le système
2. Rendre les erreurs impossibles
3. Corriger les erreurs
4. Orienter - Documentation
5. Feedback
6. Exemple en micro services


---
# Poka Yoke
Améliorer le système - Entonnoir de feedback

--

.left[![funnelFeedback](/FunnelOfFeedback_small.png)]

---
class: center, middle
Lorsq'un bug arrive. Quels éléments ont favorisé son apparition?

--

Peut-on y faire quelque chose?

---
class: center
# Rendre les erreurs impossibles

.center[![Center-aligned image](/toy.png)]

---
# Rendre les erreurs impossibles

--

```java
    Menu(){
        /** can I have starter and main course only? */
    }
```
--

```java
    Menu(String mainCourse, String starter, String dessert) {
        // Better but still...
    }

```
--

```java
    Menu(MainCourse mainCourse, Starter starter, Dessert dessert) {
        // oh yeah!
    }

```

---
# Rendre les erreurs impossibles
- Instantiation en 2 fois
- Couplage temporel
- Couplage sans cohésion



---

# Couplage temporel

--

```javascript
describe('TicTacToe', () => {
    it('can blow up!', () => {
        const ticTacToe = new TicTacToe();
        ticTacToe.occupyX(1, 1);
        ticTacToe.occupyY(0, 1);
        ticTacToe.occupyY(0, 0);
    });
});
```

--

```typescript
describe('TicTacToe', () => {
    it("just works!", () => {
        const ticTacToe = new TicTacToe();
        ticTacToe.occupy(1, 1);
        ticTacToe.occupy(0, 1);
        ticTacToe.occupy(0, 0);
    });
});
```

---

# Couplage sans cohésion 

Primitive Obsession

```typescript
ticTacToe.occupy(1, 1);
```

--

```typescript
ticTacToe.occupy(2, 3); // illegal input?
```

---
# Couplage sans cohésion 
No Primitive Obsession

```typescript
describe('TicTacToe', () => {
    it("Look ma, can't put outside board!", () => {
        const ticTacToe = new TicTacToe();
        ticTacToe.occupy(MIDDLE, CENTER);
        ticTacToe.occupy(UPPER, LEFT);
    });
});
```

--

```typescript
enum Row {
    UPPER, MIDDLE, LOWER,
}

enum Column {
    LEFT, CENTER, RIGHT,
}

class TicTacToe {

    public occupy(row: Row, column: Column) {
        // ...
    }
    
```

---
# Assertions remplacés par les types
```typescript

    public occupyY(row: number, column: number): any {
        this.assertIsPlayerYTurn();
        this.assertIsInsideLimits();
        // ...
    }
```

---
# Types avancées
```typescript
    public occupyY(row: number, column: number): any {
        this.assertIsPlayerYTurn();
        this.assertIsInsideLimits();
        this.assertBoardHasFreeCells(); // <<=====
        // ...
    }
```

---
# Types avancées
playing a tenth time shouldn't compile


.left[![funnelFeedback](/TicTacToe_Typed.png)]

---

```typescript
interface ITicTacToe<NextState> {
    occupy(row, column): NextState;
}
```
--
```typescript
class NewGame implements ITicTacToe<SecondState> {
    public occupy(row, column): SecondState {
        return new SecondState();
    }
}

class SecondState implements ITicTacToe<ThirdState> {
    public occupy(row, column): ThirdState {
        return new ThirdState();
    }
}
```

---
# null
> Null reference ... my billion dollar mistake
>
> -- <cite>Tony Hoare</cite>


--

## Javascript

```javascript
  /** 
   * javascript FTW!
   */
  function nullIsCool(value) {
     if (value == null)
	       return undefined
  }

```

--
# D'autres langages font autrement
Non nullable types!

___
Class: center, middle
# More types

Calling non-existing method

---
# More types
- Types : Total code, Maybe/Optional
- Types : Dependent types (types aussi puissantes que le runtime)  => Idris

---
## Edge-less code
Réduction de charge cognitive. Nombre de cas possibles

- No-if
- lambdas
- Eliminer exceptions
- Scoped code
  - Code focalisé, moins de possibilités
- Etat immuable
  - pas de vecteur temps
  - pas de messages implicites

---
# Maybe - Monads

```javascript
    getTripsByUser(user) {
        let tripList = [];
        let loggedUser = UserSession.getLoggedUser();
        if (loggedUser != null) {
            let isFriend = user.getFriends().indexOf(loggedUser) > -1
            if (isFriend) {
                tripList = TripDAO.findTripsByUser(user);
            }
            return tripList;
        } else {
            throw new Error('User not logged in.');
        }
    }
```

--

```javascript
    // Monadic version of the public function, returns an Either
    getTripsByUserEither(user, loggedUser) {
        return Maybe.fromNull(loggedUser)
            .toEither('User not logged in.')
            .flatMap(connectedUser => this.doGetTripsByUser(user, connectedUser));
    }

    // doGetTripsByUser :: user -> user -> Either String (List Trip)
    doGetTripsByUser(user, loggedUser) {
        let isFriend = user.isFriendsWith(loggedUser);
        return isFriend ? this._loadTripsFromDb(user) : noTrips();
    }

```

---
### Back to throwing exception
```javascript

    // Imperative version, calling the monadic one, keeping the contract of throwing
    getTripsByUser(user, loggedUser) {
        let eitherTrips = this.getTripsByUserEither(user, loggedUser);
        return eitherTrips.toImperative(
            failure => { throw new Error(failure) }
            );
    }
```

---
class: center, middle
# Corriger les erreurs

Eviter de corriger la config:

.top[![Center-aligned image](/dont_fix_the_config_file.png)]

--

Rendre le code résistant


---
## Documentation

```java
/** 
 * Lorsque le design est intuitive, la documentation est inutile. 
 */
```

--

Mais parfois la documentation est utile...

---
.right[![Center-aligned image](/toilet_documentation_s.jpg)]

---
## Documentation
> Documentation is hard because of 'cache invalidation and naming things'
>
> -- <cite>Inspired by Phil Carlton</cite>

La documentation a un coût. 

---
layout: true
## Couplage et cohésion...

___

... Micro service configuration (hell)

---
.top[![argh the diff](/diff_of_rename.png)]

---
.top[![argh the diff](/exploded_diff.png)]

---
.top[![example code](/example_of_output_code.png)]

---
.bottom[![argh the diff](/project_layout.png)]

---
layout: false

---
class: center, middle
#Merci!


---
# Feedback
Permettre à l'utilisateur de corriger son erreur.
## Ex: Go to page bug
https://gitlab.bbuzcloud.com/nextgen/beevirtua-front/blob/master/src/beevirtua/ui/book/Book.js
https://gitlab.bbuzcloud.com/nextgen/beevirtua-front/blob/master/src/beevirtua/ui/book/navigation/Navigation.js
https://gitlab.bbuzcloud.com/nextgen/beevirtua-front/blob/master/src/beevirtua/data/utility/search/SearchOccurenceContext.js
