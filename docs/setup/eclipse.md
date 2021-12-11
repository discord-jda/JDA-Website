## How to start developing with Eclipse

[ ![Download](https://shields.io/maven-metadata/v?metadataUrl=https%3A%2F%2Fm2.dv8tion.net%2Freleases%2Fnet%2Fdv8tion%2FJDA%2Fmaven-metadata.xml&color=informational&label=Download) ](https://ci.dv8tion.net/job/JDA/lastSuccessfulBuild/)

To start developing with Eclipse, follow one of the guides below:

- [Gradle Setup](#gradle-setup)
- [Maven Setup](#maven-setup)
- [Jar Setup](#jar-setup)

***
#### Gradle Setup

1. If you have *Eclipse IDE for Java Developers* installed, skip to **2.**, otherwise you need to install the *Buildship Gradle Integration plugin* first:
  - Open up Eclipse and go to the Marketplace (located under the *Help* tab)
  - Search for *"gradle"* and install ***Buildship Gradle Integration*** ([Plugin-Page](http://marketplace.eclipse.org/content/buildship-gradle-integration))
  - After the plugin is installed, relaunch Eclipse
2. Right click within *Package/Project Explorer* and select **New > Other...**

    ![New Gradle Project](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/01-newProject.png)

3. In the *Gradle* folder, select **Gradle Project**

    ![Gradle Project](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/02-gradleProject.png)

4. Type a name for your Project and click on *Finish*. Your setup should look like this at this point:

    ![Folder Structure](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/03-projectOverview.png)

6. Delete the classes within `src/main/java` and `src/test/java`

    ![Files To Delete](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/04-deleteFiles.png)

7. Open up and edit the file `build.gradle`

    ![Build_Gradle Location](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/05-gradleBuildFile.png)

8. Replace its content with the following code:

    ```groovy
    plugins {
        id("java")
        id("application")
        id("com.github.johnrengelman.shadow") version "6.0.0"
    }

    mainClassName = "com.example.jda.Bot"

    version '1.0'

    sourceCompatibility = 1.8

    repositories {
        mavenCentral()
        maven { // on kotlin dsl use `maven("https://m2.dv8tion.net/releases")` instead
            url "https://m2.dv8tion.net/releases"
        }
    }

    dependencies {
        implementation("net.dv8tion:JDA:#.#.#_###")
    }

    compileJava.options.encoding = "UTF-8"
    ```

9. Adjust the version of JDA you want to use (see dependencies-section of file) and fill in your Main-Class as soon as you have one (the one containing your `public static void main(String[] args)` method)
10. Save the file and do the following: *Right click your project > Gradle > Refresh All*

    ![Refresh Gradle Project](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/06-gradleRefresh.png)

11. Once all the dependencies have been downloaded, create your desired packages/classes in `src/main/java` and start coding!
12. To build your project you can run `gradlew shadowJar` in a terminal of your project root, and it will produce a jar filled with your compiled code and JDA included in a single jar file! The jar can be found in `build/libs`
13. [Setup Logback](logging.md)

***
#### Maven Setup

Prerequisites: Maven-Plugin and local Maven installation

1. Create a new Maven project. (File -> New -> Other -> Maven -> Maven Project)

    ![New Maven Project](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/10-newProject.png)

2. Check the `Create a simple project` box on the next page as we don't need to worry about archetypes.

    ![Check Simple Project](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/11-checkSimpleProject.png)

3. Add a groupId, artifactId and a name. Make sure you try to follow the [naming conventions](https://maven.apache.org/guides/mini/guide-naming-conventions.html) while you are at this step. The result could look like the image below.

    ![Project Values](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/12-mvnValues.png)

4. Now let's start configuring it, first off, open up your pom.xml and add the following lines right after `</description>`
  ```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>
  ```
  - This will make your project support UTF-8 characters (So you can have it on Japanese servers for example) and also force Java 8, which is needed.

5. Now let's add JDA's repository, so we can fetch the jar (Place this after `</properties>`)
  ```xml
  <repositories>
    <repository>
        <id>dv8tion</id>
        <name>m2-dv8tion</name>
        <url>https://m2.dv8tion.net/releases</url>
    </repository>
  </repositories>
  ```

6. Now, add the dependency, make sure you change `X.X.X_XXX` to the latest version number (Check out at https://ci.dv8tion.net/job/JDA/)
    ```xml
    <dependencies>
        <dependency>
        <groupId>net.dv8tion</groupId>
        <artifactId>JDA</artifactId>
        <version>X.X.X_XXX</version>
        </dependency>
    </dependencies>
    ```

7. Now you need to set up the (build) maven-shade and maven-compile plugins, add the following lines right after `</dependencies>`
  - **NOTE**: The following changes will force the compiler to use Java 8 (JDA needs it), so make sure you have it installed.

    ```xml
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.4</version>
                <configuration>
                    <transformers>
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                            <mainClass>YourMainClass</mainClass> <!-- You have to replace this with a path to your main class like me.myname.mybotproject.Main -->
                        </transformer>
                    </transformers>
                    <createDependencyReducedPom>false</createDependencyReducedPom>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    ```

8. After that, the project must be updated to download the dependencies *Right click > Maven > Update Project

    ![Maven Update](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/13-mvnUpdate.png)  

9. You are done! Now you can head to the [Javadocs](https://ci.dv8tion.net/job/JDA/javadoc/) or see examples at the [Examples](https://github.com/DV8FromTheWorld/JDA/tree/master/src/examples/java) page.
10. [Setup Logback](logging.md)

***
#### Jar Setup

1. Download the latest (Binary) version of JDA (with Dependencies):
  - (Recommended) <https://github.com/DV8FromTheWorld/JDA/releases/>
  - (Latest/Dev) <https://ci.dv8tion.net/job/JDA/>
2. Create a new Java Project

    ![New Java Project](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/20-newJavaProject.png)

3. Fill out the bot name, and set it to Java 8 (or above if available). This option might be set automatically when the `Use default location` box is checked.

    ![Java 8](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/21-java8.png)

4. Right click the project, go to **Properties**
5. Click on **Java Build Path**, then click on **Libraries**, then on **Classpath**, **Add External JARs...**

    ![Add External Jars](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/22-addExternalJars.png)

6. Add your downloaded **JDA-withDependencies-x.x.x_xxx.jar** and expand its properties
  - If you don't want Javadoc and source annotations, skip to 11 (not recommended).

    ![Add External Jars](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/23-sourceAttachment.png)

7. Click on **Source Attachment**, then on **Edit...**, then mark **External Locations** and click on **External File**

    ![External File](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/24-externalFile.png)

8. Here, add your **JDA-x.x.x_xxx-sources.jar** and click on **OK**
9. Next, click on **Javadoc Location**, then on **Edit...**, then mark **Javadoc in archive** and click on **Browse**

    ![Javadoc](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/setup/eclipse/25-javaDocs.png)

10. Here, add your **JDA-x.x.x_xxx-javadoc.jar** and click on **OK**
11. [Setup Logback](logging.md)

You Are Done! You can start by taking a look at the examples [Here](https://github.com/DV8FromTheWorld/JDA/tree/master/src/examples/java) or **Reading** the docs [Here](https://ci.dv8tion.net/job/JDA/javadoc/).