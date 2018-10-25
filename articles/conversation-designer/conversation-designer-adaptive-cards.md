---
title: Configuración de tarjetas adaptables | Microsoft Docs
description: Aprenda a configurar tarjetas adaptables.
author: vkannan
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: d41c2c24ed38fffe76cd73a6bb8a685d3861ac55
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000432"
---
# <a name="configure-adaptive-cards"></a>Configuración de tarjetas adaptables
> [!IMPORTANT]
> Conversation Designer no está disponible para todos los clientes todavía. Habrá más información sobre la disponibilidad de Conversation Designer más adelante este año.

Las <a href="http://adaptivecards.io" target="_blank">tarjetas adaptables</a> son un nuevo esquema que define tarjetas de interfaz de usuario enriquecidas para su uso en varios puntos de conexión diferentes, incluidos los canales de Microsoft Bot Framework. 

Conversation Designer proporciona un entorno de creación profundamente integrado para crear, usar y obtener una vista previa de tarjetas adaptables en los bots. 

Las tarjetas adaptables se pueden definir en distintos lugares clave.

- Una respuesta sencilla a una acción para una tarea.
- En estado de comentario en un diálogo.
- En los estados de aviso, en un diálogo. Tenga en cuenta que los avisos pueden tener tarjetas distintas: una para la respuesta y otra para volver a pedir información.

Para definir una tarjeta adaptable, vaya al editor correspondiente. Examine y elija una de las plantillas de tarjeta adaptable existentes o cree la suya propia en el editor de código JSON. 

A medida que se crea una tarjeta, se representa una vista previa completa de ella en el portal de creación.

> [!NOTE]
> Las características de las tarjetas adaptables permanecen en desarrollo continuado. En este momento, no todos los canales admiten todas las características de las tarjetas adaptables. Para ver qué características admite cada canal, consulte la sección Estado del canal.

## <a name="input-form"></a>Formulario de entrada

Las tarjetas adaptables pueden contener formularios de entrada. En Conversation Designer, los formularios se integran con entidades de tareas. Por ejemplo, si un campo tiene un `id` de **myName** y se realiza la acción `Submit` del formulario, se crea un elemento `taskEntity` con el nombre **myName** que contendrá el valor del campo. 

El fragmento de código siguiente muestra cómo la entidad **myName** se define en el código:

``javascript
{
   "type": "Input.Text",
   "id": "myName",
   "placeholder": "Last, First"
}
``

Además, si un campo tiene un identificador de `@task`, entonces el valor del campo se usa como un nombre de tarea. Cuando se desencadena este campo (p. ej.: con un clic de botón), se ejecuta entonces la tarea designada. 

Vamos a tomar este fragmento de código como ejemplo:

``javascript
{
  'type': 'Action.Submit',
  'title': 'Search',
  'speak': '<s>Search</s>',
  'data': {
    '@task': 'Hotel Search'
  }
}
``

Cuando se hace clic en este botón, se desencadena una acción de envío y `context.sticky` se establece en `Hotel Search`. Como consecuencia, se ejecuta la tarea **Hotel Search**. Para usar esta característica, asegúrese de que `@task` coincida con un nombre de tarea que haya definido en Conversation Designer.

## <a name="use-entities-and-language-generation-templates"></a>Uso de entidades y plantillas de generación de idioma
Las tarjetas adaptables admiten la resolución de generación de idioma completo.

* `entityName` usa entidades dentro de la tarjeta.
* `responseTemplateName` usa plantillas de respuesta simple o condicional dentro de la tarjeta.

Más información acerca de las tarjetas adaptables aquí TODO: Insertar vínculo a la documentación del esquema de las tarjetas adaptables -->

## <a name="sample-adaptive-card-payload"></a>Ejemplo de carga de tarjeta adaptable

El siguiente código JSON muestra la carga de una tarjeta adaptable.

```json
{
    "$schema": "https://microsoft.github.io/AdaptiveCards/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.0",
    "body": [
        {
            "speak": "<s>Serious Pie is a Pizza restaurant which is rated 9.3 by customers.</s>",
            "type": "ColumnSet",
            "columns": [
                {
                    "type": "Column",
                    "size": "2",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "[Greeting], [TimeOfDayTemplate], You can eat in {location}",
                            "weight": "bolder",
                            "size": "extraLarge"
                        },
                        {
                            "type": "TextBlock",
                            "text": "9.3 · $$ · Pizza",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "text": "[builtin.feedback.display]",
                            "wrap": true
                        }
                    ]
                },
                {
                    "type": "Column",
                    "size": "1",
                    "items": [
                        {
                            "type": "Image",
                            "url": "http://res.cloudinary.com/sagacity/image/upload/c_crop,h_670,w_635,x_0,y_0/c_scale,w_640/v1397425743/Untitled-4_lviznp.jpg",
                            "size":"auto"
                        }
                    ]
                }
            ]
        }
    ],
    "actions": [
        {
            "type": "Action.Http",
            "method": "POST",
            "title": "More Info",
            "url": "http://foo.com"
        },
        {
            "type": "Action.Http",
            "method": "POST",
            "title": "View on Foursquare",
            "url": "http://foo.com"
        }
    ]
}
```

