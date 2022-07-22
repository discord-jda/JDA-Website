[GatewayIntent]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/requests/GatewayIntent.html
[createDefault]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/JDABuilder.html#createDefault(java.lang.String)
[createLight]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/JDABuilder.html#createLight(java.lang.String)
[create]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/JDABuilder.html#create(java.lang.String,net.dv8tion.jda.api.requests.GatewayIntent,net.dv8tion.jda.api.requests.GatewayIntent...)
[CacheFlag]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/utils/cache/CacheFlag.html
[GatewayIntent.DEFAULT]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/requests/GatewayIntent.html#DEFAULT
[enableCache]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/JDABuilder.html#enableCache(net.dv8tion.jda.api.utils.cache.CacheFlag,net.dv8tion.jda.api.utils.cache.CacheFlag...)
[disableCache]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/JDABuilder.html#disableCache(net.dv8tion.jda.api.utils.cache.CacheFlag,net.dv8tion.jda.api.utils.cache.CacheFlag...)
[enableIntents]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/JDABuilder.html#enableIntents(net.dv8tion.jda.api.requests.GatewayIntent,net.dv8tion.jda.api.requests.GatewayIntent...)
[MemberCachePolicy]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/utils/MemberCachePolicy.html
[setMemberCachePolicy]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/JDABuilder.html#setMemberCachePolicy(net.dv8tion.jda.api.utils.MemberCachePolicy)
[loadMembers]: https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/Guild.html#loadMembers()

# Gateway Intents

In version **4.2.0**, we introduced the [`GatewayIntent`][GatewayIntent] enum. This marks a change in the way bots will work in the future.

Building JDA is done using one of the JDABuilder factory methods, each of which has some default intents:

- [`createDefault`][createDefault]
- [`createLight`][createLight]
- [`create`][create]

### What Intents do I need?

The necessary intents directly correlate with the features you intend to use.
Each [`GatewayIntent`][GatewayIntent] documents which events are enabled by it. Some caches in JDA also depend on these intents, so take a close look at the documentation for [`CacheFlag`][CacheFlag] as well.

For instance, a bot that only responds to messages and sends welcome messages will only need `GUILD_MESSAGES`, `MESSAGE_CONTENT`, and `GUILD_MEMBERS`. A bot like this doesn't rely on any members being cached, so the right solution is to use [`createLight`][createLight] which will disable all [`CacheFlags`][CacheFlag] and member caching.

```java
public static void main(String[] args) {
  // createLight disables unused cache flags
  // GUILD_MESSAGES enables events for messages sent in guilds
  // MESSAGE_CONTENT enables access to the content of messages sent by other users
  // GUILD_MEMBERS gives you access to guild member join events so you can send welcome messages
  // The resulting JDA instance will not cache any members since createLight disables it.
  JDABuilder.createLight(BOT_TOKEN, GatewayIntent.GUILD_MESSAGES, GatewayIntent.MESSAGE_CONTENT, GatewayIntent.GUILD_MEMBERS)
            .addEventListeners(new JoinListener())
            .addEventListeners(new CommandHandler())
            .build();
}
```

Due to `GUILD_MEMBERS` and `MESSAGE_CONTENT` being a **privileged** intents, you must also enable it in your developer dashboard:

