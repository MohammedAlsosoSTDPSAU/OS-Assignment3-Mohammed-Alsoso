# Assignment 3 - Complete Documentation

**Student Name**: [Mohammed_Alsoso]  
**Student ID**: [444052901]  
**Date Submitted**: [2026/05/02]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
>
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: https://drive.google.com/file/d/1zbn__K-EX-WG-g8_tPNFVT4qzYXl50X-/view?usp=sharing
**Video filename**: `444052901_Assignment3_Synchronization.mp4`

**Verification**:

- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [Date, Time]

**What I implemented**: Set up the repository, changed visibility, and updated the student ID to 444052901.

**Challenges encountered**: The "Change visibility" button was disabled.

**How I solved it**: Realized that a forked repository from a public upstream is already public by default, so I just proceeded to rename the repo and clone it.

**Testing approach**: Ran the base code to ensure it compiles before making functional changes.

**Time spent**: 7 mins

---

### Entry 2 - [Date, Time]

**What I implemented**: Task 1 - Implemented Fine-Grained locking for the shared counter variables (contextSwitchCount, completedProcessCount, totalWaitingTime).

**Challenges encountered**: Deciding between a single coarse-grained lock or multiple fine-grained locks.

**How I solved it**: I created separate ReentrantLock objects for each counter because they operate independently, avoiding unnecessary bottlenecks.

**Testing approach**: Checked if the counters increment accurately without losing updates during concurrent thread execution.

**Time spent**: 30 mins

---

### Entry 3 - [Date, Time]

**What I implemented**: Task 2 - Protected the executionLog ArrayList using a dedicated ReentrantLock (logLock).

**Challenges encountered**: ArrayLists are notoriously thread-unsafe and can cause crashes.

**How I solved it**: Wrapped the executionLog.add(message) inside a try-finally block ensuring the lock is released properly.

**Testing approach**: Ran the simulation to ensure no ConcurrentModificationException is thrown when multiple processes yield or start execution simultaneously.

**Time spent**: 30 mins

---

### Entry 4 - [Date, Time]

**What I implemented**: Task 3 - Implemented a Binary Semaphore (Semaphore(1)) to control CPU access in run() and runToCompletion().

**Challenges encountered**: Ensuring the semaphore is always released, even if a thread gets interrupted.

**How I solved it**: I wrapped the entire execution logic inside a try block immediately after acquire(), and placed release() inside the finally block.

**Testing approach**: Observed the console output to confirm that only one process prints its "quantum progress" bar at any given time.

**Time spent**: 20 mins

---

### Entry 5 - [Date, Time]

**What I implemented**: Final code review and compilation troubleshooting.

**Challenges encountered**: Faced 9 compilation errors after manual code editing due to missing imports and mismatched curly braces {}.

**How I solved it**: USING AI TOOL --> Re-structured the try-catch-finally blocks cleanly and ensured java.util.concurrent.Semaphore and locks.ReentrantLock were properly imported

**Testing approach**: Ran the full simulation 5 times to ensure results are consistent and no deadlocks occur.

**Time spent**: 15 mins

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions

**Q**: Identify and explain TWO race conditions in the original code. For each:

- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[Your answer here - 4-6 sentences with code examples]

## The first race condition affects the primitive counter variables like contextSwitchCount. Because the increment operation (contextSwitchCount++) is not atomic (it reads, modifies, and writes), concurrent access allows threads to read the same base value simultaneously, resulting in lost updates and an inaccurate final count. The second race condition affects the executionLog, which is an ArrayList. Since standard ArrayLists are not thread-safe, concurrent calls to executionLog.add(message) can cause internal array resizing collisions, leading to overwritten data or throwing a ConcurrentModificationException that crashes the program.

### Question 2: Locks vs Semaphores

**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[Your answer here - explain your implementation choices]

## A ReentrantLock provides mutual exclusion by allowing only the single thread that acquired the lock to execute the critical section and release it. A Semaphore uses a permit-based system to control how many threads can access a resource simultaneously, and permits can be released by any thread. In my code, I used ReentrantLock to protect data states (the counters and the execution log) because only one thread must modify that specific memory at a time. I used a binary Semaphore(1) for CPU execution control because it perfectly models a scheduler, forcing processes to wait in a queue for the single CPU permit before running their quantum.

### Question 3: Deadlock Prevention

**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[Your answer here - reference try-finally blocks, lock ordering, etc.]

