---
ms.openlocfilehash: 1de15b72731434a7bcda0bbd47fbe61e40d14b4a
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230731"
---
El [conector](~/dotnet/bot-builder-dotnet-concepts.md#connector) usa un objeto <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> para pasar información de un lado al otro entre el bot y el canal (usuario). El tipo de actividad más común es **message**, pero hay otros tipos de actividades que se pueden usar para comunicar distintos tipos de información a un bot o canal. 