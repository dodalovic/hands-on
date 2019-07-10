# Common Rendering and JVM

It is always good to question ourselves - do we really need the
JVM backend for the project or not. What if we would be able
to run the [Mandelbrot Set](https://en.wikipedia.org/wiki/Mandelbrot_set)
rendering on the client? 

Let's implement the client-side rendering and see how it goes.

## Sharing Code

We've just added the very first Kotlin file into the `commonMain`
source set. Let's move all the files from the `jvm` source set
into the `common` source set to share the code with JavaScript.

Wait a moment. It will not work directly this way because we use JVM APIs
to render images in our code. We need to pick another APIs to use in
a browser Kotlin/JS part.
We will be using the
[HTML Canvas](https://www.w3schools.com/html/html5_canvas.asp)
to render our images in JavaScript.

It is easy to share code between platforms with Kotlin. 
[Kotlin Multiplatform](https://kotlinlang.org/docs/reference/multiplatform.html)
introduces `expect` and `actual` keywords especially for that. 
We will use the `expect` keyword in the `commonMain` sources specify
the declarations that we can only implement for a given target.
The `actual` keyword binds the `expect`-ed declarations with their
platform specific counterparts.

Let's see how it works.

## Gradle Dependency

Before we proceed with the code moving, we should include yet another Gradle
dependency for the `commonMain` source set. For that let's add few more lines
to the `build.gradle.kts`:

```kotlin
kotlin.sourceSets["commonMain"].dependencies {
  implementation(kotlin("stdlib-common"))
  implementation("org.jetbrains.kotlinx:kotlinx-html-common:0.6.12")
}
```

That code adds the dependency from the Kotlin standard library. In addition
to that we include the dependency from the
[kotlinx.html](https://github.com/Kotlin/kotlinx.html)
common library. It opens the ways to re-use the HTML rendering
code between our JVM server-side and JS client-side.

We may need to refresh the Gradle project model now to make sure
the IDE sees all our project model changes. Let's click on the _refresh_
icon in the Gradle tab to implement it

## Moving Code

Sharing code can be done with the following algorithm

* move all source files into the `src/commonMain/kotlin` folder
* replace missing platform-specific symbols with `expect` declarations
* add `actual` keyword for the `expect` declarations for all platforms

Now it is time to run the algorithm on our project. We move
the following files from the `src/jvmMain/kotlin` into `src/commonMain/kotlin`:

* `colour.kt`
* `complex.kt`
* `geometry.kt`
* `render.kt`

We do not move the `main.kt` and `graphics.kt` files because these files are
platform specific. The first one starts the HTTP server, which we do not need
on the client-side. The graphics.kt` file uses AWT to render the image.
     
There should be several errors in the `colour.kt` file. We miss the
`java.awt.Color` class definition. Let's fix it by adding the `expect`
declarations for colors in the `src/commonMain/kotlin/common.kt`:

```kotlin
object Colors

expect class Color
expect fun Colors.newColor(r: Int, g: Int, b: Int): Color
expect val Colors.BLACK : Color
```

The `expect class`, `expect fun`, and `expect val` declarations in the Kotlin/Common
code instructs the compiler that we'd like to use these declarations in our Kotlin/Common
code, but we are not able to provide implementations. Implementations are provided via the
`actual` keyword independently for the JVM and JS targets (in general,
[Kotlin/Native](https://kotlinlang.org/docs/reference/native-overview.html) can
also be [used](https://kotlinlang.org/docs/tutorials/native/mpp-ios-android.html)
to target iOS and other platforms).   

Now we fix the code in the `colour.kt` file to use our expected `Color` class
and the `Colors` object to deal with colors (instead of using the `java.awt.Color` class).
We need to remove the import in the `colour.kt` and to use the `Colors.newColor`
instead of the `Color` constructor call.

The last fix has to be performed in the `render.kt` file. We need to remove the
import of the `java.awt.Color` class and to switch to the `Colors.BLACK`
property in the middle of the file.

## JVM Implementation

We've just fixed the code in the `commonMain` source set by using the `expect` declarations.
It means that every target (e.g. JVM or JS) has to provide the `actual` declarations
for every `expect` declarations in the `commonMain`. Let's fix the `jvmMain`
source set first.

We will need to re-create the `colour-actual.kt` file under the `src/jvmMain/kotlin` folder
with the following `actual` declarations:

```kotlin
actual typealias Color = java.awt.Color

actual fun Colors.newColor(r: Int, g: Int, b: Int) = Color(r, g, b)
actual val Colors.BLACK: Color get() = Color.BLACK
```
 
By these declarations we say that our expected class `Color`
should be `java.awt.Color` class from the JVM. We also provide implementations for
two more required functions.

We should be able to start the JVM application now. Let's check
it by running the Gradle `run` task. We should see the similar output
in the console

```
The Manderbrot renderer is started at http://127.0.0.1:8888

Open the following in a browser
  http://127.0.0.1:8888/mandelbrot

For a zoomed image try
  http://127.0.0.1:8888/mandelbrot?top=-0.32&left=-0.72&bottom=-0.28&right=-0.68
  http://127.0.0.1:8888/mandelbrot?top=-0.7&left=0.1&bottom=-0.6&right=0.2

```

## Completed Code

We may use the `step-005` branch of the
[github.com/kotlin-hands-on/intro-mpp](https://github.com/kotlin-hands-on/intro-mpp)
repository as the solution of the tasks that we done above. 
We may also just download the
[archive](https://github.com/kotlin-hands-on/intro-mpp/archive/step-005.zip)
from GitHub directly.
  