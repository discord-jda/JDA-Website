When you setup a JDA project you should also setup a logging implementation for SLF4J. This is only necessary if you get a warning like this on startup:

```none
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

I recommend logback-classic as it is my goto implementation. First add logback to your dependencies:

=== "Gradle"

    ```groovy
    dependencies {
        implementation("ch.qos.logback:logback-classic:1.2.8")
    }
    ```

=== "Maven"

    ```xml
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.8</version>
    </dependency>
    ```


## Configure Logback

The logback configuration needs to be in your resources directory. This is `src/main/resources` in all standard Gradle and Maven projects.
Add the following configuration into `src/main/resources/logback.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} %boldCyan(%-34.-34thread) %red(%10.10X{jda.shard}) %boldGreen(%-15.-15logger{0}) %highlight(%-6level) %msg%n</pattern>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

You can read up on how this configuration works in the [logback manual](https://logback.qos.ch/manual/configuration.html).

### How to enable debug logs

In the config above, change the level for the root logger to debug:

```xml
<root level="debug">
  ...
</root>
```

### Supported MDC Options

There are a few built-in [MDC](https://www.slf4j.org/api/org/slf4j/MDC.html) options used by JDA:

- `jda.shard` The shard id and shard total in the format `[id / total]`. (This is 0-based so with a total of 10 the last shard id `[9 / 10]`)
- `jda.shard.id` The shard id
- `jda.shard.total` The shard total

You can further configure other MDC variables for JDA threads with [JDABuilder.setContextMap](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/JDABuilder.html#setContextMap(java.util.concurrent.ConcurrentMap))!
