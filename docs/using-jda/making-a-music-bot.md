# What do I need to get started?

1. Set up your project: 
    - [IntelliJ IDEA](../setup/intellij.md)
    - [Eclipse](../setup/eclipse.md)
    - [Netbeans](../setup/netbeans.md)

2. [Set up JDA](getting-started.md)
3. Once you have your project you will need an additional dependency for your [AudioSendHandler](https://github.com/DV8FromTheWorld/JDA/blob/master/src/main/java/net/dv8tion/jda/api/audio/AudioSendHandler.java)
    - If you don't want to implement it yourself, use [LavaPlayer](#using-lavaplayer)

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

!!! note
    It may be important to do certain permission checks before trying to open an audio connection! It may result in a PermissionException throw otherwise!


### Sending Audio to an Open Audio Connection

!!! note inline end
    For LavaPlayer read [here](#using-lavaplayer)

1. Retrieve the [`AudioManager`](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/entities/Guild.html#getAudioManager()) 
   <br>`AudioManager audioManager = guild.getAudioManager();`
2. Create a **new** [AudioSendHandler](https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/audio/AudioSendHandler.html) instance for your implementation. 
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
    
    @Override
    public void onMessageReceived(MessageReceivedEvent event) 
    {
        // Make sure we only respond to events that occur in a guild
        if (!event.isFromGuild()) return;
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

!!! Important
    This example expects you to have your own AudioSendHandler implementation.  
    It is crucial you only use one AudioSendHandler per Guild!

## Using LavaPlayer

1. [Set up LavaPlayer](https://github.com/sedmelluq/LavaPlayer#readme)
2. Implement an AudioSendHandler 
    - [Example](https://github.com/sedmelluq/LavaPlayer#jda-integration)
3. [Connect to a voice channel](#connecting-to-a-voicechannel)
4. [Register your AudioSendHandler](#sending-audio-to-an-open-audio-connection)
5. Use the LavaPlayer resources: [How To Use LavaPlayer](https://github.com/sedmelluq/LavaPlayer#usage)


### More example implementations can be found in existing bots like:
- AudioEchoExample
    - [Source](https://github.com/DV8FromTheWorld/JDA/blob/development/src/examples/java/AudioEchoExample.java)
- Clarity by [@jagrosh](https://github.com/jagrosh)
    - [GitHub](https://github.com/jagrosh/MusicBot) 
    - [Wiki](https://github.com/jagrosh/MusicBot/wiki) 
- FredBoat by [@freyacodes](https://github.com/freyacodes)
    - [GitHub](https://github.com/freyacodes/archived-bot/)
    - [relevant package](https://github.com/freyacodes/archived-bot/tree/master/FredBoat/src/main/java/fredboat/audio)