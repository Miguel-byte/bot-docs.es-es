---
ms.openlocfilehash: bbdad5f0cb51d99cf886070dd8400acae0ed24bb
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464572"
---
# <a name="bot-review-guidelines"></a>Directrices de revisión de bots

Le damos la bienvenida y le agradecemos que invierta su talento y tiempo en la creación de bot, botlets, aplicaciones web, complementos o habilidades ("integraciones de aplicaciones") para los canales de Microsoft. A continuación, se especifican los requisitos mínimos que la integración de la aplicación debe cumplir para poder publicarla en un canal de Microsoft, como Skype o Microsoft Teams. Cada canal puede tener requisitos específicos además de los requisitos que se detallan a continuación. Si procede, encontrará términos específicos de cada canal en la página de configuración de cada uno de ellos, y puede que tenga que registrarse en el servicio de un canal para poder publicar un bot en dicho canal.

## <a name="app-integration-policies"></a>Directivas de integración de aplicaciones
### <a name="1-value-representation-security-and-usability"></a>1. Valor, representación, seguridad y facilidad de uso.

La integración de aplicaciones y sus metadatos asociados:

- Deben tener metadatos únicos e informativos y proporcionar una experiencia de usuario valiosa y de calidad.
- Deben reflejar con precisión y claridad el origen, la funcionalidad y las características de la integración de aplicaciones y describir cualquier limitación importante;
- No deben usar un nombre o icono similares a los de otras aplicaciones y no pueden declarar su pertenencia a una empresa, entidad gubernamental u otro organismo si no se dispone de autorización para realizar tal representación.
- No deben perjudicar ni comprometer la seguridad del usuario ni la seguridad o funcionalidad del canal de Microsoft.
- No deben intentar cambiar o ampliar la funcionalidad descrita de tal forma que se infrinjan estas directivas o los términos aplicables del canal de Microsoft.
- No deben incluir ni habilitar malware.
- Deben ofrecer la posibilidad de probarse.
- Deben seguir ejecutándose y respondiendo a la entrada del usuario. 
- Deben incluir un vínculo de trabajo a los términos de servicio.
- Deben funcionar como se describe en su descripción, perfil, términos de uso y directiva de privacidad.
- Deben funcionar de conformidad con los términos aplicables a Microsoft Bot Framework u otros términos aplicables a su desarrollo, así como con los términos de los canales de Microsoft u otros términos aplicables al canal en el que se ha publicado la integración de la aplicación.
- Deben informar a los usuarios si la integración de la aplicación incluye la interacción humana (por ejemplo, servicio de atención al cliente o soporte técnico con una persona directamente).
- Deben localizarse en todos los idiomas admitidos. El texto de la descripción de la integración de la aplicación debe localizarse en cada idioma declarado.

### <a name="2--privacy"></a>2.  Privacidad

- Si la integración de la aplicación trata información personal del usuario (por información personal se entienden todos los datos o la información que identifican o pueden usarse para identificar a una persona, o bien que están asociados con dichos datos o información; los ejemplos de información personal incluyen: nombre y dirección, número de teléfono, identificadores biométricos, ubicación, contactos, fotos, grabaciones de audio y vídeo, documentos, SMS, correo electrónico u otras comunicaciones de texto, capturas de pantalla y, en algunos casos, el historial de navegación combinado), debe proporcionar un vínculo destacado de la integración de la aplicación a una directiva de privacidad correspondiente, y dicha directiva de privacidad debe cumplir todas las leyes, los reglamentos y las disposiciones de la directiva que sean de aplicación. Esta directiva debe cubrir qué datos recopila o transmite, qué va a realizar con esos datos y, si procede, con quién los va a compartir. Si no tiene una declaración de privacidad, aquí le indicamos algunos recursos de terceros* que pueden resultarle útiles:

