---
title: Creación de una habilidad de Cortana mediante Node.js | Microsoft Docs
description: Obtenga información sobre los conceptos básicos para la creación de una habilidad de Cortana en Bot Framework SDK para Node.js.
keywords: Bot Framework, habilidad de Cortana, voz, Node.js, Bot Builder, SDK, conceptos clave, conceptos básicos
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/10/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5de773f6f8f4d46c0c1fe880588f2530c3c68f56
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2019
ms.locfileid: "59746069"
---
# <a name="key-concepts-for-building-a-bot-for-cortana-skills-using-nodejs"></a>Conceptos básicos para crear un bot para habilidades de Cortana con Node.js
 
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!NOTE]
> Este artículo es contenido y se actualizará.

En este artículo se describen los conceptos clave para crear una habilidad de Cortana en Bot Framework SDK para Node.js. 

## <a name="what-is-a-cortana-skill"></a>¿Qué es una habilidad de Cortana?
Una habilidad de Cortana es un bot que se puede invocar con el uso de un cliente de Cortana, como el que está integrado en Windows 10. El usuario lanza el bot diciendo algunas palabras clave o frases asociadas con el bot. En el portal de Framework Bot puede configurar qué palabras clave se usan para lanzar el bot. 

Cortana se puede considerar como un canal habilitado para voz que puede enviar y recibir mensajes de voz, además de tener conversaciones por escrito. Un bot que se publica como una habilidad de Cortana debe diseñarse para voz y texto. Bot Framework proporciona métodos para especificar el lenguaje de marcado de síntesis de voz (SSML) para definir los mensajes de voz que su bot envía.

## <a name="acknowledge-user-utterances"></a>Reconocimiento de grabaciones de voz del usuario 

<!-- Establishing conversational understanding -->
<!-- Placeholder: In this section, describe how you have to write your speech to sound natural -->


Al crear un bot habilitado para voz, debe tratar de establecer fundamentos comunes y un entendimiento mutuo en la conversación. El bot debe "fundamentar" las grabaciones de voz del usuario con la indicación de que se ha escuchado y entendido al usuario.

Los usuarios se sienten confundidos si se produce un error del sistema al fundamentar sus grabaciones de voz. Por ejemplo, la conversación siguiente puede ser un poco confusa cuando el bot pregunta "¿Qué es lo siguiente?":

> **Cortana**: ¿Desea revisar algo más de su perfil?  
> **User**:  No.  
> **Cortana**: Pasos siguientes

Si el bot agrega "De acuerdo" como reconocimiento, es más descriptivo para el usuario:

> **Cortana**: ¿Desea revisar algo más de su perfil?  
> **User**:  No.  
> **Cortana**: **De acuerdo**, ¿cuáles son los siguientes pasos?

Grados de fundamentación, desde el más débil al más sólido:

1. Atención continuada
2. Próxima contribución pertinente
3. Confirmación: Respuesta mínima o continuador: "sí", "ajá", "de acuerdo" o "genial"
4. Demostración: indicar que se ha comprendido mediante reformulación o conclusión.
5. Mostrar: repetir todo o solo una parte.

### <a name="acknowledgement-and-next-relevant-contribution"></a>Reconocimiento y próxima contribución pertinente

> **User**: Tengo que viajar en mayo.  
> **Cortana**: **De acuerdo**. ¿Qué día de mayo quiere viajar?  
> **User**: Tengo que estar allí del 12 al 15.  
> **Cortana**: **De acuerdo**. ¿A qué ciudad va a volar?  

### <a name="grounding-by-demonstration"></a>Fundamentación mediante demostración

> **User**: Tengo que viajar en mayo.  
> **Cortana**: **Y**, ¿qué día de mayo quiere viajar?  
> **User**: Tengo que estar allí del 12 al 15.  
> **Cortana**: **Y**, ¿a qué ciudad va a viajar?  
    
### <a name="closure"></a>Cierre

El bot que realiza una acción debe presentar pruebas de su correcta ejecución. También es importante que indique si ha producido algún error o si se ha entendido bien o no. 

* Cierre sin voz: si presiona un botón elevador, su luz se enciende.  
Este es un proceso en dos pasos:
    * Presentación (al presionar el botón)
    * Aceptación (cuando se enciende el botón)

## <a name="differences-in-content-presentation"></a>Diferencias en la presentación del contenido
Tenga en cuenta que Cortana es compatible con diversos dispositivos, de los cuales solo algunos tienen pantalla. Una de las cosas se debe considerar a la hora de diseñar un bot compatible con voz es que, con frecuencia, el diálogo hablado no será igual que los mensajes de texto que el bot muestra.
<!-- If there are differences in what the bot will say, in the text vs the speak fields of a prompt or in a waterfall, for example, discuss them here.

## Speech

You bot uses the **session.say** method to speak to the user. The speak method has three overloads:
* If you pass only one parameter to **session.say**, it can be a text parameter.
* If you pass two parameters to **session.say**, it can take text and SSML.
* If you pass three parameters, the third parameter takes an options structure that specifies all the options you can pass to build an **IMessage** object.

```javascript
var bot = new builder.UniversalBot(connector, function (session) {
    session.say("Hello... I'm a decision making bot.'.", 
        ssml.speak("Hello. I can help you answer all of life's tough questions."));
    session.replaceDialog('rootMenu');
});

```
## Speech in messages

The **IMessage** object provides a **speak** property for SSML. It can be used to play a .wav file.

The **inputHint** property helps indicate to Cortana whether your bot is expecting input. If you're using a built-in prompt, this value is automatically set to the default of **expectingInput**.

The **inputHint** property can take the following values: 
* **expectingInput**: Indicates that the bot is actively expecting a response from the user. Cortana listens for the user to speak into the microphone.
* **acceptingInput**: Indicates that the bot is passively ready for input but is not waiting on a response. Cortana accepts input from the user if the user holds down the microphone button.
* **ignoringInput**: Cortana is ignoring input. Your bot may send this hint if it is actively processing a request and will ignore input from users until the request is complete.

Prompts must use the `speak:` option.

```javascript
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```

Prompts.number has *ordinal support*, meaning that you can say "the last", "the first", "the next-to-last" to choose an item in a list.

## Using synonyms

<!-- Axl Rose example -->
```javascript   
         var choices = [
            { 
                value: 'flipCoinDialog',
                action: { title: "Flip A Coin" },
                synonyms: 'toss coin|flip quarter|toss quarter'
            },
            {
                value: 'rollDiceDialog',
                action: { title: "Roll Dice" },
                synonyms: 'roll die|shoot dice|shoot die'
            },
            {
                value: 'magicBallDialog',
                action: { title: "Magic 8-Ball" },
                synonyms: 'shake ball'
            },
            {
                value: 'quit',
                action: { title: "Quit" },
                synonyms: 'exit|stop|end'
            }
        ];
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```

## <a name="configuring-your-bot"></a>Configuración del bot

## <a name="prompts"></a>Mensajes

## <a name="additional-resources"></a>Recursos adicionales

Documentación de Cortana: [Documentación de aptitudes de Cortana](/cortana/skills/)

Referencias de SSML de Cortana: [Referencia del Lenguaje de marcado de síntesis de voz (SSML)](/cortana/skills/speech-synthesis-markup-language)
