# Thread-Safe File Logging in C++: Best Practices and Common Pitfalls
This repository demonstrates how to perform file logging in a multithreaded C++ application, focusing on best practices for thread-safe operations and common pitfalls that can lead to race conditions and data corruption.

# Contents
Race Conditions: Examples that highlight where race conditions can occur in multithreaded file logging.

Using Mutexes: Demonstrates how to use mutexes to synchronize file access across different threads.

Anti-Patterns: Illustrates dangerous practices that should be avoided to prevent data races.

std::call_once : I used call_once in this code to ensure that the process of opening the file is executed only once, regardless of how many threads are accessing it.

# std::call_once
I used call_once in this code to ensure that the process of opening the file is executed only once, regardless of how many threads are accessing it. This guarantees that the file is opened properly without the risk of concurrent attempts to open it by multiple threads, providing thread-safe initialization.

# Race Conditions
Race conditions occur when multiple threads access shared resources simultaneously, leading to unpredictable and undesirable behavior. In the context of file logging, this can result in interleaved or corrupted file content.

# Using Mutexes
Mutexes are used to ensure that only one thread at a time can write to a file, preventing the aforementioned race conditions. This section demonstrates how to use std::lock_guard with std::mutex to manage access to shared resources.
# Anti-Patterns
Certain practices, if not handled correctly, can lead to data races and other synchronization issues. This section focuses on what not to do when dealing with multithreaded file logging.
1. # Exposing Internal Resources
   In the code, the method get_ofstream returns a reference to the internal std::ofstream object. This exposes the internal resource to the outside world, allowing unauthorized access to it.
   This is an anti-pattern because it bypasses the mutex that is supposed to control access to the ofstream. Once an external user gets a reference to the ofstream,
   they can write to it directly without going through the lock, leading to potential race conditions.
2. # Bypassing Mutexes
   The process_ofstream method takes a function pointer that accepts an ofstream reference, and it directly passes the internal ofstream to that function without going through the mutex.
   This is an anti-pattern because it completely bypasses the lock mechanism and allows unauthorized manipulation of the ofstream object, as demonstrated in the malicious function. This approach can lead to data corruption and race conditions.

# Observations and Recommendations:

Using std::call_once: std::call_once ensures that the file is opened only once. This is a good use of the function to prevent the file from being opened multiple times. However, remember that any errors that occur during the call to std::call_once will propagate and the flag will be set, preventing subsequent calls, even if the first call failed.

Thread Safety: The two methods write_into_file and write_into_file_V2 use mutexes to guard against concurrent access, which is good for ensuring thread safety. But they're essentially doing the same thing, just with different methods of writing to the file.

Duplicated File Opening Code: The code to open the file inside std::call_once is duplicated in both write_into_file and write_into_file_V2. You can create a private member function, say open_file(), and call it inside std::call_once to avoid this repetition.

Resource Leak: Your logger uses RAII (Resource Acquisition Is Initialization) for managing the file's lifecycle, which is a good practice. The destructor of LogFile ensures the file is closed.
