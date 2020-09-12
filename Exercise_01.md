# Exercise 1: First Steps with OakPAL

## Objectives

* Add the `oakpal-maven-plugin` to an existing multi-module maven project.

* Configure the plugin to initialize an Oak repository with proprietary AEM nodetypes, privileges, 
and namespaces.

* Execute a simple scan of the `all` content-package artifact that includes all of its embedded packages in the scope
of its scan.

## Exercise

Build classic-app.

```bash
cd classic-app
mvn clean install
```

Expect a successful build result:

```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for classic-app 1.0-SNAPSHOT:
[INFO]
[INFO] classic-app ........................................ SUCCESS [  0.320 s]
[INFO] AEM Classic App - Core ............................. SUCCESS [  7.458 s]
[INFO] AEM Classic App - UI Frontend ...................... SUCCESS [ 11.784 s]
[INFO] AEM Classic App - Repository Structure Package ..... SUCCESS [  1.001 s]
[INFO] AEM Classic App - UI apps .......................... SUCCESS [  5.403 s]
[INFO] AEM Classic App - UI content ....................... SUCCESS [  3.077 s]
[INFO] AEM Classic App - UI config ........................ SUCCESS [  0.187 s]
[INFO] AEM Classic App - All .............................. SUCCESS [  0.411 s]
[INFO] AEM Classic App - Integration Tests ................ SUCCESS [  5.779 s]
[INFO] AEM Classic App - UI Tests ......................... SUCCESS [  0.312 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  37.095 s
[INFO] Finished at: 2020-09-12T07:29:41-07:00
[INFO] ------------------------------------------------------------------------
```

Now, add `oakpal-maven-plugin` to pluginManagement in parent pom, `classic-app/pom.xml`. Add this after 
the existing `filevault-package-maven-plugin` definition. 

```xml
<!-- OakPAL Plugin -->
<plugin>
   <groupId>net.adamcin.oakpal</groupId>
   <artifactId>oakpal-maven-plugin</artifactId>
   <version>2.2.1</version>
   <executions>
       <execution>
           <id>default-scan</id>
           <goals>
               <goal>scan</goal>
           </goals>
       </execution>
   </executions>
</plugin> 
```

Then, add `oakpal-maven-plugin` to the `all` content-package module pom (`classic-app/all/pom.xml`) in 
`build/plugins` after the `filevault-package-maven-plugin`.

```xml
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
</plugin>
```

Run `mvn clean install` again and confirm that the oakpal-maven-plugin:scan goal was executed 
during the classic-app.all module.

```
[INFO] --- oakpal-maven-plugin:2.2.1:scan (default-scan) @ classic-app.all ---
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
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
```

If you examine the report summary identified by the `Check report summary written to` line, you will see that only one 
reporter was active. The DefaultErrorListener, with no other checks listed:

```json
{
  "reports":[
    {
      "checkName":"DefaultErrorListener"
    }
  ]
}
```

This is not going to tell us much without more checks, because without specifying any checks, OakPAL only attempts to 
install the package into a clean Oak repository, without handling Sling features like Embedded Packages. This `classic-app.all` 
package is totally valid for installation into a clean Oak repository, because all of the AEM dependencies are found in 
its embedded packages, which OakPAL is currently ignoring.

Let's see what happens when we add the check that enables the Sling embedded packages support. 
In `classic-app/all/pom.xml`, add the `configuration` element to the `oakpal-maven-plugin` descriptor:

```
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <configuration>
        <checks>
            <check>
                <name>basic/sling-jcr-installer</name>
            </check>
        </checks>
    </configuration>
</plugin>
```

Now run `mvn clean install` again, and don't panic:

