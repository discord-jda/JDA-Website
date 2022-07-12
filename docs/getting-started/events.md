# Using Events

## Registering your Listener

To register your listener, we currently have 2 Systems. Annotated Listeners _(AnnotatedEventManager)_ and Listeners implementing the Interface _EventListener (InterfacedEventManager)_. By default, the Interfaced one is used.

To switch between them, you can either use `JDABuilder#setEventManager(new AnnotatedEventManager())`, or `JDA#setEventManager(new MyEventManager())`.

After that, you just need to call `JDABuilder#addEventListeners(Object...)` or `JDA#addEventListeners(Object...)` with your Listener implementation.

```java title="Using JDABuilder"
// imports {...}
public class Launcher
{
    public static void main(String[] arguments)
    throws LoginException, InterruptedException
    {
        JDA api = JDABuilder.createDefault(arguments[0])
                      .addEventListeners(new PingPongBot())
                      .build().awaitReady();
    }
}
```

```java title="Using JDA"
// imports {...}
public class MyListeners
{
    public static void registerPingPongListener(JDA api)
    {
        api.addEventListeners(new PingPongBot());
    }
}
```

## Using the Interfaced System (default)

When using the interfaced system (default), your Listener(s) have to implement the Interface _EventListener_, which only has a single function to implement: `public void onEvent(GenericEvent event)`.

For convenience, we also included the class _ListenerAdapter_, which comes with a wide set of predefined functions targeted at specific event-types.

!!! example "Examples"
    ```java title="Using EventListener"
    public class Test implements EventListener
    {
        @Override
        public void onEvent(GenericEvent event)
        {
            if(event instanceof MessageReceivedEvent)
                System.out.println(event.getMessage().getContentDisplay());
        }
    }
    ```
    ```java title="Using ListenerAdapter"
    public class Test extends ListenerAdapter
    {
        @Override
        public void onMessageReceived(MessageReceivedEvent event)
        {
            System.out.println(event.getMessage().getContentDisplay());
        }
    }
    ```
    _Don't forget actually registering this listener_


## Using the Annotated System

When using the annotated system, all listener methods have to have the `@SubscribeEvent` annotation present, and only accept a single parameter, which has to be a instance of _Event_.

!!! example
    ```java
    public class Test
    {
        public static void main(String[] args)
        throws LoginException
        {
            JDABuilder.createDefault(TOKEN)
                .setEventManager(new AnnotatedEventManager())
                .addEventListeners(new Test())
                .build();
        }

        @SubscribeEvent
        public void ohHeyAMessage(MessageReceivedEvent event)
        {
            System.out.println(event.getMessage().getContentDisplay());
        }
    }
    ```
    _Don't forget actually registering this listener_