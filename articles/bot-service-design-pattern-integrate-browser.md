---
title: Integración del bot con un explorador web | Microsoft Docs
description: Obtenga información sobre cómo diseñar una transición adecuada de usuario del bot al explorador web y viceversa.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: e3833595003edda46a6ffd1d508237262aad94e1
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999232"
---
# <a name="integrate-your-bot-with-a-web-browser"></a>Integración del bot con un explorador web

Algunos escenarios requieren algo más que solo un bot para satisfacer un requisito. Puede que un bot deba enviar al usuario a un explorador web para completar una tarea y, a continuación, reanudar la conversación con el usuario una vez completada la tarea. 

## <a name="authentication-and-authorization"></a>Autenticación y autorización
Si busca la capacidad de leer el calendario del usuario en Office 365, o quizás incluso crear citas en nombre del usuario, el usuario antes debe autenticarse en Microsoft Azure Active Directory y autorizar al bot a acceder a los datos del calendario del usuario. El bot redirigirá al usuario a un explorador web para completar las tareas de autenticación y autorización y, a continuación, reanudará la conversación con el usuario. 

## <a name="security-and-compliance"></a>Seguridad y cumplimiento normativo
A menudo, los requisitos de seguridad y cumplimiento restringen el tipo de información que un bot puede intercambiar con un usuario. En algunos casos, puede ser necesario que el usuario envíe y reciba datos fuera de la conversación actual. Por ejemplo, si un usuario desea ejecutar un pago mediante un proveedor de pagos externo, no se debe especificar un número de tarjeta de crédito dentro del contexto de la conversación. En su lugar, el bot redirigirá al usuario a un explorador web para completar el proceso de pago y, a continuación, reanudará la conversación con el usuario.

En este artículo se explora el proceso de facilitar la transición de un usuario desde el bot al explorador web y viceversa. 

> [!NOTE]
> La transición del chat al explorador web y a la inversa no es ideal, dado que el cambio entre aplicaciones puede confundir fácilmente a un usuario. Para brindar una mejor experiencia, muchos canales ofrecen ventanas HTML integradas que un bot puede usar para presentar aplicaciones que, de lo contrario, aparecerían en un explorador web. Esta técnica permite al usuario permanecer dentro de la conversación a la vez que tiene acceso a recursos externos. Este enfoque es conceptualmente similar a las aplicaciones móviles que administran flujos de autorización mediante OAuth en las vistas web incrustadas.

## <a name="bot-to-web-browser-and-back-again"></a>Del bot al explorador web y viceversa

En este diagrama se muestra el flujo de alto nivel para la integración entre el bot y un explorador web. 

![Interacción entre bot y web](~/media/bot-service-design-pattern-integrate-browser/bot-to-web1.png)

Piense en cada paso del flujo:

1. <a id="generate-hyperlink"></a>El bot genera y muestra un hipervínculo que redirigirá al usuario a un sitio web. 
   El hipervínculo suele incluir datos a través de los parámetros de cadena de consulta en la dirección URL de destino que especifican información sobre el contexto de la conversación actual, por ejemplo, el id. de la conversación, el id. del canal y el id. del usuario en el canal. 

2. El usuario hace clic en el hipervínculo y se le redirige a la dirección URL de destino dentro de un explorador web. 

