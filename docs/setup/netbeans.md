## How to start developing with Netbeans

[ ![Download](https://shields.io/maven-metadata/v?metadataUrl=https%3A%2F%2Fm2.dv8tion.net%2Freleases%2Fnet%2Fdv8tion%2FJDA%2Fmaven-metadata.xml&color=informational&label=Download) ](https://ci.dv8tion.net/job/JDA/lastSuccessfulBuild/)

To start developing with Netbeans, follow one of the guides below:
* [Maven Setup](#maven-setup)
* [Jar Setup](#jar-setup)


### Maven Setup

1. **Make a new Maven Java Application**
    
    ![MavenApp](https://i.imgur.com/smGUSi6.png)
2. **Open up the pom.xml in the Project Files**
    
    ![Pom](https://i.imgur.com/f58Dbjy.png)
3. **Add S3 as a repository**
    ```xml
    <repositories>
        <repository>
            <id>dv8tion</id>
            <name>m2-dv8tion</name>
            <url>https://m2.dv8tion.net/releases</url>
        </repository>
    </repositories>
    ```

4. **Add JDA as a dependency**
    ```xml
    <dependencies>
        <dependency>
            <groupId>net.dv8tion</groupId>
            <artifactId>JDA</artifactId>
            <version>3.8.3_464</version>
        </dependency>
    </dependencies>
    ```
>Note: These can go anywhere within the `<project></project>` tags.

5. [Setup Logback](logging.md)

6. **Start developing!**

### Jar Setup

1. **Download the latest (binary) version of JDA (with dependencies), as well as the javadocs**<br>
    https://ci.dv8tion.net/job/JDA/
    <br>![Downloads](http://i.imgur.com/fNN4vOf.png)
2. **Make a new Java Application**
    
    ![JavaApp](http://i.imgur.com/9mOkwmA.png)
3. **Right-click the `Libraries` folder in your project, and select `Add JAR/Folder...`**
    
    ![Jar](http://i.imgur.com/5CZIJYF.png)<br>
4. **Find the `JDA...withDependencies.jar` and add it.**
5. **Right-click on the newly-added Jar file, and select `Edit...`**
    
    ![Edit](http://i.imgur.com/Jvt8574.png)
6. **Select `Browse...` and add the javadoc jar**
    
    ![Browse](http://i.imgur.com/bm51esA.png)
7. [Setup Logback](logging.md)
8. **Start developing!**