---
title: Traducir la entrada de usuario para que el bot sea multilingüe | Microsoft Docs
description: Cómo traducir la entrada de usuario automáticamente al lenguaje nativo del bot y volver a traducirla al idioma del usuario.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/06/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a6bec55010124892376904070cc9ed88c659efbd
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999862"
---
# <a name="translate-from-the-users-language-to-make-your-bot-multilingual"></a>Traducir del idioma del usuario para que el bot sea multilingüe

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

El bot puede usar [Microsoft Translator](https://www.microsoft.com/en-us/translator/) para traducir automáticamente los mensajes a un lenguaje que el bot entienda y, opcionalmente, traducir las respuestas del bot al idioma del usuario.
<!-- 
- [Get a Text Services key](#get-a-text-services-key)
- [Installing Packages](#installing-packages)
- [Combine translation with LUIS or QnA Maker](#using-luis)
- [Combine translation with QnA Maker](#using-qna-maker)
-->

## <a name="get-a-text-services-key"></a>Obtener una clave de Text Services

En primer lugar, necesitará una clave para usar el servicio Microsoft Translator. Puede obtener una [clave de evaluación gratuita](https://www.microsoft.com/en-us/translator/trial.aspx#get-started) en Azure Portal.

## <a name="installing-packages"></a>Instalar paquetes

Asegúrese de que tiene los paquetes necesarios para agregar la traducción al bot.

# <a name="ctabcsrefs"></a>[C#](#tab/csrefs)

[Agregue una referencia](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) a la versión preliminar de los siguientes paquetes NuGet:

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai` (necesario para la traducción)

Si va a combinar la traducción con Language Understanding (LUIS), agregue también una referencia a:

* `Microsoft.Bot.Builder.Luis` (necesario para LUIS)

# <a name="javascripttabjsrefs"></a>[JavaScript](#tab/jsrefs)

Puede agregar cualquiera de estos servicios al bot mediante el paquete botbuilder-ai. Puede agregar este paquete al proyecto a través de npm:
* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---

## <a name="configure-translation"></a>Configurar la traducción

Puede configurar el bot de modo que llame al traductor para cada mensaje que reciba de un usuario. Para ello, basta con agregarlo a la pila de software intermedio del bot. El software intermedio usa el resultado de la traducción para modificar el mensaje del usuario mediante el objeto de contexto.


# <a name="ctabcssetuptranslate"></a>[C#](#tab/cssetuptranslate)

Comience con el ejemplo EchoBot del SDK y actualice el método `ConfigureServices` en el archivo `Startup.cs` para agregar `TranslationMiddleware` al bot. Esto configura el bot de modo que traduzca todos los mensajes recibidos de un usuario. <!--, by simply adding it to your bot's middleware set. The middleware stores the translation results on the context object. -->

**Startup.cs**
```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<TranslatorBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;
        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));

    });
}


```

> [!TIP] 
> El SDK de Bot Builder detecta automáticamente el idioma del usuario en función del mensaje que acaba de enviar. Para invalidar esta funcionalidad, puede proporcionar parámetros adicionales de devolución de llamada para agregar su propia lógica a fin de detectar y cambiar el idioma del usuario.  



Eche un vistazo al código de `EchoBot.cs`, en el que se envía "You sent" seguido de lo que dice el usuario:

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Threading.Tasks;

namespace Microsoft.Bot.Samples
{
    public class EchoBot : IBot
    {
        public EchoBot() { }

        public async Task OnTurn(ITurnContext context)
        {
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                // echo back the user's input.
                    await context.SendActivity($"You sent '{context.Activity.Text}'");


                case ActivityTypes.ConversationUpdate:
                    foreach (var newMember in context.Activity.MembersAdded)
                    {
                        if (newMember.Id != context.Activity.Recipient.Id)
                        {
                            await context.SendActivity("Hello and welcome to the echo bot.");
                        }
                    }
                    break;
            }
        }
        
    }
}
```

Cuando se agrega software intermedio de traducción, un parámetro opcional especifica si se deben volver a traducir las respuestas al idioma del usuario. En `Startup.cs`, hemos especificado `false` para simplemente traducir los mensajes de usuario al lenguaje del bot.

```cs
        // The first parameter is a list of languages the bot recognizes
        // The second parameter is your API key
        // The third parameter specifies whether to translate the bot's replies back to the user's language
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));
```

# <a name="javascripttabjssetuptranslate"></a>[JavaScript](#tab/jssetuptranslate)

Para configurar el software intermedio de traducción con un bot de eco, pegue lo siguiente en app.js.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const { LanguageTranslator, LocaleConverter } = require('botbuilder-ai');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['fr', 'de'] 
});
adapter.use(languageTranslator);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---

