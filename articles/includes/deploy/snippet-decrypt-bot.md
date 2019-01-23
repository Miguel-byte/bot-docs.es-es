---
ms.openlocfilehash: eac6abae509d92ea4714bc01221f180ea575950f
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360900"
---
Obtenga la clave de cifrado.

1. Inicie sesión en [Azure Portal](http://portal.azure.com/).
1. Abra el recurso de bot de aplicación web de su bot.
1. Abra la **configuración de la aplicación** del bot.
1. En la ventana **Configuración de la aplicación**, desplácese hacia abajo a **Configuración de la aplicación**.
1. Busque **botFileSecret** y copie su valor.

Descifre el archivo .bot.

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| --bot | La ruta de acceso relativa al archivo .bot descargado. |
| --secret | La clave de cifrado. |

Copie el archivo `.bot` descifrado al directorio que contiene el proyecto de bot local, actualice el bot para que use este nuevo archivo `.bot` y elimine el archivo `.bot` antiguo.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

En **appsettings.json**, actualice la propiedad **botFilePath** para que apunte al nuevo archivo `.bot` que ha agregado al directorio local.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

En **.env**, actualice la propiedad **botFilePath** para que apunte al nuevo archivo `.bot` que ha agregado al directorio local.

---

Una vez que se ha actualizado el bot, elimine el directorio temporal del bot descargado.