Foro sobre el futuro de la privacidad: [generador de directiva de privacidad para aplicaciones](http://www.applicationprivacy.org/do-tools/privacy-policy-generator/)

Iubenda: [generador de directiva de privacidad](http://www.iubenda.com/en)

*_Se compromete a asumir todos los riesgos y responsabilidades derivados del uso de estos recursos de terceros y acepta que Microsoft no se hace responsable de los problemas que puedan surgir de su uso._
- La integración de la aplicación no debe recopilar, almacenar ni transmitir información personal no relacionada con su objetivo principal, sin obtener primero el consentimiento expreso del usuario. Debe obtener todos los consentimientos de los usuarios para procesar los datos personales cuando así lo exija la ley. 
- No puede publicar ninguna integración de aplicaciones dirigida a menores de trece años, de conformidad con lo previsto en la ley estadounidense de protección de la Ley de Protección de la Privacidad de Menores de los Estados Unidos, sin el consentimiento expreso de Microsoft.

### <a name="3--financial-transactions"></a>3.  Transacciones financieras
- Para integraciones de aplicaciones sujetas a pago: 
  - La integración de la aplicación no puede transmitir los detalles del instrumento financiero a través de la interfaz de usuario.
  - Sin perjuicio de las restricciones de cualquier plataforma de terceros o canal, la integración de la aplicación puede: a) admitir pagos a través del [Centro de vendedores de Microsoft](https://seller.microsoft.com/) de conformidad con los términos del contrato del Centro de vendedores de Microsoft o b) transmitir vínculos a otros servicios de pago seguros.
  - Si la integración de la aplicación habilita los mecanismos de pago anteriores, debe declararlo así en los términos de uso y la directiva de privacidad de las integraciones de la aplicación y en cualquier página de perfil o sitio web de la integración de la aplicación antes de que el usuario final acepte el uso de la integración de la aplicación.
  - Debe indicar claramente que la integración de una aplicación está sujeta a pago en su perfil y proporcionar a los usuarios finales los detalles del servicio de atención al cliente del comerciante.
  - No podrá publicar integraciones de aplicaciones en cualquier canal de Microsoft que incluyan vínculos o que remitan a los usuarios de otro modo a servicios de pago para la compra de recursos digitales sin el permiso expreso de Microsoft.

### <a name="4--content"></a>4.  Contenido 
- Todo el contenido de la integración de la aplicación y los metadatos asociados debe crearlos originalmente el anunciante, deben disponer de la licencia adecuada emitida por el tercero titular de los derechos, deben utilizarse según permita el titular de los derechos o, en su defecto, de conformidad con lo que disponga la legislación aplicable.
- No publicará una integración de aplicaciones o contenido en la integración de aplicaciones si: 
  - Son ilícitos.
  - Vulneran, dañan o amenazan con dañar a menores.
  - Incluyen publicidad, spam o mensajes, publicaciones o comunicaciones en masa o no deseados o no solicitados.
  - Podrían considerarse como material inapropiado u ofensivo (lo que engloba, por ejemplo, desnudos, zoofilia, blasfemias, pornografía o violencia sexualmente explícita, gráfica o gratuita, tabaco, actividades penales, peligrosas o irresponsables, drogas, infracciones de derechos humanos, creación o uso ilegal de armas contra una persona o animal en el mundo real o uso irresponsable de productos alcohólicos).
  - Son falsos o engañosos (por ejemplo, pedir dinero con falsos pretextos, suplantar a una persona, etc.).
  - Son perjudiciales para usted, el canal de Microsoft u otros o entrañan un riesgo de seguridad (por ejemplo, transmitir virus, acoso, publicación de contenido terrorista, comunicación de incitación al odio o declaraciones discriminatorias, de odio o violencia por razones de raza, etnia, nacionalidad, idioma, sexo, edad, discapacidad, religión, orientación sexual, condición de veterano o pertenencia a cualquier otro grupo social).
  - Infringen los derechos de terceros (por ejemplo, uso compartido no autorizado de música u otro material sujetos a derechos de autor).
  - Son difamatorios, calumniosos o amenazantes.
  - Infringen la privacidad de terceros. 
  - Procesan información que a) está relacionada con la enfermedad, el tratamiento o el pago del tratamiento de un paciente, b) identifica a personas como pacientes, miembros o beneficiaros de un plan de salud o se comunica con ellos, c) está categorizada como "información de salud confidencial" en el marco de la Ley de Portabilidad y Contabilidad de los Seguros de Salud ("HIPAA"), en su versión modificada, o bien si se realiza alguna actividad regida por HIPAA en caso de tratarse de una "entidad cubierta" o "empresa asociada", según se define en el marco de HIPAA.
  - Son ofensivos en cualquier país o región a los que se destina la aplicación. El contenido se podrá considerar ofensivo en determinados países o regiones debido a las leyes locales o las normas culturales.
  - Permiten a terceros incumplir estas reglas. 
- Si realiza algún cambio importante en la integración de la aplicación, debe informar a Microsoft de antemano con el envío de un correo electrónico al canal especificado en el que la ha publicado.  Puede que sea necesario volver a revisar el bot, en caso de que se realicen cambios en el registro del bot, a fin de garantizar que sigue cumpliendo los requisitos aquí señalados.  Microsoft tiene derecho, a su entera discreción, a revisar intermitentemente las integraciones de aplicaciones en cualquier canal y a eliminarlas sin previo aviso.
