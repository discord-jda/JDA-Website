# Migration Guide 4.X to 5.X

This version utilizes Discord API v10, and includes several breaking changes and improvements.

## Dependency Installation

Before continuing, we should mention that JDA versions are now distributed via [Maven Central](https://mvnrepository.com/artifact/net.dv8tion/JDA/latest). You can remove the old `m2.dv8tion.net` resolver from your build files.

Replace `VERSION` with the latest version ![Maven Version](https://img.shields.io/maven-central/v/net.dv8tion/JDA?color=blue)

=== "Gradle"
    ```groovy
    repositories {
        mavenCentral()
    }
    
    dependencies {
        implementation("net.dv8tion:JDA:VERSION")
    }
    ```

=== "Maven"
    ```xml
    <dependency>
      <groupId>net.dv8tion</groupId>
      <artifactId>JDA</artifactId>
      <version>VERSION</version>
    </dependency>
    ```

## Additional Resources

This migration guide does not include every single detail and does not focus on any new features available in JDA 5, such as [ThreadChannels](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/channel/concrete/ThreadChannel.html). Here are some useful resources to learn more if you are curious about all the new things we added:

- [Examples](https://github.com/DV8FromTheWorld/JDA/tree/master/src/examples/java)
- [JDA 5 Javadocs](https://ci.dv8tion.net/job/JDA5/javadoc/)
- [Releases](https://github.com/DV8FromTheWorld/JDA/releases)

## Extensions That Support 5.X

Here is a list of known extensions that support 5.X.
The ones that are not checked do not support it yet. You should check that everything used in your project supports 5.X before starting migration.

- [x] [jda-reactor](https://github.com/MinnDevelopment/jda-reactor)
- [x] [LavaPlayer](https://github.com/sedmelluq/lavaplayer)
- [x] [jda-nas](https://github.com/sedmelluq/jda-nas)
    - [x] [udpqueue.rs](https://github.com/MinnDevelopment/udpqueue.rs) (for minimal Rust bindings)
- [x] [jda-ktx](https://github.com/MinnDevelopment/jda-ktx)
- [ ] [JDA-Utilities](https://github.com/JDA-Applications/JDA-Utilities)

## Introduction of Message Content Privileged Intent

As part of upgrading to API v10, accessing the following user-generated content in messages now requires the Message Content privileged intent (`GatewayIntent.MESSAGE_CONTENT`):

- Text content (`Message#getContentRaw`, `Message#getContentDisplay`, `Message#getContentStripped`)
- Embeds (`Message#getEmbeds`)
- Attachments (`Message#getAttachments`)
- Message Components (`Message#getActionRows`, `Message#getButtons`)
- Custom Emoji Mentions (`Message#getMentions()#getCustomEmojis()`)

You must enable this intent in your `JDABuilder` **AND** in the Discord Developer Portal for your app if you utilize any of the above methods. For more information on intents, read the [dedicated wiki page](/using-jda/gateway-intents-and-member-cache-policy).

## Channel Rework

There are several breaking changes to `GuildChannel` and `ChannelManager`. Firstly, the packages of most channel interfaces have changed. What was previously simply found in `net.dv8tion.jda.api.entities` is now split over the following packages:

- `net.dv8tion.jda.api.entities.channel`<br>
    Top level channel enums and the new `Channel` type interface.
- `net.dv8tion.jda.api.entities.channel.attribute`<br>
    Interfaces dedicated to specific fields, such as slowmode or permissions. This also has some interfaces for *concepts*, such as copying or categorizing.
- `net.dv8tion.jda.api.entities.channel.concrete`<br>
    The bottom level interfaces for exact concrete types, such as `VoiceChannel` and `TextChannel`.
- `net.dv8tion.jda.api.entities.channel.forums`<br>
    Types related to `ForumChannel`, such as tags.
- `net.dv8tion.jda.api.entities.channel.middleman`<br>
    Abstractions that apply to multiple concrete types, such as `MessageChannel` which covers all message related methods.
- `net.dv8tion.jda.api.entities.channel.unions`<br>
    Interfaces used for return values, allowing for downcasting into specific types. They also provided shared features of the specific interface they extend.

### Channel Return Types

Many classes in JDA had specific getters for each channel type. For example, messages had `getTextChannel()` and `getPrivateChannel()`. The goal was to allow easy conversion to concrete types that have more functionality than their abstract counterpart (such as `MessageChannel`). In JDA v5, we've introduced new **Channel Union** interfaces to deal with this problem in a more elegant way. Instead of using `foo.getTextChannel()`, you now use `foo.getChannel().asTextChannel()`. The `getChannel()` method now often returns a specific **Union** type such as `MessageChannelUnion`, which doubles as the abstract implementation for `MessageChannel` (allows to send messages) and also downcasting via various `asX()` methods.

### New Channel Attribute Interfaces

`GuildChannel` now implements `getGuild` and `getManager` *only*. It's meant to serve as a generic type to hold channels from guilds.

Specific channel attributes, such as slowmode and permissions, have been split into several interfaces. These interfaces can be found in [the `net.dv8tion.jda.api.entities.channel.attribute` package](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/channel/attribute/package-summary.html). Each interface in this package extends `GuildChannel`.

### Permission Access Changes

With JDA v5 we reduced the capabilities of the `GuildChannel` interface. The method `GuildChannel#getPermissionContainer` has been introduced to make accessing permissions easier. This allows for users to confidently access the entity that dictates the permissions for a given channel without having to check whether the channel actually supports permissions, such as in the case of threads.

Of course, if you have a channel type that already supports `IPermissionContainer`, you don't need this getter.

### Type-Trimmed Channel Managers

In JDA v4, `GuildChannel#getManager` returned a `ChannelManager` that gave every possible setter for every channel type. This caused for `UnsupportedOperationException` or `IllegalStateException` to be thrown in some cases, such as calling `setBitrate` on a `TextChannel`.

JDA v5 provides type-trimmed channel managers, which provide only the setters that we know *for sure* can work on the given channel. Each of these managers can be found in [the `net.dv8tion.jda.api.managers.channel` package](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/managers/channel/package-summary.html). These all follow the same implement/extension hierarchy as the channels do and map 1:1.

### Independent Stage Channel and News Channel Entities

[`StageChannel`](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/channel/concrete/StageChannel.html) and [`NewsChannel`](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/channel/concrete/NewsChannel.html) were previously variants of `VoiceChannel` and `TextChannel`. These are now their own independent entities. This comes with a number of changes:

- `ChannelAction#setNews` is replaced by `ChannelAction.setType`
- `ChannelManager#getType` is replaced by `ChannelManager#getChannel()#getType`
- `ChannelManager#setNews` is replaced by `ChannelManager#setType`
- `TextChannel#crosspostMessage` is replaced by `NewsChannel#crosspostMessage`
- `TextChannel#follow` is replaced by `NewsChannel#follow`
- `TextChannel#isNews` was removed

### Other Changes

- `StoreChannel` was removed
- `PrivateChannel#getUser` is now nullable
- `MessageChannel#getLatestMessageId` and `MessageChannel#getLatestMessageIdLong` no longer change to null if the message was deleted

## Permission Changes

A number of permissions have been renamed, with one permission being removed entirely.

- `Permission.MANAGE_EMOTES` renamed to `Permission.MANAGE_EMOTES_AND_STICKERS`
- `Permission.MESSAGE_WRITE` renamed to `Permission.MESSAGE_SEND`
- `Permission.USE_SLASH_COMMANDS` renamed to `Permission.USE_APPLICATION_COMMANDS`
- `Permission.USE_PUBLIC_THREADS` renamed to `Permission.CREATE_PUBLIC_THREADS`
- `Permission.USE_PRIVATE_THREADS` renamed to `Permission.CREATE_PRIVATE_THREADS`
- `Permission.MESSAGE_READ` was removed in favor of `Permission.VIEW_CHANNEL`

## Event Changes

There are several changes related to events.

### Removal of Guild Context Message Events

All message event types for guild messages have been removed. Examples of these are:

- `Generic<Context>MessageEvent` (ex: `GenericGuildMessageEvent`)
- `<Context>MessageReceivedEvent` (ex: `GuildMessageReceivedEvent`)
- `<Context>MessageUpdateEvent` (ex: `GuildMessageUpdateEvent`)
- `<Context>MessageDeleteEvent` (ex: `GuildMessageDeleteEvent`)
- `<Context>MessageEmbedEvent` (ex: `GuildMessageEmbedEvent`)
- `<Context>MessageReaction<X>Event` (ex: `GuildMessageReactionAddEvent`)

Instead, you should use the unified versions of these events:

- `GenericMessageEvent`
- `MessageReceivedEvent`
- `MessageUpdatedEvent`
- `MessageDeletedEvent`
- `MessageReaction<X>Event` (ex: `MessageReactionAddEvent`)

You can check if a message is from a `Guild` by using `Message#isFromGuild`.

### Changes to Specifying GatewayIntents from Event Types

With context-specific events being removed (such as `GuildMessageReceivedEvent`), you can no longer use these events to specify `GatewayIntent.GUILD_MESSAGES` or `GatewayIntent.DIRECT_MESSAGE_REACTIONS`. You may supply any of the unified message events, but the functionality will be slightly different. It is not recommended to use `GatewayIntent#fromEvents` if you wish to have finer grain control around intents for messages.

### Session Events

All events that update the gateway session of a bot now extend a common [`GenericSessionEvent`](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/events/session/GenericSessionEvent.html). Such events are located within [the `net.dv8tion.jda.api.events.session` package](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/events/session/package-summary.html). This also includes `ReadyEvent` and `ShutdownEvent`.

Some events relating to sessions have been renamed:

- `DisconnectEvent` renamed to `SessionDisconnectEvent`
- `ReconnectedEvent` renamed to `SessionRecreateEvent`
- `ResumedEvent` renamed to `SessionResumeEvent`

### Voice State Events

`GuildVoiceJoinEvent` and `GuildVoiceLeaveEvent` have both been removed in favor of the unified [`GuildVoiceUpdateEvent`](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/events/guild/voice/GuildVoiceUpdateEvent.html).

As an example, to detect when a user leaves a voice channel, you can use `GuildVoiceUpdateEvent#getChannelLeft`. This method will return the channel that the user left, or `null` if the user joined a channel instead.

## Intent and Member Cache Changes

A few changes to intents and member caching were also made to improve the user experience.

### Renamed Intents

The following intents have been renamed to align with the API name convention:

- `GatewayIntent.GUILD_BANS` renamed to `GatewayIntent.GUILD_MODERATION`
- `GatewayIntent.GUILD_EMOJIS` renamed to `GatewayIntent.GUILD_EMOJIS_AND_STICKERS`

### Chunking and Caching

If you used `ChunkingFilter.ALL` in the past, JDA would automatically also cache all those members it chunked. However, this is not a very flexible rule, and we changed this behavior. Instead, the `MemberCachePolicy` will control which members to keep cached after chunking.

## Sticker and Emoji Rework

We now fully support the sticker API in JDA. You can send up to 3 stickers in messages and receive both guild and nitro-only stickers. To improve the user experience with emotes/emoji we also changed this API.

### Sticker Changes

The old `MessageSticker` class has been removed in favor of an assortment of new interfaces located in `net.dv8tion.api.entities.sticker`. Instead, `Message#getStickers` now returns the new `StickerItem` interface. You can create a sendable sticker instance with `Sticker.fromId(stickerId)` and then pass it to `GuildMessageChannel#sendStickers`.

### Emote/Emoji Changes

In old versions we always made a distinction between **Emote** and **Emoji** to differentiate **Custom Emoji** from **Unicode Emoji**. This naming scheme was not ideal and caused a bit of confusion. In our redesign we instead simplify this to the names `CustomEmoji` and `UnicodeEmoji`. Some classes/interfaces had to be renamed for this new design:

- `Emote` renamed to `RichCustomEmoji`
- `EmoteManager` renamed to `CustomEmojiManager`
- `MessageReactionRemoveEmoteEvent` renamed to `MessageReactionRemoveEmojiEvent`

All methods/enums/types using `emote` have also been adjusted to say `emoji`, for example `Guild#retrieveEmotes` is now `Guild#retrieveEmojis` and `CacheFlag.EMOTE` is now `CacheFlag.EMOJI`.

This new design also had the goal to unify all usages of emoji in the API, allowing you to use them for anything that makes use of emoji. For instance, previously there was a distinction between reaction emoji (`MessageReaction.ReactionEmote`) and button emoji (`Emoji`), which now both use the same `Emoji` type:

```java
public void onMessageReactionAdd(MessageReactionAddEvent event) {
  event.getChannel().sendMessage("User reacted")
    .setActionRow(Button.primary("buttonid", event.getEmoji()))
    .queue();
}
```

To check which emoji was used in a reaction, you can use `emoji.equals(otherEmoji)` instead of checking for id/name. This has the advantage for also checking the correct type for you. For example: `event.getEmoji().equals(Emoji.fromFormatted("😃"))`.

You also now use `Emoji` instances for reactions. What was previously `message.addReaction("...")` is now `message.addReaction(Emoji.fromFormatted("..."))`. And in general you can use the following factory methods to create emoji instances:

- `Emoji.fromUnicode`<br>
    Parses codepoint notation like `"U+1F602"` or simply uses the provided unicode characters `"😃"`. This returns a concrete `UnicodeEmoji` type instance.
- `Emoji.fromCustom`<br>
    Creates a `CustomEmoji` instance from the provided name and id.
- `Emoji.fromFormatted`<br>
    Parses emoji instances from markdown such as `"<:minn:12345581261712671>"` and also supports unicode such as `"😃"` or codepoint notation `"U+1F602"`. This returns a `EmojiUnion` instance, which can be either custom or unicode.

You can see the full list of breaking changes in [#2117](https://github.com/DV8FromTheWorld/JDA/pull/2117).

## Message Send/Edit Rework

In JDA 5 we are separating the handling of message sending and editing, while also unifying all send and edit functionality in the API. The old `MessageBuilder` and `MessageAction` have been split up:

- `MessageCreateBuilder`
- `MessageCreateAction`
- `MessageEditBuilder`
- `MessageEditAction`

This change should only affect people who made use of `MessageBuilder` in the past. You need to update your code to either the edit or create builders. If you don't know whether the resulting operation is an edit or send request, you can simply always use `MessageEditBuilder` and then use `MessageCreateData.fromEdit(MessageEditData)` to convert it at call-site. You can also use similar factory methods to create a sendable message from the `Message` interface (`MessageCreateData.fromMessage(message)`).

Previously, edit requests had a method called `override(boolean)` to *replace* the entire message. This was used to remove content or embeds from a message. You can now simply use `setEmbeds(emptyList())` or `setContent("")` to remove specific parts of the message, or use `setReplace(true)` to achieve the same functionality of replacing everything.

### Method Renames

Some methods were renamed to allow for consistency between all requests.

- `setActionRows`/`addActionRows` renamed to `setComponents`/`addComponents`
- `MessageAction#tts` renamed to `MessageCreateRequest#setTTS`
- `MessageAction#content` renamed to `MessageCreateRequest#setContent`
- `MessageAction#allowedMentions` renamed to `MessageRequest#setAllowedMentions`
- `addFile` removed in favor of `addFiles` and `setAttachments` (see **File Sending** section below)

### Unifying Message Requests

All message send and edit requests now use a unified `MessageRequest` interface. This allows you to make very abstracted implementations that use high level interfaces.

![Message Rework Hierarchy](https://cdn.discordapp.com/attachments/875797959072161843/1009139444772782130/unknown.png)

This also means that all methods from `message.editMessage(...)` are consistent with the methods from `interaction.editMessage(...)`, and analogously for sending. You can now use `addActionRow` when sending a message to a channel.

### File Sending

The old `sendFile(...)`/`replyFile(...)` overloads available on `MessageChannel` and interactions has been replaced by a single `sendFiles(...)` method. This new method accepts the `FileUpload` type, which also supports file descriptions (alt text) via `setDescription(...)`. For example, old code such as `sendFile(data, name, AttachmentOption.SPOILER)` is replaced with `sendFiles(FileUpload.fromData(data, name).asSpoiler())`.

Due to changes in the Discord API v10, you can no longer add files to messages without replacing all existing attachments. We could previously support methods such as `MessageAction#addFile` for editing messages, which is no longer possible in API v10. Instead, you **must** replace the entire list of attachments, using `MessageEditAction#setAttachments`. Here is an example:

```java
// Here "message" is an instance of the Message interface

// Take the first attachment of the message, all others will be removed
AttachedFile attachment = message.getAttachments().get(0);

// The name here will be "cat.png" to discord, what the file is called on your computer is irrelevant and only used to read the data of the image.
FileUpload file = FileUpload.fromData(new File("mycat-final-copy.png"), "cat.png"); // Opens the file called "cat.png" and provides the data used for sending

// Edit request to keep the first attachment, and add one more file to the message
message.editMessage("New content")
       .setAttachments(attachment, file)
       .queue();
```


### Message Splitting

The old `MessageBuilder#buildAll` has been removed from the builder classes. Instead, you can now use the new `SplitUtil` utility class to split any string. For example:

**Old:**

```java
List<Message> messages = new MessageBuilder().setContent(someLargeString).buildAll();
```

**New:**

```java
List<String> contents = SplitUtil.split(
    someLargeString,  // input string of arbitrary length
    2000,             // the split limit, can be arbitrary (>0)
    true,             // whether to trim the strings (empty will be discarded)
    Strategy.NEWLINE, // split on '\n' characters if possible
    Strategy.ANYWHERE // otherwise split on the limit
);
// Convert to instance of MessageCreateData (optional, you can just send strings directly!)
List<MessageCreateData> messages = contents.stream().map(MessageCreateData::fromContent).toList();
```

## Mentions Rework

We made the handling of mentions consistent across both messages and interactions. You can now access mentions inside string options of slash commands through `OptionMapping#getMentions`. The old methods on `Message` have also been moved to a new `Mentions` interface, accessible via `Message#getMentions`.

- `Message#getMentionedUsers` moved to `Mentions#getUsers`
- `Message#getMentionedMembers` moved to `Mentions#getMembers`
- `Message#getMentionedChannels` moved to `Mentions#getChannels`
- `Message#getEmotes` moved to `Mentions#getCustomEmojis`
- `Message#getMentions(MentionType...)` moved to `Mentions#getMentions(MentionType...)`
- `Message#isMentioned` moved to `Mentions#isMentioned`
- `Message#getMentionedMembers(Guild)` has been removed with no replacement
- `Message#mentionsEveryone` moved to `Mentions#mentionsEveryone`
- Analogously for all bag getters.

Additionally, the return type of `Mentions#getChannels` has been adjusted to return `List<GuildChannel>` since all channel types can now be mentioned. You can use `Mentions#getChannels(Class)` to limit it to specific types, for example `List<TextChannel> channels = mentions.getChannels(TextChannel.class)`.

## Interaction Rework

To properly handle **Context Menu** and **Auto-complete** interactions, we reworked some interaction types. This includes numerous breaking changes to naming conventions and package layouts.

### Naming Changes

- `SlashCommandEvent` renamed to `SlashCommandInteractionEvent`
- `ButtonClickEvent` renamed to `ButtonInteractionEvent`
- `SelectionMenu` renamed to `StringSelectMenu`
- `SelectionMenuEvent` renamed to `StringSelectEvent`
- `Component` renamed to `ActionComponent` and `ItemComponent` (abstraction for things like `Button`)
- `ComponentLayout` renamed to `LayoutComponent` (abstraction for things like `ActionRow`)
- Introduced **new** `Component` interface to abstract both layouts and items.
- `MessageType.APPLICATION_COMMAND` renamed to `MessageType.SLASH_COMMAND` (we now also have `MessageType.CONTEXT_COMMAND`)
- `InteractionType.SLASH_COMMAND` renamed to `InteractionType.COMMAND` (we now also have `InteractionType.COMMAND_AUTOCOMPLETE`)
- `SlashCommandEvent#getCommandPath` renamed to `CommandInteractionPayload#getFullCommandName` (it also now uses spaces instead of slashes, e.g. `mod/ban` is now `mod ban`)

### Creating and Handling Commands

With the introduction of **Context Menu Commands**, we changed how commands are created. Instead of `new CommandData(...)`, you now create a slash command using the factory method `Commands.slash(...)`, which returns a `SlashCommandData` instance that has the familiar methods. You can now also use `Commands.user(...)` and `Commands.message(...)` to create new context menu commands which appear in the right-click menu option named **Apps** in the Discord Client.

Handling commands only changed slightly with regard to the way you reply. Previously, all `Interaction` types had a `reply(...)` or `deferReply(...)` method. This has been changed due to new interaction types like **AutoCompleteInteraction** and **ModalInteraction**. Now, each concrete interaction type implements specific **callback** interfaces such as:

- `IReplyCallback`<br>
    Which supports direct message replies and deferred message replies via `reply(String)` and `deferReply()`
- `IMessageEditCallback`<br>
    Which supports direct message edits and deferred message edits (or no-operation) via `editMessage(String)` and `deferEdit()`
- `IModalCallback`<br>
    Which supports replying using a Modal via `replyModal(Modal)`
- `IAutoCompleteCallback`<br>
    Which supports choice suggestions for auto-complete interactions via `replyChoices(Command.Choice...)`

If you relied on the abstract `Interaction` type to provide any of these methods, you will have to adjust your code to use these new interfaces instead.

You can find more explanations and examples in the dedicated [Interactions Wiki Page](/using-jda/interactions/).

### Command Permissions/Privileges

Discord has changed how command permissions work. Instead of limiting commands to specific roles and users on a per-guild basis, you set required permissions on each command. For instance, a ban command would require `Permission.BAN_MEMBERS`:

```java
commands.addCommands(
    Commands.slash("ban", "Ban a user from this server. Requires permission to ban users.")
        .addOption(USER, "user", "The user to ban", true)
        .addOptions(new OptionData(INTEGER, "del_days", "Delete messages from the past days.")
            .setRequiredRange(0, 7)) // Only allow values between 0 and 7 (inclusive)
        .addOptions(new OptionData(STRING, "reason", "The ban reason to use (default: Banned by <user>)"))
        // This way the command can only be executed from a guild, and not DMs
        .setGuildOnly(true)
        // Only members with the BAN_MEMBERS permission are going to see this command
        .setDefaultPermissions(DefaultMemberPermissions.enabledFor(Permission.BAN_MEMBERS))
)
```

You can also limit it to administrators with `DefaultMemberPermissions.DISABLED`.

## UserSnowflake Type

We have started to cut down on overloads which accept 3 different inputs to specify a user. Previously methods like `Guild#ban` had many overloads to account for different types such as `String`/`long`/`User`/`Member`. We removed all of these in favor of a single `UserSnowflake` input, which can be constructed with `User.fromId(long/String)`. Both `User` and `Member` also implement this interface. This is a far more elegant handling and reduces the number of overloads in various parts of the API drastically.

This applies to the following methods:

- `Guild#ban(long/String)`, `Guild#ban(long/String, int)`, `Guild#ban(long/String, int, String)`<br>
    Now just `Guild#ban(UserSnowflake, int, TimeUnit)`
- `Guild#kick(long/String)`, `Guild#kick(long/String, String)`<br>
    Now just `Guild#kick(UserSnowflake)`
- `Guild#unban(long/String)` is now `Guild#unban(UserSnowflake)`
- `Guild#retrieveBanById(long/String)` is now `Guild#retrieveBan(UserSnowflake)`
- `Guild#addMember(String, long/String)` is now `Guild#addMember(UserSnowflake)`
- `Guild#timeoutForById(long/String, Duration)` is now `Guild#timeoutFor(UserSnowflake, Duration)`
- `Guild#timeoutUntilById(long/String, TemporalAccessor)` is now `Guild#timeoutUntil(UserSnowflake, TemporalAccessor)`
- `Guild#removeTimeoutById(long/String)` is now `Guild#removeTimeout(UserSnowflake)`
- `Guild#addRoleToMember(long/String, Role)` is now `Guild#addRoleToMember(UserSnowflake)`
- `Guild#removeRoleFromMember(long/String, Role)` is now `Guild#removeRoleFromMember(UserSnowflake)`
- `AuditLogPaginationAction#user(long/String)` is now `AuditLogPaginationAction#user(UserSnowflake)`

### Ban Precision

You can now specify the age of messages to delete in seconds precision. To adjust your code simply add `TimeUnit.DAYS` to the end and move the ban reason into the `reason(...)` method:

**Old:**

```java
guild.ban(user, 7, "Naughty words").queue()
```

**New:**

```java
guild.ban(user, 7, TimeUnit.DAYS).reason("Naughty words").queue()
```

Kick reasons have also been moved into the `reason(...)` method, for example `guild.kick(user, reason)` turns into `guild.kick(user).reason(reason)`.

## Managers are no longer persistent

All getters for managers in JDA used to be lazy-idempotent, such that calling `getManager()` twice would return the same instance. This has been removed to reduce memory usage and complexity. If you previously relied on this behavior, you will need to adjust your code.

**Old:**

```java
channel.getManager().setName("new-name");
channel.getManager().setTitle("here is my title");
channel.getManager().queue();
```

**New:**

```java
// Use method chaining or declare a variable to re-use the manager
channel.getManager()
       .setName("new-name")
       .setTitle("here is my title")
       .queue();
```