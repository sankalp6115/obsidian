![](Assets/Pasted%20image%2020260829124455.png)

# Complete Notes: Mastering Object-Oriented Programming in Java

---

## 1. Objects & Object-Oriented Programming in Java (OOPS Tutorial)

_Source Video: "Objects Oriented Programming In Java - OOPS in Java Complete Tutorial"_

### Why Move from Procedural Programming to OOP?

- **Limitations of Procedural Code**: In procedural code, data and variables are handled openly. For example, storing 3 attributes (name, roll number, address) for 50 students using basic variables would require 150 individual variables.
- **Failure of Parallel Arrays**: Using separate arrays for each attribute creates indexing and sorting vulnerabilities; if the sorting or indexing gets out of alignment, student names, roll numbers, and addresses detach from one another.
- **The OOP Approach**: A single `Student` class acts as a template containing the 3 attributes. You only declare the blueprint once and instantiate 50 student objects, reducing complexity and keeping data grouped.
- **Security**: In procedural setups, variables are often globally accessible across the entire package or file. OOP introduces data hiding via access modifiers (such as `private`) to prevent unauthorized external access.

### Core Concepts & Analogies

- **Class (The Blueprint)**: A logical template that defines what properties and behaviors an entity will possess.
    - _Analogy_: A class is like a blueprint for furniture or a Power Ranger specification where powers are pre-defined.
- **Object (The Instance)**: An actual physical or memory instance created from the class blueprint.
    - _Analogy_: Pressing the transformation button activates a specific Power Ranger with their allocated powers.
- **Deconstructing `Scanner sc = new Scanner(System.in)` via OOP**:
    - `Scanner`: The class name.
    - `sc`: The object / reference variable name.
    - `new`: Allocates new memory space for the object.
    - `Scanner(System.in)`: The constructor of the `Scanner` class; `System.in` connects it to standard keyboard input.
    - `sc.nextInt()`: Invoking a method on the `sc` object to read data.

### Code Implementation

```
class Student {
    String name;
    int rollNumber;

    void printData() {
        System.out.println(name);
        System.out.println(rollNumber);
    }
}

public class Main {
    public static void main(String[] args) {
        Student std1 = new Student();
        std1.name = "Ram";
        std1.rollNumber = 15;
        std1.printData();

        Student std2 = new Student();
        std2.name = "Shyam";
        std2.rollNumber = 24;
        std2.printData();
    }
}
```

---

## 2. Class Attributes & Class Methods

_Source Video: "Class Attributes and Class Methods in Java | OOPS in Java Complete Tutorial"_

### Class Attributes (Data Members)

- Attributes are variables declared inside a class that represent the state or properties of that class (e.g., `name`, `rollNumber`, `address`).
- **Accessing Attributes**: Attributes are accessed using the dot operator (`.`) on an instance of the class (`obj.attributeName`), or directly if marked `static`.
- **State Independence**: Each object maintains its own independent copy of attributes in memory; modifying `std1.name` to `"Raman"` does not change `std2.name`.

### The `final` Keyword on Attributes

- Marking an attribute with `final` makes its value constant and immutable once initialized.
- Any subsequent attempt to reassign the variable causes a compilation error: `cannot assign a value to final variable`.

### Class Methods (Behaviors)

- Functions defined inside a class are called **methods**. They define the actions, operations, or behaviors that an object can execute.

### Code Implementation

```
class Dog {
    String name;
    int licenseId;

    public void eat() {
        System.out.println(name + " eats");
    }
}

public class Demo {
    public static void main(String[] args) {
        Dog dog1 = new Dog();
        dog1.name = "Bruno";
        dog1.licenseId = 101;
        dog1.eat(); // Output: Bruno eats
    }
}
```

---

## 3. Constructors in Java (Default & Parameterized)

_Source Video: "Constructor in Java - What is Constructor? | Default and Parameterized Constructor"_

### What is a Constructor?

- A constructor is a special type of method used to initialize objects.
- **Mandatory Rules**:
    1. The constructor name **must be exactly identical** to the class name.
    2. It **does not have a return type** (not even `void`).
    3. It executes **automatically** at the exact moment the object is instantiated via the `new` keyword.

### Use Cases & The `this` Keyword

