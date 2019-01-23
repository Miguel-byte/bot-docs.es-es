---
title: Reconocimiento de intenciones y entidades con LUIS | Microsoft Docs
description: Aprenda a habilitar el bot para entender el lenguaje natural con el uso de diálogos de LUIS en Bot Framework SDK para .NET.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 26b23c9beda872b71defd779563531328b63a0c8
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225380"
---
# <a name="recognize-intents-and-entities-with-luis"></a>Reconocimiento de intenciones y entidades con LUIS 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Este artículo usa el ejemplo de un bot para tomar notas, a fin de demostrar cómo Language Understanding ([LUIS][LUIS]) ayuda a que el bot responda correctamente a la entrada de lenguaje natural. Un bot detecta lo que un usuario desea hacer mediante la identificación de su **intención**. Esta intención se determina a partir de la entrada de texto, voz o **grabaciones de voz**. La intención asigna grabaciones de voz a acciones que realiza el bot. Por ejemplo, un bot de toma de notas reconoce una intención `Notes.Create` para invocar la funcionalidad para crear una nota. Puede que un bot también necesite extraer **entidades**, que son palabras importantes de las grabaciones de voz. En el ejemplo de un bot de toma de notas, la entidad `Notes.Title` identifica el título de cada nota.

## <a name="create-a-language-understanding-bot-with-bot-service"></a>Crear un bot de Language Understanding con Bot Service

