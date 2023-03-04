# Migration Guide 4.X to 5.X

This version utilizes Discord API v10, and includes several breaking changes and improvements.

## Extensions That Support 5.X

Here is a list of known extensions that support 5.X.
The ones that are not checked do not support it yet. You should check that everything used in your project supports 5.X before starting migration.

- [x] [jda-reactor](https://github.com/MinnDevelopment/jda-reactor)
- [x] [LavaPlayer](https://github.com/sedmelluq/lavaplayer)
- [x] [jda-nas](https://github.com/sedmelluq/jda-nas)
    - [x] [udpqueue.rs](https://github.com/MinnDevelopment/udpqueue.rs) (for minimal Rust bindings)
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

### New Channel Attribute Interfaces

`GuildChannel` now implements `getGuild` and `getManager` *only*. It's meant to serve as a generic type to hold channels from guilds.

Specific channel attributes, such as slowmode and permissions, have been split into several interfaces. These interfaces can be found in [the `net.dv8tion.jda.api.entities.channel.attribute` package](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/entities/channel/attribute/package-summary.html). Each interface in this package extends `GuildChannel`.

### Permission Access Changes

With JDA v5 introducing a more butchered down `GuildChannel` interface, `GuildChannel#getPermissionContainer() : IPermissionContainer` has been introduced to make accessing permissions easier. This allows for users to confidently access the entity that dictates the permissions for a given channel without having to check whether the channel actually supports permissions, such as in the case of threads.

Of course, if you have a channel type that already supports `IPermissionContainer`, you don't need this getter.

### Type-Trimmed Channel Managers

In JDA v4, `GuildChannel#getManager` returned a `ChannelManager` that gave every possible setter for every channel type. This caused for `UnsupportedOperationException` or `IllegalStateException` to be thrown in some cases, such as calling `setBitrate` on a `TextChannel`.

JDA v5 provides type-trimmed channel managers, which provide only the setters that we know *for sure* can work on the given channel. Each of these managers can be found in [the `net.dv8tion.jda.api.managers.channel` package](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/managers/channel/package-summary.html). These all follow the same implement/extension hierarchy as the channels do and map 1:1.

### Independant Stage Channel and News Channel Entities

[`StageChannel`](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/entities/channel/concrete/StageChannel.html) and [`NewsChannel`](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/entities/channel/concrete/NewsChannel.html) were previously variants of `VoiceChannel` and `TextChannel`. These are now their own independant entities. This comes with a number of changes:

- `ChannelAction#setNews` has been removed in favor of `ChannelAction.setType`
- `ChannelManager#getType` was removed in favor of `ChannelManager#getChannel()#getType`
- `ChannelManager#setNews` has been removed in favor of `ChannelManager#setType`
- `TextChannel#crosspostMessage` was removed in favor of `NewsChannel#crosspostMessage`
- `TextChannel#follow` has been removed in favor of `NewsChannel#follow`
- `TextChannel#isNews` has been removed

### Other Changes

- `StoreChannel` has been removed
- `PrivateChannel#getUser` is now nullable
- `MessageChannel#getLatestMessageId` and `MessageChannel#getLatestMessageIdLong` no longer change to null if the message was deleted

## Permission Changes

A number of permissions have been renamed, with one permission being removed entirely.

- `Permission.MANAGE_EMOTES` was renamed to `Permission.MANAGE_EMOTES_AND_STICKERS`
- `Permission.MESSAGE_READ` has been removed in favor of `Permission.VIEW_CHANNEL`
- `Permission.MESSAGE_WRITE` was renamed to `Permission.MESSAGE_SEND`
- `Permission.USE_SLASH_COMMANDS` was renamed to `Permission.USE_APPLICATION_COMMANDS`
- `Permission.USE_PUBLIC_THREADS` was renamed to `Permission.CREATE_PUBLIC_THREADS`
- `Permission.USE_PRIVATE_THREADS` was renamed to `Permission.CREATE_PRIVATE_THREADS`

## Event Changes

There are several changes related to events.

### Removal of Guild Context Message Events

All message event types for guild messages have been removed. Examples of these are:

- Generic<Context>MessageEvent (ex: GenericGuildMessageEvent)
- <Context>MessageReceivedEvent (ex: GuildMessageReceivedEvent)
- <Context>MessageUpdateEvent (ex: GuildMessageUpdateEvent)
- <Context>MessageDeleteEvent (ex: GuildMessageDeleteEvent)
- <Context>MessageEmbedEvent (ex: GuildMessageEmbedEvent)
- <Context>MessageReaction<X>Event (ex: GuildMessageReactionAddEvent)
- etc

Instead, you should use the unified versions of these events:

- GenericMessageEvent
- MessageReceivedEvent
- MessageUpdatedEvent
- MessageDeletedEvent
- MessageReaction<X>Event (ex: MessageReactionAddEvent)
- etc

You can check if a message is from a `Guild` by using `Message#isFromGuild()`.

### Changes to Specifying GatewayIntents from Event Types

With context-specific events being removed (such as `GuildMessageReceivedEvent`), you can no longer use these events to specify `GatewayIntent.GUILD_MESSAGES` or `GatewayIntent.DIRECT_MESSAGE_REACTIONS`. You may supply any of the unified message events, but the functionality will be slightly different. It is not recommended to use `GatewayIntent#fromEvents` if you wish to have finer grain control around intents for messages.

### Session Events

All events that update the gateway session of a bot now extend a common [`GenericSessionEvent`](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/events/session/GenericSessionEvent.html). Such events are located within [the `net.dv8tion.jda.api.events.session` pakage](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/events/session/package-summary.html). This also includes `ReadyEvent` and `ShutdownEvent`.

Some events relating to sessions have been renamed:

- `DisconnectEvent` has been renamed to `SessionDisconnectEvent`
- `ReconnectedEvent` has been renamed to `SessionRecreateEvent`
- `ResumedEvent` has been renamed to `SessionResumeEvent`

### Voice State Events

`GuildVoiceJoinEvent` and `GuildVoiceLeaveEvent` have both been removed in favor of the unified [`GuildVoiceUpdateEvent`](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/events/guild/voice/GuildVoiceUpdateEvent.html).

As an example, to detect when a user leaves a voice channel, you can use `GuildVoiceUpdateEvent#getChannelLeft`. This method will return the channel that the user left, or `null` if the user joined a channel instead.
