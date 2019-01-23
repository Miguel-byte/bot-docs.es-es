---
title: Características avanzadas de FormFlow | Microsoft Docs
description: Obtenga información sobre cómo personalizar la experiencia del usuario mediante FormFlow y Bot Framework SDK para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d04e13babef847a44438e1a748990d7405478fa2
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225960"
---
# <a name="advanced-features-of-formflow"></a>Características avanzadas de FormFlow

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[Basic features of FormFlow](bot-builder-dotnet-formflow.md) (Características básicas de FromFlow) describe una implementación básica de FormFlow que ofrece una experiencia de usuario bastante genérica. Para ofrecer una experiencia de usuario más personalizada mediante FormFlow, puede especificar el estado inicial del formulario, agregar lógica de negocios para administrar las interdependencias entre los campos y procesar los datos del usuario, y usar atributos para personalizar mensajes, reemplazar plantillas, designar campos opcionales, establecer coincidencias con la entrada del usuario y validar dicha entrada. 

## <a name="specify-initial-form-state-and-entities"></a>Especificación de las entidades y del estado inicial del formulario

Cuando inicie [FormDialog][formDialog], también puede pasar una instancia de su estado de forma opcional. Si pasa una instancia del estado, FormFlow omitirá de forma predeterminada los pasos para todos los campos que ya contienen valores. No se pedirá al usuario que rellene esos campos. Para forzar que el formulario solicite al usuario todos los campos (incluso los que ya contienen valores en el estado inicial), pase [FormOptions.PromptFieldsWithValues][promptFieldsWithValues] cuando inicie `FormDialog`. Si un campo contiene un valor inicial, el mensaje lo usará como valor predeterminado.

