---
title: Personalización de Chat en web | Microsoft Docs
description: Obtenga información sobre cómo personalizar Chat en web de Bot Framework.
keywords: bot framework, chat en web, chat, ejemplos, react, referencia
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 06/07/2019
ms.openlocfilehash: 9e61e7e9d7bc6ba06d4c239cbf8b0dc0c4e7e16e
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299095"
---
# <a name="web-chat-customization"></a>Personalización de Chat en web

En este artículo se detalla ejemplos de cómo personalizar Chat en web para ajustarse a su bot.

## <a name="integrate-web-chat-into-your-website"></a>Integrar Chat en web en el sitio web

Siga las instrucciones de la [página de información general](bot-builder-webchat-overview.md) para integrar el control de Chat en web en su sitio web.

## <a name="customizing-styles"></a>Personalización de estilos

La versión más reciente del control Chat en web proporciona abundantes opciones de personalización: puede cambiar los colores, los tamaños y la colocación de los elementos, agregar elementos personalizados e interactuar con la página web de hospedaje. A continuación, se muestran varios ejemplos de cómo personalizar los elementos de la interfaz de usuario Chat en web.

Puede encontrar la lista completa de todas las configuraciones que puede modificar fácilmente en Chat en web en el archivo [`defaultStyleOptions.js` ](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js).

Esta configuración generará un _estilo establecido_, que es un conjunto de reglas CSS mejoradas con [glamor](https://github.com/threepointone/glamor). Puede encontrar la lista completa de los estilos CSS generados en el estilo establecido en el archivo [`createStyleSet.js` ](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/createStyleSet.js).

## <a name="set-the-size-of-the-web-chat-container"></a>Establecer el tamaño del contenedor de Chat en web

Ahora es posible ajustar el tamaño del contenedor de Chat en web mediante `styleSetOptions`. En el siguiente ejemplo, el color de fondo de `body` es `paleturquoise` para mostrar el contenedor de Chat en web (sección con fondo blanco).

```js
…
<head>
  <style>
    html, body { height: 100% }
    body {
      margin: 0;
      background-color: paleturquoise;
    }

    #webchat {
      height: 100%;
      width: 100%;
    }
  </style>
</head>
<body>
  <div id="webchat" role="main"></div>
  <script>
    (async function () {
    window.WebChat.renderWebChat({
      directLine: window.WebChat.createDirectLine({ token }),
      styleOptions: {
        rootHeight: '100%',
        rootWidth: '50%'
      }
    }, document.getElementById('webchat'));
    })()
  </script>
…
```

El resultado es este:

<img alt="Web Chat with root height and root width set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/rootHeightWidth.png" width="600"/>

## <a name="change-font-or-color"></a>Cambiar la fuente o color

En lugar de usar el color de fondo predeterminado y las fuentes utilizadas dentro de las burbujas de chat, puede personalizarlos para que coincidan con el estilo de la página web de destino. El siguiente fragmento de código permite cambiar el color de fondo de los mensajes del usuario y del bot.

<img alt="Screenshot with custom style options" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-options.png" width="396" />

Si necesita hacer algunos cambios de estilo simples, puede establecerlos mediante `styleOptions`. Las opciones de estilo son un conjunto de estilos predefinidos que se pueden modificar directamente y Chat en web calculará toda la hoja de estilos basada en ellas.

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         const styleOptions = {
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleOptions' when rendering Web Chat
               styleOptions
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-css-manually"></a>Cambiar CSS manualmente

Además de los colores, puede modificar las fuentes que usa para representar los mensajes:

<img alt="Screenshot with custom style set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-set.png" width="396" />

Para cambiar el estilo en profundidad, también puede modificar de forma manual el estilo establecido mediante la configuración de las reglas de CSS directamente.

> Puesto que las reglas de CSS están estrechamente unidas a la estructura del árbol DOM, existe la posibilidad de que estas reglas deban actualizarse para que funcionen con la versión más reciente de Chat en web.

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         // "styleSet" is a set of CSS rules which are generated from "styleOptions"
         const styleSet = window.WebChat.createStyleSet({
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         });

         // After generated, you can modify the CSS rules
         styleSet.textContent = {
            ...styleSet.textContent,
            fontFamily: "'Comic Sans MS', 'Arial', sans-serif",
            fontWeight: 'bold'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleSet' when rendering Web Chat
               styleSet
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-avatar-of-the-bot-within-the-dialog-box"></a>Cambiar el avatar del bot en el cuadro de diálogo

El Chat en web más reciente admite avatares y puede personalizarlos con las propiedades `botAvatarInitials` y `userAvatarInitials`.

<img alt="Screenshot with avatar initials" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-avatar-initials.png" width="396" />

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
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing avatar initials when rendering Web Chat
               botAvatarInitials: 'BF',
               userAvatarInitials: 'WC'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

Dentro del código `renderWebChat`, agregamos `botAvatarInitials` y `userAvatarInitials`:

```js
botAvatarInitials: 'BF',
userAvatarInitials: 'WC'
```

`botAvatarInitials` establecerá el texto dentro del avatar en el lado izquierdo. Si se establece en un valor falso, se ocultará el avatar en el lado del bot. En cambio, `userAvatarInitials` establecerá el texto del avatar en el lado derecho.

## <a name="custom-rendering-activity-or-attachment"></a>Personalizar la representación de actividad o datos adjuntos

Con la versión más reciente de Chat en web, también puede representar las actividades o los datos adjuntos que Chat en web no admite de fábrica. La representación de actividades y datos adjuntos se envía a través de una canalización personalizable que se modela según el [middleware Redux](https://redux.js.org/api/applymiddleware). La canalización es lo suficientemente flexible como para que pueda realizar fácilmente las tareas siguientes:

-  Decorar actividades y datos adjuntos existentes
-  Agregar nuevas actividades y datos adjuntos
-  Reemplazar (o quitar) actividades o datos adjuntos existentes
-  Conectar en cadena middleware

### <a name="show-github-repository-as-an-attachment"></a>Mostrar el repositorio de GitHub como datos adjuntos

Si desea mostrar una baraja de tarjetas del repositorio de GitHub, puede crear un nuevo componente React para el repositorio de GitHub y agregarlo como middleware para datos adjuntos.

<img alt="Screenshot with custom GitHub repository attachment" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-github-repository-attachment.png" width="396" />

```jsx
import ReactWebChat from 'botframework-webchat';
import ReactDOM from 'react-dom';

// Create a new React component that accept render a GitHub repository attachment
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);

// Creating a new middleware pipeline that will render <GitHubRepositoryAttachment> for specific type of attachment
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};

ReactDOM.render(
   <ReactWebChat
      // Prepending the new middleware pipeline
      attachmentMiddleware={attachmentMiddleware}
      directLine={window.WebChat.createDirectLine({ token })}
   />,
   document.getElementById('webchat')
);
```

El ejemplo completo se puede encontrar en el [ejemplo customization-card-components](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components).

En este ejemplo, agregamos un nuevo componente React denominado `GitHubRepositoryAttachment`:

```jsx
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);
```

A continuación, creamos un middleware que representará el nuevo componente React cuando el bot envíe datos adjuntos del tipo de contenido `application/vnd.microsoft.botframework.samples.github-repository`. En caso contrario, continuará en el middleware mediante una llamada a `next(card)`.

```jsx
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};
```

La actividad que se envía desde el bot es similar a la siguiente:

```json
{
   "type": "message",
   "from": {
      "role": "bot"
   },
   "attachmentLayout": "carousel",
   "attachments": [
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-WebChat"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-Emulator"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-DirectLineJS"
         }
      }
   ]
}
```
