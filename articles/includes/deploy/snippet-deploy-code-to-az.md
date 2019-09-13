---
ms.openlocfilehash: 0c28cf506f2701acfc705fdcc67ec6a1e7224ad1
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386019"
---
En este momento, estamos preparados para implementar el código en la aplicación web de Azure. Ejecute el siguiente comando en la línea de comandos para realizar la implementación mediante la implementación de inserción del archivo ZIP de Kudu para una aplicación web.

```cmd
az webapp deployment source config-zip --resource-group "<resource-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| Opción   | DESCRIPCIÓN |
|:---------|:------------|
| resource-group | El nombre del grupo de recursos de Azure que contiene el bot. (Este será el grupo de recursos que usó o creó al crear el registro de la aplicación para el bot). |
| Nombre | Nombre de la aplicación web que ha usado anteriormente. |
| src  | La ruta de acceso al archivo ZIP que ha creado. |
