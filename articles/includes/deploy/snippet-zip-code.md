---
ms.openlocfilehash: 80cc69ca3d863a4f96f1dd241e6471074cc9976f
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386061"
---
Cuando se usa [ZIP deploy API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url) para implementar el código del bot, el comportamiento de la aplicación web o de Kudu es el siguiente:

_Kudu supone de forma predeterminada que las implementaciones de los archivos ZIP están listas para ejecutarse y que no necesitan pasos de compilación adicionales durante la implementación como, por ejemplo, npm install o dotnet restore/dotnet publish._

Por tanto, es importante incluir el código compilado con todas las dependencias necesarias en el archivo ZIP que se va a implementar en la aplicación web ya que, de lo contrario, el bot no funcionará como se espera.

> [!IMPORTANT]
> Antes de comprimir los archivos del proyecto, asegúrese de que está _en_ la carpeta del proyecto. 
> - Para bots de C#, es la carpeta que contiene el archivo .csproj. 
> - Para bots de JS, es la carpeta que contiene el archivo app.js o index.js. 
>
>Seleccione todos los archivos y comprímalos **mientras está en esa carpeta** y, a continuación, ejecute el comando también en esa carpeta. 
>
> Si la ubicación de la carpeta raíz es incorrecta, el **bot no podrá ejecutarse en Azure Portal**.