## A deadlock is a situation where two or more threads are permanently blocked, each waiting for a resource that the other holds. Two prevention techniques include "Guaranteed Release" (ensuring a lock is freed even if an exception occurs) and "Avoiding Nested Locks" (not requesting a new lock while already holding one). To prevent deadlocks in my code, I applied the guaranteed release technique by placing every unlock() and release() statement strictly inside a finally block (e.g., try { lock.lock(); ... } finally { lock.unlock(); }), which ensures resources are always freed even if a thread gets interrupted.

### Question 4: Lock Granularity Design Decision

**Q**: For Task 1 (protecting the three counters), explain your lock design choice:

- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[Your answer here - explain coarse-grained vs fine-grained locking, independence of counters, concurrency implications. Show understanding of when to use each approach. 5-8 sentences expected.]

## For Task 1, I chose to implement fine-grained locking by creating separate ReentrantLock objects for each of the three counters. I made this choice because the counters (contextSwitchCount, completedProcessCount, and totalWaitingTime) are completely independent variables that track different metrics. The trade-off is that coarse-grained locking (one lock for all) is easier to code but creates severe bottlenecks, whereas fine-grained locking requires more code but drastically improves system performance. Because these counters are independent, fine-grained locking provides much better concurrency. It allows one thread to increment the context switch counter while another thread simultaneously updates the completed process counter without unnecessarily blocking each other.

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: contextSwitchCount, completedProcessCount, and totalWaitingTime.

**Why they need protection**: Operations like incrementing (++) and adding (+=) are not atomic (they involve read, modify, and write steps). If multiple threads execute them concurrently, it leads to lost updates and incorrect final calculations.

**Synchronization mechanism used**: Fine-Grained Mutex Locks (ReentrantLock).

**Code snippet**:

```java
public static void incrementContextSwitch() {
        contextSwitchLock.lock();
        try {
            contextSwitchCount++;
        } finally {
            contextSwitchLock.unlock();
        }
    }
```

**Justification**: ReentrantLock ensures mutual exclusion, meaning only one thread can modify the counter at a time. Using a finally block guarantees that the lock is always released, preventing deadlocks.

---

### Critical Section #2: Execution Log

**What resource**: executionLog (which is an ArrayList<String>).

**Why it needs protection**: The standard ArrayList in Java is not thread-safe. Concurrent additions can cause the internal array to resize improperly, leading to overwritten data or throwing a fatal ConcurrentModificationException.

**Synchronization mechanism used**: Mutex Lock (ReentrantLock named logLock).

**Code snippet**:

```java
public static void logExecution(String message) {
        logLock.lock();
        try {
            executionLog.add(message);
        } finally {
            logLock.unlock();
        }
    }
```

**Justification**: The lock prevents concurrent structural modifications to the ArrayList, ensuring that each log entry is written safely one after the other into memory.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To control access to the simulated CPU, ensuring that processes execute their scheduled time quantum without overlapping with others.

**Number of permits and why**: 1 permit (Binary Semaphore). This is because we are simulating a single-core CPU scheduler, meaning only one process can execute at any given exact millisecond.

**Where implemented**: At the very beginning and end of the run() and runToCompletion() methods in the Process class.

**Code snippet**:

```java
try {
            SharedResources.cpuSemaphore.acquire();  // Wait for CPU
            try {
                // ... (Process execution and quantum progress bar logic) ...
            } finally {
                SharedResources.cpuSemaphore.release();  // Always release CPU!
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
```

**Effect on program behavior**: It successfully stops threads from printing their progress bars simultaneously. It forces them to wait in a queue and execute sequentially based on the scheduler's logic, creating a clean, realistic CPU execution timeline in the terminal.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check

**What I tested**: Running the synchronized scheduler program multiple times to verify consistent execution, accurate counter updates, and orderly console output without deadlocks.

**Testing procedure**:

```bash
# Commands used (run the program at least 5 times)
```

**Results**:
Running the program multiple times produces consistent, correct results. As seen in the output, exactly 20 processes were created and 20 processes successfully completed. The output remained perfectly sequential with no mixed or overlapping text, and the counters (37 Context Switches, 74 Log entries) were accurately tallied without any program crashes or freezes.

**Why synchronization is necessary**: Without synchronization, several severe race conditions would occur. The shared primitive counters (contextSwitchCount, completedProcessCount) would suffer from "lost updates" due to simultaneous thread access. The executionLog (an ArrayList) would likely throw a ConcurrentModificationException when multiple threads attempt to write to it at the exact same time. Finally, without the CPU semaphore, all threads would try to execute and print their progress bars to the console simultaneously, resulting in scrambled, chaotic output rather than a realistic sequential CPU simulation.

**Conclusion**: The implementation of fine-grained ReentrantLocks and the binary Semaphore successfully resolves all race conditions, ensuring thread-safe data modification and a completely stable simulation across multiple runs.

