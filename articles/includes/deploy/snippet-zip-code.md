---
ms.openlocfilehash: b3fcdea923daa96a93b7b9d223354a16878b0ac2
ms.sourcegitcommit: 7e901f5f39a0cfb0d37e532321b90a1dcf4baadd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/08/2019
ms.locfileid: "72039780"
---
Cuando se usa [ZIP deploy API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url) para implementar el código del bot, el comportamiento de la aplicación web o de Kudu es el siguiente:

_Kudu supone de forma predeterminada que las implementaciones de los archivos ZIP están listas para ejecutarse y que no necesitan pasos de compilación adicionales durante la implementación como, por ejemplo, npm install o dotnet restore/dotnet publish._

Por tanto, es importante incluir el código compilado con todas las dependencias necesarias en el archivo ZIP que se va a implementar en la aplicación web ya que, de lo contrario, el bot no funcionará como se espera.

> [!IMPORTANT]
> Antes de comprimir los archivos del proyecto, asegúrese de que está _en_ la carpeta del proyecto. 
> - Para bots de C#, es la carpeta que contiene el archivo .csproj. 
> - Para los bots de JavaScript, es la carpeta que contiene el archivo app.js o index.js. 
> - En el caso de los bots de TypeScript, es la carpeta que incluye la carpeta _src_ (donde se encuentran los archivos bot.ts e index.ts). 
>
>Seleccione todos los archivos y comprímalos **mientras está en esa carpeta** y, a continuación, ejecute el comando también en esa carpeta. 
>
> Si la ubicación de la carpeta raíz es incorrecta, el **bot no podrá ejecutarse en Azure Portal**.
