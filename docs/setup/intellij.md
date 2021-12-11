# IntelliJ IDEA Setup

[ ![Download](https://shields.io/maven-metadata/v?metadataUrl=https%3A%2F%2Fm2.dv8tion.net%2Freleases%2Fnet%2Fdv8tion%2FJDA%2Fmaven-metadata.xml&color=informational&label=Download&style=for-the-badge) ](https://ci.dv8tion.net/job/JDA/lastSuccessfulBuild/)

## Older versions of IntelliJ IDEA

??? info "This describes the changes for older versions of IntelliJ IDEA"

    1. Open the Project view
    2. Create a new Project

        ![project](https://i.imgur.com/webTCoy.png)

    3. Select `Gradle` > `Java`
    4. Configure your SDK to use Java 1.8

        ![gradle](https://i.imgur.com/qpdkFph.png)

    5. Click `Next` and fill in your groupId and your artifactId. Example: `me.name` and `bot`

        ![artifact](https://i.imgur.com/wK89v0C.png)

    6. Check `Use auto-import` and click `Next` > `Finish`

        ![settings](https://i.imgur.com/ANSIjtw.png)

    7. Continue with step 5 of the tutorial for newer IntelliJ IDEA versions


## For newer versions of IntelliJ IDEA

1. Navigate to "New Project" from any view
2. Select Gradle -> Java as the type of Project and make sure the correct JDK is selected (Java8 or higher)

    ![new_project](https://i.imgur.com/cEOAzz2.png)

3. Provide a title for your project and define your GroupId and optionally the ArtifactId and initial Version in the "Artifact Coordinates" subsection

    ![artifact_setup](https://i.imgur.com/Vn1Ocm1.png)

4. Optionally enable Auto-Importing of the gradle file in the Gradle Settings

    ![gradle_settings](https://i.imgur.com/a99Nj1G.png)
    ![auto_import](https://i.imgur.com/sT6sZof.png)

> Note: this is also the place where you could switch the runner for your project (By default, Gradle is used to run your application and tests)

5. Let intellij index your project.
6. Open `build.gradle`
7. Populate the build file with the following
    ```groovy
    plugins {
        id'application'
        id'com.github.johnrengelman.shadow' version '5.2.0'
    }
    
    mainClassName = 'com.example.jda.Bot'
    
    version '1.0'
    def jdaVersion = 'JDA_VERSION_HERE'
    
    sourceCompatibility = targetCompatibility = 1.8
    
    repositories {
        mavenCentral()
        maven { // on kotlin dsl use `maven("https://m2.dv8tion.net/releases")` instead
            url "https://m2.dv8tion.net/releases"
        }
    }
    
    dependencies {
        implementation("net.dv8tion:JDA:$jdaVersion")
    }
    
    compileJava.options.encoding = 'UTF-8'
    ```
> Note: Replace the `JDA_VERSION_HERE` with the one mentioned [here (release)](https://github.com/DV8FromTheWorld/JDA/releases/latest) or with the latest build [here](https://ci.dv8tion.net/job/JDA/lastSuccessfulBuild/)<br>
> Replace the `mainClassName` value with the path to your main class later on! 

8. If IntelliJ IDEA didn't already do so automatically, set up a source folder as `src/main/java`
9. Create your group package. Example: `me.name.bot`
10. Make your main class. Example: `Bot.java`.
    Your directory tree should look like this:
    ```
    ProjectName -> src/main/java -> me/name/bot -> Bot.java
                -> gradle/wrapper -> gradle-wrapper.properties
                -> gradle/wrapper -> gradle-wrapper.jar
                -> build.gradle
                -> settings.gradle
    ```
11. Configure the `mainClassName` value in the `build.gradle` to your class. Example: `me.name.bot.Bot`
12. To build your finished project simply use the `shadowJar` task in your gradle tool window on right hand side of your editor. This will build a jar in `build/libs`. The one with the `-all` suffix is the shadow jar.
    > You can also run your project with the `run` gradle task!
13. [Setup Logback](logging)
14. Continue with [Getting Started](../using-jda/getting-started.md)