1. Open the [application dashboard](https://discord.com/developers/applications)
1. Select your bot application
1. Open the **Bot** tab
1. Under the **Privileged Gateway Intents** section, enable **SERVER MEMBERS INTENT** and **MESSAGE CONTENT INTENT**.

If you use these intents, you are limited to 100 guilds on your bot. To allow the bot to join more guilds while using this intent, you have to [verify your bot](https://blog.discord.com/the-future-of-bots-on-discord-4e6e050ab52e). This will be available in your application dashboard when the bot joins at least 76 guilds.

You can also choose to just use [`createLight`][createLight] or [`createDefault`][createDefault] without specifying the intents you need. In that case, JDA will just use [GatewayIntent.DEFAULT][GatewayIntent.DEFAULT]. If you want to use the default but also include some additional intents like `GUILD_MEMBERS` then you can use [`enableIntents`][enableIntents]:

```java
JDABuilder.createDefault(token) // enable all default intents
          .enableIntents(GatewayIntent.GUILD_MEMBERS) // also enable privileged intent
          .addEventListeners(new JoinListener())
          .addEventListeners(new CommandHandler())
          .build();
```

#### Troubleshooting

- [I'm getting CloseCode(4014 / Disallowed intents...)](troubleshooting.md#im-getting-closecode4014-disallowed-intents)
- [My event listener code is not executed](troubleshooting.md#my-event-listener-code-is-not-executed)
- [Cannot get message content / Attempting to access message content without GatewayIntent](troubleshooting.md#cannot-get-message-content-attempting-to-access-message-content-without-gatewayintent)

## CacheFlags

JDA provides a number of different optional caches you can enable or disable.
Most of these caches are configured using the [`CacheFlag`][CacheFlag] enum.

You can manually enable or disable these caches by using [`enableCache`][enableCache] and [`disableCache`][disableCache] respectively.

Each `createX` factory method on the JDABuilder also configures a set of enabled flags automatically, based on your choice of intents. The [`CacheFlag`][CacheFlag] enum documents which intents are required to use it and JDA will automatically disable them if the required intent is missing.

If a flag is automatically disabled due to a missing intent, we print a warning telling you it was disabled and which intent was missing. To remove this warning, you have to explicitly disable the [`CacheFlag`][CacheFlag] by using [`disableCache`][disableCache] or, if you need the cache, enable the intent using [`enableIntents`][enableIntents].

The individual factory methods document which defaults will be used:

1. [`createDefault`][createDefault]
1. [`createLight`][createLight]
1. [`create`][create]

## MemberCachePolicy

Together with intents, Discord now wants to further restrict data access for bots by limiting how many members they can cache. To properly maintain a cache of all members, you need the `GUILD_MEMBERS` intent, because it will enable the `GuildMemberRemoveEvent` to remove members from cache once they leave the guild. Without this intent, JDA would infinitely grow its cache without knowing when to remove members.

To handle this new default, we now have a [`MemberCachePolicy`][MemberCachePolicy] which can be configured using [`setMemberCachePolicy`][setMemberCachePolicy]. Each factory method will set a default cache policy which will only retain members under certain conditions:

- [`createLight`][createLight] Will only cache the self member
- [`createDefault`][createDefault] Will only cache members who are connected to a voice channel, the guild owner, and the self member
- [`create`][create] Will cache all members it can properly track. See the docs for further details.

We also provide a few reasonable implementations to choose from and apply using [`setMemberCachePolicy`][setMemberCachePolicy]:

- [All](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/utils/MemberCachePolicy.html#ALL)
    Will keep all members cached (requires `GUILD_MEMBERS` intents)
- [Online](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/utils/MemberCachePolicy.html#ONLINE)
    Will keep all online members cached (requires `GUILD_PRESENCES` intent)
- [Voice](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/utils/MemberCachePolicy.html#VOICE)
    Will keep all voice members cached (requires `GUILD_VOICE_STATES` intent)
- [Owner](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/utils/MemberCachePolicy.html#OWNER)
    Will keep the guild owner cached
- [Pending](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/utils/MemberCachePolicy.html#PENDING)
    Will cache the members which have not passed membership screening yet (requires `GUILD_MEMBERS` intents)
- [None](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/utils/MemberCachePolicy.html#NONE)
    Will only keep the self member cached and nobody else

It is important to understand the difference between *cache* and *load* in this system.

```java
JDABuilder.createDefault(token)
          .enableIntents(GatewayIntent.GUILD_MEMBERS)
          .setMemberCachePolicy(MemberCachePolicy.ALL)
          .build();
```

The difference becomes clear when you try to access the member list using this configuration. The `MemberCachePolicy.ALL` will specifically cache **all** members once they are **loaded**. However, we are lazy loading members and start off with only a small subset of all members in cache.
This cache will grow over time by loading members when they are active in the guilds, such as sending a message or connecting to a voice channel.

## Loading Members

We offer a number of ways to load and cache members:

- [Guild.retrieveMembersByPrefix(String, int)](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/Guild.html#retrieveMembersByPrefix(java.lang.String,int))
- [Guild.retrieveOwner()](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/Guild.html#retrieveOwner())
- [Guild.retrieveMemberById(long|String)](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/Guild.html#retrieveMemberById(long))
- [Guild.retrieveMembersByIds(long...|String..)](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/Guild.html#retrieveMembersByIds(long...))
- [Guild.retrieveMembers(Collection&lt;User>)](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/Guild.html#retrieveMembers(java.util.Collection))
- [Message.getMember()](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/entities/Message.html#getMember()) (from events)

All of these methods will load the members from cache or fallback to requesting them from the Discord API.

You can also load the entire member list at runtime by using [`loadMembers`][loadMembers], however this requires the privileged `GUILD_MEMBERS` intent. This process is called *guild member chunking* (aka chunking).

Chunking can also be performed for many guilds at startup automatically, by using `setChunkingFilter` on the JDABuilder.  This also requires the `GUILD_MEMBERS` intent
