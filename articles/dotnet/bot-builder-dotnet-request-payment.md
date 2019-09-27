---
title: Solicitud de pago | Microsoft Docs
description: Aprenda a enviar una solicitud de pago mediante Bot Framework SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ee7711ab15da9fbbb42d857b216a6099e44880e4
ms.sourcegitcommit: 378dbffd3960a1fa063ffb314878ccd64fb8fb49
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/18/2019
ms.locfileid: "71094435"
---
# <a name="request-payment"></a>Solicitud de pago

> [!NOTE]
> El servicio de pago al que se hace referencia en este artículo dejará de usarse el 1 de diciembre de 2019.

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-request-payment.md)
> - [Node.js](../nodejs/bot-builder-nodejs-request-payment.md)

Si su bot permite a los usuarios comprar artículos, puede solicitar el pago mediante la inclusión de un tipo especial de botón dentro de una [tarjeta enriquecida](bot-builder-dotnet-add-rich-card-attachments.md). En este artículo se describe cómo enviar una solicitud de pago mediante Bot Framework SDK para .NET.

## <a name="prerequisites"></a>Requisitos previos

Para poder enviar una solicitud de pago mediante Bot Framework SDK para .NET, debe completar estas tareas previas.

### <a name="update-webconfig"></a>Actualizar el archivo web.config

Actualice el archivo **web.config** del bot para `MicrosoftAppId` y `MicrosoftAppPassword` en los valores de identificador y contraseña de la aplicación que se generaron para el bot durante el proceso de [registro](~/bot-service-quickstart-registration.md). 

