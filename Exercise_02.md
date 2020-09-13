# Exercise 2: Integrating OakPAL Checklists

## Objectives

* Integrate the `basic` checklist that is included with `oakpal-core` and override some configuration parameters.

* Embed the ACS AEM Commons Content Packages and integrate the `acs-commons-integrators` checklist to catch any issues
with how your project is using its features.

## Exercise

In Exercise 1, we added a single check to the `all` package scan, with the name `basic/sling-jcr-installer`. This name
indicates that we are referencing a check spec named `sling-jcr-installer` defined by the `basic` checklist in 
`oakpal-core` (because there aren't any other checklists on the classpath right now). 

NOTE: You can also reference this same check spec in a more specific, fully-qualified form 
(`net.adamcin.oakpal.core/basic/sling-jcr-installer`), or in a less specific form using only the name 
(`sling-jcr-installer`). 

The `basic` checklist defines many more standard package checks, which are documented on the oakpal website at 
[the basic checklist](http://adamcin.net/oakpal/oakpal-core/the-basic-checklist.html). Most of these checks are 
generally useful to include when scanning any package, and certainly provide a good starting point for AEM projects.

Instead of listing all of these checks one-by-one, let's replace the `<checks>` element in `classic-app/all/pom.xml` 
with a `<checklists>` element, where we will specify the `basic` checklist.

```xml
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <configuration>
        <checklists>
            <checklist>basic</checklist>
        </checklists>
    </configuration>
</plugin>
```

Now when you run `mvn clean install`, you will see the build fail with some new violations.

```
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] OakPAL Check Reports
[INFO]   net.adamcin.oakpal.core/basic/acHandling
[ERROR]    +- <MAJOR> acHandling mode MERGE is forbidden. allowed acHandling values in levelSet:only_add are [MERGE_PRESERVE, IGNORE] [classic-app.ui.apps-1.0-SNAPSHOT.zip]
[ERROR]    +- <MAJOR> acHandling mode MERGE is forbidden. allowed acHandling values in levelSet:only_add are [MERGE_PRESERVE, IGNORE] [classic-app.ui.content-1.0-SNAPSHOT.zip]
[ERROR]    +- <MAJOR> acHandling mode MERGE is forbidden. allowed acHandling values in levelSet:only_add are [MERGE_PRESERVE, IGNORE] [core.wcm.components.extensions.amp.content-2.11.0.zip]
[INFO]   net.adamcin.oakpal.core/basic/filterSets
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /conf/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/dam/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/experience-fragments/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /apps/rep:policy [core.wcm.components.extensions.amp.content-2.11.0.zip]
[ERROR] ** Violations were reported at or above severity: MAJOR **
```

The first thing to recognize is that violations may be reported at one of three severity levels, `MINOR`, `MAJOR`, or 
`SEVERE`. By default, `oakpal` will fail the build if any `MAJOR` or `SEVERE` violations are reported, but the 
`failOnSeverity` configuration property can be set to `MINOR` to be more strict, or `SEVERE` to be more lenient.

In the case of our most recent build, the violations reported by `basic/acHandling` were `MAJOR` violations, while those
reported by `basic/filterSets` were `MINOR` violations. With the default `failOnSeverity=MAJOR` config, we only have to
resolve the violations reported by `basic/acHandling` to get the build to pass.

The second thing to recognize is that even though we removed the explicit `basic/sling-jcr-installer` check spec from 
the plugin configuration, it is clearly still active, because the reported violations are all associated with embedded
package ids (as in `[classic-app.ui.apps-1.0-SNAPSHOT.zip]`), not the top level `classic-app.all` package id.

This can be confirmed by inspecting the oakpal-summary.json file, which should look something like the following:

```json
{
  "reports":[
    {
      "checkName":"DefaultErrorListener"
    },
    {
      "checkName":"net.adamcin.oakpal.core/basic/paths"
    },
    {
      "checkName":"net.adamcin.oakpal.core/basic/subpackages"
    },
    {
      "checkName":"net.adamcin.oakpal.core/basic/acHandling",
      "violations":[
        {
          "severity":"MAJOR",
          "description":"acHandling mode MERGE is forbidden. allowed acHandling values in levelSet:only_add are [MERGE_PRESERVE, IGNORE]",
          "packages":[
            "org.adaptto.oakpal:classic-app.ui.apps:1.0-SNAPSHOT"
          ]
        },
        {
          "severity":"MAJOR",
          "description":"acHandling mode MERGE is forbidden. allowed acHandling values in levelSet:only_add are [MERGE_PRESERVE, IGNORE]",
          "packages":[
            "org.adaptto.oakpal:classic-app.ui.content:1.0-SNAPSHOT"
          ]
        },
        {
          "severity":"MAJOR",
          "description":"acHandling mode MERGE is forbidden. allowed acHandling values in levelSet:only_add are [MERGE_PRESERVE, IGNORE]",
          "packages":[
            "adobe/cq60:core.wcm.components.extensions.amp.content:2.11.0"
          ]
        }
      ]
    },
    {
      "checkName":"net.adamcin.oakpal.core/basic/filterSets",
      "violations":[
        {
          "severity":"MINOR",
          "description":"non-default import mode MERGE defined for filterSet with root /conf/classic-app",
          "packages":[
            "org.adaptto.oakpal:classic-app.ui.content:1.0-SNAPSHOT"
          ]
        },
        {
          "severity":"MINOR",
          "description":"non-default import mode MERGE defined for filterSet with root /content/classic-app",
          "packages":[
            "org.adaptto.oakpal:classic-app.ui.content:1.0-SNAPSHOT"
          ]
        },
        {
          "severity":"MINOR",
          "description":"non-default import mode MERGE defined for filterSet with root /content/dam/classic-app",
          "packages":[
            "org.adaptto.oakpal:classic-app.ui.content:1.0-SNAPSHOT"
          ]
        },
        {
          "severity":"MINOR",
          "description":"non-default import mode MERGE defined for filterSet with root /content/experience-fragments/classic-app",
          "packages":[
            "org.adaptto.oakpal:classic-app.ui.content:1.0-SNAPSHOT"
          ]
        },
        {
          "severity":"MINOR",
          "description":"non-default import mode MERGE defined for filterSet with root /apps/rep:policy",
          "packages":[
            "adobe/cq60:core.wcm.components.extensions.amp.content:2.11.0"
          ]
        }
      ]
    },
    {
      "checkName":"net.adamcin.oakpal.core/basic/overlaps"
    },
    {
      "checkName":"net.adamcin.oakpal.core/basic/composite-store-alignment"
    },
    {
      "checkName":"net.adamcin.oakpal.core/basic/sling-jcr-installer"
    }
  ]
}
```

Notice the presence of the sling-jcr-installer check towards the end of the report.

```json
    {
      "checkName":"net.adamcin.oakpal.core/basic/sling-jcr-installer"
    }
```

So, how should we resolve the `basic/acHandling` violations? 

There are three different approaches to resolving an OakPAL violation.

1. *Change the content package in some way to satisfy the check's acceptance criteria*: This is usually the best approach, 
assuming the check's criteria is designed with content safety in mind, especially if ample consideration has already 
been given to the proper checklist configuration for the project. But if you are introducing the check to your project 
for the first time, you may want to evaluate whether the check's default assumptions are appropriate for your particular 
project's requirements before refactoring your content packages to satisfy the check. This analysis may convince you to 
choose one of the other two approaches.

2. *Configure the check so that its acceptance criteria is more closely tailored to your project requirements*: The 
checks applied by the `basic` checklist support many configuration parameters, like changing the reported severity, or
specifying the check's scope using `include`/`exclude` rules with regular expressions matched against content paths. If
the default configuration doesn't closely describe your project's content package design, it may be possible to narrowly 
tailor the check to fit a variety of different situations in your project. If you determine that it is not possible to 
reconfigure the check so that it still provides some value in your build, you can skip the check altogether.

3. *Skip the check altogether*: This is usually the easiest approach. If it's not worth keeping in the build, just skip 
the check. Any check applied by a checklist can be skipped, independent of the check's own documented configuration 
parameters.

### Configuring or Skipping Checklist Checks

At this point, let's assume that the content packages triggering the violations are structured exactly as intended, that
the project team is well aware of all the effects of installing these packages in their current state, and that it is
the checks' default acceptance criteria that is misaligned to the project requirements. We will start with the easiest 
approach, which is to skip the `basic/acHandling` check.

Add the `basic/acHandling` check with the `<skip>true</skip>` setting to the plugin configuration in 
`classic-app/all/pom.xml`, as shown below:

```xml
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <configuration>
        <checklists>
            <checklist>basic</checklist>
        </checklists>
        <checks>
            <check>
                <name>basic/acHandling</name>
                <skip>true</skip>
            </check>
        </checks>
    </configuration>
</plugin>
```

Now run `mvn clean install`. As expected, the build succeeds, and the oakpal plugin generates the following report:

```
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] OakPAL Check Reports
[INFO]   net.adamcin.oakpal.core/basic/filterSets
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /conf/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/dam/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/experience-fragments/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /apps/rep:policy [core.wcm.components.extensions.amp.content-2.11.0.zip]
```

The `basic/filterSets` violations are still reported, but since they are `MINOR`, the build succeeds because the 
`basic/acHandling` violations are no longer being reported.

Of course, the `basic/acHandling` check is designed to prevent a package from being installed to a persistent Oak 
repository that will unintentionally (or maliciously) overwrite or clear the ACLs defined by other application 
deployments or those manually created by users after the application's initial launch. 

In fact, the `basic/acHandling` check spec configuration overrides the default configuration of the 
`net.adamcin.oakpal.core.checks.AcHandling` check's Java implementation (`levelSet:no_unsafe`) to only allow 
additive ACEs (`levelSet:only_add`), but this arguably sacrifices determinism in favor of an extra layer of safety.

Instead of skipping the check, let's change the `levelSet` config property back to the implementation default value of
`no_unsafe`, to see if this is sufficient for our project requirements, so that we can still use the `basic/acHandling` 
check to prevent any packages in our project from say, setting `acHandling` to `clear` and deleting all the ACLs in the 
repository upon installation.

```xml
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <configuration>
        <checklists>
            <checklist>basic</checklist>
        </checklists>
        <checks>
            <check>
                <name>basic/acHandling</name>
                <config>
                    <levelSet>no_unsafe</levelSet>
                </config>
            </check>
        </checks>
    </configuration>
</plugin>
```

Once again, the `mvn clean install` execution succeeds with same report from the `oakpal-maven-plugin`, except this time
we know that the `basic/acHandling` check was active and satisfied:

```
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] OakPAL Check Reports
[INFO]   net.adamcin.oakpal.core/basic/filterSets
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /conf/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/dam/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/experience-fragments/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /apps/rep:policy [core.wcm.components.extensions.amp.content-2.11.0.zip]
```

### Checklist Checks as Templates

Some checklists, including the `basic` checklist, define some checks that are skipped-by-default. Once a check is skipped, 
it cannot be unskipped by overriding the spec by name as you would if you were overriding the config. To make use of a 
check that has been skipped-by-default, you must reference the check by name in the `<template>` parameter, and also 
provide your own `<name>` for your instance of the check.

For example, to use the `basic/echo` check, you have to add something like this to the plugin configuration:

```xml
<check>
    <name>my-echo</name>
    <template>basic/echo</template>
</check>
```

There are a variety of reasons for a checklist to force users to specify a check like this. For one, some checks don't 
have an obvious universal default configuration. 

For example, applying the `basic/expectPaths` check only makes sense when you have a list of required content paths 
specific to the contents of a particular package. 

Another reason for skipped-by-default is demonstrated by `basic/jcrProperties`, which, like `basic/expectPaths`, lacks 
a universal default configuration, but which is also designed to be declared perhaps many times for a single scan, 
defining a separate instance of the check for each scope of property evaluation. 

Finally, another reason for skipped-by-default is when providing checks that are useful for debugging, 
like `basic/echo`, which is very verbose and yet enforces no acceptance criteria. 

Ultimately, the minimal justification for including such definitions in a checklist at all is to provide a stable layer 
of abstraction by way of the check name, so that consumers of a checklist aren't tightly coupled to specific 
implementation details, like a java class name or a script path, identifiers which might be subject to refactoring 
without warning at some point in the future.

### Integrating 3rd-Party Checklists

Moving beyond the `basic` checklist, you may want to bring in 3rd-Party checklists published by other projects, such as
those published with ACS AEM Commons.

To end this exercise, you will add the `content-class-aem65` checklist to the scan, which checks for violations of 
the Granite content classification mixin types, `granite:PublicArea`, `granite:InternalArea`, `granite:FinalArea`, and
`granite:AbstractArea`, which are applied to proprietary content paths in AEM to mark boundaries for reuse by custom 
code vis-á-vis Sling content-as-API. 

[View the `content-class-aem65` checklist](https://github.com/Adobe-Consulting-Services/acs-aem-commons/blob/acs-aem-commons-4.8.4/oakpal-checks/src/main/resources/OAKPAL-INF/checklist/content-class-aem65.json)

The `content-class-aem65` checklist has an interesting design. It consists of only one check (`content-classifications`), 
plus a massive set of initial state declarations that install the necessary Granite namespaces and nodetypes, and 
that then create nodes at all the classified locations in AEM 6.5 with the correct mixins. 

The check is actually looking at every node imported by your package that has a `sling:resourceType` or a 
`sling:resourceSuperType` property, or that might overlay a `/libs` node with a node with the same relative path under 
`/apps`, and using the JCR Java API to check if an actual JCR node exists that classifies the imported node as a 
violation of inheritance, overlay, or usage. 

ACS AEM Commons also publishes a `content-class-aem64` checklist that uses the exact same check implementation, but 
imports a different set of root paths, to match the paths that exist in AEM 6.4. 

This demonstrates the value of publishing multiple checklists in a single OakPAL module. 
It allows you to mix and match a shared library of check implementations with different repository state initialization 
rules to cover a broad list of potentially valid, but mutually exclusive scenarios.

The checklist in the current ACS AEM Commons version was generated for AEM 6.5.0.0,
so it's a little out of date, but it should serve to adequately demonstrate the last objective of this exercise.

Let's begin.

Because the checklist we want to use is published in a 3rd-party jar, we need to make that jar available to the 
`oakpal-maven-plugin` as a dependency, just like we did in Exercise 1 when we added the AEM 6.5.5.0 CND. Update the
plugin element in `classic-app/pom.xml`, like so:

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
        <dependency>
            <groupId>com.adobe.acs</groupId>
            <artifactId>acs-aem-commons-oakpal-checks</artifactId>
            <version>4.8.4</version>
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

Then, update the plugin element in `classic-app/all/pom.xml` to add the appropriate checklist after `basic`:

```xml
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <configuration>
        <checklists>
            <checklist>basic</checklist>
            <checklist>content-class-aem65</checklist>
        </checklists>
        <checks>
            <check>
                <name>basic/acHandling</name>
                <config>
                    <levelSet>no_unsafe</levelSet>
                </config>
            </check>
        </checks>
    </configuration>
</plugin>
```

Now, run `mvn clean install`. The build should fail with new violations. 

```
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] OakPAL Check Reports
[INFO]   net.adamcin.oakpal.core/basic/filterSets
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /conf/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/dam/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/experience-fragments/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /apps/rep:policy [core.wcm.components.extensions.amp.content-2.11.0.zip]
[INFO]   com.adobe.acs.acs-aem-commons-oakpal-checks/content-class-aem65/content-classifications
[ERROR]    +- <MAJOR> /conf/core-components-examples/settings/dam/cfm/models/office/jcr:content [restricted resource type]: /libs/dam/cfm/models is marked INTERNAL [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /conf/core-components-examples/settings/dam/cfm/models/office/jcr:content [restricted super type]: /libs/dam/cfm/models is marked INTERNAL [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /conf/core-components-examples/settings/dam/cfm/models/office/jcr:content/model/cq:dialog/content/items/1556183201062 [restricted resource type]: /libs/dam/cfm/admin/components/authoring is marked INTERNAL [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR] ** Violations were reported at or above severity: MAJOR **
```

Once again, for the purpose of this exercise let's assume that our content package is already correct, and that we should resolve these new violations by
either configuring the check or skipping it altogether.

Let's first attempt to skip the check.

```xml
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <configuration>
        <checklists>
            <checklist>basic</checklist>
            <checklist>content-class-aem65</checklist>
        </checklists>
        <checks>
            <check>
                <name>basic/acHandling</name>
                <config>
                    <levelSet>no_unsafe</levelSet>
                </config>
            </check>
            <check>
                <name>content-class-aem65/content-classifications</name>
                <skip>true</skip>
            </check>
        </checks>
    </configuration>
</plugin>
```

Notice that I've followed the same naming convention of `<checklist>/<checkName>`, but you can also specify the `name`
as just `content-classifications`. 

Re-run `mvn clean install`. It should now succeed. 

```
[INFO] Check report summary written to /Users/madamcin/oss/oakpal-lab-adaptto-2020/classic-app/all/target/oakpal-plugin/reports/oakpal-summary.json
[INFO] OakPAL Check Reports
[INFO]   net.adamcin.oakpal.core/basic/filterSets
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /conf/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/dam/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /content/experience-fragments/classic-app [classic-app.ui.content-1.0-SNAPSHOT.zip]
[INFO]     +- <MINOR> non-default import mode MERGE defined for filterSet with root /apps/rep:policy [core.wcm.components.extensions.amp.content-2.11.0.zip]
```

Let's now reconfigure the check instead. First, let's revisit the original violations:

```
[INFO]   com.adobe.acs.acs-aem-commons-oakpal-checks/content-class-aem65/content-classifications
[ERROR]    +- <MAJOR> /conf/core-components-examples/settings/dam/cfm/models/office/jcr:content [restricted resource type]: /libs/dam/cfm/models is marked INTERNAL [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /conf/core-components-examples/settings/dam/cfm/models/office/jcr:content [restricted super type]: /libs/dam/cfm/models is marked INTERNAL [core.wcm.components.examples.ui.content-2.11.0.zip]
[ERROR]    +- <MAJOR> /conf/core-components-examples/settings/dam/cfm/models/office/jcr:content/model/cq:dialog/content/items/1556183201062 [restricted resource type]: /libs/dam/cfm/admin/components/authoring is marked INTERNAL [core.wcm.components.examples.ui.content-2.11.0.zip]
```

As you can see in the linked checklist file above, the check config is defined to exclude paths matching the pattern,
`/etc/dam/video(/.*)?` from the scope of validation. 

```json
  "checks":[
    {
      "name":"content-classifications",
      "impl":"com.adobe.acs.commons.oakpal.checks.ContentClassifications",
      "config":{
        "scopePaths":[
          {
            "comment":"Known usages of internal resource types.",
            "type":"exclude",
            "pattern":"/etc/dam/video(/.*)?"
          },
          {
            "comment":"Known extension of final supertype.",
            "type":"exclude",
            "pattern":"/apps/acs-commons/components/authoring/graphiciconselect"
          },
          {
            "comment":"Known extension of final supertype.",
            "type":"exclude",
            "pattern":"/apps/acs-commons/touchui-widgets/contextualpathbrowser"
          }
        ]
      }
    }
  ]
```

In our situation, the violations are reporting paths under `/conf/core-components-examples/settings/dam/cfm/models` for
a similar reason to what is described in the comment, that is -- using resource types that are marked INTERNAL. 

Keep in mind that when we override the `scopePaths` config property, because it is an array, the nested values will not 
be merged, only replaced. This means that if we still need the `/etc/dam/video` exclude, we will need to repeat it in 
our plugin config.

```xml
<check>
    <name>content-classifications</name>
    <config>
        <scopePaths>
            <scopePath>
                <comment>Known usages of internal resource types.</comment>
                <type>exclude</type>
                <pattern>/conf/core-components-examples/settings/dam/cfm/models(/.*)?</pattern>
            </scopePath>
        </scopePaths>
    </config>
</check>
```

When you re-run the build with this configuration, the build should succeed.

You might be slightly confused by how this XML structure maps to the checklist JSON configuration. In the OakPAL API,
all configurations are converted to JSON before being provided to check implementations. The `oakpal-maven-plugin` comes
with a comprehensive plugin for converting XML structures to JSON, that also follows traditional Maven conventions for
defining lists and maps. That is the short explanation for why you have a `<scopePaths>` parent with a `<scopePath>` 
child mapped to a JSON array for the `scopePaths` key, whereas the value of the `<scopePath>` element, in turn, 
is interpreted as an object, not an array, and the `scopePath` key itself does not appear in the resulting JSON.

When in doubt, you can always provide a conversion hint for the plugin to override the simple XML-to-JSON heuristics.

For example, an equivalent structure using only inline JSON looks like this:

```xml
<check>
    <name>content-classifications</name>
    <config>
        <scopePaths hint="JsonArray">
            [{
                "comment": "Known usages of internal resource types.",
                "type": "exclude",
                "pattern": "/conf/core-components-examples/settings/dam/cfm/models(/.*)?"
            }]
        </scopePaths>
    </config>
</check>
```

For more details on the XML to JSON conversion, read the [source of the JsonConverter](https://github.com/adamcin/oakpal/blob/v2.2.1/maven/src/main/java/net/adamcin/oakpal/maven/component/JsonConverter.java).

At this point you may remove the `content-class-aem65` checklist and the `content-classifications` check from 
`classic-app/all/pom.xml`, and the `acs-aem-commons-oakpal-checks` dependency from `classic-app/pom.xml`. 

In the next exercise, you will finally experience the fun of writing your own checks!

[Continue to Exercise 3: Writing your own checks](Exercise_03.md)
