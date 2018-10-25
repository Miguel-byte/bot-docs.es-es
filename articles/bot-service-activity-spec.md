---
title: Especificación de Bot Framework | Microsoft Docs
description: Especificación de Bot Framework
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/07/2018
ms.openlocfilehash: 06e2289dd0176364467d34846ffa7716483f6578
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000412"
---
# <a name="bot-framework----activity"></a>Bot Framework: actividad

## <a name="abstract"></a>Descripción breve

El esquema de la actividad de Bot Framework es una representación en el nivel de la aplicación de acciones de conversación realizadas por personas y software automatizado. El esquema incluye normas para transmitir texto, archivos multimedia y acciones sin contenido, como interacciones sociales e indicadores de escritura.

Este esquema se utiliza dentro del protocolo de Bot Framework y se implementa mediante los sistemas de chat de Microsoft y bots y clientes interoperables de varios orígenes.

## <a name="table-of-contents"></a>Tabla de contenido

1. [Introducción](#introduction)
2. [Estructura de actividad básica](#basic-activity-structure)
3. [Actividad de mensajes](#message-activity)
4. [Actividad de actualización de la relación de contacto](#contact-relation-update-activity)
5. [Actividad de actualización de la conversación](#conversation-update-activity)
6. [Actividad de finalización de la conversación](#end-of-conversation-activity)
7. [Actividad de evento](#event-activity)
8. [Actividad de invocación](#invoke-activity)
9. [Actividad de actualización de la instalación](#installation-update-activity)
10. [Actividad de eliminación de mensajes](#message-delete-activity)
11. [Actividad de actualización de mensajes](#message-update-activity)
12. [Actividad de reacción de mensajes](#message-reaction-activity)
13. [Actividad de escritura](#typing-activity)
14. [Tipos complejos](#complex-types)
15. [Referencias](#references)
16. [Apéndice I: Cambios](#appendix-i---changes)
17. [Apéndice II: Tipos de entidad que no son de IRI](#appendix-ii---non-iri-entity-types)
18. [Apéndice III: Protocolos que usan la actividad de invocación](#appendix-iii---protocols-using-the-invoke-activity)

## <a name="introduction"></a>Introducción

### <a name="overview"></a>Información general

El esquema de actividad de Bot Framework representa comportamientos de conversación de las personas y el software automatizado en las aplicaciones de chat, el correo electrónico y otros programas de interacción de texto. Cada objeto de actividad incluye un campo de tipo y representa una sola acción, que suele ser el envío de contenido de texto, pero también puede incluir archivos adjuntos multimedia y comportamientos sin contenido, como un botón "Me gusta" o un indicador de escritura.

En este documento se proporcionan significados para cada tipo de actividad y se describen los campos obligatorios y opcionales que se pueden incluir. También se definen los roles del servidor y del cliente y se proporcionan instrucciones sobre qué campos domina cada participante y cuáles pueden omitirse.

Hay tres roles importantes en esta especificación: los clientes, que envían y reciben actividades en nombre de los usuarios; los bots, que envían y reciben actividades y que suelen estar automatizados; y el canal, que almacena y reenvía las actividades entre los clientes y los bots.

Aunque esta especificación requiere que las actividades se transmitan entre roles, la naturaleza exacta de dicha transmisión no se describe aquí.

Asimismo, por motivos de espacio, las tarjetas interactivas visuales no se definen en esta especificación. En su lugar, consulte la especificación [Adaptive Cards](https://adaptivecards.io) [[11](#references)] (Tarjetas adaptables). Esta tarjeta, así como otros tipos de tarjetas sin definir, se puede incluir como datos adjuntos en las actividades de Bot Framework.

### <a name="requirements"></a>Requisitos

Las palabras clave "DEBE", "NO DEBE", "SE REQUIERE", "DEBERÁ", "NO DEBERÁ", "DEBERÍA", "NO DEBERÍA", "RECOMENDADO", "PUEDE" y "OPCIONAL" de este documento se deben interpretar como se describe en [RFC 2119](https://tools.ietf.org/html/rfc2119) [[1](#references)].

Una implementación no se considera compatible si no cumple uno o varios de los requisitos de nivel "DEBE" o "SE REQUIERE" de los protocolos que implementa. Una implementación que cumple en su totalidad el nivel "DEBE" o "SE REQUIERE" y todos los requisitos del nivel "DEBERÍA" de sus protocolos se la considera "incondicionalmente compatible"; una que cumpla todos los requisitos del nivel "DEBE", pero no todos los requisitos del nivel "DEBERÍA" de sus protocolos, se la considera "condicionalmente compatible".

### <a name="numbered-requirements"></a>Requisitos numerados

Las líneas que comienzan con marcadores con el formato `RXXXX` son requisitos específicos que se han diseñado para que se les haga referencia mediante el número cuando aparecen en una discusión fuera de este documento. Esto no significa que sean ni más ni menos importantes que las instrucciones normativas que no se incluyen en las líneas `RXXXX`.

`R1000`: los editores de esta especificación PUEDEN agregar nuevos requisitos para `RXXXX`. DEBEN buscar valores numéricos de `RXXXX` que conserven el flujo del documento.

`R1001`: los editores NO DEBEN renumerar los requisitos existentes de `RXXXX`.

`R1002`: los editores PUEDEN eliminar o revisar los requisitos de `RXXXX`. En caso de revisarlos, los editores DEBERÍAN conservar el valor `RXXXX` existente si el tema del requisito permanece intacto en gran medida.

`R1003`: los editores NO DEBERÍAN volver a usar los valores `RXXXX` retirados. Se PUEDE conservar una lista de los valores eliminados al final de este documento.

### <a name="terminology"></a>Terminología

activity
> Acción que expresa un bot, un canal o un cliente y que cumple el esquema de la actividad.

channel
> Software que envía y recibe actividades y las transforma en comportamientos de chat o aplicación y desde estos. Los canales son el almacén autoritativo de los datos de actividad.

bot
> Software que envía y recibe actividades y genera respuestas automatizadas, parcialmente automatizadas o totalmente manuales. Los bots tienen puntos de conexión que se registran con canales.

cliente
> Software que envía y recibe las actividades, normalmente en nombre de los usuarios. Los clientes no tienen puntos de conexión.

remitente
> Software que transmite una actividad.

receptor
> Software que acepta una actividad.

endpoint
> Ubicación direccionable mediante programación, donde un bot o un canal pueden recibir las actividades.

address
> Identificador o dirección donde se puede establecer contacto con un usuario o un bot.

campo
> Valor con nombre de una actividad o un objeto anidado.

### <a name="overall-organization"></a>Organización general

El objeto de actividad es una lista plana de pares nombre-valor, algunos de los cuales son objetos simples y algunos complejos (anidados). El objeto de actividad se suele expresar en formato JSON, pero también se puede proyectar en estructuras de datos en memoria en .NET o JavaScript.

El campo `type` de la actividad controla el significado de la actividad y los campos que contiene. En función del rol que tiene un participante (cliente, bot o canal), cada campo se considera obligatorio, opcional u omitido. Por ejemplo, el campo `id` cuyo control lo realiza el canal es obligatorio en algunas circunstancias, pero se omite si lo envía un bot.

Los campos que describen la identidad de la actividad y los participantes, como los campos `type` y `from`, se comparten entre todas las actividades. En muchos lenguajes de programación, resulta conveniente organizar estos campos en un tipo base principal del que deriven otros tipos de actividades más específicos.

Al almacenar o transmitir actividades, es posible que se dupliquen algunos campos del mecanismo de transporte. Por ejemplo, si una actividad se transmite a través de HTTP POST a una dirección URL que incluye el identificador de conversación, el receptor puede inferir su valor sin necesidad de que exista en el cuerpo de la actividad. En este documento solo se describen los requisitos abstractos de estos campos y se deja que el protocolo de control establezca si los valores se deben declarar de forma explícita o si se permiten valores implícitos o inferidos.

Cuando un bot o cliente envía una actividad a un canal, normalmente se trata de una solicitud para registrar la actividad. Cuando un canal envía una actividad a un bot o cliente, suele ser una notificación que indica que ya se ha registrado la actividad.

## <a name="basic-activity-structure"></a>Estructura de actividad básica

En esta sección se definen los requisitos para la estructura básica del objeto de actividad.

Entre los objetos de actividad se incluye una lista plana de pares nombre-valor, denominados campos. Los campos pueden ser tipos primitivos. JSON se usa como el formato de intercambio común y, aunque no todas las actividades se deben serializar en JSON en todo momento, deben ofrecer la posibilidad de serializarse. Esto permite que las implementaciones se basen en un conjunto sencillo de convenciones para controlar los campos de actividad conocidos y desconocidos.

`R2001`: las actividades DEBEN ser serializables para el formato JSON e incluir el cumplimiento de, por ejemplo, las restricciones de unicidad de campos.

`R2002`: los destinatarios PUEDEN permitir los nombres de campos con un uso de mayúsculas y minúsculas incorrecto, aunque no es necesario. Los destinatarios PUEDEN rechazar las actividades que no incluyan campos con un uso de mayúsculas y minúsculas correcto.

`R2003`: los destinatarios PUEDEN rechazar las actividades que contienen valores de campos cuyos tipos no coinciden con los tipos de valor descritos en esta especificación.

`R2004`: a menos que se indique lo contrario, los remitentes NO DEBERÍAN incluir valores de cadena vacíos para los campos de cadena.

`R2005`: a menos que se indique lo contrario, los remitentes PUEDEN incluir campos adicionales dentro de la actividad u objetos complejos anidados. Los destinatarios DEBEN aceptar los campos que no comprenden.

`R2006`: los destinatarios DEBERÍAN aceptar los eventos de tipos que no comprenden.

### <a name="type"></a>Escriba

El campo `type` controla el significado de cada actividad, que son cadenas cortas por convención (por ejemplo, "`message`"). Los remitentes pueden definir sus propios tipos de capa de aplicación, aunque se le anima a que elijan los valores que no tengan probabilidades de colidir con valores futuros bien definidos. Si los remitentes utilizan URI como valores de tipo, no DEBERÍAN implementar comparaciones de escala del URI para establecer la equivalencia.

`R2010`: las actividades DEBEN incluir un campo `type` con el tipo de valor de cadena.

`R2011`: dos valores `type` son equivalentes solo si son idénticos ordinalmente.

`R2012`: un remitente PUEDE generar valores `type` de actividad que no se definan en este documento.

`R2013`: un canal DEBERÍA rechazar las actividades que sean de un tipo que no comprenda.

`R2014`: un cliente o bot DEBERÍA ignorar las actividades que sean de un tipo que no comprenda.

### <a name="channel-id"></a>Channel ID

El campo `channelId` establece el canal y el almacén autoritativo de la actividad. El valor del campo `channelId` es de tipo cadena.

`R2020`: las actividades DEBEN incluir un campo `channelId` con el tipo de valor de cadena.

`R2021`: dos valores `channelId` son equivalentes solo si son idénticos ordinalmente.

`R2022`: un canal PUEDE ignorar o rechazar cualquier actividad que reciba sin un valor `channelId` esperado.

### <a name="id"></a>ID

El campo `id` establece la identidad de la actividad una vez que se registra en el canal. Las actividades en curso que aún no se han registrado no tienen identidades. No se asignan identidades a todas las actividades (por ejemplo, no se puede asignar nunca `id` a una [actividad de escritura](#typing-activity)). El valor del campo `id` es de tipo cadena.

`R2030`: los canales DEBERÍAN incluir un campo `id` si está disponible para esa actividad.

`R2031`: los clientes y los bots NO DEBERÍAN incluir un campo `id` en las actividades que generan.

Para facilitar la implementación, se debe suponer que los otros participantes no tienen conocimientos avanzados sobre los identificadores de actividad y que solo usarán la comparación ordinal para establecer la equivalencia.

Por ejemplo, un canal puede usar GUID con codificación hexadecimal para cada identificador de actividad. Aunque los GUID codificados en mayúsculas son lógicamente equivalentes a los GUID codificados en minúsculas, los remitentes NO DEBERÍAN usar estas codificaciones alternativas cuando sea posible. La versión normalizada de cada identificador se establece mediante el almacén autoritativo: el canal.

`R2032`: al generar valores `id`, los remitentes DEBERÍAN elegir valores cuya equivalencia se pueda establecer mediante la comparación ordinal. Sin embargo, los remitentes y los receptores PUEDEN permitir la equivalencia lógica de dos valores que no son equivalentes ordinalmente si tienen un conocimiento avanzado de las circunstancias.

El campo `id` está diseñado para permitir la desduplicación, pero esto resulta prohibitivo en la mayoría de las aplicaciones.

`R2033`: los destinatarios PUEDEN desduplicar actividades con el identificador, sin embargo, los remitentes NO DEBERÍAN depender de los destinatarios al realizar esta desduplicación.

### <a name="timestamp"></a>Timestamp

El campo `timestamp` registra la hora UTC exacta en que se produjo la actividad. Debido a la naturaleza distribuida de los sistemas informáticos, la hora importante es cuando el canal (el almacén autoritativo) registra la actividad. La hora en que un cliente o un bot inicia una actividad se puede transmitir por separado en el campo `localTimestamp`. El valor del campo `timestamp` es un datetime codificado por la [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) [[2](#references)] dentro de una cadena.

`R2040`: los canales DEBERÍAN incluir un campo `timestamp` si está disponible para esa actividad.

`R2041`: los clientes y los bots NO DEBERÍAN incluir ningún campo `timestamp` en las actividades que generan.

`R2042`: los clientes y los bots NO DEBERÍAN usar `timestamp` para rechazar las actividades, ya que pueden aparecer fuera de servicio. Sin embargo, PUEDEN usar `timestamp` para ordenar las actividades dentro de una interfaz de usuario o para el procesamiento de bajada.

`R2043`: los remitentes siempre DEBERÍAN codificar el valor de los campos `timestamp` como hora UTC y DEBERÍAN incluir siempre una Z como una marca de hora UTC explícita dentro del valor.

### <a name="local-timestamp"></a>Local timestamp

El campo `localTimestamp` expresa el desplazamiento de datetime y la zona horaria donde se generó la actividad. Esto puede ser diferente de la hora UTC `timestamp` en la que se registró la actividad. El valor del campo `localTimestamp` es un datetime codificado por la ISO 8601 [[3](#references)] dentro de una cadena.

`R2050`: los bots y los clientes PUEDEN incluir el campo `localTimestamp` en sus actividades. DEBERÍAN mostrar explícitamente el desplazamiento de zona horaria en el valor codificado.

`R2051`: los canales DEBERÍAN conservar `localTimestamp` al reenviar las actividades de un remitente a los destinatarios.

### <a name="from"></a>De

En el campo `from` se describe qué cliente, bot o canal generó una actividad. El valor del campo `from` es un objeto complejo de tipo [cuenta de canal](#channel-account).

El campo `from.id` identifica quién generó una actividad. Normalmente, se trata de otro usuario o bot del sistema. En algunos casos, el campo `from` identifica el propio canal.

`R2060`: los canales DEBEN incluir los campos `from` y `from.id` cuando se genera una actividad.

`R2061`: los bots y los clientes DEBERÍAN incluir los campos `from` y `from.id` cuando se genera una actividad. Un canal PUEDE rechazar una actividad porque le faltan los campos `from` y `from.id`.

El campo `from.name` es opcional y representa el nombre para mostrar de la cuenta del canal. Los canales DEBERÍAN incluir este valor para que los clientes y los bots puedan rellenar sus sistemas de back-end y las interfaces de usuario. Los bots y los clientes NO DEBERÍAN enviar este valor a los canales que tengan un registro central de este almacén, pero lo PUEDEN enviar a los canales que rellenen el valor en todas las actividades (por ejemplo, un canal de correo electrónico).

`R2062`: los canales DEBERÍAN incluir el campo `from.name` si el campo `from` existe y `from.name` está disponible.

`R2063`: los bots y los clientes NO DEBERÍAN incluir el campo `from.name` a menos que sea semánticamente valioso dentro del canal.

### <a name="recipient"></a>Recipient

El campo `recipient` describe qué cliente o bot está recibiendo esta actividad. Este campo solo es significativo cuando se transmite una actividad a un destinatario exactamente; no es significativo cuando se difunde a varios destinatarios (como sucede cuando una actividad se envía a un canal). El propósito del campo es permitir que el destinatario se identifique. Esto resulta útil cuando un cliente o un bot tiene más de una identidad dentro del canal. El valor del campo `recipient` es un objeto complejo de tipo [cuenta de canal](#channel-account).

`R2070`: los canales DEBEN incluir los campos `recipient` y `recipient.id` cuando transmiten una actividad a un solo destinatario.

`R2071`: los bots y los clientes NO DEBERÍAN incluir el campo `recipient` cuando se genera una actividad.

El campo `recipient.name` es opcional y representa el nombre para mostrar de la cuenta del canal. Los canales DEBERÍAN incluir este valor para que los clientes y los bots puedan rellenar sus sistemas de back-end y las interfaces de usuario.

`R2072`: los canales DEBERÍAN incluir el campo `recipient.name` si el campo `recipient` existe y `recipient.name` está disponible.

### <a name="conversation"></a>Conversation

El campo `conversation` describe la conversación en la que existe la actividad. El valor del campo `conversation` es un objeto complejo de tipo [cuenta de conversación](#conversation-account).

`R2080`: los canales, los bots y los clientes DEBEN incluir los campos `conversation` y `conversation.id` cuando se genera una actividad.

El campo `conversation.name` es opcional y representa el nombre para mostrar de la conversación si existe y está disponible.

`R2081`: los canales DEBERÍAN incluir los campos `conversation.name` y `conversation.isGroup` si están disponibles.

`R2082`: los bots y los clientes NO DEBERÍAN incluir el campo `conversation.name` a menos que sea semánticamente valioso dentro del canal.

`R2083`: los bots y los clientes NO DEBERÍAN incluir el campo `conversation.isGroup` en las actividades que generan.

### <a name="reply-to-id"></a>Reply to ID

El campo `replyToId` identifica la actividad previa para la que la actividad actual es una respuesta. Este campo permite que la conversación encadenada y el anidamiento de comentarios se transmitan entre los participantes. `replyToId` solo es válido en la conversación actual. (Consulte [relatesTo](#relates-to) para obtener referencias a otras conversaciones). El valor del campo `replyToId` es una cadena.

`R2090`: los remitentes DEBERÍAN incluir `replyToId` en una actividad cuando se trata de una respuesta a otra actividad.

`R2091`: un canal PUEDE rechazar una actividad si su valor `replyToId` no hace referencia a una actividad válida dentro de la conversación.

`R2092`: los bots y los clientes PUEDEN omitir `replyToId` si saben que el canal no usa el campo, aunque la actividad que se envíe sea una respuesta a otra actividad.

### <a name="entities"></a>Entidades

El campo `entities` contiene una lista plana de objetos de metadatos que pertenecen a esta actividad. A diferencia de los datos adjuntos (consulte el campo [attachments](#attachments)), las entidades no necesariamente se manifiestan como elementos de contenido interactivo por el usuario y están diseñadas para omitirse si no se entienden. Los remitentes pueden incluir entidades que consideren que pueden resultar útiles para un destinatario, aunque no estén seguros de si el destinatario podrá aceptarlas. El valor de cada elemento de lista `entities` es un objeto complejo de tipo [entidad](#entity).

`R2100`: los remitentes DEBERÍAN omitir el campo `entities` si no contiene ningún elemento.

`R2101`: los remitentes PUEDEN enviar varias entidades del mismo tipo, siempre que las entidades tengan un significado distinto.

`R2102`: los remitentes NO DEBEN incluir dos o más entidades con contenido y tipos idénticos.

`R2103`: los remitentes y los destinatarios NO DEBERÍAN confiar en el orden específico de las entidades incluidas en una actividad.

### <a name="channel-data"></a>Channel data

Los datos de extensibilidad del esquema de actividad se organizan principalmente en el campo `channelData`. Esto simplifica el establecimiento de los SDK que implementan el protocolo. El formato del objeto `channelData` lo define el canal de envío o recepción de la actividad.

`R2200`: los canales NO DEBERÍAN definir los formatos `channelData` que son primitivos de JSON (por ejemplo, cadenas o enteros). En su lugar, DEBERÍAN definir `channelData` como un tipo complejo o dejarlo sin definir.

`R2201`: si el formato `channelData` no está definido para el canal actual, los destinatarios DEBERÍAN omitir el contenido de `channelData`.

### <a name="service-url"></a>URL de servicio

Las actividades se envían con frecuencia de forma asincrónica, con conexiones independientes de transporte para el envío y la recepción de tráfico. Los canales usan el campo `serviceUrl` para denotar la dirección URL donde se pueden enviar las respuestas a la actividad actual. El valor del campo `serviceUrl` es de tipo cadena.

`R2300`: los canales DEBEN incluir el campo `serviceUrl` en todas las actividades que envíen a los bots.

`R2301`: los canales NO DEBERÍAN incluir el campo `serviceUrl` para los clientes que demuestren que ya conocen el punto de conexión del canal.

`R2302`: los bots y los clientes NO DEBERÍAN rellenar el campo `serviceUrl` en las actividades que generan.

`R2302`: los canales DEBEN ignorar el valor de `serviceUrl` en las actividades enviadas por los bots y los clientes.

`R2304`: los canales DEBERÍAN usar valores estables para el campo `serviceUrl` porque los bots pueden hacer que persistan durante períodos largos.

## <a name="message-activity"></a>Actividad de mensajes

Las actividades de mensajes representan contenido diseñado para mostrarse dentro de una interfaz de conversación. Las actividades de mensajes pueden contener texto, voz, tarjetas interactivas y datos adjuntos binarios o desconocidos. Normalmente, los canales requieren como máximo uno de estos elementos para que la actividad de mensajes tenga un formato correcto.

Las actividades de mensajes se identifican mediante un valor `type` de `message`.

### <a name="text"></a>Texto

El campo `text` incluye el contenido de texto, que puede estar en formato de Markdown, XML o como texto sin formato. El campo [`textFormat`](#text-format) controla el formato y no aplica ninguno si no se especifica o es ambiguo. El valor del campo `text` es de tipo cadena.

`R3000`: el campo `text` PUEDE contener una cadena vacía para indicar el envío de texto sin contenido.

`R3001`: los canales DEBERÍAN controlar el texto con formato `markdown` de una forma que se degrade correctamente para ese canal.

### <a name="text-format"></a>Formato de texto

El campo `textFormat` indica si el campo [`text`](#text) debe interpretarse como [Markdown](https://daringfireball.net/projects/markdown/) [[4](#references)], texto sin formato o XML. El valor del campo `textFormat` es de tipo cadena y tiene los valores definidos de `markdown`, `plain` y `xml`. El valor predeterminado es `plain`. Este campo no está diseñado para ampliarse con valores arbitrarios.

El campo `textFormat` controla los campos adicionales de los datos adjuntos, etc. Esta relación se describe en dichos campos en otra sección de este documento.

`R3010`: si un remitente incluye el campo `textFormat`, solo DEBERÍA enviar los valores definidos.

`R3011`: los remitentes DEBERÍAN omitir `textFormat` si el valor es `plain`.

`R3012`: los destinatarios DEBERÍAN interpretar los valores no definidos como `plain`.

`R3013`: los bots y los clientes NO DEBERÍAN enviar el valor `xml`, a menos que tengan conocimiento previo de que el canal lo admite y de las características del dialecto XML compatible.

`R3014`: los canales NO DEBERÍAN enviar contenido `markdown` o `xml` a los bots.

`R3015`: los canales DEBERÍAN aceptar los valores `textformat` de al menos `plain` y `markdown`.

`R3016`: los canales PUEDEN rechazar `textformat` del valor `xml`.

### <a name="locale"></a>Configuración regional

El campo `locale` transmite el código de idioma del campo [`text`](#text). El valor del campo `locale` es un código [ISO 639](https://www.iso.org/iso-639-language-codes.html) [[5](#references)] de una cadena.

`R3020`: los destinatarios DEBERÍAN tratar los valores que faltan y desconocidos del campo `locale` como desconocidos.

`R3021`: los destinatarios NO DEBERÍAN rechazar las actividades cuya configuración regional sea desconocida.

### <a name="speak"></a>Leer

El campo `speak` indica cómo la actividad debería ser oral a través de un sistema de texto a voz. El campo solo se usa para personalizar la representación de voz cuando el valor predeterminado se considera inadecuado. Reemplaza la síntesis de voz para cualquier contenido de la actividad, incluido el texto, los datos adjuntos y los resúmenes. El valor del campo `speak` tiene codificación [SSML](https://www.w3.org/TR/speech-synthesis/) [[6](#references)] en una cadena. El texto desencapsulado se permite y se actualiza automáticamente para la reconstrucción de SSML.

`R3030`: el campo `speak` PUEDE contener una cadena vacía para indicar que no se debe generar voz.

`R3031`: los destinatarios que no puedan generar voz DEBERÍAN omitir el campo `speak`.

`R3032`: si no se encuentra ningún elemento SSML raíz,los destinatarios DEBEN considerar el texto incluido como contenido interno de una etiqueta `<speak>` SSML, precedido por un [prólogo XML](https://www.w3.org/TR/xml/#sec-prolog-dtd) válido [[7](#references)] y, en caso contrario, actualizado para que sea un documento SSML válido.

`R3033`: los destinatarios NO DEBERÍAN usar la resolución de esquema o DTD XML para incluir recursos remotos de fuera del fragmento XML transmitido.

`R3034`: los canales NO DEBERÍAN enviar el campo `speak` a los bots.

### <a name="input-hint"></a>Input hint

El campo `inputHint` indica si el generador de la actividad prevé o no una respuesta. Este campo se utiliza fundamentalmente en canales que tienen interfaces de usuario modales, pero normalmente no se utiliza en canales con fuentes de chat continuas. El valor del campo `inputHint` es de tipo cadena y tiene los valores definidos de `accepting`, `expecting` y `ignoring`. El valor predeterminado es `accepting`.

`R3040`: si un remitente incluye el campo `inputHint`, solo DEBERÍA enviar los valores definidos.

`R3041`: determina si, al enviar una actividad a un canal en el que se usa `inputHint`, los bots DEBERÍAN incluir el campo, incluso cuando el valor es `accepting`.

`R3042`: los destinatarios DEBERÍAN interpretar los valores no definidos como `accepting`.

### <a name="attachments"></a>Datos adjuntos

El campo `attachments` contiene una lista plana de los objetos que se mostrarán como parte de esta actividad. El valor de cada elemento de lista `attachments` es un objeto complejo de tipo [archivo adjunto](#attachment).

`R3050`: los remitentes DEBERÍAN omitir el campo `attachments` si no contiene ningún elemento.

`R3051`: los remitentes PUEDEN enviar varias entidades del mismo tipo.

`R3052`: los destinatarios PUEDEN tratar los datos adjuntos de los tipos desconocidos como documentos descargables.

`R3053`: los destinatarios DEBERÍAN conservar la ordenación de los datos adjuntos al procesar el contenido, excepto cuando las limitaciones de representación fuercen cambios, por ejemplo, la agrupación de documentos después de imágenes.

### <a name="attachment-layout"></a>Attachment layout

El campo `attachmentLayout` indica a los representadores de la interfaz de usuario cómo se debe presentar el contenido incluido en el campo [`attachments`](#attachments). El valor del campo `attachmentLayout` es de tipo cadena y tiene los valores definidos de `list` y `carousel`. El valor predeterminado es `list`.

`R3060`: si un remitente incluye el campo `attachmentLayout`, solo DEBERÍA enviar los valores definidos.

`R3061`: los destinatarios DEBERÍAN interpretar los valores no definidos como `list`.

### <a name="summary"></a>Resumen

El campo `summary` contiene texto que se usa para reemplazar [`attachments`](#attachments) en canales que no lo admiten. El valor del campo `summary` es de tipo cadena.

`R3070`: los destinatarios DEBERÍAN considerar que el campo `summary` siguiera lógicamente el campo `text`.

`R3071`: los canales NO DEBERÍAN enviar el campo `summary` a los bots.

`R3072`: los canales que pueden procesar todos los datos adjuntos de una actividad DEBERÍAN ignorar el campo `summary`.

### <a name="suggested-actions"></a>Suggested actions

El campo `suggestedActions` contiene una carga de las acciones interactivas que pueden mostrarse al usuario. La compatibilidad con `suggestedActions` y su manifestación depende en gran medida del canal. El valor del campo `suggestedActions` es un objeto complejo de tipo [acciones sugeridas](#suggested-actions-2).

### <a name="value"></a>Valor

El campo `value` contiene una carga de programación específica de la actividad que se envía. Su significado y el formato se definen en las otras secciones de este documento que describen su uso.

`R3080`: los remitentes NO DEBERÍAN incluir los campos `value` de tipos primitivos (p. ej., cadena o entero). Los campos `value` DEBERÍAN ser tipos complejos u omitirse.

### <a name="expiration"></a>Expiration

El campo `expiration` contiene una hora en que la actividad debería considerarse "expirada" y no debería presentarse al destinatario. El valor del campo `expiration` es un datetime codificado por la [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) [[2](#references)] dentro de una cadena.

`R3090`: los remitentes siempre DEBERÍAN codificar el valor de los campos `expiration` como hora UTC y DEBERÍAN incluir siempre una Z como una marca de hora UTC explícita dentro del valor.

### <a name="importance"></a>importancia

El campo `importance` contiene un conjunto enumerado de valores para indicar al destinatario la importancia relativa de la actividad.  Depende del destinatario asignar estas sugerencias de importancia para la experiencia del usuario. El valor del campo `importance` es de tipo cadena y tiene los valores definidos de `low`, `normal` y `high`. El valor predeterminado es `normal`.

`R3100`: si un remitente incluye el campo `importance`, solo DEBERÍA enviar los valores definidos.

`R3101`: los destinatarios DEBERÍAN interpretar los valores no definidos como `normal`.

### <a name="deliverymode"></a>DeliveryMode

El campo `deliveryMode` contiene un conjunto enumerado de valores para señalar al destinatario las rutas de acceso de entrega alternativas para la actividad.  El valor del campo `DeliveryMode` es de tipo cadena y tiene los valores definidos de `normal` y `notification`. El valor predeterminado es `normal`.

`R3110`: si un remitente incluye el campo `deliveryMode`, solo DEBERÍA enviar los valores definidos.

`R3111`: los destinatarios DEBERÍAN interpretar los valores no definidos como `normal`.

## <a name="contact-relation-update-activity"></a>Actividad de actualización de la relación de contacto

Las actividades de actualización de la relación de contacto indican un cambio en la relación entre el destinatario y un usuario dentro del canal. Las actividades de actualización de la relación de contacto generalmente no contienen ningún contenido generado por el usuario. La actualización de la relación descrita por una actividad de actualización de la relación de contacto existe entre el usuario del campo `from` (a menudo, pero no siempre, el usuario que inicia la actualización) y el usuario o el bot del campo `recipient`.

Las actividades de actualización de la relación de contacto se identifican mediante un valor `type` de `contactRelationUpdate`.

### <a name="action"></a>.

El campo `action` describe el significado de la actividad de actualización de la relación de contacto. El valor del campo `action` es una cadena. Solo se definen los valores de `add` y `remove`, que indican una relación entre los usuarios o bots en los campos `from` y `recipient`.

## <a name="conversation-update-activity"></a>Actividad de actualización de la conversación

Las actividades de actualización de la conversación describen un cambio en los miembros de una conversación, la descripción, la existencia u otro. Las actividades de actualización de la conversación generalmente no contienen ningún contenido generado por el usuario. La conversación que se actualiza se describe en el campo `conversation`.

Las actividades de actualización de la conversación se identifican mediante un valor `type` de `conversationUpdate`.

`R4100`: los remitentes PUEDEN incluir cero o varios campos `membersAdded`, `membersRemoved`, `topicName` y `historyDisclosed` en una actividad de actualización de la conversación.

`R4101`: cada `channelAccount` (identificado por el campo `id`) DEBERÍA aparecer como máximo una vez en los campos `membersAdded` y `membersRemoved`. Un identificador NO DEBERÍA aparecer en ambos campos. Un identificador NO DEBERÍA estar duplicado en los campos.

`R4102`: los canales NO DEBERÍAN usar las actividades de actualización de la conversación para indicar los cambios realizados en los campos de una cuenta de canal (por ejemplo, `name`) si la cuenta de canal no se agregó a la conversación o se quitó de esta.

`R4103`: los canales NO DEBERÍAN enviar los campos `topicName` o `historyDisclosed` si la actividad no indica un cambio en el valor de uno de los campos.

### <a name="members-added"></a>Members added

El campo `membersAdded` contiene una lista de los participantes del canal (bots o usuarios) agregados a la conversación. El valor del campo `membersAdded` es una matriz de tipo [`channelAccount`](#channel-account).

### <a name="members-removed"></a>Members removed

El campo `membersRemoved` contiene una lista de los participantes de canal (bots o usuarios) quitados de la conversación. El valor del campo `membersRemoved` es una matriz de tipo [`channelAccount`](#channel-account).

### <a name="topic-name"></a>Nombre del tema

El campo `topicName` contiene la descripción o el tema del texto de la conversación. El valor del campo `topicName` es de tipo cadena.

### <a name="history-disclosed"></a>History disclosed

El campo `historyDisclosed` está en desuso.

`R4110`: los remitentes NO DEBERÍAN incluir el campo `historyDisclosed`.

## <a name="end-of-conversation-activity"></a>Actividad de finalización de la conversación

Las actividades de finalización de la conversación indican el final de una conversación desde la perspectiva del destinatario. Esto puede ser porque la conversación ha finalizado completamente o porque se ha quitado el destinatario de la conversación de forma que no se puede distinguir que finalizó. La conversación que finaliza se describe en el campo `conversation`.

Las actividades de finalización de la conversación se identifican mediante un valor `type` de `endOfConversation`.

Los campos `code` y `text` son opcionales.

### <a name="code"></a>Código

El campo `code` contiene un valor de programación que describe por qué y cómo la conversación se terminó. El valor del campo `code` es de tipo cadena y su significado lo define el canal que envía la actividad.

### <a name="text"></a>Texto

El campo `text` incluye el contenido de texto opcional que se transmitirá a un usuario. El valor del campo `text` es de tipo cadena y su formato es de texto sin formato.

## <a name="event-activity"></a>Actividad de evento

Las actividades de evento transmiten información de programación de un cliente o un canal a un bot. El significado de una actividad de evento se define en el campo `name`, que es significativo en el ámbito de un canal. Las actividades de evento están diseñadas para transportar información interactiva (por ejemplo, clics de botón) e información no interactiva (por ejemplo, una notificación de un cliente de la actualización automática de un modelo de voz insertado).

Las actividades de evento son el homólogo asincrónico de las [actividades de invocación](#invoke-activity). A diferencia de la invocación, el evento está diseñado para extenderse mediante las extensiones de aplicación de cliente.

Las actividades de evento se identifican mediante un valor `type` de `event` y valores específicos del campo `name`.

`R5000`: los canales PUEDEN permitir mensajes de eventos definidos por la aplicación entre los clientes y los bots, si los clientes permiten la personalización de la aplicación.

### <a name="name"></a>NOMBRE

El campo `name` controla el significado del evento y el esquema del campo `value`. El valor del campo `name` es de tipo cadena.

`R5001`: las actividades de evento DEBEN contener un campo `name`.

`R5002`: los destinatarios DEBEN ignorar las actividades de evento con los campos `name` que no comprenden.

### <a name="value"></a>Valor

El campo `value` contiene parámetros específicos de este evento, tal como se define en el nombre del evento. El valor del campo `value` es de tipo complejo.

`R5100`: PUEDE que el campo `value` falte o esté vacío, si lo define el nombre del evento.

`R5101`: las extensiones de la actividad de evento NO DEBERÍAN requerir que los destinatarios usen cualquier información que no sea los campos de actividad `type` y `name` para comprender el esquema del campo `value`.

### <a name="relates-to"></a>Relates to

El campo `relatesTo` hace referencia a otra conversación y, opcionalmente, una actividad específica dentro de esa actividad. El valor del campo `relatesTo` es un objeto complejo de tipo de referencia de la conversación.

`R5200`: `relatesTo` NO DEBERÍA hacer referencia a una actividad de la conversación identificada por el campo `conversation`.

## <a name="invoke-activity"></a>Actividad de invocación

Las actividades de invocación transmiten información de programación de un cliente o un canal a un bot y tienen una carga de devolución correspondiente para su uso en el canal. El significado de una actividad de invocación se define en el campo `name`, que es significativo en el ámbito de un canal. 

Las actividades de invocación son el homólogo sincrónico de las [actividades de evento](#event-activity). Las actividades de evento están diseñadas para ser extensibles. Las actividades de invocación solo difieren de estas en su capacidad para devolver cargas de respuesta al canal. Dado que el canal debe decidir dónde y cómo procesar estas cargas de respuesta, la invocación solo es útil en casos en los que se ha agregado al canal compatibilidad explícita para cada nombre de invocación. Por lo tanto, la invocación no está diseñada para ser un mecanismo de extensibilidad de la aplicación genérica.

Las actividades de invocación se identifican mediante un valor `type` de `invoke` y valores específicos del campo `name`.

La lista de actividades de invocación definidas se incluye en el [Apéndice III](#appendix-iii---protocols-using-the-invoke-activity).

`R5301`: los canales NO DEBERÍAN permitir mensajes de invocación definidos por la aplicación entre clientes y bots.

### <a name="name"></a>NOMBRE

El campo `name` controla el significado de la invocación y el esquema del campo `value`. El valor del campo `name` es de tipo cadena.

`R5401`: las actividades de invocación DEBEN contener un campo `name`.

`R5402`: los destinatarios DEBEN ignorar las actividades de evento con los campos `name` que no comprenden.

### <a name="value"></a>Valor

El campo `value` contiene parámetros específicos de este evento, tal como se define en el nombre del evento. El valor del campo `value` es de tipo complejo.

`R5500`: PUEDE que el campo `value` falte o esté vacío, si lo define el nombre del evento.

`R5501`: las extensiones de la actividad de evento NO DEBERÍAN requerir que los destinatarios usen cualquier información que no sea los campos de actividad `type` y `name` para comprender el esquema del campo `value`.

### <a name="relates-to"></a>Relates to

El campo `relatesTo` hace referencia a otra conversación y, opcionalmente, una actividad específica dentro de esa actividad. El valor del campo `relatesTo` es un objeto complejo de tipo de [referencia de la conversación](#conversation-reference).

`R5600`: `relatesTo` NO DEBERÍA hacer referencia a una actividad de la conversación que identifique el campo `conversation`.

## <a name="installation-update-activity"></a>Actividad de actualización de la instalación

Las actividades de actualización de la instalación representan una instalación o desinstalación de un bot dentro de una unidad organizativa (por ejemplo, un inquilino de cliente o "equipo") de un canal. Por lo general, las actividades de actualización de la instalación no representan la adición o eliminación de un canal.

Las actividades de actualización de la instalación se identifican mediante un valor `type` de `installationUpdate`.

`R5700`: los canales PUEDEN enviar actividades de instalación cuando se agrega o se quita un bot de un inquilino, equipo u otra unidad organizativa del canal.

`R5701`: los canales NO DEBERÍAN enviar actividades de instalación cuando el bot se instala en un canal o se quita de él.

### <a name="action"></a>.

El campo `action` describe el significado de la actividad de actualización de la instalación. El valor del campo `action` es una cadena. Solo se definen los valores de `add` y `remove`.

## <a name="message-delete-activity"></a>Actividad de eliminación de mensajes

Las actividades de eliminación de mensajes representan la eliminación de una actividad de mensajes existente en una conversación. Se hace referencia a la actividad eliminada mediante los campos `id` y `conversation` de la actividad.

Las actividades de eliminación de mensajes se identifican mediante un valor `type` de `messageDelete`.

`R5800`: los canales PUEDEN optar por enviar las actividades de eliminación de mensajes para todas las eliminaciones de una conversación, un subconjunto de las eliminaciones de una conversación (por ejemplo, solo eliminaciones de determinados usuarios) o ninguna actividad de la conversación.

`R5801`: los canales NO DEBERÍAN enviar las actividades de eliminación de mensajes para las conversaciones o las actividades que el bot no observó.

`R5802`: si un bot desencadena una eliminación, el canal NO DEBERÍA enviar una actividad de eliminación de mensajes al bot.

`R5803`: los canales NO DEBERÍAN enviar actividades de eliminación de mensajes correspondientes a actividades cuyo tipo no sea `message`.

## <a name="message-update-activity"></a>Actividad de actualización de mensajes

Las actividades de actualización de mensajes representan la actualización de una actividad de mensajes existente en una conversación. Se hace referencia a la actividad actualizada mediante los campos `id` y `conversation` de la actividad y la actividad de actualización de mensajes contiene todos los campos de la actividad de mensajes revisada.

Las actividades de actualización de mensajes se identifican mediante un valor `type` de `messageUpdate`.

`R5900`: los canales PUEDEN optar por enviar las actividades de actualización de mensajes para todas las actualizaciones de una conversación, un subconjunto de las actualizaciones de una conversación (por ejemplo, solo actualizaciones de determinados usuarios) o ninguna actividad de la conversación.

`R5901`: si un bot desencadena una actualización, el canal NO DEBERÍA enviar una actividad de actualización de mensajes al bot.

`R5902`: los canales NO DEBERÍAN enviar actividades de actualización de mensajes correspondientes a actividades cuyo tipo no sea `message`.

## <a name="message-reaction-activity"></a>Actividad de reacción de mensajes

Las actividades de reacción de mensajes representan la interacción social en una actividad de mensajes existente de una conversación. Se hace referencia a la actividad original mediante los campos `id` y `conversation` de la actividad. El campo `from` representa el origen de la reacción (es decir, el usuario que reacciona al mensaje).

Las actividades de reacción de mensajes se identifican mediante un valor `type` de `messageReaction`.

### <a name="reactions-added"></a>Reactions added

El campo `reactionsAdded` contiene una lista de las reacciones agregadas a esta actividad. El valor del campo `reactionsAdded` es una matriz de tipo [`messageReaction`](#message-reaction).

### <a name="reactions-removed"></a>Reactions removed

El campo `reactionsRemoved` contiene una lista de las reacciones quitadas de esta actividad. El valor del campo `reactionsRemoved` es una matriz de tipo [`messageReaction`](#message-reaction).

## <a name="typing-activity"></a>Actividad de escritura

Las actividades de escritura representan la entrada continua de un usuario o un bot. Esta actividad se envía a menudo cuando un usuario pulsa teclas, aunque también la usan los bots para indicar que están "pensando" y también se podría usar, por ejemplo, para indicar que se está recopilando audio de los usuarios.

Las actividades de escritura están diseñadas para conservarse en las interfaces de usuario durante tres segundos.

Las actividades de escritura se identifican mediante un valor `type` de `typing`.

`R6000`: si es posible, los clientes DEBERÍAN mostrar indicadores de escritura durante tres segundos tras la recepción de una actividad de escritura.

`R6001`: a menos que se conozca de otro modo para el canal, los remitentes NO DEBERÍAN enviar actividades de escritura con una frecuencia superior a tres segundos. (Los remitentes PUEDEN enviar actividades de escritura cada dos segundos para evitar que aparezcan espacios).

`R6002`: si un canal asigna un valor de [`id`](#id) a una actividad de escritura, PUEDE permitir que los bots y los clientes eliminen la actividad de escritura antes de su expiración.

`R6003`: si es posible, los canales DEBERÍAN enviar las actividades de escritura a los bots.

## <a name="complex-types"></a>Tipos complejos

En esta sección se definen los tipos complejos utilizados en el esquema de actividad, que se han descrito anteriormente.

### <a name="attachment"></a>Datos adjuntos

Los datos adjuntos son el contenido que se incluye en una [actividad de mensajes](#message-activity): tarjetas, documentos binarios y otro contenido interactivo. Están diseñados para mostrarse junto con contenido de texto. El contenido se puede enviar a través del URI de datos de la dirección URL en el campo `contentUrl` o en línea en el campo `content`.

`R7100`: los remitentes NO DEBERÍAN incluir los campos `content` y `contentUrl` en un único archivo adjunto.

#### <a name="content-type"></a>Tipo de contenido

El campo `contentType` describe el [tipo elemento multimedia MIME](https://www.iana.org/assignments/media-types/media-types.xhtml) [[8](#references)] del contenido de los datos adjuntos. El valor del campo `contentType` es de tipo cadena.

#### <a name="content"></a>Contenido

Cuando existe, el campo `content` contiene un archivo adjunto de objeto JSON estructurado. El valor del campo `content` es un tipo complejo que define el campo `contentType`.

`R7110`: los remitentes NO DEBERÍAN incluir tipos primitivos JSON en el campo `content` de un archivo adjunto.

#### <a name="content-url"></a>Content URL

Cuando existe, el campo `contentUrl` contiene una dirección URL al contenido de los datos adjuntos. Los URI de datos, según se definen en [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)], normalmente son compatibles con los canales. El valor del campo `contentUrl` es de tipo cadena.

`R7120`: los destinatarios DEBERÍAN aceptar las direcciones URL de HTTPS.

`R7121`: los destinatarios PUEDEN aceptar las direcciones URL de HTTPS.

`R7122`: los canales DEBERÍAN aceptar los URI de datos.

`R7123`: los canales NO DEBERÍAN enviar URI de datos a los clientes o los bots.

#### <a name="name"></a>NOMBRE

El campo `name` contiene un nombre o nombre de archivo opcional para los datos adjuntos. El valor del campo `name` es de tipo cadena.

#### <a name="thumbnail-url"></a>Thumbnail URL

Algunos clientes tienen la capacidad de mostrar miniaturas personalizadas para los datos adjuntos no interactivos o como marcadores de posición para los datos adjuntos interactivos. El campo `thumbnailUrl` identifica el origen de esta miniatura. Los URI de datos, según se definen en [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)], normalmente también se permiten.

`R7140`: los destinatarios DEBERÍAN aceptar las direcciones URL de HTTPS.

`R7141`: los destinatarios PUEDEN aceptar las direcciones URL de HTTPS.

`R7142`: los canales DEBERÍAN aceptar los URI de datos.

`R7143`: los canales NO DEBERÍAN enviar los campos `thumbnailUrl` a los bots.

### <a name="card-action"></a>Acción de tarjeta

Una acción de tarjeta representa un botón interactivo o al que se puede hacer clic para usarlo en las tarjetas o como [acciones sugeridas](#suggested-actions). Se utiliza para solicitar la intervención del usuario. A pesar de su nombre, el uso de las acciones de tarjeta no está limitado a las tarjetas.

Las acciones de tarjeta solo son significativas cuando se envían a los canales.

Los canales deciden cómo se manifiesta cada acción en su experiencia de usuario. En la mayoría de los casos, se puede hacer clic en las tarjetas. En otros casos, se pueden seleccionar mediante la entrada de voz. En los casos en que el canal no ofrece ninguna experiencia de activación interactiva (por ejemplo, cuando se interactúa a través de SMS), el canal no puede admitir la activación de ningún modo. La decisión sobre cómo se representan las acciones se controla mediante los requisitos normativos que se describen en otra sección de este documento (por ejemplo, en el formato de la tarjeta o en la definición de las [acciones sugeridas](#suggested-actions)).

#### <a name="type"></a>Escriba

El campo `type` describe el significado del botón y el comportamiento cuando se activa el botón. Los valores del campo `type` son cadenas cortas por convención (p. ej., "`openUrl`"). Consulte las secciones siguientes para conocer los requisitos específicos de cada tipo de acción.

* Una acción de tipo `messageBack` representa una respuesta de texto que se debe enviar a través del sistema de chat.
* Una acción de tipo `imBack` representa una respuesta de texto que se agrega a la fuente del chat.
* Una acción de tipo `postBack` representa una respuesta de texto que no se agrega a la fuente del chat.
* Una acción de tipo `openUrl` representa un hipervínculo que el cliente debe administrar.
* Una acción de tipo `downloadFile` representa un hipervínculo que se debe descargar.
* Una acción de tipo `showImage` representa una imagen que se puede mostrar.
* Una acción de tipo `signin` representa un hipervínculo que el sistema de inicio de sesión del cliente debe administrar.
* Una acción de tipo `playAudio` representa un elemento multimedia de audio que se puede reproducir.
* Una acción de tipo `playVideo` representa un elemento multimedia de vídeo que se puede reproducir.
* Una acción de tipo `call` representa un número de teléfono al que se puede llamar.
* Una acción de tipo `payment` representa una solicitud para proporcionar un pago.

#### <a name="title"></a>Título

El campo `title` incluye el texto que se mostrará en la cara del botón. El valor del campo `title` es de tipo cadena y no contiene marcación.

Este campo se aplica a acciones de todos los tipos.

`R7210`: los canales NO DEBERÍAN procesar la marcación dentro del campo `title` (p. ej., Markdown).

#### <a name="image"></a>Imagen

El campo `image` contiene una dirección URL que hace referencia a una imagen que se mostrará en la cara del botón. Los URI de datos, según se definen en [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)], normalmente son compatibles con los canales. El valor del campo `image` es de tipo cadena.

Este campo se aplica a acciones de todos los tipos.

`R7220`: los canales DEBERÍAN aceptar las direcciones URL de HTTPS.

`R7221`: los canales PUEDEN aceptar las direcciones URL de HTTP.

`R7222`: los canales DEBERÍAN aceptar los URI de datos.

#### <a name="text"></a>Texto

El campo `text` incluye el contenido de texto que se debe enviar a un bot e incluir en la fuente del chat cuando se hace clic en el botón. El contenido del campo `text` puede mostrarse o no, según el tipo de botón. El campo `text` puede contener marcación controlada por el campo [`textFormat`](#text-format) de la raíz de la actividad. El valor del campo `text` es de tipo cadena.

Este campo solo se utiliza en acciones de tipos determinados. Más adelante en este documento se incluyen detalles sobre cada tipo de acción.

`R7230`: el campo `text` PUEDE contener una cadena vacía para indicar el envío de texto sin contenido.

`R7231`: los canales DEBERÍAN procesar el contenido del campo `text` de acuerdo con el campo [`textFormat`](#text-format) de la raíz de la actividad.

#### <a name="display-text"></a>Display text

El campo `displayText` incluye el contenido de texto que se debe incluir en la fuente del chat cuando se hace clic en el botón. El contenido del campo `displayText` siempre SE DEBERÍA mostrar, si es técnicamente posible en el canal. El campo `displayText` puede contener marcación controlada por el campo [`textFormat`](#text-format) de la raíz de la actividad. El valor del campo `displayText` es de tipo cadena.

Este campo solo se utiliza en acciones de tipos determinados. Más adelante en este documento se incluyen detalles sobre cada tipo de acción.

`R7240`: el campo `displayText` PUEDE contener una cadena vacía para indicar el envío de texto sin contenido.

`R7241`: los canales DEBERÍAN procesar el contenido del campo `displayText` de acuerdo con el campo [`textFormat`](#text-format) de la raíz de la actividad.

#### <a name="value"></a>Valor

El campo `value` contiene contenido de programación que se enviará a un bot cuando se haga clic en el botón. Los contenidos del campo `value` son de cualquier tipo primitivo o complejo, aunque ciertos tipos de actividad restringen este campo.

Este campo solo se utiliza en acciones de tipos determinados. Más adelante en este documento se incluyen detalles sobre cada tipo de acción.

#### <a name="message-back"></a>Message Back

Una acción `messageBack` representa una respuesta de texto que se debe enviar a través del sistema de chat. Message Back utiliza los siguientes campos:
* `type` ("`messageBack`")
* `title`
* `image`
* `text`
* `displayText`
* `value` (de cualquier tipo)

`R7350`: los remitentes NO DEBERÍAN incluir los campos `value` de tipos primitivos (p. ej., cadena o entero). Los campos `value` DEBERÍAN ser tipos complejos u omitirse.

`R7351`: los canales PUEDEN rechazar o colocar campos `value` que no son de tipo complejo.

`R7352`: cuando se activa, los canales DEBEN enviar una actividad de tipo `message` a todos los destinatarios pertinentes.

`R7353`: si el canal admite el almacenamiento y la transmisión de texto, el contenido del campo `text` de la acción DEBE conservarse y transmitirse en el campo `text` de la actividad del mensaje generada.

`R7352`: si el canal admite el almacenamiento y la transmisión de valores de programación adicionales, el contenido del campo `value` DEBE conservarse y transmitirse en el campo `value` de la actividad del mensaje generada.

`R7353`: si el canal admite la conservación de un valor diferente en la fuente del chat que se envía a los bots, DEBE incluir el campo `displayText` en el historial del chat.

`R7354`: si el canal no admite `R7353`, pero admite el registro de texto en la fuente de chat, DEBE incluir el campo `text` en el historial de chat.

#### <a name="im-back"></a>IM Back

Una acción `imBack` representa una respuesta de texto que se agrega a la fuente del chat. Message Back utiliza los siguientes campos:
* `type` ("`imBack`")
* `title`
* `image`
* `value` (de tipo cadena)

`R7360`: cuando se activa, los canales DEBEN enviar una actividad de tipo `message` a todos los destinatarios pertinentes.

`R7361`: si el canal admite el almacenamiento y la transmisión de texto, el contenido del campo `title` DEBE conservarse y transmitirse en el campo `text` de la actividad del mensaje generada.

`R7362`: si falta el campo `title` en una acción y el campo `value` es de tipo cadena, el canal PUEDE transmitir el contenido del campo `value` en el campo `text` de la actividad del mensaje generada.

`R7363`: si el canal admite el registro de texto en la fuente de chat, DEBE incluir el contenido del campo `title` en el historial del chat.

#### <a name="post-back"></a>Post Back

Una acción `postBack` representa una respuesta de texto que no se agrega a la fuente del chat. Post Back utiliza los siguientes campos:
* `type` ("`postBack`")
* `title`
* `image`
* `value` (de tipo cadena)

`R7370`: cuando se activa, los canales DEBEN enviar una actividad de tipo `message` a todos los destinatarios pertinentes.

`R7371`: los canales NO DEBERÍAN incluir el texto del historial de chat cuando se activa una acción Post Back.

`R7372`: los canales DEBEN rechazar o colocar campos `value` que no sean de tipo cadena.

`R7373`: si el canal admite el almacenamiento y la transmisión de texto, el contenido del campo `value` DEBE conservarse y transmitirse en el campo `text` de la actividad del mensaje generada.

`R7374`: si el canal no puede admitir la transmisión al bot sin incluir el historial en la fuente del chat, DEBE usar el campo `title` como texto para mostrar.

#### <a name="open-url-actions"></a>Acciones Open URL

Una acción `openUrl` representa un hipervínculo que el cliente debe administrar. Open URL utiliza los siguientes campos:
* `type` ("`openUrl`")
* `title`
* `image`
* `value` (de tipo cadena)

`R7380`: los remitentes DEBEN incluir una dirección URL en el campo `value` de una acción `openUrl`.

`R7381`: los destinatarios PUEDEN rechazar la acción `openUrl` cuyo campo `value` falta o no es una cadena.

`R7382`: los destinatarios DEBERÍAN rechazar o colocar acciones `openUrl` cuyo campo `value` contenga un URI de datos, según se define en [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)].

`R7383`: los destinatarios NO DEBERÍAN rechazar acciones `openUrl` cuyo URI `value` tenga otro valor o esquema de URI inesperado.

`R7384`: los clientes que tengan conocimientos sobres esquemas de URI concretos (p. ej., HTTP) PUEDEN administrar las acciones `openUrl` en una representación insertada (p. ej., un control de explorador).

`R7385`: cuando esté disponible, los clientes DEBERÍAN delegar la administración de las acciones `openUrl` que no administra `R7354` en el controlador de URI del sistema operativo o de nivel de shell.

#### <a name="download-file-actions"></a>Acciones Download File

Una acción `downloadFile` representa un hipervínculo que se debe descargar. Download File utiliza los siguientes campos:
* `type` ("`downloadFile`")
* `title`
* `image`
* `value` (de tipo cadena)

`R7390`: los remitentes DEBEN incluir una dirección URL en el campo `value` de una acción `downloadFile`.

`R7391`: los destinatarios PUEDEN rechazar la acción `downloadFile` cuyo campo `value` falta o no es una cadena.

`R7392`: los destinatarios DEBERÍAN rechazar o colocar acciones `downloadFile` cuyo campo `value` contenga un URI de datos, según se define en [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)].

#### <a name="show-image-file-actions"></a>Acciones de archivo Show Image

Una acción `showImage` representa una imagen que se puede mostrar. Show Image utiliza los siguientes campos:
* `type` ("`showImage`")
* `title`
* `image`
* `value` (de tipo cadena)

`R7400`: los remitentes DEBEN incluir una dirección URL en el campo `value` de una acción `showImage`.

`R7401`: los destinatarios PUEDEN rechazar la acción `showImage` cuyo campo `value` falta o no es una cadena.

`R7402`: los destinatarios PUEDEN rechazar las acciones `showImage` cuyo campo `value` es un URI de datos, según se define en [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)].

#### <a name="signin"></a>Inicio de sesión

Una acción `signin` representa un hipervínculo que el sistema de inicio de sesión del cliente debe administrar. Signin utiliza los siguientes campos:
* `type` ("`signin`")
* `title`
* `image`
* `value` (de tipo cadena)

`R7410`: los remitentes DEBEN incluir una dirección URL en el campo `value` de una acción `signin`.

`R7411`: los destinatarios PUEDEN rechazar la acción `signin` cuyo campo `value` falte o no sea una cadena.

`R7412`: los destinatarios DEBEN rechazar o colocar acciones `signin` cuyo campo `value` contenga un URI de datos, según se define en [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)].

#### <a name="play-audio"></a>Play Audio

Una acción `playAudio` representa un elemento multimedia de audio que se puede reproducir. Play Audio utiliza los siguientes campos:
* `type` ("`playAudio`")
* `title`
* `image`
* `value` (de tipo cadena)

`R7420`: cuando se activa, los canales PUEDEN enviar eventos de elementos multimedia.

`R7421`: los canales DEBEN rechazar o colocar campos `value` que no sean de tipo cadena.

`R7422`: los remitentes NO DEBERÍAN enviar URI de datos, según se define en [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)], sin saber con anterioridad si el canal los admite.

#### <a name="play-video"></a>Play video

Una acción `playAudio` representa un elemento multimedia de vídeo que se puede reproducir. Play Video utiliza los siguientes campos:
* `type` ("`playVideo`")
* `title`
* `image`
* `value` (de tipo cadena)

`R7430`: cuando se activa, los canales PUEDEN enviar eventos de elementos multimedia.

`R7431`: los canales DEBEN rechazar o colocar campos `value` que no sean de tipo cadena.

`R7432`: los remitentes NO DEBERÍAN enviar URI de datos, según se define en [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)], sin saber con anterioridad si el canal los admite.

#### <a name="call"></a>Llamada

Una acción `call` representa un número de teléfono al que se puede llamar. Call utiliza los siguientes campos:
* `type` ("`call`")
* `title`
* `image`
* `value` (de tipo cadena)

`R7440`: los remitentes DEBEN incluir una dirección URL del esquema `tel` en el campo `value` de una acción `signin`.

`R7441`: los destinatarios DEBEN rechazar la acción `signin` cuyo campo `value` falte o no sea una cadena URI del esquema `tel`.

#### <a name="payment"></a>Payment

Una acción `payment` representa una solicitud para proporcionar un pago. Payment utiliza los siguientes campos:
* `type` ("`payment`")
* `title`
* `image`
* `value` (de tipo complejo de [solicitud de pago](#payment-request))

`R7450`: los remitentes DEBEN incluir un objeto complejo de tipo [solicitud de pago](#payment-request) en el campo `value` de una acción `payment`.

`R7451`: los canales DEBEN rechazar la acción `payment` cuyo campo `value` falte o no sea un objeto complejo de tipo [solicitud de pago](#payment-request).

### <a name="channel-account"></a>Cuenta de canal

Las cuentas de canal representan identidades de un canal. La cuenta de canal incluye un identificador que puede usarse para identificar la cuenta de ese canal y establecer contacto con ella. En algunos casos, estos identificadores existen dentro de un único espacio de nombres (por ejemplo, identificadores de Skype), mientras que, en otros, están federados en varios servidores (por ejemplo, direcciones de correo electrónico). Además del identificador, las cuentas de canal incluyen nombres para mostrar e identificadores de objetos de Azure Active Directory (AAD).

#### <a name="channel-account-id"></a>Identificador de la cuenta de canal

El campo `id` es el identificador y la dirección del canal. El valor del campo `id` es una cadena. Un ejemplo del valor de `id` dentro de un canal que utiliza direcciones de correo electrónico es "name@example.com".

`R7510`: los canales DEBERÍAN usar los mismos valores y convenciones para los identificadores de cuenta, independientemente de su posición dentro del esquema (`from.id`, `recipient.id`, `membersAdded`, etc.). Esto permite que los bots y los clientes usen comparaciones de cadenas ordinales, por ejemplo, para saber cuándo se describen en el campo `membersAdded` de una actividad `conversationUpdate`.

#### <a name="channel-account-name"></a>Nombre de la cuenta de canal

El campo `name` es un nombre descriptivo opcional del canal. El valor del campo `name` es una cadena. Un valor de `name` de ejemplo de un canal es "Jorge Alcalá".

#### <a name="channel-account-aad-object-id"></a>Identificador del objeto de AAD de la cuenta de canal

El campo `aadObjectId` es un identificador opcional correspondiente al identificador del objeto de la cuenta de Azure Active Directory (AAD). El valor del campo `aadObjectId` es una cadena.

### <a name="conversation-account"></a>Cuenta de conversación

Las cuentas de conversación representan la identidad de las conversaciones en un canal. En los canales que solo se admite una conversación única entre dos cuentas (p. ej., los SMS), la cuenta de conversación se conserva y no tiene un inicio o final predeterminados. En los canales que se admiten varias conversaciones paralelas (p. ej., el correo electrónico), cada conversación probablemente tendrá un identificador único.

#### <a name="conversation-account-id"></a>Identificador de la cuenta de conversación

El campo `id` es el identificador del canal. El formato de este identificador lo define el canal y se usa como una cadena opaca a lo largo del protocolo.

Los canales DEBERÍAN elegir los valores `id` que son estables para todos los participantes de una conversación. (Por ejemplo, un ejemplo deficiente para el campo `id` de una conversación de 1:1 es usar el identificador del otro participante como el valor `id`. Esto daría lugar a otro valor `id` desde la perspectiva de cada participante. Una opción mejor es ordenar los identificadores de ambos participantes y concatenarlos entre sí, lo que sería lo mismo para ambas partes).

#### <a name="conversation-account-name"></a>Nombre de la cuenta de conversación

El campo `name` es un nombre descriptivo opcional para la conversación en el canal. El valor del campo `name` es una cadena.

#### <a name="conversation-account-aad-object-id"></a>Identificador del objeto de AAD de la cuenta de conversación

El campo `aadObjectId` es un identificador opcional correspondiente al identificador del objeto de la conversación de Azure Active Directory (AAD). El valor del campo `aadObjectId` es una cadena.

#### <a name="conversation-account-is-group"></a>Cuenta de conversación Is Group

El campo `isGroup` indica si la conversación contiene más de dos participantes en el momento en que se genera la actividad. El valor del campo `isGroup` es un valor booleano; si se omite, el valor predeterminado es `false`. Este campo normalmente controla el comportamiento de las menciones de los participantes en el canal y se DEBERÍA establecer en `true`, si y únicamente si hay más de dos participantes que tienen la capacidad de enviar y recibir actividades dentro de la conversación.

### <a name="entity"></a>Entidad

Las entidades incluyen metadatos sobre una actividad o conversación. El campo `type` define el significado y la forma de cada entidad, y los campos específicos de tipo adicionales se colocan como si fueran del mismo nivel en el campo `type`.

`R7600`: los destinatarios DEBEN ignorar las entidades cuyos tipos no comprendan.

`R7601`: los destinatarios DEBERÍAN ignorar las entidades cuyo tipo comprenden, pero que no pueden procesar, por ejemplo, debido a errores sintácticos.

Se recomienda que las partes que proyectan un tipo de entidad existente en el formato de entidad de actividad resuelvan los conflictos con el nombre del campo `type` y las incompatibilidades con el requisito de serialización `R2001` como parte del enlace entre el esquema de entidad y el IRI.

#### <a name="type"></a>Escriba

El campo `type` es obligatorio y define el significado y la forma de la entidad. `type` está pensado para contener [IRI](https://tools.ietf.org/html/rfc3987) [[3](#references)], aunque hay un pequeño número de tipos de entidad que no son de IRI definidos en el [Apéndice I](#appendix-ii---non-iri-entity-types).

`R7610`: los remitentes solo DEBERÍAN usar nombres de tipos que no sean IRI para los tipos descritos en el [Apéndice II](#appendix-ii---non-iri-entity-types).

`R7611`: los remitentes PUEDEN enviar tipos IRI para los tipos que se describen en el [Apéndice II](#appendix-ii---non-iri-entity-types) si saben que el destinatario los comprende.

`R7612`: los remitentes DEBERÍAN usar o establecer IRI para los tipos de entidad que no estén definidos en el [Apéndice II](#appendix-ii---non-iri-entity-types).

### <a name="suggested-actions"></a>Acciones sugeridas

Las acciones sugeridas se pueden enviar en el contenido del mensaje para crear elementos de acción interactivos dentro de la interfaz de usuario de un cliente.

`R7700`: los clientes que no admiten una interfaz de usuario que pueda procesar las acciones sugeridas DEBERÍAN omitir el campo `suggestedActions`.

`R7701`: los remitentes DEBERÍAN omitir el campo `suggestedActions` si el campo `actions` está vacío.

#### <a name="to"></a>Para

El campo `to` contiene identificadores de la cuenta de canal a los que se deberían mostrar las acciones sugeridas. Este campo se puede utilizar para filtrar las acciones para un subconjunto de los participantes de la conversación.

`R7710`: si falta el campo `to` o está vacío, el cliente DEBERÍA mostrar las acciones sugeridas para todos los participantes de la conversación.

`R7711`: si el campo `to` contiene identificadores no válidos, se DEBERÍAN ignorar esos valores.

#### <a name="actions"></a>Acciones

El campo `actions` contiene una lista plana de las acciones que se deben mostrar. El valor de cada elemento de lista `actions` es un objeto complejo de tipo `cardAction`.

### <a name="message-reaction"></a>Reacción de los mensajes

Las reacciones de los mensajes representan una interacción social ("Me gusta", "+ 1", etc.). Las reacciones de los mensajes actualmente solo incluyen un único campo: `type`.

#### <a name="type"></a>Escriba

El campo `type` describe el tipo de interacción social. El valor del campo `type` es una cadena y su significado lo define el canal en el que se produce la interacción. Algunos valores comunes son `like` y `+1`, aunque estos son uniformes por convención y no por regla.

## <a name="references"></a>Referencias

1. [RFC 2119](https://tools.ietf.org/html/rfc2119)
2. [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)
3. [RFC 3987](https://tools.ietf.org/html/rfc3987)
4. [Markdown](https://daringfireball.net/projects/markdown/)
5. [ISO 639](https://www.iso.org/iso-639-language-codes.html)
6. [SSML](https://www.w3.org/TR/speech-synthesis/)
7. [XML](https://www.w3.org/TR/xml/)
8. [Tipos de elementos multimedia MIME](https://www.iana.org/assignments/media-types/media-types.xhtml)
9. [RFC 2397](https://tools.ietf.org/html/rfc2397)
10. [ISO 3166-1](https://www.iso.org/iso-3166-country-codes.html)
11. [Adaptive Cards](https://adaptivecards.io) (Tarjetas adaptables)

# <a name="appendix-i---changes"></a>Apéndice I: Cambios

## <a name="2018-02-07"></a>07/02/2018

* Borrador inicial

# <a name="appendix-ii---non-iri-entity-types"></a>Apéndice II: Tipos de entidad que no son de IRI

Las [entidades](#entity) de la actividad transmiten metadatos adicionales acerca de la actividad, como la ubicación de un usuario o la versión de la aplicación de mensajería que usa. Los tipos de actividad están diseñados para ser IRI, pero existe una pequeña lista de nombres que no son de IRI cuyo uso también es común. Este apéndice contiene una lista completa de los tipos de entidad que no son de IRI compatibles.

| Escriba           | Equivalente del URI                          | DESCRIPCIÓN               |
| -------------- | --------------------------------------- | ------------------------- |
| GeoCoordinates | https://schema.org/GeoCoordinates       | Coordenadas geográficas de Schema.org |
| Mention        | https://botframework.com/schema/mention | @-mention                 |
| Lugar          | https://schema.org/Place                | Lugar de Schema.org          |
| Thing          | https://schema.org/Thing                | Cosa de Schema.org          |
| clientInfo     | N/D                                     | Información del cliente de Skype         |

### <a name="clientinfo"></a>clientInfo

La entidad clientInfo contiene información ampliada sobre el software cliente que se usa para enviar un mensaje a un usuario. Contiene tres propiedades, que son opcionales.

`R9201`: los bots NO DEBERÍAN enviar la entidad `clientInfo`.

`R9202`: los remitentes solo DEBERÍAN incluir la entidad `clientInfo` cuando se rellenen uno o varios campos.

#### <a name="locale-deprecated"></a>Locale (en desuso)

El campo `locale` contiene la configuración regional del usuario. Este campo duplica el campo [`locale`](#locale) en la raíz de la actividad. El valor del campo `locale` es un código [ISO 639](https://www.iso.org/iso-639-language-codes.html) [[5](#references)] de una cadena.

El campo `locale` de `clientInfo` está en desuso.

`R9211`: los destinatarios NO DEBERÍAN usar el campo `locale` dentro del objeto `clientInfo`.

`R9212`: los remitentes PUEDEN rellenar el campo `locale` en `clientInfo` por motivos de compatibilidad. Si no se requiere compatibilidad con destinatarios anteriores, los remitentes NO DEBERÍAN enviar la propiedad `locale`.

#### <a name="country"></a>País

El campo `country` contiene la ubicación detectada del usuario. Este valor puede diferir de cualquier dato del valor [`locale`](#locale) porque `country` se detecta, mientras que `locale` suele ser la configuración de la aplicación o del usuario. El valor del campo `country` es un código de país de 2 o 3 letras de la [ISO 3166-1](https://www.iso.org/iso-3166-country-codes.html) [[10](#references)].

`R9220`: los canales NO DEBERÍAN permitir que los clientes especifiquen valores arbitrarios para el campo `country`. Los canales DEBERÍAN usar un mecanismo, como el GPS, la API de ubicación o la detección de direcciones IP, para establecer el país en que se genera una solicitud.

#### <a name="platform"></a>Plataforma

El campo `platform` describe la plataforma de cliente de mensajería utilizada para generar la actividad. El valor del campo `platform` es una cadena y la lista de valores posibles y su significado la define el canal que la envía.

Tenga en cuenta que, en los canales con una fuente de chat persistente, `platform` normalmente solo resulta útil a la hora de decidir qué contenido se debe incluir y no para el formato de ese contenido. Por ejemplo, si un usuario solicita ayuda de soporte técnico para un producto desde un dispositivo móvil, un bot podría generar la ayuda específica para su dispositivo móvil. Sin embargo, a continuación, el usuario puede volver a abrir la fuente de chat en su equipo para leerla en esa pantalla mientras realiza cambios en su dispositivo móvil. En este caso, el campo `platform` está pensado para informar del contenido, pero el contenido debe ser visible en otros dispositivos.

`R9230`: los bots NO DEBERÍAN usar el campo `platform` para controlar cómo se da formato a los datos de respuesta, a menos que tengan conocimientos específicos de que el contenido que envían solo se verá en el dispositivo en cuestión.

# <a name="appendix-iii---protocols-using-the-invoke-activity"></a>Apéndice III: Protocolos que usan la actividad de invocación

La [actividad de invocación](#invoke-activity) está diseñada para usarse solo en protocolos que admiten los canales de Bot Framework (es decir, no es un mecanismo de extensibilidad genérico). Este apéndice contiene una lista de todos los protocolos de Bot Framework que usan esta actividad.

## <a name="payments"></a>Pagos

El protocolo de pagos de Bot Framework usa la invocación para calcular las tasas de envío e impuestos y para transmitir una confirmación segura de los pagos completados.

El protocolo de pagos define tres operaciones (que se definen en el campo `name` de una actividad de invocación):
* `payment/shippingAddressChange`
* `payment/shppingOptionsChange`
* `payment/paymentResponse`

Se pueden encontrar más detalles en la página [Request payment](https://docs.microsoft.com/en-us/bot-framework/dotnet/bot-builder-dotnet-request-payment) (Solicitar pago).

## <a name="teams-compose-extension"></a>Extensión de redacción de Teams

El canal de Microsoft Teams usa la invocación para las [extensiones de redacción](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/messaging-extensions). Este uso de invocación es específico de Microsoft Teams.



