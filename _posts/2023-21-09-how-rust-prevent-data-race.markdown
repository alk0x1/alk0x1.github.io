---
layout: post
title:  "How Rust prevent data races"
date:   2023-09-21 15:00:50 -0300
categories: ["Rust", "Computer Science"]
---


One of the main concurrency problem that programmers face is the Data Race problem.
Basically a Data Race occurs when two threads access the same mutable object and while one of the thread is accessing the object the other thread is writting on it.

![](/assets/images/datarace.png)

Let's take this python code to examplify an data race problem:
```python	
import threading

# Shared counter variable
counter = 0

# Function to increment the counter
def increment_counter():
    global counter
    for _ in range(1000000):
        counter += 1

# Create two threads that concurrently increment the counter
thread1 = threading.Thread(target=increment_counter)
thread2 = threading.Thread(target=increment_counter)

# Start both threads
thread1.start()
thread2.start() 

# Wait for both threads to finish
thread1.join()
thread2.join()

# Print the final value of the counter
print("Final Counter Value:", counter)
```
In this example, two threads (thread1 and thread2) are created to increment the counter variable concurrently. They both run the increment_counter function, which loops 1,000,000 times, incrementing the counter by 1 in each iteration.
As a result, the final value of the counter is unpredictable, and it will almost certainly be less than 2,000,000.

#### Data Race prevention in Rust
Unlike the most multi-threaded languages, Rust make it significantly harder to write code that results in data races, thanks to the Ownership and Borrowing functionalities. The rust version of the above Python code give us an compiler error:

```rust 
fn main() {
    // Shared counter variable
    let mut counter = 0;

    // Function to increment the counter
    fn increment_counter(counter: &mut i32) {
        for _ in 0..1_000_000 {
            *counter += 1;
        }
    }

    // Create two threads that concurrently increment the counter
    let thread1 = std::thread::spawn(|| {
        increment_counter(&mut counter);
    });

    let thread2 = std::thread::spawn(|| {
        increment_counter(&mut counter);
    });

    // Wait for both threads to finish
    thread1.join().unwrap();
    thread2.join().unwrap();

    // Print the final value of the counter
    println!("Final Counter Value: {}", counter);
}
```
```rust 
error[E0502]: cannot borrow `counter` as mutable because it is also borrowed as immutable
  --> main.rs:13:5
   |
11 |     let thread1 = std::thread::spawn(|| {
   |                  ------------------ immutable borrow occurs here
12 |         increment_counter(&mut counter);
13 |     });
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ mutable borrow occurs here
14 | 
15 |     let thread2 = std::thread::spawn(|| {
   |                  ------------------ immutable borrow ends here

```
As we can see, Rust don't let we use the same mutable variable twice in the same scope, which prevent Data Races problems in compiler time.