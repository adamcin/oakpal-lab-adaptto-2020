# oakpal-lab-adaptto-2020
Exercises for the OakPAL Lab at adaptTo 2020 conf

## Exercises

* [Exercise 1: First Steps with OakPAL](Exercise_01.md)

* [Exercise 2: Integrating 3rd-Party Checklists](Exercise_02.md)

* [Exercise 3: Writing your own checks](Exercise_03.md)

## Notes on the Construction of this repository

1. ran the following command within the root directory to create the `classic-app` folder:

    ```bash
    mvn -Padobe-public -U org.apache.maven.plugins:maven-archetype-plugin:LATEST:generate \
        -DarchetypeGroupId=com.adobe.aem  \
        -DarchetypeArtifactId=aem-project-archetype  \
        -DarchetypeVersion=24 \
        -DgroupId=org.adaptto.oakpal \
        -DartifactId=classic-app \
        -DappId=classic-app \
        -DaemVersion=6.5.5 \
        -DappTitle="AEM Classic App" \
        -DincludeDispatcherConfig=n \
        -Dversion=1.0-SNAPSHOT
    ```

1. created the root pom.xml to serve as a reactor for importing all subprojects into an IDE
