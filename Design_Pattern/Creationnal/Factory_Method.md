# Factory Method
**Catégorie**: Création | **Portée**: Classe

#### Intent

***Factory Method*** is a creational design pattern that 
- ***provides an interface for creating an object in a superclass*** (`Creator`),
- but ***let subclasses*** (`ConcreteCreatorA`, `ConcreteCreatorB`) to ***decide the type of objects***(concrete classe to instantiate) (`ConcreteProductA`, `ConcreteProductB`, `ConcreteProductC` type implementing `Creator` interface) ***that will be created***.
>- "***Separate object creation from object usage.***" : Instead of every class deciding HOW to build objects,
one place handles creation.
- ***Factory Method*** is a specialization of **Template Method** design pattern.

<pre>
Creator «abstract class or class»: AnimalFactory

ConcreteCreators : *RandomFactory*, *BalancedFactory*, *DogFactory*, *CatFactory* 

Product «Interface»: Animal

Products : *Cat*, *Dog*, *Duck*
</pre>

### Structure

![Factory method structure](..\images\FactoryMethod\FactoryMethod_structure.png)

>1. The **Product** declares the interface, which is **common to all objects** that can be produced by the ***creator and its subclasses***.

2. **Concrete Products** are different implementations of the product interface.

 - 3.The **Creator** class declares the factory method that returns new product objects. It’s important that the return type of this method matches the product interface.
    - You can **declare the** (*static*) ***factory method*** as *abstract* to force all **subclasses** to *implement their own versions* of the method. As an alternative, the base factory method can return some default product type.
    - Note, despite its name, **product creation is not the primary responsibility** of the ***creator***. Usually, the creator class already has some core business logic related to products. The factory method helps to decouple this logic from the concrete product classes. Here is an analogy: a large software development company can have a training department for programmers. However, the primary function of the company as a whole is still writing code, not producing programmers.

4. **Concrete Creators** override the base factory method so it returns a different type of product.

>Note that the factory method **doesn’t have to create new instances all the time**. It can also *return existing objects from a cache, an object pool, or another source*.

### Use cases
-  You **don’t know beforehand the exact types** and dependencies of the **objects your code should work with**.
    - The Factory Method **separates** product **construction code** from the **code that actually uses the product**. Therefore it’s easier to extend the product construction code independently from the rest of the code.
-  You want to **provide users of your library** or framework **with a way to extend its internal components**.
- You want to **save system resources by reusing existing objects** instead of rebuilding them each time. (Ex: "String" object)
    -  You often experience this need when dealing with large, resource-intensive objects such as database connections, file systems, and network resources.

> 👉 **Good Rule of Thumb**. Use Factory Method when:

- Object creation is complex
- Many object (product) types exist
- Creation may change later
- you want loose coupling

>🚫 **Avoid it** when:

- Object creation is trivial
- Application is very small
- Only one implementation exists
- Factory only wraps "new"

### Exemple
![asteroid game](..\images\FactoryMethod\FactoryMethod_Example_asteroid_game.png)

Client

![Client game](..\images\FactoryMethod\FactoryMethod_Example_asteroid_game_Client.png)