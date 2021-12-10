# What do I need to get started?

1. [[Setup your project|2)-Setup]]
2. [[Setup JDA|3)-Getting-Started]]
3. Once you have your project you will need an additional dependency for your [AudioSendHandler](/DV8FromTheWorld/JDA/tree/master/src/main/java/net/dv8tion/jda/core/audio/AudioSendHandler.java)
    - If you don't want to implement it yourself use [LavaPlayer](#using-lavaplayer)

### Connecting to a VoiceChannel

1. Getting a VoiceChannel (`guild` references an instance of `Guild`)
    - By the channel id: [`guild.getVoiceChannelById(CHANNEL_ID)`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/entities/Guild.html#getVoiceChannelById(long))
    <br>`VoiceChannel myChannel = guild.getVoiceChannelById(CHANNEL_ID);`
    - By the channel name: [`guild.getVoiceChannelsByName(CHANNEL_NAME, true)`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/entities/Guild.html#getVoiceChannelsByName(java.lang.String,boolean))
    <br>`VoiceChannel myChannel = guild.getVoiceChannelsByName(CHANNEL_NAME, true).get(0);`
    - By the voice state of a member [`member.getVoiceState().getChannel()`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/entities/GuildVoiceState.html#getChannel())
    <br>`VoiceChannel myChannel = member.getVoiceState().getChannel();`
2. Retrieve the [`AudioManager`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/entities/Guild.html#getAudioManager()) 
    <br>`AudioManager audioManager = guild.getAudioManager();`
3. Open an audio connection [`audioManager.openAudioConnection()`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/managers/AudioManager.html#openAudioConnection(net.dv8tion.jda.api.entities.VoiceChannel)) 
    <br>`audioManager.openAudioConnection(myChannel);`

> **Note**: It may be important to do certain permission checks before trying to open an audio connection! It may result in a PermissionException throw otherwise!

### Sending Audio to an Open Audio Connection

1. Retrieve the [`AudioManager`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/entities/Guild.html#getAudioManager()) 
   <br>`AudioManager audioManager = guild.getAudioManager();`
2. Create a **new** [AudioSendHandler](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/audio/AudioSendHandler.html) instance for your implementation. 
   (Note: For LavaPlayer read [[here|4)-Making-a-Music-Bot#using-lavaplayer]])
3. Register your AudioSendHandler: 
  [`audioManager.setSendingHandler(myAudioSendHandler)`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/managers/AudioManager.html#setSendingHandler(net.dv8tion.jda.api.audio.AudioSendHandler))
    <br>You may only use __one__ AudioSendHandler per Guild and not use the same instance on another Guild! 
    <br>Doing that will result in speedup due to multiple send threads pulling from the same instance!

### A Working Example

```java
public class MusicBot extends ListenerAdapter 
{
    public static void main(String[] args)
    throws IllegalArgumentException, LoginException, RateLimitedException
    {
        JDABuilder.createDefault(args[0]) // Use token provided as JVM argument
            .addEventListeners(new MusicBot()) // Register new MusicBot instance as EventListener
            .build(); // Build JDA - connect to discord
    }

    // Note that we are using GuildMessageReceivedEvent to only include messages from a Guild!
    @Override
    public void onGuildMessageReceived(GuildMessageReceivedEvent event) 
    {
        // This makes sure we only execute our code when someone sends a message with "!play"
        if (!event.getMessage().getContentRaw().startsWith("!play")) return;
        // Now we want to exclude messages from bots since we want to avoid command loops in chat!
        // this will include own messages as well for bot accounts
        // if this is not a bot make sure to check if this message is sent by yourself!
        if (event.getAuthor().isBot()) return;
        Guild guild = event.getGuild();
        // This will get the first voice channel with the name "music"
        // matching by voiceChannel.getName().equalsIgnoreCase("music")
        VoiceChannel channel = guild.getVoiceChannelsByName("music", true).get(0);
        AudioManager manager = guild.getAudioManager();

        // MySendHandler should be your AudioSendHandler implementation
        manager.setSendingHandler(new MySendHandler());
        // Here we finally connect to the target voice channel 
        // and it will automatically start pulling the audio from the MySendHandler instance
        manager.openAudioConnection(channel);
    }
}
```

> Note: This example expects you to have your own AudioSendHandler implementation<br>
> It is crucial you only use one AudioSendHandler per Guild!

## Using LavaPlayer

1. Setup LavaPlayer [guide](/sedmelluq/LavaPlayer#lavaplayer---audio-player-library-for-discord)
2. Implement an AudioSendHandler -> [Example](/sedmelluq/LavaPlayer#jda-integration)
3. Connect to a voice channel [[how?|4)-Making-a-Music-Bot#Connecting-to-a-VoiceChannel]]
4. Register your AudioSendHandler [[how?|4)-Making-a-Music-Bot#Sending-Audio-to-an-Open-Audio-Connection]]
5. Use the LavaPlayer resources [How To Use LavaPlayer](/sedmelluq/LavaPlayer#usage)


### More example implementations can be found in existing bots like:
- AudioEchoExample [Source](https://github.com/DV8FromTheWorld/JDA/blob/development/src/examples/java/AudioEchoExample.java)
- Clarity by @jagrosh [GitHub](/jagrosh/MusicBot) [Wiki](/jagrosh/MusicBot/wiki) 
- FredBoat by @Frederikam [GitHub](/Frederikam/FredBoat) -> [relevant package](/Frederikam/FredBoat/tree/master/FredBoat/src/main/java/fredboat/audio)