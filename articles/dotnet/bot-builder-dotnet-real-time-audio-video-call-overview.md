---
title: Compilación de un bot de elementos multimedia en tiempo real para Skype | Microsoft Docs
description: Obtenga información sobre cómo compilar un bot que realiza las llamadas de audio y vídeo en tiempo real con Skype, mediante el SDK del generador de bots para .NET y el SDK de RealTimeMediaCalling del generador de bots para .NET.
author: MalarGit
ms.author: malarch
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6ceeca9adc9cad9e60a73c1c7c91bea43b97fdd9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997934"
---
# <a name="build-a-real-time-media-bot-for-skype"></a>Compilar de un bot de elementos multimedia en tiempo real para Skype

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

La Plataforma de elementos multimedia en tiempo real para bots es una funcionalidad avanzada que permite que el bot envíe y reciba contenido de voz y vídeo trama a trama y fotograma a fotograma. El bot tiene acceso "sin procesar" a las modalidades de voz, vídeo y pantalla compartida en tiempo real. En este artículo se proporciona información general sobre la compilación de un bot que realiza llamadas de audio y vídeo y el acceso a las modalidades en tiempo real.

En este artículo, el bot se ejecuta en un servicio en la nube de Azure, como un rol web o un rol de trabajo que autohospeda la plataforma ASP.NET Web API.

> [!IMPORTANT]
> El contenido de este artículo es preliminar. Consulte la carpeta de ejemplos del repositorio<a href="https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling">BotBuilder-RealTimeMediaCalling</a> de GitHub para ver todo el código de los bots de elementos multimedia en tiempo real de ejemplo.

## <a name="configure-the-service-hosting-the-real-time-media-bot"></a>Configuración del servicio que hospeda el bot de elementos multimedia en tiempo real

Para usar la plataforma de elementos multimedia en tiempo real, se necesitan las configuraciones de servicio siguientes.

* El bot debe conocer el nombre de dominio completo (FQDN) de su servicio. No lo proporciona la API Azure RoleEnvironment; en su lugar, el nombre de dominio completo se debe almacenar en la configuración de servicio en la nube del bot y se debe leer durante el inicio del servicio.

* Bot Service debe tener un certificado emitido por una entidad de certificación reconocida. La huella digital del certificado se debe almacenar en la configuración del servicio en la nube del bot y se debe leer durante el inicio del servicio.

* Se debe aprovisionar un <a href="/azure/cloud-services/cloud-services-enable-communication-role-instances#instance-input-endpoint">punto de conexión de entrada de instancia</a> público. Esto asigna un puerto público único a cada instancia de máquina virtual (VM) en el servicio del bot. Este puerto lo usa la plataforma de elementos multimedia en tiempo real para comunicarse con la nube de llamadas de Skype.
  ```xml
  <InstanceInputEndpoint name="InstanceMediaControlEndpoint" protocol="tcp" localPort="20100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="20200" min="20101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

  También es útil para crear otro punto de conexión de entrada de instancia para devoluciones de llamada y notificaciones relacionadas con las llamadas. Usar un punto de conexión de entrada de instancia garantiza que las devoluciones de llamada y las notificaciones se entreguen a la misma instancia de VM en la implementación del servicio que hospeda la sesión multimedia en tiempo real para la llamada.
  ```xml
  <InstanceInputEndpoint name="InstanceCallControlEndpoint" protocol="tcp" localPort="10100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="10200" min="10101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

* Cada instancia de VM debe tener una dirección IP pública de nivel de instancia (ILPIP). Durante el inicio, el bot debe detectar la dirección ILPIP asignada a cada instancia de servicio. Consulte <a href="/azure/virtual-network/virtual-networks-instance-level-public-ip">ILPIP</a> para más información sobre cómo obtener y configurar una dirección ILPIP.
  ```xml
  <NetworkConfiguration>
  <AddressAssignments>
    <InstanceAddress roleName="WorkerRole">
    <PublicIPs>
        <PublicIP name="InstancePublicIP" domainNameLabel="InstancePublicIP" />
    </PublicIPs>
    </InstanceAddress>
  </AddressAssignments>
  </NetworkConfiguration>
  ```

* Durante el inicio de la instancia de servicio, el script `MediaPlatformStartupScript.bat` (proporcionado como parte del paquete NuGet) se debe ejecutar como una tarea de inicio con privilegios elevados. La ejecución del script se debe completar antes de llamar al método de inicialización de la plataforma. 

