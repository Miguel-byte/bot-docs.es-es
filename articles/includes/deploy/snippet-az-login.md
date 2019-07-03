---
ms.openlocfilehash: f8aad539a2d1e415833609f66cd5b398c88206f1
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252667"
---
Abra un símbolo del sistema para iniciar sesión en Azure Portal.

```cmd
az login
```

Se abrirá una ventana del explorador, lo que le permite iniciar sesión.

### <a name="set-the-subscription"></a>Establecimiento de la suscripción

Establezca la suscripción predeterminada que desea usar.

```cmd
az account set --subscription "<azure-subscription>"
```

Si no está seguro de la suscripción que debe usar para implementar el bot y desea ver la lista de `subscriptions` para su cuenta, use el comando `az account list`.

Vaya a la carpeta del bot.

```cmd
cd <local-bot-folder>
```