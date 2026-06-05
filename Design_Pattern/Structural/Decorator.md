# Pattern Decorator
**Catégorie**: Structure | **Portée**: Objet

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

#### Exemple
>***Decorator*** pattern is used heavily in **`Java IO (Input/Output)`**:

##### exemple 1 IO
`FileReader` (class wrappé) -> `BufferedReader` -> `LineNumberReader` : Each wrapper adds new behavior 

```Java
import java. io.BufferedReader;
import java. io.FileReader;
import java. io. LineNumberReader;
import java.io.IOException;

public class Main {

    public static void main(String[] args) {

        try {
            // Layer 1: reods characters from file
            FileReader fileReader = new FileReader("test.txt");

            // Layer 2: odds buffering capability
            BufferedReader bufferedReader = new BufferedReader(fileReader);

            // Layer 3: odds Line numbering capability
            LineNumberReader lineNumberReader = new LineNumberReader(bufferedReader);

            String line;

            while ((line = lineNumberReader.readLine()) != null) {
                
                System.out.println(
                    "Line " +
                    lineNumberReader.getLineNumber()
                    + ": " + line
                );
            }

            lineNumberReader.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

What Each Layer Adds
| Layer | Responsibility |
|---|---|
| `FileReader` | Reads characters from a file | 
| `BufferedReader` | Adds buffering for performance |
| `lineNumberReader` | Adds line-number tracking |


##### example Encrypt Data While Writing to File
`FileOutputStream` (class wrappé) -> `BufferedOutputStream` -> `CipherOutputStream` : Each wrapper adds new behavior 

- `FileoutputStream` ➡️ writes bytes to file
- `BufferedoutputStream` ➡️ improves performance
- `CipherOutputStream` ➡️ encrypts data before writing

This is a real ***decorator*** chain used in **Java security APIs**.

```Java
import javax.crypto.Cipher;
import javax.crypto.CipherOutputStream;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;

import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.OutputStream;

public class EncryptExample {

    public static void main(String[] args) throws Exception {

        // Generate AES key
        KeyGenerator keyGen = KeyGenerator.getInstance("AES");
        keyGen.init(128);

        SecretKey secretKey = keyGen.generateKey();

        // Create AES cipher
        Cipher cipher = Cipher.getInstance("AES");
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);

        // layer 1: write to file
        FileOutputStream fos = new FileOutputStream("encrypted.txt");

        // layer 2: add buffering
        BufferedOutputStream bos = new BufferedOutputStream(fos);

        // layer 3: add encryption
        CipherOutputStream cos = new CipherOutputStream(bos, cipher);

        String text = "Hello Decorator Pattern";

        cos.write(text.getBytes());

        cos.close();

        System.out.println("Encrypted data written!");
    }
}
```

>**Why This Is Powerful**. You can mix behaviors dynamically:
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

exemple part 1
- Step 1 **Component Interface**
```java
public interface Coffee {
    String getDescription();
    double getCost();
}
```
- Step 2 **Concrete Component**
```java
public class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }

    @Override
    public double getCost() {
        return 5.0;
    }
}
```
- Step 3 **Base Decorator**
```java
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}
```

- Step 4 **Concrete Decorator**

MilkDecorator
```java
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 1.0;
    }
}
```
SugarDecorator

```java
public class SugarDecorator extends CoffeeDecorator {

    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }

    @Override
    public double getCost() {
        return coffee.getCost() + 0.5;
    }
}
```

- **Usage**
```java
public class Main {

    public static void main(String[] args) {

        Coffee coffee = new SimpleCoffee();

        coffee = new MilkDecorator(coffee);
        coffee = new SugarDecorator(coffee);

        System.out.println(coffee.getDescription());
        System.out.println("Cost: $" + coffee.getCost());
    }
}
```
- Output
<pre>
Simple Coffee, Milk, Sugar
Cost: $6.5
</pre>