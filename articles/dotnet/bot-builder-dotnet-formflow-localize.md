---
title: Localización del contenido de un formulario | Microsoft Docs
description: Obtenga información sobre cómo localizar el contenido de un formulario con FormFlow y Bot Framework SDK para. NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/02/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 795232b401284becd940daed6bf7da8642c12efd
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297949"
---
# <a name="localize-form-content"></a>Localización del contenido de un formulario

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

El idioma de localización de un formulario viene determinado por las propiedades [CurrentUICulture](https://msdn.microsoft.com/library/system.threading.thread.currentuiculture(v=vs.110).aspx) y [CurrentCulture](https://msdn.microsoft.com/library/system.threading.thread.currentculture(v=vs.110).aspx) del subproceso actual.
De manera predeterminada, la referencia cultural se obtiene del campo **Configuración regional** del mensaje actual, pero es posible invalidar este comportamiento predeterminado.
Según cómo se construye el bot, la información localizada puede proceder de hasta tres orígenes diferentes:

- la localización integrada para **PromptDialog** y **FormFlow**
- un archivo de recursos que genera para las cadenas estáticas del formulario
- un archivo de recursos que crea con cadenas para los mensajes, confirmaciones o campos que se calculan dinámicamente

## <a name="generate-a-resource-file-for-the-static-strings-in-your-form"></a>Generación de un archivo de recursos para las cadenas estáticas del formulario

Las cadenas estáticas en un formulario incluyen las cadenas que genera el formulario a partir de la información de la clase de C# y las cadenas que especifica como avisos, plantillas, mensajes o confirmaciones.
Las cadenas que se generan a partir de plantillas integradas no se consideran cadenas estáticas, puesto que esas cadenas ya están localizadas.
Dado que muchas de las cadenas en un formulario se generan automáticamente, no es posible usar directamente las cadenas de recursos de C# normales.
En su lugar, puede generar un archivo de recursos para las cadenas estáticas en el formulario mediante una llamada a `IFormBuilder.SaveResources` o mediante la herramienta **RView** que se incluye con el Bot Builder SDK para. NET.

### <a name="use-iformbuildersaveresources"></a>Uso de IFormBuilder.SaveResources

Puede generar un archivo de recursos mediante una llamada a [IFormBuilder.SaveResources][saveResources] en el formulario para guardar las cadenas en un archivo .resx.

### <a name="use-rview"></a>Uso de RView

Como alternativa, puede generar un archivo de recursos que se base en el archivo .dll o .exe usando la herramienta <a href="https://aka.ms/v3-cs-RView-library" target="_blank">RView</a> que se incluye en el Bot Builder SDK para. NET.
Para generar el archivo .resx, ejecute **rview** y especifique el ensamblado que contiene el método de creación de formularios estáticos y la ruta de acceso a ese método.
En este fragmento de código se muestra cómo generar el archivo de recursos `Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.resx` con **RView**.

```csharp
rview -g Microsoft.Bot.Sample.AnnotatedSandwichBot.dll Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.BuildForm
```

En este extracto se muestra parte del archivo .resx que se genera al ejecutar este comando **rview**.

```xml
<data name="Specials_description;VALUE" xml:space="preserve">
<value>Specials</value>
</data>
<data name="DeliveryAddress_description;VALUE" xml:space="preserve">
<value>Delivery Address</value>
</data>
<data name="DeliveryTime_description;VALUE" xml:space="preserve">
<value>Delivery Time</value>
</data>
<data name="PhoneNumber_description;VALUE" xml:space="preserve">
<value>Phone Number</value>
</data>
<data name="Rating_description;VALUE" xml:space="preserve">
<value>your experience today</value>
</data>
<data name="message0;LIST" xml:space="preserve">
<value>Welcome to the sandwich order bot!</value>
</data>
<data name="Sandwich_terms;LIST" xml:space="preserve">
<value>sandwichs?</value>
</data>
```

## <a name="configure-your-project"></a>Configuración del proyecto

Después de haber generado un archivo de recursos, agréguelo al proyecto y, a continuación, siga estos pasos para establecer el idioma neutro: 

1. Haga clic con el botón derecho en el proyecto y seleccione **Aplicación**.
2. Haga clic en **Información de ensamblado**.
3. Seleccione el valor **Idioma neutro** que corresponde al idioma en que ha desarrollado el bot.

Cuando se crea el formulario, el método [IFormBuilder.Build][build] busca automáticamente recursos que contienen el nombre del tipo de formulario y los usa para localizar las cadenas estáticas en el formulario. 

> [!NOTE]
> No es posible localizar los campos calculados dinámicamente que se definen mediante [Advanced.Field.SetDefine][setDefine] (como se describe en [Using Dynamic Fields](bot-builder-dotnet-formflow-formbuilder.md#dynamically-define-field-values-confirmations-and-messages) [Uso de campos dinámicos]) de la misma manera que los campos estáticos, puesto que las cadenas para los campos que se calculan dinámicamente se construyen en el momento en que se rellena el formulario. Sin embargo, puede localizar los campos calculados dinámicamente usando mecanismos de localización de C# normales.

### <a name="localize-resource-files"></a>Localización de archivos de recursos 

Después de haber agregado archivos de recursos al proyecto, puede localizarlos con el <a href="https://developer.microsoft.com/windows/develop/multilingual-app-toolkit" target="_blank">Kit de herramientas para aplicaciones multilingües (MAT)</a>. Instale **MAT** y siga los pasos a continuación para habilitarlo para su proyecto:

1. Seleccione el proyecto en el Explorador de soluciones de Visual Studio.
2. Haga clic en **Herramientas**, **Kit de herramientas de aplicaciones multilingües** y **Habilitar**.
3. Haga clic con el botón derecho en el proyecto y seleccione **Kit de herramientas de aplicaciones multilingües**, **Add Translations** (Agregar traducciones) para seleccionar las traducciones. Esto creará archivos <a href="https://en.wikipedia.org/wiki/XLIFF" target="_blank">XLF</a> estándares del sector, que puede traducir de automática o manualmente.

> [!NOTE]
> Aunque este artículo describe cómo usar el Kit de herramientas de aplicaciones multilingües para traducir el contenido, puede implementar la localización a través de una variedad de otros medios.

## <a name="see-it-in-action"></a>Véalo en acción

Este ejemplo de código se basa en el de [Customize a form using FormBuilder](bot-builder-dotnet-formflow-formbuilder.md) (Personalización de un formulario mediante FormBuilder) para implementar la localización como se describió anteriormente. En este ejemplo, la clase `DynamicSandwich` (no se muestra aquí) contiene información de localización para los campos que se calculan dinámicamente, mensajes y confirmaciones.

[!code-csharp[Build localized form](../includes/code/dotnet-formflow-localize.cs#buildLocalizedForm)]

Este fragmento de código muestra la interacción resultante entre el bot y el usuario cuando `CurrentUICulture` es **Francés**.

```console
Bienvenue sur le bot d'ordre "sandwich" !
Quel genre de "sandwich" vous souhaitez sur votre "sandwich"?
 1. BLT
 2. Jambon Forêt Noire
 3. Poulet Buffalo
 4. Faire fondre le poulet et Bacon Ranch
 5. Combo de coupe à froid
 6. Boulette de viande Marinara
 7. Poulet rôti au four
 8. Rôti de boeuf
 9. Rotisserie poulet
 10. Italienne piquante
 11. Bifteck et fromage
 12. Oignon doux Teriyaki
 13. Thon
 14. Poitrine de dinde
 15. Veggie
> 2

Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> ?
* Vous renseignez le champ longueur.Réponses possibles:
* Vous pouvez saisir un numéro 1-2 ou des mots de la description. (Six pouces, ou Pied Long)
* Retourner à la question précédente.
* Assistance: Montrez les réponses possibles.
* Abandonner: Abandonner sans finir
* Recommencer remplir le formulaire. (Vos réponses précédentes sont enregistrées.)
* Statut: Montrer le progrès en remplissant le formulaire jusqu'à présent.
* Vous pouvez passer à un autre champ en entrant son nom. ("Sandwich", Longueur, Pain, Fromage, Nappages, Sauces, Adresse de remise, Délai de livraison, ou votre expérience aujourd'hui).
Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> 1

Quel genre de pain vous souhaitez sur votre "sandwich"?
 1. Neuf grains de blé
 2. Neuf grains miel avoine
 3. Italien
 4. Fromage et herbes italiennes
 5. Pain plat
> neuf
Par pain "neuf" vouliez-vous dire (1. Neuf grains miel avoine, ou 2. Neuf grains de blé)
```

## <a name="sample-code"></a>Código de ejemplo

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Recursos adicionales

- [Características básicas de FormFlow](bot-builder-dotnet-formflow.md)
- [Características avanzadas de FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Customize a form using FormBuilder](bot-builder-dotnet-formflow-formbuilder.md) (Personalización de un formulario mediante FormBuilder)
- [Define a form using JSON schema](bot-builder-dotnet-formflow-json-schema.md) (Definición de un formulario mediante esquemas JSON)
- [Personalización de la experiencia de usuario con lenguaje de patrones](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Referencia de Bot Framework SDK para .NET</a>

[build]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1.build 

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[saveResources]: /dotnet/api/microsoft.bot.builder.formflow.iform-1.saveresources