- Eliminates the need to write separate initialization methods to populate data members.
- In the ATM project, placing `checkPin()` logic inside the constructor ensures verification runs immediately upon creating an ATM object without manual method calls.
- **`this` Keyword**: Used inside constructors to resolve variable shadowing by distinguishing class instance variables from constructor parameters (`this.id = id`).

### Types of Constructors

1. **Default (Non-Parameterized) Constructor**: Takes no parameters; initializes fields to default values or runs default setup logic.
2. **Parameterized Constructor**: Takes arguments to initialize class data members with specific values upon creation.
3. **Constructor Overloading**: Defining multiple constructors within the same class differing in the number, types, or order of arguments.

### Code Implementation

```
class Student {
    int id;
    int age;

    // Default Constructor
    Student() {
        System.out.println("Default constructor called");
    }

    // Parameterized Constructor (2 parameters)
    Student(int id, int age) {
        this.id = id;
        this.age = age;
        System.out.println("Parameterized constructor called");
    }

    // Overloaded Parameterized Constructor (3 parameters)
    Student(int id, String name, int age) {
        this.id = id;
        this.age = age;
        System.out.println("Parameterized constructor 2 called");
    }
}
```

---

## 4. The 4 Pillars of Object-Oriented Programming (Overview)

_Source Video: "4 Pillars of Object Oriented Programming - Abstraction, Inheritance, Encapsulation and Polymorphism"_

### Overview of the Four Pillars

1. **Abstraction**: Providing only essential functionality to the user while hiding underlying internal complexities.
    - _Analogy_: Driving a car using the push-start button, steering wheel, and pedals without needing to understand piston mechanics, fuel injection, or oil pressure. A TV remote displays only buttons, hiding internal circuitry.
    - _Implementation_: Achieved in Java via **Abstract Classes** and **Interfaces**.
2. **Inheritance**: The mechanism by which a child class acquires common attributes and methods from an existing parent class.
    - _Analogy_: A biological child inheriting physical or behavioral traits from parents.
    - _Relationship_: Establishes an **IS-A** relationship (e.g., `Car IS-A Vehicle`). Promotes code reusability and eliminates redundancy.
3. **Encapsulation**: Wrapping data fields and related methods together into a single unit (class) and restricting direct external modification of variables.
    - _Analogy_: A medical capsule containing multiple medicinal compounds inside a single protective gelatin shell.
    - _Implementation_: Achieved using **Access Modifiers** (`private`, `public`, `protected`, `default`).
4. **Polymorphism**: An entity existing in multiple distinct forms in different contexts.
    - _Analogy_: A single method `eat()` performs different behaviors for humans and animals. Lord Vishnu manifesting across different yugas in distinct avatars (Rama, Krishna, Parashurama) to fulfill different roles.
    - _Types_: Method Overloading (Compile-Time / Static) and Method Overriding (Runtime / Dynamic).

---

## 5. Inheritance in Java (Core Concepts)

_Source Video: "Inheritance in java - What is Inheritance in java | Java OOPS Complete course"_

### Foundations of Inheritance

- Inheritance allows a new class (subclass/child class) to inherit members and methods from an existing class (superclass/parent class).
- **Core Benefits**:
    - **Code Reusability**: Common code written in the superclass does not need to be rewritten in child classes.
    - **Reduces Code Duplication**: Shrinks codebase size and maintenance overhead.
    - **Enables Method Overriding**: Child classes can customize parent behaviors.

### Key Terminology & Relationships

- **Superclass / Parent Class**: The class whose features are inherited.
- **Subclass / Child Class**: The class that inherits the features.
- **`extends` Keyword**: The Java keyword used in class declarations to inherit from a superclass.
- **IS-A vs HAS-A**:
    - `Car IS-A Vehicle`: Achieved via inheritance (`extends`).
    - `Car HAS-A Engine`: Achieved via aggregation/composition (embedding an instance of `Engine` inside `Car`).

### Code Implementation

