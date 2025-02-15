

# Module jda-ktx

Collection of useful Kotlin extensions for JDA.
Great in combination with `kotlinx-coroutines`.

For more details, checkout the [README](https://github.com/MinnDevelopment/jda-ktx/blob/master/README.md).

## Examples

You can look at my own bot ([strumbot](https://github.com/MinnDevelopment/strumbot)) for inspiration, or look at the examples listed here.

The most useful feature of this library is the `CoroutineEventManager` which adds the ability to use suspending functions in your event handlers.

```kotlin
// enableCoroutines (default true) changes the event manager to CoroutineEventManager
// this event manager uses a default scope generated by getDefaultScope() 
// but can be configured to use a custom scope if you set it manually
val jda = light("token", enableCoroutines=true) {
    intents += listOf(GatewayIntent.GUILD_MEMBERS, GatewayIntent.MESSAGE_CONTENT)
}

// This can only be used with the CoroutineEventManager
jda.listener<MessageReceivedEvent> {
    val guild = it.guild
    val channel = it.channel
    val message = it.message
    val content = message.contentRaw

    if (content.startsWith("!profile")) {
        // Send typing indicator and wait for it to arrive
        channel.sendTyping().await()
        val user = message.mentionedUsers.firstOrNull() ?: run {
            // Try loading user through prefix loading
            val matches = guild.retrieveMembersByPrefix(content.substringAfter("!profile "), 1).await()
            // Take first result, or null
            matches.firstOrNull()
        }

        if (user == null) // unknown user for name
            channel.send("${it.author.asMention}, I cannot find a user for your query!").queue()
        else // load profile and send it as embed
            channel.send("${it.author.asMention}, here is the user profile:", embeds=profile(user).into()).queue()
    }
}

jda.onCommand("ban", timeout=2.minutes) { event -> // 2 minute timeout listener
    val user = event.getOption<User>("user")!!
    val confirm = danger("${user.id}:ban", "Confirm")
    event.reply_(
        "Are you sure you want to ban **${user.asTag}**?",
        components=confirm.into(),
        ephemeral=true
    ).queue()

    withTimeoutOrNull(1.minutes) { // 1 minute scoped timeout
        val pressed = event.user.awaitButton(confirm) // await for user to click button
        pressed.deferEdit().queue() // Acknowledge the button press
        event.guild.ban(user, 0).queue() // the button is pressed -> execute action
    } ?: event.hook.editMessage(/*id="@original" is default */content="Timed out.", components=emptyList()).queue()
}

jda.onButton("hello") { // Button that says hello
    it.reply_("Hello :)").queue()
}
```


## Download

### Gradle

```gradle
repositories {
    mavenCentral()
    maven("https://jitpack.io/")
}

dependencies {
    implementation("net.dv8tion:JDA:${JDA_VERSION}")
    implementation("com.github.minndevelopment:jda-ktx:${COMMIT}")
}
```

### Maven

```xml
<repository>
    <id>jitpack</id>
    <name>jitpack</name>
    <url>https://jitpack.io/</url>
</repository>
```

```xml
<dependency>
  <groupId>net.dv8tion</groupId>
  <artifactId>JDA</artifactId>
  <version>$JDA_VERSION</version>
</dependency>
<dependency>
  <groupId>com.github.minndevelopment</groupId>
  <artifactId>jda-ktx</artifactId>
  <version>$COMMIT</version>
</dependency>
```


# Package dev.minn.jda.ktx.coroutines

Implements extension functions to easily integrate [RestAction](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/requests/RestAction.html), [Task](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/utils/concurrent/Task.html), and [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) into coroutine scopes by adding `await()` for each.

This also adds any other useful extensions like `Flow` integrations.

# Package dev.minn.jda.ktx.events

Adds the `CoroutineEventManager` and coroutine-based event listeners which can be used via the `listener` extension function on any JDA instance.

## Example

```kotlin
val jda = light(token, enableCoroutines=true)

jda.listener<MessageReceivedEvent> { event ->
    if (event.message.contentRaw.startsWith("!hello")) {
        event.channel.sendTyping().await() // <- suspending function
        event.channel.send("Hello!").queue()
    }
}
```

# Package dev.minn.jda.ktx.generics

Adds inline functions to support rectified generics.

# Package dev.minn.jda.ktx.interactions

Extensions specifically designed to make use of [Interactions](https://jda.wiki/using-jda/interactions/) easier with idiomatic builders.

# Package dev.minn.jda.ktx.interactions.commands

Builders and utilities for application commands.

# Package dev.minn.jda.ktx.interactions.components

Builders and utilities for message and modal components.

# Package dev.minn.jda.ktx.jdabuilder

Adds `JDABuilder` and `DefaultShardManagerBuilder` extensions to quickly get going with coroutines and jda-ktx.

# Package dev.minn.jda.ktx.logback

Add logging capabilities using webhooks.

This requires [discord-webhooks](https://github.com/MinnDevelopment/discord-webhooks) and [logback](https://logback.qos.ch/).

### Example

```xml
<appender name="WEBHOOK" class="dev.minn.jda.ktx.logback.WebhookAppender" >
    <!-- (OPTIONAL) Minimum logging level for webhook messages. Prevents spam of debug messages -->
    <level>warn</level>
    <!-- (OPTIONAL) Timeout for messages in milliseconds. Prevents spam in case of disconnects -->
    <timeout>10000</timeout>
    <!-- (REQUIRED) Webhook URL -->
    <url>https://discord.com/api/webhooks/:id/:token</url>
    <!-- (REQUIRED) Logging pattern encoder -->
    <encoder>
        <pattern>%boldWhite(%d{HH:mm:ss.SSS}) %boldCyan(%thread) %boldGreen(%logger{0}) %highlight(%level)\n%msg%n</pattern>
    </encoder>
</appender>
```

# Package dev.minn.jda.ktx.messages

Some extensions to handle message sending and editing with named parameters and custom defaults.

# Package dev.minn.jda.ktx.util

Miscellaneous utility extensions.