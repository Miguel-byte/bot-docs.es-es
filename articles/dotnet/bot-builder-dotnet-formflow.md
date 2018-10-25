---
title: Características básicas de FormFlow | Microsoft Docs
description: Aprenda a guiar los flujos de conversación con FormFlow en el SDK de Bot Builder para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f5b36e1f916539b78f9bdcdd0970317db723f408
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000382"
---
# <a name="basic-features-of-formflow"></a>Características básicas de FormFlow

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Los [cuadros de diálogo ](bot-builder-dotnet-dialogs.md) son extremadamente eficaces y flexibles, pero administrar una conversación guiada, como ordenar un sándwich, puede requerir un gran esfuerzo. En cada punto de la conversación se puede dar una gran cantidad de posibilidades, como las que se detallan a continuación. Por ejemplo, es posible que necesite aclarar una ambigüedad, proporcionar ayuda, retroceder o mostrar el progreso. Al usar **FormFlow** en el SDK de Bot Builder para .NET, puede simplificar enormemente el proceso para administrar una conversación guiada como esta. 

FormFlow genera automáticamente los diálogos necesarios para gestionar una conversación guiada, según las pautas que especifique. Aunque al usar FormFlow sacrificará parte de la flexibilidad que de otro modo obtendría creando y administrando diálogos por su cuenta, la opción de diseñar una conversación guiada con FormFlow puede reducir significativamente el tiempo que lleva desarrollar el bot. Además, puede construir su bot utilizando una combinación de cuadros de diálogo que haya generado FormFlow y otros tipos de cuadros de diálogo. Por ejemplo, un diálogo FormFlow puede guiar al usuario a través del proceso para completar un formulario, mientras que [LuisDialog][LuisDialog] puede evaluar la entrada del usuario para determinar la intención.

En este artículo se describe cómo crear un bot que utilice las funciones básicas de FormFlow para recopilar información de un usuario.

## <a id="forms-and-fields"></a>Formularios y campos

Para crear un bot con FormFlow, debe especificar la información que el robot debe recopilar del usuario. Por ejemplo, si el objetivo del bot es obtener el pedido de un sándwich para el usuario, entonces debe definir un formulario que contenga campos adecuados para recopilar los datos que el bot necesita para realizar el pedido. Puede definir el formulario creando una clase de C# que contenga una o más propiedades públicas para representar los datos que el bot recopilará del usuario. Cada propiedad debe ser uno de estos tipos de datos:

- Integral (sbyte, byte, short, ushort, int, uint, long, ulong)
- Punto flotante (float, double)
- string
- Datetime
- Enumeration
- Lista de enumeraciones

Es necesario que cualquiera de los tipos de datos admitan valores NULL, ya que los puede usar para especificar que el campo no tiene un valor. Si un campo de formulario se basa en una propiedad de enumeración que no admite valores NULL, el valor **0** de la enumeración representa **null** (es decir, indica que el campo no tiene un valor), y sus valores de enumeración deben comenzar por **1**. FormFlow ignora el resto de métodos y tipos de propiedad.

En cuanto a los objetos complejos, debe crear un formulario para la clase C# de nivel superior, y otro para el objeto complejo. Puede crear los formularios mediante la típica semántica de [diálogo](bot-builder-dotnet-dialogs.md). También es posible definir un formulario implementando directamente [Advanced.IField][iField] o usando [Advanced.Field][field] y rellenando los diccionarios que se encuentran en ese elemento. 

> [!NOTE]
> Puede definir un formulario con una clase de C# o un esquema JSON. En este artículo se describe cómo definir un formulario mediante una clase de C#. Para obtener más información sobre cómo usar el esquema JSON, consulte [Define a form using JSON schema](bot-builder-dotnet-formflow-json-schema.md) (Definir un formulario con el esquema JSON).

## <a name="simple-sandwich-bot"></a>Bot simple de sándwich

En este ejemplo le mostramos un bot simple de sándwich diseñado para obtener el pedido que hizo un usuario para recibir un sándwich. 

### <a id="create-class"></a> Crear el formulario

La clase `SandwichOrder` define la forma, y las enumeraciones definen las opciones con los ingredientes para hacer el sándwich. La clase también incluye el método estático `BuildForm` que usa [FormBuilder][formBuilder] para crear el formulario y definir un mensaje de bienvenida simple. 

Para usar FormFlow, primero debe importar el espacio de nombres `Microsoft.Bot.Builder.FormFlow`.

