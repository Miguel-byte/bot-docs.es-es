---
title: Personalización de la experiencia de usuario con lenguaje de patrones | Microsoft Docs
description: Aprenda a personalizar los avisos FormFlow y a invalidar las plantillas FormFlow mediante un lenguaje de patrones con el SDK de Bot Builder para .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bc0a2819f3adea63b53e464808f3bbaf5b93814a
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998052"
---
# <a name="customize-user-experience-with-pattern-language"></a>Personalización de la experiencia de usuario con lenguaje de patrones

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Para personalizar un aviso o invalidar una plantilla predeterminada, puede usar un lenguaje de patrones para especificar el contenido o el formato del aviso. 

## <a name="prompts-and-templates"></a>Avisos y plantillas

Un [aviso][promptAttribute] define el mensaje que se envía al usuario para solicitar información o para pedir confirmación. Puede personalizar un aviso con el [atributo Prompt](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-prompt-attribute) o de manera implícita por medio de [IFormBuilder<T>.Field][field]. 

Los formularios usan plantillas para construir automáticamente avisos y otras cosas como la ayuda. Puede invalidar la plantilla predeterminada de una clase o un campo mediante el [atributo Template](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-template-attribute). 

> [!TIP]
> La clase [FormConfiguration.Templates][formConfiguration] define un conjunto de plantillas integradas que ofrecen buenos ejemplos de cómo usar lenguaje de patrones.

## <a name="elements-of-pattern-language"></a>Elementos de lenguaje de patrones

El lenguaje de patrones usa llaves (`{}`) para identificar los elementos que se reemplazarán en tiempo de ejecución por valores reales. En esta tabla se enumeran los elementos del lenguaje de patrones.

| Elemento | DESCRIPCIÓN |
|----|----|
| `{<format>}` | Muestra el valor del campo actual (el campo al que se aplica el atributo). |
| `{&}` | Muestra la descripción del campo actual (a menos que se especifique lo contrario, este es el nombre del campo). |
| `{<field><format>}` | Muestra el valor del campo designado. | 
| `{&<field>}` | Muestra la descripción del campo designado. |
| <code>{&#124;&#124;}</code> | Muestra las opciones actuales, que podrían ser el valor actual de un campo, "sin preferencias" o los valores de una enumeración. |
| `{[{<field><format>} ...]}` | Muestra una lista de valores de los campos designados mediante [Separator][separator] y [LastSeparator][lastSeparator] para separar los valores individuales de la lista. |
| `{*}` | Muestra una línea para cada campo activo; cada línea contiene la descripción del campo y el valor actual. | 
| `{*filled}` | Muestra una línea para cada campo activo que contiene un valor actual; cada línea contiene la descripción del campo y el valor actual. |
| `{<nth><format>}` | Un especificador normal de formato en C# que se aplica al enésimo argumento de una plantilla. Para obtener la lista de argumentos disponibles, consulte [TemplateUsage][templateUsage]. |
| `{?<textOrPatternElement>...}` | Sustitución condicional. Si todo lo referente a elementos de patrón tiene valores, estos se sustituyen y se usa la expresión entera. |

Para los elementos mencionados anteriormente:

- El marcador de posición `<field>` es el nombre de un campo en la clase de formulario. Por ejemplo, si la clase contiene un campo con el nombre `Size`, puede especificar `{Size}` como el elemento de patrón. 

- Un botón de puntos suspensivos (`"..."`) dentro de un elemento de patrón indica que el elemento puede contener varios valores.

- El marcador de posición `<format>` es un especificador de formato en C#. Por ejemplo, si la clase contiene un campo con el nombre `Rating` y de tipo `double`, puede especificar `{Rating:F2}` como elemento de patrón para mostrar dos dígitos de precisión.

- El marcador de posición `<nth>` hace referencia al enésimo argumento de una plantilla.

### <a name="pattern-language-within-a-prompt-attribute"></a>Lenguaje de patrones dentro de un atributo Prompt

En este ejemplo se usa el elemento `{&}` para mostrar la descripción del campo `Sandwich` y el elemento `{||}` para mostrar la lista de opciones para ese campo.

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns1)]

Este es el aviso resultante:

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

## <a name="formatting-parameters"></a>Parámetros de formato

Los avisos y las plantillas admiten estos parámetros de formato.

