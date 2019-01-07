Si usa servicios como LUIS, también deberá pasar `luisAuthoringKey`. Si desea usar un grupo de recursos existente en Azure, utilice el argumento `groupName` con el comando anterior.

Se recomienda encarecidamente utilizar la opción `verbose` para ayudar a solucionar los problemas que puedan aparecer durante la implementación del bot. A continuación se describen otras opciones que se usan con el comando `msbot clone services`:

| Argumentos    | DESCRIPCIÓN |
|--------------|-------------|
| `folder`     | Ubicación del archivo `bot.recipe`. De forma predeterminada el archivo de receta se crea en `DeploymentsScript/MSBotClone`. NO MODIFIQUE dicho archivo.|
| `location`   | Ubicación geográfica utilizada para crear los recursos del servicio de bot. Por ejemplo, eastus, westus, westus2, etc.|
| `proj-file`  | En el caso del bot de C#, es el archivo .csproj. En el caso del bot de JS, es el nombre de archivo del proyecto de startup (por ejemplo, index.js) de su bot local.|
| `name`       | Nombre único que se usa para implementar el bot en Azure. El nombre puede ser el mismo que el de su bot local. NO incluya espacios ni guiones bajos en el nombre.|
| `luisAuthoringKey` | La clave de creación de la región de creación adecuada de LUIS para los recursos de este. |

Para poder crear recursos de Azure, se le solicitará que complete la autenticación. Siga las instrucciones que aparecen en pantalla para completar este paso.

Tenga en cuenta que el paso anterior tarda _muy poco tiempo_ en completarse y que los nombres de los recursos que se crean en Azure están alterados. Para más información acerca de la alteración de nombres, consulte el [problema 796](https://github.com/Microsoft/botbuilder-tools/issues/796) en el repositorio de GitHub.
