> # Exercises 1

## Exercise 1.1

Consider the lecture’s `LongCounter` example found in file `TestLongCounterExperiments.java`, and **remove** the `synchronized` keyword from method `increment` so you get this class:

```java
class LongCounter {
    private long count = 0;
    public void increment() {
        count = count + 1;
    }
    public synchronized long get() {
        return count;
    }
}
```

1. The `main` method creates a `LongCounter` object. Then it creates and starts two threads that run concurrently, and each increments the `count` field 10 million times by calling method `increment`. What kind of final values do you get when the increment method is **not** synchronized?

**Answer: **The output ranges between 10000000 and 20000000 inclusive because of the race condition.

We can use the following code for testing:

```java
public class TestLongCounterExperiments {
    public static void main(String[] args) {
        final LongCounter lc = new LongCounter();
        final int counts = 10000000;

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < counts; ++i) 
                lc.increment();
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < counts; ++i) 
                lc.increment();
        });

        t1.start(); t2.start();
        
        try { t1.join(); t2.join(); }
        catch (InterruptedException exn) { 
            System.out.println("Some thread was interrupted");
        }
        System.out.println("Count is " + lc.get() + " and should be " + 2*counts);
    }
}

class LongCounter {
    private long count = 0;
    public void increment() {
        count = count + 1;
    }
    public synchronized long get() { 
        return count; 
    }
}
```

The outputs are:

```
Count is 20000000 and should be 20000000
Count is 19863149 and should be 20000000
Count is 19903942 and should be 20000000
```

2. Reduce the `counts` value from 10 million to 100, recompile, and rerun the code. It is now likely that you get the correct result (200) in every run. Explain how this could be. Would you consider this software correct, in the sense that you would guarantee that it always gives 200?

**Answer: ** We can not guarantee the result is always 200. It may be that the first thread has finished before the second thread starts.

3. The `increment` method in `LongCounter` uses the assignment

```java
count = count + 1;
```

to add one to `count`. This could be expressed also as `count += 1` or as `count++`.
Do you think it would make any difference to use one of these forms instead? Why? Change the code and run it, do you see any difference in the results for any of these alternatives?

**Answer: **Race condition still happens. `+=` operator equals `count = count + 1`。`count++` is same like that. The difference between them is only  syntactic difference.

4. Extend the `LongCounter` class with a `decrement()` method which subtracts 1 from the `count` field. Change the code in `main` so that `t1` calls `decrement` 10 million times, and `t2` calls `increment` 10 million times, on a `LongCounter` instance. In particular, initialize `main`’s `counts` variable to 10 million as before. What should the final value be, after both threads have completed? Note that decrement is called only from one thread, and increment is called only from another thread. So do the methods have to be synchronized for the example to produce the expected final value? Explain why (or why not).

**Answer: **If we don't use synchronized for `increment` and `decrement`:

```java
public class TestLongCounterExperiments {
    public static void main(String[] args) {
        final LongCounter lc = new LongCounter();
        final int counts = 10000000;

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < counts; ++i) 
                lc.decrement();
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < counts; ++i) 
                lc.increment();
        });

        t1.start(); t2.start();
        
        try { t1.join(); t2.join(); }
        catch (InterruptedException exn) { 
            System.out.println("Some thread was interrupted");
        }
        System.out.println("Count is " + lc.get() + " and should be " + 0);
    }
}

class LongCounter {
    private long count = 0;
    public void increment() {
        count = count + 1;
    }
    public void decrement() {
        count = count - 1;
    }
    
    public synchronized long get() { 
        return count; 
    }
}
```

The outputs are:

```
Count is -53460 and should be 20000000
Count is -683 and should be 20000000
Count is -4940998 and should be 20000000
```

From results above, we can see race condition happens. So if we want to get the expected value, we should use synchronized.

```java
public class TestLongCounterExperiments {
    public static void main(String[] args) {
        final LongCounter lc = new LongCounter();
        final int counts = 10000000;

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < counts; ++i) 
                lc.increment();
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < counts; ++i) 
                lc.decrement();
        });

        t1.start(); t2.start();
        
        try { t1.join(); t2.join(); }
        catch (InterruptedException exn) { 
            System.out.println("Some thread was interrupted");
        }
        System.out.println("Count is " + lc.get() + " and should be " + 2*counts);
    }
}

class LongCounter {
    private long count = 0;
    public synchronized void increment() {
        count = count + 1;
    }
    public synchronized void decrement() {
        count = count - 1;
    }
    
    public synchronized long get() { 
        return count; 
    }
}
```

