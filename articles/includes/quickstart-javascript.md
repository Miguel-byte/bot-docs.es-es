## <a name="prerequisites"></a>Requisitos previos

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- [Yeoman](http://yeoman.io/), que usa un generador para crear un bot de forma automática.
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)
- Conocimientos sobre [restify](http://restify.com/) y programación asincrónica en JavaScript.

> [!NOTE]
> La instalación de las herramientas de compilación de Windows que se indican a continuación solo se necesitan si se usa Windows como sistema operativo de desarrollo. En algunas instalaciones, el paso de instalación de restify genera un error relacionado con node-gyp.
> Si este es el caso, intente ejecutar este comando con permisos elevados.
> Esta llamada también puede bloquearse si Python ya está instalado en el sistema:
> ```bash
> npm install -g windows-build-tools
> ```

## <a name="create-a-bot"></a>Creación de un bot

Creación del bot e inicialización de sus paquetes

1. Abra un terminal o un símbolo del sistema con privilegios elevados.
1. Si aún no tiene un directorio para sus bots de JavaScript, cree uno y mueva los directorios a él. (Vamos a crear un directorio para sus bots de JavaScript en general, aunque solo se va a crear un bot en este tutorial).

   ```bash
   mkdir myJsBots
   cd myJsBots
   ```

1. Asegúrese de que la versión de npm está actualizada.

   ```bash
   npm install -g npm
   ```

1. Después, instale Yeoman y el generador para JavaScript.

   ```bash
   npm install -g yo generator-botbuilder
   ```

1. Luego, use el generador para crear un bot de eco.

   ```bash
   yo botbuilder
   ```

Yeoman le solicitará alguna información con la que se va a crear el bot. En este tutorial, use los valores predeterminados.

- Escriba un nombre para el bot. (myChatBot)
- Escriba una descripción. (Demostrar las capacidades básicas de Microsoft Bot Framework)
- Elija el idioma del bot. (JavaScript)
- Elija la plantilla que se va a usar. (Echo)

Gracias a la plantilla, el proyecto contiene todo el código necesario para crear el bot en esta guía de inicio rápido. Realmente no necesita escribir ningún código adicional.

> [!NOTE]
> Si decide crear un bot `Basic`, necesitará un modelo de lenguaje LUIS. Puede crear uno en [luis.ai](https://www.luis.ai). Después de crear el modelo, actualice el archivo .bot. El archivo del bot deb tener un aspecto similar a [este](../v4sdk/bot-builder-service-file.md).

## <a name="start-your-bot"></a>Inicio del bot

En un terminal o símbolo del sistema, mueva los directorios al que creó para el bot e inícielo con `npm start`. En este momento, el bot se ejecuta de forma local.

## <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot

1. Inicie Bot Framework Emulator.
2. Haga clic en el vínculo **Open bot** (Abrir bot) de la pestaña de bienvenida del emulador.
3. Seleccione el archivo .bot ubicado en el directorio donde se creó el proyecto.

Envíe un mensaje al bot y este responderá con un mensaje.
![Emulador en ejecución](../media/emulator-v4/js-quickstart.png)
