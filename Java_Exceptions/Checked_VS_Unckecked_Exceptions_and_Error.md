# Checked VS Unckecked Exceptions and Error

    Object
      │
      └── Throwable
          |
          ├── Error
          │    ├── OutOfMemoryError
          │    ├── StackOverflowError
          │    └── VirtualMachineError
          │
          └── Exception
                ├── RuntimeException   <-- unchecked
                │    ├── NullPointerException
                │    ├── IllegalArgumentException
                │    ├── IndexOutOfBoundsException
                │    └── ArithmeticException
                │
                └── Checked Exceptions
                    ├── IOException
                    ├── SQLException
                    ├── ClassNotFoundException
                    └── ParseException



Difference summary



| Type              | Checked? | Typical Cause        | Should Catch? |
| ----------------- | -------- | -------------------- | ------------- |
| Checked Exception | Yes      | External/recoverable | Often yes     |
| RuntimeException  | No       | Programming bug      | Usually no    |
| Error             | No       | JVM/system failure   | Almost never  |



#### Unchecked Exception (RuntimeException types)



Idée :

Unchecked exceptions = **programming mistakes** (erreur/bug de programmations) / **Contract violations**



* 💡On ne doit pas les ***catch***.**💡**The **'caller'** usually **not** **expected** to **handle** (**try/catch**) **it routinely**

  * **Exception** si on travaille sur une Web API et qu'on veut renvoyer un message request suite à la RuntimeException qui est apparu
<pre>
  try {
    processPayment();
  } catch (IllegalArgumentException e) {
    return BAD_REQUEST;
  }
</pre>

* **facultatif** de les **déclarer** dans le `**throws**` 
* 💡Par contre on peut (**facultatif**) les "**throw**" mais c'est **à** **titre** purement **indicatif, pour le développeur** (**Bonne pratique** de les **indiquer explicitement** car on sais (humain) ce qui peut se produire au ***runtime***)
* On peut définir des **customs** qui extends **RuntimeException**
* **💡Unchecked** exceptions **occur** (se produisent) **at runtime.**



Example, Things caused by bad code:

* null dereference (NullPointerException)
* wrong index (IndexOutOfBoundsException)
* illegal arguments (IllegalArgumentException)
* divide by zero (ArithmeticException)



Java says:

“Don’t force handling. **Fix the code**.”

* you should **FIX the code**. **not surround** everything with **try/catch**



#### Checked Exception



**Expected** operational **failures 💡*during normal operation*💡** (différent de **unchecked: contract violations/programming mistakes)**



**Compiler** forces you to either:

* ***catch*** it (**try/catch**). **💡**The **'caller'** usually **expected** to **handle** (**try/catch**) **it routinely**
* or declare it into `**throws**`



* On peut définir des **customs** qui extends **Exception**
* **💡Checked** exceptions **are checked** (sont vérifiées) at **compile time.**



Things outside your control:

Exemple

* file missing
* network down
* database unavailable
* invalid user input file
* API timeout



These **may happen even with perfect code**.



#### Error class

* On ne doit pas les ***catch***.
* On peut définir des **customs** qui extends **Error (!! Déconseillé)**



Error indicate:

* **🚨JVM problem**
* **🚨serious** environment **failure**
* **🚨system failure**
* 🚨catastrophic state



Exemple

* OutOfMemoryError (heap memory full)
* StackOverflowError (stack memory full)
* NoClassDefFoundError
* VirtualMachineError



#### exception being “hidden” behind another



* Exception chaining / **wrapped exceptions** (**most common**) : “This happened BECAUSE OF another exception.”
* **Suppressed** exceptions (usually with `**try-with-resources**`): “Another exception ALSO happened during cleanup.”
* **Exception masking** (less formal term) : “One exception accidentally hid another.” (**🚨dangerous** for **debugging**)



