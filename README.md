# Project1: File-Processing-System

Student Name: Tahira Malik

CMPSC 472: Operating Systems

Instructor: Janghoon Yang

## Project Overview:

This project is designed to develop a system for efficiently processing large text files and count the frequency of a specific word or multiple words "the", across multiple files, using multiprocessing and multithreading. Each file is processed in a separate child process created using fork(). Inside each process, multiple threads are used to handle different portions of the file, which enables parallel processing of large data. The system also implements an inter-process communication (IPC) mechanism, allowing child processes to send their results to the parent process by using shared memory.

This project allows for comparing the efficiency and resource utilization of multiprocessing versus multithreading in file processing tasks. The results are evaluated based on execution time, CPU usage, and memory consumption, helping to understand the trade-offs between these two approaches.

## Technologies Used:

Programming Language: C

Libraries:

Standard C libraries for file handling and memory management.

POSIX threads (pthreads) for multithreading (to be implemented).

System calls for process management and IPC (shared memory).


## Implementation Process

### 1. File Setup:

Calgary folder is saved in Google Drive, I used python to access it.

![image](https://github.com/user-attachments/assets/740cef80-37e6-4de9-9b8b-7f4fbccef2bb)

A collection of 7 files from the Calgary Corpus was selected for processing. Each file was assigned to a separate process.

![image](https://github.com/user-attachments/assets/976b4572-a1d3-4abb-bb6a-72f97c847ace)


![image](https://github.com/user-attachments/assets/850bd39e-d296-44fa-945e-fa7bbdd013d4)

![image](https://github.com/user-attachments/assets/0baecea9-dbbb-46bc-8776-3490d53c0b39)


### 2. Multiprocessing:

The parent process uses fork() to create a child process for each file which is to perform tasks concurrently
The parent process is the process that initiates and oversees the entire word counting operation.
The parent process itself does not directly count the words. Instead, it delegates this task to its child processes and waits for them by using wait() to finish.
After creating the child processes, the parent waits for all of them to finish using wait(NULL).

Child Processes:

Each child process is a separate process created by the parent to handle one specific task in this case, counting words in a file.
After a successful fork, the child process performs the word counting in the assigned file by calling count_word_in_file(&file_wc[i]).
After completing the word count task, the child process terminates itself by calling exit(0).
Example:
I have 7 files and the program is using multiprocessing, the parent process will create 7 child processes. Each child process will be responsible for reading one file and counting the occurrences of the specified word in that file.


### 3. Multithreading:

Thread Function Definition:

void *thread_count_word(void *arg): This function is called for each thread created. It takes a pointer to a WordCount struct as an argument, which contains the filename and word to count.

count_word_in_file((WordCount *)arg): This line calls the count_word_in_file function, passing the argument which is cast to the correct type. This function counts the occurrences of the specified word in the corresponding file.

Creating Threads:

pthread_t threads[NUM_FILES]: This declares an array of thread identifiers to keep track of the threads created.
pthread_create(&threads[i], NULL, thread_count_word, &wc[i]): This line creates a new thread for each file. It passes the address of the thread identifier, specifies default thread attributes (NULL), points to the thread function (thread_count_word), and passes the corresponding WordCount struct (&wc[i]) as an argument.

Waiting for Threads to Complete:

pthread_join(threads[i], NULL): This line ensures that the main thread waits for each created thread to finish execution before continuing. This is important for proper synchronization and to ensure all word counts are completed before the program ends.
In this code, the parent or main thread creates multiple threads instead of child processes. It creates multiple threads using pthread_create(), each responsible for processing one file. Each thread invokes count_word_in_file, which handles the counting of the specified word in its assigned file.



### 4. Inter-Process Communication (IPC):

- Shared Memory Setup:
The mmap function is used to create a shared memory segment, allowing multiple processes to access the same data.

The shared data structure (SharedData) contains:
word_count: An array to hold the count of the target word for each file.

total_word_count: An array to hold the total counts of all unique words across all files.

words: An array to hold unique words encountered in the files.

unique_words: A counter for the number of unique words.

- Forking Processes:

The code forks a new child process for each file in the file_wc array.
Each child process executes the count_word_in_file function to count occurrences of the target word and update the shared memory with unique word counts
### 5. Unmapping Shared Memory:
I added the munmap() call at the end of multiprocess_with_shared_memory(). This ensures that the shared memory is released once it's no longer needed. If munmap() fails, it reports the error with perror().

Error Handling: munmap() is wrapped in an error-handling block. If it returns -1, an error is printed using perror("munmap").

Why It’s Important:
Memory Management: Without munmap(), the shared memory region is not released, leading to memory leaks.
Good Practice: Always releasing allocated resources ensures the program doesn't consume unnecessary system resources.

### 6. Evaluation:

Both approaches were compared by measuring the time taken for word counting, as well as the CPU and memory usage for each approach.

## Implemented Features
### 1. Process Creation:

The system uses fork() to create separate child processes, with each process handling a single file from the Calgary Corpus.
### 2. Multithreading:

Each file is split into chunks and assigned to multiple threads within a process, allowing for parallel word counting within a file.

![image](https://github.com/user-attachments/assets/6339c9e7-f021-4391-b818-e87b727fb6b8)

### 3. Inter-Process Communication (IPC):
Overview of IPC in the Code
Shared Memory Setup:

The mmap function is used to create a shared memory segment, allowing multiple processes to access the same data.

The shared data structure (SharedData) contains:

word_count: An array to hold the count of the target word for each file.

total_word_count: An array to hold the total counts of all unique words across all files.

words: An array to hold unique words encountered in the files.

unique_words: A counter for the number of unique words.

![image](https://github.com/user-attachments/assets/e29e4369-cd9c-48fd-abb5-9be56cddbe61)


Forking Processes:

The code forks a new child process for each file in the file_wc array.
Each child process executes the count_word_in_file function to count occurrences of the target word and update the shared memory with unique word counts.

![image](https://github.com/user-attachments/assets/739bff2e-c8d1-407e-afb9-fa1cc2b1177f)


Counting Logic:

Each child process opens its assigned file and reads words one by one.
For each word, it checks if it matches the target word and increments the count.
It also updates the shared memory with the total count of all unique words across files, ensuring that multiple processes can access and update this data.
![image](https://github.com/user-attachments/assets/43535956-b918-41fd-ad9a-143421f770fb)

Synchronization:

In this example, there is no explicit synchronization mechanism (like semaphores or mutexes) used in the counting logic. However, since the child processes do not read/write to shared memory simultaneously, it may work without immediate issues.
It's essential to ensure that shared memory access is synchronized if there are concurrent modifications to avoid race conditions.

Waiting for Child Processes:

After forking, the parent process waits for all child processes to finish using wait(NULL). This ensures that the parent only continues once all counting operations are complete.
![image](https://github.com/user-attachments/assets/f6c81f06-05fb-4d4c-9e9c-0a9b37173c15)

Sorting and Displaying Results:

Once all child processes are done, the parent collects the results from shared memory, sorts the words based on their frequency, and prints the top words.
![image](https://github.com/user-attachments/assets/87cf11d2-2f80-468c-b6cc-0700e0d046a4)

Cleanup:

After using shared memory, the munmap function is called to unmap the shared memory segment, releasing the resources.
![image](https://github.com/user-attachments/assets/291d5067-6e9f-4459-9f5e-37ec72df1368)

Summary
In summary, this code effectively demonstrates IPC using shared memory by allowing multiple processes to count word occurrences in parallel. It sets up shared memory to store results, forks processes for concurrent file processing, and collects results once all processes are complete. However, consideration of implementing explicit synchronization mechanisms to handle concurrent access safely in more complex scenarios.

### 4. Error Handling:

This system includes error handling for file Opening, memory mapping, process creation (fork()), and IPC setup.

1.  File Opening:

When system attempts to open a file, it also checks if the file pointer is NULL. If it is, then there is an error message using perror() and exit the function.
![image](https://github.com/user-attachments/assets/aa1fb0f8-2e6e-4116-98cc-2be95a657de1)

2.  Memory Mapping:

When using mmap to create a shared memory segment, System checks if the return value is MAP_FAILED. If it fails, then it prints an error message and exit the program.
![image](https://github.com/user-attachments/assets/110ea704-b012-4d16-a157-b2371fb11a8c)

3.  Process Creation:

System checks the return value of fork(). If fork() returns a negative value, it indicates that the process creation failed, and it prints an error message and exit the program.
![image](https://github.com/user-attachments/assets/3a14905d-b3c5-42d3-af73-d6266696795f)

4.  Unmapping Shared Memory:

This system checks if munmap returns -1, which indicates a failure to unmap the shared memory. In that case, you print an error message.
c
Copy code
if (munmap(shared_data, sizeof(SharedData)) == -1) {
    perror("munmap");
}
Recommendations for Improvement
While your current error handling addresses several critical areas, you could enhance it further. Here are some suggestions:

Graceful Exit:

Instead of using exit(1) on errors, consider returning an error code to the calling function. This allows for cleaner exits and potential logging of errors. In main(), you can then decide how to handle these errors (e.g., clean up resources, log errors).
Detailed Error Messages:

When printing error messages, include context about where the error occurred. This can be particularly useful for debugging. For example, in your perror messages, include additional context such as which file failed to open.
Resource Cleanup:

If an error occurs after allocating resources (like shared memory), ensure that you clean up any allocated resources before exiting. For example, if mmap fails after forking some processes, those processes should be waited on and terminated properly.
Handle Invalid Input:

Validate the input parameters and data. For instance, check if the number of files defined (NUM_FILES) is reasonable, or if the file paths are valid before attempting to open them.
Signal Handling:

Consider implementing signal handlers for SIGCHLD to handle child termination and other signals gracefully, particularly in a multi-process environment.
Error Logging:

Instead of just printing errors to the console, consider logging them to a file for later review. This can help in tracking down issues in production environments.
Example of Enhanced Error Handling
Here’s a snippet showing enhanced error handling when opening a file:

c
Copy code
FILE *file = fopen(file_wc->filepath, "r");
if (file == NULL) {
    fprintf(stderr, "Error: Failed to open file '%s': ", file_wc->filepath);
    perror("fopen");
    return; // or handle the error accordingly
}
Summary
Your code has a solid foundation for error handling, particularly in critical areas like file operations and memory management. By implementing the recommended improvements, you can make your error handling more robust, informative, and conducive to maintaining the program's stability and reliability.
### 5. Performance Metrics:

Execution time, CPU usage, and memory consumption were recorded and compared for both multiprocessing and multithreading approaches.
### 6. Word Frequency Counting:

The system correctly counts the frequency of a specified word or set of words in each file.
