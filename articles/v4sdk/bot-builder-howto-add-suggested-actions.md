---
title: Uso del botón para la entrada de datos | Microsoft Docs
description: Aprenda a enviar acciones sugeridas dentro de mensajes mediante Bot Framework SDK para JavaScript.
keywords: acciones sugeridas, botones, entrada adicional
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 43c9b0f062f4db909612f3fb9dc048be65c64c57
ms.sourcegitcommit: d29d3d7ccef401aa1e84e19e623db33b5ff13e63
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2019
ms.locfileid: "67160640"
---
# <a name="use-button-for-input"></a>Uso del botón para la entrada de datos

[!INCLUDE[applies-to](../includes/applies-to.md)]

Puede habilitar el bot para que presente botones en los que el usuario puede pulsar para proporcionar una entrada. Los botones mejoran la experiencia del usuario al permitir que este responda una pregunta o haga una selección simplemente pulsando un botón, en lugar de tener que escribir una respuesta con un teclado. A diferencia de los botones que aparecen en las tarjetas enriquecidas (que permanecen visibles y accesibles para el usuario incluso después de que se pulsen), los botones que aparecen en el panel de acciones sugeridas desaparecerán una vez que el usuario haya hecho una selección. Esto evita que el usuario pulse botones obsoletos dentro de una conversación y simplifica el desarrollo de bots (ya que no necesitará tener en cuenta ese escenario). 

## <a name="suggest-action-using-button"></a>Acción sugerida con un botón

Las *acciones sugeridas* permiten al bot presentar botones. Puede crear una lista de acciones sugeridas (también conocidas como "respuestas rápidas") que se mostrarán al usuario para un único turno de la conversación: 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

El código fuente que se muestra a continuación se basa en el ejemplo [Sugerir acciones](https://aka.ms/SuggestedActionsCSharp).

[!code-csharp[suggested actions](~/../botbuilder-samples/samples/csharp_dotnetcore/08.suggested-actions/Bots/SuggestedActionsBot.cs?range=87-101)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El código fuente que se muestra a continuación se basa en el ejemplo [Sugerir acciones](https://aka.ms/SuggestActionsJS).

[!code-javascript[suggested actions](~/../botbuilder-samples/samples/javascript_nodejs/08.suggested-actions/bots/suggestedActionsBot.js?range=61-64)]

---

## <a name="additional-resources"></a>Recursos adicionales

Puede acceder al código fuente completo desde aquí: [ejemplo de CSharp](https://aka.ms/SuggestedActionsCSharp) o [ejemplo de JavaScript](https://aka.ms/SuggestActionsJS).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Guardar usuario y datos de conversación](./bot-builder-howto-v4-state.md)
