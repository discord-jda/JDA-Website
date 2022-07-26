# Using Events

## Registering your Listener

JDA uses the observer pattern for its event system.  In order to use events, a listener has to be used and registered with JDA.

There are currently two systems for listeners: the default implementation which requires subclassing/implementing `EventListener` or `ListenerAdapter`, and an annotated listener `AnnotatedEventManager` which runs annotated methods based on their signature.

To switch between them, you can use `JDA#setEventManager(new InterfacedEventManager())` or `JDABuilder#setEventManager(new AnnotatedEventManager())`.

After that, you just need to call `JDABuilder#addEventListeners(Object...)` or `JDA#addEventListeners(Object...)` with your listeners.

Adding listeners to the `JDABuilder` as opposed to the `JDA` instance will ensure that all listeners will receive events during startup. 
If the listeners are added after JDA is built, they will not receive any events that occurred before they were added.

```java
public class ExampleListener extends ListenerAdapter
{
    public static void main(String[] args) throws LoginException
    {
        JDA jda = JDABuilder.createDefault(DISCORD_TOKEN)
                            .addEventListeners(new ExampleListener())
                            .build();
    }
    
    @Override
    public void onReady(ReadyEvent event)
    {
        System.out.println("JDA Started!");    
    }
}
```

## Using the Interfaced System (default)

When using the interfaced system (default), your Listener(s) have to implement the Interface _EventListener_, which only has a single function to implement: `public void onEvent(GenericEvent event)`.

For convenience, we also included the class _ListenerAdapter_, which comes with a wide set of predefined functions targeted at specific event-types.

!!! Warning


!!! example "Examples"
    Using `instanceof` checks is cumbersome, so JDA provides the `ListenerAdapter` to ease this.
    ```java title="Using EventListener"
    public class Test implements EventListener
    {
        @Override
        public void onEvent(GenericEvent event)
        {
            if (event instanceof MessageReceivedEvent)
            {
                MessageReceivedEvent messageEvent = (MessageReceivedEvent) event;
                System.out.println(messageEvent.getMessage().getContentDisplay());
            }
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


## Using the Annotated System

When using the annotated system, all listener methods have to have the `@SubscribeEvent` annotation present, and only accept a single parameter, which has to be an implementation of _Event_.

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