---
layout: post
title:  "Memory Leak"
date:   2023-06-04 18:26:30 +0100
tags:   coding Kotlin Jetpack Compose
---

#### C (programming language)

Since the early 70\'s, C allows dynamically allocating memory for a variable.  With the successful allocation, the address of the memory is returned to the program where a pointer to the address is used to further access the memory.

The memory needs to be released when not used, otherwise memory will be leaked, meaning the system still holds the memory for the program but the program lost access to it.  If the program repeats leaking memory while running, more and more memory of the system will be occupied and eventually the memory runs out.  This would cause crash of some unreliable systems like the early Windows resulting in the famous blue screen, at which point the user has to power off the hardware and reboot, resulting in loss of data of all the running programs on the system.  A more reliable system like Unix/Linux wouldn't crash but will kill the mal-functioning program, resulting in loss of data of the program.  

In C's policy, releasing allocated memory is the sole responsibility of the program (or developer).  So the syntax goes like this:
```
// Allocate memory for an integer
int *p = malloc(sizeof(int));

// use the memory pointed by p

// Deallocate the memory
free(p);
```

#### C++
Later since the 80\'s, C++ adopted the same policy of C on dynamically allocating memory: releasing allocated memory is the sole responsibility of the program (or developer).  C++ is object oriented and allows allocating memory for an object of certain class.  The syntax goes like:
```
class ClassA {
private:
  // Data members
  int id;
public:
  // Member functions
  void setId(int id) { this->id = id; }
  int getId() { return id; }
};
// ......

// Create object of ClassA and allocate memory for it
ClassA* object = new ClassA();

// Do something with object

// Deallocate the memory
delete object;
```
As a program gets more complex and the language features get more sophisticated, developers spend serious amount of efforts and time ensuring correctly deallocating memory.  But the good thing is the policy is as cast iron

#### Java
In the mid 90\'s, Java was developed by Sun MicroSystems (now part of Oracle).  In many ways, Java is very similar to C++: object-oriented, syntax, etc.  However, regarding allocated memory, Java made a distinction: instead of a pointer, a reference is used for accessing the allocated memory, and a garbage collector (GC) takes care of automatically cleaning up unused objects and releasing the memory.  The programmer is no longer responsible and not even encouraged for releasing the memory.  The syntax is like:
```
public class ClassA {
  // Data members
  private int id;
  private String name;

  // Constructor
  public ClassA(int id, String name) {
    this.id = id;
    this.name = name;
  }

  // Methods
  public int getId() {
    return id;
  }

  public void setId(int id) {
    this.id = id;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }
}

ClassA object = new ClassA(10, "John Doe");
object.setId(20);

// no need to release memory
```

#### Kotlin
In 2016, Kotlin 1.0 was released by JetBrains.  Kotlin is designed to interoperate fully with Java and promises a cross-platform, statically typed, general-purpose programming language with type inference which allows more concise syntax.  Like Java, Kotlin uses a reference for accessing allocated memory and a garbage collector to manage cleanups of memory.  

```
class ClassA {
    val name: String = "ClassA"

    fun sayHello() {
        println("Hello, I am ClassA")
    }
}

val instance = new ClassA()
instance.sayHello()

// no need to release memory
```

#### GC Not a panacea
Not having to worry about deallocating memory was a great news for developers as quite some time and efforts could be saved, which turned out true initially.  But as a program gets more complex and language features get more sophisticated, problems of garbage collectors start to show up.  

First, the GC can cause pauses in the execution of the program, then the GC can cause memory fragmentation which makes it difficult for the GC to allocate large objects, and leads to performance problems.  And ultimately, the GC can't take care of all situation to collect unused memory leading to memory leaks.  

Particularly due to the last point, developers found themselves again having to deal about memory leaks.  Yes, with a GC, explicit statements on clearing memory can be saved, but as policy becomes less cast iron, oftentimes, more efforts might be needed to fix unspecified and unexpected memory leaks.  It's like you have a partner who promises to clean the house but fails or even refuses to do so in various unspecified and unexpected situations and you have to figure out what happened and which parts of the house are not cleaned.  So it might put you in more difficult situations than simply cleaning the house yourself.

#### In the case of Jetpack Compose

Recently, I was put to the difficult situation.

My Android app for learning languages MensLingua was initially written in Java with the traditional Android way (with XML and databinding and so forth).  Early last year I migrated to Kotlin and later to Jetpack Compose.  It has shown signs of possible memory leaks.  Lately, I tried to find out what were going on.

For those unfamiliar with Android programs, every screen of an app you interact with on an Android phone is based on a class termed as an activity.  Normally, when you tap on a button, and a new screen appears, that new screen belongs to a newly created activity.  When you hit the back button, the activity is destroyed and Android presents you with the screen of the previous activity.

Jetpack Compose, first released in early 2021, is a modern UI toolkit (only working in Kotlin) for building native Android apps.  It was to replace the traditional clumsy XML approach.

To display a clickable text "Save" which performs function save() when clicked looks like this:
```
Text(text = "Save",
    modifier = Modifier
        .clickable {
            save() 
        }
)
```
When the above code is put into the right place of the code structure, the text will show up on the screen at the right location and allow the click to perform the right function.  Pretty neat, isn't it?

Note save() is a member function of the activity.

Unfortunately, it's reported the above code leaks memory.  Upon investigation, it turns out the code block after clickable calling save() implicitly holds a reference to the activity and causes GC not able to clear memory when the activity is destroyed.

Digging through the documentation, I saw mentioning the use of WeakReference, which sounds like a good idea, so I changed the above to this:
```
Text(text = "Save",
    modifier = Modifier
        .clickable {
            WeakReference(this@MyActivity).get()?.save()
         }
)
```
So the code block should now only hold a WeakReference, which is supposed to not hold too tightly when the GC tries to clear for the activity.  Unfortunately, this approach didn't seem to help and I don't understand why.

Upon further investigation, I got the thinking that I need to figure out a way to explicitly clear the reference.  So this is a situation where the GC unexpectedly fails to do the job.  The right approach is to clear the implicit reference in the code block when the activity is destroyed, but the reference is within the Jetpack Compose code inaccessible to me?  

After trials and errors, the following appears the right approach, where I use a coroutine and cancel the coroutine scope upon dispose.
```
val crScope = rememberCoroutineScope()
DisposableEffect(crScope) {
    onDispose {
        crScope.cancel()
    }
}
//..........

Text(text = "Save",
    modifier = Modifier
        .clickable {
            crScope.launch { save() }
        }
)
```
The first part with the DisposableEffect is in the same composable function as the Text.  When the activity is destroyed, onDispose will be called which cancels the coroutine scope and clears any references in the scope.

Fixing the problem associated with the small piece of code took a lot of efforts and time, and I have many similar pieces in the entire project!  The idea of GC had great promises, it is helpful in someways, but undeniably it presents further problems that can be harder to solve by a developer.

## Cheers!  Santé!  Prost!