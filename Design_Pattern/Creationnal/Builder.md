# Builder 
**Catégorie**: Création | **Portée**: Objet

Spring **lombok**: `@Builder`

#### Intent

***Builder*** is a creational design pattern that ***lets you construct complex objects step by step***. The pattern ***allows you to produce different types and representations of an object*** using the same construction code.

#### Solution

The ***Builder*** pattern suggests that you ***extract the object construction code out of its own class*** and **move it to separate objects** called ***builders***. 

!! The ***Builder*** **doesn’t allow** other objects to **access the product while it’s being built**. Le **constructeur** du **Product** est ***private***.

![builder example](..\images\Builder\Builder_example.png)

>### Structure 
![builder structure](..\images\Builder\Builder_structure.png)

>1. The ***Builder*** interface declares product construction steps that are common to all types of builders.

>2. **Concrete Builders** provide different implementations of the construction steps. Concrete builders may produce products that don’t follow the common interface.

>3. !! **Products** are resulting objects. Products constructed by different builders don’t have to belong to the same class hierarchy or interface.

>4. The **Director** class defines the order in which to call construction steps, so you can create and reuse specific configurations of products. 
***(facultatif)*** Having a **director** class isn’t strictly necessary. You can always call the building steps in a specific order directly from the client code. However, the **director class might be a good place** to **put various construction routines** so you can reuse them across your program.

>5. The **Client** must associate one of the ***builder*** objects with the director. Usually, it’s done just once, via parameters of the director’s constructor. Then the director uses that ***builder*** object for all further construction. However, there’s an alternative approach for when the client passes the ***builder*** object to the production method of the director. In this case, you can use a different ***builder*** each time you produce something with the director.

ou 

**sans le director**
```Java
public class Main {
    public static void main(String[] args){
        System.out.println("Has a garage: " + house.isHasGarage());
    }
}

// House class
@Getter
@Setter
public class House {
    private int floors;
    private int bedrooms;
    private int bathrooms;
    private boolean hasGarage;

    public House() {}

    public House(int floors, int bedrooms, int bathrooms, boolean hasGarage) { ... }

    public static HouseBuilder builder() { return new HouseBuilder(); }

    // House Builder
    public static class HouseBuilder {
        private House house = new House();

        // Etapes
        public HouseBuilder floors(int floors) {
            house.floors = floors;
            return this;
        }

        public HouseBuilder bedrooms(int bedrooms) { ... }

        public HouseBuilder bathrooms(int bathrooms) { ... }

        public HouseBuilder hasGarage(boolean hasGarage) { ... }

        // Build method to create the House
        public House build() { return this.house; }
    }
}
```

>### Use cases
- To get rid of a “telescoping constructor”.
>- You want your code to be able to **create different representations of some product** (for example, stone and wooden houses). on a la possibilité de **fournir des steps de creation par default pour differentes representations** (gaming computer and office computer) d'un **même produit et si ça fit pas on peut toujour tweak les paramètres manuellement lors du build**.
- To construct Composite trees or other complex objects (qui contiennent beaucoup d'attributs).

>**Produits**: computer (gaming builder, office builder etc), bicycle (**vtc builder, road builder, vtt builder** etc avec respectivement **vtcBuilder(), vttBuilder(), roadBuilder()** dans la classe ***bicycle***), car(mercedes, ferrari, etc), house(maison piscine, maison jardin etc), bank account (courant, livret A etc) etc