## <a name="run-the-bot-and-see-translated-input"></a>Ejecutar el bot y ver la entrada traducida

Ejecute el bot y escriba algunos mensajes en otros idiomas. Verá que el bot ha traducido el mensaje de usuario e indica la traducción en su respuesta.

![el bot detecta el idioma y traduce la entrada](./media/how-to-bot-translate/bot-detects-language-translates-input.png)




## <a name="invoke-logic-in-the-bots-native-language"></a>Invocar lógica en el lenguaje nativo del bot

Ahora, agregue la lógica que busque palabras en inglés. Si el usuario dice "ayuda" o "cancelar" en un idioma que no sea inglés, el bot lo traduce al inglés y se invoca la lógica que busca las palabras en inglés "help" o "cancel".

# <a name="ctabcshelp"></a>[C#](#tab/cshelp)
```cs
        case ActivityTypes.Message:
            // check the message text before calling context.SendActivity
            var messagetext = context.Activity.Text.Trim().ToLower();
            if (string.Equals("help", messagetext))
            {
                await context.SendActivity("You asked for help.");
            }
```

# <a name="javascripttabjshelp"></a>[JavaScript](#tab/jshelp)
```javascript
if (context.activity.type === 'message') {
    // check the message text before calling context.sendActivity
    var messagetext = context.activity.text.trim().toLowerCase();
    if(messagetext === "help"){
        await context.sendActivity("you said help");
    }
}
```

---

![el bot detecta la palabra "ayuda" en francés](./media/how-to-bot-translate/bot-detects-help-french.png)



## <a name="translate-replies-back-to-the-users-language"></a>Traducir las respuestas de nuevo al idioma del usuario

También puede volver a traducir las respuestas al idioma del usuario. Para ello, establezca el último parámetro de constructor en `true`.

# <a name="ctabcstranslateback"></a>[C#](#tab/cstranslateback)
```cs
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "TRANSLATION-SUBSCRIPTION-KEY", true));
```

# <a name="javascripttabjstranslateback"></a>[JavaScript](#tab/jstranslateback)
```javascript
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);
```

---

## <a name="run-the-bot-to-see-replies-in-the-users-language"></a>Ejecutar el bot para ver las respuestas en el idioma del usuario

Ejecute el bot y escriba algunos mensajes en otros idiomas. Verá que detecta el idioma del usuario y traduce la respuesta.

![el bot detecta el idioma y traduce la respuesta](./media/how-to-bot-translate/bot-detects-language-translates-response.png)


## <a name="adding-logic-for-detecting-or-changing-the-user-language"></a>Agregar lógica para detectar o cambiar el idioma del usuario

En lugar de dejar que Botbuilder SDK detecte automáticamente el idioma del usuario, puede proporcionar devoluciones de llamada para agregar su propia lógica a fin de determinar el idioma del usuario, o bien determinar cuándo ha cambiado el idioma del usuario.

> [!TIP] 
> Vea el ejemplo de traducción del SDK para obtener un bot de ejemplo que implementa devoluciones de llamada para detectar y cambiar el idioma del usuario. 

En el ejemplo siguiente, la devolución de llamada `CheckUserChangedLanguage` comprueba si hay un mensaje de usuario específico para cambiar el idioma. 

# <a name="ctabcschangelanguage"></a>[C#](#tab/cschangelanguage)
```cs
// In Startup.cs
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", null, TranslatorLocaleHelper.GetActiveLanguage, TranslatorLocaleHelper.CheckUserChangedLanguage));

// Add a file TranslatorLocalHelper.cs with the following:

    class CurrentUserState
    {
        public string Language { get; set; }
    }

    public static void SetLanguage(ITurnContext context, string language) => context.GetConversationState<CurrentUserState>().Language = language;

    public static bool IsSupportedLanguage(string language) => _supportedLanguages.Contains(language);

    public static async Task<bool> CheckUserChangedLanguage(ITurnContext context)
    {
        bool changeLang = false;
        
        // use a specific message from user to change language
        if (context.Activity.Type == ActivityTypes.Message)
        {
            var messageActivity = context.Activity.AsMessageActivity();
            if (messageActivity.Text.ToLower().StartsWith("set my language to"))
            {
                changeLang = true;
            }
            if (changeLang)
            {
                var newLang = messageActivity.Text.ToLower().Replace("set my language to", "").Trim();
                if (!string.IsNullOrWhiteSpace(newLang)
                        && IsSupportedLanguage(newLang))
                {
                    SetLanguage(context, newLang);
                    await context.SendActivity($@"Changing your language to {newLang}");
                }
                else
                {
                    await context.SendActivity($@"{newLang} is not a supported language.");
                }
                // return true to intercept message from further processesing
                return true;
            }
        }

        return false;
    }

    public static string GetActiveLanguage(ITurnContext context)
    {

        if (context.Activity.Type == ActivityTypes.Message
            && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Language != null)
        {
            return context.GetConversationState<CurrentUserState>().Language;
        }

        return "en";
    }

```

