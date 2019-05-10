---
ms.openlocfilehash: b5809b6d46cdc09035efb36c3ea58c2ca9dc6c3e
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563616"
---
Normalmente los usuarios intentan tener acceso a ciertas funcionalidades dentro de un bot con palabras clave como "ayuda", "cancelar" o "empezar de nuevo". A menudo, esto ocurre en medio de una conversación, cuando el bot está esperando una respuesta diferente. Al implementar **controladores de mensajes globales**, puede diseñar al bot para que controle dichas solicitudes de manera correcta.
Los controladores examinarán la entrada del usuario para las palabras clave que especifique, como "ayuda", "cancelar" o "empezar de nuevo" y responderán según corresponda. 

![cómo hablan los usuarios](~/media/designing-bots/capabilities/trigger-actions.png)

> [!NOTE]
> Al definir la lógica en los controladores de mensajes globales, la hace accesible desde todos los diálogos. Los diálogos y mensajes individuales se pueden configurar para ignorar las palabras clave de manera segura.
