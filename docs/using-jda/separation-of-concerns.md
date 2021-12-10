# Separation of Concerns

In JDA we follow the pattern of [Separation of Concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) (SoC) for our entities.
This means that updates and moderation all happens in dedicated classes rather than on the entities themself (with a few exceptions like deletion)

Each entity that can be updated directly has a [Manager](#managers) instance which can be used to update one or more of the entity properties such as its name.

## Managers

The managers in JDA are useful to update entities without much complications, you can update all of the properties in a single [[RestAction|7)-Using-RestAction]] execution (http request).

### Updating an Entity

We will make a small example here on how to update the name and the topic of a [TextChannel](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/entities/TextChannel.html).

```java
public void updateChannel(TextChannel channel) {
    ChannelManager manager = channel.getManager(); // get the manager
    manager.setName("testing-2").setTopic("This is a testing channel, no memes allowed"); // set the new values
    manager.queue(); // execute update, this updates both name and topic
}
```
> Note: `queue()` is async so the update is not done when this method returns!

### Re-usability of Managers

Every manager in JDA is cached for re-use and can be updated for an interval and then executed upon command, very useful for bots!

[Example-State-Machine](https://gist.github.com/MinnDevelopment/190b79109b17c3bb446eea13be57c43c)

## Exceptions to SoC Pattern

The only exceptions we have are deletion and creation.
All entities are deleted directly using its delete() method, for instance [Channel.delete()](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/entities/Channel.html#delete()).

You can create copies of entities in the same fashion with one twist. Some `createCopy()` methods allow you to modify the new copy before execution of the **RestAction**

```java
public void copyChannel(Channel channel, String newName) {
    channel.createCopy().setName(newName).queue();
}
```
> Note: This will copy the provided `Channel` and set its name to the provided `newName`!

These things can be overlooked, so we do recommend to inspect the return type of these operations: `getManager()`, `create...()`, `delete()`, `ban(...)`, `kick(...)`, etc.