```
[ERROR]    +- <MAJOR> /content/dam/core-components-examples/library/sample-assets/simple-fragment/jcr:content/renditions/short - Importer error: java.lang.IllegalStateException "Parent node not found." [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /content/dam/core-components-examples/library/jcr:content - Importer error: java.lang.IllegalStateException "Parent node not found." [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /content/dam/core-components-examples/library/jcr:content/folderThumbnail - Importer error: java.lang.IllegalStateException "Parent node not found." [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /content/dam/core-components-examples/jcr:content - Importer error: java.lang.IllegalStateException "Parent node not found." [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /content/dam/core-components-examples/jcr:content/folderThumbnail - Importer error: java.lang.IllegalStateException "Parent node not found." [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /content/experience-fragments/core-components-examples - Importer error: javax.jcr.nodetype.NoSuchNodeTypeException "Node type sling:OrderedFolder does not exist" [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /content/experience-fragments/core-components-examples/library - Importer error: java.lang.IllegalStateException "Parent node not found." [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /content/experience-fragments/core-components-examples/library/simple-experience-fragment - Importer error: java.lang.IllegalStateException "Parent node not found." [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /content/experience-fragments/core-components-examples/library/simple-experience-fragment/master - Importer error: java.lang.IllegalStateException "Parent node not found." [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR] ** Violations were reported at or above severity: MAJOR **
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for classic-app 1.0-SNAPSHOT:
[INFO]
[INFO] classic-app ........................................ SUCCESS [  0.298 s]
[INFO] AEM Classic App - Core ............................. SUCCESS [  9.253 s]
[INFO] AEM Classic App - UI Frontend ...................... SUCCESS [ 11.373 s]
[INFO] AEM Classic App - Repository Structure Package ..... SUCCESS [  1.014 s]
[INFO] AEM Classic App - UI apps .......................... SUCCESS [  4.870 s]
[INFO] AEM Classic App - UI content ....................... SUCCESS [  3.434 s]
[INFO] AEM Classic App - UI config ........................ SUCCESS [  0.263 s]
[INFO] AEM Classic App - All .............................. FAILURE [ 11.816 s]
[INFO] AEM Classic App - Integration Tests ................ SKIPPED
[INFO] AEM Classic App - UI Tests ......................... SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  43.780 s
[INFO] Finished at: 2020-09-12T09:09:42-07:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal net.adamcin.oakpal:oakpal-maven-plugin:2.2.1:scan (default-scan) on project classic-app.all: ** Violations were reported at or above severity: MAJOR ** -> [Help 1]
```

The scan is now failing with a bunch of errors, but they all derive from the same issue. Now that we enabled the 
Sling Jcr Installer check, we are detecting all the embedded packages under `/apps/.../install` and trying to install them,
but they contain content nodes that depend on proprietary AEM nodetypes. The `filevault-package-maven-plugin` is already
configured in our project to validate the packages according to AEM nodetypes, so luckily we just need to tell OakPAL about them.

Since this nodetype dependency will affect all executions, we might as well add this configuration to the pluginManagement 
declaration in `classic-app/pom.xml`:

