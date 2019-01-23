---
title: Personalización de un formulario mediante FormBuilder | Microsoft Docs
description: Obtenga información sobre cómo cambiar y personalizar de forma dinámica el flujo y el contenido de las conversaciones con FormBuilder para Bot Framework SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8a13358afca190606e235475a58f6aedd146fee5
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225410"
---
# <a name="customize-a-form-using-formbuilder"></a>Personalización de un formulario mediante FormBuilder

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

En [Características básicas de FormFlow](bot-builder-dotnet-formflow.md) se describe una implementación básica de FormFlow que ofrece una experiencia de usuario más o menos genérica y en [Características avanzadas de FormFlow](bot-builder-dotnet-formflow-advanced.md) se describe cómo puede personalizar la experiencia del usuario mediante el uso de atributos y lógica de negocios. En este artículo se describe cómo puede usar [FormBuilder][formBuilder] para personalizar la experiencia del usuario aún más, especificando la secuencia en que el formulario ejecuta los pasos y definiendo de forma dinámica los valores de campos, las confirmaciones y los mensajes. 

## <a name="dynamically-define-field-values-confirmations-and-messages"></a>Definición dinámica de valores de campo, confirmaciones y mensajes

Con FormBuilder, puede definir de forma dinámica valores de campo, confirmaciones y mensajes.

### <a name="dynamically-define-field-values"></a>Definición dinámica de valores de campo 

Un bot de sándwich diseñado para agregar una bebida o galleta gratuitas a cualquier pedido que especifique un sándwich grande usa el campo `Sandwich.Specials` para almacenar datos sobre artículos gratuitos. En este caso, el valor del campo `Sandwich.Specials` debe establecerse de forma dinámica para cada pedido en función e si el pedido contiene o no un sándwich grande. 

El campo `Specials` se especifica como opcional y "Ninguno" se designa como texto para la opción que no indica ninguna preferencia.

[!code-csharp[Field definition](../includes/code/dotnet-formflow-formbuilder.cs#fieldDefinition)]

Este ejemplo de código muestra cómo establecer el valor del campo `Specials` de forma dinámica. 

[!code-csharp[Define value](../includes/code/dotnet-formflow-formbuilder.cs#defineValue)]

En este ejemplo, el método [Advanced.Field.SetType][setType] especifica el tipo de campo (`null` representa un campo de enumeración). El método [Advanced.Field.SetActive][setActive] especifica que el campo solo debe habilitarse si la longitud del sándwich es `Length.FootLong`. Por último, el método [Advanced.Field.SetDefine][setDefine] especifica un delegado asincrónico que define el campo. Al delegado se le pasa el objeto de estado actual y el valor [Advanced.Field][field] definido de forma dinámica. El delegado usa los métodos fluidos del campo para definir los valores de forma dinámica. En este ejemplo, los valores son cadenas y los métodos `AddDescription` y `AddTerms` especifican las descripciones y los términos de cada valor.

> [!NOTE]
> Para definir un valor de campo de forma dinámica, puede implementar [Advanced.IField][iField] por sí mismo o agilizar el proceso con el uso de la clase [Advanced.FieldReflector][FieldReflector] como se muestra en el ejemplo anterior. 

### <a name="dynamically-define-messages-and-confirmations"></a>Definición dinámica de mensajes y confirmaciones

Con FormBuilder, también puede definir de forma dinámica confirmaciones y mensajes. Cada mensaje y confirmación se ejecuta solo cuando los pasos anteriores del formulario están inactivos o completados. 

Este ejemplo de código muestra una confirmación generada de forma dinámica que calcula el costo del sándwich. 

[!code-csharp[Define confirmation](../includes/code/dotnet-formflow-formbuilder.cs#defineConfirmation)]

## <a name="customize-a-form-using-formbuilder"></a>Personalización de un formulario mediante FormBuilder

Este ejemplo de código usa FormBuilder para definir los pasos del formulario, [validar las selecciones](bot-builder-dotnet-formflow-advanced.md#add-business-logic) y [definir de forma dinámica un valor de campo y una confirmación](#dynamically-define-field-values-confirmations-and-messages). De forma predeterminada, los pasos del formulario se ejecutarán en la secuencia en que están enumerados. Sin embargo, se pueden omitir los pasos para los campos que ya contienen valores o si se especifica la navegación explícita. 

[!code-csharp[FormBuilder form](../includes/code/dotnet-formflow-formbuilder.cs#formBuilderForm)]

En este ejemplo, el formulario ejecuta estos pasos:

- Muestra un mensaje de bienvenida. 
- Rellena `SandwichOrder.Sandwich`. 
- Rellena `SandwichOrder.Length`. 
- Rellena `SandwichOrder.Bread`. 
- Rellena `SandwichOrder.Cheese`. 
- Rellena `SandwichOrder.Toppings` y agrega los valores que faltan si el usuario seleccionó `ToppingOptions.Everything`. -. Muestra un mensaje que confirma los ingredientes seleccionados. 
- Rellena `SandwichOrder.Sauces`. 
- [Define de forma dinámica](#dynamically-define-field-values) el valor del campo para `SandwichOrder.Specials`. 
- [Define de forma dinámica](#dynamically-define-messages-and-confirmations) la confirmación del costo del sándwich. 
- Rellena `SandwichOrder.DeliveryAddress` y [verifica](bot-builder-dotnet-formflow-advanced.md#add-business-logic) la cadena resultante. Si la dirección no empieza con un número, el formulario devuelve un mensaje. 
- Rellena `SandwichOrder.DeliveryTime` con un mensaje personalizado. 
- Confirma el pedido. 
- Agrega los campos restantes definidos en la clase pero a los que `Field` no hace referencia de forma explícita. (Si el ejemplo no llamó al método `AddRemainingFields`, el formulario no incluiría ningún campo al que no se hiciera referencia de forma explícita). 
- Muestra un mensaje de agradecimiento. 
- Define un controlador `OnCompletionAsync` para procesar el pedido. 

## <a name="sample-code"></a>Código de ejemplo

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Recursos adicionales

- [Características básicas de FormFlow](bot-builder-dotnet-formflow.md)
- [Características avanzadas de FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Localización del contenido del formulario](bot-builder-dotnet-formflow-localize.md)
- [Definición de un formulario mediante esquemas JSON](bot-builder-dotnet-formflow-json-schema.md)
- [Personalización de la experiencia de usuario con lenguaje de patrones](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1

[setType]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.settype

[setActive]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setactive

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[FieldReflector]: /dotnet/api/microsoft.bot.builder.formflow.advanced.fieldreflector-1
