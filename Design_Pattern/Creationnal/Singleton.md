# Singleton 
**Catégorie**: Création | **Portée**: Objet

#### Intention
***Singleton*** est un design pattern créationnel qui garantit que:

- ***l’instance d’une classe n’existe qu’en un seul exemplaire***, (Exemple avec le `@Bean` d'une classe dans **Spring**, on a une seule instance par défaut)
- tout ***en fournissant un point d’accès global*** à cette **instance**.  (Exemple avec le `@Bean` dans **Spring**, il est ajouté dans l'application Context ce qui permet la réutilisation de l'instance dans toute l'application **Spring**)

> Le Constructeur est ***private***.

> On a une **méthode de création** ***static*** qui va se comporter comme un constructeur. En coulisse, cette méthode appelle le constructeur privé pour créer un objet et le sauvegarde dans un **attribut static**. 

### Approche Thread unsafe

```java
package fr.koor.samples;

public final class Singleton { //Approche naïve du Singleton

    private static Singleton instance = null;

    private Singleton() {}

    public static Singleton getInstance () {
        if ( instance == null ) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

### Approche Thread Safe

```java
public class Singleton {

    // volatile ensures visibility of changes across threads
    private static volatile Singleton instance;

    // private constructor prevents instantiation
    private Singleton() {}

    public static Singleton getInstance() {
        // First check (no locking for better performance)
        if (instance == null) {

            synchronized (Singleton.class) {
                // Second check (with Locking)
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }

        return instance;
    }
}
```

>### (Privilégié) Approche instance crée dès le chargement

```java
package fr.koor.samples;

public final class Singleton {

    private static final Singleton INSTANCE = new Singleton ();

    private Singleton() {

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

### Use cases
-  En général, cette situation se présente lorsque l’on veut contrôler l’accès à une ressource partagée — une **base de données** ou un **fichier** par exemple.
- Utiliser le singleton lorsqu'on veut un contrôle absolu sur nos variables globales (singleton garantit l’unicité de l’instance de la classe)