> [!NOTE]
> Para buscar los valores **AppID** y **AppPassword** del bot, consulte [MicrosoftAppID y MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

### <a name="create-and-configure-merchant-account"></a>Crear y configurar la cuenta de comerciante

1. <a href="https://dashboard.stripe.com/register" target="_blank">Cree y active una cuenta de Stripe si todavía no tiene una.</a>

2. <a href="https://seller.microsoft.com/dashboard/registration/seller/?accountprogram=botframework" target="_blank">Inicie sesión en el Centro de vendedores con su cuenta Microsoft.</a>

3. En el Centro de vendedores, conecte su cuenta con Stripe.

4. En el Centro de vendedores, vaya al panel y copie el valor de **MerchantID**.

5. Actualice el archivo **Web.config** del bot para establecer `MerchantId` en el valor que ha copiado del panel del Centro de vendedores. 

[!INCLUDE [Payment process overview](../includes/snippet-payment-process-overview.md)]

## <a name="payment-bot-sample"></a>Ejemplo del bot de pago

El ejemplo de <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">bot de pago</a> proporciona un ejemplo de un bot que envía una solicitud de pago mediante .NET. Para ver este bot de ejemplo en acción, puede <a href="https://webchat.botframework.com/embed/paymentsample?s=d39Bk7JOMzQ.cwA.Rig.dumLki9bs3uqfWFMjXPn5PFnQVmT2VAVR1Zl1iPi07k" target="_blank">probarlo en chat web</a>, <a href="https://join.skype.com/bot/9fbc0f17-43eb-40fe-bf3b-af151e6ce45e" target="_blank">agregarlo como contacto de Skype</a>, o descargar el ejemplo de bot de pago y ejecutarlo localmente con Bot Framework Emulator. 

> [!NOTE]
> Para completar el proceso de pago de un extremo a otro mediante el ejemplo de **bot de pago** en un chat web o Skype, debe especificar una tarjeta de crédito o de débito válidas desde la cuenta Microsoft (es decir, una tarjeta válida de un emisor de tarjetas de Estados Unidos). No se le cobrará a la tarjeta y el CVV de la tarjeta no se comprobará, ya que el ejemplo del **bot de pago** se ejecuta en modo de prueba (es decir, `LiveMode` está establecido en `false` en **Web.config**).

En las secciones siguientes de este artículo se describen las tres partes del proceso de pago, en el contexto del ejemplo del **bot de pago**.

## <a id="request-payment"></a> Solicitar el pago

Para que el bot solicite el pago a un usuario, puede enviar un mensaje que contenga una [tarjeta enriquecida adjunta](bot-builder-dotnet-add-rich-card-attachments.md) con un botón que especifique `CardAction.Type` de "pago". En este fragmento de código del ejemplo del **bot de pago** se crea un mensaje que contiene una tarjeta principal con un botón **Buy** (Comprar) donde el usuario puede hacer clic (o pulsar) para iniciar el proceso de pago. 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#requestPayment)]

En este ejemplo, el tipo del botón se especifica como `PaymentRequest.PaymentActionType`, que la biblioteca de Bot Builder define como "pago". El valor del botón se rellena con el método `BuildPaymentRequest`, que devuelve un objeto `PaymentRequest` que contiene información sobre los métodos de pago admitidos, detalles y opciones. Para más información acerca de los detalles de implementación, consulte **Dialogs/RootDialog.cs** dentro del ejemplo del <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">bot de pago</a>.

Esta captura de pantalla muestra la tarjeta principal prominente (con el botón **Buy**) generada por el fragmento de código anterior. 
 
![Bot de ejemplo de pagos](../media/payments-bot-buy.png) 

> [!IMPORTANT]
> Cualquier usuario que tenga acceso al botón **Buy** puede usarlo para iniciar el proceso de pago. En el contexto de una conversación grupal, no es posible designar un botón para su uso solo por un usuario específico. 

## <a id="user-experience"></a> Experiencia del usuario

Cuando un usuario hace clic en el botón **Buy**, se dirige a este a la experiencia web de pago para que proporcione toda la información necesaria de pago, de envío y de contacto mediante su cuenta Microsoft. 

![Pago de Microsoft](../media/microsoft-payment.png)

### <a name="http-callbacks"></a>Devoluciones de llamadas HTTP

Las devoluciones de llamada HTTP se enviarán al bot para indicar que debe realizar determinadas operaciones. Cada devolución de llamada será una [actividad](bot-builder-dotnet-activities.md) que contiene estos valores de propiedad: 

| Propiedad | Valor |
|----|----|
| `Activity.Type` | invoke | 
| `Activity.Name` | Indica el tipo de operación que debe realizar el bot (p. ej., actualización de la dirección de envío, actualización de la opción de envío, finalización del pago). | 
| `Activity.Value` | La carga útil de la solicitud en formato JSON. | 
| `Activity.RelatesTo` |  Describe el canal y el usuario que están asociados con la solicitud de pago. | 

> [!NOTE]
> `invoke` es un tipo de actividad especial que está reservado para uso en Microsoft Bot Framework. El remitente de una actividad `invoke` esperará que su bot confirme la devolución de llamada mediante el envío de una respuesta HTTP.

## <a id="process-callbacks"></a> Procesamiento de las devoluciones de llamada

[!INCLUDE [Process callbacks overview](../includes/snippet-payment-process-callbacks-overview.md)]

### <a name="shipping-address-update-and-shipping-option-update-callbacks"></a>Devoluciones de llamada de actualización de dirección de envío y actualización de opción de envío

[!INCLUDE [Process shipping address and shipping option callbacks](../includes/snippet-payment-process-callbacks-1.md)]

### <a name="payment-complete-callbacks"></a>Devoluciones de llamada de finalización de pago

Al recibir una devolución de llamada de finalización de pago, el bot le proporcionará una copia de la solicitud de pago inicial, sin modificar, así como los objetos de la respuesta de pago en la propiedad `Activity.Value`. El objeto de respuesta de pago contendrá las selecciones finales realizadas por el cliente junto con un token de pago. El bot debe tener la oportunidad de recalcular la solicitud de pago final en función de la solicitud de pago inicial y las selecciones finales del cliente. Suponiendo que se determina que las selecciones del cliente son válidas, el bot debería comprobar el importe y la moneda en el encabezado del token de pago para asegurarse de que coincidan con la solicitud de pago final.  Si el bot decide cobrar al cliente, solo debe cobrar el importe en el encabezado del token de pago, ya que este es el precio que confirmó el cliente. Si hay una discrepancia entre los valores que espera el bot y los valores que recibió en la devolución de llamada de finalización de pago, se puede producir un error en la solicitud de pago mediante el envío del código de estado HTTP `200 OK`, además de establecer el campo de resultados en `failure`.   

Además de comprobar los detalles de pago, el bot también debe comprobar que el pedido puede cumplirse, antes de iniciar el procesamiento de pagos. Por ejemplo, puede que desee comprobar que los elementos que se compran siguen estando en existencias. Si los valores son correctos y el procesador de pago ha cargado correctamente el token de pago, a continuación, el bot debería responder con el código de estado HTTP `200 OK`, además de establecer el campo de resultados en `success` a fin de que la experiencia web de pago muestre la confirmación de pago. El token de pago que recibe el bot solo puede usarse una vez, por parte del comerciante que lo ha solicitado, y debe enviarse a Stripe (el único procesador de pagos que admite actualmente Bot Framework). El envío de cualquier código de estado HTTP en el rango de `400` o `500` producirá un error genérico para el cliente.

El método `OnInvoke` del ejemplo del **bot de pago** procesa las devoluciones de llamada que recibe el bot. 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#processCallback)]

En este ejemplo, el bot examina la propiedad `Name` de la actividad entrante para identificar el tipo de operación que debe realizarse y, a continuación, llama al método adecuado para procesar la devolución de llamada. Para más información acerca de los detalles de implementación, consulte **Controllers/MessagesControllers.cs** dentro del ejemplo del <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">bot de pago</a>.

## <a name="testing-a-payment-bot"></a>Probar un bot de pago

[!INCLUDE [Test a payment bot](../includes/snippet-payment-test-bot.md)]

En el ejemplo del <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">bot de pago</a>, los valores de configuración de `LiveMode` en **Web.config** determinan si las devoluciones de llamada de finalización de pago contendrán tokens de pago emulados o tokens de pago reales. Si `LiveMode` está establecido en `false`, se agrega un encabezado a la solicitud de pago saliente del bot para indicar que el bot está en modo de prueba, y la devolución de llamada de finalización de pago contendrá un token de pago emulado que no se puede cargar. Si `LiveMode` está establecido en `true`, el encabezado que indica que el bot está en modo de prueba se omite de la solicitud de pago saliente del bot, y la devolución de llamada de finalización de pago contendrá un token de pago real que el bot enviará a Stripe para el procesamiento del pago. Se trata de una transacción real que genera cargos en el instrumento de pago especificado. 

## <a name="additional-resources"></a>Recursos adicionales

- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">Ejemplo del bot de pago</a>
- [Introducción a las actividades](bot-builder-dotnet-activities.md)
- [Incorporación de tarjetas enriquecidas a mensajes](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="http://www.w3.org/Payments/" target="_blank">Pagos web en W3C</a> 
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>
