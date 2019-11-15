---
title: Incorporación de actividades de seguimiento al bot | Microsoft Docs
description: Describe qué es la actividad de seguimiento en Bot Framework SDK y cómo se usa.
keywords: seguimiento, actividad, bot, Bot Framework SDK
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 54c663a370cc4f613e0f38bb8057b10e49bf8c69
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933766"
---
# <a name="add-trace-activities-to-your-bot"></a>Incorporación de actividades de seguimiento al bot

[!INCLUDE[applies-to](../includes/applies-to.md)]

<!-- What is it and why use it -->

Una _actividad de seguimiento_ es una actividad que el bot puede enviar a Bot Framework Emulator.
Puede usar las actividades de seguimiento para depurar un bot de forma interactiva, ya que permiten ver información sobre el bot mientras se ejecuta localmente.

<!-- Details -->

Las actividades de seguimiento solo se envían al emulador y no a ningún otro cliente o canal.
El emulador las muestra en el registro, pero no en el panel principal de chats.

- Las actividades de seguimiento enviadas mediante el contexto del turno se envían a través de los _controladores de actividad de envío_ registrados en el contexto del turno.
- Las actividades de seguimiento enviadas mediante el contexto del turno se asocian a la actividad de entrada aplicando la referencia de la conversación, en caso de que haya alguna.
  En el caso de un mensaje proactivo, el _identificador Responder a_ será un nuevo GUID.
- Independientemente de cómo se envíe, una actividad de seguimiento nunca establece la marca _responded_.

## <a name="to-use-a-trace-activity"></a>Para utilizar una actividad de seguimiento

Para ver una actividad de seguimiento en el emulador, necesita un escenario en el que el bot enviará una actividad de seguimiento, como generar una excepción y enviar una actividad de seguimiento desde el controlador de errores del turno del adaptador.

Para enviar una actividad de seguimiento desde el bot:

1. Cree una nueva actividad.
   - Establezca su propiedad _type_ en "trace". Este es un paso necesario.
   - Opcionalmente, puede establecer sus propiedades _name_, _label_, _value_ y _value type_, según corresponda para el seguimiento.
1. Use el método _send activity_ del objeto del _contexto del turno_ para enviar la actividad de seguimiento.
   - Este método agrega los valores de las restantes propiedades necesarias de la actividad, en función de la actividad de entrada.
     Entre ellas se incluyen las propiedades _channel ID_, _service URL_, _from_ y _recipient_.

Para ver una actividad de seguimiento en el emulador:

1. Ejecute el ejemplo localmente en la máquina.
1. Pruébelo con el emulador.
   - Interactúe con el bot y siga los pasos del escenario para generar la actividad de seguimiento.
   - Cuando el bot emite la actividad de seguimiento, la actividad de seguimiento se muestra en el registro del emulador.

Esta es una actividad de seguimiento que puede ver si ejecutó el bot básico sin configurar primero la base de conocimiento de QnAMaker en la que se basa el bot.

![Captura de pantalla del emulador](./media/using-trace-activities.png)

## <a name="add-a-trace-activity-to-the-adapters-on-error-handler"></a>Incorporación de una actividad de seguimiento al controlador de errores del adaptador

El controlador de _errores del turno_ del adaptador captura cualquier excepción no detectada de otro modo que se produzca en el bot durante un turno.
Este es un buen lugar para una actividad de seguimiento, ya que puede enviar un mensaje descriptivo al usuario y enviar información de depuración sobre la excepción al emulador.

Este código de ejemplo procede del ejemplo **Bot básico**. Puede ver el ejemplo completo en [**C#** ](https://aka.ms/cs-core-sample) o [**JavaScript**](https://aka.ms/js-core-sample).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

El método auxiliar **SendTraceActivityAsync** definido en este ejemplo envía información de la excepción al emulador como una actividad de seguimiento.

**AdapterWithErrorHandler.cs**

[!code-csharp[SendTraceActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/AdapterWithErrorHandler.cs?range=16-51&highlight=33-34)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El controlador **onTurnError** del adaptador crea la actividad de seguimiento para incluir la información de la excepción y la envía al emulador.

**index.js**

[!code-javascript[onTurnError ](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/index.js?range=35-57&highlight=8-14)]

---

## <a name="additional-resources"></a>Recursos adicionales

- [Cómo depurar un bot con middleware de inspección](../bot-service-debug-inspection-middleware.md) describe cómo agregar middleware que emite actividades de seguimiento.
- Para depurar un bot implementado, puede utilizar Application Insights. Para más información, consulte [Adición de telemetría al bot](bot-builder-telemetry.md).
- Para más información acerca de cada tipo de actividad, consulte el [Esquema de actividades de Bot Framework](https://aka.ms/botSpecs-activitySchema).