---

### Test 2: Exception Testing

**What I tested**: Checking for ConcurrentModificationException in the `executionLog` ArrayList.

**Testing procedure**: Ran the simulation with 20 concurrent processes, which creates high thread contention and rapid context switching, to force simultaneous writes to the log.

**Results**: The program executed to the end without throwing any `java.util.ConcurrentModificationException` or crashing.

**What this proves**: This proves that the `logLock` (`ReentrantLock`) successfully queued the threads and protected the ArrayList, making all memory writes completely thread-safe.

---

### Test 3: Correctness Verification

**What I tested**: Verifying correct final values for the total completed processes.

**Expected values**: Total Completed Processes should be exactly 20 (since 20 processes were added to the ready queue).

**Actual values**: Total Completed Processes: 20 (as confirmed in the console output summary).

**Analysis**: The `completedProcessLock` worked perfectly. Without it, concurrent increments (`completedProcessCount++`) would cause lost updates, and the actual value would have been less than 20. The final match proves the lock prevented all race conditions on the counter.

---

### Test 4: Different Scenarios

**Scenario tested**: Changing the CPU semaphore from a binary semaphore `Semaphore(1)` to a counting semaphore `Semaphore(2)`.

**Purpose**: To observe the difference between a single-core simulation (1 permit) and a multi-core/parallel simulation (2 permits).

**Results**: The console output became scrambled and chaotic. Two different processes started executing and printing their "quantum progress" bars at the exact same time, causing the terminal characters to interleave and mix together.

**What I learned**: I learned practically how semaphores control execution flow. A binary semaphore forces strict sequential access (mutual exclusion), whereas increasing the permits allows true parallel execution, which requires even more careful synchronization for shared resources like the terminal output.

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

Through this assignment, I learned that writing multithreaded applications requires a completely different mindset compared to sequential programming. I discovered how easily unseen race conditions can corrupt shared data, such as our counter variables, when operations are not strictly atomic. Implementing `ReentrantLock` taught me how to enforce mutual exclusion to protect critical sections of memory safely. I also learned the crucial practical difference between locks (used for protecting data state) and semaphores (used for controlling execution flow and resource availability). One major challenge was ensuring that deadlocks do not occur, which highlighted the absolute necessity of using `try-finally` blocks to guarantee resource release. Ultimately, this assignment provided deep insights into how real operating systems manage concurrent processes efficiently without crashing.

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: **Banking and Financial Systems.** When processing thousands of simultaneous transactions, locks must be used to protect account balances. Without synchronization, concurrent deposits and withdrawals would overwrite each other (lost updates), causing severe financial errors.

**Example 2**: **Online Ticket Booking Systems.** When booking flights or concert tickets, semaphores and mutex locks ensure that if thousands of users try to book the exact same seat at the same millisecond, only the first thread gets the permit, preventing double-booking.

---

### How I would explain synchronization to others:

Imagine a busy restaurant kitchen with multiple chefs (threads) trying to use a single oven (CPU). Without rules, they would pull each other's food out early or bump into each other (race condition). Synchronization is like giving the chefs a strict management system: a "Lock" is a physical key to the spice cabinet so only one person can change the recipe at a time, preventing ruined food. A "Semaphore" is a queue manager ensuring only one chef uses the oven until their dish is fully cooked. It's simply a set of rules so programs can share resources politely without causing a mess.

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/MohammedAlsosoSTDPSAU/OS-Assignment3-Mohammed-Alsoso.git

**Number of commits**: 4

**Commit messages**:

1.Initial setup: updated student ID to 444052901 and verified base code. 2. Task 1: Implemented Fine-Grained ReentrantLocks for shared counters 3. Task 2: Secured executionLog ArrayList with a dedicated ReentrantLock 4. Task 3: Added Binary Semaphore to control CPU scheduling execution

---

## Summary

**Total time spent on assignment**: Approximately 3 hours.

**Key takeaways**:

1. Always place `unlock()` and `release()` methods inside a `finally` block to absolutely guarantee deadlock prevention.
2. Fine-grained locking (multiple specific locks) drastically improves application performance and concurrency compared to coarse-grained locking (one massive lock).
3. Standard Java data structures like `ArrayList` are not thread-safe and will crash the system if accessed concurrently without strict synchronization.

**Most challenging aspect**: Identifying the precise starting and ending boundaries of the critical sections. Locking too much code would ruin the simulation's performance, while locking too little would leave race conditions exposed.

**What I'm most proud of**: Successfully implementing the binary semaphore and fine-grained locks to produce perfectly consistent, error-free console output and accurate counter statistics across multiple consecutive runs.

---

**End of Documentation**