| Uso | DESCRIPCIÓN |
|----|----|
| `AllowDefault` | Se aplica a los elementos de patrón <code>{&#124;&#124;}</code>. Determina si el formulario debe mostrar el valor actual del campo como una posible opción. Si es `true`, se muestra el valor actual como posible valor. El valor predeterminado es `true`. |
| `ChoiceCase` | Se aplica a los elementos de patrón <code>{&#124;&#124;}</code>. Determina si el texto de cada opción se normaliza (por ejemplo, si la primera letra de cada palabra se pone en mayúscula). El valor predeterminado es `CaseNormalization.None`. Para conocer los valores posibles, consulte [CaseNormalization][caseNormalization]. |
| `ChoiceFormat` | Se aplica a los elementos de patrón <code>{&#124;&#124;}</code>. Determina si se debe mostrar una lista de opciones como una lista numerada o una lista con viñetas. Para obtener una lista numerada, establezca `ChoiceFormat` en `{0}` (valor predeterminado). Para obtener una lista con viñetas, establezca `ChoiceFormat` en `{1}`. |
| `ChoiceLastSeparator` | Se aplica a los elementos de patrón <code>{&#124;&#124;}</code>. Determina si una lista alineada de opciones incluye un separador delante de la última opción. |
| `ChoiceParens` | Se aplica a los elementos de patrón <code>{&#124;&#124;}</code>. Determina si una lista alineada de opciones se muestra entre paréntesis. Si es `true`, la lista de opciones se muestra entre paréntesis. El valor predeterminado es `true`. |
| `ChoiceSeparator` | Se aplica a los elementos de patrón <code>{&#124;&#124;}</code>. Determina si una lista alineada de opciones incluye un separador delate de cada opción excepto la última. | 
| `ChoiceStyle` | Se aplica a los elementos de patrón <code>{&#124;&#124;}</code>. Determina si la lista de opciones se muestra alineada o por línea. El valor predeterminado es `ChoiceStyleOptions.Auto` que determina en tiempo de ejecución si la opción se debe mostrar alineada o en una lista. Para conocer los valores posibles, consulte [ChoiceStyleOptions][choiceStyleOptions]. |
| `Feedback` | Se aplica solo a los avisos. Determina si el formulario repite la elección del usuario para indicar que comprende la selección. El valor predeterminado es `FeedbackOptions.Auto` que repite la entrada del usuario solo si alguna de sus partes no se entiende. Para conocer los valores posibles, consulte [FeedbackOptions][feedbackOptions]. |
| `FieldCase` | Determina si el texto de la descripción del campo se normaliza (por ejemplo, si la primera letra de cada palabra se escribe en mayúsculas). El valor predeterminado es `CaseNormalization.Lower` que pasa la descripción a minúsculas. Para conocer los valores posibles, consulte [CaseNormalization][caseNormalization]. |
| `LastSeparator` | Se aplica a los elementos de patrón `{[]}`. Determina si una matriz de elementos incluye un separador delante de la última opción. |
| `Separator` | Se aplica a los elementos de patrón `{[]}`. Determina si una matriz de elementos incluye un separador delante de cada elemento de la matriz excepto el último. |
| `ValueCase` | Determina si el texto del valor del campo se normaliza (por ejemplo, si la primera letra de cada palabra se escribe en mayúsculas). El valor predeterminado es `CaseNormalization.InitialUpper` que convierte la primera letra de cada palabra a mayúsculas. Para conocer los valores posibles, consulte [CaseNormalization][caseNormalization]. |

### <a name="prompt-attribute-with-formatting-parameter"></a>Atributo Prompt con parámetro de formato 

En este ejemplo se usa el parámetro `ChoiceFormat` para especificar que la lista de opciones debe mostrarse como una lista con viñetas.

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns2)]

Este es el aviso resultante:

```console
What kind of sandwich would you like?
* BLT
* Black Forest Ham
* Buffalo Chicken
* Chicken And Bacon Ranch Melt
* Cold Cut Combo
* Meatball Marinara
* Oven Roasted Chicken
* Roast Beef
* Rotisserie Style Chicken
* Spicy Italian
* Steak And Cheese
* Sweet Onion Teriyaki
* Tuna
* Turkey Breast
* Veggie
>
```

## <a name="sample-code"></a>Código de ejemplo

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Recursos adicionales

- [Características básicas de FormFlow](bot-builder-dotnet-formflow.md)
- [Características avanzadas de FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Personalización de un formulario mediante FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Localización del contenido del formulario](bot-builder-dotnet-formflow-localize.md)
- [Definición de un formulario mediante esquemas JSON](bot-builder-dotnet-formflow-json-schema.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Builder SDK para .NET</a>

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[field]: /dotnet/api/microsoft.bot.builder.formflow.iformbuilder-1.field

[formConfiguration]: /dotnet/api/microsoft.bot.builder.formflow.formconfiguration

[separator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.separator

[lastSeparator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.lastseparator

[templateUsage]: /dotnet/api/microsoft.bot.builder.formflow.templateusage

[caseNormalization]: /dotnet/api/microsoft.bot.builder.formflow.casenormalization

[choiceStyleOptions]: /dotnet/api/microsoft.bot.builder.formflow.choicestyleoptions

[feedbackOptions]: /dotnet/api/microsoft.bot.builder.formflow.feedbackoptions
