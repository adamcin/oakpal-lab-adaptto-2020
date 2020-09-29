# Exercise 4: On Your Own

## Objectives

* Implement your own checks under `classic-app/all/src/test/java` or `classic-app/all/src/test/resources`

* Validate your check logic against the output of the `adamcin/oakpal-exercise-solutions` docker image.

```bash
docker run -it --rm -v $(pwd):/work \
  adamcin/oakpal-exercise-solutions:latest \
  classic-app.all-1.0-SNAPSHOT.zip
```

## Exercise

Before your begin, replace your `oakpal-maven-plugin` configuration in the `classic-app/all/pom.xml` file with just the
`sling-jcr-installer` check:

```xml
<plugin>
    <groupId>net.adamcin.oakpal</groupId>
    <artifactId>oakpal-maven-plugin</artifactId>
    <configuration>
        <checks>
            <check>
                <name>sling-jcr-installer</name>
            </check>
        </checks>
    </configuration>
</plugin>
```

### Requirement 1: Component Groups

Our AEM project delivers quite a number of custom components for authors to use, and it's important that developers pay
attention to the `componentGroup` property that is specified on each `cq:Component` node so that authors don't have to
hunt to find the right component to add to their page, and so that template policies that define Allowed Components for
containers remain stable as we create new features.

Authors expect to see the following component groups available in the editor:

* `AEM Classic App - Structure`: Components which should only be added to structural regions of a page template, like 
search and navigation.

* `AEM Classic App - Content`: Components which are generally useful anywhere on a page, like text, title, and image 
components.

* `AEM Classic App - Form`: Components which are used to assemble forms on specific templates, like a contact us page. 
The components that belong in this group are maintained under the `/apps/classic-app/components/form` path.

Any custom components we define that should not be visible in any of these groups should be made invisible to authors
when they are browsing for components, by assigning them to the special `.hidden` component group.

Write a progress check that enforces our desired component group model. Report any of the following as MAJOR violations:

* Any custom `cq:Component` node that does not define a `componentGroup` property.

* Any custom `cq:Component` node under `/apps/classic-app/components/form` whose `componentGroup` property is neither 
`.hidden` or `AEM Classic App - Form`.

* Any custom `cq:Component` node under `/apps/classic-app/components` whose `componentGroup` property is not one of:

  * `.hidden`
  
  * `AEM Classic App - Structure`
  
  * `AEM Classic App - Content`
  
  * `AEM Classic App - Form`

### Requirement 2: Clientlibs

Our team is in the midst of migrating from AEM 6.2 to AEM as a Cloud Service and have already moved our client libraries 
from `/etc/clientlibs` to `/apps/classic-app/clientlibs`. However, we still seem to be seeing some issues where 
clientlib includes are returning 404s to the browsers. Our suspicion is that some `cq:ClientLibraryFolder` nodes are 
missing the `allowProxy=true` property.

At the same time, the Frontend Devs on the team recently changed their `webpack` build to compile to ES6, which was 
discovered to break the Cloud Service build in some situations when the GCC JS minifier was configured for the 
associated `cq:ClientLibraryFolder`. Instead of reverting the webpack changes, we've decided to adjust the `jsProcessor` 
property in all client libraries with a `js.txt` file to either disable the GCC minifier or set the `languageIn` 
attribute to `ECMASCRIPT6`.

Write a progress check to guide our continuing migration and prevent future regressions. Report any of the following as 
MAJOR violations:

* Any `cq:ClientLibraryFolder` under `/apps/classic-app/clientlibs` that doesn't specify a boolean `allowProxy` 
property. Don't report a violation if the property is defined, but has an explicit value of `false`.

* Any `cq:ClientLibraryFolder` under `/apps/classic-app/clientlibs` with a `js.txt` child node that does not specify a
`jsProcessor` multivalued string property with a value of `min:none`, or `min:gcc` with a `;languageIn=ECMASCRIPT6` 
attribute.


### Requirement 3: Template Policies

Our AEM authors have recently been demanding more and more Editable Templates. They really can't get enough of them. 
What's more, they are adamant that developers port the Editable Templates they have created in Production to all the 
lower environments, which are frequently rebuilt, to support their testing activities. Our developers have been working 
frantically to keep up with the furious pace of Template Editing, and have done a remarkably good job keeping the 
contents of the `/conf/classic-app/settings/wcm` path in the `ui.content` package in sync with production. However, the 
process is mostly manual and some mistakes have been made in the templates, template types, and policy mappings. While 
our authoring team is pretty forgiving, and these backported templates don't overwrite the originals in production, 
we should try to make the QA experience as painless as possible.

Write a progress check to help our developers identify where they have made mistakes, before the authors get a chance to
run into them. Report any of the following as MINOR violations:

* Any policy mapping node under `/conf/classic-app/settings/wcm/templates/*/policies` or 
`/conf/classic-app/settings/wcm/template-types/*/policies` that does not specify a `cq:policy` property, or whose `cq:policy` 
property does not resolve to a policy definition under `/conf/classic-app/settings/wcm/policies`.

* Any empty container component node defined the `initial` or `structure` pages of a `template` or `template-type` under 
`/conf/classic-app/settings/wcm` whose mapped policy under `/conf/classic-app/settings/wcm/policies` does not specify a 
`components` property.

## More Practice

After completing the above requirements, you may want to try implementing some of the 
[OakPAL checks described in the Adobe Cloud Manager documentation](https://docs.adobe.com/help/en/experience-manager-cloud-service/implementing/using-cloud-manager/test-results/custom-code-quality-rules.html#oakpal-rules)
that are part of the custom code quality rules that are executed as part of the AEM Cloud Service pipelines, so that 
you can try to catch the same issues before you push your commit to the Adobe Git repo.

* Customer Packages Should Not Create or Modify Nodes Under /libs

* Packages Should Not Contain Duplicate OSGi Configurations

* Config and Install Folders Should Only Contain OSGi Nodes

* Packages Should Not Overlap

* Default Authoring Mode Should Not Be Classic UI

* Components With Dialogs Should Have Touch UI Dialogs

* Packages Should Not Mix Mutable and Immutable Content

* Reverse Replication Agents Should Not Be Used

## Source for Solutions

You can review the implementation of the solutions docker image in the 
[`oakpal-exercise-solutions` github repository](https://github.com/adamcin/oakpal-exercise-solutions).



