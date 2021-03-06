# JAICF dialog API for Aimybox SDK

This module adds [JAICF](https://github.com/just-ai/jaicf-kotlin) support to Aimybox SDK and enables to embed dialogue scenarios into the voice application.
Thus you don't need to host JAICF scenarios anywhere in the cloud.

## Example

Here is a [working example](https://github.com/just-ai/jaicf-android-template) of Android voice assistant powered by this module.

## How to start using

1. Add dependencies to your module's build.gradle:
```kotlin
repositories {
    jcenter()
}

dependencies {
    implementation("com.justai.aimybox:core:${version}")
    implementation("com.justai.aimybox:google-platform-speechkit:${version}")
    implementation("com.justai.aimybox:jaicf-api:${version}")
    // Add other required JAICF modules with the latest versions here
}
```

2. Create dialogue scenarios

Create dialogue scenarios using [JAICF scenario DSL](https://github.com/just-ai/jaicf-kotlin/wiki/Scenario-DSL):

```kotlin
object MainScenario: Scenario() {
    init {
        state("hello") {
            activators {
                regex("hello")
            }

            action {
                reactions.sayRandom("Hello!", "Hello there!")
            }
        }
    }
}
```

3. Provide JAICF dialog API into Aimybox configuration object:

```kotlin
private fun createAimybox(context: Context): Aimybox {
    val unitId = UUID.randomUUID().toString()

    val textToSpeech = GooglePlatformTextToSpeech(context, Locale.ENGLISH)
    val speechToText = GooglePlatformSpeechToText(context, Locale.ENGLISH)
    val dialogApi = JAICFDialogApi(unitId, MainScenario.model)

    return Aimybox(Config.create(speechToText, textToSpeech, dialogApi))
}
```

### Use other JAICF libraries

JAICF provides a wide set of ready to use libraries of [NLU engines](https://github.com/just-ai/jaicf-kotlin/tree/master/activators) or [databases](https://github.com/just-ai/jaicf-kotlin/tree/master/managers).
You can refer and pick the most appropriate for your needs (supported languages and other features) and add its dependencies to the gradle file.

For example:

```kotlin
repositories {
    jcenter()
}

dependencies {
    implementation("com.justai.aimybox:core:${version}")
    implementation("com.justai.aimybox:google-platform-speechkit:${version}")
    implementation("com.justai.aimybox:jaicf-api:${version}")
    implementation("com.justai.jaicf:caila:0.8.2") // Adds CAILA NLU support
}
```

And then configure your JAICF dialog API using these libraries:

```kotlin
private fun createAimybox(context: Context): Aimybox {
    val unitId = UUID.randomUUID().toString()

    val textToSpeech = GooglePlatformTextToSpeech(context, Locale.ENGLISH)
    val speechToText = GooglePlatformSpeechToText(context, Locale.ENGLISH)
    
    val engine = BotEngine(
        MainScenario.model,
        activators = arrayOf(CailaIntentActivator.Factory(
            CailaNLUSettings("your access token")
        ))
    )
    
    val dialogApi = JAICFDialogApi(unitId, engine)

    return Aimybox(Config.create(speechToText, textToSpeech, dialogApi))
}
```

Such configuration enables your assistant to process every user's request through a [CAILA NLU](https://app.jaicp.com/) and then send a retreived intent to your dialogue scenario,

### Generating content

JAICF dialog API transparently uses [Aimybox library](https://github.com/just-ai/jaicf-kotlin/tree/master/channels/aimybox) enabling to use _aimybox_ context to build Aimybox-specified responses. For example:

```kotlin
object MainScenario: Scenario() {
  init {
    state("bye") {
        activators {
            intent("bye")
        }

        action {
            reactions.run {
                say("Bye bye!")
                aimybox?.endConversation()
            }
        }
    }
  }  
}
```

## Documentation

There is a full Aimybox documentation available [here](https://help.aimybox.com)
