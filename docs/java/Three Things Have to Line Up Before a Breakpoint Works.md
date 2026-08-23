Here's something a little unsettling: set a breakpoint inside a library class — some file deep in a dependency you pulled from Maven — and your debugger stops right there, on that exact line, with the variables laid out in front of you. It feels like the JVM read that file and paused mid-sentence.

It didn't. That `.java` file was never executed. It may not even have been on your machine until the moment you clicked into it. What's really happening is three separate pieces working together, and once you see how they fit, it stops feeling like magic.

## What the JVM is actually running

Start with the thing that's really running. `javac` turns `.java` files into `.class` files containing bytecode — and the same is true of every dependency you pull in. Those jars hold compiled classes, never source. The file you're looking at in the editor was never shipped, loaded, or run.

The JVM then works through that bytecode. It interprets it at first, then hands the hot paths to a JIT compiler that turns them into native machine code on the fly. So by the time your breakpoint hits, that code may be running as raw machine instructions — several steps removed from the text on your screen.

Which makes the puzzle sharper. If the JVM only ever sees bytecode, how does anything know that a given instruction is line 42 of a file it never read? And where did that file come from in the first place?

## Where the source on your screen comes from

Take the second question first. The code IntelliJ shows you isn't what's running — it's a separate lookup, and it happens one of two ways.

Its first choice is a `-sources.jar` (for example `some-library-2.x.x-sources.jar`), either already cached in your local `.m2` repo or fetched from Maven Central on the spot. Most popular open-source libraries publish one. It's a completely separate artifact from the compiled jar your app runs — it exists purely so a human can read it, and IntelliJ matches it to the bytecode using class and method names.

If no sources jar exists for a dependency, IntelliJ falls back to its built-in decompiler (Fernflower) and rebuilds readable-ish source straight from the bytecode. You can spot this in the UI — it shows a banner saying something like "Class file decompiled — no sources found." The rebuilt code won't match the original as closely; things like real variable names can be missing if the class wasn't compiled with full debug info, so you'll see placeholders instead.

## The bridge back to line numbers

Now the harder question — how a bytecode instruction knows which source line it came from. The answer is that the mapping is built into the bytecode itself. When code is compiled, `javac` can embed debug attributes next to the instructions: a `LineNumberTable` and a `LocalVariableTable`, which tie bytecode offsets back to source lines and variable names.

This debug info is *optional* — the JVM spec doesn't require it. It's only there if the code was compiled with debug flags on (`javac -g`, or the more specific `-g:lines,vars,source`). In practice most published library jars include all of it, since build plugins turn it on by default and stripping it out is rare. That's why breakpoints in dependency code usually just work, even though nothing forces a library author to make that happen.

When you run in debug mode, the JVM exposes a debugger interface (JDWP). IntelliJ uses it to say: "pause when execution reaches the bytecode instruction for line N in class X." The JVM does that using the embedded line-number table — the sources jar plays no part here at all. It only comes in afterward, to show readable code once execution has paused, and to let you click a line to set the breakpoint in the first place.

## So my production image ships all that debug info too?

Yes — the jars in your container image are the same artifacts your build tool downloaded, debug info and all. But the cost is smaller than you'd think, and you want most of it anyway.

Together the two attributes add roughly 10–20% to raw `.class` size, and jars are zip-compressed, so the packaged difference is smaller still. Runtime cost is basically zero: the JIT never reads them. And `LineNumberTable` is what puts line numbers in your stack traces — strip it and every frame reads `Unknown Source`, a bad trade when a stack trace is often all you have during an incident.

## Putting it together

- **Compiled bytecode** (from the Maven jar) — what the JVM actually loads and runs, interpreted and then JIT-compiled
- **Sources jar, or a decompiler as fallback** — what IntelliJ shows you, purely so you can read it and click breakpoint gutters
- **Debug metadata inside the bytecode, if the library was compiled with it** — the bridge that ties "line 42 in Foo.java" to "bytecode offset 17 in Foo.class"

It *looks* like your program is running from source. In reality the JVM is running bytecode the whole time, and the debugger plus the sources jar are just giving you a readable window into it.

---

Happy coding! 💻