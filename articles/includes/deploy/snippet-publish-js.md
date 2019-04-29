---
ms.openlocfilehash: 0891b9652154f8ed086cc45ce6018aa0be1a67b8
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905177"
---
Para publicar el bot de JavaScript local de nuevo en Azure, primero debe crear manualmente un único archivo comprimido que contenga todos los archivos usados para crear y ejecutar el bot localmente. Esto incluye todas las bibliotecas npm descargadas en la carpeta `node_modules`. Cuando cree este archivo zip _, asegúrese de que el directorio raíz que utiliza es el mismo directorio donde reside su archivo index.js_.

Cuando se crea un archivo zip que contiene todo el código fuente del bot, abra una ventana del símbolo del sistema y ejecute el siguiente comando _Az cli_. 

Este paso puede tardar unos minutos.

```cmd
az webapp deployment source config-zip --resource-group <resource-group-name> --name <bot-resource-name> --src <directory-path>
```

| Opción | DESCRIPCIÓN |
|:---|:---|
| --resource-group | Nombre del grupo de recursos. |
| --name | El nombre de recurso del bot en Azure. |
| --src | La ruta de acceso completa del directorio desde el que se carga el código de bot comprimido. Por ejemplo, `c:\my-local-repository\this-app-folder\my-zipped-code.zip`. |

Cuando esto se completa correctamente, el bot se implementa en Azure.
