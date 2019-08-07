---
title: Preguntas frecuentes de Bot Service| Microsoft Docs
description: Una lista de preguntas frecuentes acerca de los elementos de Bot Framework y cuándo estarán disponibles las nuevas características.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/21/2019
ms.openlocfilehash: 57efc2f1d137988792d9484d7f1f857bd193ecfa
ms.sourcegitcommit: 23a1808e18176f1704f2f6f2763ace872b1388ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/25/2019
ms.locfileid: "67405782"
---
# <a name="bot-framework-frequently-asked-questions"></a>Preguntas frecuentes de Bot Framework

En este artículo se muestran respuestas a preguntas frecuentes acerca de Bot Framework.

## <a name="background-and-availability"></a>Segundo plano y disponibilidad
### <a name="why-did-microsoft-develop-the-bot-framework"></a>¿Por qué Microsoft desarrolló Bot Framework?

A pesar de que ya contamos con la interfaz de usuario de conversación (CUI), por el momento, pocos desarrolladores tienen la experiencia y las herramientas necesarias para crear nuevos servicios de conversación o habilitar aplicaciones y servicios existentes con una interfaz de conversación que sus usuarios puedan disfrutar. Hemos creado Bot Framework para facilitar a los desarrolladores la compilación y conexión de bots excelentes para los usuarios, dondequiera que conversen, incluidos los canales Premier de Microsoft.