```xml
<Startup>
<Task commandLine="MediaPlatformStartupScript.bat" executionContext="elevated" taskType="simple" />      
</Startup> 
```

## <a name="initialize-the-media-platform-on-service-startup"></a>Inicialización de la plataforma multimedia al inicio del servicio

Durante el inicio de la instancia de servicio, se debe inicializar la plataforma multimedia en tiempo real. Esto debe hacerse solo una vez antes de que el bot pueda aceptar las llamadas de Skype de audio y vídeo en esa instancia. La inicialización de la plataforma multimedia requiere proporcionar varias opciones de configuración del servicio, incluido el nombre de dominio completo del servicio, la dirección ILPIP pública, el valor del puerto del punto de conexión de entrada de instancia y el **identificador de aplicación de Microsoft** del bot.

> [!NOTE]
> Para buscar loPara buscar los valores **AppID** y **AppPassword** del bot, consulte [MicrosoftAppID y MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

```cs
var mediaPlatformSettings = new MediaPlatformSettings()
{
    MediaPlatformInstanceSettings = new MediaPlatformInstanceSettings()
    {
        CertificateThumbprint = certificateThumbprint,
        InstanceInternalPort = instanceMediaControlEndpointInternalPort,
        InstancePublicIPAddress = instancePublicIPAddress,
        InstancePublicPort = instanceMediaControlEndpointPublicPort,
        ServiceFqdn = serviceFqdn
    },

    ApplicationId = MicrosoftAppId
};

MediaPlatform.Initialize(mediaPlatformSettings);            
```

## <a name="register-to-receive-incoming-call-requests"></a>Registro para recibir solicitudes de llamadas entrantes

Defina una clase `CallController`. Esto permite que Bot Service se registre para las llamadas entrantes de Skype y se asegure de que las solicitudes de devolución de llamada y notificación se reenvían al objeto `RealTimeMediaCall` correspondiente.

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallController : ApiController
{
    static CallController()
    {
        RealTimeMediaCalling.RegisterRealTimeMediaCallingBot(
            c => { return new RealTimeMediaCall(c); },
            new RealTimeMediaCallingBotServiceSettings()
        );
    }

    [Route("call")]
    public async Task<HttpResponseMessage> OnIncomingCallAsync()
    {
        // forwards the incoming call to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.IncomingCall);
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> OnCallbackAsync()
    {
        // forwards the incoming callback to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.CallingEvent);
    }

    [Route("notification")]
    public async Task<HttpResponseMessage> OnNotificationAsync()
    {
        // forwards the incoming notification to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.NotificationEvent);
    }
}
```

`RealTimeMediaCallingBotServiceSettings` implementa `IRealTimeMediaCallServiceSettings` y proporciona vínculos de webhook para eventos de devolución de llamada y notificación.

## <a name="register-for-incoming-events-for-the-call"></a>Registro para los eventos entrantes de la llamada

Defina una clase `RealTimeMediaCall` que implemente `IRealTimeMediaCall`. Para cada llamada que el bot recibe, Bot Framework crea una instancia de `RealTimeMediaCall`. El objeto `IRealTimeMediaCallService` pasado al constructor permite que el bot se registre para eventos con el fin de controlar los eventos asociados con la llamada multimedia en tiempo real.

```cs
internal class RealTimeMediaCall : IRealTimeMediaCall
{
     public RealTimeMediaCall(IRealTimeMediaCallService callService)
     {
         if (callService == null)
             throw new ArgumentNullException(nameof(callService));

         CallService = callService;
         CorrelationId = callService.CorrelationId;
         CallId = CorrelationId + ":" + Guid.NewGuid().ToString();

         // Register for the call events
         CallService.OnIncomingCallReceived += OnIncomingCallReceived;
         CallService.OnAnswerAppHostedMediaCompleted += OnAnswerAppHostedMediaCompleted;
         CallService.OnCallStateChangeNotification += OnCallStateChangeNotification;
         CallService.OnRosterUpdateNotification += OnRosterUpdateNotification;
         CallService.OnCallCleanup += OnCallCleanup;
     }
}
```

## <a name="create-audio-and-video-sockets"></a>Creación de sockets de audio y vídeo
Antes de que el bot pueda aceptar una llamada entrante de audio o vídeo de Skype, debe crear los objetos `AudioSocket` y `VideoSocket` para admitir el envío y la recepción de elementos multimedia en tiempo real. (Si el bot no quiere admitir el vídeo, debe crear solo un objeto AudioSocket).

El bot debe decidir por adelantado qué modalidades quiere admitir y deberá crear los objetos AudioSocket y VideoSocket adecuados. El bot no puede cambiar las modalidades que admite para la llamada una vez que se acepta la llamada entrante.

Para cada objeto AudioSocket y VideoSocket, el bot especifica si el socket multimedia es para admitir tanto el envío como la recepción de elementos multimedia o solo el envío o la recepción. Por ejemplo, es posible que el bot quiera enviar y recibir audio ("Sendrecv"), pero solo enviar vídeo ("Sendonly"). El bot también debe especificar los formatos multimedia que se admiten para cada socket multimedia. Para un objeto AudioSocket, el formato actualmente compatible es "Pcm16K": (firmado) codificación PCM de 16 bits, frecuencia de muestreo de 16 KHz. Para un objeto VideoSocket, los formatos multimedia para envío y recepción se especifican por separado. Solo se admite el formato "NV12" para recibir vídeo, mientras que se admiten varios formatos distintos para envío.

```cs
_audioSocket = new AudioSocket(new AudioSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    SupportedAudioFormat = AudioFormat.Pcm16K,
    CallId = correlationId
});

_videoSocket = new VideoSocket(new VideoSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    ReceiveColorFormat = VideoColorFormat.NV12,
    SupportedSendVideoFormats = new VideoFormat[]
    {
        VideoFormat.Yuy2_1280x720_30Fps,
        VideoFormat.Yuy2_720x1280_30Fps,
    },
    CallId = correlationId
});