This time we can get `0`.

5. Make four experiments:

   * Run the example without synchronized on any of the methods;

   If we remove synchronized, it makes no difference. Because when we use `get` method, two threads has finished.

   * with only `decrement` being synchronized;

   Only `decremnet` being synchronized means we can make sure that `count` variable is decreased by 10 million times. But we can not make sure that `count` variable is increased by 10 million times. So the result is lower than 0.

   * with only `increment` being synchronized;

   It is same as the question before. The result is higher than 0.

   * with both being synchronized.

## Exercise 1.2

Consider this class, whose `print`method prints a dash “-”, waits for 50 milliseconds, and then prints a vertical bar “|”:

```java
class Printer {
    public void print() {
        System.out.print("-");
        try { Thread.sleep(50); } catch (InterruptedException exn) { }
        System.out.print("|");
    }
}
```

1. Write a program that creates a Printer object p, and then creates and starts two threads. Each thread must call `p.print()` forever. You will observe that most of the time the dash and bar symbols alternate neatly as in `-|-|-|-|-|-|-|`. But occasionally two bars are printed in a row, or two dashes are printed in a row, creating small “weaving faults” like those shown below:

```
|-|-|-|-|-|-||--|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-||--
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
|-|-|-|-|-|-|-|-|-|-|-|-||--|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
```

Since each thread always prints a dash after printing a bar, and vice versa, this phenomenon can be caused only by one thread printing a bar and then the other thread printing a bar before the first one gets to print its dash. Describe a scenario involving the two threads where this happens.

```java
public class TestPrinterExperiments {
    public static void main(String[] args) {
        final Printer p = new Printer();

        Thread t1 = new Thread(() -> {
            while (true) {
            	p.print();
            }
        });

        Thread t2 = new Thread(() -> {
            while (true) {
            	p.print();
            }
        });

        t1.start(); t2.start();

        try { t1.join(); t2.join(); }
        catch (InterruptedException exn) { 
            System.out.println("Some thread was interrupted");
        }
    }
}

class Printer {
    public void print() {
        System.out.print("-");
        try { Thread.sleep(50); } catch (InterruptedException exn) { }
        System.out.print("|");
    }
}
```

2. Making method `print` synchronized should prevent this from happening. Explain why. Compile and run the improved program to see whether it works.
3. Rewrite `print` to use a synchronized statement in its body instead of the method being synchronized.

```java
class Printer {
    private final Object innerLock = new Object();
    public void print() {
        synchronized(innerLock) {
            System.out.print("-");
            try { Thread.sleep(50); } catch (InterruptedException exn) { }
            System.out.print("|");
        }
    }
}
```

4. Make the `print` method static, and change the `synchronized` statement inside it to lock on the Printer class’s reflective Class object instead. For beauty, you should also change the threads to call static method `Printer.print()` instead of instance method `p.print()`.

```java
class Test {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            while (true) {
            	Printer.print();
            }
        });

        Thread t2 = new Thread(() -> {
            while (true) {
            	Printer.print();
            }
        });

        t1.start(); t2.start();

        try { t1.join(); t2.join(); }
        catch (InterruptedException exn) { 
            System.out.println("Some thread was interrupted");
        }
    }
}

class Printer {
    private static final Object innerLock = new Object();
    public static void print() {
        synchronized(innerLock) {
            System.out.print("-");
            try { Thread.sleep(50); } catch (InterruptedException exn) { }
            System.out.print("|");
        }
    }
}
```

## Exercise 1.3

Consider the lecture’s example in file `TestMutableInteger.java`,which contains this definition of class `MutableInteger`:

```java
class MutableInteger { // WARNING: USELESS IN THIS FORM
    private int value = 0;
    public void set(int value) {
        this.value = value;
    }
    public int get() {
        return value;
    }
}
```

As said in the Goetz book and the lecture, this cannot be used to reliably communicate an integer from one thread to another, as attempted here:

```java
final MutableInteger mi = new MutableInteger();
Thread t = new Thread(new Runnable() { public void run() {
    while (mi.get() == 0) { }
    System.out.println("I completed, mi = " + mi.get());
}});
t.start();
System.out.println("Press Enter to set mi to 42:");
System.in.read(); // Wait for enter key
mi.set(42);
System.out.println("mi set to 42, waiting for thread ...");
try { t.join(); } catch (InterruptedException exn) { }
System.out.println("Thread t completed, and so does main");
```

