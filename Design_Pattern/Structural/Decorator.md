# Pattern Decorator

#### Intention

Permet 
- d'***ajouter dynamiquement des fonctionnalités (comportements) additionnelles à un objet***,  **en le plaçant dans des emballeurs** qui implémentent ces comportements. 
- 🚨 **without changing object** original **code**.

Ce pattern peut donc être vue comme une **alternative à l'héritage**.

### Structure

The pattern usually has:
>- **Component** → common ***interface*** for both wrappers and wrapped objects

>- **Concrete Component** → original object (class de l'objet wrappé), It defines the basic behavior, which can be altered by decorators.

>- **Decorator** → *wrapper* class (***abstract*** class), **has a** ***field*** for referencing a **wrapped object**. The field’s **type should be** declared as the **component interface** so it can contain both concrete components and decorators. The base decorator **delegates all operations to the** ***wrapped*** object.

>- **Concrete Decorators** → define **extra behaviors** that can **be added** to components dynamically.

![decorator structure](..\images\Decorator\Decorator_structure.png)

>#### Key Benefits 💡:

\- ✅ Add behavior at runtime

\- ✅ Follows **Open/Closed** Principle

\- ✅ Avoids huge inheritance trees

\- ✅ **Flexible combinations** of features

### Use cases
-  When **you need to be able to assign extra behaviors to objects at** ***runtime*** without breaking the code that uses these objects.

- Use when it’s awkward or not possible to extend an object’s behavior using inheritance.
    >- Many programming languages have the **final** keyword that can be used to prevent further extension of a class. For a **final** class, **the only way to reuse the existing behavior would be to wrap the class** with your own wrapper, using the ***Decorator*** pattern.

##### Exemple
>***Decorator*** pattern is used heavily in **`Java IO (Input/Output)`**:

`FileReader` (class wrappé) -> `BufferedReader` -> `LineNumberReader` : Each wrapper adds new behavior 

![decorator exemple 1 IO](..\images\Decorator\Decorator_exemple_IO_1.png)

`FileOutputStream` (class wrappé) -> `BufferedOutputStream` -> `CipherOutputStream` : Each wrapper adds new behavior 

![decorator exemple 2 IO](..\images\Decorator\Decorator_exemple_IO_2.png)

**Why This Is Powerful**. You can mix behaviors dynamically:
- buffering
- compression (`GZIPOutputStream`)
- encryption
- object serialization (`ObjectOutputStream`)
- data conversion

without creating hundreds of subclasses.

>Avec un **coffee shop**:
- Basic coffee = $5
- Add milk = +$1
- Add sugar = +$0.5

Au lieu de créer plusieurs classes comme:
- CoffeeWithMilk
- CoffeeWithSugar
- CoffeeWithMilkAndSugar

On utilise des decorators to combine features dynamically.

![decorator exemple part 1](..\images\Decorator\Decorator_exemple_part_1.png)
![decorator exemple part 2](..\images\Decorator\Decorator_exemple_part_2.png)