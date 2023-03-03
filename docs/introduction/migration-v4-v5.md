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

You must enable this intent in your `JDABuilder` **AND** in the Discord Developer Portal for your app if you utilize any of the above methods.

## Channel Rework

There are several breaking changes to `GuildChannel` and `ChannelManager`

### New Channel Attribute Interfaces

`GuildChannel` now implements `getGuild` and `getManager` *only*. It's meant to serve as a generic type to hold channels from guilds.

Specific channel attributes, such as slow mode, permissions, etc have been split into several interfaces. These interfaces can be found in [the `net.dv8tion.jda.api.entities.channel.attribute` package](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/entities/channel/attribute/package-summary.html). Each interface in this package extends `GuildChannel`.

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
