---
ms.openlocfilehash: f09d0a7b81e3cfa69fd42356faf27f79e3bc038c
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563671"
---
Cuando se recibe una devolución de llamada de actualización de dirección de envío o de actualización de opción de envío, se indicará al bot el estado actual de los detalles de pago del cliente en la propiedad `Activity.Value`.
Como comerciante, debe tratar estas devoluciones de llamada como estáticas; dados los detalles de pago de entrada, calculará algunos detalles de pago de salida y se producirá un error si el estado de entrada proporcionado por el cliente no es válido por algún motivo. 
Si el bot determina que la información especificada es válida tal cual, simplemente envíe el código de estado HTTP `200 OK` junto con los detalles de pago sin modificar. Como alternativa, el bot puede enviar el código de estado HTTP `200 OK` junto con los detalles de pago actualizados que deben aplicarse antes de que se pueda procesar el pedido. En algunos casos, el bot puede determinar que la información actualizada no es válida y que el pedido no puede procesarse tal cual. Por ejemplo, la dirección de envío del usuario puede especificar un país al que el proveedor del producto no realiza envíos. En ese caso, el bot puede enviar el código de estado HTTP `200 OK` y un mensaje que rellena la propiedad de error del objeto de detalles de pago. El envío de cualquier código de estado HTTP en el rango de `400` o `500` producirá un error genérico para el cliente.