```xml
<!-- OakPAL Plugin -->
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <version>2.2.1</version>
    <configuration>
        <slingNodeTypes>true</slingNodeTypes>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>biz.netcentric.aem</groupId>
            <artifactId>aem-nodetypes</artifactId>
            <version>6.5.5.0</version>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <id>default-scan</id>
            <goals>
                <goal>scan</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

We've made two additions here. First, we enabled the `slingNodeTypes` config flag, which triggers detection of 
`Sling-Nodetypes` Manifest headers in jars on the plugin and test-scope classpaths. Second, we added the same 
`aem-nodetypes` dependency that is listed in the `filevault-package-maven-plugin` descriptor in the same pom.xml file.

Now let's run `mvn clean install`, and once again, don't panic:

```
[ERROR] E /conf/core-components-examples/settings/wcm/templates/rep:policy (org.xml.sax.SAXException: No such privilege crx:replicate
javax.jcr.security.AccessControlException: No such privilege crx:replicate)
[ERROR] There were errors during package install. Please check the logs for details.
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] OakPAL Check Reports
[INFO]   DefaultErrorListener
[ERROR]    +- <MAJOR> /conf/classic-app/settings/cloudconfigs/commerce/rep:policy - Importer error: org.xml.sax.SAXException "No such privilege crx:replicate" [classic-app.ui.content-1.0-SNAPSHOT.zip]
[ERROR]    +- <MAJOR> /conf/classic-app/settings/wcm/policies/rep:policy - Importer error: org.xml.sax.SAXException "No such privilege crx:replicate" [classic-app.ui.content-1.0-SNAPSHOT.zip]
[ERROR]    +- <MAJOR> /conf/classic-app/settings/wcm/templates/rep:policy - Importer error: org.xml.sax.SAXException "No such privilege crx:replicate" [classic-app.ui.content-1.0-SNAPSHOT.zip]
[ERROR]    +- <MAJOR> /conf/classic-app/sling:configs/rep:policy - Importer error: org.xml.sax.SAXException "No such privilege crx:replicate" [classic-app.ui.content-1.0-SNAPSHOT.zip]
[ERROR]    +- <MAJOR> /conf/core-components-examples/sling:configs/rep:policy - Importer error: org.xml.sax.SAXException "No such privilege crx:replicate" [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /conf/core-components-examples/settings/wcm/policies/rep:policy - Importer error: org.xml.sax.SAXException "No such privilege crx:replicate" [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /conf/core-components-examples/settings/wcm/templates/rep:policy - Importer error: org.xml.sax.SAXException "No such privilege crx:replicate" [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR] ** Violations were reported at or above severity: MAJOR **
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for classic-app 1.0-SNAPSHOT:
[INFO]
[INFO] classic-app ........................................ SUCCESS [  0.332 s]
[INFO] AEM Classic App - Core ............................. SUCCESS [  7.042 s]
[INFO] AEM Classic App - UI Frontend ...................... SUCCESS [ 11.424 s]
[INFO] AEM Classic App - Repository Structure Package ..... SUCCESS [  1.019 s]
[INFO] AEM Classic App - UI apps .......................... SUCCESS [  4.949 s]
[INFO] AEM Classic App - UI content ....................... SUCCESS [  3.152 s]
[INFO] AEM Classic App - UI config ........................ SUCCESS [  0.275 s]
[INFO] AEM Classic App - All .............................. FAILURE [  7.130 s]
[INFO] AEM Classic App - Integration Tests ................ SKIPPED
[INFO] AEM Classic App - UI Tests ......................... SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  36.760 s
[INFO] Finished at: 2020-09-12T09:18:47-07:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal net.adamcin.oakpal:oakpal-maven-plugin:2.2.1:scan (default-scan) on project classic-app.all: ** Violations were reported at or above severity: MAJOR ** -> [Help 1]
```

Now we have only a relative handful of violations listed, which is still more than the zero violations we would expect, 
but luckily we can solve these with one additional configuration. All of these violations report the same SAXException,
`Importer error: org.xml.sax.SAXException "No such privilege crx:replicate"`. This is due to OakPAL not knowing about 
another proprietary AEM JCR dependency, the `crx:replicate` privilege, and its associated namespace URI. These are
trivial to add to the global OakPAL config in `classic-app/pom.xml`, so let's do that:

```xml
<!-- OakPAL Plugin -->
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <version>2.2.1</version>
    <configuration>
        <slingNodeTypes>true</slingNodeTypes>
        <jcrNamespaces>
            <namespace>
                <prefix>crx</prefix>
                <uri>http://www.day.com/crx/1.0</uri>
            </namespace>
        </jcrNamespaces>
        <jcrPrivileges>
            <privilege>crx:replicate</privilege>
        </jcrPrivileges>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>biz.netcentric.aem</groupId>
            <artifactId>aem-nodetypes</artifactId>
            <version>6.5.5.0</version>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <id>default-scan</id>
            <goals>
                <goal>scan</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Now when we run `mvn clean install`, the build should succeed.

```
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for classic-app 1.0-SNAPSHOT:
[INFO]
[INFO] classic-app ........................................ SUCCESS [  0.301 s]
[INFO] AEM Classic App - Core ............................. SUCCESS [  7.023 s]
[INFO] AEM Classic App - UI Frontend ...................... SUCCESS [ 11.324 s]
[INFO] AEM Classic App - Repository Structure Package ..... SUCCESS [  0.981 s]
[INFO] AEM Classic App - UI apps .......................... SUCCESS [  5.083 s]
[INFO] AEM Classic App - UI content ....................... SUCCESS [  3.428 s]
[INFO] AEM Classic App - UI config ........................ SUCCESS [  0.159 s]
[INFO] AEM Classic App - All .............................. SUCCESS [  6.321 s]
[INFO] AEM Classic App - Integration Tests ................ SUCCESS [  5.971 s]
[INFO] AEM Classic App - UI Tests ......................... SUCCESS [  0.357 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  42.332 s
[INFO] Finished at: 2020-09-12T09:43:15-07:00
[INFO] ------------------------------------------------------------------------
```

This configuration activates the bare minimum of automated testing features provided by OakPAL. It might not be obvious 
that any additional testing is happening, but you should note that the `oakpal-maven-plugin` is now confirming with every 
build that the JcrPackageManager will not throw an exception when the `all` package artifact is actually uploaded and 
installed to an AEM content repository. 

In the next exercise, we will explore how to ensure that your content package artifacts follow best practices published 
by other projects.

[Continue to Exercise 2: Integrating 3rd-Party Checklists](Exercise_02.md)