---
title: Administración del bot mediante herramientas de Bot Builder
description: Las herramientas de Bot Builder le permiten administrar los recursos de bot directamente desde la línea de comandos
keywords: plantillas de botbuilder, ludown, qna, luis, msbot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ef57cdf6a202679f9fc3a83e3e44640b43adb67f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998368"
---
# <a name="bot-builder-tools"></a>Herramientas de Bot Builder

Las [herramientas][cliTools] de Bot Builder abarcan el flujo de trabajo completo de desarrollo de bots que incluye las fases de planeación, compilación, prueba, publicación, conexión y evaluación. Veamos cómo estas herramientas pueden ayudarle con cada fase del ciclo de desarrollo.

[Plan](#plan)
- Comience por revisar las [directrices de diseño](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-design-principles) del bot para conocer los procedimientos recomendados
- Cree conversaciones ficticias mediante la herramienta [Chatdown](#create-mock-conversations-using-chatdown)

[Compilar](#build)
- Arranque de Language Understanding con [Ludown](#bootstrap-language-understanding-with-ludown)
- Realice un seguimiento de las referencias de servicio mediante [MSBot](#keep-track-of-service-references-using-bot-file)
- Cree y administre aplicaciones de LUIS mediante la [CLI de LUIS](#create-and-manage-luis-applications-using-luis-cli)
- Cree una base de conocimiento de QnA Maker mediante la [CLI de QnA Maker](#create-qna-maker-kb-using-qna-maker-cli)
- Cree un modelo de distribución mediante la [CLI de Dispatch](#create-dipsatch-model-using-dispatch-cli)

[Prueba](#test)
- Pruebe el bot mediante [Bot Framework Emulator V4](https://aka.ms/bot-framework-emulator-v4-overview)

[Publicar](#publish)
- Cree, descargue y publique los bots en Azure Bot Service mediante la [CLI de Azure][azureCli]

[Conexión](#configure-channels)
- Conecte el bot a los canales de Azure Bot Service mediante la [CLI de Azure][azureCli]

## <a name="plan"></a>Plan

### <a name="create-mock-conversations-using-chatdown"></a>Creación de conversaciones ficticias mediante Chatdown

ChatDown es un generador de transcripciones que usa un archivo .chat para generar transcripciones ficticias. Los archivos de transcripción ficticios generados son la salida de stdout.

Un buen bot, al igual que cualquier aplicación o sitio web correcto, se inicia con claridad en escenarios admitidos. La creación de conversaciones ficticias entre el bot y el usuario es útil para:

- Delimitar los escenarios que admite el bot
- Para que los responsables de las empresas revisen y proporcionen comentarios.
- Definir una "ruta ideal" (así como otras rutas de acceso) a través de flujos de conversación entre un usuario y un bot. El formato de archivo .chat le ayuda a crear conversaciones ficticias entre un usuario y un bot. La CLI de Chatdown convierte archivos .chat en transcripciones de conversaciones (archivos .transcript) que se pueden visualizar en [Bot Framework Emulator V4](https://github.com/microsoft/botframework-emulator).

Este es un archivo `.chat` de ejemplo:

```markdown
user=Joe
bot=LulaBot

bot: Hi!
user: yo!
bot: [Typing][Delay=3000]
Greetings!
What would you like to do?
* update - You can update your account
* List - You can list your data
* help - you can get help

user: I need the bot framework logo.

bot:
Here you go.
[Attachment=bot-framework.png]
[Attachment=http://yahoo.com/bot-framework.png]
[AttachmentLayout=carousel]

user: thanks
bot:
Here's a form for you
[Attachment=card.json adaptivecard]

```

### <a name="create-a-transcript-file-from-chat-file"></a>Creación de un archivo de transcripción a partir de un archivo .chat
Un comando de Chatdown tiene el siguiente aspecto:

```bash
chatdown sample.chat > sample.transcript
```

Este comando usará `sample.chat` y la salida será `sample.transcript`. Consulte [CLI de Chatdown][chatdown] para más información.

## <a name="build"></a>Compilación
### <a name="create-a-luis-application-with-ludown"></a>Creación de una aplicación de LUIS con LUDown
La herramienta LUDown se puede usar para crear modelos de .json para LUIS y QnA.  
Puede definir [intenciones](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents) y [entidades](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities) para una aplicación de LUIS, tal como lo haría desde el portal de LUIS.

"#\<nombre-de-intención\>" describe la sección en que se define una nueva intención. Todas las líneas siguientes enumeran las [expresiones](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances) que describen dicha intención.

Por ejemplo, puede crear varias intenciones LUIS en un único archivo .lu de esta forma: 

```LUDown
# Greeting
- Hi
- Hello
- Good morning
- Good evening

# Help
- help
- I need help
- please help
```

### <a name="create-qna-pairs-with-ludown"></a>Creación de pares de QnA con LUDown

El formato de archivo .lu también admite pares de QnA mediante la notación siguiente: 

```LUDown
> comment
### ? question ?
  ```markdown
    answer
  ```

La herramienta LUDown separará automáticamente la pregunta y las respuestas en un archivo JSON de QnA Maker que, después, se puede usar para crear una nueva base de conocimiento [QnaMaker.ai](http://qnamaker.ai).

```LUDown
### ? How do I change the default message for QnA Maker?
  ```markdown
  You can change the default message if you use the QnAMakerDialog. 
  See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
  ```

También se pueden agregar varias preguntas a la misma respuesta si se agregan nuevas líneas de variaciones de preguntas para una sola respuesta.

```LUDown
### ? What is your name?
- What should I call you?
  ```markdown
    I'm the echoBot! Nice to meet you.
  ```

### <a name="generate-json-models-with-ludown"></a>Generación de modelos .json con LUDown

Después de definir los componentes del lenguaje de LUIS o QnA en formato .lu, puede publicarlos en un archivo .json de LUIS, o bien .json o .tsv de QnA. Cuando se ejecute, la herramienta LUDown buscará los archivos .lu en el mismo directorio de trabajo para analizarlos. Dado que la herramienta LUDown puede tener como destino LUIS o QnA con archivos .lu, solo hay que especificar para qué servicio de lenguaje se va a generar mediante el comando general **ludown parse <Service> --in <luFile>**. 

En el directorio de trabajo de ejemplo, hay dos archivos .lu para analizar: "1.lu", para crear el modelo de LUIS, y "qna1.lu", para crear una base de conocimiento de QnA.

#### <a name="generate-luis-json-models"></a>Generación de modelos .json para LUIS

Para generar un modelo de LUIS mediante LUDown, escriba lo siguiente en el directorio de trabajo actual:

```shell
ludown parse ToLuis --in <luFile>
```

#### <a name="generate-qna-knowledge-base"></a>Generación de la base de conocimiento de QnA

De forma similar, para crear una base de conocimiento de QnA, solo hay que cambiar el destino del análisis.

```shell
ludown parse ToQna --in <luFile> 
```

LUIS y QnA pueden usar los archivos JSON resultantes a través de sus portales correspondientes, o bien a través de las nuevas herramientas de la CLI.

Consulte el repositorio de la [CLI de LUdown][ludown] de GitHub para más información.
## <a name="track-service-references-using-bot-file"></a>Realización de un seguimiento de las referencias de servicio mediante un archivo .bot

La nueva herramienta [MSBot][msbotCli] permite crear un archivo **.bot**, en el que se almacenan metadatos sobre otros servicios que consume el bot, todo en una sola ubicación. Este archivo también permite que el bot se conecte a estos servicios desde la CLI. La herramienta está disponible como un módulo npm; para instalarlo, ejecute lo siguiente:

```shell
npm install -g msbot
```

Para crear un archivo bot desde la CLI, escriba **msbot init** seguido del nombre del bot y la dirección URL del punto de conexión de destino, por ejemplo:

```shell
msbot init --name TestBot --endpoint http://localhost:9499/api/messages
```
Para conectar el bot a un servicio, escriba **msbot connect** en la CLI, seguido del servicio adecuado:

```shell
msbot connect [Service]
```

Para obtener la lista de servicios admitidos, consulte el archivo [Léame][msbotCli].

## <a name="create-and-manage-luis-applications-using-luis-cli"></a>Creación y administración de aplicaciones de LUIS mediante la CLI de LUIS

En el nuevo conjunto de herramientas se incluye una [extensión de LUIS][luisCli] que permite administrar de forma independiente los recursos de LUIS. Está disponible como módulo npm que se puede descargar de la siguiente manera:

```shell
npm install -g luis-apis
```
El uso de comandos básicos para la herramienta de LUIS desde la CLI es el siguiente:

```shell
luis <action> <resource> <args...>
```
Para conectar el bot a LUIS, deberá crear un archivo **.luisrc**. Se trata de un archivo de configuración que aprovisiona el identificador y la contraseña de la aplicación de LUIS para el punto de conexión de servicio cuando la aplicación realiza llamadas salientes. Puede crear este archivo mediante la ejecución de **luis init** como se muestra a continuación:

```shell
luis init
```
En el terminal, se le pedirá que escriba la clave de creación de LUIS, la región y el identificador de la aplicación antes de que la herramienta genere el archivo.  

Una vez que se genere este archivo, la aplicación podrá utilizar el archivo .json de LUIS (generado desde LUDown) mediante el comando siguiente de la CLI.

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```
Consulte el repositorio de la [CLI de LUIS][luisCli] de GitHub para más información.

## <a name="create-qna-maker-kb-using-qna-maker-cli"></a>Creación de una base de conocimiento de QnA Maker mediante la CLI de QnA Maker

En el nuevo conjunto de herramientas se incluye una [extensión de QnA][qnaCli] que permite administrar de forma independiente los recursos de LUIS. Está disponible como módulo npm que se puede descargar.

```shell
npm install -g qnamaker
```
Con la herramienta QnA Maker, puede crear, actualizar, publicar, eliminar y entrenar la base de conocimiento. Puede usar archivos generados mediante el comando [ludown parse toqna](#generate-qna-knowledge-base) para crear o reemplazar una base de conocimiento.

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

Consulte el repositorio de la [CLI de QnA Maker][qnaCli] de GitHub para más información.

## <a name="create-dispatch-model-using-dispatch-cli"></a>Creación de un modelo de distribución mediante la CLI de Dispatch

Dispatch es una herramienta para crear y evaluar modelos de LUIS que se usan para distribuir intenciones en varios módulos de bot como, por ejemplo, modelos de LUIS, bases de conocimiento de QnA, etc (que se agregan para su distribución como un tipo de archivo).

Use el modelo de Dispatch en los siguientes casos:

- El bot consta de varios módulos y usted necesita ayuda para enrutar expresiones del usuario a estos módulos y evaluar la integración del bot.
- Evalúe la calidad de la clasificación de intenciones de un único modelo de LUIS.
- Cree un modelo de clasificación de texto a partir de los archivos de texto.

Una vez que haya montado el archivo .bot con las [aplicaciones de LUIS][msbotCli-luis] y las [bases de conocimiento de QnA Maker][msbotCli-qna] en las que se basa el bot, puede simplemente crear un modelo de distribución mediante: 

```shell
dispatch create -b <YOUR-BOT-FILE> | msbot connect dispatch --stdin
```
Consulte la [CLI de Dispatch][dispatchCli] para más información.

## <a name="test"></a>Prueba

[Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) es una aplicación de escritorio que permite que los desarrolladores de bots prueben y depuren sus bots en localhost o mediante la ejecución remota mediante un túnel.

## <a name="publish"></a>Publicar

Puede usar la [CLI de Azure][azureCli] para [crear](#create-azure-bot-service-bot), [descargar](#download-azure-bot-service-bot) y [publicar](#publish-azure-bot-service-bot) su bot en Azure Bot Service. Instale la extensión del bot mediante: 
```shell
az extension add -n botservice
```

## <a name="create-azure-bot-service-bot"></a>Creación de un bot en Azure Bot Service

Inicie sesión en su cuenta de Azure mediante 
```shell
az login
```

Una vez que haya iniciado sesión, puede crear un nuevo bot mediante Azure Bot Service: 
```shell
az bot create [options]
```

Para crear un bot y actualizar el archivo .bot con la configuración de bot,  
```shell
az bot create [options] --msbot | msbot connect bot --stdin
```

Si ya tiene un bot existente,  
```shell
az bot show [options] --msbot | msbot connect bot --stdin
```

| Opción                            | DESCRIPCIÓN                                   |
|-----------------------------------|-----------------------------------------------|
| --kind -k [Obligatorio]              | La variante del bot.  Valores permitidos: function, registration, webapp.|
| --name -n [Obligatorio]              | El nombre de recurso del bot. |
| --appid                           | El identificador de cuenta msa que se va a usar con el bot.   |
| --location -l                     | Ubicación. Puede configurar la ubicación predeterminada mediante `az configure --defaults location=<location>`.  Valor predeterminado: westus.|
| --msbot                           | Muestra la salida como un json compatible con un archivo .bot.  Valores permitidos: false, true.|
| --password -p                     | La contraseña de msa para el bot del portal de desarrolladores. |
| --resource-group -g               | Nombre del grupo de recursos. Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`.  Predeterminado: build2018. |
| --tags                            | Conjunto de etiquetas para agregar al bot. |


### <a name="configure-channels"></a>Configuración de canales

Puede usar la CLI de Azure para administrar los canales del bot. 
```shell
>az bot -h
Group
   az bot: Manage Bot Services.
    Subgroups:
        directline: Manage Directline Channel on a Bot.
        email     : Manage Email Channel on a Bot.
        facebook  : Manage Facebook Channel on a Bot.
        kik       : Manage Kik Channel on a Bot.
        msteams   : Manage Msteams Channel on a Bot.
        skype     : Manage Skype Channel on a Bot.
        slack     : Manage Slack Channel on a Bot.
        sms       : Manage Sms Channel on a Bot.
        telegram  : Manage Telegram Channel on a Bot.
        webchat   : Manage Webchat Channel on a Bot.

    Commands:
        create    : Create a new Bot Service.
        delete    : Delete an existing Bot Service.
        download  : Download an existing Bot Service.
        publish   : Publish to an existing Bot Service.
        show      : Get an existing Bot Service.
        update    : Update an existing Bot Service.

```

## <a name="additional-information"></a>Información adicional
- [Herramientas de Bot Builder][cliTools]

<!-- Footnote links -->

[cliTools]: https://aka.ms/botbuilder-tools-readme
[azureCli]: https://aka.ms/botbuilder-tools-azureCli
[msbotCli]: https://aka.ms/botbuilder-tools-msbot-readme
[msbotCli-luis]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-luis-application
[msbotCli-qna]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-qna-maker-knowledge-base
[chatdown]: https://aka.ms/botbuilder-tools-chatdown
[ludown]: https://aka.ms/botbuilder-ludown
[luisCli]: https://aka.ms/botbuilder-luis-cli
[qnaCli]: https://aka.ms/botbuilder-tools-qnaMaker
[dispatchCli]: https://aka.ms/botbuilder-tools-dispatch