```
class Vehicle {
    double price;
    double mileage;
    String color;

    void display() {
        System.out.println("Price: " + price);
        System.out.println("Mileage: " + mileage);
        System.out.println("Color: " + color);
    }
}

class Car extends Vehicle {
    String fuelType;
    boolean sunroof;
    String brand;
}

public class InheritanceDemo {
    public static void main(String[] args) {
        Car car1 = new Car();
        car1.brand = "Tata";
        car1.price = 1500000;
        car1.mileage = 18.5;
        car1.color = "Red";
        car1.sunroof = true;

        car1.display(); // Calls parent Vehicle display() method
        System.out.println("Sunroof: " + car1.sunroof);
    }
}
```

---

## 6. Single Inheritance

_Source Video: "Single Inheritance In Java - Types Of Inheritance | Object Oriented Programming in Java"_

### Concept

- The simplest inheritance model where **one single child class directly extends one single parent class**.
- Establishes a direct, one-to-one IS-A relationship.

### Code Implementation

```
class Employee {
    void salary() {
        System.out.println("Salary = 70,000");
    }
}

class HR extends Employee {
    void bonus() {
        System.out.println("Bonus = 20,000");
    }
}

public class SingleInheritanceDemo {
    public static void main(String[] args) {
        HR hr = new HR();
        hr.salary(); // Inherited from Employee
        hr.bonus();  // Specific to HR
    }
}
```

---

## 7. Multilevel Inheritance

_Source Video: "Multi Level Inheritance In Java - Object Oriented Programming In Java | Indian Programmer"_

### Concept & Hierarchy

- Inheritance structured across multiple cascading levels (e.g., Grandparent \(\rightarrow\) Parent \(\rightarrow\) Child).
- Class `B` extends Class `A`, and Class `C` extends Class `B`.
- Class `C` acquires the members and methods of both Class `B` and Class `A`.
- **Directional Access**: Subclasses at the bottom can access members up the entire chain; however, superclasses at the top cannot access methods of their subclasses.

### Code Implementation

```
class A {
    void methodA() {
        System.out.println("Method of A class");
    }
}

class B extends A {
    void methodB() {
        System.out.println("Method of B class");
    }
}

class C extends B {
    void methodC() {
        System.out.println("Method of C class");
    }
}

public class MultiLevelDemo {
    public static void main(String[] args) {
        C obj = new C();
        obj.methodC(); // Own method
        obj.methodB(); // From class B
        obj.methodA(); // From class A
    }
}
```

---

## 8. Multiple Inheritance & Why Java Does Not Support It Directly

_Source Video: "Multiple Inheritance in Java Part -1 | Why Multiple Inheritance is not supported in java"_

### Concept

- Multiple inheritance occurs when a single child class attempts to extend two or more parent classes simultaneously (`class C extends A, B`).

### Why Java Disallows It (The Diamond Problem / Ambiguity)

- If Parent Class `A` has a method `void show()`, and Parent Class `B` also has an identical method `void show()`, and Class `C` inherits from both:
    
    ```
    C obj = new C();
    obj.show(); // Which parent method should execute?
    ```
    
- The Java compiler cannot resolve which version of `show()` to execute, resulting in method ambiguity.
- To prevent this ambiguity conflict (commonly termed the Diamond Problem), Java strictly forbids multiple inheritance with classes using `extends`. It is resolved exclusively through **Interfaces**.

---

## 9. Interfaces in Java

_Source Video: "Interface In Java - What is Interface in Java | Object Oriented Programming in Java"_

### What is an Interface?

- An interface is a contract containing a blueprint of methods that implementing classes must define.
- All methods declared in an interface are implicitly `public` and `abstract` (no method bodies allowed in standard interfaces).
- A class uses the `implements` keyword to adopt an interface.

### The Head Chef Pizza Analogy

- A master head chef at a pizzeria creates a contract recipe specifying 5 mandatory steps: `makeDough()`, `proofDough()`, `prepareToppings()`, `shapeDough()`, and `bakePizza()`.
- The junior chefs are required to implement all 5 steps, but each chef retains freedom in _how_ they execute each step (e.g., hand-stretching vs. rolling dough).

### Core Properties & Differences from Abstract Classes

- **Level of Abstraction**: Interfaces provide **100% abstraction**, whereas abstract classes provide partial to full abstraction (0% to 100%).
- **No Constructors**: Interfaces cannot have constructors because interfaces can never be directly instantiated.
- **Instance Creation**: You cannot execute `new InterfaceName()`; an interface reference variable must hold an instance of a concrete implementing class.
- **Loose Coupling**: Minimizes inter-component software dependencies, preventing bugs in one module from breaking others.

