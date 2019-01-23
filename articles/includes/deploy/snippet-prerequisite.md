---
ms.openlocfilehash: 266cdc2bbeeb140e4b601c1bf0fb3fa8eb085dda
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360925"
---
- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Instale de la versión más reciente de la [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
- Instale la versión más reciente de la herramienta [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
- Instale la versión más reciente de [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Instale y configure [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Conocimientos del archivo [.bot](~/v4sdk/bot-file-basics.md).

Con msbot 4.3.2 y posteriores, necesita la versión 2.0.54 de la CLI de Azure o posterior. Si ha instalado la extensión botservice, elimínela con este comando.

```cmd
az extension remove --name botservice
```