[!code-csharp[Define form](../includes/code/dotnet-formflow.cs#defineForm)]

### <a name="connect-the-form-to-the-framework"></a>Conectar el formulario al marco 

Para conectar el formulario al marco, debe agregarlo al controlador. En este ejemplo, el método `Conversation.SendAsync` llama al método estático `MakeRootDialog`, que a su vez llama al método `FormDialog.FromForm` para crear el formulario `SandwichOrder`. 

[!code-csharp[Connect form to framework](../includes/code/dotnet-formflow.cs#connectToFramework)]

### <a name="see-it-in-action"></a>Véalo en acción

Solo tiene que definir el formulario con una clase C# y conectarlo al marco, para habilitar FormFlow para administrar automáticamente la conversación entre el bot y el usuario. Las interacciones de ejemplo que se muestran a continuación demuestran las capacidades de un bot que se crea mediante el uso de las funciones básicas de FormFlow. En cada interacción, un símbolo **>** indica el punto en el que el usuario escribe una respuesta. 

#### <a name="display-the-first-prompt"></a>Mostrar el primer aviso 

Este formulario rellena la propiedad `SandwichOrder.Sandwich`. El formulario genera automáticamente el mensaje "Seleccione un sándwich", donde la palabra "sándwich" de la solicitud deriva del nombre de propiedad `Sandwich`. La enumeración `SandwichOptions` define las opciones que se presentan al usuario, y cada valor de enumeración se divide automáticamente en palabras en función de los cambios en mayúsculas, minúsculas y guiones bajos.

```console
Please select a sandwich
1. BLT
2. Black Forest Ham
3. Buffalo Chicken
4. Chicken And Bacon Ranch Melt
5. Cold Cut Combo
6. Meatball Marinara
7. Oven Roasted Chicken
8. Roast Beef
9. Rotisserie Style Chicken
10. Spicy Italian
11. Steak And Cheese
12. Sweet Onion Teriyaki
13. Tuna
14. Turkey Breast
15. Veggie
>
```

#### <a name="provide-guidance"></a>Proporcionar una guía

El usuario puede escribir "ayuda" en cualquier punto de la conversación para obtener ayuda acerca de cómo completar el formulario. Por ejemplo, si el usuario escribe "ayuda" en la solicitud correspondiente al sándwich, el bot responderá con esta guía. 

```console
> help
* You are filling in the sandwich field. Possible responses:
* You can enter a number 1-15 or words from the descriptions. (BLT, Black Forest Ham, Buffalo Chicken, Chicken And Bacon Ranch Melt, Cold Cut Combo, Meatball Marinara, Oven Roasted Chicken, Roast Beef, Rotisserie Style Chicken, Spicy Italian, Steak And Cheese, Sweet Onion Teriyaki, Tuna, Turkey Breast, and Veggie)
* Back: Go back to the previous question.
* Help: Show the kinds of responses you can enter.
* Quit: Quit the form without completing it.
* Reset: Start over filling in the form. (With defaults from your previous entries.)
* Status: Show your progress in filling in the form so far.
* You can switch to another field by entering its name. (Sandwich, Length, Bread, Cheese, Toppings, and Sauce).
```

#### <a name="advance-to-the-next-prompt"></a>Avanzar a la siguiente solicitud

Si el usuario escribe "2" en respuesta la solicitud de sándwich inicial, el bot muestra una solicitud para ir a la siguiente propiedad que se define mediante el formulario: `SandwichOrder.Length`.

```console
Please select a sandwich
 1. BLT
 2. Black Forest Ham
 3. Buffalo Chicken
 4. Chicken And Bacon Ranch Melt
 5. Cold Cut Combo
 6. Meatball Marinara
 7. Oven Roasted Chicken
 8. Roast Beef
 9. Rotisserie Style Chicken
 10. Spicy Italian
 11. Steak And Cheese
 12. Sweet Onion Teriyaki
 13. Tuna
 14. Turkey Breast
 15. Veggie
> 2
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="return-to-the-previous-prompt"></a>Volver a la solicitud anterior 

Si el usuario escribe "volver" en este punto de la conversación, el bot devolverá la solicitud anterior. El mensaje mostrará la elección actual del usuario ("Jamón de la Selva Negra"); el usuario puede cambiar esa selección escribiendo un número diferente o confirmar esa selección con la letra "c".

```console
> back
Please select a sandwich(current choice: Black Forest Ham)
 1. BLT
 2. Black Forest Ham
 3. Buffalo Chicken
 4. Chicken And Bacon Ranch Melt
 5. Cold Cut Combo
 6. Meatball Marinara
 7. Oven Roasted Chicken
 8. Roast Beef
 9. Rotisserie Style Chicken
 10. Spicy Italian
 11. Steak And Cheese
 12. Sweet Onion Teriyaki
 13. Tuna
 14. Turkey Breast
 15. Veggie
> c
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="clarify-user-input"></a>Aclarar la entrada del usuario

Si el usuario responde con texto (en lugar de con un número) para indicar una opción, el bot automáticamente solicitará una aclaración si la entrada del usuario coincide con más de una opción. 

```console
Please select a bread
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> nine grain
By "nine grain" bread did you mean (1. Nine Grain Honey Oat, 2. Nine Grain Wheat)
> 1
```

Si la entrada del usuario no coincide con ninguna de las opciones válidas, el bot automáticamente solicitará al usuario una aclaración.

```console
Please select a cheese (1. American, 2. Monterey Cheddar, 3. Pepperjack)
> amercan
"amercan" is not a cheese option.
> american smoked
For cheese I understood American. "smoked" is not an option.
```

Si la entrada del usuario especifica múltiples opciones para una propiedad y el bot no entiende ninguna de las opciones especificadas, automáticamente solicitará al usuario una aclaración.

```console
Please select one or more toppings
 1. Banana Peppers
 2. Cucumbers
 3. Green Bell Peppers
 4. Jalapenos
 5. Lettuce
 6. Olives
 7. Pickles
 8. Red Onion
 9. Spinach
 10. Tomatoes
> peppers, lettuce and tomato
By "peppers" toppings did you mean (1. Green Bell Peppers, 2. Banana Peppers)
> 1
```

#### <a name="show-current-status"></a>Mostrar estado actual

Si el usuario escribe "estado" en cualquier punto del pedido, la respuesta del bot indicará qué valores se han especificado ya y qué valores quedan por especificar. 

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> status
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Unspecified  
```

#### <a name="confirm-selections"></a>Confirmar selecciones

Cuando el usuario complete el formulario, el bot le pedirá que confirme lo que ha seleccionado.

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> 1
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
>
```

Si el usuario responde escribiendo "no", el bot le permite actualizar cualquiera de las selecciones anteriores. Si el usuario responde "sí", el formulario se habrá completado y el control se devuelve al cuadro de diálogo que realiza la llamada. 

```console
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> no
What do you want to change?
 1. Sandwich(Black Forest Ham)
 2. Length(Six Inch)
 3. Bread(Nine Grain Honey Oat)
 4. Cheese(American)
 5. Toppings(Lettuce, Tomatoes, and Green Bell Peppers)
 6. Sauce(Honey Mustard)
> 2
Please select a length (current choice: Six Inch) (1. Six Inch, 2. Foot Long)
> 2
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Foot Long
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> y
```

## <a name="handling-quit-and-exceptions"></a>Administrar la salida del proceso y las excepciones

Si el usuario escribe "salir" en el formulario o se produce una excepción en algún momento de la conversación, el bot deberá saber el paso en el que ocurrió el evento, el estado del formulario cuando ocurrió ese evento y qué pasos del formulario se completaron correctamente antes del evento. El formulario devuelve esta información a través de la clase `FormCanceledException<T>`. 

En este ejemplo de código se muestra cómo detectar la excepción y mostrar un mensaje de acuerdo con el evento que ocurrió. 

[!code-csharp[Handle exception or quit](../includes/code/dotnet-formflow.cs#handleExceptionOrQuit)]

## <a name="summary"></a>Resumen

En este artículo se ha descrito cómo usar las características básicas de FormFlow para crear un bot que pueda:

- Crear y administrar automáticamente la conversación.
- Proporcionar una guía y ayuda claras.
- Comprender los números y las entradas de texto.
- Proporcionar comentarios al usuario sobre lo que entiende y lo que no. 
- Hacer preguntas aclaratorias cuando sea necesario. 
- Permitir al usuario navegar entre pasos.

Aunque la funcionalidad básica de FormFlow es suficiente en algunos casos, le recomendamos que considere los beneficios potenciales que conlleva incorporar algunas de las características más avanzadas de FormFlow en el bot. Para obtener más información, consulte [Advanced features of FormFlow](bot-builder-dotnet-formflow-advanced.md) (Funciones avanzadas de FormFlow) y [Customize a form using FormBuilder](bot-builder-dotnet-formflow-formbuilder.md) (Personalizar un formulario utilizando FormBuilder).

## <a name="sample-code"></a>Código de ejemplo

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="next-steps"></a>Pasos siguientes

FormFlow simplifica el desarrollo del cuadro de diálogo. Las características avanzadas de FormFlow le permiten personalizar cómo se comporta un objeto de FormFlow.

> [!div class="nextstepaction"]
> [Características avanzadas de FormFlow](bot-builder-dotnet-formflow-advanced.md)

## <a name="additional-resources"></a>Recursos adicionales

- [Personalización de un formulario mediante FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Localización del contenido del formulario](bot-builder-dotnet-formflow-localize.md)
- [Definición de un formulario mediante esquemas JSON](bot-builder-dotnet-formflow-json-schema.md)
- [Personalización de la experiencia de usuario con lenguaje de patrones](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Builder SDK para .NET</a>

[LuisDialog]: /dotnet/api/microsoft.bot.builder.dialogs.luisdialog-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1
