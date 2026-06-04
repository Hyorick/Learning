# Singleton 
**Catégorie**: Création | **Portée**: Objet

#### Intention
***Singleton*** est un design pattern créationnel qui garantit que:

- ***l’instance d’une classe n’existe qu’en un seul exemplaire***, (Exemple avec le `@Bean` d'une classe dans **Spring**, on a une seule instance par défaut)
- tout ***en fournissant un point d’accès global*** à cette **instance**.  (Exemple avec le `@Bean` dans **Spring**, il est ajouté dans l'application Context ce qui permet la réutilisation de l'instance dans toute l'application **Spring**)

> Le Constructeur est ***private***.

> On a une **méthode de création** ***static*** qui va se comporter comme un constructeur. En coulisse, cette méthode appelle le constructeur privé pour créer un objet et le sauvegarde dans un **attribut static**. 

### Approche Thread unsafe

![singleton naive approach](..\images\Singleton\Singleton_naive_approach.png)
### Approche Thread Safe

![singleton naive approach](..\images\Singleton\Singleton_thread_safe_approach.png)
>### (Privilégié) Approche instance crée dès le chargement

![singleton 3rd approach](..\images\Singleton\Singleton_3rd_approach.png)

### Use cases
-  En général, cette situation se présente lorsque l’on veut contrôler l’accès à une ressource partagée — une **base de données** ou un **fichier** par exemple.
- Utiliser le singleton lorsqu'on veut un contrôle absolu sur nos variables globales (singleton garantit l’unicité de l’instance de la classe)