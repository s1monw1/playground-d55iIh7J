# "Kotlin".plus("Vert.x").plus("Gradle")

_Disclaimer: My articles are published under 
<a href="https://creativecommons.org/licenses/by-nc-nd/4.0/legalcode" target="_blank">"Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)"</a>._

© Copyright: Simon Wirtz, 2017
https://blog.simon-wirtz.de/setup-vert-x-application-written-in-kotlin-gradle-build/

Feel free to share.

Since I’m really interested in **Reactive Programming** and also love to
use **Kotlin**, I decided to use **Vert.x** in combination with Kotlin
in a simple example application. This post will give some basic
information on Vert.x as a tool set for writing reactive applications on
the JVM as well as some words on Kotlin. In the end, I want
to demonstrate how this application can be set up in Gradle. Of course,
this will be just one out of a million possible solutions…

## Vert.x 


Vert.x isn’t described as a framework, but a "tool-kit" for writing
reactive apps on the Java Virtual Machine. As defined in the ["Reactive
Manifesto"](https://www.reactivemanifesto.org) (Published on September
16, 2014) "Reactive Programming" is some kind of architectural software
pattern, which tries to solve today’s requirements on modern software
applications. More precisely, reactive applications are expected to be
"more flexible, loosely-coupled and scalable", "easier to develop", and
finally "highly **responsible**", which indeed sounds desirable, doesn’t
it?

As one important implication, these applications need to be
**message-driven**, which means applications only communicate via
*asynchronous*, *non-blocking* messages. Vert.x in particular applies to
the Manifesto and provides a great toolset that allows us to develop
reactive software quite easily.

> **Tip**
>
> One important thing to mention: Vert.x is "polyglot", i.e. it can be
> used with multiple languages including Java, JavaScript, Groovy,
> Scala, Ruby and *Kotlin*.

It is also modular, very fast and yet very lightweight (\~650kB). 
Altogether it provides a reasonable alternative to other, maybe better
known, microservice frameworks.

## Kotlin 


[Kotlin](http://kotlinlang.org) is a pretty new JVM language developed
by the great guys of [JetBrains](https://www.jetbrains.com), currently
available in Version 1.1.50. It is statically typed and fully interoperable
with Java. JetBrains' idea in
creating this language was to find some kind of alternative to Java,
still being compatible to each other. I think every Java developer who
has been working with e.g. Groovy or Scala knows that Java requires a
lot of boilerplate code for some simple tasks and has quite a few flaws.
Kotlin is inspired by Java, Scala, C# and others trying to combine the advantages of
these.

Just a few months ago, Google announced Kotlin to be an official Android
programming language on this year’s Google I/O conference. 

I think it’s time to give some examples of features available in Kotlin, also consider reading my other [articles](https://tech.io/users/2188757/s1m0nw1):

**Local Variable Type Inference**

The following lines of code demonstrate how variables are defined in
Kotlin:

``` kotlin
var myvar1 = 100
val myvar2 = "text"
```

The types of `myvar1` and `myvar2` (`Int` and `String`), are being
inferred behind the scenes for us. The `val` implies "read-only", whereas `var` can be reassigned.

**Data classes**

Just think of a Java class `Person`, having attributes like `name`,
`age`, `job`. The class is supposed to override `Object` 's
`toString()`, `hashCode()` & `equals()` methods and also provide
getters and setters for each attribute.

``` kotlin
data class Person(val name: String, val age: Int, val job: String)
```

That’s all you need to write in Kotlin - a real time saver.

**Null Safety**

Well… We all know what a `NullpointerException` is - we all hate it.
Kotlin tries to eliminate it by distinguishing between nullable and
not-null types. Guess, what kind is used by default? Absolutely,
the non-null reference type, i.e. a variable of type `String` cannot
hold `null`. Just for the records, the corresponding nullable type would
be `String?`.

This alone would not make a big difference, so using nullable types
makes us handle all those NPE-prone cases like we can see here:

``` kotlin runnable
fun main(args: Array<String>){
    val iCanBeNull:String? = null;
    //do some hard work with the string
    println(iCanBeNull?.length)
}

```

The `length` call will evaluate to the String’s length if `iCanBeNull` is not
`null`, and to `null` otherwise.

**String Templates**

Another very useful feature anybody will easily understand:

``` kotlin runnable
fun main(args: Array<String>){
    var insertMe = "Reader"
    println("Hello $insertMe")
}
```

**More**

The examples shown above are just a tiny, little set of features I
really love about Kotlin and wished to find in Java as well. I have been
working with Kotlin for a few months now and became a huge fan. I think
in the beginning, it will take some time to get comfortable with the
Java-unlike syntax, but after some lines of code you’ll expirience so
many things, you’ll actually love and do not want to miss anymore…

By the way: Some of the above mentioned language features are available
in the long list of "JDK Enhancement Proposals" (JPE), e.g. [JEP
286:Local-Variable Type Inference](https://openjdk.java.net/jeps/286),
authored by Brian Goetz.

## Finally - The Setup 

In the following you can see my Gradle file. What actually happens is:

a.  Kotlin is configured to compile for Java 1.8 (we do not need 1.6
    support here).

b.  Dependencies are added for Vert.x, Kotlin, Logging & Testing

c.  We use the `application` plugin for building an executable JAR
    (Main-Class is added to Manifest)

> **Tip**
>
> My Kotlin-File "Starter.kt" contains the application entry point: the
> main method. It is being compiled to "StarterKt" and so this has to be
> my Main-Class

**build.gradle.**

``` kotlin
import org.gradle.jvm.tasks.Jar
import org.jetbrains.kotlin.gradle.tasks.*

val kotlin_version = "1.1.50"
val vertx_version = "3.4.2"

plugins {
    application
    eclipse

    kotlin("jvm")
}

tasks.withType<KotlinCompile> {
    kotlinOptions.jvmTarget = "1.8"
}

application {
    mainClassName = "de.swirtz.vertx.standalone.webserver.StarterKt"
    applicationName = "kotlinwithvertx"
    version = "1.0-SNAPSHOT"
    group = "example"
}


dependencies {
    compile(kotlin("stdlib", kotlin_version))
    compile(kotlin("reflect", kotlin_version))

    with("io.vertx:vertx") {
        compile("$this-core:$vertx_version")
        compile("$this-web:$vertx_version")
        compile("$this-web-templ-thymeleaf:$vertx_version")
    }

    compile("org.slf4j:slf4j-api:1.7.14")
    compile("ch.qos.logback:logback-classic:1.1.3")

    testCompile(kotlin("test-junit", kotlin_version))
    testCompile("junit:junit:4.11")
    testCompile("io.vertx:vertx-unit:$vertx_version")
}

repositories {
    mavenCentral()
    jcenter()
}

val fatJar = task("fatJar", type = Jar::class) {
    baseName = application.applicationName
    manifest {
        attributes["Main-Class"] = application.mainClassName
    }
    from(configurations.runtime.map {
        if (it.isDirectory) it else zipTree(it)
    })
    with(tasks["jar"] as CopySpec)
}

tasks {
    "build" {
        dependsOn(fatJar)
    }
}

```

Afterwards you can `build` and `run` your application. 

1.  ./gradlew build
2.  ./gradlew run

You can check out my sample code on
[GitHub](https://github.com/s1monw1/kotlin_vertx_example/blob/master/README.md).
It includes some examples on HTTP routing using *Vert.x web*, also
showing how to work with template engines.