_audioSocket.AudioMediaReceived += OnAudioMediaReceived;
_audioSocket.AudioSendStatusChanged += OnAudioSendStatusChanged;
_audioSocket.DominantSpeakerChanged += OnDominantSpeakerChanged;
_videoSocket.VideoMediaReceived += OnVideoMediaReceived;
_videoSocket.VideoSendStatusChanged += OnVideoSendStatusChanged;
```                             

## <a name="create-a-mediaconfiguration"></a>Creación de un objeto MediaConfiguration
Una vez que se crean los sockets multimedia, el bot debe crear un objeto `MediaConfiguration` que es necesario para asociar los sockets multimedia con una llamada entrante de audio o vídeo de Skype.

```cs
var mediaConfiguration = MediaPlatform.CreateMediaConfiguration(_audioSocket, _videoSocket);
```

##  <a name="answer-an-incoming-audiovideo-call"></a>Respuesta a una llamada de audio o vídeo entrante
El evento `OnIncomingCallReceived` se invoca para permitir que el bot acepte la llamada de audio o vídeo entrante de Skype. Para hacerlo, el bot crea un objeto `AnswerAppHostedMedia` con el objeto `MediaConfiguration`. El bot se registra para las notificaciones asociadas con esta llamada de Skype.

```cs
private Task OnIncomingCallReceived(RealTimeMediaIncomingCallEvent incomingCallEvent)
{
    // ... create Audio/VideoSocket objects and MediaConfiguration ...

    incomingCallEvent.RealTimeMediaWorkflow.Actions = new ActionBase[]
    {
        new AnswerAppHostedMedia
        {
            MediaConfiguration = mediaConfiguration,
            OperationId = Guid.NewGuid().ToString()
        }
    };

    // subscribe for roster and call state changes
    incomingCallEvent.RealTimeMediaWorkflow.NotificationSubscriptions = new NotificationType[]
    {
        NotificationType.CallStateChange,
        NotificationType.RosterUpdate
    };
}
```

## <a name="outcome-of-the-call"></a>Resultado de la llamada
`OnAnswerAppHostedMediaCompleted` se produce cuando se completa la acción `AnswerAppHostedMedia`. La propiedad `Outcome` en `AnswerAppHostedMediaOutcomeEvent` indica si la acción se realizó correctamente o con errores. Si no se puede establecer la llamada, el bot debe eliminar los objetos AudioSocket y VideoSocket que creó para la llamada.

## <a name="receive-audio-media"></a>Recepción de elementos multimedia de audio
Si el objeto `AudioSocket` se creó con la capacidad de recibir audio, el evento `AudioMediaReceived` se invocará cada vez que se reciba una trama de audio. El bot debe esperar a controlar este evento aproximadamente 50 veces por segundo, independientemente del par que podría originar el contenido de audio (debido a que los búferes de ruido aceptable se generan localmente si no se recibe audio del par). Cada paquete de contenido de audio se entrega en un objeto `AudioMediaBuffer`. Este objeto contiene un puntero a un búfer de memoria asignados por montón nativo que contiene el contenido de audio descodificado. 

```cs
void OnAudioMediaReceived(
            object sender,
            AudioMediaReceivedEventArgs args)
{
   var buffer = args.Buffer;

   // native heap-allocated memory containing decoded content
   IntPtr rawData = buffer.Data;            
}
```

El controlador de eventos se debe devolver rápidamente. Se recomienda que la aplicación ponga en la cola el objeto `AudioMediaBuffer` que se procesará de manera asincrónica. Los eventos de `OnAudioMediaReceived` se serializarán a través de la plataforma multimedia en tiempo real (es decir, el evento siguiente no se generará hasta que se devuelva el actual). Una vez que se consume un objeto `AudioMediaBuffer`, la aplicación debe llamar al método Dispose del búfer para que la plataforma multimedia pueda reclamar la memoria subyacente no administrada. 

```cs
   // release/dispose buffer when done 
   buffer.Dispose();
