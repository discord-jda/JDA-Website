# Interactions

This guide will give you a brief introduction to an API for adding and handling interactions in Discord. Interactions are a way to integrate your bot features directly into the Discord User Interface. These things include features such as:

- Slash Commands
- Buttons

### Ephemeral Messages

These ephemeral messages are only visible to the user who used your Slash-Command/Button. They are similar to the messages discord sends you when you update your nickname with `/nick`.

Ephemeral messages have some limitations and will be removed once the user restarts their client.

Limitations:

- Cannot be deleted by the bot
- Cannot contain any files/attachments
- Cannot be reacted to
- Cannot be retrieved

Example:

![EphemeralMessage](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/assets/assets/wiki/interactions/EphemeralMessage.png)

You can only create ephemeral messages with interactions. For example with `deferReply(true)`, `reply(content).setEphemeral(true)`, or `getHook().sendMessage(content).setEphemeral(true)`. For convenience you can also configure the `InteractionHook` to default to ephemeral messages with `hook.setEphemeral(true)`.


## Slash Commands

A slash command is something you might already be familiar with from the olden times of Discord. Commands such as `/shrug` or `/me` have existed for quite a long time. With Slash Command interactions you can now make your very own commands like this! But these commands come with some limitations, which I have explained in this gist: [Slash Command Limitations](https://gist.github.com/MinnDevelopment/b883b078fdb69d0e568249cc8bf37fe9)

![Example Slash Commands](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/52377f69d1f3bfba909c51a449ac6b258f606956/assets/wiki/interactions/SlashCommands.png)

All of these commands are used through the interactions API. They do not require the user to send an actual message in the channel and you don't have to do string parsing either.

To create commands you need to make some API requests. There are 2 types of commands you can create called **global commands** and **guild commands**.

- **Global**: These commands are available in every server your bot is in (regardless of sharding!) and direct message (Private Channels). These commands can take up to 1 hour to show up. _It is recommended to use guild commands for testing purposes._
- **Guild**: These commands are only in the specific guild that you created them in and cannot be used in direct messages. These commands show up immediately after creation.

You can create commands through these methods in JDA:

- `updateCommands()`
- `upsertCommand(name, description)`

To create global commands you need to call these on a `JDA` instance and for guild commands on a `Guild` instance. Your bot needs the `applications.commands` scope in addition to the `bot` scope for your bot invite link. Example: https://discord.com/oauth2/authorize?client_id=123456789&scope=bot+applications.commands <- here

Once a command is created, it will continue persisting even when your bot restarts. Commands stay until the bot is either kicked or your bot explicitly deletes the command. **You don't need to create your commands every time your bot starts!**

### Responding to Slash Commands

When a user tries to use one of your commands you will receive a `SlashCommandEvent`. This event needs to be handled by your event listener.
The flow of a slash command response is as follows:

1. Acknowledge the command

This means you need to either **reply** or **deferReply**. You only have ***3 SECONDS*** to acknowledge a command.
Since some commands may take longer than 3 seconds you may want to use `deferReply` to have more time for handling. This will instead send a `Thinking...` message to channel which is later updated by a followup message (see step 2).

![Example Thinking](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/52377f69d1f3bfba909c51a449ac6b258f606956/assets/wiki/interactions/DeferredReply.gif)

2. Send followup messages

Sometimes commands need more than one response. However, you can only send one initial reply to a command. To send additional messages for the same slash command you need to use the `InteractionHook` attached to the event with `getHook()`. This is a **webhook** that allows you to send additional messages for up to 15 minutes after the initial command.

When you use `deferReply` the first message sent to this webhook will act identically to using `editOriginal(...)`. The message you send is also referred to as *deferred reply* in this case. Your deferred reply will **edit** your initial `Thinking...` message instead of sending an additional message to channel. This means you cannot use `setEphemeral` on this deferred reply since you already decided whether the message will be ephemeral through your initial acknowledgment.

**Example Reply**

```java
public class SayCommand extends ListenerAdapter {
  @Override
  public void onSlashCommand(SlashCommandEvent event) {
    if (event.getName().equals("say")) {
      event.reply(event.getOption("content").getAsString()).queue(); // reply immediately
    }
  }
}
```

**Example Deferred Reply**

```java
public class TagCommand extends ListenerAdapter {
  @Override
  public void onSlashCommand(SlashCommandEvent event) {
    if (event.getName().equals("tag")) {
      event.deferReply().queue(); // Tell discord we received the command, send a thinking... message to the user
      String tagName = event.getOption("name").getAsString();
      TagDatabase.fingTag(tagName,
        (tag) -> event.getHook().sendMessage(tag).queue() // delayed response updates our inital "thinking..." message with the tag value
      );
    }
  }
}
```


## Buttons

Every message supports a set of components within specific layouts. Currently, the only component is [Button](https://javadoc.jitpack.io/com/github/dv8fromtheworld/jda/fe6dc24/javadoc/net/dv8tion/jda/api/interactions/components/Button.html) and the only layout is [ActionRow](https://javadoc.jitpack.io/com/github/dv8fromtheworld/jda/fe6dc24/javadoc/net/dv8tion/jda/api/interactions/components/ActionRow.html).

Each button can be enabled or disabled, have a specific style, label, and emoji:

![Example Button Styles](https://raw.githubusercontent.com/DV8FromTheWorld/JDA/52377f69d1f3bfba909c51a449ac6b258f606956/assets/wiki/interactions/ButtonExamples.png)

To add such buttons to a message you can use up to 5 ActionRows.

You can add multiple ActionRows with either `setActionRows` or `addActionRows` (depending on whether you are sending or editing a message).
For the common case of a single ActionRow you can also use `setActionRow(Button...)` or `addActionRow(Button...)`.

### Handling ButtonClickEvent

To properly use an interactive Button, meaning any non-link button, you need to use the **Component ID** (aka **Custom ID**, **Button ID**).
Each non-link button requires such an ID in order to be used and this ID can also be used to then *identify* which button was pressed by the user.

When a user presses one of these buttons, you will receive a `ButtonClickEvent` for the respective interaction. This event provides the configured ID with `getComponentId()`.

These button interactions, or component interactions, offer 4 response types:

- Reply
- Deferred Reply
- Edit Message
- Deferred Edit Message

The reply and deferred reply responses are identical to the Slash-Commands response types. However, these new edit response types are used to update the existing message the button is attached to. If you simply want to acknowledge that the button was pressed, you can simply do a `deferEdit()` without any further updates, which will prevent the interaction from failing on the user side.

**Example**:

```java
 public class HelloBot extends ListenerAdapter {
   @Override
   public void onSlashCommand(SlashCommandEvent event) {
       if (event.getName().equals("hello")) {
           event.reply("Click the button to say hello")
               .addActionRow(
                 Button.primary("hello", "Click Me"), // Button with only a label
                 Button.success("emoji", Emoji.fromMarkdown("<:minn:245267426227388416>"))) // Button with only an emoji
               .queue();
       } else if (event.getName().equals("info")) {
           event.reply("Click the buttons for more info")
               .addActionRow( // link buttons don't send events, they just open a link in the browser when clicked
                   Button.link("https://github.com/DV8FromTheWorld/JDA", "GitHub")
                     .withEmoji(Emoji.fromMarkdown("<:github:849286315580719104>")), // Link Button with label and emoji
                   Button.link("https://ci.dv8tion.net/job/JDA/javadoc/", "Javadocs")) // Link Button with only a label
               .queue();
       }
   }

   @Override
   public void onButtonClick(ButtonClickEvent event) {
       if (event.getComponentId().equals("hello")) {
           event.reply("Hello :)").queue(); // send a message in the channel
       } else if (event.getComponentId().equals("emoji")) {
           event.editMessage("That button didn't say click me").queue(); // update the message
       }
   }
 }
```