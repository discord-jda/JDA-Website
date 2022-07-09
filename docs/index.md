---
template: home.html
---

<!-- Placeholder to make the page render -->

<div class="mdx-grid-container" markdown>
<div class="mdx-grid-wrapper mdx-grid-size" markdown>
<div class="mdx-grid-child" markdown>
## Getting Started

JDA is distributed through MavenCentral, allowing you an easy inclusion into your Java project by the dependency manager of your choice.
</div>
<div class="mdx-grid-child" markdown>
## Configuration

=== "Maven"
    ```xml
    <dependency>
      <groupId>net.dv8tion</groupId>
      <artifactId>JDA</artifactId>
      <version>VERSION</version>
    </dependency>
    ```

=== "Maven (No Audio)"
    ```xml
    <dependency>
      <groupId>net.dv8tion</groupId>
      <artifactId>JDA</artifactId>
      <version>VERSION</version>
      <exclusions>
        <exclusion>
          <groupId>club.minnced</groupId>
          <artifactId>opus-java</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    ```

=== "Gradle"
    ```groovy
    repositories {
      mavenCentral()
    }
    
    dependencies {
      implementation("net.dv8tion:JDA:VERSION")
    }
    ```

=== "Gradle (No Audio)"
    ```groovy
    repositories {
      mavenCentral()
    }
    
    dependencies {
      implementation("net.dv8tion:JDA:VERSION") {
        exclude module: 'opus-java'
      }
    }
    ```

</div>
</div>
</div>
