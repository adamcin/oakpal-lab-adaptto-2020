# Exercise 3: Writing your own checks

## Objectives

* Write a JavaScript Progress Check for the `all` package and add the check to the OakPAL Scan.

* Write a Java Progress Check for the `all` package and add the check to the scan.

* Write a Groovy script check for the `all` package and add the check to the OakPAL Scan.

## Exercise

The basic task for defining an OakPAL check from scratch to implement the [`ProgressCheck` interface](https://github.com/adamcin/oakpal/blob/v2.2.1/api/src/main/java/net/adamcin/oakpal/api/ProgressCheck.java).
Fundamentally, this is a simple Java interface, but OakPAL also includes support for script-based progress checks via the JSR-223 
scripting API. 

We will get our feet wet with writing a JavaScript check to explore the interface, then we will implement the same logic 
in a Java `ProgressCheck` class.

### Writing a JavaScript Check

Before we jump in, here are some basic facts about how check implementations are loaded by the `oakpal-maven-plugin`.

* A `<check>` element listed in the plugin configuration that has a child element named `<impl>` will attempt to load a 
script or class by that fully-qualified class name or resource name from the plugin classpath or the module's 
`test`-scope classpath. This includes:

  * any built-in dependencies of the `oakpal-maven-plugin` plugin, including `oakpal-api` and `oakpal-core`.

  * the `<dependencies>` added directly to the `oakpal-maven-plugin` plugin descriptor, which can be inherited from a 
  parent pom.
  
  * the `<dependency>` elements in the current pom (such as `classic-app/all/pom.xml`) marked as `<scope>test</scope>`.
  
  * the contents of the `${project.build.testOutputDirectory}` folder (i.e. `target/test-classes`), which can be 
  copied as-is from `src/test/resources`, or compiled from `src/test/java`, `src/test/kt`, etc.
  
  * oakpal checklist modules can define their own dependencies, which should come along for the ride when you use them 
  in your project. 
  

First, under `classic-app/all/src`, create a `test` folder, and a `resources` folder under that. The 
`classic-app/all/src/test/resources` path should now exist.

Create a new file under your new folder named `myFirstCheck.js`. It should now exist at the path, 
`classic-app/all/src/test/resources/myFirstCheck.js`. Leave it empty for now.

Next, add a `<check>` to the plugin in `classic-app/all/pom.xml` for your new progress check. Add an 
`<impl>myFirstCheck.js</impl>` element under that. While we're at it, let's replace the basic checklist and its 
overrides in our configuration with just a single spec to keep the `sling-jcr-installer` check enabled,
to reduce the noise while we focus on our check development. It should now look like this:

```xml
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <configuration>
        <checks>
            <check>
                <name>sling-jcr-installer</name>
            </check>
            <check>
                <impl>myFirstCheck.js</impl>
            </check>
        </checks>
    </configuration>
</plugin>
```

Now run `mvn clean install`. This should succeed. From now on, while we are working within the `all` module, you can
run a quicker command that avoids rebuilding all the dependencies every time: 

    mvn clean install -pl :classic-app.all

Congratulations! You have implemented your first valid Progress Check. 

An empty javascript file is completely valid. It's loaded (and eval'd). It just doesn't do anything, yet.

The easiest way to prove that it's working is to quickly change the script while muttering, "It's only JavaScript," 
and then wait to see what happens at runtime.

Add this statement to your script:

```js
println("Do I look like a JavaScript expert?");
```

Now run `mvn clean install -pl :classic-app.all` again. You should experience a catastrophic failure. See? It WAS 
working before.

```
[INFO] --- oakpal-maven-plugin:2.2.2-SNAPSHOT:scan (default-scan) @ classic-app.all ---
[INFO] Sling-Nodetypes: Discovered node types: jar:file:/Users/madamcin/.m2/repository/biz/netcentric/aem/aem-nodetypes/6.5.5.0/aem-nodetypes-6.5.5.0.jar!/aem.cnd
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.139 s
[INFO] Finished at: 2020-09-13T11:57:23-07:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal net.adamcin.oakpal:oakpal-maven-plugin:2.2.2-SNAPSHOT:scan (default-scan) on project classic-app.all: Failed to execute package scan. Error while loading progress checks.: Failed to load package check . (impl: myFirstCheck.js): ReferenceError: "println" is not defined in <eval> at line number 17 -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
```

