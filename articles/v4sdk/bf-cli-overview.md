---
title: Introducción a la Interfaz de la línea de comandos (CLI) de Azure Bot Framework | Microsoft Docs
description: Obtenga información sobre la interfaz de la línea de comandos (CLI) de Bot Framework.
keywords: Interfaz de la línea de comandos de Bot Framework, CLI de Bot Framework
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4780d5258af7d2c93fafece361326fd2b0f8df77
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2019
ms.locfileid: "73443193"
---
<!--TODO:
- [?] Add to TOC: Reference/Bot Framework CLI/Reference
- [?] Add other topics to the same node for each of the command groups
-->
# <a name="bot-framework-cli-overview"></a>Introducción a la CLI de Bot Framework

[!INCLUDE[applies-to](../includes/applies-to.md)]

La interfaz de la línea de comandos (CLI) de Bot Framework es una herramienta multiplataforma que permite administrar bots y servicios relacionados. Reemplaza a una colección de herramientas de la CLI independientes más antiguas mediante su agregación en una única herramienta. 

## <a name="prerequisites"></a>Requisitos previos

* [Node.js](https://nodejs.org/), versión 10.14.1 o posterior.

## <a name="installation"></a>Instalación

Instale la CLI de BF desde la línea de comandos.

~~~cmd
npm i -g @microsoft/botframework-cli
~~~

## <a name="available-commands"></a>Comandos disponibles

Los siguientes comandos están disponibles actualmente.

| Herramienta antigua | Conjunto de comandos de BF | DESCRIPCIÓN |
| :--- | :--- | :--- |
| ChatDown | [`bf chatdown`](bf-cli-reference.md#bf-chatdown) | Comandos para trabajar con archivos de diálogos de chat ( **. chat**). |
| na | [`bf config`](bf-cli-reference.md#bf-config) | Configura varias opciones dentro de la CLI. |
| LuDown, LuisGen | [`bf luis`](bf-cli-reference.md#bf-luis) | Comandos para trabajar con archivos de recursos de LUIS y administrar modelos de LUIS. |
| QnAMaker | [`bf qnamaker`](bf-cli-reference.md#bf-qnamaker) | Comandos para trabajar con archivos de recursos de QnA Maker y administrar bases de conocimiento. |

Las siguientes herramientas se trasladarán en próximas versiones:
- LUIS (API)
- Dispatch

Consulte [Asignación de portabilidad](https://github.com/microsoft/botframework-cli/blob/master/PortingMap.md) para obtener una referencia de asignación entre las herramientas antiguas y las nuevas.

_Nota: Las herramientas de la CLI más antiguas dejarán de usarse en las próximas versiones y el soporte técnico finalizará en el futuro. Todas las nuevas inversiones, correcciones de errores y nuevas características de esta área solo se dirigirán a la CLI de BF._

## <a name="overview"></a>Información general

La CLI de BF administra bots y servicios relacionados. Forma parte de Microsoft Bot Framework, una plataforma completa para crear experiencias de IA de conversación de nivel empresarial. Además de administrar los recursos relacionados con el bot, la CLI de BF se puede usar como parte de las canalizaciones de integración continua e implementación continua (CI/CD). Al crear el bot, es posible que también necesite integrar servicios de IA como LUIS para comprensión del lenguaje, QnAMaker para que el bot responda a preguntas sencillas en formato de Preguntas y respuestas, etc. Para integrar el servicio de IA en el bot, use:

* El comando [`bf luis`](bf-cli-reference.md#bf-luis) para trabajar con archivos de recursos de LUIS **.lu** y administrar los modelos de LUIS. También puede generar el código fuente (C# o JavaScript) correspondiente.
* [La herramienta de las API de LUIS](https://github.com/microsoft/botbuilder-tools/tree/master/packages/LUIS/readme.md) para implementar los archivos locales, entrenarlos, probarlos y publicarlos como modelos de Language Understanding en el servicio LUIS.
* El comando [`bf qnamaker`](bf-cli-reference.md#bf-qnamaker) para trabajar con bases de conocimiento de QnAMaker. Puede crear y administrar recursos de QnA Maker localmente y en el servicio QnA Maker.

* Consulte la [documentación](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/README.md) de la biblioteca lu para trabajar con formatos de archivo **.lu** y **.qna**.

A medida que el bot se vuelve más sofisticado, use la herramienta [Dispatch](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) de la CLI para crear, evaluar y enviar la intención entre varios modelos de LUIS y bases de conocimiento de QnA Maker.

Puede usar [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) para probar y depurar el bot. El emulador permite probar y depurar los bots en el equipo local o en la nube.

Durante las primeras fases de diseño, puede que desee crear conversaciones ficticias entre el usuario y el bot para los escenarios específicos que admitirá el bot. Use el comando [`bf chatdown`](bf-cli-reference.md#bf-chatdown) para crear archivos de conversaciones ficticias **.chat**, convertirlos en transcripciones enriquecidas y ver las conversaciones en el emulador.

Por último, con la [CLI de Azure](https://github.com/microsoft/botframework-cli/blob/master/AzureCli.md) (comando `az bot`), puede crear, descargar, publicar y configurar canales con [Azure Bot Service](https://azure.microsoft.com/services/bot-service/). Se trata de un complemento que extiende la funcionalidad de la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) para administrar los recursos de Azure Bot Service.

## <a name="privacy-and-instrumentation"></a>Privacidad e instrumentación
La CLI de BF contiene opciones de instrumentación diseñadas para ayudarnos a mejorar la herramienta en función de patrones de uso **anónimos**. __Está deshabilitada de forma predeterminada__. Si opta por participar, Microsoft recopila algunos datos de uso:

* Llamadas a grupos de comandos
* Marcas usadas, **excluyendo** valores específicos. Por ejemplo, para el parámetro `--folder:name`, solo se recopila el uso de `--folder`, no se recopila el nombre de la carpeta.

Para modificar el comportamiento de la recopilación de datos, use el comando [`bf config`](bf-cli-reference.md#bf-config).

Consulte la [Declaración de privacidad de Microsoft](https://privacy.microsoft.com/privacystatement) para más detalles.  

## <a name="issues-and-feature-requests"></a>Problemas y solicitudes de características
- Puede enviar los problemas y las solicitudes de características [aquí](https://github.com/microsoft/botframework-cli/issues).
- Puede encontrar los problemas conocidos [aquí](https://github.com/microsoft/botframework-cli/labels/known-issues).

## <a name="next-steps"></a>Pasos siguientes
- [Referencia de la CLI de BF](bf-cli-reference.md)
