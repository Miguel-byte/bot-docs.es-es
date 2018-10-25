---
title: Definición de un formulario mediante esquemas JSON y FormFlow | Microsoft Docs
description: Aprenda a definir un formulario mediante esquemas JSON y FormFlow con Bot Builder SDK para. NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8bcc957dbe2d69790cdfa7c2d7c377ed28b5fa12
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000332"
---
# <a name="define-a-form-using-json-schema"></a>Definición de un formulario mediante esquemas JSON

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Si usa una [clase de C#](bot-builder-dotnet-formflow.md#create-class) para definir el formulario cuando crea un bot con FormFlow, el formulario se deriva de la definición estática de su tipo en C#. En su lugar, puede definir el formulario mediante <a href="http://json-schema.org/documentation.html" target="_blank">esquemas JSON</a>. Un formulario que se define mediante esquemas JSON se basa por completo en datos. Puede cambiar el formulario (y por tanto, el comportamiento del bot) solo con actualizar el esquema. 

El esquema JSON describe los campos dentro de <a href="http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JObject.htm" target="_blank">JObject</a> e incluye anotaciones que controlan los avisos, las plantillas y los términos. Para usar el esquema JSON con FormFlow, primero debe agregar el paquete NuGet `Microsoft.Bot.Builder.FormFlow.Json` al proyecto e importar el espacio de nombres `Microsoft.Bot.Builder.FormFlow.Json`.

## <a name="standard-keywords"></a>Palabras clave estándar 

FormFlow admite estas palabras clave estándar de <a href="http://json-schema.org/documentation.html" target="_blank">esquema JSON</a>:

| Palabra clave | DESCRIPCIÓN | 
|----|----|
| Tipo | Define el tipo de datos que contiene el campo. |
| enum | Define los valores válidos para el campo. |
| minimum | Define el valor numérico mínimo permitido para el campo (como se describe en [NumericAttribute][numericAttribute]). |
| maximum | Define el valor numérico máximo permitido para el campo (como se describe en [NumericAttribute][numericAttribute]). |
| requerido | Define qué campos son obligatorios. |
| Patrón | Valida los valores de cadena (como se describe en [PatternAttribute][patternAttribute]). |

## <a name="extensions-to-json-schema"></a>Extensiones de esquemas JSON

FormFlow extiende el <a href="http://json-schema.org/documentation.html" target="_blank">esquema JSON</a> estándar para admitir varias propiedades adicionales.

### <a name="additional-properties-at-the-root-of-the-schema"></a>Propiedades adicionales en la raíz del esquema

| Propiedad | Valor |
|----|----|
| OnCompletion | Script de C# con argumentos `(IDialogContext context, JObject state)` para rellenar el formulario. |
| Referencias | Referencias que se van a incluir en los scripts. Por ejemplo, `[assemblyReference, ...]`. Las rutas de acceso deben ser absolutas o estar relacionadas con el directorio actual. De forma predeterminada, el script incluye `Microsoft.Bot.Builder.dll`. |
| Importaciones | Importaciones que se van a incluir en los scripts. Por ejemplo, `[import, ...]`. De forma predeterminada, el script incluye los espacios de nombres `Microsoft.Bot.Builder`, `Microsoft.Bot.Builder.Dialogs`, `Microsoft.Bot.Builder.FormFlow`, `Microsoft.Bot.Builder.FormFlow.Advanced`, `System.Collections.Generic` y `System.Linq`. |

### <a name="additional-properties-at-the-root-of-the-schema-or-as-peers-of-the-type-property"></a>Propiedades adicionales en la raíz del esquema o como elementos del mismo nivel de la propiedad type

| Propiedad | Valor |
|----|----|
| Plantillas | `{ TemplateUsage: { Patterns: [string, ...], <args> }, ...}` |
| Prompt | `{ Patterns:[string, ...] <args>}` |

Para especificar plantillas y avisos en el esquema JSON, use el mismo vocabulario que se definió en [TemplateAttribute][templateAttribute] y [PromptAttribute][promptAttribute]. Los nombres y valores de propiedad del esquema deben coincidir con los nombres y valores de propiedad de la enumeración de C# subyacente. Por ejemplo, este fragmento de esquema define una plantilla que reemplaza la plantilla `TemplateUsage.NotUnderstood` y especifica un `TemplateBaseAttribute.ChoiceStyle`: 

```json
"Templates":{ "NotUnderstood": { "Patterns": ["I don't get it"], "ChoiceStyle":"Auto"}}
```

### <a name="additional-properties-as-peers-of-the-type-property"></a>Propiedades adicionales como elementos del mismo nivel de la propiedad type

|   Propiedad   |          Contenido           |                                                   DESCRIPCIÓN                                                    |
|--------------|-----------------------------|------------------------------------------------------------------------------------------------------------------|
|   Datetime   |            booleano             |                                  Indica si el campo es un campo `DateTime`.                                  |
|   Describe   |      cadena u objeto       |                  Descripción de un campo como se describe en [DescribeAttribute][describeAttribute].                  |
|    Términos     |       `[string,...]`        |                  Expresiones regulares para la coincidencia con un valor de campo como se describe en TermsAttribute.                  |
|  MaxPhrase   |             int             |                  Ejecuta los términos a través de `Language.GenerateTerms(string, int)` para expandirlos.                   |
|    Valores    | `{ string: {Describe:string |                                  object, Terms:[string, ...], MaxPhrase}, ...}`                                  |
|    Active    |           script            | Script de C# con argumentos `(JObject state)->bool` para comprobar si el campo, mensaje o confirmación están activos.  |
|   Validación   |           script            |      Script de C# con argumentos `(JObject state, object value)->ValidateResult` para validar un valor de campo.      |
|    Define    |           script            |        Script de C# con argumentos `(JObject state, Field<JObject> field)` para definir un campo de forma dinámica.        |
|     Pasos siguientes     |           script            | Script de C# con argumentos `(object value, JObject state)` para determinar el siguiente paso después de rellenar un campo. |
|    Antes    |          `[confirm          |                                                  message, ...]`                                                  |
|    Después     |          `[confirm          |                                                  message, ...]`                                                  |
| Dependencias |        [string, ...]        |                           Campos de los que depende este campo, mensaje o confirmación.                           |

Use `{Confirm:script|[string, ...], ...templateArgs}` en el valor de la propiedad **Before** o de la propiedad **After** para definir una confirmación mediante un script de C# con un argumento `(JObject state)` o mediante un conjunto de patrones que se seleccionará aleatoriamente con argumentos de plantilla opcionales.

Use `{Message:script|[string, ...] ...templateArgs}` en el valor de la propiedad **Before** o de la propiedad **After** para definir un mensaje mediante un script de C# con un argumento `(JObject state)` o mediante un conjunto de patrones que se seleccionará aleatoriamente con argumentos de plantilla opcionales.

## <a name="scripts"></a>Scripts

Algunas de las propiedades que se han descrito anteriormente contienen un script como valor de propiedad. Un script puede ser cualquier fragmento de código de C# que puede encontrar normalmente en el cuerpo de un método. Puede agregar referencias mediante la propiedad **References** y la propiedad **Imports**. Entre las variables globales especiales se incluyen:

| Variable | DESCRIPCIÓN |
|----|----|
| choice | Distribución interna que ejecuta el script. |
| state | Límite de estado de formulario `JObject` para todos los scripts. |
| ifield | `IField<JObject>` para permitir el razonamiento sobre el campo actual para todos los scripts, excepto los generadores de mensajes o avisos de confirmación. |
| value | Valor de objeto que se va a validar con **Validate**. |
| campo | `Field<JObject>` para permitir la actualización dinámica de un campo en **Define**. |
| contexto | `IDialogContext` contexto para permitir la publicación de resultados en **OnCompletion**. |

Los campos que se definen mediante el esquema JSON tienen la misma capacidad para ampliar o reemplazar las definiciones mediante programación que tiene cualquier otro campo. También se pueden localizar de la misma manera.

## <a name="json-schema-example"></a>Ejemplo de esquema JSON

La manera más sencilla de definir un formulario es definir todo, incluido el código de C#, directamente en el esquema JSON. En este ejemplo se muestra el esquema JSON de un bot anotado que se usa para encargar un sándwich que se ha descrito en [Personalización de un formulario mediante FormBuilder](bot-builder-dotnet-formflow-formbuilder.md).

```json
{
  "References": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.dll" ],
  "Imports": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.Resource" ],
  "type": "object",
  "required": [
    "Sandwich",
    "Length",
    "Ingredients",
    "DeliveryAddress"
  ],
  "Templates": {
    "NotUnderstood": {
      "Patterns": [ "I do not understand \"{0}\".", "Try again, I don't get \"{0}\"." ]
    },
    "EnumSelectOne": {
      "Patterns": [ "What kind of {&} would you like on your sandwich? {||}" ],
      "ChoiceStyle": "Auto"
    }
  },
  "properties": {
    "Sandwich": {
      "Prompt": { "Patterns": [ "What kind of {&} would you like? {||}" ] },
      "Before": [ { "Message": [ "Welcome to the sandwich order bot!" ] } ],
      "Describe": { "Image": "https://placeholdit.imgix.net/~text?txtsize=16&txt=Sandwich&w=125&h=40&txttrack=0&txtclr=000&txtfont=bold" },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "BLT",
        "BlackForestHam",
        "BuffaloChicken",
        "ChickenAndBaconRanchMelt",
        "ColdCutCombo",
        "MeatballMarinara",
        "OvenRoastedChicken",
        "RoastBeef",
        "RotisserieStyleChicken",
        "SpicyItalian",
        "SteakAndCheese",
        "SweetOnionTeriyaki",
        "Tuna",
        "TurkeyBreast",
        "Veggie"
      ],
      "Values": {
        "RotisserieStyleChicken": {
          "Terms": [ "rotis\\w* style chicken" ],
          "MaxPhrase": 3
        }
      }
    },
    "Length": {
      "Prompt": {
        "Patterns": [ "What size of sandwich do you want? {||}" ]
      },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "SixInch",
        "FootLong"
      ]
    },
    "Ingredients": {
      "type": "object",
      "required": [ "Bread" ],
      "properties": {
        "Bread": {
          "type": [
            "string",
            "null"
          ],
          "Describe": {
            "Title": "Sandwich Bot",
            "SubTitle": "Bread Picker"
          },
          "enum": [
            "NineGrainWheat",
            "NineGrainHoneyOat",
            "Italian",
            "ItalianHerbsAndCheese",
            "Flatbread"
          ]
        },
        "Cheese": {
          "type": [
            "string",
            "null"
          ],
          "enum": [
            "American",
            "MontereyCheddar",
            "Pepperjack"
          ]
        },
        "Toppings": {
          "type": "array",
          "items": {
            "type": "integer",
            "enum": [
              "Everything",
              "Avocado",
              "BananaPeppers",
              "Cucumbers",
              "GreenBellPeppers",
              "Jalapenos",
              "Lettuce",
              "Olives",
              "Pickles",
              "RedOnion",
              "Spinach",
              "Tomatoes"
            ],
            "Values": {
              "Everything": { "Terms": [ "except", "but", "not", "no", "all", "everything" ] }
            }
          },
          "Validate": "var values = ((List<object>) value).OfType<string>(); var result = new ValidateResult {IsValid = true, Value = values} ; if (values != null && values.Contains(\"Everything\")) { result.Value = (from topping in new string[] {  \"Avocado\", \"BananaPeppers\", \"Cucumbers\", \"GreenBellPeppers\", \"Jalapenos\", \"Lettuce\", \"Olives\", \"Pickles\", \"RedOnion\", \"Spinach\", \"Tomatoes\"} where !values.Contains(topping) select topping).ToList();} return result;",
          "After": [ { "Message": [ "For sandwich toppings you have selected {Ingredients.Toppings}." ] } ]
        },
        "Sauces": {
          "type": [
            "array",
            "null"
          ],
          "items": {
            "type": "string",
            "enum": [
              "ChipotleSouthwest",
              "HoneyMustard",
              "LightMayonnaise",
              "RegularMayonnaise",
              "Mustard",
              "Oil",
              "Pepper",
              "Ranch",
              "SweetOnion",
              "Vinegar"
            ]
          }
        }
      }
    },
    "Specials": {
      "Templates": {
        "NoPreference": { "Patterns": [ "None" ] }
      },
      "type": [
        "string",
        "null"
      ],
      "Active": "return (string) state[\"Length\"] == \"FootLong\";",
      "Define": "field.SetType(null).AddDescription(\"cookie\", DynamicSandwich.FreeCookie).AddTerms(\"cookie\", Language.GenerateTerms(DynamicSandwich.FreeCookie, 2)).AddDescription(\"drink\", DynamicSandwich.FreeDrink).AddTerms(\"drink\", Language.GenerateTerms(DynamicSandwich.FreeDrink, 2)); return true;",
      "After": [ { "Confirm": "var cost = 0.0; switch ((string) state[\"Length\"]) { case \"SixInch\": cost = 5.0; break; case \"FootLong\": cost=6.50; break;} return new PromptAttribute($\"Total for your sandwich is {cost:C2} is that ok?\");" } ]
    },
    "DeliveryAddress": {
      "type": [
        "string",
        "null"
      ],
      "Validate": "var result = new ValidateResult{ IsValid = true, Value = value}; var address = (value as string).Trim(); if (address.Length > 0 && (address[0] < '0' || address[0] > '9')) {result.Feedback = DynamicSandwich.BadAddress; result.IsValid = false; } return result;"
    },
    "PhoneNumber": {
      "type": [ "string", "null" ],
      "pattern": "(\\(\\d{3}\\))?\\s*\\d{3}(-|\\s*)\\d{4}"
    },
    "DeliveryTime": {
      "Templates": {
        "StatusFormat": {
          "Patterns": [ "{&}: {:t}" ],
          "FieldCase": "None"
        }
      },
      "DateTime": true,
      "type": [
        "string",
        "null"
      ],
      "After": [ { "Confirm": [ "Do you want to order your {Length} {Sandwich} on {Ingredients.Bread} {&Ingredients.Bread} with {[{Ingredients.Cheese} {Ingredients.Toppings} {Ingredients.Sauces} to be sent to {DeliveryAddress} {?at {DeliveryTime}}?" ] } ]
    },
    "Rating": {
      "Describe": "your experience today",
      "type": [
        "number",
        "null"
      ],
      "minimum": 1,
      "maximum": 5,
      "After": [ { "Message": [ "Thanks for ordering your sandwich!" ] } ]
    }
  },
  "OnCompletion": "await context.PostAsync(\"We are currently processing your sandwich. We will message you the status.\");"
}
```

## <a name="implement-formflow-with-json-schema"></a>Implementación de FormFlow con esquema JSON

Para implementar FormFlow con un esquema JSON, use `FormBuilderJson`, que es compatible con la misma interfaz fluida que `FormBuilder`. En este ejemplo de código se muestra cómo implementar el esquema JSON de un bot anotado que se usa para encargar un sándwich que se ha descrito en [Personalización de un formulario mediante FormBuilder](bot-builder-dotnet-formflow-formbuilder.md).

[!code-csharp[Use JSON schema](../includes/code/dotnet-formflow-json-schema.cs#useSchema)]

## <a name="sample-code"></a>Código de ejemplo

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Recursos adicionales

- [Características básicas de FormFlow](bot-builder-dotnet-formflow.md)
- [Características avanzadas de FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Personalización de un formulario mediante FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Localización del contenido del formulario](bot-builder-dotnet-formflow-localize.md)
- [Personalización de la experiencia de usuario con lenguaje de patrones](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Builder SDK para .NET</a>

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute
