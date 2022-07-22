# JDA (Java Discord API)

This page provides a fast introduction to JDA.  For a more comprehensive guide, please view [the setup guide](../using-jda/getting-started.md). 

## Download
Whilst downloads are available on the [Jenkins Server](https://ci.dv8tion.net/job/JDA5/) and the [JDA GitHub releases](https://github.com/DV8FromTheWorld/JDA/releases) pages, 
it is highly recommended to use a build tool like Gradle or Maven to manage JDA and its dependencies.

!!! Information ""
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
    Remember to replace the `VERSION` with the version you would like to use.  Version information can be found from 
    [JDA Discord Server](https://discord.gg/0hMr4ce0tIk3pSjp), [JDA GitHub](https://github.com/DV8FromTheWorld/JDA/releases) or 
    [Maven Central](https://mvnrepository.com/artifact/net.dv8tion/JDA/).
    
    [![Download](https://img.shields.io/maven-central/v/net.dv8tion/JDA?color=blue)](https://ci.dv8tion.net/job/JDA5/lastSuccessfulBuild/)


## Minimal Examples
!!! example "Simple Ready Listener Example"
    ```java
    public class ReadyListener implements ListenerAdapter
    {
        public static void main(String[] args)
        throws LoginException
        {
            JDA jda = JDABuilder.createDefault(BOT_TOKEN)
                .addEventListeners(new ReadyListener()).build();
        }

        @Override
        public void onReady(ReadyEvent event)
        {
            System.out.println("JDA has started!");
        }
    }
    ```

!!! example "Simple Message Logging Example"
    ```java
    public class MessageListener extends ListenerAdapter
    {
        public static void main(String[] args)
        throws LoginException
        {
            JDA jda = JDABuilder.createDefault(BOT_TOKEN)
                                .enableIntents(GatewayIntent.MESSAGE_CONTENT)
                                .build();
            jda.addEventListeners(new MessageListener());
        }

        @Override
        public void onMessageReceived(MessageReceivedEvent event)
        {
            if (event.isFromType(ChannelType.TEXT))
            {
                System.out.printf("[%s][%s] %#s: %s%n", event.getGuild().getName(),
                    event.getChannel().getName(), event.getAuthor(), event.getMessage().getContentDisplay());
            }
            else
            {
                System.out.printf("[PM] %#s: %s%n", event.getAuthor(), event.getMessage().getContentDisplay());
            }
        }
    }
    ```
!!! information "More Examples"
    We provide a small set of Examples in the [Example Directory](https://github.com/DV8FromTheWorld/JDA/tree/master/src/examples/java).


## Docs
You can find the [latest Javadocs](https://ci.dv8tion.net/job/JDA5/javadoc/) and the [legacy Javadocs](https://ci.dv8tion.net/job/JDA/javadoc/) on the Jenkins server.

For other versions of JDA, the javadocs jar can be downloaded from the version's page.

For example, the v4.2.0_200 javadocs can be found at:
<https://ci.dv8tion.net/job/JDA/200/>, which can be extracted for its docs.

## Getting Help
If you need help, or just want to talk with the JDA or other Devs, you can join the [Official JDA Discord Guild](https://discord.gg/0hMr4ce0tIl3SLv5).

Alternatively, you can visit the `#java_jda` channel in the [Unofficial Discord API Guild](https://discord.gg/discord-api).
The dedicated JDA guild is generally more active, and will often help you faster.
<br>
For guides and setup help, this wiki aims to provide information.

If you're looking for guides or setup help, check out our [Getting Started](../using-jda/getting-started.md) pages.

## Contributing to JDA
If you want to contribute to JDA, make sure to base your branch off of our **master** branch (or a feature-branch)
and create your PR into that **same** branch.

It's recommended to ask (preferably in the JDA Guild) about a PR before starting one, to ensure that it is a welcome change and that it isn't already being worked on.

More information can be found on our [Contributing](../contributing/contributing.md) page.

## Dependencies
This project requires **Java 8**.<br>
For other dependencies, see [README](https://github.com/DV8FromTheWorld/JDA5/tree/master/README.md)