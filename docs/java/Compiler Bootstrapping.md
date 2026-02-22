You may have heard that a **language X is written in X**, and you may be wondering how that is possible. How can a language exist before it exists? The answer is **compiler bootstrapping** — a staged engineering process that allows a language to evolve into a self-hosted system. 

Let’s walk through it using Kotlin as a concrete example, developed by the team at JetBrains.


## Step 1 — The First Compiler

Kotlin was designed for the JVM ecosystem, where:

* Java was dominant
* The JVM was mature
* `javac` already existed

So the Kotlin team at JetBrains wrote the first compiler in Java.

The pipeline looked like this:

```text
Kotlin compiler v0 (Java source)
    ↓
javac
    ↓
v0.jar (JVM bytecode)
```

Key point:

* v0 was just a Java program.
* It translated Kotlin source into JVM bytecode.
* There was no self-hosting yet.


Here is the revised section with the JVM/bytecode explanation integrated naturally and concisely:


## Step 2 — Introducing Kotlin into the Compiler

Once Kotlin became expressive and stable enough, the JetBrains team started rewriting parts of the compiler in Kotlin itself.

Now the pipeline changed:

```text
v0.jar
    ↓
Compiles Kotlin-based compiler source (v1)
    ↓
v1.jar
```

Important details:

* v1 is written (partly or fully) in Kotlin.
* It is compiled by v0’s **binary**, not its source.
* There is no circular dependency.

### Why the JVM and Bytecode Matter

Each compiler version is distributed as:

* Precompiled `.class` files
* Packaged into a `.jar`
* Executed by the JVM


Bootstrapping relies on an already compiled compiler (**binary artifacts**), not recursive source evaluation.


## Step 3 — Full Self-Hosting

After several iterations, the compiler became fully written in Kotlin.

From then on:

```text
Compiler v(N-1)
    ↓
Compiles compiler vN
    ↓
Compiler vN
```

At this stage, Kotlin is self-hosting.

But there is always a prior stable compiler in the chain.


## What Runs When You Compile Kotlin Today?

When you run:

```bash
kotlinc MyFile.kt
```

You are executing:

* A prebuilt Kotlin compiler `.jar`
* Already compiled by a previous Kotlin compiler
* Running as JVM bytecode

The original Java-based compiler (v0) is not part of your toolchain anymore. It was part of the bootstrap history — not the current execution path.


## Bootstrapping vs Self-Hosting

They are related but different:

* **Self-hosting** → The compiler is written in its own language.
* **Bootstrapping** → The staged process that gets you there.

Bootstrapping looks like this:

1. Implement compiler in another language.
2. Produce working compiler binary.
3. Rewrite compiler in target language.
4. Use version N to build version N+1.

It’s versioned recursion, not circular dependency.

---

Happy coding! 💻
