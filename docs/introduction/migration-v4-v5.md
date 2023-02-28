# Migration Guide 4.X to 5.X

This version includes several breaking changes and improvements.

## Extensions That Support 5.X

Here is a list of known extensions that support 5.X.
The ones that are not checked do not support it yet. You should check that everything used in your project supports 5.X before starting migration.

- [ ] [jda-reactor](https://github.com/MinnDevelopment/jda-reactor)
- [ ] [LavaPlayer](https://github.com/sedmelluq/lavaplayer)
- [ ] [jda-nas](https://github.com/sedmelluq/jda-nas)
    - [ ] [udpqueue.rs](https://github.com/MinnDevelopment/udpqueue.rs) (for minimal Rust bindings)
- [ ] [JDA-Utilities](https://github.com/JDA-Applications/JDA-Utilities)

## Channel Rework

There are several breaking changes to `GuildChannel` and `ChannelManager`

### New Channel Attribute Interfaces

`GuildChannel` now implements `getGuild` and `getManager` *only*. It's meant to serve as a generic type to hold channels from guilds.

Specific channel attributes, such as slow mode, permissions, etc have been split into several interfaces. These interfaces can be found in [the `net.dv8tion.jda.api.entities.channel.attribute` package](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/entities/channel/attribute/package-summary.html). Each interface in this package extends `GuildChannel`.

### Independant Stage Channel and News Channel Entities

[`StageChannel`](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/entities/channel/concrete/StageChannel.html) and [`NewsChannel`](https://javadoc.io/doc/net.dv8tion/JDA/latest/net/dv8tion/jda/api/entities/channel/concrete/NewsChannel.html) were previously variants of `VoiceChannel` and `TextChannel`. These are now their own independant entities. This comes with a number of changes:

- `ChannelAction#setNews` has been removed in favor of `ChannelAction.setType`
- `ChannelManager#getType` was removed in favor of `ChannelManager#getChannel()#getType`
- `ChannelManager#setNews` has been removed in favor of `ChannelManager#setType`
- `TextChannel#crosspostMessage` was removed in favor of `NewsChannel#crosspostMessage`
- `TextChannel#follow` has been removed in favor of `NewsChannel#follow`
- `TextChannel#isNews` has been removed
