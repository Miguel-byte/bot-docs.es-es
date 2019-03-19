---
title: Implementación de plantillas de bot de empresa | Microsoft Docs
description: Aprenda a implementar todos los recursos de Azure auxiliares para el bot de empresa.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e888b2473269cf576fd9edda0d99a30aa6212f7b
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/07/2019
ms.locfileid: "57571882"
---
# <a name="enterprise-bot-template---getting-started"></a>Enterprise Bot Template (introducción)

> [!NOTE]
> Este tema se aplica a la versión v4 del SDK. 

## <a name="prerequisites"></a>Requisitos previos

Instale lo siguiente:
- [VSIX de Enterprise Bot Template](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4enterprise)
- [.NET Core](https://www.microsoft.com/net/download) (la versión más reciente)
- [Administrador de paquetes de Node](https://nodejs.org/en/)
- [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0) (la versión más reciente)
- [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Las herramientas de la CLI de Azure Bot Service](https://github.com/Microsoft/botbuilder-tools) (versiones más recientes)
    ```shell
    npm install -g ludown luis-apis qnamaker botdispatch msbot chatdown
    ```
- [LuisGen](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/LUISGen/src/npm/readme.md)
    ```shell
    dotnet tool install -g luisgen
    ```

## <a name="create-your-bot-project"></a>Creación de un proyecto de un bot
1. En Visual Studio, haga clic en **Archivo -> Nuevo proyecto**.
1. En **Bot**, seleccione **Enterprise Bot Template**.

![Archivar nueva plantilla de proyecto](media/enterprise-template/new_project.jpg)

1. Asigne un nombre al proyecto y haga clic en **Crear**.
1. Haga clic con el botón derecho en el proyecto y haga clic en **Compilar** para restaurar los paquetes de NuGet.

## <a name="deploy-your-bot"></a>Implementación del bot

Las instancias de Enterprise Template Bot requieren los siguientes servicios de Azure para una operación de un extremo a otro:
- Aplicación web de Azure
- Cuenta de Azure Storage (transcripciones)
- Azure Application Insights (datos de telemetría)
- Azure CosmosDb (almacenamiento del estado de la conversación)
- Azure Cognitive Services: Language Understanding
- Azure Cognitive Services: QnA Maker (incluye Azure Search y Azure Web App)

Los pasos siguientes le ayudarán a implementar estos servicios mediante los scripts de implementación proporcionados:

1. Recupere su clave de creación de LUIS.
   - En [esta](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions) página de documentación podrá ver cuál es el portal de LUIS correcto para la región de su implementación. Tenga en cuenta que www.luis.ai se refiere a la región de EE. UU. y una clave de creación recuperada de este portal no funcionará con una implementación en Europa.
   - Una vez que haya iniciado sesión, haga clic en el nombre que aparece en la esquina superior derecha.
   - Seleccione **Configuración** y anote la clave de creación para usarla más adelante.
    
    ![Captura de pantalla de la clave de creación de LUIS](./media/enterprise-template/luis_authoring_key.jpg)

1. Abra una ventana del símbolo del sistema.
1. Inicie sesión en la cuenta de Azure mediante la CLI de Azure. Puede encontrar una lista de las suscripciones a las que tiene acceso en la página [Suscripciones](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) de Azure Portal.
    ```shell
    az login
    az account set --subscription "YOUR_SUBSCRIPTION_NAME"
    ```

1. Ejecute el comando msbot clone services para implementar los servicios y configurar un archivo .bot en el proyecto. **NOTA: Una vez completada la implementación, debe anotar el secreto del archivo del bot que se muestra en la ventana del símbolo del sistema para poder usarlo posteriormente.**

    ```shell
    msbot clone services --name "YOUR-BOT-NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\LOCALE_FOLDER" --location "REGION"
    ```

    > **Notas**:
    >- El parámetro **YOUR-BOT-NAME** debe ser **globalmente único** y solo puede contener letras minúsculas, números y guiones ("-").
    >- Asegúrese de que la región de implementación que se proporciona en este paso coincide con la región del portal de su clave de creación de LUIS (por ejemplo, westus para luis.ai).
    >- Hay un problema conocido que pueden sufrir algunos usuarios al aprovisionar un AppId y contraseña de MSA. Si recibe este error `ERROR: Unable to provision MSA id automatically. Please pass them in as parameters and try again`, vaya a [https://apps.dev.microsoft.com](https://apps.dev.microsoft.com) y cree manualmente una aplicación, anote el AppId y la contraseña o secreto. Vuelva a ejecutar el comando msbot clone services anterior, pero proporcione dos nuevos argumentos, `--appId` y `--appSecret`, con los valores que acaba de recuperar. Es posible que tenga que aplicar una cadena de escape a los caracteres especiales de la contraseña que el shell pueda interpretar como un comando:
    >   - En el caso del *símbolo del sistema de Windows*, escriba el valor de appSecret entre comillas dobles. p. ej. `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret "YOUR_APP_SECRET"`
    >   - En el caso de *Windows PowerShell, intente pasar el valor de appSecret después del argumento --%. p. ej. `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --% --appSecret "YOUR_APP_SECRET"`
    >   - En el caso de *MacOS o Linux*, escriba el valor de appSecret entre comillas simples. p. ej. `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret 'YOUR_APP_SECRET'`

1. Una vez completada la implementación, actualice el archivo **appsettings.json** con el secreto del archivo de bot. 
    
    ```
    "botFilePath": "./YOUR_BOT_FILE.bot",
    "botFileSecret": "YOUR_BOT_SECRET",
    ```
1. Ejecute el siguiente comando y recupere la clave InstrumentationKey de la instancia de Application Insights.
    ```
    msbot list --bot YOUR_BOT_FILE.bot --secret "YOUR_BOT_SECRET"
    ```

1. Actualice el archivo **appsettings.json** con la clave de instrumentación:

    ```
    "ApplicationInsights": {
        "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
    }
    ```

## <a name="test-your-bot"></a>Prueba del bot

Una vez completada, ejecute el proyecto de bot dentro de su entorno de desarrollo y abra **Bot Framework Emulator**. En Bot Framework Emulator, haga clic en **File > Open Bot** (Archivo > Abrir bot) y vaya al archivo .bot del directorio.

![Captura de pantalla de punto de conexión de desarrollo de emulador](./media/enterprise-template/development_endpoint.jpg)

Debería recibir un mensaje de introducción al comenzar la conversación.

Escriba ```hi``` para comprobar que todo funciona.

## <a name="publish-your-bot"></a>Publicar el bot

Las pruebas pueden realizarse de un extremo a otro localmente. Cuando esté listo para implementar el bot en Azure para realizar pruebas adicionales puede usar el comando siguiente para publicar el código fuente:

```shell
az bot publish -g YOUR-BOT-NAME -n YOUR-BOT-NAME --proj-name YOUR-BOT-NAME.csproj --version v4
```

## <a name="view-your-bot-analytics"></a>Visualización de los análisis del bot
Enterprise Bot Template incluye un panel de Power BI preconfigurado que se conecta al servicio Application Insights para proporcionar análisis de las conversaciones. Una vez que haya probado el bot localmente, puede abrir dicho panel para ver los datos. 

1. Descargue el panel de Power BI (archivo .pbit) [aquí](https://github.com/Microsoft/AI/blob/master/solutions/analytics/ConversationalAnalyticsSample_02132019.pbit).
1. Creación del panel en [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/).
1. Escriba el identificador de la aplicación de Application Insights (que se encuentra en el archivo .bot).

    ![Dónde encontrar id. de la aplicación AppInsights en el archivo de bot](./media/enterprise-template/appInsights_appId.jpg)

1. Más información acerca de las características del panel de Power BI [aquí](https://github.com/Microsoft/AI/tree/master/solutions/analytics).

# <a name="next-steps"></a>Pasos siguientes

Tras implementar correctamente el bot desde el principio, puede personalizarlo para adaptarlo a sus necesidades y al escenario. Continúe con [Personalización del bot](bot-builder-enterprise-template-customize.md).