1. Compile and run the example as is. Do you observe the same problem as in the lecture, where the "`main`" thread’s write to `mi.value` remains invisible to the `t` thread, so that it loops forever?

```java
class Test {
    public static void main(String[] args) {
        final MutableInteger mi = new MutableInteger();
        Thread t = new Thread(new Runnable() { public void run() {
            while (mi.get() == 0) { }
            System.out.println("I completed, mi = " + mi.get());
        }});
        t.start();
        System.out.println("Press Enter to set mi to 42:");
        System.in.read(); // Wait for enter key
        mi.set(42);
        System.out.println("mi set to 42, waiting for thread ...");
        try { t.join(); } catch (InterruptedException exn) { }
        System.out.println("Thread t completed, and so does main");
    }
}

class MutableInteger { // WARNING: USELESS IN THIS FORM
    private int value = 0;
    public void set(int value) {
        this.value = value;
    }
    public int get() {
        return value;
    }
}
```

`System.in.read();` is used to wait thread start and enter the loop. At the end, we get `Execution Timed Out` result.

2. Now declare both the `get` and `set` methods `synchronized`, compile and run. Does thread `t` terminate as expected now?

```java
class Test {
    public static void main(String[] args) {
        final MutableInteger mi = new MutableInteger();
        Thread t = new Thread(new Runnable() { public void run() {
            while (mi.get() == 0) { }
            System.out.println("I completed, mi = " + mi.get());
        }});
        t.start();
        System.out.println("Press Enter to set mi to 42:");
        System.in.read(); // Wait for enter key
        mi.set(42);
        System.out.println("mi set to 42, waiting for thread ...");
        try { t.join(); } catch (InterruptedException exn) { }
        System.out.println("Thread t completed, and so does main");
    }
}

class MutableInteger { // WARNING: USELESS IN THIS FORM
    private int value = 0;
    public synchronized void set(int value) {
        this.value = value;
    }
    public synchronized int get() {
        return value;
    }
}
```

```
Press Enter to set mi to 42:
mi set to 42, waiting for thread ...
I completed, mi = 42
Thread t completed, and so does main
```

3. Now remove the `synchronized` modifier from the `get` methods. Does thread `t` terminate as expected now? If it does, is that something one should rely on? Why is synchronized needed on both methods for the reliable communication between the threads?

No it no longer works as intended, because the "read" thread still does not know that the write thread has written to the variable.

4. Remove both `synchronized` declarations and instead declare field `value` to be `volatile`. Does thread `t` terminate as expected now? Why should it be sufficient to use `volatile` and not `synchronized` in class `MutableInteger`?

```java
class MutableInteger { // WARNING: USELESS IN THIS FORM
    private volatile int value = 0;
    public void set(int value) {
        this.value = value;
    }
    public int get() {
        return value;
    }
}
```

Just two threads, `volatile` is enough to make the `read` thread notice that write thread has changed `value`.

## Exercise 1.4

Consider the lecture’s example in file `TestCountPrimes.java`.

1. Run the sequential version on your computer and measure its execution time. From a Linux or MacOS shell you can time it with `time java TestCountPrimes`; within Windows Powershell you can probably use `Measure-Command java TestCountPrimes`; from a Windows Command Prompt you probably need to use your wristwatch or your cellphone’s timer.

```shell
$ time java TestCountPrimes.java
Sequential result:     664579
java TestCountPrimes.java  9.83s user 0.11s system 117% cpu 8.467 total

Parallel2  result:     664579
java TestCountPrimes.java  10.86s user 0.08s system 183% cpu 5.947 total

Parallel4  result:     664579
java TestCountPrimes.java  13.12s user 0.16s system 304% cpu 4.366 total
```

2. Now run the 10-thread version and measure its execution time; is it faster or slower than the sequential version?

```shell
Parallel10 result:     664579
java TestCountPrimes.java  14.60s user 0.17s system 347% cpu 4.256 total
```

3. Try to remove the synchronization from the `increment()` method and run the 2-thread version. Does it still produce the correct result (664,579)?

The result is not 664579 because of race condition.

4. In this particular use of `LongCounter`, does it matter in practice whether the `get` method is synchronized? Does it matter in theory? Why or why not?

`get` method doesn't need to be synchronized, because it is called after all threads finished  with no race condition.









