También puede pasar entidades de [LUIS](https://luis.ai/) para enlazarlas con el estado. Si `EntityRecommendation.Type` es una ruta de acceso a un campo de la clase de C#, `EntityRecommendation.Entity` se pasará por el reconocedor para enlazarla con el campo. FormFlow omitirá los pasos para todos los campos que estén enlazados a una entidad; no se pedirá al usuario que rellene esos campos. 

## <a name="add-business-logic"></a>Adición de lógica de negocios 

Puede especificar la lógica de negocios dentro de una función de validación para manipular las interdependencias entre los campos de formulario o aplicar una lógica específica durante el proceso de obtener o establecer el valor de un campo. Una función de validación le permite manipular estados y devolver un objeto [ValidateResult][validateResult] que puede contener: 

- una cadena de comentarios que describe el motivo por el que un valor no es válido
- un valor transformado
- un conjunto de opciones para aclarar un valor

Este ejemplo de código muestra una función de validación para el campo `Toppings`. Si la entrada para el campo contiene el valor de enumeración `ToppingOptions.Everything`, la función garantiza que el valor del campo `Toppings` contiene la lista completa de los ingredientes.

[!code-csharp[Validation function](../includes/code/dotnet-formflow-advanced.cs#validationFunction)]

Además de la función de validación, puede agregar el atributo [Term](#match-user-input-using-the-terms-attribute) para que coincida con las expresiones de usuario como “todo” o “no”.

[!code-csharp[Terms for Toppings](../includes/code/dotnet-formflow-advanced.cs#toppingsTerms)]

Con la función de validación mostrada anteriormente, este fragmento de código muestra la interacción entre el bot y el usuario cuando el usuario solicita “todo excepto Jalapeños”. 

```console
Please select one or more toppings (current choice: No Preference)
 1. Everything
 2. Avocado
 3. Banana Peppers
 4. Cucumbers
 5. Green Bell Peppers
 6. Jalapenos
 7. Lettuce
 8. Olives
 9. Pickles
 10. Red Onion
 11. Spinach
 12. Tomatoes
> everything but jalapenos
For sandwich toppings you have selected Avocado, Banana Peppers, Cucumbers, Green Bell Peppers, Lettuce, Olives, Pickles, Red Onion, Spinach, and Tomatoes.
```

## <a name="formflow-attributes"></a>Atributos de FormFlow

Puede agregar estos atributos de C# a la clase para personalizar el comportamiento de un diálogo de FormFlow.

| Atributo | Propósito |
|----|----| 
| [Describe][describeAttribute] | Modifica cómo se muestra un campo o un valor en una plantilla o tarjeta. |
| [Numeric][numericAttribute] | Restringe los valores aceptados de un campo numérico. |
| [Optional][optionalAttribute] | Marca un campo como opcional. |
| [Pattern][patternAttribute] | Define una expresión regular para validar un campo de cadena. |
| [Prompt][promptAttribute] | Define el mensaje para un campo. |
| [Template][templateAttribute] | Define la plantilla utilizada para generar mensajes o valores en los mensajes. |
| [Terms][termsAttribute] | Define los términos de entrada que coinciden con un campo o valor. |

## <a name="customize-prompts-using-the-prompt-attribute"></a>Personalización de mensajes con el atributo Prompt

Los mensajes predeterminados se generan automáticamente para cada campo del formulario, pero puede especificar un mensaje personalizado para cualquier campo con el atributo `Prompt`. Por ejemplo, si el mensaje predeterminado para el campo `SandwichOrder.Sandwich` es “Seleccione un sándwich”, puede agregar el atributo `Prompt` para especificar un mensaje personalizado para el campo.

[!code-csharp[Prompt attribute](../includes/code/dotnet-formflow-advanced.cs#promptAttribute)]

Este ejemplo utiliza [lenguaje de patrones](bot-builder-dotnet-formflow-pattern-language.md) para rellenar dinámicamente el mensaje con los datos del formulario durante el tiempo de ejecución: `{&}` se reemplaza por la descripción del campo y `{||}` se reemplaza por la lista de opciones en la enumeración. 

> [!NOTE]
> De forma predeterminada, la descripción de un campo se genera a partir del nombre del campo. Para especificar una descripción personalizada para un campo, agregue el atributo `Describe`.

En este fragmento de código se muestra el mensaje personalizado que se ha especificado en el ejemplo anterior.

```console
What kind of sandwich would you like?
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

Un atributo `Prompt` también puede especificar los parámetros que afectan a cómo el formulario muestra el mensaje. Por ejemplo, el parámetro `ChoiceFormat` determina cómo el formulario representa la lista de opciones.

[!code-csharp[Prompt attribute ChoiceFormat parameter](../includes/code/dotnet-formflow-advanced.cs#promptChoice)]

En este ejemplo, el valor del parámetro `ChoiceFormat` indica que se deben mostrar las opciones como una lista con viñetas (en lugar de una lista numerada).

```console
What kind of sandwich would you like?
- BLT
- Black Forest Ham
- Buffalo Chicken
- Chicken And Bacon Ranch Melt
- Cold Cut Combo
- Meatball Marinara
- Oven Roasted Chicken
- Roast Beef
- Rotisserie Style Chicken
- Spicy Italian
- Steak And Cheese
- Sweet Onion Teriyaki
- Tuna
- Turkey Breast
- Veggie
>
```

## <a name="customize-prompts-using-the-template-attribute"></a>Personalización de mensajes con el atributo Template

Mientras el atributo `Prompt` le permite personalizar el mensaje para un solo campo, el atributo `Template` le permite reemplazar las plantillas predeterminadas que usa FormFlow para generar automáticamente los mensajes. Este ejemplo de código utiliza el atributo `Template` para redefinir la forma en que el formulario manipula todos los campos de enumeración. El atributo indica que el usuario puede seleccionar solo un elemento, establece el texto del mensaje mediante el [lenguaje de patrones](bot-builder-dotnet-formflow-pattern-language.md) y especifica que el formulario debe mostrar solo un elemento por línea. 

[!code-csharp[Template attribute](../includes/code/dotnet-formflow-advanced.cs#templateAttribute)]

En este fragmento de código se muestran los mensajes resultantes para los campos `Bread` y `Cheese`.

```console
What kind of bread would you like on your sandwich?
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> 

What kind of cheese would you like on your sandwich? 
 1. American
 2. Monterey Cheddar
 3. Pepperjack
> 
```

Si usa el atributo `Template` para reemplazar las plantillas predeterminadas que usa FormFlow en la generación de mensajes, puede interponer alguna variación en los mensajes y los avisos generados por el formulario. Para ello, puede definir varias cadenas de texto mediante el [lenguaje de patrones](bot-builder-dotnet-formflow-pattern-language.md) y el formulario elegirá aleatoriamente entre las opciones disponibles cada vez que necesite mostrar un aviso o un mensaje.

Este ejemplo de código redefine la plantilla [TemplateUsage.NotUnderstood][notUnderstood] para especificar dos variaciones del mensaje. Cuando el bot necesite comunicar que no comprende la entrada del usuario, determinará el contenido del mensaje seleccionando aleatoriamente una de las dos cadenas de texto. 

[!code-csharp[Template variations of message](../includes/code/dotnet-formflow-advanced.cs#templateMessages)]

En este fragmento de código se muestra un ejemplo de la interacción resultante entre el usuario y el bot. 

```console
What size of sandwich do you want? (1. Six Inch, 2. Foot Long)
> two feet
I do not understand "two feet".
> two feet
Try again, I don't get "two feet"
> 
```

## <a name="designate-a-field-as-optional-using-the-optional-attribute"></a>Designación de un campo como opcional con el atributo Optional

Para designar un campo como opcional, use el atributo `Optional`. Este ejemplo de código especifica que el campo `Cheese` es opcional.

[!code-csharp[Optional attribute](../includes/code/dotnet-formflow-advanced.cs#optionalAttribute)]

Si un campo es opcional y no se ha especificado ningún valor, se mostrará la opción actual como “Sin preferencias”.

```console
What kind of cheese would you like on your sandwich? (current choice: No Preference)
  1. American
  2. Monterey Cheddar
  3. Pepperjack
 >
```

Si un campo es opcional y el usuario ha especificado un valor, se mostrará “Sin preferencias” como la última opción en la lista.

```console
What kind of cheese would you like on your sandwich? (current choice: American)
 1. American
 2. Monterey Cheddar
 3. Pepperjack
 4. No Preference
>
```

## <a name="match-user-input-using-the-terms-attribute"></a>Coincidencias con la entrada del usuario mediante el atributo Terms

Cuando un usuario envía un mensaje a un bot creado mediante FormFlow, el bot intenta identificar el significado de la entrada del usuario estableciendo una coincidencia de la entrada con una lista de términos. De forma predeterminada, la lista de términos se genera al realizar los siguientes pasos para el campo o valor: 

1. Insertar un salto en cambios de mayúsculas y guiones bajos (_).
2. Generar cada <a href="https://en.wikipedia.org/wiki/N-gram" target="_blank">n-grama</a> hasta una longitud máxima.
3. Agregar “s?” al final de cada palabra (para admitir plurales). 

Por ejemplo, el valor “PizzaDeCarneAngusConAjo” generaría estos términos: 

- 'pizzas?'
- 'carnes?'
- 'angus?'
- 'ajos?'
- 'carnes? angus?'
- 'pizzas? ajos?'
- 'pizza de carne angus con ajo'

Para invalidar este comportamiento predeterminado y definir la lista de términos que se usan para hacer coincidencias con la entrada del usuario en un campo o en el valor de un campo, use el atributo `Terms`. Por ejemplo, puede usar el atributo `Terms` (con una expresión regular) para considerar que los usuarios suelen escribir mal la palabra “rotisserie” en inglés. 

[!code-csharp[Terms attribute](../includes/code/dotnet-formflow-advanced.cs#termsAttribute)]

Mediante el uso del atributo `Terms`, aumentarán las posibilidades de que exista una coincidencia entre la entrada del usuario y una de las opciones válidas. El parámetro `Terms.MaxPhrase` en este ejemplo hace que `Language.GenerateTerms` genere más variaciones de términos. 

En este fragmento de código se muestra la interacción resultante entre el usuario y el bot cuando el usuario ha escrito incorrectamente “rotisserie”.

```console
What kind of sandwich would you like?
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
> rotissary checkin
For sandwich I understood Rotisserie Style Chicken. "checkin" is not an option.
```

## <a name="validate-user-input-using-the-numeric-attribute-or-pattern-attribute"></a>Validación de la entrada del usuario mediante los atributos Numeric o Pattern

Para restringir el intervalo de valores permitidos en un campo numérico, use el atributo `Numeric`. Este ejemplo de código utiliza el atributo `Numeric` para especificar que la entrada para el campo `Rating` debe ser un número entre 1 y 5. 

[!code-csharp[Numeric attribute](../includes/code/dotnet-formflow-advanced.cs#numericAttribute)]

Para especificar el formato que debe seguir el valor de un campo específico, use el atributo `Pattern`. Este ejemplo de código usa el atributo `Pattern` para especificar el formato que debe seguir el valor del campo `PhoneNumber`.

[!code-csharp[Pattern attribute](../includes/code/dotnet-formflow-advanced.cs#patternAttribute)]

## <a name="summary"></a>Resumen

En este artículo se describe cómo ofrecer una experiencia de usuario personalizada con FormFlow al especificar el estado inicial del formulario, agregar lógica de negocios para administrar las interdependencias entre los campos y procesar los datos del usuario, y usar atributos para personalizar mensajes, reemplazar plantillas, designar campos opcionales, establecer coincidencias con la entrada del usuario y validar dicha entrada. Para obtener más información sobre otras formas de personalizar la experiencia del usuario con FormFlow, consulte [Customize a form using FormBuilder](bot-builder-dotnet-formflow-formbuilder.md) (Personalización de un formulario mediante FormBuilder).

## <a name="sample-code"></a>Código de ejemplo

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Recursos adicionales

- [Características básicas de FormFlow](bot-builder-dotnet-formflow.md)
- [Customize a form using FormBuilder](bot-builder-dotnet-formflow-formbuilder.md) (Personalización de un formulario mediante FormBuilder)
- [Localización del contenido del formulario](bot-builder-dotnet-formflow-localize.md)
- [Definición de un formulario mediante esquemas JSON](bot-builder-dotnet-formflow-json-schema.md)
- [Personalización de la experiencia de usuario con lenguaje de patrones](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>

[formDialog]: /dotnet/api/microsoft.bot.builder.formflow.formdialog

[promptFieldsWithValues]: /dotnet/api/microsoft.bot.builder.formflow.formoptions.promptfieldswithvalues

[validateResult]: /dotnet/api/microsoft.bot.builder.formflow.validateresult

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[optionalAttribute]: /dotnet/api/microsoft.bot.builder.formflow.optionalattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[termsAttribute]: /dotnet/api/microsoft.bot.builder.formflow.termsattribute

[notUnderstood]: /dotnet/api/microsoft.bot.builder.formflow.templateusage.notunderstood