### Code Implementation

```
interface Car {
    void start(); // public and abstract by default
}

class ElectricCar implements Car {
    public void start() {
        System.out.println("Electric car starts silently with push button");
    }
}

class DieselCar implements Car {
    public void start() {
        System.out.println("Diesel car starts with key ignition");
    }
}

public class InterfaceDemo {
    public static void main(String[] args) {
        Car tesla = new ElectricCar();
        Car xuv = new DieselCar();

        tesla.start();
        xuv.start();
    }
}
```

---

## 10. Multiple Inheritance Using Interfaces

_Source Video: "Multiple Inheritance in Java Using Interface Part - 2 | Indian Programmer"_

### How Interfaces Solve the Diamond Problem

- When classes `A` and `B` have concrete method bodies, inheriting both causes ambiguity.
- With interfaces, Interface `A` and Interface `B` only _declare_ `void run();` without providing an implementation body.
- When Class `C implements A, B`, Class `C` writes the single, definitive implementation body for `run()`.
- The ambiguity is eliminated because there is only one implementation body in existence (inside Class `C`).

### Code Implementation

```
interface A {
    void run();
}

interface B {
    void run();
}

class C implements A, B {
    public void run() {
        System.out.println("C class run method implementation");
    }
}

public class MultipleInheritanceResolved {
    public static void main(String[] args) {
        C obj = new C();
        obj.run(); // Successfully runs with zero compiler ambiguity
    }
}
```

---

## 11. The `super` Keyword

_Source Video: "Super Keyword in Java - Object Oriented Programming In Java | Indian Programmer"_

### Purpose

- A reference variable used within a subclass to explicitly access and invoke members of its immediate parent (super) class.

### Three Primary Use Cases

1. **Invoking Parent Instance Variables**: Used when a subclass variable shadows a superclass variable of the same name (`super.variableName`).
2. **Invoking Parent Methods**: Used to call the superclass version of a method that has been overridden in the child class (`super.methodName()`).
3. **Invoking Parent Constructors**: Calling `super()` or `super(arguments)` invokes the constructor of the parent class. It must be the very first line inside the subclass constructor.

### Code Implementation

```
class Animal {
    String color = "white";

    Animal() {
        System.out.println("Animal is created");
    }

    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    String color = "black";

    Dog() {
        super(); // Invokes parent constructor
        System.out.println("Dog is created");
    }

    void printColor() {
        System.out.println(color);       // Prints black (child variable)
        System.out.println(super.color); // Prints white (parent variable)
    }

    void eat() {
        System.out.println("Eating bread");
    }

    void work() {
        super.eat(); // Calls Animal eat() method
        eat();       // Calls Dog eat() method
    }
}
```

---

## 12. Hierarchical Inheritance

_Source Video: "Hierarchical Inheritance in Java - Java OOPS Complete Course | Indian Programmer"_

### Concept

- An inheritance model where **one single parent class is extended by multiple child classes**.
- _Analogy_: A family tree where a single father has multiple children; all siblings share the father's properties, but are independent of each other.

### Code Implementation

```
class A {
    void print() {
        System.out.println("Class A method");
    }
}

class B extends A {
    void printB() {
        System.out.println("Class B method");
    }
}

class C extends A {
    void printC() {
        System.out.println("Class C method");
    }
}

public class HierarchicalDemo {
    public static void main(String[] args) {
        B objB = new B();
        C objC = new C();

        objB.print(); // Accesses parent A
        objC.print(); // Accesses parent A
    }
}
```

---

## 13. Method Overloading

_Source Video: "Method Overloading In Java - Object Oriented Programming in Java | Indian Programmer"_

### What is Method Overloading?

- Defining multiple methods inside the **same class** with the **same method name**, but with **different parameter signatures**.
- Also referred to as _Function Overloading_ in procedural languages like C, but called _Method Overloading_ in Java because all functions reside inside classes.

### Rules to Overload Methods

1. Method names must be identical.
2. Must have a different number of arguments/parameters, **OR**
3. Must have different data types for the parameters. _(Note: Changing only the return type without modifying parameters is not valid overloading.)_

