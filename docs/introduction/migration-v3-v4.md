# Migration Guide 3.X to 4.X

This version includes a number of breaking changes.

## Extensions That Support 4.X

Here is a list of known extensions that support 4.X.
The ones that are not checked do not support it yet. You should check that everything used in your project supports 4.X before starting migration.

- [x] [jda-reactor](https://github.com/MinnDevelopment/jda-reactor)
- [x] [LavaPlayer](https://github.com/sedmelluq/lavaplayer)
- [x] [jda-nas](https://github.com/sedmelluq/jda-nas) (since [1.1.0](https://bintray.com/sedmelluq/com.sedmelluq/jda-nas/1.1.0))
- [x] [JDA-Utilities](https://github.com/JDA-Applications/JDA-Utilities) (since [3.0.1](https://bintray.com/jagrosh/maven/JDA-Utilities/3.0.1))
- [x] [Lavalink Client](https://github.com/FredBoat/Lavalink-Client) (since [4.0](https://github.com/FredBoat/Lavalink-Client/tree/4.0))
- [ ] [JDAction](https://github.com/sedmelluq/jdaction) (see [#6](https://github.com/sedmelluq/jdaction/pull/6))

## Package layout changes

Since we removed the client-only features from JDA the package didn't make much sense anymore.
Instead we have merged `core`, `bot`, and `client` into `api`. The new package prefix for the API functionality is now `net.dv8tion.jda.api`.

If you used internal classes they will not be available under this prefix but instead use `net.dv8tion.jda.internal`.
Something like `net.dv8tion.jda.core.entities.impl` has been moved to `net.dv8tion.jda.internal.entities` since the internal packages only contain implementations anyway.

In the case that `PermissionUtil.checkPermissions` was used in your code, this should be changed to `member.hasPermission(...)` or `role.hasPermission(...)`.

## Removed Features

Almost everything from `net.dv8tion.jda.client` has been removed as its not supported by the API and can lead to account termination which we don't want to happen. Additionally, most of the features advertised by the interfaces in `client` were not even implemented or documented properly.
Everything on `JDABot` has been moved into `JDA`.

The flag to disable the audio system with `setAudioEnabled(false)` has been removed. This flag only enabled an exception for `Guild.getAudioManager()` which is unnecessary.

The entire `net.dv8tion.jda.webhook` package has been moved to a dedicated library at <https://github.com/MinnDevelopment/discord-webhooks>

The methods `buildBlocking` and `buildAsync` have been removed as they were already deprecated. To replace `buildBlocking` you should use `awaitReady()` on the `JDA` instance returned by `build()`.

The overloads for `sendFile` that accepted a `Message` as their last parameter have been removed in favor of `MessageAction.addFile`. For example `sendFile(file, message)` becomes `sendMessage(message).addFile(file)`.

Both `getWebSocketTrace` and `getCloudflareRays` have been removed without replacements. These are not officially supported by the Discord API and subject to change.

## Renamed Classes

Some classes have been renamed to better represent their meaning.

- `Channel` -> `GuildChannel`
- `Game` -> `Activity` (also `CacheFlag.GAME` -> `CacheFlag.ACTIVITY`)
- `GuildMemberNickChangeEvent` -> `GuildMemberUpdateNicknameEvent`

## Renamed Methods

A few methods were renamed for better consistency and clarity.

- `JDABuilder.addEventListener` -> `JDABuilder.addEventListeners`
- `JDABuilder.setGame` -> `JDABuilder.setActivity` (same for `Presence` and `DefaultShardManagerBuilder`)
- `Invite.getURL` -> `Invite.getUrl`
- `JDA.getPing` -> `JDA.getGatewayPing`
- `JDA.getWebhookById` -> `JDA.retrieveWebhookById`
- `Guild.getWebhooks` -> `Guild.retrieveWebhooks` (same for `TextChannel`)
- `Guild.getInvites` -> `Guild.retrieveInvites` (same for `GuildChannel`)
- `Guild.getAuditLogs` -> `Guild.retrieveAuditLogs`
- `Guild.getBan[ById|List]` -> `Guild.retrieveBan[ById|List]`
- `Guild.getVanityUrl` -> `Guild.retrieveVanityUrl`
- `MessageChannel.getMessageById` -> `MessageChannel.retrieveMessageById`
- `JDA.getApplicationInfo` -> `JDA.retrieveApplicationInfo`
- `AudioManager.getReceiveHandler` -> `AudioManager.getReceivingHandler`
- `GuildController.addSingleRoleToMember` -> `Guild.addRoleToMember` (same for `removeSingleRoleFromMember`)
- `GuildController.setNickname` -> `Guild.modifyNickname`
- `GuildMemberNickChangeEvent.getPrevNick` -> `GuildMemberUpdateNicknameEvent.getOldNickname`
- `GuildMemberNickChangeEvent.getNewNick` -> `GuildMemberUpdateNicknameEvent.getNewNickname`
- `ISnowflake.getCreationTime` -> `ISnowflake.getTimeCreated`

## Changed Functionality

Some methods have changed their signature in a breaking way. This is supposed to improve either internal code or make the interface more versatile.

### JDABuilder and DefaultShardManagerBuilder

As of **4.2.0** the JDABuilder constructor has been deprecated in favor of 3 factory methods:

[createDefault]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/JDABuilder.html#createDefault(java.lang.String)
[createLight]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/JDABuilder.html#createLight(java.lang.String)
[create]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/JDABuilder.html#create(java.lang.String,net.dv8tion.jda.api.requests.GatewayIntent,net.dv8tion.jda.api.requests.GatewayIntent...)

- [`createDefault`][createDefault]
- [`createLight`][createLight]
- [`create`][create]

Further details are explained in a dedicated wiki page here: [Gateway Intents and Member Cache Policy](../using-jda/gateway-intents-and-member-cache-policy.md)

### Permission Lists

Everything that once returned `List<Permission>` has been changed to return `EnumSet<Permission>` instead for a better representation.

### Message Attachments

You can no longer use blocking operations to download attachments of a message. Instead we replaced the old `download()` with `downloadToFile()` and similar methods. These use `CompletableFuture` to provide continuations such as `thenAccept` and `whenComplete`. Examples are provided in the documentation for the respective methods.

### Audio System

`AudioSendHandler.provide20MsAudio()` now returns `ByteBuffer` instead of `byte[]`. An easy transition is to use `ByteBuffer.wrap(byte[])`. This was done to allow re-using the same buffer for multiple packets instead of allocating a fitting array every time.

### PermissionOverride

`PermissionOverride.getManager()` now returns a `PermissionOverrideAction` instead to remove code duplication. You can now also use `GuildChannel.upsertPermissionOverride` which will either return `getManager()` on an existing override or use `putPermissionOverride()` if there is no existing override.

### Events

`IEventManager.handle`, `EventListener.onEvent`, and `ListenerAdapter.onGenericEvent` now accepts `GenericEvent` which is a new interface that allows a more versatile hierarchy for the events it can handle. `UpdateEvent` is now a subinterface for `GenericEvent`.

### GuildController

The methods provided by the old `GuildController` class have been moved into the `Guild` interface. The `GuildController` has been removed completely. Additionally, `Member` received a few new shortcuts like `Member.ban(int)`.

The methods `addRolesToMember` and `removeRolesFromMember` have been removed and should be replaced with `modifyMemberRoles`.

### Messages

Since we added `@Nonnull` and `@Nullable` annotations we had to compromise on conditional nullability. This means the methods like `Message.getTextChannel` now **throw** instead of returning `null` if the message was not sent in a `TextChannel`. You can use `Message.isFromType` to test for the channel type.

### Compression

To support other compression algorithms like `zstd` that Discord may offer in the future we changed `setCompressionEnabled` to `setCompression`. In this change we also replaced the `boolean` with a new enum for each algorithm. The new way to disable compression is `setCompression(Compression.NONE)`.

### RestAction

All `RestAction` types have been changed to be interfaces to improve maintainability. Additionally, `submitAfter` now returns `DelayedCompletableFuture` to allow usage of continuations.