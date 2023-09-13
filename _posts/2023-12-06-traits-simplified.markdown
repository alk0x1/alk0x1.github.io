---
layout: post
title:  "Simplifying Traits in Rust"
date:   2023-06-23 14:35:50 -0300
categories: ["Rust"]

---

I rarely write about some specific feature of an language I prefer study about general concepts but the way that traits apply polymorphism is really cool because is a really well mix of OOP and FP programming concepts that build something different and powerful.

1. What are traits
2. Why traits are unique and powerful feature
3. Code example to show the differences between polymorphism in Java and Rust 


## What are traits?
Traits allow developers to define shared behaviors that can be implemented by different types, enabling code reuse and polymorphism.

**Example:**
```rust
trait Animal {
    fn make_sound(&self);
}

struct Dog {
    name: String,
}

struct Cat {
    name: String,
}

impl Animal for Dog {
    fn make_sound(&self) {
        println!("{} says: Woof!", self.name);
    }
}

impl Animal for Cat {
    fn make_sound(&self) {
        println!("{} says: Meow!", self.name);
    }
}

fn main() {
    let dog = Dog {
        name: String::from("Buddy"),
    };
    let cat = Cat {
        name: String::from("Whiskers"),
    };

    let animals: Vec<Box<dyn Animal>> = vec![
        Box::new(dog),
        Box::new(cat),
    ];

    for animal in animals {
        animal.make_sound();
    }
}
```

## How traits are different?
Polymorphism in Java or C++ is achieved through inheritance and virtual function dispatch. It relies on class hierarchies, where a superclass defines a common interface or behavior, and subclasses provide specific implementations. Polymorphism allows objects of different classes to be treated as objects of a common superclass, enabling dynamic method dispatch and runtime polymorphic behavior.

On the other hand, traits in Rust provide a form of ad hoc polymorphism or interface-based programming. Traits define a set of methods that can be implemented by different types. Unlike class hierarchies, traits are not tied to a specific type or inheritance hierarchy. Any type can implement a trait as long as it provides the required methods. This decoupling of traits from types allows for greater flexibility and composability.

#### Some key differences:

**Inheritance vs. Composition:** Polymorphism relies on inheritance and subclassing, where subclasses inherit and override methods from a superclass. Traits, on the other hand, promote composition over inheritance. They allow unrelated types to implement shared behavior without relying on a common superclass.

**Static vs. Dynamic Dispatch:** Polymorphism in Java or C++ typically uses dynamic dispatch, where the appropriate method implementation is determined at runtime based on the actual type of the object. Traits in Rust, by default, use static dispatch, where the appropriate method implementation is determined at compile-time based on the type information.

**Open vs. Closed:** Inheritance-based polymorphism allows for open extension, where new subclasses can be added to extend the behavior of existing classes. Traits, in contrast, are closed for extension. Once a type is defined, its trait implementations are fixed. However, you can create new traits and implement them for existing types, allowing for ad hoc extension.

**Multiple Inheritance:** Polymorphism in Java or C++ supports multiple inheritance, where a class can inherit from multiple superclasses. Traits in Rust do not support multiple inheritance. Instead, Rust encourages using trait bounds and associated types to achieve similar functionality while avoiding the complexities of multiple inheritance.

**Zero-cost Abstractions:** Traits in Rust are designed to support zero-cost abstractions, meaning that using traits imposes no runtime overhead compared to concrete types. The trait methods are statically dispatched, and the compiler can optimize the code based on the concrete types implementing the trait.

### Code Example
In the Java Example the Animal class defines a common interface (makeSound method) that is shared by its subclasses Dog and Cat. Objects of different classes (Dog and Cat) can be treated as objects of the common superclass Animal. This enables dynamic method dispatch, where the appropriate method implementation is determined at runtime based on the actual type of the object.
```java
// Polymorphism in Java
class Animal {
    public void makeSound() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {
    public void makeSound() {
        System.out.println("Dog barks");
    }
}

class Cat extends Animal {
    public void makeSound() {
        System.out.println("Cat meows");
    }
}

public class PolymorphismExample {
    public static void main(String[] args) {
        Animal animal1 = new Dog();
        Animal animal2 = new Cat();

        animal1.makeSound(); // Output: Dog barks
        animal2.makeSound(); // Output: Cat meows
    }
}
```
In the Rust example, traits are used for ad hoc polymorphism. The Animal trait defines a set of methods (in this case, only make_sound) that can be implemented by different types (Dog and Cat). The types are not tied to a specific inheritance hierarchy, allowing any type to implement the Animal trait as long as it provides the required methods. This decoupling of traits from types allows for greater flexibility and composability. Objects of different types can be treated as objects implementing the trait, enabling dynamic dispatch of the trait methods at runtime.
```rust
trait Animal {
    fn make_sound(&self);
}

struct Dog;
struct Cat;

impl Animal for Dog {
    fn make_sound(&self) {
        println!("Dog barks");
    }
}

impl Animal for Cat {
    fn make_sound(&self) {
        println!("Cat meows");
    }
}

fn main() {
    let animal1: Box<dyn Animal> = Box::new(Dog);
    let animal2: Box<dyn Animal> = Box::new(Cat);

    animal1.make_sound(); // Output: Dog barks
    animal2.make_sound(); // Output: Cat meows
}
```	

So as we see, traits provide a powerful and flexible mechanism for achieving polymorphic behavior in Rust. They combine the best of object-oriented and functional programming paradigms, enabling developers to write clean, reusable, and performant code. 