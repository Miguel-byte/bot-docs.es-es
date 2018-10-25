---
title: Migración de un bot a Azure | Microsoft Docs
description: Obtenga información sobre cómo migrar su bot desde el Portal de Bot Framework heredado a un servicio de bot en Azure portal.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 6/22/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6a1ae4d0966928a1b95e45ea7bcc62e9c1d14666
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000072"
---
# <a name="migrate-your-bot-to-azure"></a>Migración de un bot a Azure



Todos los bots de **Azure Bot Service (versión preliminar)** creados en el [Portal de Bot Framework](http://dev.botframework.com) se deben migrar al nuevo Bot Service en Azure. El servicio se ofreció con disponibilidad general (GA) en diciembre de 2017. 

Tenga en cuenta que *no* es necesario migrar los bots de registro conectados solo a los siguientes canales: **Teams**, **Skype** o **Cortana**. Por ejemplo, un bot de registro conectado a **Facebook** y **Skype** *se debe* migrar, pero con un bot de registro conectado a **Skype**y **Cortana** *no es necesario* realizar la migración.

> [!IMPORTANT]
> Antes de migrar un bot de Functions creado con Node.js, es necesario utilizar **Azure Functions Pack** para empaquetar juntos los módulos **node_modules**. Si lo hace, mejorará el rendimiento durante la migración y la ejecución del bot de Functions después de su migración. Para empaquetar los módulos, consulte [Empaquetado de un bot de Functions con Funcpack](#package-a-functions-bot-with-funcpack).

Para migrar el bot, realice lo siguiente:

1. Inicie sesión en el [Portal de Bot Framework](http://dev.botframework.com) y haga clic en **Mis bots**.
2. Haga clic en el botón **Migrar** del bot que desea migrar.
3. Acepte los **Términos** y haga clic en **Migrar** para iniciar el proceso de migración o haga clic en **Cancelar** para cancelar esta acción.

> [!IMPORTANT]
> Mientras la tarea de migración está en curso, no salga de la página ni actualice la página. Si lo hace, hará que la tarea de migración se detenga de forma prematura y tendrá que realizar la acción de nuevo. Para asegurarse de que la migración se realiza correctamente, espere el mensaje de confirmación.

Después de que el proceso de migración finaliza correctamente, el **Estado de migración** indicará que el bot se ha migrado y estará disponible un botón **Revertir la migración** durante una semana después de la fecha de migración en caso de problemas.

Al hacer clic en el nombre de un bot migrado, se abrirá el bot en [Azure Portal](http://portal.azure.com).

## <a name="package-a-functions-bot-with-funcpack"></a>Empaquetado de un bot de Functions con Funcpack

Los bots de Functions creados con Node.js deben empaquetarse con [Funcpack](https://github.com/Azure/azure-functions-pack) antes de su migración. Para empaquetar el proyecto con Funcpack, siga estos pasos:

1.  [Descargue](bot-service-build-download-source-code.md) el código localmente si no lo ha hecho ya.
2.  Actualice todos los paquetes de npm de **packages.json** a las versiones más recientes y ejecute `npm install`.
3.  Abra **messages/index.js** y cambie `module.exports = { default: connector.listen() }` por `module.exports = connector.listen();`
4.  Instale Funcpack con npm: `npm install -g azure-functions-pack`
5.  Para empaquetar el directorio **node_modules**, ejecute el siguiente comando: `funcpack pack ./`
6.  Pruebe el bot localmente mediante la ejecución del bot de Functions con el emulador de Bot Framework. Más información sobre cómo ejecutar el bot de *funcpack* [aquí](https://github.com/Azure/azure-functions-pack#how-to-run). 
7.  Cargue el código a Azure. Asegúrese de que se carga el directorio `.funcpack`. No es necesario cargar el directorio **node_modules**.
8. Pruebe el bot en remoto para asegurarse de que responde según lo previsto.
9. [Realice la migración del bot](#migrate-your-bot-to-azure) siguiendo los pasos anteriores.

## <a name="migration-under-the-hood"></a>Funcionamiento interno de la migración

En función del tipo de bot que vaya a migrar, la lista siguiente puede ayudarle a entender mejor lo que sucede en segundo plano.

* **Bot de aplicación web** o **Bot de Functions**: para estos tipos de bots, el código fuente del proyecto se copia desde el bot antiguo al nuevo bot. Otros recursos, como el almacenamiento del bot, Application Insights, LUIS, etc. se dejan como están. En esos casos, el bot nuevo contiene una copia de los identificadores, las claves y las contraseñas de esos recursos existentes. 
* **Bot de registro de canales**: para este tipo de bots, el proceso de migración simplemente crear un nuevo **Bot de registro de canales** y copia el punto de conexión del bot antiguo. 
* Independientemente de los tipos de los bots que se van a migrar, el proceso de migración no modifica el estado del bot existente de ningún modo. Esto le permitirá revertir de forma segura si necesita hacerlo.
* Si tiene configurada la [implementación continua](bot-service-build-continuous-deployment.md), deberá configurarla de nuevo para que el control de código fuente se conecte con el nuevo bot.

## <a name="understanding-azure-resources-after-migration"></a>Descripción de los recursos de Azure después de la migración
Una vez que se realiza la migración, el **grupo de recursos** contendrá una serie de recursos de Azure que son necesarios para que el bot funcione. El tipo y número de recursos dependen del tipo de bot que haya migrado. Consulte las secciones siguientes para más información.

### <a name="registration-bot"></a>Bot de registro

Este es el tipo más sencillo. El grupo de recursos de Azure solo contendrá un recurso del tipo "Bot de registro de canales". Esto es el equivalente al anterior registro del bot en el Portal para desarrolladores de Bot Framework.

![Listas de bots de registro de canales en Azure](~/media/bot-service-migrate-bot/channel-registration-bot.png)

### <a name="web-app-bot"></a>Bot de aplicación web
La migración de un bot de aplicación web aprovisionará un recurso de Bot Service del tipo "Bot de aplicación Web" y una nueva aplicación web de App Service (en verde en la siguiente captura de pantalla). El bot de Azure Bot Service (versión preliminar) anterior seguirá existiendo y se puede eliminar (en rojo en la siguiente captura de pantalla).

![Listas de bots de aplicación web en Azure](~/media/bot-service-migrate-bot/web-app-bot.png)

### <a name="functions-bot"></a>Bot de Functions
La migración de un bot de Functions aprovisionará un recurso de Bot Service del tipo "Bot de Functions" y una nueva aplicación de función de App Service (en verde en la siguiente captura de pantalla). El bot de Azure Bot Service (versión preliminar) anterior seguirá existiendo y se puede eliminar (en rojo en la siguiente captura de pantalla).

![Listas de bots de Functions en Azure](~/media/bot-service-migrate-bot/functions-bot.png)


## <a name="roll-back-migration"></a>Reversión de la migración

En caso de que algo haya ido mal con el bot durante la migración o después de la misma, puede **revertir la migración**. Para revertir una migración, realice lo siguiente:

1. Inicie sesión en el [Portal de Bot Framework](http://dev.botframework.com) y haga clic en **Mis bots**.
2. Haga clic en el botón **Revertir migración** del bot que desea revertir. Aparecerá un aviso.
3. Haga clic en **Sí, revertir** para continuar o en **Cancelar** para cancelar la acción de reversión.

> [!NOTE]
> La funcionalidad de reversión estará disponible durante una semana después de la migración y debe usarse solo si surgen problemas en el bot migrado.

## <a name="migration-troubleshootingknown-issues"></a>Solución de problemas de migración y problemas conocidos
Mi bot de Node.js o de Functions se ha migrado correctamente, pero no responde:

* **Compruebe el punto de conexión**
  * Vaya a la hoja **Configuración** en el recurso del bot y compruebe que el punto de conexión del bot tiene un parámetro de cadena de consulta llamado "code" con un valor en él. Si no es así, debe agregarlo.
* **Compruebe si hay copias de seguridad en la carpeta de secretos de Kudu**
  * En algunos casos excepcionales, puede haber algunos archivos secretos de copia de seguridad que están creando un conflicto. Vaya a la carpeta **home\data\Functions\secrets** en Kudu y elimine los archivos **host.snapshot** (o **host.backup**) que encuentre. Debería haber solo un archivo **host.json** y un **messages.json**. Finalmente, reinicie App Service y vuelva a intentar charlar con el bot.

Para cualquier otro problema, envíe un CRI al soporte técnico de Azure o envíe un problema en [GitHub](https://github.com/MicrosoftDocs/bot-framework-docs/issues).


## <a name="next-steps"></a>Pasos siguientes

Ahora que el bot se migrado, obtenga información sobre cómo administrar el bot desde Azure Portal.

> [!div class="nextstepaction"]
> [Administración de un bot](bot-service-manage-overview.md)