### <a name="what-is-the-v4-sdk"></a>¿Qué es el SDK v4?
Bot Framework SDK v4 aprovecha la experiencia y los aprendizajes de los SDK anteriores. Incorpora los niveles adecuados de abstracción y permite la creación de componentes avanzados de los bloques de compilación del bot. Puede comenzar con un bot simple y hacer que su bot sea cada vez más sofisticado con un marco modular y extensible. Puede encontrar [preguntas frecuentes](https://github.com/Microsoft/botbuilder-dotnet/wiki/FAQ) sobre el SDK en GitHub.

## <a name="bot-framework-sdk-version-3-lifetime-support"></a>Compatibilidad mientras dure la vigencia del SDK versión 3 de Bot Framework 
Los bots de SDK V3 se seguirán ejecutando y serán compatibles con Azure Bot Service.  Después de la versión 4 del SDK de Bot Framework, al igual que con otras plataformas, la versión 3 continuará siendo compatible mediante correcciones de errores de seguridad de alta prioridad y actualizaciones de capas de conectores o protocolos.  Los clientes pueden confiar en que la compatibilidad con la versión v3 continuará durante 2019.

### <a name="what-is-microsoft-plan-for-supporting-existing-v3-bots-what-happens-to-my-v3-bots-will-my-v3-bots-stop-working"></a>¿Cuál es el plan de Microsoft para la compatibilidad con los bots de V3 existentes? ¿Qué ocurrirá con mis bots V3? ¿Dejarán de funcionar mis bots V3?
Los bots de SDK V3 se seguirán ejecutando y serán compatibles con Azure Bot Service.  Después de la versión 4 del SDK de Bot Framework, al igual que con otras plataformas, la versión 3 continuará siendo compatible mediante correcciones de errores de seguridad de alta prioridad y actualizaciones de capas de conectores o protocolos.  Los clientes pueden confiar en que la compatibilidad con la versión v3 continuará durante 2019.
- Azure Bot Service y Bot Framework V3 son ambos productos con disponibilidad general y son totalmente compatibles. Las bibliotecas subyacentes de protocolos y conectores de Bot Framework no han cambiado y las comparten las versiones V3 y V4.  
- Los bots creados con SDK versión 3 de Bot Framework (BotBuilder) seguirán siendo compatibles durante 2019. 
- Los clientes pueden continuar creando bots de la versión V3 mediante las herramientas de Azure Portal o de la CLI de Azure.

### <a name="what-happens-to-my-bot-written-to-rest--bot-framework-protocol-31"></a>¿Qué le pasará a mi bot escrito para REST y para el protocolo 3.1 de Bot Framework?
- Azure Bot Service y Bot Framework V3 son ambos productos con disponibilidad general y son totalmente compatibles.
- El protocolo de Bot Framework no ha cambiado y los comparten las versiones V3 y V4 de los SDK.  

### <a name="will-there-be-more-updates-additional-development-for-the-v3-sdk-or-just-bugfixes"></a>¿Habrá más actualizaciones y desarrollo adicional para el SDK versión V3 o solo correcciones de errores?  
- Vamos a actualizar la versión V3 con pequeñas mejoras, principalmente en la capa del conector, y con correcciones de errores de seguridad y de alta prioridad.  
- Las actualizaciones de la versión V3 se publicarán como mínimo dos veces al año y cuando sea necesario, según las correcciones de errores y los cambios de protocolos necesarios. 
- El plan actual es publicar versiones menores y revisiones de la versión V3 para NuGet y NPM de los SDK de C# y JavaScript.

### <a name="why-v4-is-not-backwards-compatible-with-v3"></a>¿Por qué la versión V4 no es compatible con V3?
- En el nivel de protocolo, la comunicación entre la aplicación de conversación (también conocida como su bot) con distintos canales usa el protocolo de actividad de Bot Framework que es idéntico para las versiones V3 y V4. La misma infraestructura subyacente de Azure Bot Service (AZURE BOT SERVICE) admite bots de ambas versiones.
- El SDK V4 de Bot Framework ofrece una experiencia de desarrollo centrada en la conversación con una arquitectura de SDK modular y extensible que permite a los desarrolladores crear aplicaciones sólidas y sofisticadas. El diseño ampliable de la versión V4 se basó en los comentarios de los clientes, que sugerían que los modelos de diálogo y los elementos primitivos del SDK versión V3 eran complejos y limitaban la extensibilidad.  

### <a name="what-is-the-general-migration-strategy-i-have-a-v3-bot-how-can-i-migrate-it-to-v4-can-i-migrate-my-v3-bot-to-v4"></a>¿Qué es la estrategia general de migración? Tengo un bot de la versión V3, ¿cómo puedo migrarlo a la versión V4? ¿Es posible?

- Consulte las [diferencias entre las versiones v3 y v4 del SDK para .NET](v4sdk/migration/migration-about.md) para más información sobre la migración de bots de v3 a v4.
- En este momento, la ayuda para migrar los bots que se crearon con la versión v3 a v4 vendrá en forma de documentación y ejemplos. En estos momentos, no está previsto incluir en la versión 4 de SDK ninguna de las capas de compatibilidad de la versión 3 que permita a los bots de compilación trabajar con los bots de la versión 4.
- Si ya tiene bots con el SDK versión V3 de Bot Framework en producción, no se preocupe, seguirán funcionando de la misma manera en un futuro inmediato.
- El SDK V4 de Bot Framework es una evolución del exitoso SDK V3. La versión V4 es una versión principal que incluye cambios importantes que impiden a los bots de la versión V3 ejecutarse en la versión V4 que es más reciente.

### <a name="should-i-build-new-a-bot-using-v3-or-v4"></a>¿Debo compilar un nuevo bot con la versión V3 o la V4?
- Para nuevas experiencias de conversación, se recomienda que inicie un nuevo bot con el SDK versión V4 de Bot Framework.
- Si ya está familiarizado con el SDK versión 3 de Bot Framework, dedique un tiempo a conocer la nueva versión y las características que ofrece el nuevo [SDK versión V4 de Bot Framework](http://aka.ms/botframeowrkoverview).
- Si ya tiene bots con el SDK versión V3 de Bot Framework en producción, no se preocupe, seguirán funcionando de la misma manera en un futuro inmediato.
- Puede crear bots con el SDK versión V4 y con la versión anterior V3 de Bot Framework a través de Azure Portal y de la línea de comandos de Azure. 

### <a name="how-can-i-migrate-azure-bot-service-from-one-region-to-another"></a>¿Cómo puedo migrar Azure Bot Service de una región a otra?

Azure Bot Service no admite el movimiento de región. Es un servicio global que no está ligado a ninguna región específica.

## <a name="channels"></a>Canales
### <a name="when-will-you-add-more-conversation-experiences-to-the-bot-framework"></a>¿Cuándo se agregarán más experiencias de conversación a Bot Framework?

Planeamos realizar mejoras continuas a Bot Framework, incluida la incorporación de más canales, pero no podemos proporcionar una programación en este momento.  
Si quiere que se agregue un canal específico al marco, [díganoslo][Support].

### <a name="i-have-a-communication-channel-id-like-to-be-configurable-with-bot-framework-can-i-work-with-microsoft-to-do-that"></a>Me gustaría poder configurar un canal de comunicación que tengo con Bot Framework. ¿Puedo trabajar con Microsoft para hacer eso?

No hemos proporcionado ningún mecanismo general para que los desarrolladores agreguen nuevos canales a Bot Framework, pero se puede conectar el bot a la aplicación a través de la [API Direct Line][DirectLineAPI]. Si es programador de un canal de comunicación y le gustaría colaborar con nosotros para habilitar el canal en Bot Framework, [nos encantaría conocer su opinión][Support].

### <a name="if-i-want-to-create-a-bot-for-microsoft-teams-what-tools-and-services-should-i-use"></a>Si quiero crear un bot de Microsoft Teams, ¿qué herramientas y servicios debo usar?

Bot Framework está diseñado para la compilación, la conexión y la implementación de bots de alta calidad, de capacidad de respuesta, de alto rendimiento y escalables para Teams y muchos otros canales. El SDK puede usarse para crear bots de texto/sms, imágenes, botones y compatibles con tarjetas (que constituyen la mayoría de las interacciones de bots actuales en todas las experiencias de conversación), así como interacciones de bot específicas de Teams, como servicios de audio y vídeo de alta calidad.

Si ya tiene un bot excelente y le gustaría llegar a la audiencia de Teams, su bot se puede conectar fácilmente a Teams (o a cualquier canal admitido) mediante Bot Framework para la API de REST (siempre y cuando tenga un punto de conexión REST accesible a través de Internet).

### <a name="how-do-i-create-a-bot-that-uses-the-us-government-data-center"></a>¿Cómo se puede crear un bot que use el centro de datos del Gobierno de Estados Unidos?

Hay 2 pasos principales necesarios para crear un bot que use un centro de datos del Gobierno de Estados Unidos.
1. Agregar un "proveedor de canal" en appsettings.json (o configuración de App Service). Debe establecerse específicamente en esta constante de nombre/valor: ChannelService = "https://botframework.azure.us". A continuación, se muestra un ejemplo de uso de appsetting.json.

```json
{
  "MicrosoftAppId": "", 
  "MicrosoftAppPassword": "",
  "ChannelService": "https://botframework.azure.us"
}
```
2. Si se usa .NET Core, deberá agregar un elemento ConfigurationChannelProvider en el archivo startup.cs. La forma de hacerlo varía en función de la versión del SDK que use.

- Para las versiones 4.3 y posteriores, en el método ConfigureServices, deberá crear una instancia de ConfigurationChannelProvider. Cuando se usa la clase BotFrameworkHttpAdapter, se inyecta como singleton en la colección de servicios, de esta manera:

```csharp  
services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>();
```
- Para las versiones anteriores a 4.3, en el método ConfigureServices, busque el método AddBot. Al configurar las opciones, asegúrese de que agregar:

```csharp
options.ChannelProvider = new ConfigurationChannelProvider();
```
Puede encontrar más información sobre Government Services [aquí](https://docs.microsoft.com/en-us/azure/azure-government/documentation-government-services-aiandcognitiveservices#azure-bot-service)

## <a name="security-and-privacy"></a>Seguridad y privacidad
### <a name="do-the-bots-registered-with-the-bot-framework-collect-personal-information-if-yes-how-can-i-be-sure-the-data-is-safe-and-secure-what-about-privacy"></a>¿Los bots registrados con Bot Framework recopilan información personal? En caso afirmativo, ¿cómo puedo estar seguro de que los datos están a salvo y protegidos? ¿Qué pasa con la privacidad?

Cada bot es su propio servicio y los desarrolladores de esos servicios tienen que proporcionar los Términos del servicio y la Declaración de privacidad según su Código de conducta.  Puede acceder a esta información en la tarjeta del bot en el respectivo directorio.

Para proporcionar el servicio de E/S, Bot Framework transmite el mensaje y el contenido del mensaje (incluido el id.) desde el servicio de chat que usó para el bot.

### <a name="can-i-host-my-bot-on-my-own-servers"></a>¿Puedo hospedar el bot en mis propios servidores?
Sí. El bot puede hospedarse en cualquier lugar de Internet. En sus propios servidores, en Azure o en cualquier otro centro de datos. El único requisito es que el bot debe exponer un punto de conexión HTTPS accesible públicamente.

### <a name="how-do-you-ban-or-remove-bots-from-the-service"></a>¿Cómo prohibir o quitar bots del servicio?

Los usuarios tienen una manera de notificar un bot que presenta un comportamiento incorrecto a través de la tarjeta de contacto del bot en el directorio. Los desarrolladores deben atenerse a los términos del servicio de Microsoft para participar en el servicio.

### <a name="which-specific-urls-do-i-need-to-whitelist-in-my-corporate-firewall-to-access-bot-framework-services"></a>¿Qué direcciones URL específicas necesito incluir en la lista de direcciones permitidas del firewall corporativo para acceder a los servicios de Bot Framework?
Si tiene un firewall de salida que bloquea el tráfico procedente del bot hacia Internet, necesitará incluir en la lista de direcciones permitidas de dicho firewall las direcciones URL siguientes:
- login.botframework.com (autenticación de bot)
- login.microsoftonline.com (autenticación de bot)
- westus.api.cognitive.microsoft.com (para la integración de NLP de Luis.ai)
- state.botframework.com (almacenamiento de estado del bot para la creación de prototipos)
- cortanabfchanneleastus.azurewebsites.net (canal de Cortana)
- cortanabfchannelwestus.azurewebsites.net (canal de Cortana)
- *.botframework.com (canales)

### <a name="can-i-block-all-traffic-to-my-bot-except-traffic-from-the-bot-connector-service"></a>¿Puedo bloquear todo el tráfico al bot excepto el tráfico procedente del servicio Bot Connector?
No. Este tipo de listas de direcciones permitidas para direcciones IP o DNS no es práctico. El servicio Bot Framework Connector se hospeda en centros de datos de Azure en todo el mundo y la lista de direcciones IP de Azure cambia constantemente. La creación de listas de direcciones permitidas con direcciones IP determinadas puede funcionar un día y al siguiente no, cuando cambien las direcciones IP de Azure.
 
### <a name="what-keeps-my-bot-secure-from-clients-impersonating-the-bot-framework-connector-service"></a>¿Qué protege al bot de los clientes que suplanten el servicio Bot Framework Connector?
1. El token de seguridad que acompaña a cada solicitud para el bot incluye la dirección URL de servicio codificada, lo que significa que aunque un atacante obtenga acceso al token, no podrá redirigir la conversación a una nueva dirección URL de servicio. Esto se aplica en todas las implementaciones del SDK y está documentado en los materiales de [referencia](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-connector-authentication?view=azure-bot-service-3.0#bot-to-connector) de autenticación.

2. Si el token entrante es incorrecto o falta, el SDK de Bot Framework no generará un token en la respuesta. Esto limita los posibles daños si el bot se configura incorrectamente.
3. En el bot, puede comprobar manualmente la dirección URL de servicio proporcionada en el token. Esto hace que el bot sea más frágil en caso de cambios en la topología del servicio por lo que, aunque es posible, no se recomienda.


Tenga en cuenta que estas son conexiones salientes desde el bot a Internet. No hay una lista de los nombres DNS o las direcciones IP que el servicio Bot Framework Connector utiliza para comunicarse con el bot. No se admite la creación de listas de direcciones IP de entrada permitidas.

## <a name="rate-limiting"></a>Limitación de frecuencia
### <a name="what-is-rate-limiting"></a>¿Qué es la limitación de frecuencia?
El servicio Bot Framework debe protegerse a sí mismo y a sus clientes contra patrones de llamada abusivos (p. ej., ataque por denegación de servicio), de modo que ningún bot pueda afectar negativamente al rendimiento de otros bots. Para lograr este tipo de protección, agregamos límites de frecuencia (también conocidos como limitaciones) a todos los puntos de conexión. Al aplicar un límite de frecuencia, podemos restringir la frecuencia con la que un bot puede realizar una llamada específica. Por ejemplo: al habilitar la limitación de frecuencia, si un bot quiere publicar un gran número de actividades, deberá hacerlo a intervalos durante un período de tiempo. Tenga en cuenta que el propósito de limitación de frecuencia no es limitar el volumen total de un bot. Está diseñado para evitar el abuso de la infraestructura de conversación que no sigue los patrones de conversación humana.

### <a name="how-will-i-know-if-im-impacted"></a>¿Cómo sé si esto me afecta?
Es poco probable que sufra una limitación de frecuencia, incluso en grandes volúmenes. La mayoría de limitaciones de frecuencia ocurren debido al envío masivo de actividades (desde un bot o desde un cliente), pruebas de carga extrema o un error. Cuando se limita una solicitud, se devuelve una respuesta HTTP 429 (demasiadas solicitudes) junto con un encabezado Retry-After que indica el tiempo (en segundos) que se debe esperar para que el reintento de la solicitud se realice correctamente. Puede recopilar esta información habilitando los análisis para el bot a través de Azure Application Insights. O bien, puede agregar un código en el bot para registrar los mensajes. 

### <a name="how-does-rate-limiting-occur"></a>¿Cómo ocurre la limitación de frecuencia?
Esto puede suceder si:
-   Un bot envía mensajes con demasiada frecuencia.
-   El cliente de un bot envía mensajes con demasiada frecuencia.
-   Los clientes de Direct Line solicitan un nuevo socket web con demasiada frecuencia.

### <a name="what-are-the-rate-limits"></a>¿Qué son los límites de frecuencia?
Ajustamos continuamente los límites de frecuencia para que sean tan flexibles como sea posible y al mismo tiempo para proteger nuestro servicio y nuestros usuarios. Ya que en ocasiones los umbrales cambiarán, no publicamos los números por el momento. En caso de verse afectado por la limitación de frecuencia, no dude en ponerse en contacto con nosotros en [bf-reports@microsoft.com](mailto://bf-reports@microsoft.com).

## <a name="related-services"></a>Servicios relacionados
### <a name="how-does-the-bot-framework-relate-to-cognitive-services"></a>¿Cómo se relaciona Bot Framework con Cognitive Services?

Tanto Bot Framework como [Cognitive Services](http://www.microsoft.com/cognitive) son funcionalidades nuevas en [Microsoft Build 2016](http://build.microsoft.com) que también se integrarán en Cortana Intelligence Suite cuando esté disponible de forma general. Estos dos servicios se crearon tras varios años de investigación y uso en los productos populares de Microsoft. Estas funcionalidades combinadas con "Cortana Intelligence" permiten que cualquier organización pueda aprovechar la eficacia de los datos, la nube y la inteligencia para crear sus propios sistemas inteligentes que ofrezcan nuevas oportunidades, aumentar la velocidad del negocio y liderar los sectores en los que atiende a sus clientes.

### <a name="what-is-cortana-intelligence"></a>¿Qué es Cortana Intelligence?

Cortana Intelligence es un conjunto completamente administrado de macrodatos, análisis avanzados e inteligencia que transforma los datos en acciones inteligentes.  
Es un conjunto completo que reúne las tecnologías que se basan en años de investigación e innovación en Microsoft (incluido el análisis avanzado, el aprendizaje automático, el almacenamiento de macrodatos y el procesamiento en la nube) y:

* Permite recopilar, administrar y almacenar todos los datos que pueden crecer sin problemas y de forma rentable a lo largo del tiempo de una forma escalable y segura.
* Proporciona análisis accionables y simplificados con la tecnología de la nube que permiten predecir, establecer y automatizar la toma de decisiones para los problemas más exigentes.
* Habilita los sistemas inteligentes a través de agentes y servicios cognitivos que permiten ver, oír, interpretar y comprender el mundo que le rodea de formas más naturales y contextuales.

Con Cortana Intelligence, esperamos ayudar a nuestros clientes empresariales a desbloquear nuevas oportunidades, aumentar la velocidad de su negocio y ser líderes en sus sectores.

### <a name="what-is-the-direct-line-channel"></a>¿Qué es el canal Direct Line?

Direct Line es una API REST que le permite agregar el bot en su servicio, aplicación móvil o página web.

Puede escribir a un cliente para la API Direct Line en cualquier lenguaje. Basta con escribir el código para el [protocolo de Direct Line][DirectLineAPI], generar un secreto en la página de configuración de Direct Line y comunicarse con el bot desde donde se encuentra el código.

Direct Line es adecuado para:

* Aplicaciones móviles en iOS, Android y Windows Phone, entre otras
* Aplicaciones de escritorio en Windows, OSX y muchas más
* Las páginas web donde necesita más personalización de la que ofrece el [canal insertable de Chat en web][WebChat]
* Aplicaciones de servicio a servicio


## <a name="app-registration"></a>Registro de aplicaciones

### <a name="i-need-to-manually-create-my-app-registration-how-do-i-create-my-own-app-registration"></a>Tengo que crear manualmente el registro de la aplicación. ¿Cómo creo mi propio registro de aplicación?

Crear su propio registro de aplicación será necesario para situaciones como las siguientes:

- Creó el bot en el portal de Bot Framework (como https://dev.botframework.com/bots/new). 
- No puede realizar registros de aplicaciones en su organización y necesita otra entidad para crear el id. de aplicación para el bot que compila.
- En caso contrario, deberá crear manualmente su propio id. de aplicación (y contraseña).

Para crear su propio id. de aplicación, siga estos pasos.

1. Inicie sesión en la [cuenta de Azure](https://portal.azure.com). Si aún no tiene ninguna cuenta de Azure, puede [registrarse para obtener una cuenta gratuita](https://azure.microsoft.com/free/).
2. Vuelva a [la hoja de registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) y haga clic en **Nuevo registro** en la barra de acciones de la parte superior.

    ![nuevo registro](media/app-registration/new-registration.png)

3. Escriba un nombre para mostrar para el registro de aplicación en el campo *Nombre* y seleccione los tipos de cuenta compatibles. El nombre no tiene que coincidir con el id. de bot.

    > [!IMPORTANT]
    > En *Tipos de cuenta compatibles*, seleccione el botón de radio *Cuentas en cualquier directorio organizativo y cuentas Microsoft personales (por ejemplo, Skype, Xbox, Outlook.com)* . Si se selecciona cualquiera de las otras opciones, **se producirá un error en la creación del bot**.

    ![detalles de registro](media/app-registration/registration-details.png)

4. Haga clic en **Registrar**.

    Transcurridos unos instantes, el registro de aplicación recién creado debe abrir una hoja. Copie *Application (client) ID* (Id. de aplicación [cliente]) en la hoja de información general y péguelo en el campo de id. de aplicación.

    ![id. de aplicación](media/app-registration/app-id.png)

Si va a crear su bot a través del portal de Bot Framework, ya ha terminado la configuración del registro de aplicación; el secreto se generará automáticamente. 

Si crea el bot en Azure Portal, deberá generar un secreto para el registro de aplicación. 

1. Haga clic en **Certificates & secrets** (Certificados y secretos) en la columna de navegación izquierda de la hoja del registro de aplicación.
2. En esa hoja, haga clic en el botón **Nuevo secreto de cliente**. En el cuadro de diálogo que aparece, escriba una descripción opcional para el secreto y seleccione **Nunca** del grupo de botones de radio Expiración. 

    ![nuevo secreto](media/app-registration/new-secret.png)

3. Copie el valor del secreto de la tabla en *secretos de cliente* y péguelo en el campo *Contraseña* para la aplicación y haga clic en **Aceptar** en la parte inferior de esa hoja. A continuación, continúe con la creación del bot. 

    > [!NOTE]
    > El secreto solo estará visible mientras se encuentre en esta hoja y no podrá recuperarlo después de abandonar la página. Asegúrese de copiarlo en un lugar seguro.

    ![nuevo id. de aplicación](media/app-registration/create-app-id.png)


[DirectLineAPI]: https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts
[Support]: bot-service-resources-links-help.md
[WebChat]: bot-service-channel-connect-webchat.md