### Code Implementation

```
class Calculator {
    // 2 integer parameters
    static int sum(int a, int b) {
        return a + b;
    }

    // 3 integer parameters
    static int sum(int a, int b, int c) {
        return a + b + c;
    }

    // 2 String parameters (different data type)
    static String sum(String a, String b) {
        return a + b;
    }
}

public class OverloadingDemo {
    public static void main(String[] args) {
        System.out.println(Calculator.sum(5, 7));             // Output: 12
        System.out.println(Calculator.sum(5, 7, 8));          // Output: 20
        System.out.println(Calculator.sum("Indian ", "Prog")); // Output: Indian Prog
    }
}
```

---

## 14. Method Overriding

_Source Video: "Method Overriding In Java - Object Oriented Programming In Java | Indian Programmer"_

### What is Method Overriding?

- When a subclass provides a specific implementation of a method that is already defined in its superclass.
- **Difference from Overloading**: Overloading works with parameter lists inside the same class, whereas overriding changes the **method body** across classes linked by **inheritance**.
- **Requirements**:
    1. Method name, return type, and parameter list must be identical to the parent method.
    2. Classes must have an inheritance (`extends`) relationship.

### The Bank Interest Example & `@Override` Annotation

- A base `Bank` class provides a generic `getRateOfInterest()` returning 5%. Subclasses `SBI` (6.5%) and `PNB` (6%) override this method to provide their own specific interest rates.
- **Best Practice**: Always prepend `@Override` above the overridden method to alert the compiler to verify the signature and ensure cleaner code maintenance.

### Code Implementation

```
class Bank {
    int getRateOfInterest() {
        return 5;
    }
}

class SBI extends Bank {
    @Override
    int getRateOfInterest() {
        return 7; // Custom rate for SBI
    }
}

class PNB extends Bank {
    @Override
    int getRateOfInterest() {
        return 6; // Custom rate for PNB
    }
}
```

---

## 15. Polymorphism in Java

_Source Video: "Polymorphism in Java - Object Oriented Programming In Java | Indian Programmer"_

### Definition & Real-Life Analogies

- Derived from Greek: _Poly_ (many) + _Morph_ (forms) — the ability of an entity to take on different forms or behaviors depending on the context.
- **The Mother Analogy**: A woman functions as a mother to her children, a daughter to her parents, a sister to her siblings, an employee at her workplace, and a friend to her peers. The person remains the same, but her behaviors and roles adapt to the setting.
- **The Avatars Analogy**: Lord Vishnu manifesting in different incarnations (Rama, Krishna, Parashurama) to achieve different purposes across yugas.
- In Java, polymorphism is achieved through **Method Overloading** and **Method Overriding**.

---

## 16. Types of Polymorphism: Compile-Time vs. Runtime

_Source Video: "Types Of Polymorphism in Java - Runtime vs Compile Time Polymorphism in Java"_

### Comparison Matrix

|Feature|Compile-Time Polymorphism|Runtime Polymorphism|
|:--|:--|:--|
|**Alternative Names**|Static Binding / Early Binding|Dynamic Binding / Late Binding|
|**Achieved Via**|Method Overloading|Method Overriding|
|**Resolution Time**|Method calls are resolved at compilation time by the compiler|Method calls are resolved at runtime by the JVM based on the actual object|
|**Inheritance Requirement**|Not required; can exist inside a single class|Strictly requires inheritance between classes|
|**Execution Speed**|Faster (compiler already knows the exact target method)|Slower (method lookup happens dynamically during execution)|
|**Flexibility**|Less flexible (locked at compile time)|Highly flexible (behavior changes dynamically based on runtime object)|

---

## 17. Access Modifiers in Java

_Source Video: "Access Modifiers in Java - PUBLIC, PRIVATE, PROTECTED and DEFAULT"_

### Purpose

- Keywords that regulate the scope, visibility, and accessibility of classes, constructors, methods, and variables.

### The 4 Access Modifiers

1. **`public`**: Accessible from anywhere across the entire project (any class, package, or subclass).
    - _Why `main` is public_: The Java Virtual Machine (JVM) runs externally to the application package and requires public access to invoke `main()`.
