---
template: overrides/home.html
---

# Installation
![latest](https://img.shields.io/maven-central/v/net.dv8tion/JDA?color=blue){ draggable="false" }
![snapshot](https://img.shields.io/jitpack/v/github/DV8FromTheWorld/JDA?label=Snapshots&color=blue){ draggable="false" }

JDA can be downloaded through MavenCentral using the dependency manager of your choosing.
<br>Please make sure to replace `VERSION` with one of the above listed ones.

!!! jda ""
    === "Maven"
        ```xml
        <dependencies>
            <groupId>net.dv8tion</groupId>
            <artifactId>JDA</artifactId>
            <version>VERSION</version>
        </dependencies>
        ```
    
    === "Gradle"
        ```groovy
        repositories {
            mavenCentral()
        }
        
        dependencies {
            // Change 'implementation' to 'compile' in older Gradle versions
            implementation("net.dv8tion:JDA:VERSION")
        }    
        ```