1. En [Azure Portal](https://portal.azure.com), haga clic en **Crear nuevo recurso** en la hoja de menú y, después, en **Ver todo**.

    ![Crear recurso](../media/bot-builder-dotnet-use-luis/bot-service-creation.png)

2. En el cuadro de búsqueda, busque **Bot de aplicación web**. 

    ![Crear recurso](../media/bot-builder-dotnet-use-luis/bot-service-selection.png)

3. En la hoja **Servicio de bots**, proporcione la información necesaria y haga clic en **Crear**. Esto crea e implementa el servicio de bots y la aplicación de LUIS en Azure. 
   * Establezca **Nombre de la aplicación** en el nombre del bot. El nombre se usa como el subdominio cuando el bot se implementa en la nube (por ejemplo, mynotesbot.azurewebsites.net). Este nombre también se utiliza como el nombre de la aplicación de LUIS asociada con el bot. Cópielo para usarlo más adelante, para encontrar la aplicación de LUIS asociada con el bot.
   * Seleccione la suscripción, el [grupo de recursos](/azure/azure-resource-manager/resource-group-overview), el plan de App Service y la [ubicación](https://azure.microsoft.com/en-us/regions/).
   * Seleccione la plantilla **Language Understanding (C#)** en el campo **Bot template** (Plantilla de bot).

     ![Hoja Servicio de bots](../media/bot-builder-dotnet-use-luis/bot-service-setting-callout-template.png)

   * Active la casilla para confirmar los términos del servicio.

4. Confirme que se ha implementado el servicio de bots.
    * Haga clic en Notificaciones (el icono de la campana que se encuentra en el borde superior de Azure Portal). La notificación cambiará de **Implementación iniciada** a **Implementación correcta**.
    * Después de que la notificación cambie a **Implementación correcta**, haga clic en **Ir al recurso** en esa notificación.

## <a name="try-the-bot"></a>Probar el bot

Para confirmar que el bot se ha implementado, active las **Notificaciones**. Las notificaciones cambiarán de **Implementación en curso...** a **Implementación correcta**. Haga clic en el botón **Ir al recurso** para abrir la hoja de recursos del bot.

Una vez registrado el bot, haga clic en **Test in Web Chat** (Probar en Chat en web) para abrir el panel Chat en web. Escriba "hello" ("hola") en el Chat en web.

  ![Probar el bot en Chat en web](../media/bot-builder-dotnet-use-luis/bot-service-web-chat.png)

El bot responde "You have reached Greeting. You said: hello" ("Se ha puesto en contacto con Saludo. Ha dicho: hola"). Esto confirma que el bot ha recibido el mensaje y lo ha pasado a una aplicación de LUIS predeterminada que ha creado. Esta aplicación de LUIS predeterminada ha detectado una intención Saludo.

## <a name="modify-the-luis-app"></a>Modificación de la aplicación de LUIS

Inicie sesión en [https://www.luis.ai](https://www.luis.ai) con la misma cuenta que usa para iniciar sesión en Azure. Haga clic en **Mis aplicaciones**. En la lista de aplicaciones, busque la aplicación que comienza con el nombre especificado en **Nombre de la aplicación** en la hoja **Servicio de bots** cuando creó el servicio de bots. 

La aplicación de LUIS se inicia con 4 intenciones: Cancelar: Greeting, Help y None. <!-- picture -->

Los siguientes pasos agregan las intenciones Note.Create, Note.ReadAloud y Note.Delete: 

1. Haga clic en **Dominios creados previamente** en la parte inferior izquierda de la página. Busque el dominio **Nota** y haga clic en **Agregar dominio**.

2. En este tutorial no se usan todas las intenciones incluidas en el dominio **Nota** creado previamente. En la página **Intenciones**, haga clic en cada uno de los siguientes nombres de intenciones y luego haga clic en el botón **Eliminar intención**.
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   Las únicas intenciones que deben permanecer en la aplicación de LUIS son las siguientes: 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * None
   * Ayuda
   * Greeting
   * Cancelar 

     ![intenciones mostradas en la aplicación de LUIS](../media/bot-builder-dotnet-use-luis/luis-intent-list.png)

3. Haga clic en el botón **Entrenar** en la esquina superior derecha para entrenar la aplicación.
4. Haga clic en **PUBLICAR** en la barra de navegación superior para abrir la página **Publicar**. Haga clic en el botón **Publish to production slot** (Publicar en el espacio de producción). Tras una publicación correcta, copie la dirección URL mostrada en la columna **Punto de conexión** de la página **Publicar aplicación**, en la fila que empieza con el nombre de recurso Starter_Key. Guarde esta dirección URL para usarla más adelante en el código del bot. La dirección URL tiene un formato similar a este ejemplo: `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`.

## <a name="modify-the-bot-code"></a>Modificar el código del bot

Haga clic en **Compilar** y después en **Open online code editor** (Abrir el editor de código en línea).
    ![Abrir el editor de código en línea](../media/bot-builder-dotnet-use-luis/bot-service-build.png)

En el editor de código, abra `BasicLuisDialog.cs`. Contiene el código siguiente para controlar las intenciones de la aplicación de LUIS.
```cs
using System;
using System.Configuration;
using System.Threading.Tasks;

using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

namespace Microsoft.Bot.Sample.LuisBot
{
    // For more information about this template visit http://aka.ms/azurebots-csharp-luis
    [Serializable]
    public class BasicLuisDialog : LuisDialog<object>
    {
        public BasicLuisDialog() : base(new LuisService(new LuisModelAttribute(
            ConfigurationManager.AppSettings["LuisAppId"], 
            ConfigurationManager.AppSettings["LuisAPIKey"], 
            domain: ConfigurationManager.AppSettings["LuisAPIHostName"])))
        {
        }

        [LuisIntent("None")]
        public async Task NoneIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        // Go to https://luis.ai and create a new intent, then train/publish your luis app.
        // Finally replace "Greeting" with the name of your newly created intent in the following handler
        [LuisIntent("Greeting")]
        public async Task GreetingIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Cancel")]
        public async Task CancelIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Help")]
        public async Task HelpIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        private async Task ShowLuisResult(IDialogContext context, LuisResult result) 
        {
            await context.PostAsync($"You have reached {result.Intents[0].Intent}. You said: {result.Query}");
            context.Wait(MessageReceived);
        }
    }
}
```
### <a name="create-a-class-for-storing-notes"></a>Creación de una clase para almacenar notas

Agregue la siguiente instrucción `using` en BasicLuisDialog.cs.

```cs
using System.Collections.Generic;
```

Agregue el siguiente código en la clase `BasicLuisDialog`, después de la definición del constructor.

```cs
        // Store notes in a dictionary that uses the title as a key
        private readonly Dictionary<string, Note> noteByTitle = new Dictionary<string, Note>();
        
        [Serializable]
        public sealed class Note : IEquatable<Note>
        {

            public string Title { get; set; }
            public string Text { get; set; }

            public override string ToString()
            {
                return $"[{this.Title} : {this.Text}]";
            }

            public bool Equals(Note other)
            {
                return other != null
                    && this.Text == other.Text
                    && this.Title == other.Title;
            }

            public override bool Equals(object other)
            {
                return Equals(other as Note);
            }

            public override int GetHashCode()
            {
                return this.Title.GetHashCode();
            }
        }

        // CONSTANTS        
        // Name of note title entity
        public const string Entity_Note_Title = "Note.Title";
        // Default note title
        public const string DefaultNoteTitle = "default";
```

### <a name="handle-the-notecreate-intent"></a>Control de la intención Note.Create
Para controlar la intención Note.Create, agregue el código siguiente a la clase `BasicLuisDialog`.

```cs
        private Note noteToCreate;
        private string currentTitle;
        [LuisIntent("Note.Create")]
        public Task NoteCreateIntent(IDialogContext context, LuisResult result)
        {
            EntityRecommendation title;
            if (!result.TryFindEntity(Entity_Note_Title, out title))
            {
                // Prompt the user for a note title
                PromptDialog.Text(context, After_TitlePrompt, "What is the title of the note you want to create?");
            }
            else
            {
                var note = new Note() { Title = title.Entity };
                noteToCreate = this.noteByTitle[note.Title] = note;

                // Prompt the user for what they want to say in the note           
                PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");
            }
            
            return Task.CompletedTask;
        }
        
        
        private async Task After_TitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            EntityRecommendation title;
            // Set the title (used for creation, deletion, and reading)
            currentTitle = await result;
            if (currentTitle != null)
            {
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = currentTitle };
            }
            else
            {
                // Use the default note title
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = DefaultNoteTitle };
            }

            // Create a new note object 
            var note = new Note() { Title = title.Entity };
            // Add the new note to the list of notes and also save it in order to add text to it later
            noteToCreate = this.noteByTitle[note.Title] = note;

            // Prompt the user for what they want to say in the note           
            PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");

        }

        private async Task After_TextPrompt(IDialogContext context, IAwaitable<string> result)
        {
            // Set the text of the note
            noteToCreate.Text = await result;
            
            await context.PostAsync($"Created note **{this.noteToCreate.Title}** that says \"{this.noteToCreate.Text}\".");
            
            context.Wait(MessageReceived);
        }
```

### <a name="handle-the-notereadaloud-intent"></a>Control de la intención Note.ReadAloud
El bot puede usar la intención `Note.ReadAloud` para mostrar el contenido de una nota o de todas las notas, si no se encuentra el título de la nota.

Pegue el código siguiente en la clase `BasicLuisDialog`.
```cs
        [LuisIntent("Note.ReadAloud")]
        public async Task NoteReadAloudIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                await context.PostAsync($"**{note.Title}**: {note.Text}.");
            }
            else
            {
                // Print out all the notes if no specific note name was detected
                string NoteList = "Here's the list of all notes: \n\n";
                foreach (KeyValuePair<string, Note> entry in noteByTitle)
                {
                    Note noteInList = entry.Value;
                    NoteList += $"**{noteInList.Title}**: {noteInList.Text}.\n\n";
                }
                await context.PostAsync(NoteList);
            }

            context.Wait(MessageReceived);
        }
        
        public bool TryFindNote(string noteTitle, out Note note)
        {
            bool foundNote = this.noteByTitle.TryGetValue(noteTitle, out note); // TryGetValue returns false if no match is found.
            return foundNote;
        }
        
        public bool TryFindNote(LuisResult result, out Note note)
        {
            note = null;

            string titleToFind;

            EntityRecommendation title;
            if (result.TryFindEntity(Entity_Note_Title, out title))
            {
                titleToFind = title.Entity;
            }
            else
            {
                titleToFind = DefaultNoteTitle;
            }

            return this.noteByTitle.TryGetValue(titleToFind, out note); // TryGetValue returns false if no match is found.
        }
```

### <a name="handle-the-notedelete-intent"></a>Control de la intención Note.Delete
Pegue el código siguiente en la clase `BasicLuisDialog`.

```cs
        [LuisIntent("Note.Delete")]
        public async Task NoteDeleteIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {                             
                // Prompt the user for a note title
                PromptDialog.Text(context, After_DeleteTitlePrompt, "What is the title of the note you want to delete?");                         
            }           
        }

        private async Task After_DeleteTitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            Note note;
            string titleToDelete = await result;
            bool foundNote = this.noteByTitle.TryGetValue(titleToDelete, out note);

            if (foundNote)
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {
                await context.PostAsync($"Did not find note named {titleToDelete}.");
            }

            context.Wait(MessageReceived);
        }
```

## <a name="build-the-bot"></a>Compilar el bot
Haga clic con el botón derecho en **build.cmd** en el editor de código y elija **Ejecutar desde la consola**.

   ![Ejecutar build.cmd](../media/bot-builder-dotnet-use-luis/bot-service-run-console.png)

## <a name="test-the-bot"></a>Probar el bot

En Azure Portal, haga clic en **Test in Web Chat** (Probar en Chat en web) para probar el bot. Intente escribir mensajes como "Crear una nota", "Leer mis notas" y "Eliminar notas".
   ![Probar el bot de notas en Chat en web](../media/bot-builder-dotnet-use-luis/bot-service-test-notebot.png)

> [!TIP]
> Si ve que el bot no reconoce siempre la intención o las entidades correctas, proporcione a la aplicación de LUIS más ejemplos de expresiones para entrenarla y mejorar su rendimiento. Puede volver a entrenar la aplicación de LUIS sin modificar el código del bot. Vea [Add example utterances](/azure/cognitive-services/LUIS/add-example-utterances) (Agregar expresiones de ejemplo) y [Train and test your LUIS app](/azure/cognitive-services/LUIS/train-test) (Entrenar y probar la aplicación de LUIS).

> [!TIP]
> Si se ejecuta el código del bot en un problema, compruebe lo siguiente:
> * Ha [compilado el bot](./bot-builder-dotnet-luis-dialogs.md#build-the-bot).
> * El código del bot define un controlador para cada intención en la aplicación de LUIS.

## <a name="next-steps"></a>Pasos siguientes

Al probar el bot, puede observar como una intención de LUIS invoca las tareas. Sin embargo, este sencillo ejemplo no permite interrumpir el diálogo activo actualmente. Permitir y controlar interrupciones como "ayuda" o "cancelar" es un diseño flexible que representa lo que el usuario hace realmente. Obtenga más información sobre el uso de diálogos puntuables, para que los diálogos puedan controlar las interrupciones.

> [!div class="nextstepaction"]
> [Controladores de mensajes globales mediante diálogos puntuables](bot-builder-dotnet-scorable-dialogs.md)

## <a name="additional-resources"></a>Recursos adicionales

- [Diálogos](bot-builder-dotnet-dialogs.md)
- [Administración del flujo de conversación con diálogos](bot-builder-dotnet-manage-conversation-flow.md)
- <a href="https://www.luis.ai" target="_blank">LUIS</a>
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>

[LUIS]: https://www.luis.ai/
[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample
[NotesSampleJSON]: https://github.com/Microsoft/BotFramework-Samples/blob/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample/Notes.json
