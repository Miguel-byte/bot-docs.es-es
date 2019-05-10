---
ms.openlocfilehash: e6e76b284fe2b382af0592bba17e15b3d0ca9cbd
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563436"
---
El [conector](~/dotnet/bot-builder-dotnet-concepts.md#connector) usa un objeto <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> para pasar información de un lado al otro entre el bot y el canal (usuario). El tipo de actividad más común es **message**, pero hay otros tipos de actividades que se pueden usar para comunicar distintos tipos de información a un bot o canal. 