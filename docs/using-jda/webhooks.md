# Webhook Support is moving to [discord-webhooks](https://github.com/MinnDevelopment/discord-webhooks)

# Webhooks

In this section we break down how JDA allows to look at and send to webhooks.

[Wikipedia](https://en.wikipedia.org/wiki/Webhook) 
<br>[Discord API Documentation](https://discordapp.com/developers/docs/resources/webhook)

In the Discord client webhooks are used to send messages and files to specific channels using a webhook endpoint.

## Using Webhooks

Webhooks do not require a Discord account and thus do not have to be used with a running JDA instance.
<br>JDA allows to get webhook objects from a TextChannel and from a Guild:
<br>[`guild.getWebhooks(): RestAction<List<Webhook>>`](http://home.dv8tion.net:8080/job/JDA/javadoc/net/dv8tion/jda/core/entities/TextChannel.html#getWebhooks--),
[`channel.getWebhooks(): RestAction<List<Webhook>>`](http://home.dv8tion.net:8080/job/JDA/javadoc/net/dv8tion/jda/core/entities/Guild.html#getWebhooks--)

### Sending to Webhooks

We provide a class called [`WebhookClient`](http://home.dv8tion.net:8080/job/JDA/javadoc/net/dv8tion/jda/webhook/WebhookClient.html)
which allows to send messages to a single Webhook and automatically handles rate limits.
<br>Similar to the JDA setup you can use a [`WebhookClientBuilder`](http://home.dv8tion.net:8080/job/JDA/javadoc/net/dv8tion/jda/webhook/WebhookClientBuilder.html) to build a WebhookClient.
These builders require either a `Webhook` instance or the id and token for a Webhook. We also allow getting a WebhookClientBuilder via [`webhook.newClient(): WebhookClientBuilder`](http://home.dv8tion.net:8080/job/JDA/javadoc/net/dv8tion/jda/core/entities/Webhook.html#newClient--).

**_Remember that these clients have an internal pool that can be changed from the WebhookClientBuilder and should be closed with WebhookClient.close()_**

#### Example: Getting a Client
```java
Webhook webhook; // some webhook instance
WebhookClientBuilder builder = webhook.newClient();
WebhookClient client = builder.build(); //remember to close this client when you are done
```

To send to this Webhook you can either use the direct send methods on the client via a String or you can create a [`WebhookMessage`](http://home.dv8tion.net:8080/job/JDA/javadoc/net/dv8tion/jda/webhook/WebhookMessage.html)
which allows to set a **username** and **avatar** for each message!
<br>WebhookMessage instances also allow to send up to 10 embeds in one message allowing to send more information in each message without hitting rate limits for sending.

Creating WebhookMessages is also done through a builder via [`WebhookMessageBuilder`](http://home.dv8tion.net:8080/job/JDA/javadoc/net/dv8tion/jda/webhook/WebhookMessageBuilder.html).
<br>In addition you can send simple [`Message`](http://home.dv8tion.net:8080/job/JDA/javadoc/net/dv8tion/jda/core/entities/Message.html) instances or extend them by providing them to the WebhookMessageBuilder constructor.

#### Example: Sending
```java
WebhookMessageBuilder builder = new WebhookMessageBuilder();
builder.setContent("This is a normal message content");
MessageEmbed firstEmbed = new EmbedBuilder().setColor(Color.RED).setDescription("This is one embed").build();
MessageEmbed secondEmbed = new EmbedBuilder().setColor(Color.GREEN).setDescription("This is another embed").build();
builder.addEmbeds(firstEmbed, secondEmbed)
       .setUsername("Minn");
WebhookMessage message = builder.build();
client.send(message);
```

After you are done using the client it is recommended to close it by using:
```Java
client.close();
```

## Using WebhookCluster
To send to multiple Webhooks at once you can create a [`WebhookCluster`](http://home.dv8tion.net:8080/job/JDA/javadoc/net/dv8tion/jda/webhook/WebhookCluster.html) and add or build Webhooks for it.
<br>The WebhookCluster can be used as central control unit for all Webhooks and allows setting default values for creating WebhookClients:
```java
WebhookCluster cluster = new WebhookCluster();
cluster.setDefaultDaemon(true); // make all builders daemon
cluster.buildWebhook(webhookId, webhookToken); // automatically adds the built webhook
cluster.addWebhooks(webhook);
```
> [What's daemon?](https://stackoverflow.com/questions/2213340/what-is-daemon-thread-in-java)

Due to the WebhookCluster being a central control unit it allows both **broadcasting** and **multicasting** messages to registered Webhooks.
<br>Broadcasting will send a message to every Webhook and Multicasting will send a message to a filtered selection of Webhooks.
#### Example: Broadcast & Multicast
```java
cluster.broadcast("PSA: JDA is pretty powerful");
WebhookMessage message = new WebhookMessageBuilder().setContent("This is only for you: I love you <3").build();
cluster.multicast((client) -> client.getIdLong() == 351016865780334613L, message);
```