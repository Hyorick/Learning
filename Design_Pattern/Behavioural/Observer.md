## Pattern Observer
>**Catégorie**: Comportement | **Portée**: Objet

Le pattern ***Observer*** definit une relation entre les objets de type 1..* de sorte que lorsqu'un objet change d'état, tous ce qui en dépendent en soient informés et mis à jour automatiquement.

```Mermaid
classDiagram
    %% ============================
    %%        INTERFACES
    %% ============================

    class Observable {
        <<interface>>
        - List~Observer~ observers
        + subscribe(o: Observer)
        + unsubscribe(o: Observer)
        + notify()
    }

    class Observer {
        <<interface>>
        + update()
    }

    %% ============================
    %%        CONCRETE CLASSES
    %% ============================

    class ConcreteObservable {
        <<class>>
        - state
        + getState()
        + setState(s)
    }

    class ConcreteObserver {
        <<class>>
        - observerState
        + update()
    }

    %% ============================
    %%        RELATIONS
    %% ============================

    Observable "1" --> "0..*" Observer : notifies
    Observable <|-- ConcreteObservable
    Observer <|-- ConcreteObserver
```
2 Interfaces:

#### Observable
 - L'objet observé (**Observable**) détient l'état (information), 
 - gèrent une liste d'observateurs (**Observer**) avec subscribe/unsubscribe
 - et les notifie (en appelant la méthode **update()** de l'**Observer**) automatiquement lors d'un changement de son état ou de l'exécution de certains comportements.

#### Observer
- s’abonne (peut aussi se désabonner quand il veut) à l’**Observable** 
- et implémente **update()** pour réagir et se synchroniser avec  l'état de l'**Observable**.

### 👉 Cas d'utilisation 
- Utilisez ce patron quand des modifications de l’**état d’un objet peuvent en impacter d’autres**, et que l’*ensemble des objets n’est pas connu à l’avance* ou qu’il change dynamiquement. 
- Utiliser ce patron quand **certains objets de votre application doivent en suivre d’autres**, mais *seulement pendant un certain temps ou dans des cas spécifiques*

### Lien avec les autres design pattern
>La **Chain of responsability**, **Commander**, **Mediator** et l’**Observer** proposent différentes solutions pour associer les senders et les receivers.
>- *Chain of Responsibility* passes a request sequentially along a dynamic chain of potential receivers until one of them handles it.
>- *Command* establishes unidirectional connections between senders and receivers.
>- *Mediator* eliminates direct connections between senders and receivers, forcing them to communicate indirectly via a mediator object.
>- *Observer* lets receivers dynamically subscribe to and unsubscribe from receiving requests
