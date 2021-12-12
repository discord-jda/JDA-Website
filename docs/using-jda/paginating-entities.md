# PaginationAction

In some scenarios the Discord API allows to _paginate_ endpoints to retrieve entities in bulks.
<br>Such an endpoint is especially often used for retrieving past messages in the client by scrolling up.

In JDA we allow iterating such endpoints with the [Iterable](https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html) `PaginationAction` implementations.
Using it inside an [enhanced for-loop](https://blogs.oracle.com/corejavatechtips/using-enhanced-for-loops-with-your-classes) is a blocking and expensive operation so it is recommended to use the async methods instead.

!!! example "Example Messages"

    ```java
    /**
    * Retrieves up to 1000 messages from the provided MessageChannel
    * and then provides them to the callback.
    */
    public void get1000(MessageChannel channel, Consumer<List<Message>> callback)
    {
        List<Message> messages = new ArrayList<>(1000);
        channel.getIterableHistory().cache(false).forEachAsync((message) ->
        {
            messages.add(message);
            return messages.size() < 1000;
        }).thenRun(() -> callback.accept(messages));
    }

    get1000(channel, (messages) -> channel.purgeMessages(messages));
    ```

## Entity Cache

!!! note inline end
    PaginationActions support the Stream api which was introduced in Java 1.8.
     Stream requires blocking operations and can take long to finish.

Every PaginationAction has a cache of already retrieved entities.
<br>This cache is enabled by default but can be disabled via `PaginationAction.cache(false)`. 
Doing so is recommended if the cache is not needed.

To retrieve all cached entities use `PaginationAction.getCached()`. This returns
an immutable List representing all cached entities (thread-safe but expanding).
<br>You may also get the first/last entity with `getLast()` and `getFirst()` for convenience.

By default the PaginationAction will retrieve the maximum amount of entities per request (complete/queue).
<br>It is advised to lower this limit if only a small number of entities is required. (See `limit(int)`).



!!! example "Usage examples"

    ```java
    public class PaginationUtil
    {
        // Calls the callback if the user has reacted
        public static void ifReacted(MessageReaction reaction, User user, Runnable callback)
        {
            reaction.getUsers().cache(false).forEachAsync(u -> {
                if (u.equals(user)) {
                    callback.run(); // user has reacted -> call the callback
                    return false; // end iteration
                }
                return true; // continue iteration
            });
        }

        // Blocking iteration using the Iterable interface (not recommended)
        public static List<Message> forEachMessage(int limit, MessageChannel channel, Consumer<Message> action)
        {
            MessagePaginationAction paginator = channel.getIterableHistory();
            for (Message message : paginator)
            {
                action.apply(message);
                if (--limit <= 0) break;
            }
            return paginator.getCached();
        }
    }
    ```