3. El bot entra en un estado en espera de comunicación desde el sitio web que indique que se ha completado el flujo del sitio web.  
   > [!TIP]
   > Diseñe este flujo para que el bot no permanezca permanentemente en el estado "en espera" si el usuario nunca completa el flujo del sitio web. En otras palabras, si el usuario abandona el explorador web y vuelve a comunicarse con el bot, el bot debe reconocer esa entrada, no [ignorarla](~/bot-service-design-navigation.md#the-mysterious-bot).

4. El usuario completa las tareas necesarias a través del explorador web. 
   Podría tratarse de un flujo de OAuth o cualquier secuencia de eventos necesaria para el escenario en cuestión. 

5. <a id="generate-magic-number"></a>Cuando el usuario completa el flujo del sitio web, este genera un "[número mágico](#verify-identity)" y se indica al usuario que copie el valor y lo pegue en la conversación con el bot. 

6. <a id="signal-to-bot"></a>El sitio web [ indica al bot](#website-signal-to-bot) que el usuario ha completado el flujo del sitio web. 
   Comunica el 'número mágico' al bot y proporciona todos los datos pertinentes.
   Por ejemplo, en el caso de un flujo de OAuth, el sitio web podría proporcionar un token de acceso al bot.

7. El usuario vuelve al bot y pega el "número mágico" en el chat. 
   El bot valida que el "número mágico" proporcionado por el usuario coincida con el valor esperado, comprobando que el usuario actual es el mismo usuario que previamente hizo clic en el hipervínculo para iniciar el flujo del sitio web. 

### <a id="verify-identity"></a> Comprobación de la identidad del usuario mediante el "número mágico"

La generación de un "número mágico" durante el flujo del bot al sitio web ([paso 5](#generate-magic-number) anterior) permite al bot comprobar posteriormente que el usuario que inició el flujo del sitio web sea realmente el usuario para el que estaba previsto. Por ejemplo, si un bot está llevando a cabo un chat grupal con varios usuarios, cualquiera de ellos podría haber hecho clic en el hipervínculo para iniciar el flujo del sitio web. Sin el proceso de validación del "número mágico", el bot no tiene manera de saber qué usuario ha completado el flujo. Un usuario podría autenticarse e insertar los tokens de acceso en la sesión de otro usuario. 

> [!WARNING] 
> Este no es simplemente un riesgo de los chats grupales. Sin el proceso de validación del "número mágico", cualquier persona que obtenga el hipervínculo para iniciar el flujo del sitio web puede suplantar la identidad de un usuario. 

El número mágico debe ser un número aleatorio generado mediante una biblioteca de criptografía fuerte. Para obtener un ejemplo del proceso de generación en C#, consulte <a href="https://github.com/MicrosoftDX/botauth/tree/master/CSharp" target="_blank">este código</a> dentro de la biblioteca <a href="https://www.nuget.org/packages/BotAuth" target="_blank">BotAuth</a>. BotAuth permite que los bots que están integrados en Microsoft Bot Framework implementen el flujo del bot al sitio web para autenticar a un usuario en un sitio web y, a continuación, usar el token de acceso que se generó a partir del proceso de autenticación. Puesto que BotAuth no realizar ninguna suposición acerca de las funcionalidades del canal, estos flujos deberían funcionar bien con la mayoría de los canales. 

> [!NOTE]
> La necesidad del proceso de validación del "número mágico" debería caer desuso a medida que canales creen sus propias vistas web incrustadas.

### <a id="website-signal-to-bot"></a> ¿Cómo hace el sitio web para "indicar" al bot?

Cuando el bot [genera el hipervínculo](#generate-hyperlink) donde el usuario hará clic para iniciar el flujo del sitio web, incluye información a través de los parámetros de cadena de consulta en la dirección URL de destino sobre el contexto de la conversación actual, por ejemplo, el id. de la conversación, el id. del canal y el id. del usuario en el canal. Posteriormente, el sitio web puede usar esta información para leer y escribir variables de estado para ese usuario o esa conversación con el Bot Builder SDK o la API de REST. Consulte el [paso 6](#signal-to-bot) anterior para obtener un ejemplo de cómo el sitio web "indica" al bot que el flujo del sitio web está completo.

## <a name="sample-code"></a>Código de ejemplo

Como se describe en este artículo, la biblioteca <a href="https://github.com/MicrosoftDX/botauth" target="_blank">BotAuth</a> permite que los flujos de OAuth se enlacen con los bots que se compilan con .NET y Node en Microsoft Bot Framework.

## <a name="additional-resources"></a>Recursos adicionales

- [Diálogos](~/dotnet/bot-builder-dotnet-dialogs.md)
- [Administración del flujo de conversación con diálogos (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [Manage conversation flow with dialogs (Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md) (Administración del flujo de conversación con cuadros de diálogo [Node.js])
