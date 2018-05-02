
class: center, middle
# Bug Free, By Design
21 facons de rendre son code résistant aux bugs sans parler de tests!

.footnote[.red.bold[@johan_alps], Developer]

---
class: center

# Bugs...

--

.middle[![Center-aligned image](shit-happens.gif)]

---
class: split-40, center
# Design

--
.column[![Center-aligned image](frustrating-kettle-small.jpg)]

.column[![Center-aligned image](coffeepot-masochists-small-2.jpeg)]


???
Fais plus attention!

---
# Poka Yoke
Améliorer le système - Entonnoir de feedback

--

.left[![funnelFeedback](FunnelOfFeedback_small.png)]

---
class: split-50, middle
# Lorsq'un bug arrive. 
.column[

A quel moment a-t-il été introduit? Pourquoi?

Quels éléments ont favorisé son apparition?

Peut-on ramener la détection plus haut dans l'entonnoir?
]

--
.column[![funnelFeedback](FunnelOfFeedback_small.png)]

---
### Des idées pour l'éviter?

.center[![solutions](solutions.jpg)]


---
# Priorités
1. Rendre les erreurs impossibles
2. Corriger les erreurs
3. Orienter - Documentation

---
class: center
# Rendre les erreurs impossibles

.center[![Center-aligned image](toy.png)]

---
# Rendre les erreurs impossibles
## Construction d'objets

```java
    public Menu() {
        // Can I have only starter and main course?
    }
```
--

```java
    public Menu(String mainCourse, String starter, String dessert) {
        // attention à l'ordre...
    }

```
--

```java
    public Menu(MainCourse mainCourse, Starter starter, Dessert dessert) {
        // oh yeah!
    }
```
--


```javascript
    static starterAndMainCourse(starter, mainCourse) {
        // in languages where you have only one constructor
        // Factory method / Named constructor
    }
``` 


---
# Rendre les erreurs impossibles
- Instantiation en 2 fois
- Couplage temporel
- Primitive Obsession
- Couplage sans cohésion

---
# Couplage temporel


```typescript
    let buggyConnector = new BuggyConnector(port);
    buggyConnector.connect(); // we can forget this
    buggyConnector.putData("hello");
```
--

```typescript    
    new Connector(port)
        .connect() // impossible to forget
        .putData("hello");

```

--

```typescript
	class Connector {
	    connect() {
	        return new OpenConnection()
	    }	
	}
	
	class OpenConnection {
	    putData() { ... }	
	}
```
---

# Couplage temporel encore

Exception?
```javascript
		const ticTacToe = new TicTacToe();
		ticTacToe.occupyX(1, 1);
		ticTacToe.occupyY(0, 1);
		ticTacToe.occupyY(0, 0);
```

--

No error possible
```typescript
		const ticTacToe = new TicTacToe();
		ticTacToe.occupy(1, 1);
		ticTacToe.occupy(0, 1);
		ticTacToe.occupy(0, 0);  
```


---

# Primitive Obsession

```typescript
		ticTacToe.occupy(1, 1);
```

--

```typescript
		ticTacToe.occupy(2, 3); // illegal input?
```

---

# No Primitive Obsession

```typescript
		const ticTacToe = new TicTacToe();
		ticTacToe.occupy(MIDDLE, CENTER);
		ticTacToe.occupy(UPPER, LEFT);
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
        this.assertIsPlayerYTurn('Y');
        this.assertIsInsideLimits(row, column);
        // ...
    }
```

---
# Types avancées
```typescript
    public occupyY(row: number, column: number): any {
        this.assertIsPlayerYTurn('Y');
        this.assertIsInsideLimits(row, column);
        this.assertBoardHasFreeCells(); // <<=====
        // ...
    }
```

---
# Types avancées
On ne peut jouer que 9 fois.
--
.left[![funnelFeedback](TicTacToe_Typed.png)]

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
--
```typescript
class NinthState implements ITicTacToe<EndState> {
    public occupy(row: any, column: any): EndState {
        return new EndState();
    }
}

class EndState  {
}
```

---
class: center, middle
# Couplage sans cohésion
Demo time!

---
background-image: url(trivial.jpg)


---
# Code testable caché

