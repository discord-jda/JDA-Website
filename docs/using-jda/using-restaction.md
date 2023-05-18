# What is a RestAction?

If you understand RestAction you understand JDA.

In JDA 3.0 we introduced the new `RestAction` class which basically is a terminal between the JDA user and the Discord REST API.
<br>The `RestAction` is a step between specifying what the user wants to do and executing it, it allows the user to specify _how_ JDA should
deal with their `Request`.

However this only works if you actually tell the `RestAction` to do _something_. That is why we recommend checking out whether or not something in JDA returns a `RestAction`. If that is the case you **have** to execute it using one of the `RestAction` execution operations:

[`queue()`](#using-queue), [`queue(Consumer)`](#using-queue), [`queue(Consumer, Consumer)`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/RestAction.html#queue(java.util.function.Consumer,java.util.function.Consumer))
:   These operations are __asynchronous__ and will not execute within the same Thread.
    <br>This means that you cannot use procedural logic when you use `queue()`, unless you use the callback Consumers.
    <br>Only similar requests are internally executed in sequence such as sending messages in the same channel or adding reactions to the same message.

[`submit()`](#using-submit)
:   Provides request future to cancel tasks later and avoid callback hell.

[`complete()`](#using-complete)
:   This operation will __block__ the current Thread until the request has been finished and will return the response type.

!!! note
    We recommend using `queue()` or `submit()` when possible as blocking the current Thread can cause downtime and will use more resources.

Since **4.1.1** you can use a few RestAction operators to avoid callback hell with queue:

- [`map`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/RestAction.html#map%28java.util.function.Function%29)
    Convert the result of the `RestAction` to a different value
- [`flatMap`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/RestAction.html#flatMap%28java.util.function.Function%29)
    Chain another `RestAction` on the result
- [`delay`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/RestAction.html#delay%28java.time.Duration%29)
    Delay the element of the previous step

**JavaDocs**: <https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/RestAction.html>

## AuditLog Reasons

Some operations return a special `RestAction` implementation called `AuditableRestAction`.
<br>This extension allows to set a reason field for that action.

!!! example
    ```java
    public class ModerationUtil
    {
        public static void deleteMessage(Message message, String reason)
        {
            message.delete().reason(reason).queue();
        }

        public static void ban(Guild guild, User user, String reason)
        {
            guild.ban(user, 7, reason).queue();
        }
    }
    ```

## Using `queue()`

The most common way to execute a `RestAction` is by simply calling `.queue()` after the operation: 
```java
public void sendMessage(MessageChannel channel, String message) 
{
    channel.sendMessage(message).queue();
}
```

This will always simply execute the `RestAction<Message>` which was returned by `MessageChannel.sendMessage(String)`.
<br>Note that this might happen after calling `sendMessage(MessageChannel, String)` because `queue()` is __asynchronous__!

> You: Why can't I access the Message that was sent with `queue()`?<br>
> Minn: Use the success callback!

Is one of the common conversations we had when people started using JDA 3.0.

> You: What does that mean?

A success [callback](https://en.wikipedia.org/wiki/Callback_\(computer_programming\)) is what we call the primary [`Consumer`](https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html) that can be passed to a `queue()` statement:
```java
public void sendAndLog(MessageChannel channel, String message) 
{
    channel.sendMessage(message).queue(new Consumer<Message>()
    {
        @Override
        public void accept(Message t)
        {
            System.out.printf("Sent Message %s\n", t);
        }
    });
}
```

Here we used an inline implementation of `Consumer<Message>` that handles the response of a REST Request.
<br>The method `Consumer.accept(Message)` is automatically called once the response has been received by the JDA Requester.

> Minn: But that looks really ugly...<br>
> You: Yeah but it works!!

Since JDA requires you to use Java 1.8 we can use one of the new features: Lambda Expressions
```java
public void sendAndLog(MessageChannel channel, String message)
{
    // Here we use a lambda expressions which names the callback parameter -response- and uses that as a reference
    // in the callback body -System.out.printf("Sent Message %s\n", response)-
    Consumer<Message> callback = (response) -> System.out.printf("Sent Message %s\n", response);
    channel.sendMessage(message).queue(callback); // ^ calls that
}
```

> You: Wow that looks so much better!<br>
> Minn: Yes, please learn more about lambda expressions: [lambda quickstart](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/Lambda-QuickStart/index.html)

!!! example "Example: Sending a Private Message"

    ```java
    public void sendPrivateMessage(User user, String content)
    {
        // openPrivateChannel provides a RestAction<PrivateChannel> 
        // which means it supplies you with the resulting channel
        user.openPrivateChannel().queue((channel) ->
        {
            // value is a parameter for the `accept(T channel)` method of our callback.
            // here we implement the body of that method, which will be called later by JDA automatically.
            channel.sendMessage(content).queue();
            // here we access the enclosing scope variable -content-
            // which was provided to sendPrivateMessage(User, String) as a parameter
        });
    }
    ```
    Since this only calls a single method in the callback you can use the short form:
    ```java
    public void sendPrivateMessage(User user, String content)
    {
        // notice that we are not placing a semicolon (;) in the callback this time!
        user.openPrivateChannel().queue( (channel) -> channel.sendMessage(content).queue() );
    }
    ```

## Using `submit()`

Sometimes execution needs to be cancelled if it isn't required anymore. This can be challenging to do if you use `queue()` or `complete()`.
<br>In `submit()` JDA will provide a `CompletableFuture` (aka promise) which allows the cancellation of a request.

If you don't need to use the `CompletableFuture` you may use `queue()` instead!

!!! example

    ```java
    public void setTestingChannel(TextChannel channel)
    {
        channel.getManager().setName("testing-channel").queue( (v) ->
            channel.sendMessage("Update Channel").queue( (m) ->
                m.delete().queueAfter(30, TimeUnit.SECONDS, (t) ->
                    logChannel.sendMessage("Deleted Response in %s", channel).queue()
                )
            )
        );
    }

    // turns into
    public void setTestingChannel(TextChannel channel)
    {
        channel.getManager().setName("testing-channel").submit()                   // CompletableFuture<Void>
            .thenCompose((v) -> channel.sendMessage("Update Channel").submit()) // CompletableFuture<Message>
            .thenCompose((m) -> m.delete().submitAfter(30, TimeUnit.SECONDS))   // CompletableFuture<Void>
            .thenCompose((v) -> logChannel.sendMessage("Deleted Response in %s", channel).submit())
            .whenComplete((s, error) -> {
                // this will be called for every termination (success/failure)
                // if the result is successful the error will be null
                // otherwise you should handle the error here to prevent it from being eaten and never printed
                if (error != null) error.printStackTrace();
            });
    }
    ```
    !!! note
        You can do the same with [`RestAction#flatMap`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/RestAction.html#flatMap(java.util.function.Function)) in 4.1.1

    ```java
    public class RateLimitListener extends ListenerAdapter
    {
        private final long guildId;
        private final long userId;
        private final Queue<RequestFuture<Void>> tasks = new LinkedList<>();

        public RateLimitListener(Guild guild, User user)
        {
            guildId = guild.getIdLong();
            userId = user.getIdLong();
            // only store IDs as JDA objects can be disposed by cache invalidation
            //when disposed the entity is not usable anymore, since we only need the id this is good enough
        }

        @Override
        public void onGuildMessageReceived(GuildMessageReceivedEvent event)
        {
            if (event.getAuthor().getIdLong() != userId)
                return; // ignore other users
            if (event.getGuild().getIdLong() != guildId)
                return; // ignore other guilds
            RequestFuture<Void> task = event.getMessage().delete().submit();
            tasks.add(task); // add task to cancel queue in case user gets banned
            task.thenRun(() -> tasks.remove(task)); // remove once completed
        }

        @Override
        public void onGuildBan(GuildBanEvent event)
        {
            if (event.getUser().getIdLong() != userId)
                return; // ignore other users
            if (event.getGuild().getIdLong() != guildId)
                return; // ignore other guilds

            // stop deleting messages for banned user
            RequestFuture<Void> current;
            while ((current = tasks.poll()) != null)
                current.cancel(true); 
            tasks.clear();

            // remove this as listener, our task has completed!
            event.getJDA().removeEventListener(this);
        }
    }
    ```

## Using `complete()`

The `complete()` operation is simply for your convenience. It will **block** the Thread that you call it on which means it will not be
able to continue with other tasks in the meantime.
<br>If you don't use the return value or don't need the request to be completed before continuing with other operations it is recommended to use `queue()` instead!

!!! example

    ```java
    public void setTestingChannel(TextChannel channel)
    {
        channel.getManager().setName("testing-channel").queue( (v) ->
            channel.sendMessage("Update Channel").queue( (m) ->
                m.delete().queueAfter(30, TimeUnit.SECONDS, (t) ->
                    logChannel.sendMessage("Deleted Response in %s", channel).queue()
                )
            )
        );
    }

    public void setTestingChannelBlocking(TextChannel channel)
    {
        channel.getManager().setName("testing-channel").complete();
        Message m = channel.sendMessage("Update Channel").complete();
        m.delete().completeAfter(30, TimeUnit.SECONDS);
        logChannel.sendMessage("Deleted Response in %s", channel).queue();
        // note how we used queue in the end because we don't need it sequenced anymore.
    }
    ```
    This is called a [callback hell](../assets/images/callback_hell.png)
    ```java
    public Message sendAndLog(MessageChannel channel, String message)
    {
        Message response = channel.sendMessage(message).complete();
        System.out.printf("Sent Message %s\n", response);
        return response;
    }
    ```
    ```java
    public PermissionOverride getOverride(Channel channel, Member member)
    {
        final PermissionOverride override = channel.getPermissionOverride(member);
        
        if (override == null)
            return channel.createPermissionOverride(member).complete();
        
        return override;
    }
    ```
    You can do this asynchronously by using a `CompletableFuture`:

    ```java
    public CompletableFuture<PermissionOverride> getOverride(Channel channel, Member member)
    {
        final PermissionOverride override = channel.getPermissionOverride(member);
        
        if (override == null)
            return channel.createPermissionOverride(member).submit();
        
        return CompletableFuture.completedFuture(override);
    }

    getOverride(channel, member).thenAccept(override -> ...);
    ```

## Using `completeAfter`, `submitAfter` and `queueAfter`

These three methods are also known under the term of __Planned Execution__ 
as they use a [ScheduledExecutorService](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html) to [schedule](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html#schedule-java.lang.Runnable-long-java.util.concurrent.TimeUnit-)
calls to either [complete](#using-complete) or [queue](#using-queue).

There are three possible ways to plan a RestAction execution

`completeAfter(long, TimeUnit)`
:   Blocks and executes on the current Thread, similar to `complete()`!
    Similar to using `Thread.join()` this will block until the action has completed.

`submitAfter(long, TimeUnit)`, `submitAfter(long, TimeUnit, ScheduledExecutorService)`
:   Creates a `DelayedCompletableFuture<T>` which will hold the response type as its generic value. This means using `get()` on the returned Future will cause the current thread to block and await the execution of the RestAction and receive the response type.

`queueAfter(long, TimeUnit)`, `queueAfter(long, TimeUnit, Consumer<T>)`, `queueAfter(long, TimeUnit, Consumer<T>, Consumer<Throwable>)`
:   Schedules the RestAction execution to be started after the specified delay, this will not block the thread and handle the execution in the background. You can optionally provide a ScheduledExecutorService to any of the `queueAfter` operations as the last argument.

When no ScheduledExecutorService is provided, these operations will use the default internal JDA ScheduledExecutorService that is also used to
execute queue callback consumers.

!!! example "Example `completeAfter`"

    ```java
    public Message waitForEdit(Message message)
    {
        return message.editMessage("5 Minutes are over").completeAfter(5, TimeUnit.MINUTES);
    }
    ```

!!! example "Example `queueAfter`"

    ```java
    public void remind(User user, String reminder, long delay, TimeUnit unit)
    {
        user.openPrivateChannel().queue(
            (channel) -> channel.sendMessage(reminder).queueAfter(delay, unit)
        );
    }

    public void remindAlternate(User user, String reminder, long delay, TimeUnit unit)
    {
        user.openPrivateChannel().queueAfter(delay, unit,
            (channel) -> channel.sendMessage(reminder).queue()
        );
    }
    ```

!!! example "Example `submitAfter`"

    ```java
    private Map<String, DelayedCompletableFuture<Message>> tasks = new HashMap<>();

    public ScheduledFuture<Message> sendWithTask(MessageChannel channel, String message)
    {
        DelayedCompletable<Message> task = channel.sendMessage(message).submitAfter(5, TimeUnit.SECONDS);
        return task;
    }

    public void doSomething(MessageChannel channel, String message)
    throws Exception
    {
        tasks.add(channel.getId(), sendWithTask(channel, message));
        for (DelayedCompletable<Message> task : tasks.values())
        {
            // non-blocking alternative is `thenAccept`
            System.out.printf("Task completed: %s\n", task.get());
        }
    }
    ```
