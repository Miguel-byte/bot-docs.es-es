---
title: Claves de Application Insights | Microsoft Docs
description: Obtenga información sobre cómo obtener claves de Application Insights para agregar datos de telemetría a un bot.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 1ce24216cfd59259ff4869d4898fb3ad3bbf41c2
ms.sourcegitcommit: abde9e0468b722892f94caf2029fae165f96092f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/09/2018
ms.locfileid: "48875692"
---
# <a name="application-insights-keys"></a>Claves de Application Insights

Azure **Application Insights** muestra los datos de la aplicación en un recurso de Microsoft Azure. Para agregar datos de telemetría al bot, necesita una suscripción de Azure y un recurso de Application Insights creado para el bot. A partir de este recurso, puede obtener las tres claves para configurar el bot:

1. Clave de instrumentación
2. Identificador de aplicación
3. Clave de API

En este tema le mostraremos cómo crear estas claves de Application Insights.

> [!NOTE]
> Durante la creación del bot o el proceso de registro, tuvo la opción de *activar* o *desactivar* **Application Insights**. Si lo había *activado*, el bot ya tiene todas las claves necesarias de Application Insights. Sin embargo, si lo había *desactivado*, puede seguir las instrucciones de este tema para crear manualmente estas claves.

## <a name="instrumentation-key"></a>Clave de instrumentación

Para obtener la clave de instrumentación, haga lo siguiente:
1. Desde [Azure Portal](http://portal.azure.com), en la sección Monitor, cree un nuevo recurso de **Application Insights** (o use uno existente).
![Captura de pantalla del portal de la lista de Application Insights](~/media/portal-app-insights-add-new.png)

2. En la lista de recursos de Application Insights, haga clic en el recurso de Application Insights que acaba de crear.

3. Haga clic en **Descripción general**.

4. Expanda el bloque **Información esencial** y busque la **clave de instrumentación**. 
![Captura de pantalla de Información general en el portal](~/media/portal-app-insights-instrumentation-key-dropdown.png)
![Captura de pantalla de Clave de instrumentación en el portal](~/media/portal-app-insights-instrumentation-key.png)

5. Copie la **clave de instrumentación** y péguela en el campo **Clave de instrumentación de Application Insights** de la configuración del bot.

## <a name="application-id"></a>Identificador de aplicación

Para obtener el id. de la aplicación, haga lo siguiente:
1. Desde el recurso de Application Insights, haga clic en **Acceso de API**.

2. Copie el **id. de la aplicación** y péguelo en el campo **Id. de la aplicación de Application Insights** de la configuración del bot. 
![Captura de pantalla del portal del id. de la aplicación](~/media/portal-app-insights-appid.png)

## <a name="api-key"></a>Clave de API

Para obtener la clave de API, haga lo siguiente:
1. Desde el recurso de Application Insights, haga clic en **Acceso de API**.

2. Haga clic en **Crear clave de API**.

3. Escriba una descripción breve, compruebe la opción **Read telementry** (Leer datos de telemetría) y haga clic en el botón **Generar clave**.
![Captura de pantalla del portal del id. de la aplicación y de la clave de API](~/media/portal-app-insights-appid-apikey.png)

   > [!WARNING]
   > Copie esta **clave de API** y guárdela porque nunca se le volverá a mostrar. Si pierda esta clave, deberá crear una nueva.

4. Copie la clave de API en el campo **Clave de API de Application Insights** de la configuración del bot.

## <a name="additional-resources"></a>Recursos adicionales
Para obtener más información sobre cómo conectar estos campos en la configuración del bot, consulte [Enable analytics](~/bot-service-manage-analytics.md#enable-analytics) (Habilitación de los análisis).