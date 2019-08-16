---
title: Compatibilidad con la localización | Microsoft Docs
description: Obtenga información sobre cómo determinar dónde está el usuario y habilitar la funcionalidad de localización mediante Bot Framework SDK para Node.js.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ce1b3f073c932cd4042b91ae9afc1e332a7443f2
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2019
ms.locfileid: "67404903"
---
# <a name="support-localization"></a>Compatibilidad con la localización

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Builder incluye un completo sistema de localización para la creación de bots que pueden comunicarse con el usuario en varios idiomas. Todos los avisos del bot se pueden localizar mediante archivos JSON almacenados en la estructura de directorios del bot. Si usa un sistema como LUIS para realizar el procesamiento de lenguaje natural, puede configurar [LuisRecognizer][LUISRecognizer] con un modelo independiente para cada idioma que admita su bot, y el SDK seleccionará automáticamente el modelo que coincida con la configuración regional preferida del usuario.

## <a name="determine-the-locale-by-prompting-the-user"></a>Preguntar al usuario para determinar la configuración regional
El primer paso para localizar su bot es agregar la posibilidad de identificar el idioma preferido del usuario. El SDK proporciona un método [session.preferredLocale()][preferredLocal] para guardar y recuperar esta preferencia de forma individualizada. El ejemplo siguiente es un diálogo que pregunta al usuario su idioma preferido y, a continuación, guarda su elección.

``` javascript
bot.dialog('/localePicker', [
    function (session) {
        // Prompt the user to select their preferred locale
        builder.Prompts.choice(session, "What's your preferred language?", 'English|Español|Italiano');
    },
    function (session, results) {
        // Update preferred locale
        var locale;
        switch (results.response.entity) {
            case 'English':
                locale = 'en';
                break;
            case 'Español':
                locale = 'es';
                break;
            case 'Italiano':
                locale = 'it';
                break;
        }
        session.preferredLocale(locale, function (err) {
            if (!err) {
                // Locale files loaded
                session.endDialog(`Your preferred language is now ${results.response.entity}`);
            } else {
                // Problem loading the selected locale
                session.error(err);
            }
        });
    }
]);
```

## <a name="determine-the-locale-by-using-analytics"></a>Determinación de la configuración regional mediante análisis
Otra manera de determinar la configuración regional del usuario es usar un servicio como [Text Analytics API](/azure/cognitive-services/cognitive-services-text-analytics-quick-start) para detectar automáticamente el lenguaje del usuario según el texto del mensaje que envía.

El siguiente fragmento de código muestra cómo se puede incorporar este servicio a su propio bot.
``` javascript
var request = require('request');

bot.use({
    receive: function (event, next) {
        if (event.text && !event.textLocale) {
            var options = {
                method: 'POST',
                url: 'https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/languages?numberOfLanguagesToDetect=1',
                body: { documents: [{ id: 'message', text: event.text }]},
                json: true,
                headers: {
                    'Ocp-Apim-Subscription-Key': '<YOUR API KEY>'
                }
            };
            request(options, function (error, response, body) {
                if (!error && body) {
                    if (body.documents && body.documents.length > 0) {
                        var languages = body.documents[0].detectedLanguages;
                        if (languages && languages.length > 0) {
                            event.textLocale = languages[0].iso6391Name;
                        }
                    }
                }
                next();
            });
        } else {
            next();
        }
    }
});
```

Una vez que agregue el fragmento de código anterior al bot, la llamada a [session.preferredLocale()][preferredLocal] devolverá automáticamente el idioma detectado. El orden de búsqueda de `preferredLocale()` es así:
1. Configuración regional guardada mediante la llamada a `session.preferredLocale()`. Este valor se almacena en `session.userData['BotBuilder.Data.PreferredLocale']`.
2. Configuración regional detectada asignada a `session.message.textLocale`.
3. La configuración regional predeterminada configurada para el bot (p. ej.: Inglés ('en')).

Puede configurar la configuración regional predeterminada del bot mediante su constructor:

```javascript
var bot = new builder.UniversalBot(connector, {
    localizerSettings: { 
        defaultLocale: "es" 
    }
});
```

