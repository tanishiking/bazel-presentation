---
theme: gaia
_class:
  - lead
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
marp: true
---

![bg left:30% 80%](https://upload.wikimedia.org/wikipedia/en/7/7d/Bazel_logo.svg)

# **Introduction to Bazel**

**{ Fast, Correct } — Choose two**

https://bazel.build/

Rikito Taniguchi

<!--
intro to bazel
cuz, narita migrating sbt to bazel
I think it's key to success to let
all know about what and how to Bazel
-->

---

# Hi! :wave:

- Rikito Taniguchi (@tanishiking on github)
- Working from Japan :japan:
- Scala Space, working on Scala ecosystem
  - LSP, compiler, formatter and linter
- Bazel experience: 1.5 month
  - but I'm loving it!

<!--
new to Bazel
Kryzstof told me about Bazel migration 1.5 month ago
-->


---

# Agenda

- The "problem"
- What and Why Bazel
- Bazel tutorial along with Scala
  - How to build Scala files
  - How to use 3rd party library

<!--
Here's todays's agenda
Start with what is the problem

...

And I gonna going through a Bazel
tutorial along with building
simple Scala application

so we have 60 minutes including Q&A
I have 30 pages of slides, so I will go totally smoothly I'm sure
-->

---

![bg left:50% 90%](./imgs/compiling.jpeg)

# **The "Problem"**

## ・Building large applicaton is slow

## ・Building slowly is expensive

<!--
So, what is the PROBLEM we're facing and trying to solve

the problem is the slow compilation

narita take a mono-repo approach, so it's becoming larger
building large application is slow
and Scala compilation is Slow

as a result,
when dev submit a small PR and wait 30 min for CI
CI fails fix it wait another 30 min

it's a huge burdon of productivity
it's almost impossible to going into flow
-->

---

# Options to alleviate the problem

- Optimize Scala compilation (maybe using [scalac-profiling](https://www.scala-lang.org/blog/2018/06/04/scalac-profiling.html))
- Optimize sbt build
  - [-Dsbt.traces=true](https://github.com/sbt/sbt/pull/4576)
  - [Custom configuration](https://www.scala-sbt.org/1.x/docs/Advanced-Configurations-Example.html)
  - build cache
- Split to multi-repo
- [Compile Scala Faster with Hydra - Triplequote](https://triplequote.com/hydra)


Still, they're not **scalable**

<!--
What we can do for the problem
migrating to Bazel isn't only opton

- profiling Scala..., usually it's all about shapeless auto derivation
- optimizing sbt build, sbt has a useful option to learn the bottleneck
  - there's not so much thing we can do: making smaller granularity of modules
  - build cache, It's so hard in sbt (I'll talk more later) 
- Or, multi-repo
- And I'm not sure it's still alive, but using faster compiler is also 

still build time grows as the project and code size grows
we can't help even if we use other programming languages.

Question is how the big techs like Google is dealing with the large application
-->

---

![bg left:20% 80%](https://upload.wikimedia.org/wikipedia/en/7/7d/Bazel_logo.svg)

# **Bazel for rescue!**

Build system developed by Google

- **Artifact based build system**
  - :left_right_arrow: Task based build system (make, ant, sbt...)
- **{Fast, Correct} Choose two**
  - :arrow_right: Scalable even in Google scale

<!--
Here it comes Bazel
solve scalability issue, Google built a scalable build system called Bazel

What is Bazel

We can describe Bazel with 2 properties

- artifact based
- fast adn correct
-->


---

# {Task, Artifact}-based build system
- **Task based build system** (ant, make, maven, sbt)
  - **Imperative** set of tasks (imagine Makefile).
  - You can do pretty much anything :+1:
- **Artifact based build system** (bazel, pants, buck)
  - **Declative** set of artifacts to build, deps, and limited options
  - Only build, test, and run.
  - **Your build is pure function**

[Software Engineering at Google | chapter 18](https://abseil.io/resources/swe-book/html/ch18.html)

<!--
I mentioned, Bazel is artifact based build system: but what is artifact...

The traditional build systems like ... are called task based build system
in the build file we describe imperative set of tasks
do task A then task B then task C
you can do ... (e.g narita runs npm from sbt)
run npm -> copy the js to resources dir -> clean up dir -> then build scala

On the other hand, artifact based ...
the unit of build step is something like build function
in he build file declative set of ...
what we can do is only build ... 

So, the basic idea of Bazel is
Your build is a pure function.
There’s sources that come in, and then the artifacts go out.
There's no side effects,

but in task-based build system we may introduce side-effects like
- writing some files during build files
-->

---

# {Fast, Correct} choose two ✌️
How to make build a "pure function"?

**[Hermeticity](https://bazel.build/basics/hermeticity)**
> When given the same input source code and product configuration, a hermetic build system always returns the same output **<span style="color:red">by isolating the build from changes to the host system</span>**

:arrow_right: **Correct** :arrow_right: reliable remote build cache :arrow_right: **Fast**

<!--
So, in Bazel, build is a pure function
but in reality, computer has a lot of states like tmp files
To keep our build pure, we have to run build in a clean enviornment

So Bazel runs something called hermetic build (or hermeticity)

quote...

so Bazel copies the sources artifacts into a clean env called sandbox 
whose filesystem is isolated from host filesystem, it's like linux container

by doing this, we always have a correct build

by having a correct build, we can reliably share the build cache
because as long as the input is the same, the output is the same
on anyone's computer.

with the remote build cache, we have fast build
so the idea of bazel is more like speed-through-correctness
Whereas a lot of people view those two one another.
-->

---

# Dark side of Bazel

- Poor IDE support (it's getting better though...)
- [ How your Monorepo breaks the IDE. And what we’re doing about it. - Justin Kaeser - YouTube](https://www.youtube.com/watch?v=kbP40Xd_sR8)
- Less flexibility
- More explicit build settings


![bg right:50% 80%](https://pbs.twimg.com/media/Fgqrd2rVIAAc6yT?format=jpg&name=small)

<!--
Ok, great, but why not everyone use Bazel

there're many trade-offs

- one thing is poor IDE support: IDE usually communicate with build sysytem
  - maybe Lucasz has an opinon...
- also mono-repo is basically bad for IDE, because IDE try to index everthing in the project...
- Less flexibility
- I'll show later but Bazel requires much more build settings
-->

---

# Is Bazel a right path?

Not sure, yet!

- We have only around 200k lines of Scala code
- but we're going mono-repo and project will grow
- Scala compile is slow for LOC...

[When to use Bazel? - Earthly Blog](https://earthly.dev/blog/bazel-build/)

## Depends on how much developers willing to deal with the trade-offs

<!--
So, should we really use Bazel? I'm not sure TBH

- There's only 200k lines of Scala code, which is relatively small for Bazel I think
- but ... so in the future, we may have 1m loc (in that case)
- project size is bit small, but anyway compile is slow, so Bazel may worth it

That's being said, I think it depends on how much team members ...
-->


---


# All team members MUST learn Bazel

Otherwise... 

> New team mebers didn't learn Bazel ... most of the members could not write Bazel-related code and they just use what there is.

(Japanese blog) [Say goodbye to Bazel and start using make](https://mixi-developers.mixi.co.jp/byebye-bazel-welcome-make-b966bfd37fce)


<!--
So the key of success for Bazel migration is that all team members know Bazel 

I know the case study by Japanese largest social game developer company
introduced Bazel but switch back to make 
and one reason is team members didn't learn Bazel
and they couldn't write or modify build settings, which is important for Bazel project 

so it's importatnt ot learn Bazel, and we should add learning Bazel to
project introduction
-->

---

# Questions so far?
next we're going to go through Bazel/Scala tutorial

---

![bg left:30% 80%](https://upload.wikimedia.org/wikipedia/en/7/7d/Bazel_logo.svg)

# **Bazel Tutorial for Scala**

**What you'll learn**

- Bazel 101
  - What the Bazel project looks like
  - What inside `WORKSPACE` and `BUILD` files
  - What is `Label` in Bazel
- How to build jar from Scala fiels using `rules_scala`

[tanishiking/bazel-tutorial-scala](https://github.com/tanishiking/bazel-tutorial-scala)

<!--

...
I published this tutorial to the repository.

-->

---


# Install Bazel

Use Bazelisk! It reads `.bazelversion` and download Bazel executable.

[bazelbuild/bazelisk: A user-friendly launcher for Bazel.](https://github.com/bazelbuild/bazelisk)

> Install it as the bazel binary in your PATH (e.g. copy it to /usr/local/bin/bazel). Never worry about upgrading Bazel to the latest version again

```
alias bazel="bazelisk" # I personally do
```

<!--
basically, I recommend installing CLI called Bazelisk
bazelisk is a bazel launcher and it reads...

Also, I recommned, aliasing bazel as bazelisk so you don't have to worry about
updating bazel version

so, let's get started
-->

---

[bazel-tutorial-scala/01_scala_tutorial](https://github.com/tanishiking/bazel-tutorial-scala/tree/main/01_scala_tutorial)

```
|-- WORKSPACE
`-- src
    `-- main
        `-- scala
            |-- cmd
            |   |-- BUILD
            |   `-- Runner.scala
            `-- lib
                |-- BUILD
                `-- Greeting.scala
```

- `WORKSPACE` file is about getting stuff from the outside world into your Bazel project. Located at the project root.
- `BUILD` files are about what happening inside of your Bazel project

<!--
take a look at the project structure,
in this project we build a pretty simple scala app
have to src dir: cmd and lib 

the Bazel setting files are `WORKSPACE` located on the project root
and `BUILD` files in each directory

...

I'll show you the details just remember there's WOKRSPACE and BUILD
-->

---

# Understand `WORKSPACE`

```
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")
# ...
http_archive(
    name = "io_bazel_rules_scala",
    sha256 = "77a3b9308a8780fff3f10cdbbe36d55164b85a48123033f5e970fdae262e8eb2",
    strip_prefix = "rules_scala-20220201",
    type = "zip",
    url = "https://github.com/bazelbuild/rules_scala/releases/download/20220201/rules_scala-20220201.zip",
)
```

https://github.com/tanishiking/bazel-tutorial-scala/blob/main/01_scala_tutorial/WORKSPACE

Basically, just copy and pasted from [bazelbuild/rules_scala](https://github.com/bazelbuild/rules_scala)

<!--
The first thing we should take a look in Bazel workspace is a `WORKSPACE` file

`WORKSPACE` file contains the external dependencies
In this example, we download a rules_scala which is a Bazel build rule for Scala.
and do some primary settings for rules_scala

if you see the whole things, it looks puzzle but

basically this is just copy and pasted from `rules_scala`'s instruction
don't worry

by setting WORSPACE file, we're ready to start bazel project
-->

---

# Scala files

```scala
// cat src/main/scala/lib/Greeting.scala
package lib
object Greeting { def sayHi = println("Hi!") }
```

```scala
// cat src/main/scala/cmd/Runner.scala
package cmd
import lib.Greeting
object Runner { def main(args: Array[String]) = { Greeting.sayHi } }
```

- `lib/Greeting.scala` is a library moduel that provides `lib.Greeting`.
- `cmd/Runner.scala` depends on `lib.Greeting`.

<!--
Now let's see Scala files
we have 2 Scala files, lib/Greeting and cmd/Runner
In this project, we build simple CLI application

In lib/Greeting ...


it's simple
so let's build this application using Bazel
-->

---

# Understand `BUILD` file for `lib`

```
# cat src/main/scala/lib/BUILD
load("@io_bazel_rules_scala//scala:scala.bzl", "scala_library")

scala_library(
    name = "greeting",
    srcs = ["Greeting.scala"],
)
```

- `scala_library` is called `rule` in Bazel that describes what to build
- An instance of `rule` is called `target`.

[document: rules_scala/scala_library.md](https://github.com/bazelbuild/rules_scala/blob/master/docs/scala_library.md)

<!--
Ok let's build lib with Bazel
in Bazel we write BUILD file to let Bazel know how to build

In the topline, we import `scala_library` from `rules_scala`
`scala_library` is something called build `rule` in Bazel,
and we declare the build using this rule.
and declared instance of rule is called target

`scala_library` is a build rule to build a library jar file for Scala
The required attributes are `name` and `srcs`

- name attribute defines the name of the target.
- srcs attribute defines, which scala sources to build
-->

---

# Let's build!

`bazel build <targets>`

```sh
❯ bazel build //src/main/scala/lib:greeting
...
Target //src/main/scala/lib:greeting up-to-date:
  bazel-bin/src/main/scala/lib/greeting.jar
```

:tada:

Wait, what `//src/main/scala/lib:greeting` means!?

<!--
Ok, now we put `BUILD` file,
we can build, we type `bazel build` and specify the target to build in arguments

To build the greeting library, we type `bazel build` ... and specify the target, and it produces `greeting.jar` file

that's all

(I think everyone questions what this `//src/main/scala...` part)

-->

---

# Label

Label uniquely identifies a `target`. Canonical form of label looks like

**<span style="color:red">@myrepo//</span><span style="color:blue">my/app/main</span><span style="color:maroon">:app_binary</span>**

- **<span style="color:red">@myrepo//</span>** - repository name to access workspace, we can omit `@myrepo` and `//` to refer same repository.
- **<span style="color:blue">my/app/main</span>** - path to the package relative to repository root.
- **<span style="color:maroon">:app_binary</span>** - target name

[Labels  |  Bazel](https://bazel.build/concepts/labels)

<!--
This is something called Label
Label is a format to uniquely identifies a target

canonical form of label looks like this
and it has 3 components

- 1st: repository name, we can name the repository name here to access other workspace, but usually we can omit @myrepo part if we access the targe in same repo
- 2nd: path to the package ...
- 3rd: target name

-->

---

# Label

```sh
❯ bazel build //src/main/scala/lib:greeting
...
Target //src/main/scala/lib:greeting up-to-date:
  bazel-bin/src/main/scala/lib/greeting.jar
```

**<span style="color:red">//</span><span style="color:blue">src/main/scala/lib</span><span style="color:maroon">:greeting</span>**

- **<span style="color:red">//</span>** - (abbreviated) repo name
- **<span style="color:blue">src/main/scala/lib</span>** - path to `BUILD` file (from workspace root)
- **<span style="color:maroon">:greeting</span>** - target name to build

<!--

so let's take a look again

- // is the repo name, but we omit @ part because refer the target in same repo
- next, ... aka package,
- greeting is the taret name defined in BUILD file

that's how we can identifies the greeting build target with //src/main...

any questions so far
-->

---

# Depends on `lib` target!

```
# cat src/main/scala/cmd/BUILD
load("@io_bazel_rules_scala//scala:scala.bzl", "scala_binary")
scala_binary(
    name = "runner", main_class = "cmd.Runner",
    srcs = ["Runner.scala"],
    deps = ["//src/main/scala/lib:greeting"],
)
```

[scala_binary](https://github.com/bazelbuild/rules_scala/blob/master/docs/scala_binary.md) rule generate a jar file, and shell script to run the jar

Enumerate all dependent targets in `deps` attr

[Dependencies  |  Bazel](https://bazel.build/concepts/dependencies)

<!--
Next let's build cmdline that uses lib
as we build an application endpoint, we use scala_binary ...
except we specify main_class, basically the same with scala_library,
but now, we have deps attribute
we enumerate dep target here.

cmd depends on lib, so we add target label to deps
-->

---

# Build the binary!

Oops build failed

```
❯ bazel build //src/main/scala/cmd:runner

ERROR: .../01_scala_tutorial/src/main/scala/cmd/BUILD:3:13:
in scala_binary rule //src/main/scala/cmd:runner:
target '//src/main/scala/lib:greeting' is not visible from
target '//src/main/scala/cmd:runner'.
```

Bazel has a concept of visibility, and by default, all targets' visibility is `private`, targets in the same package can access them.

<!--
Now, we should be able to build the application by bazel build ....
but it fails,

and it says lib:greeting is not visible from target cmd:runner

because, bazel has a concept of visibility and ...

-->

---

# Make `lib` visible from `cmd` package

```diff
 scala_library(
     name = "greeting",
     srcs = ["Greeting.scala"],
+    visibility = ["//src/main/scala/cmd:__pkg__"],
 )
```

[visibility](https://bazel.build/concepts/visibility) controll access grants to packages 

- `//src/main/scala/cmd:__pkg__"`grants access to the package `//src/main/scala/cmd`
- `"//visibility:public"` grants access to all packages 

<!--
To make lib visibile from cmd package,
we have to add visibility attr to lib:greeting,

__pkg__ means grants access to the package //src/main... which means all targets defined in //src/main cmd can access
also we can grants access to all packages by ...
-->

---

# Build the binary! (again)

```sh
❯ bazel build //src/main/scala/cmd:runner
...
INFO: Found 1 target...
Target //src/main/scala/cmd:runner up-to-date:
  bazel-bin/src/main/scala/cmd/runner.jar
  bazel-bin/src/main/scala/cmd/runner
```

```sh
❯ ./bazel-bin/src/main/scala/cmd/runner
Hi!
```

<!--
Now, it should build fine,

bazel build //...

output jar and shell script
we can run shell script

Now, you learnt the basics of Bazel project, questions?
-->

---

# Tips: Wildcard

Usually build all targets by `$ bazel build //...`

- `//...`	All targets in packages in the workspace. 
- `//foo/...` All rule targets in all packages under foo dir

[Building multiple targets](https://bazel.build/run/build#specifying-build-targets)

<!--
you may think it's so annoying to write the label like //src/main everytime,
don't worry, there's a shortcuts

the most used one, //... which expands to the all targets in the workspace
so bazel build //... means build everything

you can bazel build //foo/... means build all targets under foo dir

-->

---
# Tips: bazel query

[bazel query](https://bazel.build/query/quickstart) is useful to find target

```
❯ bazel query //... | grep lib
//src/main/scala/lib:greeting
```

- `bazel query //...` to list all targets in the repo
- `bazel query //... --output=location` to show the location of target definitions
- `bazel query "rdeps(//..., //src/main/scala/lib:greeting)"`
  - reverse deps of `:greeting` from `//...`

<!--
Another tip is: I often forget the label for some specific target,
I remember the directory name but can't remember the fully qualified label

in that case we can use `bazel query`
bazel query can do a lot of things, but I often list all targets defined in
the workspace and grep by string to find a label

there're lot usage but you don't have to remember everything :)

so that's the end of basic scala bazel tutorial and next moving to using 3rd party lib
-->

---

![bg left:30% 80%](https://upload.wikimedia.org/wikipedia/en/7/7d/Bazel_logo.svg)

# **Tutorial for `rules_jvm_external`**

**What you'll learn**
- How to download external dependencies from maven repositories.
- How to depend on downloaded packages

https://github.com/tanishiking/bazel-tutorial-scala/blob/main/02_scala_maven

<!--
next we're going to download third party scala library and build a
simple application using them

you'll learn ...

the sample repo is also published here...

-->

---

# rules_jvm_external
[bazelbuild/rules_jvm_external](https://github.com/bazelbuild/rules_jvm_external) is a popular ruleset to resolve and download JVM dependencies.

Download `rules_jvm_external` in `WORKSPACE` as always

```
RULES_JVM_EXTERNAL_TAG = "2.5"
RULES_JVM_EXTERNAL_SHA = "249e8129914be6d987ca57754516be35a14ea866c616041ff0cd32ea94d2f3a1"
http_archive(
    name = "rules_jvm_external",
    sha256 = RULES_JVM_EXTERNAL_SHA,
    strip_prefix = "rules_jvm_external-%s" % RULES_JVM_EXTERNAL_TAG,
    url = "https://github.com/bazelbuild/rules_jvm_external/archive/%s.zip" % RULES_JVM_EXTERNAL_TAG,
)
```

<!--
To download 3rd party libs, we use something called rules_jvm_external
same with rules_scala, we just copied and pasted from document
-->

---

# Download JVM deps

```
# WORKSPACE
load("@rules_jvm_external//:defs.bzl", "maven_install")
maven_install(
    artifacts = [
        "org.scalameta:scalameta_2.13:4.5.13",
        "com.lihaoyi:pprint_2.13:0.7.3",
    ],
    repositories = [
        "https://repo1.maven.org/maven2",
    ],
)
```

<!--
and in WORKSPACE file, we list the deps,
in this case, we download scalameta library to parse scala
and pprint to pretty-print scala object

Note that, unlike sbt, we have to write `_2.13` explicitly, because
bazel and rules_jvm_external doesn't know scala add version to the postfix
-->

---

# Use downloaded libraries

```
# cat src/main/scala/example/BUILD
scala_binary(
# ...
    deps = [
        "@maven//:com_lihaoyi_pprint_2_13",
        "@maven//:org_scalameta_scalameta_2_13",
    ],
)
```

> The default label syntax for an artifact `foo.bar:baz-qux:1.2.3` is `@maven//:foo_bar_baz_qux`
https://github.com/bazelbuild/rules_jvm_external#usage

<!--
Now, we can depends on those libraries using `deps` attr

rules_jvm_external generates the targets for the downloaded
libraries, and we can access them using the labels like these wired stuffs

it's basically, just replace . or - to underscore and joined everthing with _

-->

---

# Tips: find library's label

[bazel query](https://bazel.build/query/quickstart) again!

Enumerate all targets under `@maven` repo, and grep `pprint`

```sh
❯ bazel query @maven//... | grep pprint
@maven//:com_lihaoyi_pprint_2_13
@maven//:com_lihaoyi_pprint_2_13_0_7_3
```

<!--
To find the library's label, bazel query is useful again
As all the maven targets are defined under `@maven` repository,
we can list all maven deps by `bazel query @maven//...`

and we can grep by pprint
and we get the label for pprint
-->


---

# Build it!

```sh
❯ bazel build //src/main/scala/example:app
Target //src/main/scala/example:app up-to-date:
  bazel-bin/src/main/scala/example/app.jar
  bazel-bin/src/main/scala/example/app

❯ bazel-bin/src/main/scala/example/app "object main { println(1) }"
Source(
  stats = List(
    Defn.Object(
      ...
```

## Now you learnt all Bazel basics :tada:

<!--
Now we can build it
and run the applciation that uses scalameta and pprint
-->



---

# Wanna learn more?
- [Bazel getting started](https://bazel.build/start)
  - Recommend to skim through **Java tutorial** and **Build concepts**
- [bazelbuild/rules_scala](https://github.com/bazelbuild/rules_scala)
- [bazelbuild/rules_jvm_external](https://github.com/bazelbuild/rules_jvm_external)
- [tanishiking/bazel-playground](https://github.com/tanishiking/bazel-playground)
  - You can find my Bazel example projects :smile:
- [Software Engineering at Google, chapter 18](https://abseil.io/resources/swe-book/html/ch18.html)
  - To learn the philosophy of Bazel

---

# Topics I didn't cover

- Target granularity and trade-offs
  - read [How to choose the right build unit granularity | by Natan Silnitsky | Wix Engineering | Medium](https://medium.com/wix-engineering/migrating-to-bazel-from-maven-or-gradle-part-1-how-to-choose-the-right-build-unit-granularity-a58a8142c549)
- Bazel devtools (attached some links in the following slide)
- [Remote Caching](https://bazel.build/remote/caching)
- [Remote Execution](https://bazel.build/remote/rbe)

---

# Interesting Bazel talks and articles
- [Awesome Bazel | awesome-bazel](https://awesomebazel.com/)
- [How to successfully migrate to Bazel from Maven or Gradle. (Natan Silnitsky, Israel - Youtube](https://www.youtube.com/watch?v=2UOFm-Cc_cU&t=2882s])
- [When to use Bazel? - Earthly Blog](https://earthly.dev/blog/bazel-build/)
- [How to choose the right build unit granularity | Medium](https://medium.com/wix-engineering/migrating-to-bazel-from-maven-or-gradle-part-1-how-to-choose-the-right-build-unit-granularity-a58a8142c549)
- [A Bable in Bazel](https://sluongng.hashnode.dev/) Blog posts about Bazel internal

---

**Bazel dev tools**

- [IntelliJ with Bazel](https://ij.bazel.build/)
  - Bazel IDE for IntelliJ, developed by Jetbrains + Bazel team
- [Bazel - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=BazelBuild.vscode-bazel)
  - Syntax highlight + format + lint
- [bazel-stack-vscode - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=StackBuild.bazel-stack-vscode)
  - Experimental IDE for VSCode
- [JetBrains/bazel-bsp](https://github.com/JetBrains/bazel-bsp) required for Scala IDE work with Bazel
- [buildtools/buildifier](https://github.com/bazelbuild/buildtools/tree/master/buildifier) Bazel formatter and linter
- [Gazelle](https://github.com/bazelbuild/bazel-gazelle) Bazel build file generator, Scala is not yet supported

---

![bg left:30% 80%](https://upload.wikimedia.org/wikipedia/en/7/7d/Bazel_logo.svg)

# [Home - BazelCon 2022](https://opensourcelive.withgoogle.com/events/bazelcon2022?utm_source=BazelCon&utm_medium=Social&utm_campaign=BazelCon%2B2022) is around the corner!
