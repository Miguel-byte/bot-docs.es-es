---
ms.openlocfilehash: 8073e5f635d20de457e1bf5880c1b5c4c564fab4
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386097"
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
> El comando `az bot prepare-depoloy` debería generar un archivo `.deployment` en la carpeta de proyecto del bot.