## <a name="localize-prompts"></a>Localización de los avisos
El sistema de localización predeterminado para Bot Framework SDK está basado en archivo y permite que un bot admita varios idiomas con archivos JSON almacenados en disco. De forma predeterminada, el sistema de localización buscará los avisos del bot en el archivo **./locale/<IETF TAG>/index.json** donde <IETF TAG> es una [etiqueta de idioma IETF][IEFT] válida que representa la configuración regional preferida para la que se buscarán los avisos. 

La captura de pantalla siguiente muestra la estructura de directorios de un bot que admite tres lenguajes: Inglés, italiano y español.

![Estructura de directorio de tres configuraciones regionales](../media/locale-dir.png)

La estructura del archivo es una sencilla asignación JSON de identificadores de mensaje a cadenas de texto localizadas. Si el valor es una matriz en lugar de una cadena, se elige un aviso de la matriz de forma aleatoria cuando el valor se recupera mediante [session.localizer.gettext()][GetText]. 

El bot recupera automáticamente la versión localizada de un mensaje si se pasa el identificador del mensaje en una llamada a [session.send()](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#send) en lugar de texto específico del idioma:

```javascript
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("greeting");
        session.send("instructions");
        session.beginDialog('/localePicker');
    },
    function (session) {
        builder.Prompts.text(session, "text_prompt");
    }
]);
```

Internamente, el SDK llama a [`session.preferredLocale()`][preferredLocale] para obtener la configuración regional preferida del usuario y luego la usa en una llamada a [`session.localizer.gettext()`][GetText] para asignar el identificador de mensaje a su cadena de texto localizada.  Hay ocasiones en que es posible que deba llamar manualmente al localizador. Por ejemplo, los valores de enumeración pasados a [`Prompts.choice()`][promptsChoice] nunca se localizan automáticamente, por lo que es posible que tenga que recuperar manualmente una lista localizada antes de llamar al aviso:

```javascript
var options = session.localizer.gettext(session.preferredLocale(), "choice_options");
builder.Prompts.choice(session, "choice_prompt", options);
```

El localizador predeterminado busca un identificador de mensaje en varios archivos y, si no encuentra uno (o no se han proporcionado archivos de localización), simplemente devuelve el texto del identificador, lo que hace que el uso de archivos de localización sea transparente y opcional.  Se busca en los archivos en el orden siguiente:

1. Se busca en el archivo **index.json** con la configuración regional devuelta por [`session.preferredLocale()`][preferredLocale].
2. Si la configuración regional incluía una subetiqueta opcional como **en-US**, se busca en la etiqueta raíz de **en**.
3. Se busca en la configuración regional predeterminada configurada del bot.

## <a name="use-namespaces-to-customize-and-localize-prompts"></a>Uso de espacios de nombres para personalizar y localizar avisos
El localizador predeterminado admite el uso de espacios de nombres de avisos para evitar conflictos entre identificadores de mensaje.  El bot puede invalidar los avisos de un espacio de nombres para personalizar o reformular los avisos de otro espacio de nombres.  Puede aprovechar esta funcionalidad para personalizar los mensajes integrados del SDK, y así agregar compatibilidad con idiomas adicionales o simplemente reformular los mensajes actuales del SDK.  Por ejemplo, puede cambiar el mensaje de error predeterminado del SDK con solo agregar un archivo llamado **BotBuilder.json** al directorio de configuración regional del bot y luego agregar una entrada para el identificador de mensaje `default_error`:

![BotBuilder.json para el uso de espacios de nombres de configuración regional](../media/locale-namespacing.png)


## <a name="additional-resources"></a>Recursos adicionales

Para información sobre cómo localizar un reconocedor, consulte [Reconocimiento de intenciones](bot-builder-nodejs-recognize-intent-messages.md).


[LUIS]: https://www.luis.ai/
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[IntentRecognizerSetOptions]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html
[LUISRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISSample]: https://aka.ms/v3-js-luisSample
[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute
[preferredLocal]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[preferredLocale]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[promptsChoice]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice
[GetText]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ilocalizer.html#gettext
[IEFT]: https://en.wikipedia.org/wiki/IETF_language_tag