```

> [!IMPORTANT]
> El bot no debe llamar al método Dispose del búfer hasta que haya terminado de acceder al búfer.

## <a name="receive-video-media"></a>Recepción de elementos multimedia de vídeo
La recepción de vídeo es similar a recibir audio, excepto en que el número de búferes por segundo dependerá del valor de la velocidad de fotogramas. `VideoMediaBuffer` tiene las propiedades `VideoFormat` y `OriginalVideoFormat`. `OriginalVideoFormat` representa el formato original del búfer cuando se originó. Solo está disponible cuando se reciben búferes de vídeo a través del controlador de eventos `IVideoSocket.VideoMediaReceived`. Si el tamaño del búfer se cambió antes de empezar la transmisión, la propiedad `OriginalVideoFormat` tendrá el ancho y el alto original, mientras que `VideoFormat` tendrá los valores actuales después del cambio de tamaño. Si las propiedades de ancho y alto de `OriginalVideoFormat` difieren de la propiedad `VideoFormat`, el consumidor del objeto `VideoMediaBuffer` que se produjo en el evento `VideoMediaReceived` debe cambiar el tamaño del búfer para ajustarlo al tamaño de `OriginalVideoFormat`. Actualmente solo se admite el formato NV12 para recepción.

## <a name="send-audio-media"></a>Envío de elementos multimedia de audio
Si el objeto `AudioSocket` está configurado para enviar elementos multimedia, el bot se debe registrar para el controlador de eventos `AudioSendStatusChanged` en el objeto `AudioSocket` para obtener notificaciones sobre los cambios en el estado del envío. El bot debería empezar a enviar audio solo después de que el estado del envío cambie a Activo.

```cs
void AudioSocket_OnSendStatusChanged(
             object sender,
             AudioSendStatusChangedEventArgs args)
{
    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

Para enviar elementos multimedia de audio, se supone que el contenido de audio de PCM está dentro de un búfer asignado por montón nativo. El bot debe derivar de la clase abstracta `AudioMediaBuffer`. Los elementos multimedia se envían asincrónicamente a través del método `Send` del objeto AudioSocket y la plataforma llamará a `Dispose` en `AudioMediaBuffer` cuando se complete el envío. El bot no debe liberar (ni devolverse a un asignador de grupos) los recursos no administrados cuando se devuelve `Send`. Debe esperar a que se llame a `Dispose`.

## <a name="send-video-media"></a>Envío de elementos multimedia de vídeo
Enviar elementos multimedia de vídeo es similar al envío de elementos multimedia de audio. El bot se debe registrar para el evento `VideoSendStatusChanged` y esperar que `MediaSendStatus` sea `Active`. El bot no debe liberar ni reclamar los recursos no administrados del búfer hasta que se llame al método `Dispose`. Se admiten los formatos de color RGB24, NV12 y Yuy2.

Cuando se admiten varios elementos `VideoFormat` para enviar vídeo, el elemento `VideoFormat` actualmente preferido se comunica al bot a través del evento `VideoSendStatusChanged`. El elemento `VideoFormat` preferido para enviar vídeo puede cambiar debido a las condiciones del ancho de banda de red o al cliente del mismo nivel que cambia de tamaño su ventana de vídeo.

```cs
void VideoSocket_OnSendStatusChanged(
            object sender,
            VideoSendStatusChangedEventArgs args)
{
    VideoFormat preferredVideoFormat;

    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        // bot is recommended to use this format for sourcing video content.
        preferredVideoFormat = args.PreferredVideoSourceFormat;
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

Cada `VideoMediaBuffer` tiene una propiedad VideoFormat para indicar el formato del contenido de vídeo de ese búfer en particular. Si bien no es necesario que la propiedad `VideoFormat` coincida con la propiedad `PreferredVideoSourceFormat` indicada en el evento `VideoSendStatusChanged`, se recomienda usar el elemento `PreferredVideoSourceFormat` indicado para evitar la pérdida de ciclos de CPU dedicados a cambiar el tamaño de los fotogramas de vídeo.

## <a name="call-notifications"></a>Notificaciones de llamada
### <a name="call-state-changes"></a>Cambios en el estado de las llamadas
El boto puede obtener notificaciones de cambio del estado de la llamada si se suscribe a `NotificationType.CallStateChange` en `NotificationSubscriptions` de `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow`.

```cs
private Task OnCallStateChangeNotification(CallStateChangeNotification callStateChangeNotification)
{
    if (callStateChangeNotification.CurrentState == CallState.Terminated)
    {   
        // stop sending media and dispose the media sockets
        _audioSocket.Dispose();
        _videoSocket.Dispose();
    }

    return Task.CompletedTask;
}
 ```

### <a name="roster-update"></a>Actualización de la lista
Si el blog se agrega a una conferencia, puede escuchar la lista de la conferencia si se suscribe a `NotificationType.RosterUpdate` en `NotificationSubscriptions` de `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow`. El elemento `RosterUpdateNotification` tiene los detalles de todos los participantes en la conferencia. El bot podría optar por esperar una notificación con un participante que envía vídeo y luego suscribirse `Subscribe` al vídeo del participante en uno de sus objetos `VideoSocket`.

```cs
private async Task OnRosterUpdateNotification(RosterUpdateNotification rosterUpdateNotification)
{
    // Find a video source in the conference to subscribe
    foreach (RosterParticipant participant in rosterUpdateNotification.Participants)
    {
        if (participant.MediaType == ModalityType.Video
            && (participant.MediaStreamDirection == "sendonly" || participant.MediaStreamDirection == "sendrecv")
            )
        {
            var videoSubscription = new VideoSubscription
            {
                ParticipantIdentity = participant.Identity,
                OperationId = Guid.NewGuid().ToString(),
                SocketId = _videoSocket.SocketId,
                VideoModality = ModalityType.Video,
                VideoResolution = ResolutionFormat.Hd720p
            };

            await CallService.Subscribe(videoSubscription).ConfigureAwait(false);
            break;
        }
    }  
}
```

## <a name="end-the-call"></a>Finalización de la llamada

### <a name="handle-call-termination-from-the-caller"></a>Control de la finalización de la llamada desde el autor de la llamada
Cuando el usuario desconecta la llamada al bot en una conversación de punto a punto o cuando el bot se quita de la conferencia, el bot recibe una notificación de cambio de estado de la llamada con el objeto CallState como Terminated (Finalizado). El bot debería eliminar los sockets multimedia que creó.

### <a name="terminate-the-call-from-the-bot"></a>Finalización de la llamada desde el bot
El bot puede elegir finalizar la llamada si se llama a `EndCall` en `IRealTimeMediaCallService`.

### <a name="handle-call-clean-up-by-the-bot-framework"></a>Control de la limpieza de llamada por parte de Bot Framework
En condiciones de error (por ejemplo, si `AnswerAppHostedMediaOutcomeEvent` no se recibe dentro de un plazo razonable), Bot Framework puede finalizar la llamada. El bot se debe registrar para el evento `OnCallCleanup` y eliminar los sockets multimedia.