The `oakpal` plugin reports this failure:

    Failed to load package check . (impl: myFirstCheck.js): ReferenceError: "println" is not defined in <eval> at line number 17
    
This tells us the check name (we didn't provide one), the `impl`, much more helpful, and the exception message, which 
we can easily correct.

Let's change the `println` to `print` so that that line of the script becomes:

```js
print("Do I look like a JavaScript expert?");
```

Now run `mvn clean install -pl :classic-app.all`. You should see a successful result. If you look closely, you should 
see the message effect of that ~~println~~ print statement in the output:

```
[INFO] --- oakpal-maven-plugin:2.2.2-SNAPSHOT:scan (default-scan) @ classic-app.all ---
[INFO] Sling-Nodetypes: Discovered node types: jar:file:/Users/madamcin/.m2/repository/biz/netcentric/aem/aem-nodetypes/6.5.5.0/aem-nodetypes-6.5.5.0.jar!/aem.cnd
Do I look like a JavaScript expert?
[INFO] Found a new index node [reference]. Reindexing is requested
```

You should be feeling pretty good right about now, but we really haven't used any OakPAL features yet. Take a moment to 
read through the javascript check template, [exampleCheck.js](https://github.com/adamcin/oakpal/blob/master/maven/src/site/resources/exampleCheck.js), 
which stubs out all the supported ProgressCheck events that you can copy and paste as necessary.

At the beginning of the file, a javadoc comment describes the high-level behavior of a Script Progress Check.

```js
/**
 * 1. Implement the optional listener functions stubbed out below to perform your Oak package acceptance tests.
 *
 * 2. Use the oakpal global object to report violations
 *
 *   For MINOR violations:
 *     oakpal.minorViolation(description, packageId);
 *
 *   For MAJOR violations:
 *     oakpal.majorViolation(description, packageId);
 *
 *   For SEVERE violations:
 *     oakpal.severeViolation(description, packageId);
 *
 * 3. If configuration parameters are available, they can be accessed using the "config" global object.
 *
 * Java type imports for reference:
 *
 * import org.apache.jackrabbit.vault.packaging.PackageId;
 * import org.apache.jackrabbit.vault.packaging.PackageProperties;
 * import org.apache.jackrabbit.vault.fs.config.MetaInf;
 * import java.io.File;
 * import java.util.jar.Manifest;
 * import javax.jcr.Node;
 * import javax.jcr.Session;
 * import net.adamcin.oakpal.api.PathAction;
 * import net.adamcin.oakpal.api.SlingSimulator;
 * import net.adamcin.oakpal.api.SlingInstallable;
 * import net.adamcin.oakpal.api.EmbeddedPackageInstallable;
 */
```

One of the most commonly used events is the `importedPath` event. 

```js
/**
 * Notified when package importer adds, modifies, or leaves a node untouched.
 *
 * @param packageId         the current package
 * @param path              the imported path
 * @param node              the imported JCR node
 * @param action            the import action (NOOP, ADDED, MODIFIED, or REPLACED)
 */
function importedPath(packageId /* PackageId */, path /* String */, node /* Node */, action /* PathAction */) {

}
```

Paste the snippet above into your script. Then, add the following statement inside the new function to demonstrate how 
verbose you can be:

```js
print("imported a path: " + path);
```

Now run `mvn clean install -pl :classic-app.all`. You should see just about every path affected by your `all` package 
installation listed in the build output, before the build ultimately succeeds.

```
...
imported a path: /content/dam/core-components-examples/library/jcr:content/folderThumbnail/jcr:content
imported a path: /content/dam/core-components-examples/jcr:content/folderThumbnail
imported a path: /content/dam/core-components-examples/jcr:content/folderThumbnail/jcr:content
imported a path: /content/experience-fragments
imported a path: /content/experience-fragments/core-components-examples
imported a path: /content/experience-fragments/core-components-examples/library
imported a path: /content/experience-fragments/core-components-examples/library/simple-experience-fragment
imported a path: /content/experience-fragments/core-components-examples/library/simple-experience-fragment/jcr:content
imported a path: /content/experience-fragments/core-components-examples/library/simple-experience-fragment/master
imported a path: /content/experience-fragments/core-components-examples/library/simple-experience-fragment/master/jcr:content
imported a path: /content/experience-fragments/core-components-examples/library/simple-experience-fragment/master/jcr:content/root
imported a path: /content/experience-fragments/core-components-examples/library/simple-experience-fragment/master/jcr:content/root/image
imported a path: /content/experience-fragments/core-components-examples/library/simple-experience-fragment/master/jcr:content/root/text
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] 
```

Let's be less verbose, and focus on a single path that is created by our ui.content package. Specifically, 
`/content/classic-app/us/en/jcr:content`, should be interesting enough for our needs. Let's only print our statement 
for that particular path, and let's raise the stakes by setting the path itself to a global variable. Our script will 
look like this:

```js
print("Do I look like a JavaScript expert?");

var myInterestingPath = "/content/classic-app/us/en/jcr:content";

/**
 * Notified when package importer adds, modifies, or leaves a node untouched.
 *
 * @param packageId         the current package
 * @param path              the imported path
 * @param node              the imported JCR node
 * @param action            the import action (NOOP, ADDED, MODIFIED, or REPLACED)
 */
function importedPath(packageId /* PackageId */, path /* String */, node /* Node */, action /* PathAction */) {
    if (path == myInterestingPath) {
        print("imported a path: " + path);
    }
}
```

Run `mvn clean install -pl :classic-app.all`, and confirm that you get some output like this:

```
[INFO] --- oakpal-maven-plugin:2.2.2-SNAPSHOT:scan (default-scan) @ classic-app.all ---
[INFO] Sling-Nodetypes: Discovered node types: jar:file:/Users/madamcin/.m2/repository/biz/netcentric/aem/aem-nodetypes/6.5.5.0/aem-nodetypes-6.5.5.0.jar!/aem.cnd
Do I look like a JavaScript expert?
[INFO] Found a new index node [reference]. Reindexing is requested
[INFO] Reindexing will be performed for following indexes: [/oak:index/uuid, /oak:index/reference, /oak:index/nodetype]
[INFO] Indexing report
    - /oak:index/uuid*(0)
    - /oak:index/reference*(0)
    - /oak:index/nodetype*(1258)

[INFO] Reindexing completed
[INFO] Reindexing will be performed for following indexes: [/oak:index/principalName, /oak:index/authorizableId, /oak:index/acPrincipalName, /oak:index/repMembers]
[INFO] Indexing report
    - /oak:index/principalName*(2)
    - /oak:index/authorizableId*(2)
    - /oak:index/acPrincipalName*(0)
    - /oak:index/repMembers*(0)

[INFO] Reindexing completed
[INFO] Node type change for vlt:PackageDefinition appears to be trivial
[INFO] Node type changes: [vlt:PackageDefinition] appear to be trivial, repository will not be scanned
imported a path: /content/classic-app/us/en/jcr:content
...
```

If you see `imported a path: /content/classic-app/us/en/jcr:content`, and no other paths printed with that pattern, 
you can give yourself a pat on the back.

But production-ready OakPAL checks shouldn't print to standard output. It's rude. Instead, checks should limit their 
output to reported violations.

Let's have some fun and turn our `print` statement in the `importedPath` function into a minor violation! (P.S. You can 
now remove that other print statement at the top of the script, if you haven't already).

Scripts are provided an `oakpal` helper variable in the global context. It has a handful of convenient functions 
defined for reporting violations.

```
 *   For MINOR violations:
 *     oakpal.minorViolation(description, packageId);
 *
 *   For MAJOR violations:
 *     oakpal.majorViolation(description, packageId);
 *
 *   For SEVERE violations:
 *     oakpal.severeViolation(description, packageId);
```

Rewrite your `print` statement to instead call:

    oakpal.minorViolation("imported a path: " + path, packageId);
    
The script should now look like this:

```js
var myInterestingPath = "/content/classic-app/us/en/jcr:content";

/**
 * Notified when package importer adds, modifies, or leaves a node untouched.
 *
 * @param packageId         the current package
 * @param path              the imported path
 * @param node              the imported JCR node
 * @param action            the import action (NOOP, ADDED, MODIFIED, or REPLACED)
 */
function importedPath(packageId /* PackageId */, path /* String */, node /* Node */, action /* PathAction */) {
    if (path == myInterestingPath) {
        oakpal.minorViolation("imported a path: " + path, packageId);
    }
}
```

When you run `mvn clean install -pl :classic-app.all`, it should still succeed, but your message should be included
as a minor violation under the `myFirstCheck.js` report.

```
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] OakPAL Check Reports
[INFO]   myFirstCheck.js
[INFO]     +- <MINOR> imported a path: /content/classic-app/us/en/jcr:content [classic-app.ui.content-1.0-SNAPSHOT.zip]
```

However, it's probably not realistic to design your check to report a violation when it imports a node that you are 
expecting to import. Instead, a more likely requirement is that it should report a violation when a scan fails to 
import an expected node. 

This will require a slightly more complex design, but it's still doable. We'll need to handle another ProgressCheck 
event, so that we can report a violation at the end of the scan if our expected path wasn't imported earlier.

Our new script version will still handle the `importedPath` event, but instead of reporting a violation immediately, 
it will set a global boolean variable indicating that the path in question was in fact imported. Then, we'll implement 
the simple `finishedScan` event to report a major violation.

Change your script to match:

```js
var myInterestingPath = "/content/classic-app/us/en/jcr:content";

var myPathWasImported = false;

/**
 * Notified when package importer adds, modifies, or leaves a node untouched.
 *
 * @param packageId         the current package
 * @param path              the imported path
 * @param node              the imported JCR node
 * @param action            the import action (NOOP, ADDED, MODIFIED, or REPLACED)
 */
function importedPath(packageId /* PackageId */, path /* String */, node /* Node */, action /* PathAction */) {
    if (path == myInterestingPath) {
        myPathWasImported = true;
    }
}

/**
 * Called once at the end of the scan.
 */
function finishedScan() {
    if (!myPathWasImported) {
        oakpal.majorViolation("did NOT import my path: " + myInterestingPath);
    }
}
```

Of course, this shouldn't actually fail now, because we know our interesting path does get imported. Let's change the 
`myInterestingPath` variable temporarily to one that won't exist, so that we can confirm that our check logic does what 
it's supposed to:

```js
var myInterestingPath = "/content/classic-app/us/en/jcr:malcontent";
```

Run `mvn clean install -pl :classic-app.all` again, and expect the build to fail with your new major violation:

```
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] OakPAL Check Reports
[INFO]   myFirstCheck.js
[ERROR]    +- <MAJOR> did NOT import my path: /content/classic-app/us/en/jcr:malcontent
[ERROR] ** Violations were reported at or above severity: MAJOR **
```

Change the `myInterestingPath` variable back and run the build again, to verify success:

```js
var myInterestingPath = "/content/classic-app/us/en/jcr:content";
```

If your build was a success, but you feel like maybe leaving your check implemented as a raw JS file with the look 
and feel of an HTML form onSubmit validation script from late 2003, you may now try converting the same logic into 
beautiful, polymorphic, type-checked Java code.

### Writing a Java ProgressCheck

We'll skip the verbosity about the ProgressCheck events and move right to the pom.xml changes. The benefit of writing 
your checks in Java is that you get the benefit of real-time type checking in your IDE, instead of having to
use `mvn clean install -pl :classic-app.all` as a poor man's REPL.

Of course, this requires explicitly adding the `oakpal-core` dependency to your `classic-app/all/pom.xml`, but first,
let's prepare for proper version management, since this will dependency will likely be reused in other modules going 
forward.

In the `classic-app/pom.xml` parent pom, add the following dependency at the end of the `<depedencyManagement>` section:

```xml
        <dependency>
            <groupId>net.adamcin.oakpal</groupId>
            <artifactId>oakpal-core</artifactId>
            <version>${oakpal.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Then, in `all` package pom, `classic-app/all/pom.xml`, add the following dependency to the end of the `<dependencies>`
section. Notice the `<scope>test</scope>` element:

```xml
    <dependency>
        <groupId>net.adamcin.oakpal</groupId>
        <artifactId>oakpal-core</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

Now, under `classic-app/all/src/test`, create a `java` folder, and an `oakpal` folder under that. The 
`classic-app/all/src/test/java/oakpal` path should now exist, representing the `oakpal` java package in the 
`src/test/java` folder in the `all` package module.

In the `oakpal` java package, create a new Java class file named `MyFirstJavaCheck.java`. Just go ahead and paste the 
following into it:

```java
package oakpal;

import net.adamcin.oakpal.api.PathAction;
import net.adamcin.oakpal.api.Severity;
import net.adamcin.oakpal.api.SimpleProgressCheck;
import org.apache.jackrabbit.vault.packaging.PackageId;

import javax.jcr.Node;
import javax.jcr.RepositoryException;

public class MyFirstJavaCheck extends SimpleProgressCheck {
    private final String myInterestingPath = "/content/classic-app/us/en/jcr:malcontent"; // notice the bad path
    private boolean myPathWasImported = false;

    @Override
    public void importedPath(final PackageId packageId, final String path, final Node node, final PathAction action) throws RepositoryException {
        if (myInterestingPath.equals(path)) {
            myPathWasImported = true;
        }
    }

    @Override
    public void finishedScan() {
        if (!myPathWasImported) {
            reporting(builder -> builder
                    .withSeverity(Severity.MAJOR)
                    .withDescription("did NOT import my path: " + myInterestingPath));
        }
    }
}
```

At a quick glance, you should be able to recognize the similarities with the `myFirstCheck.js` script. The 
`importedPath` and `finishedScan` method signatures are nearly identical, and even though the syntactic sugar for the 
`reporting` method is a little different than the JavaScript `oakpal` helper, the translation from the JS to the Java 
should not require much explanation.

Save the file, then edit the `classic-app/all/pom.xml` the change the `<check>` element in your plugin config reference
the Java implementation instead of the JS implementation:

```xml
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <configuration>
        <checks>
            <check>
                <name>sling-jcr-installer</name>
            </check>
            <check>
                <impl>oakpal.MyFirstJavaCheck</impl>
            </check>
        </checks>
    </configuration>
</plugin>
```

Now, run `mvn clean install -pl :classic-app.all` and, once again, expect a single major violation that breaks the 
build, with a description just like the JS script, but with the new check name, `MyFirstJavaCheck`:

```
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] OakPAL Check Reports
[INFO]   MyFirstJavaCheck
[ERROR]    +- <MAJOR> did NOT import my path: /content/classic-app/us/en/jcr:malcontent
[ERROR] ** Violations were reported at or above severity: MAJOR **
```

Same violation report as before, except the check name is different.

Now edit `MyFirstCheck.java` and fix the value of `myInterestingPath`:

```java
private final String myInterestingPath = "/content/classic-app/us/en/jcr:content";
```    

Save the file, and rerun `mvn clean install -pl :classic-app.all`. The command should once again succeed:

```
[INFO] --- oakpal-maven-plugin:2.2.2-SNAPSHOT:scan (default-scan) @ classic-app.all ---
[INFO] Sling-Nodetypes: Discovered node types: jar:file:/Users/madamcin/.m2/repository/biz/netcentric/aem/aem-nodetypes/6.5.5.0/aem-nodetypes-6.5.5.0.jar!/aem.cnd
[INFO] Found a new index node [reference]. Reindexing is requested
[INFO] Reindexing will be performed for following indexes: [/oak:index/uuid, /oak:index/reference, /oak:index/nodetype]
[INFO] Indexing report
    - /oak:index/uuid*(0)
    - /oak:index/reference*(0)
    - /oak:index/nodetype*(1258)

[INFO] Reindexing completed
[INFO] Reindexing will be performed for following indexes: [/oak:index/principalName, /oak:index/authorizableId, /oak:index/acPrincipalName, /oak:index/repMembers]
[INFO] Indexing report
    - /oak:index/principalName*(2)
    - /oak:index/authorizableId*(2)
    - /oak:index/acPrincipalName*(0)
    - /oak:index/repMembers*(0)

[INFO] Reindexing completed
[INFO] Node type change for vlt:PackageDefinition appears to be trivial
[INFO] Node type changes: [vlt:PackageDefinition] appear to be trivial, repository will not be scanned
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO]

```

### Writing a Groovy Script Check

We're really moving now. Let's return to the JSR-223 support demonstrated by the JavaScript check, and see if we can use
a different flavor of scripting that might give us better real-time type checking. We'll continue using our basic check
logic, to expect that the package imports our interesting path. 

To use the `importedPath` signature with typed arguments, you will need the `oakpal-core` dependency on the `test`-scope
class path that we added for the Java check.

We will also need to add the `groovy-all` pom dependency the same way, so that oakpal has access to the groovy script
engine. Edit `classic-app/all/pom.xml` and add it to the end of the `<dependencies>` element, immediately after your 
`oakpal-core` dependency, like this:

```xml
        <dependency>
            <groupId>net.adamcin.oakpal</groupId>
            <artifactId>oakpal-core</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy-all</artifactId>
            <version>3.0.5</version>
            <scope>test</scope>
            <type>pom</type>
        </dependency>
    </dependencies>
```

Because we intend to load this as a script, not compiled as class, the proper location for the file is the same 
`src/test/resources` folder that contains `myFirstCheck.js`. So, create a new file in that folder named 
`myFirstGroovyCheck.groovy`, and paste the following:

```groovy
import groovy.transform.Field
import net.adamcin.oakpal.api.PathAction
import org.apache.jackrabbit.vault.packaging.PackageId

import javax.jcr.Node

@Field String myInterestingPath = "/content/classic-app/us/en/jcr:malcontent" // notice the bad path

@Field boolean myPathWasImported = false

def importedPath(PackageId packageId, String path, Node node, PathAction action) {
    if (path == myInterestingPath) {
        myPathWasImported = true
    }
}

def finishedScan() {
    if (!myPathWasImported) {
        oakpal.majorViolation("did NOT import my path: " + myInterestingPath)
    }
}
```

Save the file, then edit the `classic-app/all/pom.xml` the change the `<check>` element in your plugin config reference
the Groovy implementation instead of the Java implementation:

```xml
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <configuration>
        <checks>
            <check>
                <name>sling-jcr-installer</name>
            </check>
            <check>
                <impl>myFirstGroovyCheck.groovy</impl>
            </check>
        </checks>
    </configuration>
</plugin>
```

Now, run `mvn clean install -pl :classic-app.all` and, once again, expect a single major violation that breaks the 
build, with a description just like the JS script, but with the new check name, `myFirstGroovyCheck.groovy`:

```
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] OakPAL Check Reports
[INFO]   myFirstGroovyCheck.groovy
[ERROR]    +- <MAJOR> did NOT import my path: /content/classic-app/us/en/jcr:malcontent
[ERROR] ** Violations were reported at or above severity: MAJOR **
```

Same violation report as before, except the check name is different.

Now edit `myFirstGroovyCheck.groovy` and fix the value of `myInterestingPath`:

```groovy
@Field String myInterestingPath = "/content/classic-app/us/en/jcr:content"
```    

Save the file, and rerun `mvn clean install -pl :classic-app.all`. The command should once again succeed:

```
[INFO] --- oakpal-maven-plugin:2.2.2-SNAPSHOT:scan (default-scan) @ classic-app.all ---
[INFO] Sling-Nodetypes: Discovered node types: jar:file:/Users/madamcin/.m2/repository/biz/netcentric/aem/aem-nodetypes/6.5.5.0/aem-nodetypes-6.5.5.0.jar!/aem.cnd
[INFO] Found a new index node [reference]. Reindexing is requested
[INFO] Reindexing will be performed for following indexes: [/oak:index/uuid, /oak:index/reference, /oak:index/nodetype]
[INFO] Indexing report
    - /oak:index/uuid*(0)
    - /oak:index/reference*(0)
    - /oak:index/nodetype*(1258)

[INFO] Reindexing completed
[INFO] Reindexing will be performed for following indexes: [/oak:index/principalName, /oak:index/authorizableId, /oak:index/acPrincipalName, /oak:index/repMembers]
[INFO] Indexing report
    - /oak:index/principalName*(2)
    - /oak:index/authorizableId*(2)
    - /oak:index/acPrincipalName*(0)
    - /oak:index/repMembers*(0)

[INFO] Reindexing completed
[INFO] Node type change for vlt:PackageDefinition appears to be trivial
[INFO] Node type changes: [vlt:PackageDefinition] appear to be trivial, repository will not be scanned
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO]

```

Once you have reached this point, you have become familiar enough with the features of OakPAL to be able to use it 
effectively on your own projects to test for a wide variety of scenarios specific to your organization and application
requirements.

The upcoming exercises will demonstrate how to make your OakPAL checks configurable, and then to centralize your oakpal 
checks into a single shared module within your maven project and publish your own checklist. Later exercises will 
demonstrate how to operationalize your maven scan configuration as an Oakpal Plan that can even be run outside of Maven 
in a CI container pipeline.

[Continue to Exercise 4: Designing OakPAL Checks for Reuse](Exercise_04.md)
