---
title: Introducción a Chat en web | Microsoft Docs
description: Obtenga información sobre cómo configurar Chat en web de Bot Framework.
keywords: bot framework, chat en web, chat, ejemplos, react, referencia
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/07/2019
ms.openlocfilehash: 1d787d375fcd1ddade544724bb28dea9aaeb1dd6
ms.sourcegitcommit: f3fda6791f48ab178721b72d4f4a77c373573e38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2019
ms.locfileid: "68671418"
---
# <a name="web-chat-overview"></a>Introducción a Chat en web

En este artículo se incluyen los detalles del componente [Chat en web de Bot Framework](https://github.com/microsoft/BotFramework-WebChat). El componente Chat en web de Bot Framework es un cliente basado en web muy personalizable para el SDK de Bot Framework V4. El SDK de Framework Bot v4 permite a los desarrolladores modelar la conversación y compilar aplicaciones sofisticadas de bot.

Si desea migrar desde Chat en web v3 a v4, vaya a la [sección sobre la migración](#migrating-from-web-chat-v3-to-v4).

## <a name="how-to-use"></a>Modo de uso

> [!NOTE]
> Para versiones anteriores de Chat en web (v3), visite la [rama del Chat en web v3](https://github.com/Microsoft/BotFramework-WebChat/tree/v3).

En primer lugar, cree un bot con [Azure Bot Service](https://azure.microsoft.com/services/bot-service/).
Una vez creado el bot, deberá [obtener el secreto de Chat en web del bot](../bot-service-channel-connect-webchat.md#step-1) en Azure Portal. A continuación, use el secreto para [generar un token](../rest-api/bot-framework-rest-direct-line-3-0-authentication.md) y páselo a Chat en web.

A continuación, se muestra cómo puede agregar el control Chat en web a su sitio web:

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID',
               username: 'Web Chat User',
               locale: 'en-US',
               botAvatarInitials: 'WC',
               userAvatarInitials: 'WW'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

> `userID`, `username`, `locale`, `botAvatarInitials` y `userAvatarInitials` son parámetros opcionales para pasar al método `renderWebChat`. Para obtener más información sobre las propiedades de Chat en web, examine la sección [Referencia de la API de Chat en web ](#web-chat-api-reference) de este `README`.
> ![Captura de pantalla de Chat en web](https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/weatherquery.png.jpg)

### <a name="integrate-with-javascript"></a>Integración con JavaScript

Chat en web está diseñado para integrarse con su sitio web existente mediante JavaScript o React. La integración con JavaScript le proporcionará una capacidad moderada de personalización y aplicación de estilos.

Puede usar el paquete completo y típico de Chat en web, que contiene las características usadas más habitualmente.

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

Vea el ejemplo funcional del [conjunto completo de Chat en web](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle).

### <a name="integrate-with-react"></a>Integración con React

Para una capacidad de personalización completa, puede usar React para volver a componer los componentes de Chat en web.

Para instalar la compilación de producción desde NPM, ejecute `npm install botframework-webchat`.

```jsx
import { DirectLine } from 'botframework-directlinejs';
import React from 'react';
import ReactWebChat from 'botframework-webchat';

export default class extends React.Component {
  constructor(props) {
    super(props);

    this.directLine = new DirectLine({ token: 'YOUR_DIRECT_LINE_TOKEN' });
  }

  render() {
    return (
      <ReactWebChat directLine={ this.directLine } userID='YOUR_USER_ID' />
      element
    );
  }
}
```

> También puede ejecutar `npm install botframework-webchat@master` para instalar una compilación de desarrollo sincronizada con la rama `master` del GitHub de Chat en web.

Vea un ejemplo funcional de [Chat en web representado vía React](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react/).

## <a name="customize-web-chat-ui"></a>Personalizar la interfaz de usuario de Chat en web

Chat en web está diseñado para ser personalizable sin bifurcación del código fuente. En la tabla siguiente se describe qué tipo de personalizaciones puede realizar al importar Chat en web de maneras diferentes. Esta lista no es exhaustiva.

|                               | Agrupación CDN         | React              |
| ----------------------------- | ------------------ | ------------------ |
| Cambiar colores                 | :heavy_check_mark: | :heavy_check_mark: |
| Cambiar tamaños                  | :heavy_check_mark: | :heavy_check_mark: |
| Actualizar y reemplazar estilos CSS     | :heavy_check_mark: | :heavy_check_mark: |
| Escuchar eventos              | :heavy_check_mark: | :heavy_check_mark: |
| Interactuar con la página web de hospedaje | :heavy_check_mark: | :heavy_check_mark: |
| Personalizar actividades de representación      |                    | :heavy_check_mark: |
| Personalizar datos adjuntos de representación     |                    | :heavy_check_mark: |
| Agregar nuevos componentes de interfaz de usuario         |                    | :heavy_check_mark: |
| Recomponer toda la interfaz de usuario        |                    | :heavy_check_mark: |

Consulte la información sobre la [personalización de Chat en web](https://github.com/Microsoft/BotFramework-WebChat/blob/master/SAMPLES.md) para aprender más sobre la personalización.

## <a name="migrating-from-web-chat-v3-to-v4"></a>Migración desde Chat en web v3 a v4

Existen tres posibles rutas que la migración puede tomar al migrar de v3 a v4. En primer lugar, compare su escenario inicial:

### <a name="my-current-website-integrates-web-chat-using-an-iframe-element-obtained-from-azure-bot-services-i-want-to-upgrade-to-v4"></a>Mi sitio web actual integra Chat en web con un elemento `<iframe>` obtenido de Azure Bot Service. Quiero actualizar a v4.

A partir de mayo de 2019, estamos implementando v4 en sitios web que integran Chat en web con el elemento `<iframe>`. Consulte [la documentación insertada](https://github.com/Microsoft/BotFramework-WebChat/tree/master/packages/embed) para obtener información sobre la integración de Chat en web mediante `<iframe>`.

### <a name="my-website-is-integrated-with-web-chat-v3-and-uses-customization-options-provided-by-web-chat-no-customization-at-all-or-very-little-of-my-own-customization-that-was-not-available-with-web-chat"></a>Mi sitio web está integrado con Chat en web v3 y usa las opciones de personalización proporcionadas por Chat en web, sin ninguna personalización o muy poca personalización propia, que no estaba disponible con Chat en web.

Siga la implementación de ejemplo [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration) para convertir su página web de Chat en web v3 a v4.

### <a name="my-website-is-integrated-with-a-fork-of-web-chat-v3-i-have-implemented-a-lot-of-customization-in-my-version-of-web-chat-and-i-am-concerned-v4-is-not-compatible-with-my-needs"></a>Mi sitio web se integra con una bifurcación de Chat en web v3. He implementado una gran cantidad de personalización en mi versión de Chat en web y me preocupa que v4 no sea compatible con mis necesidades.

Una de las cosas favoritas para nuestro equipo de Chat en web v4 es la capacidad de agregar personalización **sin necesidad de bifurcación de Chat en web**. Aunque esto crea una sobrecarga adicional para los usuarios de v3 que anteriormente habían bifurcado Chat en web, haremos todo lo posible para admitir clientes que se encuentren en esta situación. Use las sugerencias siguientes:

-  Eche un vistazo a la implementación de ejemplo [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration). Se trata de un punto de partida excelente para poner en funcionamiento Chat en web.
-  A continuación, recorra la [lista de ejemplos](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples) para comparar los requisitos de personalización con la compatibilidad que Chat en web ya ha proporcionado. Estos ejemplos están formados por características solicitadas habitualmente para Chat en web.
-  Si una o varias de sus características no están disponibles en los ejemplos, busque en las [incidencias abiertas y cerradas](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+), la [etiqueta Samples](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3ASample) y la [etiqueta Migration Support](https://github.com/Microsoft/BotFramework-WebChat/issues?q=is%3Aissue+migrate+label%3A%22Migration+Support%22) para encontrar solicitudes de ejemplo y la compatibilidad de personalización para una característica que está buscando. Agregar un comentario a las incidencias abiertas ayudará a que el equipo dé prioridad a las solicitudes que se encuentran en la alta demanda y le animamos a participar en nuestra comunidad.
-  Si no encontró la característica en la lista de las solicitudes abiertas, no dude en [presentar su propia solicitud](https://github.com/Microsoft/BotFramework-WebChat/issues/new). Igual que en el elemento anterior, otros clientes que agreguen comentarios a su incidencia abierta nos ayudarán a dar prioridad a las características que los usuarios en Chat en web necesitan más comúnmente.
-  Por último, si necesita la característica lo antes posible, agradecemos [solicitudes de incorporación de cambios](https://github.com/Microsoft/BotFramework-WebChat/compare) para Chat en web. Si tiene experiencia en codificación para implementar la característica por sí mismo, le agradeceríamos mucho el soporte técnico adicional. Crear la característica por sí mismo significará que estará disponible para su uso en Chat en web más rápidamente y que otros clientes que busquen la misma característica u otra similar puedan usar su contribución.
-  Asegúrese de consultar el resto de este `README` para obtener más información sobre v4.

## <a name="samples-list"></a>Lista de ejemplos

| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ejemplo&nbsp;Nombre&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | DESCRIPCIÓN                                                                                                                                                                                                                         | Vínculo                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| [`01.a.getting-started-full-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle)                                                                                       | Presenta Chat en web insertado desde una red CDN y se muestra un Chat en web simple y con características completas. Esto incluye tarjetas adaptables, Cognitive Services y las dependencias de Markdown-It.                                                            | [Demostración del conjunto completo](https://microsoft.github.io/BotFramework-WebChat/01.a.getting-started-full-bundle)                               |
| [`01.b.getting-started-es5-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.b.getting-started-es5-bundle)                                                                                         | Presenta el Chat en web con todas las características y compatibilidad con versiones anteriores para los exploradores ES5 mediante ES5 ponyfill de Chat en web.                                                                                                                | [Demostración del conjunto ES5](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle)                                 |
| [`01.c.getting-started-migration`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.c.getting-started-migration)                                                                                           | Muestra cómo migrar desde el bot de Chat en web v3 a v4.                                                                                                                                                                        | [Demostración de migración](https://microsoft.github.io/BotFramework-WebChat/01.c.getting-started-migration)                                   |
| [`02.a.getting-started-minimal-bundle`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/02.a.getting-started-minimal-bundle)                                                                                 | Presenta la red CDN minimizada con solo las dependencias básicas. Esto no incluye las tarjetas adaptables, las dependencias de Cognitive Services o las dependencias de Markdown-It.                                                                      | [Demostración del conjunto mínimo](https://microsoft.github.io/BotFramework-WebChat/02.a.getting-started-minimal-bundle)                         |
| [`02.b.getting-started-minimal-markdown`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/02.b.getting-started-minimal-markdown)                                                                             | Muestra cómo agregar la red CDN para la dependencia de Markdown-It sobre el conjunto mínimo.                                                                                                                                            | [Demostración de mínimo con Markdown](https://microsoft.github.io/BotFramework-WebChat/02.b.getting-started-minimal-markdown)                |
| [`03.a.host-with-react`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react)                                                                                                               | Muestra cómo crear un componente React que hospeda el Chat en web con todas las características.                                                                                                                                                 | [Demostración de host con React](https://microsoft.github.io/BotFramework-WebChat/03.a.host-with-react)                                       |
| [`03.b.host-with-Angular`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.b.host-with-angular)                                                                                                           | Muestra cómo crear un componente Angular que hospeda el Chat en web con todas las características.                                                                                                                                              | [Demostración de host con Angular](https://stackblitz.com/github/omarsourour/ng-webchat-example)                                              |
| [`04.a.display-user-bot-initials-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/04.a.display-user-bot-initials-styling)                                                                           | Muestra cómo mostrar las iniciales de ambos participantes de Chat en web.                                                                                                                                                                | [Demostración de iniciales de bot](https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/04.a.display-user-bot-initials-styling/)  |
| [`04.b.display-user-bot-images-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/04.b.display-user-bot-images-styling)                                                                               | Muestra cómo mostrar las imágenes e iniciales de ambos participantes de Chat en web.                                                                                                                                                     | [Demostración de imágenes de usuario](https://microsoft.github.io/BotFramework-WebChat/04.b.display-user-bot-images-styling)                           |
| [`05.a.branding-webchat-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.a.branding-webchat-styling)                                                                                             | Presenta la capacidad de aplicar estilos a Chat en web para que coincida con su marca. Este método para aplicar estilos personalizados no se interrumpirá en actualizaciones de Chat en web.                                                                                                   | [Demostración de personalización de la marca de Chat en web](https://microsoft.github.io/BotFramework-WebChat/05.a.branding-webchat-styling)                            |
| [`05.b.idiosyncratic-manual-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.b.idiosyncratic-manual-styling/)                                                                                    | Muestra cómo realizar cambios manuales de estilo y es una manera de personalizar el estilo de Chat en web más complicada y lenta. Los estilos manuales pueden dejar de funcionar tras las actualizaciones de Chat en web.                                                | [Demostración de aplicación de estilos idiosincrásicos](https://microsoft.github.io/BotFramework-WebChat/05.b.idiosyncratic-manual-styling)                    |
| [`05.c.presentation-mode-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.c.presentation-mode-styling)                                                                                           | Muestra cómo configurar el modo de presentación, que muestra el historial del chat, pero no se muestra el cuadro de envío y deshabilita la interactividad de las tarjetas adaptables.                                                                         | [Demostración del modo de presentación](https://microsoft.github.io/BotFramework-WebChat/05.c.presentation-mode-styling)                           |
| [`05.d.hide-upload-button-styling`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/05.d.hide-upload-button-styling)                                                                                         | Muestra cómo ocultar el botón de carga de archivos a través de la aplicación de estilos.                                                                                                                                                                            | [Demostración de cómo ocultar el botón de carga](https://microsoft.github.io/BotFramework-WebChat/05.d.hide-upload-button-styling)                         |
| [`06.a.cognitive-services-bing-speech-js`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.a.cognitive-services-bing-speech-js)                                                                           | Presenta la capacidad de voz a texto y texto a voz con Bing Speech API de Cognitive Services (en desuso) y JavaScript.                                                                                                      | [Demostración de Bing Speech con JS](https://microsoft.github.io/BotFramework-WebChat/06.a.cognitive-services-bing-speech-js)                 |
| [`06.b.cognitive-services-bing-speech-react`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.b.cognitive-services-bing-speech-react)                                                                     | Presenta la capacidad de voz a texto y texto a voz con Bing Speech API de Cognitive Services (en desuso) y React.                                                                                                           | [Demostración de Bing Speech con React](https://microsoft.github.io/BotFramework-WebChat/06.b.cognitive-services-bing-speech-react)           |
| [`06.c.cognitive-services-speech-services-js`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.c.cognitive-services-speech-services-js)                                                                   | Presenta la capacidad de voz a texto y texto a voz con Speech Services API de Cognitive Services.                                                                                                                                  | [Demostración de Speech Services con JS](https://microsoft.github.io/BotFramework-WebChat/06.c.cognitive-services-speech-services-js)         |
| [`06.d.speech-web-browser`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.d.speech-web-browser)                                                                                                         | Muestra cómo implementar texto a voz mediante Web Speech API basada en explorador de Chat en web. (Vincular a W3C estándar en el ejemplo)                                                                                                    | [Demostración de Web Speech API](https://microsoft.github.io/BotFramework-WebChat/06.d.speech-web-browser)                                     |
| [`06.e.cognitive-services-speech-services-with-lexical-result`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.e.cognitive-services-speech-services-with-lexical-result)                                 | Muestra cómo usar el resultado léxico de Speech Services API de Cognitive Services.                                                                                                                                                 | [Demostración del resultado léxico](https://microsoft.github.io/BotFramework-WebChat/06.e.cognitive-services-speech-services-with-lexical-result) |
| [`06.f.hybrid-speech`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/06.f.hybrid-speech)                                                                                                                   | Muestra cómo usar Web Speech API basada en explorador para voz a texto y Speech Services API de Cognitive Services para texto a voz.                                                                                        | [Demostración de voz híbrida](https://microsoft.github.io/BotFramework-WebChat/06.f.hybrid-speech)                                           |
| [`07.a.customization-timestamp-grouping`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/07.a.customization-timestamp-grouping)                                                                             | Muestra cómo personalizar las marcas de tiempo mostrando u ocultando las marcas de tiempo y cambiando la agrupación de mensajes por tiempos.                                                                                                             | [Demostración de la agrupación por marca de tiempo](https://microsoft.github.io/BotFramework-WebChat/07.a.customization-timestamp-grouping)                   |
| [`07.b.customization-send-typing-indicator`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/07.b.customization-send-typing-indicator)                                                                       | Muestra cómo enviar la actividad de escritura cuando el usuario empieza a escribir en el cuadro de envío.                                                                                                                                                | [Demostración del indicador de escritura del usuario](https://microsoft.github.io/BotFramework-WebChat/07.b.customization-send-typing-indicator)             |
| [`08.customization-user-highlighting`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/08.customization-user-highlighting)                                                                                   | Muestra cómo personalizar el estilo de las actividades en función de si el mensaje es del usuario o del bot.                                                                                                                      | [Demostración del resaltado del usuario](https://microsoft.github.io/BotFramework-WebChat/08.customization-user-highlighting)                       |
| [`09.customization-reaction-buttons`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/09.customization-reaction-buttons/)                                                                                    | Presenta la posibilidad de crear componentes personalizados para Chat en web que son únicos para las necesidades de su bot. Este tutorial muestra la capacidad de agregar un emoji de reacción como :thumbsup: y :thumbsdown: a las actividades de la conversación. | [Demostración de botones de reacción](https://microsoft.github.io/BotFramework-WebChat/09.customization-reaction-buttons)                         |
| [`10.a.customization-card-components`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components)                                                                                   | Muestra cómo crear datos adjuntos de tarjetas de actividades personalizadas; en este caso, las tarjetas de repositorio de GitHub.                                                                                                                                  | [Demostración de componentes de tarjeta](https://microsoft.github.io/BotFramework-WebChat/10.a.customization-card-components)                         |
| [`10.b.customization-password-input`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/10.b.customization-password-input)                                                                                     | Muestra cómo crear una actividad personalizada para la entrada de la contraseña.                                                                                                                                                                      | [Demostración de entrada de contraseña](https://microsoft.github.io/BotFramework-WebChat/10.b.customization-password-input)                           |
| [`11.customization-redux-actions`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/11.customization-redux-actions)                                                                                           | Tutorial avanzado: muestra cómo incorporar middleware redux en su aplicación de Chat en web enviando acciones redux a través del bot. En este ejemplo se muestra aplicación manual de estilo en función de las actividades entre el bot y el usuario.             | [Demostración de acciones Redux](https://microsoft.github.io/BotFramework-WebChat/11.customization-redux-actions)                               |
| [`12.customization-minimizable-web-chat`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/12.customization-minimizable-web-chat)                                                                             | Tutorial avanzado: muestra cómo agregar la interfaz de Chat en web a su sitio web como mostrar/ocultar cuadro de chat minimizable.                                                                                                              | [Demostración de Chat en web minimizable](https://microsoft.github.io/BotFramework-WebChat/12.customization-minimizable-web-chat)                 |
| [`13.customization-speech-ui`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/13.customization-speech-ui)                                                                                                   | Tutorial avanzado: muestra cómo personalizar completamente los componentes clave del bot, en este caso voz, lo que reemplaza completamente la interfaz de usuario de transcripción basada en texto y, en su lugar, se muestra un botón simple de voz con la respuesta del bot.      | [Demostración de la interfaz de usuario de voz](https://microsoft.github.io/BotFramework-WebChat/13.customization-speech-ui)                                       |
| [`14.customization-piping-to-redux`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/14.customization-piping-to-redux)                                                                                       | Tutorial avanzado: muestra cómo canalizar las actividades del bot para su propio almacén de Redux y utilizar su bot para controlar la página a través de las actividades del bot y Redux.                                                                          | [Demostración de canalización para Redux](https://microsoft.github.io/BotFramework-WebChat/14.customization-piping-to-redux)                           |
| [`15.a.backchannel-piggyback-on-outgoing-activities`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.a.backchannel-piggyback-on-outgoing-activities)                                                     | Tutorial avanzado: muestra cómo agregar datos personalizados a todas las actividades de salida.                                                                                                                                                | [Demostración de superposición de Backchannel](https://microsoft.github.io/BotFramework-WebChat/15.a.backchannel-piggyback-on-outgoing-activities) |
| [`15.b.incoming-activity-event`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.b.incoming-activity-event)                                                                                               | Tutorial avanzado: muestra cómo reenviar todas las actividades entrantes a un evento de JavaScript para su posterior procesamiento.                                                                                                                | [Demostración de actividad entrante](https://microsoft.github.io/BotFramework-WebChat/15.b.incoming-activity-event)                             |
| [`15.c.programmatic-post-activity`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.c.programmatic-post-activity)                                                                                         | Tutorial avanzado: muestra cómo enviar un mensaje mediante programación.                                                                                                                                                             | [Demostración de publicación mediante programación](https://microsoft.github.io/BotFramework-WebChat/15.c.programmatic-post-activity)                       |
| [`15.d.backchannel-send-welcome-event`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/15.d.backchannel-send-welcome-event)                                                                                 | Tutorial avanzado: muestra cómo enviar eventos bienvenida con capacidades de cliente como el idioma del explorador.                                                                                                                        | [Demostración de evento de bienvenida](https://microsoft.github.io/BotFramework-WebChat/15.d.backchannel-send-welcome-event)                          |
| [`16.customization-selectable-activity`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/16.customization-selectable-activity)                                                                               | Tutorial avanzado: muestra cómo agregar comportamiento personalizado del clic a cada actividad.                                                                                                                                                  | [Demostración de actividad seleccionable](https://microsoft.github.io/BotFramework-WebChat/16.customization-selectable-activity)                   |
| [`17.chat-send-history`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/17.chat-send-history)                                                                                                               | Tutorial avanzado: muestra la capacidad para guardar la entrada del usuario y permitir al usuario regresar a través de los mensajes enviados anteriormente.                                                                                                      | [Demostración de historial de envío de chat](https://microsoft.github.io/BotFramework-WebChat/17.chat-send-history)                                     |
| [`18.customization-open-url`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/18.customization-open-url)                                                                                                     | Tutorial avanzado: muestra cómo personalizar el comportamiento de abrir dirección URL.                                                                                                                                                             | [Demostración de la personalización de abrir dirección URL](https://microsoft.github.io/BotFramework-WebChat/18.customization-open-url)                               |
| [`19.a.single-sign-on-for-enterprise-apps`](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps)                                                                         | Muestra cómo usar el inicio de sesión único para aplicaciones empresariales con OAuth                                                                                                                                                              | [Inicio de sesión único con demostración de OAuth](https://microsoft.github.io/BotFramework-WebChat/19.a.single-sign-on-for-enterprise-apps)         |
 

## <a name="web-chat-api-reference"></a>Referencia de Web Chat API

Hay varias propiedades que pueden transmitirse en el componente de React del Chat en web (`<ReactWebChat>`) o el método `renderWebChat()`. No dude en examinar el código fuente a partir de [`packages/component/src/Composer.js`](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Composer.js#L378). A continuación, encontrará una breve descripción de las propiedades disponibles.

| Propiedad                   | DESCRIPCIÓN                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `activityMiddleware`       | Una cadena de middleware, modelada según el [middleware Redux](https://medium.com/@jacobp100/you-arent-using-redux-middleware-enough-94ffe991e6), que permite al desarrollador agregar nuevos componentes de DOM en el DOM de actividades existentes. La firma de middleware es la siguiente: `options => next => card => children => next(card)(children)`.                                                                                                                                                                                                                                           |
| `activityRenderer`         | La versión "plana" de `activityMiddleware`, similar al concepto [store enhancer](https://github.com/reduxjs/redux/blob/master/docs/Glossary.md#store-enhancer) en Redux.                                                                                                                                                                                                                                                                                                                                                                                                                |
| `adaptiveCardHostConfig`   | Pase una configuración personalizada de host de tarjetas adaptables. Asegúrese de comprobar la configuración del host con la versión de tarjetas adaptables que se usa. Para obtener más información, consulte la [configuración personalizada del host](https://github.com/microsoft/BotFramework-WebChat/issues/2034#issuecomment-501818238).                                                                                                                                                                                                                                                                                                                                    |
| `attachmentMiddleware`     | Una cadena de middleware que permite al desarrollador agregar sus propios elementos HTML personalizados en los datos adjuntos. La firma es la siguiente: `options => next => card => next(card)`.                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `attachmentRenderer`       | La versión "plana" de `attachmentMiddleware`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `cardActionMiddleware`     | Una cadena de middleware que permite al desarrollador modificar las acciones de la tarjeta, como las tarjetas adaptables o las acciones sugeridas. La firma de middleware es la siguiente: `cardActionMiddleware: () => next => ({ cardAction, getSignInUrl }) => next(cardAction)`                                                                                                                                                                                                                                                                                                                                           |
| `createDirectLine`         | Factory Method para crear instancias del objeto Direct Line. Los usuarios de Azure Government deben usar `createDirectLine({ domain: 'https://directline.botframework.azure.us/v3/directline', token });` para cambiar el punto de conexión. La lista completa de los parámetros es: `conversationId`, `domain`, `fetch`, `pollingInterval`, `secret`, `streamUrl`, `token`, `watermark` `webSocket`.                                                                                                                                                                                                                         |
| `createStore`              | Una cadena de middleware que permite al desarrollador modificar las acciones de almacén. La firma de middleware es la siguiente: `createStore: ({}, ({ dispatch }) => next => action => next(cardAction)`                                                                                                                                                                                                                                                                                                                                                                                                |
| `directLine`               | Especifique el objeto DirectLine con el token DirectLine.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `disabled`                 | Deshabilite la interfaz de usuario (es decir, para el modo de presentación) de Chat en web.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `grammars`                 | Especifique una lista de gramática para voz (Bing Speech o servicios de voz de Cognitive Services).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `groupTimeStamp`           | Cambie la configuración predeterminada de las agrupaciones de marca de tiempo.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `locale`                   | Indique el idioma predeterminado de Chat en web. Se recomiendan encarecidamente códigos de cuatro letras (como `en-US`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `renderMarkdown`           | Cambie el objeto representador de Markdown predeterminado.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `sendTypingIndicator`      | Muestre una señal de escritura del usuario al bot para indicar que el usuario no está inactivo.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `store`                    | Especifique un almacén personalizado, por ejemplo, para agregar la actividad mediante programación al bot.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `styleOptions`             | Objeto que almacena los valores de personalización de su estilo de Chat en web. Para obtener una lista completa de opciones de estilo predeterminado (actualizada con frecuencia), consulte el archivo [defaultStyleOptions.js](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js).                                                                                                                                                                                                                                                                              |
| `styleSet`                 | La manera no recomendada para invalidar estilos.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `userID`                   | Especifique un userID. Hay dos maneras de especificar `userID`: en propiedades o en el token al generar la llamada al token (`createDirectLine()`). Si se usan ambos métodos para especificar userID, se usará la propiedad userID del token y `console.warn` aparecerá en tiempo de ejecución. Si `userID` se proporciona a través de propiedades, pero tiene como prefijo `'dl'`, por ejemplo, `'dl_1234'`, el valor será eliminará y se generará un `ID` nuevo. Si no se especifica `userID`, el valor predeterminado será un identificador de usuario aleatorio. No se recomienda que varios usuarios compartan el mismo id. de usuario; se compartirá su estado de usuario. |
| `username`                 | Especifique un nombre de usuario.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `webSpeechPonyFillFactory` | Especifique el objeto Voz web para texto a voz y voz a texto.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
## <a name="browser-compatibility"></a>Compatibilidad con el explorador
Chat en web admite las 2 últimas versiones de los exploradores modernos como Chrome, Edge y FireFox.
Si necesita Chat en web en Internet Explorer 11, vea [Demostración del conjunto ES5](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle).

No obstante, tenga en cuenta:
- Chat en web no es compatible con una instancia de Internet Explorer anterior a la versión 11
- No se admite la personalización tal como se muestra en los ejemplos que no son de ES5 para Internet Explorer. Dado que IE11 no es un explorador moderno, no es compatible con ES6 y muchos ejemplos que utilizan funciones de flecha y promesas modernas tendrían que convertirse manualmente a ES5.  Si necesita una personalización intensa de la aplicación, se recomienda encarecidamente desarrollar la aplicación para un explorador moderno, como Google Chrome o Edge.
- Chat en web no tiene ningún plan para admitir ejemplos de IE11 (ES5).
   - Para los clientes que deseen volver a escribir manualmente nuestros otros ejemplos para trabajar en IE11, se recomienda revisar la conversión de código ES6+ a ES5 con polyfills y transpilers como [`babel`](https://babeljs.io/docs/en/next/babel-standalone.html).

## <a name="how-to-test-with-web-chats-latest-bits"></a>Cómo realizar pruebas con los bits más recientes del Chat en web

*La prueba de características no publicadas solo está disponible a través del empaquetado de MyGet en este momento.*

Si desea probar una característica o corrección de error que aún no se ha publicado, el paquete de Chat en web tendrá que apuntar a la fuente diaria de Chat en web, en lugar de la fuente npmjs oficial.

Actualmente, puede tener acceso a fuentes diarias de Chat en web suscribiéndose a nuestra fuente MyGet. Para ello, deberá actualizar el registro en el proyecto. **Este cambio es irreversible y nuestras instrucciones incluyen cómo volver a suscribirse a la versión oficial**.

### <a name="subscribe-to-latest-bits-on-mygetorg"></a>Suscribirse a los bits más recientes en `myget.org`

Para ello puede agregar los paquetes y, a continuación, cambiar el registro del proyecto.

1. Agregue las dependencias del proyecto que no sean Chat en web.
1. En el directorio raíz del proyecto, cree un archivo `.npmrc`.
1. Agregue la siguiente línea al archivo: `registry=https://botbuilder.myget.org/F/botframework-webchat/npm/`
1. Agregue Chat en web a las dependencias del proyecto `npm i botframework-webchat --save`.
1. Tenga en cuenta que en su `package-lock.json`, los registros a los que se apuntan ahora son MyGet. El proyecto de Chat en web tiene el origen ascendente habilitado para proxy, que redirigirá los paquetes no MyGet a `npmjs.com`.

### <a name="re-subscribe-to-official-release-on-npmjscom"></a>Volver a suscribirse a la versión oficial en `npmjs.com`
Volver a suscribirse requiere que restablezca el registro.

1. Elimine su `.npmrc file`.
1. Elimine su `package-lock.json` raíz.
1. Quite el directorio `node_modules`.
1. Vuelva a instalar los paquetes con `npm i`
1. Tenga en cuenta que en su `package-lock.json`, los registros apuntan a https://npmjs.com/ de nuevo.


## <a name="contributing"></a>Contribuir

Consulte nuestra [página de contribución](https://github.com/Microsoft/BotFramework-WebChat/tree/master/.github/CONTRIBUTING.md) para obtener más información sobre cómo compilar el proyecto y nuestras directrices de repositorio para las solicitudes de incorporación de cambios.

Este proyecto ha adoptado el [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/) (Código de conducta de código abierto de Microsoft).
Para más información, consulte las [preguntas más frecuentes del código de conducta](https://opensource.microsoft.com/codeofconduct/faq/) o póngase en contacto con [opencode@microsoft.com](mailto:opencode@microsoft.com) si tiene cualquier otra pregunta o comentario.

## <a name="reporting-security-issues"></a>Informe de problemas de seguridad

Las incidencias y problemas de seguridad se deben informar de forma privada, por correo electrónico al Centro de respuestas de seguridad de Microsoft (MSRC) en [secure@microsoft.com](mailto:secure@microsoft.com). Debería recibir una respuesta en 24 horas. Si por algún motivo no es así, haga un seguimiento por correo electrónico para asegurarse de que se recibió el mensaje original. Puede encontrar más información, incluida la clave [MSRC PGP](https://technet.microsoft.com/security/dn606155), en [TechCenter de seguridad](https://technet.microsoft.com/security/default).

Copyright (c) Microsoft Corporation. Todos los derechos reservados.