```typescript
		let data = callToDependency()
		//
		// pure logic
		// pure logic
		// pure logic
		// pure logic
		// pure logic
		//
		callToDependency2(transformedData)
```

---
# Code testable caché
```typescript

requestExternalServer().then((key) =>
    persistence.get(key)).then(result => {

        const langToUpdate = {};
        versionsLangs.map((versionLang) => {
            let restPath = versionLang.entity.toRestPath();

            result.dates.langsDates.map(datePayload => {
                if (restPath === datePayload.langRestPath) {
                    langToUpdate[restPath] = langToUpdate[restPath] || {};
                    langToUpdate[restPath].dates = datePayload.payload.dates;
                }
            });

        });
        return persistence.update(key, langToUpdate)
})
```
--
Extract pure function

---
# Code testable caché
```typescript

requestExternalServer().then(() =>
    persistence.get(key)).then(result => {
        let langToUpdate = makeLang(key, result)
        return persistence.update(key, langToUpdate)
})

function makeLang(key, result) {
	const langToUpdate = {};
	versionsLangs.map((versionLang) => {
	    let restPath = versionLang.entity.toRestPath();
	
	    result.dates.langsDates.map(datePayload => {
	        if (restPath === datePayload.langRestPath) {
	            langToUpdate[restPath] = langToUpdate[restPath] || {};
	            langToUpdate[restPath].dates = datePayload.payload.dates;
	        }
	    });
	});
	return langToUpdate;

}
```

---
## Edge-less code
Réduction de charge cognitive. Nombre de cas possibles

- If-less 
	- Option, List, Map/Dictionary, NullObject, Polymorphisme
- lambdas 
	- ex filter
- Eliminer exceptions

## Petites méthodes  
  - Code focalisé, moins de possibilités

## Etat immuable
  - pas de vecteur temps
  - pas de messages implicites

---

# Trop de conditionnels
Git log
* fix suppression
* correction suppression
* fix delete
...
--

```typescript
   private findPokerFaceIndex(srcPokerFaces, updatePockerFace){
        const indexToDelete = [];
        let indexOriginal = 0;
        srcPokerFaces.forEach((pokerFaceOriginal) => {
            let index = 0;
            while (index < updatePockerFace.length 
	            && updatePockerFace[index].alternativeFaceId !==
	             pokerFaceOriginal.alternativeFaceId) {
                index++;
            }
            console.log("@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@",
	            index === srcPokerFaces.length,index , updatePockerFace.length)
            if (index === updatePockerFace.length) {
                console.log("@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@tetst")
                indexToDelete.push(indexOriginal + 1);
            }
            indexOriginal++;
        });
        return indexToDelete;
    }
```

---
# C'est nul!
>  I call it my billion-dollar mistake. It was the invention of the null reference in 1965
>
> -- <cite>Tony Hoare - quick sort inventor</cite>

--

```javascript
  /** 
   * javascript <3 <3 <3
   */
  function nullCestNul(value) {
     if (value == null)
	       return undefined
  }
```

---

# #NoNull

## Types non nullables!
typescript, kotlin, ...


--
## Ailleurs
Et si on ne retournait null de nul part?

---
class: center
# Corriger les erreurs

.top[![Center-aligned image](dont_fix_the_config_file.png)]

--

Evitons de corriger la config. Rendons le code résistant
--

```typescript
Url.join(hostname, path) 
```
--

```typescript
socketHost.appendPath(path)
```

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
.right[![Center-aligned image](toilet_documentation_s.jpg)]

---
layout: true
## Couplage et cohésion...

... Micro service configuration (hell)

---

.top[![argh the diff](config-generator-problem.png)]

---

.top[![argh the diff](config-generator-solution.png)]

---
.top[![argh the diff](diff_of_rename.png)]

---
.top[![argh the diff](exploded_diff.png)]

---
.bottom[![argh the diff](project_layout.png)]

---
layout: false


# Proposition pour demain
1. Au prochain bug: pourquoi? Que peut-on changer?
2. Lire du code sous cet angle
3. Se méfier des primitives 
4. S'exercer sur [BugsZero Kata](https://github.com/martinsson/BugsZero-Kata)

.footnote[.red.bold[@johan_alps    
martinsson.johan@blogspot.com ]]


---

# Une étude de cas

- Configs
- lambdas au lieu de for/while
- Compteur parallèle 



