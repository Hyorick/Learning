# Design Pattern

2 familles
* Patterns ***tactiques*** (Strategy, Observer ...)
* Patterns ***architectures*** (Ex: MVC { dérivés: MVP, MVVM}, CQRS, Event Sourcing, Circuit breaker) : il est composé d'un ensemble de pattern tactiques

![Design Pattern intro](images\designPatternIntro.png)

## Patterns tactiques
3 catégories:

>#### **Creation**

- Description de ***la manière dont un objet ou un ensemble
d'objets peuvent être créés, initialisés, et configurés***

- Isolation du code relatif à la création, à l'initialisation afin de
rendre l'application indépendante de ces aspects

- Exemples : Abstract Factory, Factory Method, Builder, Prototype, Singleton

>#### **Structure:** diagramme qui montre la **connexion entre les classes**

- Description de ***la manière dont des objets de l'application doivent etre connectés*** afin de rendre ces connections indépendantes des évolutions futures de l'application

- Exemples : Adapter(objet), Composite, Bridge, Decorator, Facade, Proxy

>#### **Comportement:** diagramme de **sequence, objets**

- Description de ***comportements d'interaction entre objets***
- Gestion des **interactions dynamiques entre des classes et des objets**

- Exemples : Strategy, Observer, Iterator, Mediator, Visitor, State


### **Portée** des design patterns

2 types:

>#### Portée de classe

- Focalisation sur les relations entre les classes et leurs sous-classes

- Réutilisation par Héritage

>#### Portée d'instance (Objet)

- Focalisation sur les relations entre les objets

- Réutilisation par Composition (association)



### **Héritage** et **Composition**
![Heritage et Composition](images\Couplage_Faible-Fort.png)
>#### Composition
- La composition traduit le terme ***« A un »*** ou
***« A plusieurs »***.

- Couplage ***faible*** si classe a un attribut (classe) qui est une ***interface*** : class C -> Interface A -> Class A ou B etc

- et couplage **fort** si classe a un attribut (classe) qui est une **classe** concrète (**à éviter**) : class C -> Class A

>#### Heritage

- L'héritage traduit le terme ***« Est un »*** ou ***« Une
sorte de »***

- Par défaut couplage **fort** : B extends A donc on a class B -> class A

## Design Pattern List
![Design Pattern Table](images\TableauListDesignPatterns.png)