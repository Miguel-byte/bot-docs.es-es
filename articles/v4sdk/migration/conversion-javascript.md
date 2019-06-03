---
title: Migración de un bot de JavaScript v3 existente a un nuevo proyecto de v4 | Microsoft Docs
description: Tomamos un bot de JavaScript v3 existente y lo migramos al SDK v4 utilizando un nuevo proyecto.
keywords: JavaScript, bot migration, dialogs, v3 bot
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 591f58e1cefca576e2e3e4a486ecc6fbe0a6b0e4
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215606"
---
# <a name="migrate-a-sdk-v3-javascript-bot-to-v4"></a>Migración de un bot de Javascript del SDK v3 a v4

En este artículo, se portará el bot [core-MultiDialogs-v3](https://aka.ms/v3-js-core-multidialog-migration-sample) del SDK para JavaScript v3 a un nuevo bot de JavaScript v4.
Esta conversión se divide en estas fases:

1. Crear el nuevo proyecto y agregar las dependencias.
1. Actualizar el punto de entrada y definir las constantes.
1. Crear los diálogos y volver a implementarlos con el SDK v4.
1. Actualizar el código del bot para ejecutar los diálogos.
1. Portar el archivo de utilidad **store.js**.

Al final de este proceso tendremos un bot v4 en funcionamiento. También hay una copia del bot convertido en el repositorio de ejemplos, [core-MultiDialogs-v4](https://aka.ms/v4-js-core-multidialog-migration-sample).

El SDK Bot Framework v4 se basa en las mismas API REST subyacentes que el SDK v3. Sin embargo, el SDK v4 es una refactorización de la versión anterior que ofrece a los desarrolladores más flexibilidad y control sobre sus bots. Algunos cambios importantes en el SDK son:

- El estado se administra mediante objetos de administración de estado y descriptores de acceso de propiedad.
- Ha cambiado el procedimiento de control de turnos, es decir, el modo en el que el bot recibe y responde a una actividad de entrada del canal del usuario.
- V4 no utiliza un objeto `session`, en su lugar, tiene un objeto de _contexto de turno_ que contiene información acerca de la actividad de entrada y se puede usar para enviar de vuelta una actividad de respuesta.
- Una nueva biblioteca de diálogos muy diferente de la existente en la versión v3. Deberá convertir los diálogos antiguos al nuevo sistema de diálogos, con diálogos de componente y en cascada.

<!-- TODO
For more information about specific changes, see [differences between the v3 and v4 JavaScript SDK](???.md).
-->

> [!NOTE]
> Como parte de la migración, también se hace limpieza en parte del código, pero solo destacaremos los cambios realizados en la lógica de la versión v3 como parte del proceso de migración.

## <a name="prerequisites"></a>Requisitos previos

- Node.js
- Visual Studio Code
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)

## <a name="about-this-bot"></a>Acerca de este bot

El bot que estamos migrando muestra el uso de varios diálogos para administrar el flujo de conversación. El bot puede buscar información de vuelos y hoteles.

- El diálogo principal pregunta al usuario qué tipo de información está buscando.
- El diálogo de hoteles pide al usuario los parámetros de búsqueda y, a continuación, realiza una búsqueda simulada.
- El diálogo de vuelos genera un error que el bot detecta y trata de forma correcta.

## <a name="create-and-open-a-new-v4-bot-project"></a>Creación y apertura de un nuevo proyecto de bot v4

1. Necesitará un proyecto de la versión v4 al que se va a portar el código del bot. Para crear un proyecto de forma local, consulte [Creación de un bot con el SDK Bot Framework para JavaScript](../../javascript/bot-builder-javascript-quickstart.md).

    > [!TIP]
    > También puede crear un proyecto en Azure; para ello, consulte [Creación de un bot con Azure Bot Service](../../bot-service-quickstart.md).
    > Sin embargo, estos dos métodos tienen una pequeña diferencia en los archivos auxiliares. El proyecto de la versión v4 de este artículo se creó como un proyecto local.

1. A continuación, abra el proyecto en Visual Studio Code.

## <a name="update-the-packagejson-file"></a>Actualización del archivo package.json

1. Para agregar una dependencia del paquete **botbuilder-dialogs**, escriba `npm i botbuilder-dialogs` en la ventana de terminal de Visual Studio Code.

1. Edite **./package.json** y actualice `name`, `version`, `description` y otras propiedades como desee.

## <a name="update-the-v4-app-entry-point"></a>Actualización del punto de entrada de la aplicación de la versión v4

La plantilla de la versión v4 crea un archivo **index.js** para el punto de entrada de la aplicación y un archivo **bot.js** para la lógica específica del bot. En pasos posteriores, cambiaremos el nombre del archivo **bot.js** a **bots/reservationBot.js** y agregaremos una clase para cada diálogo.

Edite **./index.js**, que es el punto de entrada para la aplicación del bot. Va a contener las partes del archivo **app.js** de la versión v3 que configuran el servidor HTTP.

1. Además de `BotFrameworkAdapter`, importe `MemoryStorage` y `ConversationState` desde el paquete **botbuilder**. Importe también los módulos del bot y el diálogo principal. (Los vamos a crear pronto, pero tenemos que hacer referencia a ellos aquí).

    ```javascript
    // Import required bot services.
    // See https://aka.ms/bot-services to learn more about the different parts of a bot.
    const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');

    // This bot's main dialog.
    const { MainDialog } = require('./dialogs/main')
    const { ReservationBot } = require('./bots/reservationBot');
    ```

1. Defina un controlador `onTurnError` para el adaptador.

    ```javascript
    // Catch-all for errors.
    adapter.onTurnError = async (context, error) => {
        const errorMsg = error.message ? error.message : `Oops. Something went wrong!`;
        // This check writes out errors to console log .vs. app insights.
        console.error(`\n [onTurnError]: ${ error }`);
        // Clear out state
        await conversationState.delete(context);
        // Send a message to the user
        await context.sendActivity(errorMsg);
    };
    ```

    En la versión v4, usamos un _adaptador de bot_ para enrutar las actividades entrantes al bot. El adaptador nos permite detectar y reaccionar ante los errores antes de que finalice un turno. En este caso, si se produce un error de aplicación, se borra el estado de la conversación, lo que restablecerá todos los diálogos y evitará que el bot se quede en un estado de la conversación dañado.

1. Reemplace el código de la plantilla para crear el bot con este.

    ```javascript
    // Define state store for your bot.
    const memoryStorage = new MemoryStorage();

    // Create conversation state with in-memory storage provider.
    const conversationState = new ConversationState(memoryStorage);

    // Create the base dialog and bot
    const dialog = new MainDialog();
    const reservationBot = new ReservationBot(conversationState, dialog);
    ```

    Ahora la clase `MemoryStorage` proporciona la capa de almacenamiento en memoria y tenemos que crear explícitamente un objeto de administración de estado de la conversación.

    El código de definición del diálogo se ha movido a una clase `MainDialog` que se definirá en breve. También se va a migrar el código de definición del bot a una clase `ReservationBot`.

1. Por último, actualizamos el controlador de solicitudes del servidor para que use el adaptador para enrutar las actividades al bot.

    ```javascript
    // Listen for incoming requests.
    server.post('/api/messages', (req, res) => {
        adapter.processActivity(req, res, async (context) => {
            // Route incoming activities to the bot.
            await reservationBot.run(context);
        });
    });
    ```

    En la versión v4, el bot se deriva de `ActivityHandler`, que define el método `run` para recibir una actividad para un turno.

## <a name="add-a-constants-file"></a>Adición de un archivo de constantes

Cree un archivo **./const.js** para contener los identificadores del bot.

```javascript
module.exports = {
    MAIN_DIALOG: 'mainDialog',
    INITIAL_PROMPT: 'initialPrompt',
    HOTELS_DIALOG: 'hotelsDialog',
    INITIAL_HOTEL_PROMPT: 'initialHotelPrompt',
    CHECKIN_DATETIME_PROMPT: 'checkinTimePrompt',
    HOW_MANY_NIGHTS_PROMPT: 'howManyNightsPrompt',
    FLIGHTS_DIALOG: 'flightsDialog',
};
```

En la versión v4, los identificadores se asignan a objetos de diálogos y de avisos y los diálogos y avisos se invocan por identificador.

## <a name="create-new-dialog-files"></a>Creación de los nuevos archivos de diálogos

Cree estos archivos:

| Nombre de archivo | DESCRIPCIÓN |
|:---|:---|
| **./dialogs/flights.js** | Contendrá la lógica migrada del diálogo `hotels`. |
| **./dialogs/hotels.js** | Contendrá la lógica migrada del diálogo `flights`. |
| **./dialogs/main.js** | Contendrá la lógica migrada del bot y sustituirá el diálogo _root_. |

No hemos migrado el diálogo auxiliar. Para obtener un ejemplo de cómo implementar un diálogo de ayuda en la versión v4, consulte [Control de las interrupciones del usuario](../bot-builder-howto-handle-user-interrupt.md?tabs=javascript).

### <a name="implement-the-main-dialog"></a>Implementación del diálogo principal

En la versión v3, los bots se creaban sobre un sistema de diálogos. En la versión v4, la lógica de bot y la del diálogo ahora son independientes. Hemos tomado lo que era el _diálogo raíz_ en el bot de la versión v3 y creado una clase `MainDialog` para ocupar su lugar.

Edite **./dialogs/main.js**.

1. Importe las clases y las constantes que necesitamos para el diálogo.

    ```javascript
    const { DialogSet, DialogTurnStatus, ComponentDialog, WaterfallDialog,
        ChoicePrompt } = require('botbuilder-dialogs');
    const { FlightDialog } = require('./flights');
    const { HotelsDialog } = require('./hotels');
    const { MAIN_DIALOG,
        INITIAL_PROMPT,
        HOTELS_DIALOG,
        FLIGHTS_DIALOG
    } = require('../const');
    ```

1. Defina y exporte la clase `MainDialog`.

    ```javascript
    const initialId = 'mainWaterfallDialog';

    class MainDialog extends ComponentDialog {
        constructor() {
            super(MAIN_DIALOG);

            // Create a dialog set for the bot. It requires a DialogState accessor, with which
            // to retrieve the dialog state from the turn context.
            this.addDialog(new ChoicePrompt(INITIAL_PROMPT, this.validateNumberOfAttempts.bind(this)));
            this.addDialog(new FlightDialog(FLIGHTS_DIALOG));

            // Define the steps of the base waterfall dialog and add it to the set.
            this.addDialog(new WaterfallDialog(initialId, [
                this.promptForBaseChoice.bind(this),
                this.respondToBaseChoice.bind(this)
            ]));

            // Define the steps of the hotels waterfall dialog and add it to the set.
            this.addDialog(new HotelsDialog(HOTELS_DIALOG));

            this.initialDialogId = initialId;
        }
    }

    module.exports.MainDialog = MainDialog;
    ```

    Esto declara los otros diálogos y avisos a los que el diálogo principal hace referencia directamente.

    - El diálogo en cascada principal que contiene los pasos de este diálogo. Cuando se inicia el diálogo de componente, se inicia su _diálogo inicial_.
    - El aviso de elección que vamos a usar para pedir al usuario qué tarea desea realizar. Hemos creado el aviso de elección con un validador.
    - Los dos diálogos secundarios, vuelos y hoteles.

1. Agregue un método auxiliar `run` a la clase.

    ```javascript
    /**
     * The run method handles the incoming activity (in the form of a TurnContext) and passes it through the dialog system.
     * If no dialog is active, it will start the default dialog.
     * @param {*} turnContext
     * @param {*} accessor
     */
    async run(turnContext, accessor) {
        const dialogSet = new DialogSet(accessor);
        dialogSet.add(this);

        const dialogContext = await dialogSet.createContext(turnContext);
        const results = await dialogContext.continueDialog();
        if (results.status === DialogTurnStatus.empty) {
            await dialogContext.beginDialog(this.id);
        }
    }
    ```

    En la versión v4, un bot interactúa con el sistema de diálogos mediante la creación de un contexto de diálogo en primer lugar y, a continuación, una llamada a `continueDialog`. Si hay un diálogo activo, se le pasa el control; en caso contrario, se devuelve la llamada. Un resultado `empty` indica que no estaba activo ningún diálogo y se inicia de nuevo el diálogo principal.

    El parámetro `accessor` pasa el descriptor de acceso de la propiedad de estado del diálogo. El estado de la _pila de diálogos_ se almacena en esta propiedad. Para más información acerca de cómo funcionan el estado y los diálogos en la versión v4, consulte [Administración del estado](../bot-builder-concept-state.md) y [Biblioteca de diálogos](../bot-builder-concept-dialog.md), respectivamente.

1. Para la clase, agregue los pasos de cascada del diálogo principal y el validador para el aviso de elección.

    ```javascript
    async promptForBaseChoice(stepContext) {
        return await stepContext.prompt(
            INITIAL_PROMPT, {
                prompt: 'Are you looking for a flight or a hotel?',
                choices: ['Hotel', 'Flight'],
                retryPrompt: 'Not a valid option'
            }
        );
    }

    async respondToBaseChoice(stepContext) {
        // Retrieve the user input.
        const answer = stepContext.result.value;
        if (!answer) {
            // exhausted attempts and no selection, start over
            await stepContext.context.sendActivity('Not a valid option. We\'ll restart the dialog ' +
                'so you can try again!');
            return await stepContext.endDialog();
        }
        if (answer === 'Hotel') {
            return await stepContext.beginDialog(HOTELS_DIALOG);
        }
        if (answer === 'Flight') {
            return await stepContext.beginDialog(FLIGHTS_DIALOG);
        }
        return await stepContext.endDialog();
    }

    async validateNumberOfAttempts(promptContext) {
        if (promptContext.attemptCount > 3) {
            // cancel everything
            await promptContext.context.sendActivity('Oops! Too many attempts :( But don\'t worry, I\'m ' +
                'handling that exception and you can try again!');
            return await promptContext.context.endDialog();
        }

        if (!promptContext.recognized.succeeded) {
            await promptContext.context.sendActivity(promptContext.options.retryPrompt);
            return false;
        }
        return true;
    }
    ```

    El primer paso de la cascada pide al usuario realizar una selección, iniciando el aviso de elección, que es en sí mismo un diálogo. El segundo paso de la cascada consume el resultado del aviso de elección. O bien inicia un diálogo secundario (si se ha realizado una elección) o finaliza el diálogo principal (si el usuario no pudo realizar una selección).

    El aviso de elección devuelve la elección del usuario, si eligió una opción válida o volverá a solicitar al usuario que realice la elección. El validador comprueba el número de veces que se ha realizado el aviso al usuario y permite al aviso producir un error después de 3 intentos incorrectos, devolviendo el control al diálogo en cascada principal.

### <a name="implement-the-flights-dialog"></a>Implementación del diálogo de vuelos

En el bot de la versión v3, el diálogo de vuelos era un código auxiliar para mostrar cómo controla el bot un error de conversación. En este caso, hacemos lo mismo.

Edite **./dialogs/flights.js**.

```javascript
const { ComponentDialog, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'flightsWaterfallDialog';

class FlightDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(initialId, [
            async () => {
                throw new Error('Flights Dialog is not implemented and is instead ' +
                    'being used to show Bot error handling');
            }
        ]));
    }
}

exports.FlightDialog = FlightDialog;
```

### <a name="implement-the-hotels-dialog"></a>Implementación del diálogo de hoteles

Se mantiene el mismo flujo general del diálogo de hoteles: pedir un destino, solicitar una fecha, pedir el número de noches de permanencia y, a continuación, mostrar al usuario una lista de opciones que coincidan con su búsqueda.

Edite **./dialogs/hotels.js**.

1. Importe las clases y las constantes que necesitaremos para el diálogo.

    ```javascript
    const { ComponentDialog, WaterfallDialog, TextPrompt, DateTimePrompt } = require('botbuilder-dialogs');
    const { AttachmentLayoutTypes, CardFactory } = require('botbuilder');
    const store = require('../store');
    const {
        INITIAL_HOTEL_PROMPT,
        CHECKIN_DATETIME_PROMPT,
        HOW_MANY_NIGHTS_PROMPT
    } = require('../const');
    ```

1. Defina y exporte la clase `HotelsDialog`.

    ```javascript
    const initialId = 'hotelsWaterfallDialog';

    class HotelsDialog extends ComponentDialog {
        constructor(id) {
            super(id);

            // ID of the child dialog that should be started anytime the component is started.
            this.initialDialogId = initialId;

            // Register dialogs
            this.addDialog(new TextPrompt(INITIAL_HOTEL_PROMPT));
            this.addDialog(new DateTimePrompt(CHECKIN_DATETIME_PROMPT));
            this.addDialog(new TextPrompt(HOW_MANY_NIGHTS_PROMPT));

            // Define the conversation flow using a waterfall model.
            this.addDialog(new WaterfallDialog(initialId, [
                this.destinationPromptStep.bind(this),
                this.destinationSearchStep.bind(this),
                this.checkinPromptStep.bind(this),
                this.checkinTimeSetStep.bind(this),
                this.stayDurationPromptStep.bind(this),
                this.stayDurationSetStep.bind(this),
                this.hotelSearchStep.bind(this)
            ]));
        }
    }

    exports.HotelsDialog = HotelsDialog;
    ```

1. En la clase, agregue un par de funciones auxiliares que vamos a usar en los pasos del diálogo.

    ```javascript
    addDays(startDate, days) {
        const date = new Date(startDate);
        date.setDate(date.getDate() + days);
        return date;
    };

    createHotelHeroCard(hotel) {
        return CardFactory.heroCard(
            hotel.name,
            `${hotel.rating} stars. ${hotel.numberOfReviews} reviews. From ${hotel.priceStarting} per night.`,
            CardFactory.images([hotel.image]),
            CardFactory.actions([
                {
                    type: 'openUrl',
                    title: 'More details',
                    value: `https://www.bing.com/search?q=hotels+in+${encodeURIComponent(hotel.location)}`
                }
            ])
        );
    }
    ```

    `createHotelHeroCard` crea una tarjeta de imagen prominente que contiene información sobre un hotel.

1. En la clase, agregue los pasos de cascada utilizados en el diálogo.

    ```javascript
    async destinationPromptStep(stepContext) {
        await stepContext.context.sendActivity('Welcome to the Hotels finder!');
        return await stepContext.prompt(
            INITIAL_HOTEL_PROMPT, {
                prompt: 'Please enter your destination'
            }
        );
    }

    async destinationSearchStep(stepContext) {
        const destination = stepContext.result;
        stepContext.values.destination = destination;
        await stepContext.context.sendActivity(`Looking for hotels in ${destination}`);
        return stepContext.next();
    }

    async checkinPromptStep(stepContext) {
        return await stepContext.prompt(
            CHECKIN_DATETIME_PROMPT, {
                prompt: 'When do you want to check in?'
            }
        );
    }

    async checkinTimeSetStep(stepContext) {
        const checkinTime = stepContext.result[0].value;
        stepContext.values.checkinTime = checkinTime;
        return stepContext.next();
    }

    async stayDurationPromptStep(stepContext) {
        return await stepContext.prompt(
            HOW_MANY_NIGHTS_PROMPT, {
                prompt: 'How many nights do you want to stay?'
            }
        );
    }

    async stayDurationSetStep(stepContext) {
        const numberOfNights = stepContext.result;
        stepContext.values.numberOfNights = parseInt(numberOfNights);
        return stepContext.next();
    }

    async hotelSearchStep(stepContext) {
        const destination = stepContext.values.destination;
        const checkIn = new Date(stepContext.values.checkinTime);
        const checkOut = this.addDays(checkIn, stepContext.values.numberOfNights);

        await stepContext.context.sendActivity(`Ok. Searching for Hotels in ${destination} from 
            ${checkIn.toDateString()} to ${checkOut.toDateString()}...`);
        const hotels = await store.searchHotels(destination, checkIn, checkOut);
        await stepContext.context.sendActivity(`I found in total ${hotels.length} hotels for your dates:`);

        const hotelHeroCards = hotels.map(this.createHotelHeroCard);

        await stepContext.context.sendActivity({
            attachments: hotelHeroCards,
            attachmentLayout: AttachmentLayoutTypes.Carousel
        });

        return await stepContext.endDialog();
    }
    ```

    Hemos migrado los pasos del diálogo de hoteles de la versión v3 a los pasos de cascada del diálogo de hoteles de la versión v4.

## <a name="update-the-bot"></a>Actualización del bot

En la versión v4, los bots pueden reaccionar frente a actividades fuera del sistema de diálogos. La clase `ActivityHandler` define controladores para los tipos comunes de actividades, para que sea más fácil administrar el código.

Cambie el nombre de **. /bot.js** a **./bots/reservationBot.js** y edítelo.

1. El archivo ya importa **ActivityHandler**, que proporciona una implementación de base de un bot.

    ```javascript
    const { ActivityHandler } = require('botbuilder');
    ```

1. Cambie el nombre de la clase a `ReservationBot`.

    ```javascript
    class ReservationBot extends ActivityHandler {
        // ...
    }

    module.exports.ReservationBot = ReservationBot;
    ```

1. Actualice la firma del constructor, para que acepte los objetos que se reciben.

    ```javascript
    /**
     *
     * @param {ConversationState} conversationState
     * @param {Dialog} dialog
     * @param {any} logger object for logging events, defaults to console if none is provided
    */
    constructor(conversationState, dialog, logger) {
        super();
        // ...
    }
    ```

1. En el constructor, agregue la comprobación de parámetros NULL y defina las propiedades del constructor de clases.

    ```javascript
    if (!conversationState) throw new Error('[DialogBot]: Missing parameter. conversationState is required');
    if (!dialog) throw new Error('[DialogBot]: Missing parameter. dialog is required');
    if (!logger) {
        logger = console;
        logger.log('[DialogBot]: logger not passed in, defaulting to console');
    }

    this.conversationState = conversationState;
    this.dialog = dialog;
    this.logger = logger;
    this.dialogState = this.conversationState.createProperty('DialogState');
    ```

    Aquí creamos el descriptor de acceso de la propiedad de estado del diálogo que va a almacenar el estado de la pila de diálogos.

1. En el constructor, actualice el controlador `onMessage` y agregue un controlador `onDialog`.

    ```javascript
    this.onMessage(async (context, next) => {
        this.logger.log('Running dialog with Message Activity.');

        // Run the Dialog with the new message Activity.
        await this.dialog.run(context, this.dialogState);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });

    this.onDialog(async (context, next) => {
        // Save any state changes. The load happened during the execution of the Dialog.
        await this.conversationState.saveChanges(context, false);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler` enruta las actividades de mensaje a `onMessage`. Este bot controla todas las entradas de usuario mediante diálogos.

    `ActivityHandler` llama a `onDialog` al final del turno, antes de devolver el control al adaptador. Es necesario guardar explícitamente el estado antes de salir el turno. En caso contrario, no se guardarán los cambios de estado y el diálogo no se ejecutará correctamente.

1. Por último, actualice el controlador `onMembersAdded` en el constructor.

    ```javascript
    this.onMembersAdded(async (context, next) => {
        const membersAdded = context.activity.membersAdded;
        for (let cnt = 0; cnt < membersAdded.length; ++cnt) {
            if (membersAdded[cnt].id !== context.activity.recipient.id) {
                await context.sendActivity('Hello and welcome to Contoso help desk bot.');
            }
        }
        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler` llama a `onMembersAdded` cuando recibe una actividad de actualización de conversación que indica que se han sumado a la conversación participantes que no son el bot. Actualizamos este método para enviar un mensaje de bienvenida cuando un usuario se une a la conversación.

## <a name="create-the-store-file"></a>Creación del archivo de almacenamiento

Cree el archivo **./store.js**, utilizado por el diálogo de hoteles. `searchHotels` es una función de búsqueda de hoteles simulada, igual que en la versión v3 del bot.

```javascript
module.exports = {
    searchHotels: destination => {
        return new Promise(resolve => {

            // Filling the hotels results manually just for demo purposes
            const hotels = [];
            for (let i = 1; i <= 5; i++) {
                hotels.push({
                    name: `${destination} Hotel ${i}`,
                    location: destination,
                    rating: Math.ceil(Math.random() * 5),
                    numberOfReviews: Math.floor(Math.random() * 5000) + 1,
                    priceStarting: Math.floor(Math.random() * 450) + 80,
                    image: `https://placeholdit.imgix.net/~text?txtsize=35&txt=Hotel${i}&w=500&h=260`
                });
            }

            hotels.sort((a, b) => a.priceStarting - b.priceStarting);

            // complete promise with a timer to simulate async response
            setTimeout(() => { resolve(hotels); }, 1000);
        });
    }
};
```

## <a name="test-the-bot-in-the-emulator"></a>Prueba del bot en el emulador

En este momento, podremos ejecutar el bot localmente y conectarlo al emulador.

1. Ejecute el ejemplo localmente en la máquina.
    Si inicia una sesión de depuración en Visual Studio Code, se envía la información de registro a la consola de depuración a medida que prueba el bot.
1. Inicie el emulador y conéctelo al bot.
1. Envíe mensajes para probar el diálogo principal, el de vuelos y el de hoteles.

## <a name="additional-resources"></a>Recursos adicionales

Temas conceptuales de la versión v4:

- [Funcionamiento de los bots](../bot-builder-basics.md)
- [Administración de estados](../bot-builder-concept-state.md)
- [Biblioteca de cuadros de diálogo](../bot-builder-concept-dialog.md)

Temas de procedimientos de la versión v4:

- [Envío y recepción de mensajes de texto](../bot-builder-howto-send-messages.md)
- [Guardar usuario y datos de conversación](../bot-builder-howto-v4-state.md)
- [Implementación de flujo de conversación secuencial](../bot-builder-dialog-manage-conversation-flow.md)