2. **`private`**: Strictly accessible **only within the exact same class** in which it is declared. Neither external classes nor subclasses can access private members.
3. **`default` (package-private)**: Applied automatically when no modifier keyword is specified. Accessible from any class **within the same package**, but completely inaccessible outside that package.
4. **`protected`**: Accessible within the same package, and accessible across different packages **only through inheritance (subclasses)** via `extends`.

### Accessibility Matrix

|Access Modifier|Same Class|Same Package (Subclass)|Same Package (Non-subclass)|Different Package (Subclass)|Different Package (Non-subclass)|
|:--|:-:|:-:|:-:|:-:|:-:|
|**`public`**|Yes|Yes|Yes|Yes|Yes|
|**`protected`**|Yes|Yes|Yes|Yes (via `extends`)|No|
|**`default`**|Yes|Yes|Yes|No|No|
|**`private`**|Yes|No|No|No|No|

---

## 18. Encapsulation & Getter/Setter Methods

_Source Video: "Encapsulation in Java - Getter and Setter Methods with Code and Examples"_

### Concept of Encapsulation

- Encapsulation is the mechanism of binding variables (data) and methods into a single protective class capsule while restricting direct external manipulation.
- **The Fort Analogy**: Imagine a fortified palace containing treasures, armories, and council halls. The palace does not allow open entry from all sides; it exposes only two strictly guarded gates: a **Get Gate** (to view/read) and a **Set Gate** (to deposit/update).

### Implementation Pattern

1. Declare all class data members as `private`.
2. Expose `public` getter methods to return the values.
3. Expose `public` setter methods with validation logic to initialize or update the values.

### Code Implementation

```
class Student {
    private String name;
    private int age;
    private int rollNumber;

    // Getter and Setter for Name
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }

    // Getter and Setter for Age
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }

    // Getter and Setter for Roll Number
    public int getRollNumber() {
        return rollNumber;
    }
    public void setRollNumber(int rollNumber) {
        this.rollNumber = rollNumber;
    }
}

public class EncapsulationDemo {
    public static void main(String[] args) {
        Student std = new Student();
        // std.name = "Rahul"; // ERROR: name has private access in Student

        std.setName("Rahul");
        std.setAge(21);
        std.setRollNumber(101);

        System.out.println(std.getName());
        System.out.println(std.getAge());
        System.out.println(std.getRollNumber());
    }
}
```

---

## 19. Abstraction in Java (Abstract Classes & Methods)

_Source Video: "Abstraction in Java - Abstract Classes and Abstract Methods with Code and Examples"_

### What is Abstraction?

- The process of hiding internal background implementation mechanics and showing only the essential operational features to the user.
- **Consumer vs Developer Benefit**:
    - _For the Consumer_: Simplicity; they only need to know which button or method to call without worrying about background complexity.
    - _For the Developer_: Security and modularity; core business logic and critical infrastructure remain hidden and protected from direct tampering.

### Abstract Class Rules

1. Declared using the `abstract` keyword (`abstract class Car`).
2. **Cannot be instantiated directly**: Calling `new Car()` produces a compilation error.
3. Can contain both regular (concrete) methods with bodies and abstract methods.

### Abstract Method Rules

1. Declared using the `abstract` keyword with **no method body** and terminated with a semicolon: `abstract void fuelType();`.
2. Any regular subclass that `extends` the abstract class **must override and provide method bodies for all inherited abstract methods**, or the subclass itself must be declared `abstract`.

### Code Implementation

```
abstract class Car {
    // Abstract method (no body)
    abstract void fuelType();

    // Concrete regular method
    void color() {
        System.out.println("Black");
    }
}

class Tata extends Car {
    // Mandatory implementation of abstract method
    @Override
    void fuelType() {
        System.out.println("Diesel");
    }
}

public class AbstractionDemo {
    public static void main(String[] args) {
        Tata nexon = new Tata();
        nexon.fuelType(); // Output: Diesel
        nexon.color();    // Output: Black
    }
}
```

---

_(Note: The playlist also features dedicated hands-on project tutorials—including the ATM Machine, Car Showroom Management System, Employee Payroll System, and Car Rental System—which tie these conceptual lessons together into full-length software implementations.)_

---

💡 Would you like to generate a structured study quiz or flashcard set to test your recall on these core Java OOP principles?