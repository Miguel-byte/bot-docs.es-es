---
title: Migración de un bot de C# de un plan de consumo a un plan de App Service | Microsoft Docs
description: Migre un bot de Bot Service de C# de un plan de hospedaje de consumo a un plan de hospedaje de App Service.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: aafbfb2a38e2d5370cb2db5721dd7bc130497d74
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999222"
---
# <a name="change-the-hosting-plan-for-your-bot-service"></a>Cambio del plan de hospedaje para el servicio de bots

En este tema se explica cómo puede migrar un bot de script de C# con un plan de consumo a un bot de C# con un plan de App Service. 

## <a name="advantages-of-a-bot-on-an-app-service-plan"></a>Ventajas de un bot en un plan de App Service

Los bots en un plan de App Service se ejecutan como aplicaciones web de Azure. Los bots de aplicación web pueden hacer cosas que los bots del plan de consumo no pueden:

- Un bot de aplicación web puede agregar definiciones de ruta personalizadas.
- Un bot de aplicación web puede habilitar un servidor Websocket. 
- Un bot que usa un plan de hospedaje de consumo tiene las mismas limitaciones que todo el código que se ejecuta en Azure Functions. Para obtener más información, consulte <a target='_blank' href='/azure/azure-functions/functions-scale'>Plan de consumo y plan de App Service de Azure Functions</a>.

## <a name="download-your-existing-bot-source"></a>Descarga del origen del bot existente

Siga estos pasos para descargar el código fuente de su bot existente:

1. Dentro del bot de Azure, haga clic en la pestaña **Configuración** y expanda la sección **Implementación continua**.  
2. Haga clic en el botón azul para descargar el archivo ZIP que contiene el código fuente del bot.  
    ![Descarga del archivo ZIP del bot](~/media/continuous-deployment-consumption-download.png)
3. Extraiga en una carpeta local el contenido del archivo ZIP descargado. 


## <a name="create-a-bot-template"></a>Creación de una plantilla de bot

Bot Service tiene las mismas plantillas para los bots del plan de consumo y el plan de App Service. Para migrar de un bot de plan de consumo, cree un nuevo bot de plan de App Service en Bot Service, basado en la misma plantilla. El código subyacente puede diferir entre los dos tipos de hospedaje para la misma plantilla, pero la nueva aplicación web tiene una estructura y características de configuración similares que las que usa el bot existente.

## <a name="download-the-new-bot-source"></a>Descarga del origen del nuevo bot

Siga estos pasos para descargar el código fuente del nuevo bot:

1. Dentro de un bot de Azure, haga clic en la pestaña **BUILD** (Compilar), busque la sección **Download source code** (Descargar código fuente) y haga clic en **Descargar archivo ZIP**. 
2. Extraiga en la carpeta local el contenido del archivo ZIP descargado.

## <a name="add-source-files-to-new-solution"></a>Adición de archivos de origen a la nueva solución

Algunos archivos .csx podrían compilarse y ejecutarse como archivos .cs en la nueva solución. Crear un archivo .cs para cada archivo .csx de la solución, excepto `run.csx`. Migrará la lógica `run.csx` a mano. En los archivos. cs, debe agregar una declaración de clase y la una declaración de espacio de nombres opcional.

## <a name="migrate-runcsx-logic-into-your-project"></a>Migración de la lógica de run.csx al proyecto

Los proyectos de script de C# tienen un método `Run`donde se controlan diferentes valores de `ActivityTypes`. Importe la lógica de control de actividades al método `MessageController.Post`, en `MessageController.cs`.

## <a name="remove-compiler-keywords"></a>Eliminación de palabras clave del compilador

Los archivos de script de C# pueden incluir un módulo de referencia con la palabra clave `#r`. Quite estas líneas y, además, agréguelas como referencias al proyecto de Visual Studio. Además, quitar las palabra clave `#load`, que insertan otros archivos de código fuente en la compilación de archivos. En su lugar, agregue todos los archivos `.csx` al proyecto como código fuente `.cs`.

## <a name="add-references-from-projectjson"></a>Adición de referencias de project.json

Si su bot de plan de consumo agrega referencias de NuGet a su archivo `project.json`, agregue estas referencias a la nueva solución de Visual Studio haciendo clic con el botón derecho en el proyecto en el panel Explorador de soluciones y haciendo clic en **Agregar referencia**.

### <a name="add-references-that-were-implicit"></a>Adición de referencias que eran implícitas

Un bot de Bot Service en un plan de consumo incluye implícitamente estas referencias en todos los archivos de origen .csx. Es posible que el origen migrado a los archivos de origen .cs archivos tengan que agregar referencias explícitas para estas clases:

- `TraceWriter` en el paquete NuGet `Microsoft.Azure.WebJobs` que proporciona los tipos de espacio de nombres `Microsoft.Azure.WebJobs.Host`. 
- Desencadenadores de temporizador en el paquete de NuGet `Microsoft.Azure.WebJobs.Extensions`
- `Newtonsoft.Json`, `Microsoft.ServiceBus` y otros ensamblados a los que se hace referencia automáticamente
- `System.Threading.Tasks` y otros espacios de nombres importados automáticamente

Para obtener más información, consulte *Converting to class files* (Conversión a archivos de clase) en <a target='_blank' href='https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/'>Publishing a .NET class library as a Function App</a> (Publicación de una biblioteca de clases .NET como Function App).

## <a name="debug-your-new-bot"></a>Depuración del nuevo bot

Es mucho más sencillo depurar localmente un bot en un plan de App Service que un bot en un plan de consumo. Puede usar el [emulador](bot-service-debug-emulator.md) para depurar el código migrado localmente.

## <a name="publish-from-visual-studio-or-set-up-continuous-deployment"></a>Publicación desde Visual Studio, o configuración de una implementación continua

Por último, publique el código fuente migrado a Bot Service ya sea importando el archivo `.PublishSettings` y haciendo clic en **Publicar**, o [configurando la implementación continua](bot-service-debug-bot.md).