# <a name="javascripttabjschangelanguage"></a>[JavaScript](#tab/jschangelanguage)
```javascript
// When the user inputs 'set my language to fr'
// The bot will automatically change all text to French
async function setUserLanguage(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my language to')) {
        state.language = context.activity.text.toLowerCase().replace('set my language to', '').trim();
        await context.sendActivity(`Setting your language to ${state.language}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR MICROSOFT TRANSLATOR API KEY>",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    setUserLanguage: setUserLanguage,
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`You said: ${context.activity.text}`)
        }
    });
});

```

---

## <a name="combining-luis-or-qna-with-translation"></a>Combinar LUIS o QnA con la traducción

Si combina la traducción con otros servicios en el bot, como LUIS o QnA Maker, agregue primero el software intermedio de traducción para que los mensajes se traduzcan antes de pasarlos a otro software intermedio que espera el lenguaje nativo del bot.

# <a name="ctabcslanguageluis"></a>[C#](#tab/cslanguageluis)
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;

        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes. Input in these language won't be translated.
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));

        // Add LUIS middleware after translation middleware to pass translated messages to LUIS
        middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel("luisAppId", "subscriptionId", new Uri("luisModelBaseUrl"))));
    });
}
```

# <a name="javascripttabjslanguageluis"></a>[JavaScript](#tab/jslanguageluis)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false
});
adapter.use(languageTranslator);

// Add Luis recognizer middleware
const luisRecognizer = new LuisRecognizer({
    appId: "xxxxxx",
    subscriptionKey: "xxxxxxx"
});
adapter.use(luisRecognizer);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            let results = luisRecognizer.get(context);
            for (const intent in results.intents) {
                await context.sendActivity(`intent: ${intent.toString()} : ${results.intents[intent.toString()]}`)
            }
        }
    });
});

```

---

En el código del bot, los resultados de LUIS se basan en la entrada que ya se ha traducido al lenguaje nativo del bot. Pruebe a modificar el código del bot para comprobar los resultados de una aplicación de LUIS:

```cs
public async Task OnTurn(ITurnContext context)
{
    switch (context.Activity.Type)
    {
        case ActivityTypes.Message:
            // check LUIS results
            var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
            (string key, double score) topItem = luisResult.GetTopScoringIntent();
            await context.SendActivity($"The top intent was: '{topItem.key}', with score {topItem.score}");

            await context.SendActivity($"You sent '{context.Activity.Text}'");

        case ActivityTypes.ConversationUpdate:
            foreach (var newMember in context.Activity.MembersAdded)
            {
                if (newMember.Id != context.Activity.Recipient.Id)
                {
                    await context.SendActivity("Hello and welcome to the echo bot.");
                }
            }
            break;
        }
    }  
}          
```

## <a name="bypass-translation-for-specified-patterns"></a>Omitir la traducción de patrones especificados
Es posible que no le interese que el bot traduzca ciertas palabras, como los nombres propios. Puede proporcionar expresiones regulares para indicar los patrones que no hay que traducir. Por ejemplo, si el usuario dice "Me llamo…" en un lenguaje no nativo para el bot y quiere impedir que su nombre se traduzca, puede usar un patrón para especificarlo.

# <a name="ctabcsbypass"></a>[C#](#tab/csbypass)
```cs
    // Startup.cs

    // Pattern representing input to not translate
    Dictionary<string, List<string>> patterns = new Dictionary<string, List<string>>();
    // single pattern for fr language, to avoid translating what follows "my name is"
    patterns.Add("fr", new List<string> { "mon nom est (.+)" });
    
    middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR API KEY>", patterns, TranslatorLocaleHelper.GetActiveLanguage, TranslatorLocaleHelper.CheckUserChangedLanguage));

```
<!-- TODO: ADD more explanation (both of these callbacks are run every time), fix image by debugging regex for l'etat -->

# <a name="javascripttabjsbypass"></a>[JavaScript](#tab/jsbypass)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR API KEY>",
    // single pattern for fr language, to avoid translating what follows "my name is"
    noTranslatePatterns: new Set("fr", "/mon nom est (.+)/"),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false,
    
});
adapter.use(languageTranslator);

```

---

![el bot omite la traducción de un patrón](./media/how-to-bot-translate/bot-no-translate-name-fr.png)

