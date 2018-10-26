---
title: Creación de bots con la CLI de Azure
description: Las herramientas de Bot Builder permiten administrar los recursos de bot directamente desde la línea de comandos
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/31/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 96660ecb8bf7a69115e517bfa8ec97a79a3e8c90
ms.sourcegitcommit: f0b22c6286e44578c11c9f15d22b542c199f0024
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/27/2018
ms.locfileid: "47404021"
---
# <a name="create-bots-with-azure-cli"></a>Creación de bots con la CLI de Azure

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

En este tutorial, aprenderá a: 
- Crear un bot con la CLI de Azure 
- Descargar una copia local para el desarrollo
- Usar la nueva herramienta MSBot para almacenar toda la información de recursos de bot
- Administrar, crear o actualizar los modelos de LUIS y QnA con LUDown
- Conexión a servicios de LUIS y QnA Maker desde la CLI
- Implementar el bot en Azure desde la CLI

## <a name="prerequisites"></a>Requisitos previos

Para usar estas herramientas desde la línea de comandos, necesita Node.js instalado en el equipo: 
- [Node.js (v8.5 o superior)](https://nodejs.org/en/)

## <a name="1-install-tools"></a>1. Instalación de herramientas
1. [Instale](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) la versión más reciente de la CLI de Azure.
2. [Instale](https://aka.ms/botbuilder-tools-readme) las herramientas de Bot Builder.

Ahora puede administrar los bots mediante la CLI de Azure como cualquier otro recurso de Azure.

>[!TIP]
> En la actualidad, Azure Bot Extension solo admite los bots v3.
  
3. Inicie sesión en la CLI de Azure mediante la ejecución del comando siguiente.

```azurecli
az login
```
Se abrirá una ventana del explorador, lo que le permite iniciar sesión. Después de iniciar sesión, verá el mensaje siguiente:

![Inicio de sesión del dispositivo de MS](media/bot-builder-tools/az-browser-login.png)

Y en la ventana de la línea de comandos, verá la siguiente información:

![Comando de inicio de sesión de Azure](media/bot-builder-tools/az-login-command.png)

## <a name="2-create-a-new-bot-from-azure-cli"></a>2. Creación de un bot desde la CLI de Azure

Use la CLI de Azure para crear bots completamente desde la línea de comandos. 

```azurecli
az bot [command]
```
|Comandos:|  |
|----|----|
| create      |Agregar un recurso|
| delete     |Clonar un recurso|
| descargar   | Descargar el código fuente del bot|
| Publicar   |Publicar en un servicio de bot existente|
| show |Mostrar los recursos de bot existentes.|
| update| Actualizar un servicio de bot existente|

Para crear un bot desde la CLI, debe seleccionar un [grupo de recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) existente, o bien crear uno. 

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --version v3 --description "description-of-my-bot"
```
Los valores permitidos para `--kind` son: `function, registration, webapp` y para `--version` son `v3, v4`.  Después de una solicitud correcta, verá el mensaje de confirmación.
```
Obtained msa app id and password. Provisioning bot now.
```

> [!TIP]
> Si recibe un mensaje de error en el que se indica que **no se encontró el grupo de recursos**, es posible que deba establecer su [suscripción](https://docs.microsoft.com/en-us/azure/architecture/cloud-adoption-guide/adoption-intro/subscription-explainer) en la CLI de Azure. La suscripción de Azure debe coincidir con lo que se escribió al crear el grupo de recursos. Para establecerla, escriba lo siguiente:
> ```azurecli
> az account set --subscription "your-subscription-name"
> ```
> Para ver una lista de suscripciones para la cuenta, escriba el comando siguiente:
> ```azurecli
> az account list
> ```

De forma predeterminada, se crea un bot de .NET. Puede especificar el SDK de la plataforma si indica el lenguaje mediante el argumento **--lang**. En la actualidad, el paquete de extensión de bot es compatible con los SDK de C# y Node.js bot. Por ejemplo, para **crear un bot de Node.js**:

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot" --lang Node 
```
El nuevo bot de eco se aprovisionará en el grupo de recursos en Azure; para probarlo simplemente haga clic en **Probar en chat en web** bajo el encabezado de administración de bots de la vista Bot de aplicación web. 

![Bot de eco de Azure](media/bot-builder-tools/az-echo-bot.png) 

## <a name="3-download-the-bot-locally"></a>3. Descarga local del bot

Hay dos maneras de descargar el código fuente:
- Desde Azure Portal.
- Mediante la nueva CLI de Azure.

Para descargar el código fuente del bot desde [Azure portal](http://portal.azure.com), simplemente seleccione el recurso de bot y haga clic en **Compilar** bajo la administración de bots. Hay varias opciones disponibles para administrar o recuperar el código fuente del bot de forma local.

![Descarga del bot desde Azure Portal](media/bot-builder-tools/az-portal-manage-code.png)

Para descargar el código fuente del bot mediante la CLI, escriba el comando siguiente. El bot se descargará a una ubicación local.

```azurecli
az bot download --name "my-bot-name" --resource-group "my-resource-group"
```

![Comando de descarga desde la CLI](media/bot-builder-tools/cli-bot-download-command.png)

## <a name="4-store-your-bot-information-with-msbot"></a>4. Almacenamiento de la información del bot con MSBot

La nueva herramienta MSBot permite crear un archivo **.bot**, en el que se almacenan metadatos sobre otros servicios que consume el bot, todo en una sola ubicación. Este archivo también permite que el bot se conecte a estos servicios desde la CLI. Las herramientas de MSBot admiten varios comandos, consulte el archivo [Léame](https://aka.ms/botbuilder-tools-msbot-readme) para más información. 

Para instalar MSBot, ejecute:

```shell
npm install -g msbot
```

Para crear un archivo de bot desde la CLI, escriba **msbot init** seguido del nombre del bot y la dirección URL del punto de conexión de destino, por ejemplo:

```shell
msbot init --name name-of-my-bot --endpoint http://localhost:bot-port-number/api/messages
```

Para conectar el bot a un servicio, escriba **msbot connect** en la CLI, seguido del servicio adecuado:

```shell
msbot connect service-type
```

| Tipo de servicio | DESCRIPCIÓN |
| ------ | ----------- |
| azure  |Conecta el bot a un registro de Azure Bot Service.|
|endpoint| Conectar el bot a un punto de conexión como localhost|
|luis     | Conecta el bot a una aplicación de LUIS. |
| qna     |Conecta el bot a una base de conocimiento de QnA.|
|help [cmd]  |Muestra la ayuda para [cmd].|

Consulte el archivo [Léame](https://aka.ms/botbuilder-tools-msbot-readme) para obtener una lista completa de los servicios admitidos.

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>Conexión del bot a ABS con el archivo .bot

Con la herramienta MSBot instalada, puede conectar fácilmente el bot a un grupo de recursos existente en Azure Bot Service si ejecuta el comando az bot **show**.

```azurecli
az bot show -n my-bot-name -g my-resource-group --msbot | msbot connect azure --stdin
```

Esta operación tomará el punto de conexión, Id. de la aplicación y contraseña de MSA actuales del grupo de recursos de destino y actualizará la información correspondiente en el archivo .bot.


## <a name="5-manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>5. Administración, actualización o creación de servicios de LUIS y QnA con las nuevas herramientas de Bot Builder

Las [herramientas de Bot Builder](https://aka.ms/botbuilder-tools) son un conjunto de herramientas nuevo que permite administrar los recursos de bot e interactuar con ellos directamente desde la línea de comandos.

>[!TIP]
> Cada herramienta del generador de bots incluye un comando de ayuda global, accesible desde la línea de comandos si se escribe **-h** o **--help**. Este comando está disponible en cualquier momento desde cualquier acción, lo que proporcionará una representación útil de las opciones disponibles junto con sus descripciones.

### <a name="ludown"></a>LUDown

[LUDown](https://aka.ms/botbuilder-ludown) permite describir y crear componentes de lenguaje eficaces para los bots mediante archivos **.lu**. El nuevo archivo .lu es un tipo de formato Markdown que la herramienta LUDown usa para generar los archivos .json específicos del servicio de destino. Actualmente, puede usar los archivos .lu para crear una aplicación de [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app) o una base de conocimiento de [QnA](https://qnamaker.ai/Documentation/CreateKb) con formatos diferentes para cada una. LUDown está disponible como un módulo npm y se puede usar si se instala de forma global en la máquina:

```shell
npm install -g ludown
```

La herramienta LUDown se puede usar para crear modelos de .json para LUIS y QnA.  

### <a name="creating-a-luis-application-with-ludown"></a>Creación de una aplicación de LUIS con LUDown

Puede definir [intenciones](https://docs.microsoft.com/azure/cognitive-services/luis/add-intents) y [entidades](https://docs.microsoft.com/azure/cognitive-services/luis/add-entities) para una aplicación de LUIS, tal como lo haría desde el portal de LUIS.

`# \<intent-name\>` describe la sección de definición de una intención nueva. Las líneas siguientes contienen [expresiones](https://docs.microsoft.com/azure/cognitive-services/luis/add-example-utterances) que describen esa intención.

Por ejemplo, puede crear varias intenciones LUIS en un único archivo .lu de esta forma: 

```ludown
# Greeting
Hi
Hello
Good morning
Good evening

# Help
help
I need help
please help
```

### <a name="qna-pairs-with-ludown"></a>Pares de QnA con LUDown

El formato de archivo .lu también admite pares de QnA mediante la notación siguiente: 

  ```ludown
  > This is a comment. QnA definitions have the general format:
  ### ? this-is-the-question-string
  - this-is-an-alternate-form-of-the-same-question
  - this-is-another-one
    ```markdown
    this-is-the-answer
    ```
  ```
La herramienta LUDown separará automáticamente la pregunta y las respuestas en un archivo JSON de QnA Maker que, después, se puede usar para crear una nueva base de conocimiento [QnaMaker.ai](http://qnamaker.ai).

  ```ludown
  ### ? How do I change the default message for QnA Maker?
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

También se pueden agregar varias preguntas a la misma respuesta si se agregan nuevas líneas de variaciones de preguntas para una sola respuesta. 

  ```ludown
  ### ? What is your name?
  - What should I call you?
    ```markdown
    I'm the echoBot! Nice to meet you.
    ```
  ```

### <a name="generating-json-models-with-ludown"></a>Generación de modelos .json con LUDown

Después de definir los componentes del lenguaje de LUIS o QnA en formato .lu, puede publicarlos en un archivo .json de LUIS, o bien .json o .tsv de QnA. Cuando se ejecuta, la herramienta LUDown buscará los archivos .lu en el mismo directorio de trabajo para analizarlos. Como la herramienta LUDown puede tener como destino archivos .lu de LUIS o QnA, solo hay que especificar para qué servicio de lenguaje se va generar, mediante el comando general **ludown parse <Service> --in <luFile>**. 

En el directorio de trabajo de ejemplo, hay dos archivos .lu para analizar: "luis-sample.lu" para crear el modelo de LUIS, y "qna-sample.lu" para crear una base de conocimiento de QnA.


#### <a name="generate-luis-json-models"></a>Generar modelos .json de LUIS

**luis-sample.lu** 
```ludown
# Greeting
- Hi
- Hello
- Good morning
- Good evening
```

Para generar un modelo de LUIS mediante LUDown, escriba lo siguiente en el directorio de trabajo actual:

```shell
ludown parse ToLuis --in ludown-file-name.lu
```

#### <a name="generate-qna-knowledge-base-json"></a>Generar el archivo .json de base de conocimiento de QnA

**qna-sample.lu**
  ```ludown
  > This is a sample ludown file for QnA Maker.

  ### ? How do I change the default message
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

De forma similar, para crear una base de conocimiento de QnA, solo hay que cambiar el destino del análisis. 

```shell
ludown parse ToQna --in ludown-file-name.lu
```

LUIS y QnA pueden usar los archivos JSON resultantes a través de sus portales correspondientes, o bien a través de las nuevas herramientas de la CLI. 

## <a name="6-connect-to-luis-an-qna-maker-services-from-the-cli"></a>6. Conexión a servicios de LUIS y QnA Maker desde la CLI

### <a name="connect-to-luis-from-the-cli"></a>Conexión a LUIS desde la CLI 

En el nuevo conjunto de herramientas se incluye una [extensión de LUIS](https://aka.ms/botbuilder-luis-cli) que permite administrar de forma independiente los recursos de LUIS. Está disponible como módulo npm que se puede descargar de la siguiente manera:

```shell
npm install -g luis-apis
```
El uso de comandos básicos para la herramienta de LUIS desde la CLI es el siguiente:

```shell
luis action-name resource-name arguments-list
```
Para conectar el bot a LUIS, deberá crear un archivo **.luisrc**. Se trata de un archivo de configuración que aprovisiona el identificador y la contraseña de la aplicación de LUIS para el punto de conexión de servicio cuando la aplicación realiza llamadas salientes. Puede crear este archivo mediante la ejecución de **luis init** como se muestra a continuación:

```shell
luis init
```
En el terminal, se le pedirá que escriba la clave de creación de LUIS, la región y el identificador de la aplicación antes de que la herramienta genere el archivo.  

![Comando init de LUIS](media/bot-builder-tools/luis-init.png) 


Una vez generado este archivo, la aplicación podrá consumir el archivo .json de LUIS (generado desde LUDown) mediante el comando siguiente desde la CLI: 

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>Conectarse a QnA desde la CLI

En el nuevo conjunto de herramientas se incluye una [extensión QnA](https://aka.ms/botbuilder-tools-qnaMaker) que permite administrar de forma independiente los recursos de LUIS. Está disponible como módulo npm que se puede descargar:

```shell
npm install -g qnamaker
```
Con la herramienta QnA Maker, puede crear, actualizar, publicar, eliminar y entrenar la base de conocimiento. Para empezar, deberá crear un archivo **.qnamakerrc**, necesario para habilitar el punto de conexión al servicio. Puede crear fácilmente este archivo si ejecuta **qnamaker init** y sigue las indicaciones, junto con el identificador de la base de conocimiento de QnA Maker. 

```shell
qnamaker init 
```
![Archivo init de QnaMaker](media/bot-builder-tools/qnamaker-init.png)

Una vez generado el archivo .qnamakerrc, se puede conectar a la base de conocimiento de QnA para consumir el archivo de base de conocimiento .json o .tsv (generado desde LUDown) mediante el comando siguiente:

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="7-publish-to-azure-from-the-cli"></a>7. Publicación en Azure desde la CLI

Después de realizar cambios en el código fuente del bot, puede publicarlos sin problemas si ejecuta lo siguiente:

```azurecli
az bot publish --name "my-bot-name" --resource-group "my-resource-group"
```

## <a name="references"></a>Referencias
- [Herramientas de Bot Builder](https://aka.ms/botbuilder-tools-readme)
- [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
