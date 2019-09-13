---
ms.openlocfilehash: 28f49af1610f3c154e76d1cfcdb1333f08374f6c
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386045"
---
Registrar la aplicación significa que puede usar Azure AD para autenticar usuarios y solicitar acceso a recursos de usuarios. El bot requiere una aplicación registrada en Azure que le proporcione acceso al servicio Bot Framework para enviar y recibir mensajes autenticados. Para crear el registro de una aplicación mediante la CLI de Azure, ejecute el siguiente comando:

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| Opción   | DESCRIPCIÓN |
|:---------|:------------|
| display-name | Nombre para mostrar de la aplicación. |
| password | Contraseña de aplicación, también conocida como "secreto de cliente". La contraseña debe tener al menos 16 caracteres, contener al menos un carácter alfabético en mayúsculas o minúsculas, y contener al menos 1 carácter especial.|
| available-to-other-tenants| Indica si se puede usar la aplicación desde cualquier inquilino de Azure AD. Establezca `true` para permitir al bot que funcione con los canales de Azure Bot Service.|

El comando anterior da como resultado un JSON con la clave `appId`, guarde el valor de esta clave para la implementación de ARM, donde se usará para el parámetro `appId`. La contraseña proporcionada se usará para el parámetro `appSecret`.

> [!NOTE] 
> Si desea usar un registro de aplicación existente, puede usar el comando:
> ``` cmd
> az bot create --kind webapp --resource-group "<name-of-resource-group>" --name "<name-of-web-app>" --appid "<existing-app-id>" --password "<existing-app-password>" --lang <Javascript|Csharp>
> ```