## <a name="localize-dates"></a>Localizar fechas

Si necesita localizar fechas, puede agregar `LocaleConverterMiddleware`. Por ejemplo, si sabe que el bot espera las fechas en el formato `MM/DD/YYYY` y es posible que los usuarios de otras configuraciones regionales escriban las fechas en el formato `DD/MM/YYYY`, el software intermedio del convertidor de configuración regional puede convertir automáticamente las fechas al formato que espera el bot.

> [!NOTE]
> El software intermedio del convertidor de configuración regional está pensado para convertir solo fechas. No tiene ningún conocimiento sobre los resultados del software intermedio de traducción. Si usa software intermedio de traducción, tenga cuidado al combinarlo con el convertidor de configuración regional. El software intermedio de traducción traduce las fechas que están en formato de texto junto con el resto de la entrada de texto, pero no traduce otras fechas.

Por ejemplo, en la imagen siguiente se muestra un bot que devuelve la entrada de usuario después de traducirla de inglés a francés. Usa `TranslationMiddleware` sin usar `LocaleConverterMiddleware`.

![bot que traduce las fechas sin conversión de fechas](./media/how-to-bot-translate/locale-date-before.png)

A continuación se muestra el mismo bot con `LocaleConverterMiddleware` agregado.

![bot que traduce las fechas sin conversión de fechas](./media/how-to-bot-translate/locale-date-after.png)

Los convertidores de configuración regional son compatibles con las configuraciones regionales de inglés, francés, alemán y chino. <!-- TODO: ADD DETAIL ABOUT SUPPORTED LOCALES -->

# <a name="ctabcstranslation"></a>[C#](#tab/cstranslation)
```cs
// Startup.cs
// Add locale converter middleware
middleware.Add(new LocaleConverterMiddleware(TranslatorLocaleHelper.GetActiveLocale, TranslatorLocaleHelper.CheckUserChangedLocale, "en-us", LocaleConverter.Converter));

// TranslatorLocaleHelper.cs
        public static async Task<bool> CheckUserChangedLocale(ITurnContext context)
        {
            bool changeLocale = false;//logic implemented by developper to make a signal for language changing 
            //use a specific message from user to change language
            if (context.Activity.Type == ActivityTypes.Message)
            {
                var messageActivity = context.Activity.AsMessageActivity();
                if (messageActivity.Text.ToLower().StartsWith("set my locale to"))
                {
                    changeLocale = true;
                }
                if (changeLocale)
                {
                    var newLocale = messageActivity.Text.ToLower().Replace("set my locale to", "").Trim(); //extracted by the user using user state 
                    if (!string.IsNullOrWhiteSpace(newLocale)
                            && IsSupportedLanguage(newLocale))
                    {
                        SetLocale(context, newLocale);
                        await context.SendActivity($@"Changing your language to {newLocale}");
                    }
                    else
                    {
                        await context.SendActivity($@"{newLocale} is not a supported locale.");
                    }
                    //intercepts message
                    return true;
                }
            }

            return false;
        }
        public static string GetActiveLocale(ITurnContext context)
        {
            if (currentLocale != null)
            {
                //the user has specified a different locale so update the bot state
                if (context.GetConversationState<CurrentUserState>() != null
                    && currentLocale != context.GetConversationState<CurrentUserState>().Locale)
                {
                    SetLocale(context, currentLocale);
                }
            }
            if (context.Activity.Type == ActivityTypes.Message
                && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Locale != null)
            {
                return context.GetConversationState<CurrentUserState>().Locale;
            }

            return "en-us";
        }
```

# <a name="javascripttabjstranslation"></a>[JavaScript](#tab/jstranslation)

<!-- this snippet only works if the user doesn't actually try to change their locale.  Emailed Mostafa about the issue 
It should change the locale after you type in 'set my locale to....' -->

```javascript
// Delegates for getting and setting user locale
function getUserLocale(context) {
    const state = conversationState.get(context)
    if (state.locale == undefined) {
        return 'en-us';
    } else {
        return state.locale;
    }
}

// Function to change the user's locale.
async function setUserLocale(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my locale to')) {        
        state.locale = context.activity.text.toLowerCase().replace('set my locale to', '').trim();
        await context.sendActivity(`Setting your locale to ${state.locale}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add locale converter middleware
// Will convert input to fr-fr
const localeConverter = new LocaleConverter({
    toLocale: 'fr-fr',
    setUserLocale: setUserLocale,
    getUserLocale: getUserLocale
});
adapter.use(localeConverter);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`The message that you sent was:  ${context.activity.text}`)
        }
    });
});

```

---
