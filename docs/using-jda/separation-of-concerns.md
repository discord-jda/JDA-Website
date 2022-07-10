# Separation of Concerns

In JDA we follow the pattern of [Separation of Concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) (SoC) for our entities.
This means that updates and moderation all happens in dedicated classes rather than on the entities themselves (with a few exceptions like deletion)

Each entity that can be updated directly has a [Manager](#managers) instance which can be used to update one or more of the entity properties such as its name.

## Managers

The managers in JDA are useful to update entities.
Without many complications, you can update many of properties in a single [RestAction](using-restaction.md) execution (HTTP request).

### Updating an Entity

We will make a small example here on how to update the name and the topic of a [TextChannel](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/TextChannel.html).

```java
public void updateChannel(TextChannel channel) {
    ChannelManager manager = channel.getManager(); // get the manager
    manager.setName("testing-2").setTopic("This is a testing channel, no memes allowed"); // set the new values
    manager.queue(); // execute update, this updates both name and topic
}
```
!!! note 
    `queue()` is async so the update is not done when this method returns!

### Re-usability of Managers

Every manager in JDA is cached for re-use and can be updated for an interval and then executed upon command, very useful for bots!

??? example "Example State Machine"
    ```java
    import net.dv8tion.jda.api.events.message.MessageReceivedEvent;
    import net.dv8tion.jda.api.hooks.ListenerAdapter;
    import net.dv8tion.jda.api.entities.*;
    import net.dv8tion.jda.api.MessageBuilder;
    import net.dv8tion.jda.api.requests.restaction.MessageAction;
    
    // derived with permission from https://gist.github.com/MinnDevelopment/190b79109b17c3bb446eea13be57c43c
    public class UpdateStateMachine extends ListenerAdapter {
    private final TextChannel channel;
    private String name = null;
    private String topic = null;
    private int state = 0;
    
        public UpdateStateMachine(TextChannel channel) {
            this.channel = channel;
        }
    
        @Override
        public void onMessageReceived(MessageReceivedEvent event) {
            if (!event.isFromGuild()) return;
            if (!event.getChannel().equals(channel)) return;
            String content = event.getMessage().getContentRaw();
            if (!content.startsWith("!update ")) return;
            String parts[] = content.split(" ", 2);
            String arguments = parts.length > 1 ? parts[1] : "";
            switch (state) {
                case 0: // setting mode
                    if (arguments.equals("done")) {              // finish update
                        state = 2; // enter update mode
                        sendStatus().append("\nType `!update yes` to finish!").queue();
                    } else if (arguments.equals("reset")) {      // reset state
                        state = 1; // enter reset mode
                        sendStatus().append("\nType `!update yes` to reset!").queue();
                    } else if (arguments.startsWith("topic ")) { // update topic
                        this.topic = arguments.split(" ", 2)[1];
                        channel.getManager().setTopic(this.topic);
                    } else if (arguments.startsWith("name ")) {  // update name
                        this.name = arguments.split(" ", 3)[1];
                        channel.getManager().setName(this.name);
                    } else {
                        channel.sendMessage("I'm sorry I did not understand.").queue();
                    }
                    break;
                case 1: // reset mode
                    state = 0;
                    if (arguments.equals("yes"))  // we should reset
                        resetState();
                    else                          // nevermind, we are not done
                        channel.sendMessage("Ok, what do you want to change?").queue();
                    break;
                case 2: // update mode
                    state = 0;
                    if (arguments.equals("yes")) // we should update
                        channel.getManager().queue((v) -> resetState());
                    else                         // nevermind, we are not done
                        channel.sendMessage("Ok, what do you want to change?").queue();
                    break;
            }
        }
    
        protected void resetState() {
            channel.getManager().reset();
            this.name = null;
            this.topic = null;
        }
    
        private MessageAction sendStatus() {
            MessageBuilder builder = new MessageBuilder();
            builder.append("**Current Status**");
            if (name != null || topic != null) {
                StringBuilder changes = new StringBuilder();
                if (name != null)
                    changes.append("name -> ").append(name).append("\n");
                if (topic != null)
                    changes.append("topic -> ").append(topic);
                builder.appendCodeBlock(changes, "");
            }
            return builder.sendTo(channel);
        }
    }
    ```

## Exceptions to SoC Pattern

The only exceptions we have are deletion and creation.
All entities are deleted directly using its delete() method, for instance [Channel.delete()](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/Channel.html#delete()).

You can create copies of entities in the same fashion with one twist. Some `createCopy()` methods allow you to modify the new copy before execution of the **RestAction**

!!! note inline end 
    This will copy the provided `Channel` and set its name to the provided `newName`!

```java
public void copyChannel(Channel channel, String newName) {
    channel.createCopy().setName(newName).queue();
}
```


These things can be overlooked, so we do recommend inspecting the return type of these operations: `getManager()`, `create...()`, `delete()`, `ban(...)`, `kick(...)` and similar when you use them so that you are not caught out.