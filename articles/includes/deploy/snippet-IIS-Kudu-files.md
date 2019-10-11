---
ms.openlocfilehash: bbe74a9a82d3bd04593384d825d373bfab35e3db
ms.sourcegitcommit: 7e901f5f39a0cfb0d37e532321b90a1dcf4baadd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/08/2019
ms.locfileid: "72039781"
---
Debe preparar los archivos de proyecto para poder implementar el bot. 
<!-- **C# bots** -->
##### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

Debe proporcionar la ruta de acceso al archivo .csproj relacionada con --code-dir. Esto puede realizarse mediante el argumento --proj-file-path. El comando debería resolver --code-dir y --proj-file-path en "./MyBot.csproj"

<!-- **JavaScript bots** -->
##### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

Este comando capturará un archivo web.config que es necesario para que las aplicaciones de Node.js funcionen con IIS en Azure App Services. Asegúrese de que el archivo web.config se guarda en la raíz del bot.

<!-- **TypeScript bots** -->
##### <a name="typescripttabtypescript"></a>[TypeScript](#tab/typescript)

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

Este comando funciona de forma similar al de JavaScript que se mencionó anteriormente, pero para un bot de TypeScript.

---

> [!NOTE]
>  En el caso de los bots de C# y JavaScript, el comando `az bot prepare-depoloy` generará un archivo `.deployment` en la carpeta de proyecto del bot.
> En el caso de los bots de TypeScript, el comando generará dos archivos `web.config`. Uno está en la carpeta del proyecto y otro en la carpeta **src** dentro de la carpeta del proyecto. 
