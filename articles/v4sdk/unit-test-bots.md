---
title: Pruebas unitarias de bots | Microsoft Docs
description: Describe cómo realizar pruebas unitarias de bots mediante plataformas de pruebas.
keywords: bot, testing bots, bot testing framework
author: gabog
ms.author: ggilaber
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1e9d079b46c1cc4cc8c49e234b58540aeb4b2e7c
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298983"
---
# <a name="how-to-unit-test-bots"></a>Cómo hacer pruebas unitarias de bots

[!INCLUDE[applies-to](../includes/applies-to.md)]

En este tema se mostrará cómo:

- Crear pruebas unitarias para bots.
- Usar aserciones para comprobar si las actividades devueltas por un diálogo se encuentran dentro de los valores esperados.
- Usar aserciones para comprobar los resultados devueltos por un diálogo.
- Crear diferentes tipos de pruebas controladas por datos.
- Crear objetos ficticios para las diferentes dependencias de un diálogo (es decir, reconocedores de LUIS, etc.).

## <a name="prerequisites"></a>Requisitos previos

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

El ejemplo [CoreBot Tests](https://aka.ms/cs-core-test-sample) que se usa en este tema utiliza el paquete [Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/), [XUnit](https://xunit.net/) y [Moq](https://github.com/moq/moq) para crear pruebas unitarias.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El ejemplo [CoreBot Tests](https://aka.ms/js-core-test-sample) que se usa en este tema utiliza el paquete [botbuilder-testing](https://www.npmjs.com/package/botbuilder-testing) y [Mocha](https://mochajs.org/) para crear pruebas unitarias, y [Mocha Test Explorer](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter) para ver los resultados de las pruebas en VS Code.

---

## <a name="testing-dialogs"></a>Pruebas de diálogos

En el ejemplo CoreBot, las pruebas unitarias de los diálogos se realizan con la clase `DialogTestClient`, que proporciona un mecanismo para probarlos de forma aislada fuera de un bot y sin tener que implementar el código en un servicio web.

Con esta clase, puede escribir pruebas unitarias que validen las respuestas de los diálogos por turnos. Las pruebas unitarias que usan la clase `DialogTestClient` deben funcionar con otros diálogos creados con la biblioteca de diálogos de botbuilder.

En el ejemplo siguiente se muestran las pruebas derivadas de `DialogTestClient`:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut);

var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("Seattle");
Assert.Equal("Where are you traveling from?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("New York");
Assert.Equal("When would you like to travel?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("tomorrow");
Assert.Equal("OK, I will book a flight from Seattle to New York for tomorrow, Is this Correct?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("yes");
Assert.Equal("Sure thing, wait while I finalize your reservation...", reply.Text);

reply = testClient.GetNextReply<IMessageActivity>();
Assert.Equal("All set, I have booked your flight to Seattle for tomorrow", reply.Text);
```

La clase `DialogTestClient` se define en el espacio de nombres `Microsoft.Bot.Builder.Testing` y se incluye en el paquete de NuGet [Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/).

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut);

let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');

reply = await testClient.sendActivity('Seattle');
assert.strictEqual(reply.text, 'Where are you traveling from?');

reply = await testClient.sendActivity('New York');
assert.strictEqual(reply.text, 'When would you like to travel?');

reply = await testClient.sendActivity('tomorrow');
assert.strictEqual(reply.text, 'OK, I will book a flight from Seattle to New York for tomorrow, Is this Correct?');

reply = await testClient.sendActivity('yes');
assert.strictEqual(reply.text, 'Sure thing, wait while I finalize your reservation...');

reply = testClient.getNextReply();
assert.strictEqual(reply.text, 'All set, I have booked your flight to Seattle for tomorrow');
```

La clase `DialogTestClient` se incluye en el paquete de npm [botbuilder-testing]().

---

### <a name="dialogtestclient"></a>DialogTestClient

El primer parámetro de `DialogTestClient` es el canal de destino. Esto le permite probar una lógica de representación diferente en función del canal de destino del bot (Teams, Slack, Cortana, etc.). Si no está seguro de cuál es su canal de destino, puede usar los identificadores de canal `Emulator` o `Test`, pero tenga en cuenta que algunos componentes se comportarán de forma diferente en función del canal actual; por ejemplo, `ConfirmPrompt` representa las opciones sí/no de forma diferente para los canales `Test` y `Emulator`. También puede usar este parámetro para probar la lógica de representación condicional en el diálogo en función del identificador de canal.

El segundo parámetro es una instancia del diálogo que se está probando. (Nota: **"sut"** significa "sistema en pruebas"; usamos este acrónimo en los fragmentos de código de este artículo).

El constructor `DialogTestClient` proporciona parámetros adicionales que le permiten personalizar aún más el comportamiento del cliente o pasar parámetros al diálogo que se está probando, si es necesario. Puede pasar los datos de inicialización del diálogo, agregar middleware personalizado o usar su propio TestAdapter y su propia instancia de `ConversationState`.

### <a name="sending-and-receiving-messages"></a>Envío y recepción de mensajes

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

El método `SendActivityAsync<IActivity>` permite enviar una expresión de texto o un `IActivity` al diálogo y devuelve el primer mensaje que recibe. El parámetro `<T>` se usa para devolver una instancia fuertemente tipada de la respuesta para que pueda validarla sin tener que convertirla.

```csharp
var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);
```

En algunos escenarios, el bot puede enviar varios mensajes en respuesta a una sola actividad; en estos casos, `DialogTestClient` pondrá en cola las respuestas y puede usar el método `GetNextReply<IActivity>` para extraer el siguiente mensaje de la cola de respuesta.

```csharp
reply = testClient.GetNextReply<IMessageActivity>();
Assert.Equal("All set, I have booked your flight to Seattle for tomorrow", reply.Text);
```

`GetNextReply<IActivity>` devolverá null si no hay más mensajes en la cola de respuesta.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El método `sendActivity` permite enviar una expresión de texto o un `Activity` al diálogo y devuelve el primer mensaje que recibe.

```javascript
let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');
```

En algunos escenarios, el bot puede enviar varios mensajes en respuesta a una sola actividad; en estos casos, `DialogTestClient` pondrá en cola las respuestas y puede usar el método `getNextReply` para extraer el siguiente mensaje de la cola de respuesta.

```javascript
reply = testClient.getNextReply();
assert.strictEqual(reply.text, 'All set, I have booked your flight to Seattle for tomorrow');
```

`getNextReply` devolverá null si no hay más mensajes en la cola de respuesta.

---

### <a name="asserting-activities"></a>Aserción de actividades

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

El código del ejemplo CoreBot solo valida la propiedad `Text` de las actividades devueltas. En bots más complejos, es posible que desee validar otras propiedades, como `Speak`, `InputHint` o `ChannelData`, por ejemplo.

```csharp
Assert.Equal("Sure thing, wait while I finalize your reservation...", reply.Text);
Assert.Equal("One moment please...", reply.Speak);
Assert.Equal(InputHints.IgnoringInput, reply.InputHint);
```

Para ello, puede comprobar cada propiedad individualmente como se muestra arriba, puede escribir sus propias utilidades auxiliares para la aserción de actividades o puede usar otras plataformas, como [FluentAssertions](https://fluentassertions.com/), para escribir aserciones personalizadas y simplificar el código de prueba.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El código del ejemplo CoreBot solo valida la propiedad `text` de las actividades devueltas. En bots más complejos, es posible que desee validar otras propiedades, como `speak`, `inputHint` o `channelData`, por ejemplo.

```javascript
assert.strictEqual(reply.text, 'Sure thing, wait while I finalize your reservation...');
assert.strictEqual(reply.speak, 'One moment please...');
assert.strictEqual(reply.inputHint, InputHints.IgnoringInput);
```

Para ello, puede comprobar cada propiedad individualmente como se muestra arriba, puede escribir sus propias utilidades auxiliares para la aserción de actividades o puede usar otras bibliotecas, como [Chai](https://www.chaijs.com/), para escribir aserciones personalizadas y simplificar el código de prueba.

---

### <a name="passing-parameters-to-your-dialogs"></a>Paso de parámetros a los diálogos

El constructor `DialogTestClient` tiene `initialDialogOptions` que se puede usar para pasar parámetros al diálogo. Por ejemplo, en este ejemplo, `MainDialog` inicializa un objeto `BookingDetails` a partir de los resultados de LUIS con las entidades que resuelve a partir de la expresión del usuario, y pasa este objeto en la llamada a `BookingDialog`.

Puede implementarlo en una prueba de la siguiente manera:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var inputDialogParams = new BookingDetails()
{
    Destination = "Seattle",
    TravelDate = $"{DateTime.UtcNow.AddDays(1):yyyy-MM-dd}"
};

var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut, inputDialogParams);

```

`BookingDialog` recibe este parámetro y accede a él en la prueba de la misma manera que si se hubiera invocado desde `MainDialog`.

```csharp
private async Task<DialogTurnResult> DestinationStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    var bookingDetails = (BookingDetails)stepContext.Options;
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const inputDialogParams = {
    destination: 'Seattle',
    travelDate: formatDate(new Date().setDate(now.getDate() + 1))
};

const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut, inputDialogParams);
```

`BookingDialog` recibe este parámetro y puede acceder a él en la prueba de la misma manera que si se hubiera invocado desde `MainDialog`.

```javascript
async destinationStep(stepContext) {
    const bookingDetails = stepContext.options;
    ...
}
```

---

### <a name="asserting-dialog-turn-results"></a>Aserción de los resultados del turno de diálogo

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Algunos diálogos, como `BookingDialog` o `DateResolverDialog`, devuelven un valor al diálogo que realiza la llamada. El objeto `DialogTestClient` expone una propiedad `DialogTurnResult` que se puede utilizar para analizar y validar los resultados devueltos por el diálogo.

Por ejemplo:

```csharp
var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut);

var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);

...

var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
Assert.Equal("New York", bookingResults?.Origin);
Assert.Equal("Seattle", bookingResults?.Destination);
Assert.Equal("2019-06-21", bookingResults?.TravelDate);
```

La propiedad `DialogTurnResult` también se puede utilizar para inspeccionar y validar los resultados intermedios devueltos por los pasos de una cascada.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Algunos diálogos, como `BookingDialog` o `DateResolverDialog`, devuelven un valor al diálogo que realiza la llamada. El objeto `DialogTestClient` expone una propiedad `dialogTurnResult` que se puede utilizar para analizar y validar los resultados devueltos por el diálogo.

Por ejemplo:

```javascript
const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut);

let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');

...

const bookingResults = client.dialogTurnResult.result;
assert.strictEqual('New York', bookingResults.destination);
assert.strictEqual('Seattle', bookingResults.origin);
assert.strictEqual('2019-06-21', bookingResults.travelDate);
```

La propiedad `dialogTurnResult` también se puede utilizar para inspeccionar y validar los resultados intermedios devueltos por los pasos de una cascada.

---

### <a name="analyzing-test-output"></a>Análisis de la salida de la prueba

A veces es necesario leer una transcripción de la prueba unitaria para analizar la ejecución sin tener que depurar la prueba.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

El paquete [Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/) incluye `XUnitDialogTestLogger`, que registra los mensajes enviados y recibidos por el diálogo en la consola.

Para usar este middleware, la prueba debe exponer un constructor que recibe un objeto `ITestOutputHelper` proporcionado por el ejecutor de pruebas XUnit, y crear un `XUnitDialogTestLogger` que se pasará a `DialogTestClient` en el parámetro `middlewares`.

```csharp
public class BookingDialogTests
{
    private readonly IMiddleware[] _middlewares;

    public BookingDialogTests(ITestOutputHelper output)
        : base(output)
    {
        _middlewares = new[] { new XUnitDialogTestLogger(output) };
    }

    [Fact]
    public async Task SomeBookingDialogTest()
    {
        // Arrange
        var sut = new BookingDialog();
        var testClient = new DialogTestClient(Channels.Msteams, sut, middlewares: _middlewares);

        ...
    }
}
```

Este es un ejemplo de lo que `XUnitDialogTestLogger` registra en la ventana de salida cuando se configura:

![XUnitMiddlewareOutput](media/how-to-unit-test/cs/XUnitMiddlewareOutput.png)

Para más información sobre cómo enviar la salida de la prueba a la consola cuando se usa XUnit, consulte [Captura de la salida](https://xunit.net/docs/capturing-output.html) en la documentación de XUnit.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

El paquete [botbuilder-testing](https://www.npmjs.com/package/botbuilder-testing) incluye `DialogTestLogger`, que registra los mensajes enviados y recibidos por el diálogo en la consola.

Para usar este middleware, simplemente páselo a `DialogTestClient` en el parámetro `middlewares`.

```javascript
const client = new DialogTestClient('msteams', sut, testData.initialData, [new DialogTestLogger()]);
```

Este es un ejemplo de lo que `DialogTestLogger` registra en la ventana de salida cuando se configura:

![DialogTestLoggerOutput](media/how-to-unit-test/js/DialogTestLoggerOutput.png)

---

Esta salida también se registrará en el servidor de compilación durante las compilaciones de integración continua, y le ayudará a analizar los errores de compilación.

## <a name="data-driven-tests"></a>Pruebas controladas por datos

En la mayoría de los casos, la lógica de los diálogos no cambia y las distintas rutas de ejecución de una conversación se basan en expresiones del usuario. En lugar de escribir una prueba unitaria única para cada variante de la conversación, es más fácil usar pruebas controladas por datos (también conocidas como pruebas parametrizadas).

En la prueba de ejemplo de la sección Información general de este documento se muestra cómo probar un flujo de ejecución, pero ¿qué sucede si el usuario responde no a la confirmación? o ¿qué ocurre si usan una fecha diferente?, por ejemplo.

Las pruebas controladas por datos nos permiten probar todas estas permutaciones sin tener que volver a escribir las pruebas.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

En el ejemplo CoreBot, usamos pruebas `Theory` de XUnit para parametrizar las pruebas.

### <a name="theory-tests-using-inlinedata"></a>Pruebas teóricas mediante InlineData

La prueba siguiente comprueba que un diálogo se cancela cuando el usuario dice "Cancelar".

```csharp
[Fact]
public async Task ShouldBeAbleToCancel()
{
    var sut = new TestCancelAndHelpDialog();
    var testClient = new DialogTestClient(Channels.Test, sut);

    var reply = await testClient.SendActivityAsync<IMessageActivity>("Hi");
    Assert.Equal("Hi there", reply.Text);
    Assert.Equal(DialogTurnStatus.Waiting, testClient.DialogTurnResult.Status);

    reply = await testClient.SendActivityAsync<IMessageActivity>("cancel");
    Assert.Equal("Cancelling...", reply.Text);
}
```

Para cancelar un diálogo, los usuarios pueden escribir "salir", "déjalo" y "para". En lugar de escribir un nuevo caso de prueba para cada palabra posible, escriba un único método de prueba `Theory` que acepte parámetros de una lista de valores `InlineData` para definir los parámetros para cada caso de prueba:

```csharp
[Theory]
[InlineData("cancel")]
[InlineData("quit")]
[InlineData("never mind")]
[InlineData("stop it")]
public async Task ShouldBeAbleToCancel(string cancelUtterance)
{
    var sut = new TestCancelAndHelpDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, middlewares: _middlewares);

    var reply = await testClient.SendActivityAsync<IMessageActivity>("Hi");
    Assert.Equal("Hi there", reply.Text);
    Assert.Equal(DialogTurnStatus.Waiting, testClient.DialogTurnResult.Status);

    reply = await testClient.SendActivityAsync<IMessageActivity>(cancelUtterance);
    Assert.Equal("Cancelling...", reply.Text);
}
```

La nueva prueba se ejecutará cuatro veces con los distintos parámetros y cada caso se mostrará como un elemento secundario en la prueba `ShouldBeAbleToCancel`, en el explorador de pruebas de Visual Studio. Si se produce un error en cualquiera de ellos, tal y como se muestra a continuación, puede hacer clic con el botón derecho y depurar el escenario en el que se produjo un error en lugar de volver a ejecutar todo el conjunto de pruebas.

![InlineDataTestResults](media/how-to-unit-test/cs/InlineDataTestResults.png)

### <a name="theory-tests-using-memberdata-and-complex-types"></a>Pruebas teóricas mediante MemberData y tipos complejos

`InlineData` es útil para realizar pruebas pequeñas controladas por datos que reciben parámetros de tipo de valor simple (String, int, etc.).

`BookingDialog` recibe un objeto `BookingDetails` y devuelve un nuevo objeto `BookingDetails`. La versión sin parámetros de una prueba para este diálogo tendría el siguiente aspecto:

```csharp
[Fact]
public async Task DialogFlow()
{
    // Initial parameters
    var initialBookingDetails = new BookingDetails
    {
        Origin = "Seattle",
        Destination = null,
        TravelDate = null,
    };

    // Expected booking details
    var expectedBookingDetails = new BookingDetails
    {
        Origin = "Seattle",
        Destination = "New York",
        TravelDate = "2019-06-25",
    };

    var sut = new BookingDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, initialBookingDetails);

    // Act/Assert
    var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
    ...

    var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
    Assert.Equal(expectedBookingDetails.Origin, bookingResults?.Origin);
    Assert.Equal(expectedBookingDetails.Destination, bookingResults?.Destination);
    Assert.Equal(expectedBookingDetails.TravelDate, bookingResults?.TravelDate);
}
```

Para parametrizar esta prueba, creamos una clase `BookingDialogTestCase` que contiene los datos de los casos de prueba. Contiene el objeto `BookingDetails` inicial, el `BookingDetails` esperado y una matriz de cadenas que contienen las expresiones enviadas por el usuario y las respuestas que se espera del diálogo en cada turno.

```csharp
public class BookingDialogTestCase
{
    public BookingDetails InitialBookingDetails { get; set; }

    public string[,] UtterancesAndReplies { get; set; }

    public BookingDetails ExpectedBookingDetails { get; set; }
}
```

También hemos creado una clase auxiliar `BookingDialogTestsDataGenerator` que expone un método `IEnumerable<object[]> BookingFlows()` que devuelve una colección de los casos que se van a usar en la prueba.

Para mostrar cada caso de prueba como un elemento independiente en el explorador de pruebas de Visual Studio, el ejecutor de pruebas XUnit requiere que los tipos complejos como `BookingDialogTestCase` implementen `IXunitSerializable`. Para simplificar esto, la plataforma Bot.Builder.Testing proporciona una clase `TestDataObject` que implementa esta interfaz y que se puede usar para encapsular los datos del caso sin tener que implementar `IXunitSerializable`. 

Este es un fragmento de `IEnumerable<object[]> BookingFlows()` que muestra cómo se usan las dos clases:

```csharp
public static class BookingDialogTestsDataGenerator
{
    public static IEnumerable<object[]> BookingFlows()
    {
        // Create the first test case object
        var testCaseData = new BookingDialogTestCase
        {
            InitialBookingDetails = new BookingDetails(),
            UtterancesAndReplies = new[,]
            {
                { "hi", "Where would you like to travel to?" },
                { "Seattle", "Where are you traveling from?" },
                { "New York", "When would you like to travel?" },
                { "tomorrow", $"Please confirm, I have you traveling to: Seattle from: New York on: {DateTime.Now.AddDays(1):yyyy-MM-dd}. Is this correct? (1) Yes or (2) No" },
                { "yes", null },
            },
            ExpectedBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = $"{DateTime.Now.AddDays(1):yyyy-MM-dd}",
            }, 
        };
        // wrap the test case object into TestDataObject and return it.
        yield return new object[] { new TestDataObject(testCaseData) };

        // Create the second test case object
        testCaseData = new BookingDialogTestCase
        {
            InitialBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = null,
            },
            UtterancesAndReplies = new[,]
            {
                { "hi", "When would you like to travel?" },
                { "tomorrow", $"Please confirm, I have you traveling to: Seattle from: New York on: {DateTime.Now.AddDays(1):yyyy-MM-dd}. Is this correct? (1) Yes or (2) No" },
                { "yes", null },
            },
            ExpectedBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = $"{DateTime.Now.AddDays(1):yyyy-MM-dd}",
            },
        };
        // wrap the test case object into TestDataObject and return it.
        yield return new object[] { new TestDataObject(testCaseData) };
    }
}
```

Una vez que se crea un objeto para almacenar los datos de prueba y una clase que expone una colección de casos de prueba, usamos el atributo `MemberData` de XUnit en lugar de `InlineData` para insertar los datos en la prueba. El primer parámetro de `MemberData` es el nombre de la función estática que devuelve la colección de casos de prueba, y el segundo parámetro es el tipo de la clase que expone este método.

```csharp
[Theory]
[MemberData(nameof(BookingDialogTestsDataGenerator.BookingFlows), MemberType = typeof(BookingDialogTestsDataGenerator))]
public async Task DialogFlowUseCases(TestDataObject testData)
{
    // Get the test data instance from TestDataObject
    var bookingTestData = testData.GetObject<BookingDialogTestCase>();
    var sut = new BookingDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, bookingTestData.InitialBookingDetails);

    // Iterate over the utterances and replies array.
    for (var i = 0; i < bookingTestData.UtterancesAndReplies.GetLength(0); i++)
    {
        var reply = await testClient.SendActivityAsync<IMessageActivity>(bookingTestData.UtterancesAndReplies[i, 0]);
        Assert.Equal(bookingTestData.UtterancesAndReplies[i, 1], reply?.Text);
    }

    // Assert the resulting BookingDetails object
    var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.Origin, bookingResults?.Origin);
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.Destination, bookingResults?.Destination);
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.TravelDate, bookingResults?.TravelDate);
}
```

Este es un ejemplo de los resultados de las pruebas `DialogFlowUseCases` en el explorador de pruebas de Visual Studio cuando se ejecuta la prueba:

![BookingDialogTests](media/how-to-unit-test/cs/BookingDialogTestsResults.png)

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="simple-data-driven-tests"></a>Pruebas simples controladas por datos

La prueba siguiente comprueba que un diálogo se cancela cuando el usuario dice "Cancelar".

```javascript
describe('ShouldBeAbleToCancel', () => {
    it('Should cancel', async () => {
        const sut = new TestCancelAndHelpDialog();
        const client = new DialogTestClient('test', sut, null, [new DialogTestLogger()]);

        // Execute the test case
        let reply = await client.sendActivity('Hi');
        assert.strictEqual(reply.text, 'Hi there');
        assert.strictEqual(client.dialogTurnResult.status, 'waiting');

        reply = await client.sendActivity('cancel');
        assert.strictEqual(reply.text, 'Cancelling...');
    });
});
```

Tenga en cuenta que, más adelante, debemos ser capaces de controlar otras expresiones para cancelar como "salir", "déjalo" y "para". En lugar de escribir tres pruebas más repetitivas para cada nueva expresión, podemos refactorizar la prueba para usar una lista de expresiones para definir los parámetros de cada caso de prueba:

```javascript
describe('ShouldBeAbleToCancel', () => {
    const testCases = ['cancel', 'quit', 'never mind', 'stop it'];

    testCases.map(testData => {
        it(testData, async () => {
            const sut = new TestCancelAndHelpDialog();
            const client = new DialogTestClient('test', sut, null, [new DialogTestLogger()]);

            // Execute the test case
            let reply = await client.sendActivity('Hi');
            assert.strictEqual(reply.text, 'Hi there');
            assert.strictEqual(client.dialogTurnResult.status, 'waiting');

            reply = await client.sendActivity(testData);
            assert.strictEqual(reply.text, 'Cancelling...');
        });
    });
});
```

La nueva prueba se ejecutará cuatro veces con los distintos parámetros y cada caso se mostrará como un elemento secundario en el conjunto de pruebas `ShouldBeAbleToCancel`, en [Mocha Test Explorer](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter). Si se produce un error en cualquiera de ellos, tal y como se muestra a continuación, puede depurar el escenario en el que se produjo un error en lugar de volver a ejecutar todo el conjunto de pruebas.

![SimpleCancelTestResults](media/how-to-unit-test/js/SimpleCancelTestResults.png)

### <a name="data-driven-tests-with-complex-types"></a>Pruebas controladas por datos con tipos complejos

Utilizar una lista sencilla de expresiones resulta útil para realizar pruebas pequeñas controladas por datos que reciben parámetros de tipo de valor simple (string, int, etc.) u objetos pequeños.

`BookingDialog` recibe un objeto `BookingDetails` y devuelve un nuevo objeto `BookingDetails`. La versión sin parámetros de una prueba para este diálogo tendría el siguiente aspecto:

```javascript
describe('BookingDialog', () => {
    it('Returns expected booking details', async () => {
        // Initial parameters
        const initialBookingDetails = {
            origin: 'Seattle',
            destination: undefined,
            travelDate: undefined
        };

        // Expected booking details
        const expectedBookingDetails = {
            origin: 'Seattle',
            destination: 'New York',
            travelDate: '2019-06-25'
        };

        const sut = new BookingDialog('bookingDialog');
        const client = new DialogTestClient('test', sut, initialBookingDetails, [new DialogTestLogger()]);

        // Execute the test case
        const reply = await client.sendActivity('Hi');
        ...

        // Check dialog results
        const result = client.dialogTurnResult.result;
        assert.strictEqual(result.destination, expectedBookingDetails.destination);
        assert.strictEqual(result.origin, expectedBookingDetails.origin);
        assert.strictEqual(result.travelDate, expectedBookingDetails.travelDate);
    });
});
```

Para parametrizar esta prueba, creamos un módulo `bookingDialogTestCases` que devuelve los datos de los casos de prueba. Cada elemento contiene el nombre del caso de prueba, el objeto "BookingDetails" inicial, el objeto "BookingDetails" esperado y una matriz de cadenas con las expresiones enviadas por el usuario y las respuestas que se espera del diálogo en cada turno.

```javascript
module.exports = [
    // Create the first test case object
    {
        name: 'Full flow',
        initialData: {},
        steps: [
            ['hi', 'To what city would you like to travel?'],
            ['Seattle', 'From what city will you be travelling?'],
            ['New York', 'On what date would you like to travel?'],
            ['tomorrow', `Please confirm, I have you traveling to: Seattle from: New York on: ${ tomorrow }. Is this correct? (1) Yes or (2) No`],
            ['yes', null]
        ],
        expectedResult: {
            destination: 'Seattle',
            origin: 'New York',
            travelDate: tomorrow
        }
    },
    // Create the second test case object
    {
        name: 'Destination and Origin provided',
        initialData: {
            destination: 'Seattle',
            origin: 'New York'
        },
        steps: [
            ['hi', 'On what date would you like to travel?'],
            ['tomorrow', `Please confirm, I have you traveling to: Seattle from: New York on: ${ tomorrow }. Is this correct? (1) Yes or (2) No`],
            ['yes', null]
        ],
        expectedStatus: 'complete',
        expectedResult: {
            destination: 'Seattle',
            origin: 'New York',
            travelDate: tomorrow
        }
    }
];
```

Una vez creada la lista que contiene los datos de prueba, podemos refactorizar nuestra prueba para asignar esta lista a casos de prueba individuales.

```javascript
describe('DialogFlowUseCases', () => {
    const testCases = require('./testData/bookingDialogTestCases.js');

    testCases.map(testData => {
        it(testData.name, async () => {
            const sut = new BookingDialog('bookingDialog');
            const client = new DialogTestClient('test', sut, testData.initialData, [new DialogTestLogger()]);

            // Execute the test case
            for (let i = 0; i < testData.steps.length; i++) {
                const reply = await client.sendActivity(testData.steps[i][0]);
                assert.strictEqual((reply ? reply.text : null), testData.steps[i][1]);
            }

            // Check dialog results
            const actualResult = client.dialogTurnResult.result;
            assert.strictEqual(actualResult.destination, testData.expectedResult.destination);
            assert.strictEqual(actualResult.origin, testData.expectedResult.origin);
            assert.strictEqual(actualResult.travelDate, testData.expectedResult.travelDate);
        });
    });
});
```

Este es un ejemplo de los resultados del conjunto de pruebas `DialogFlowUseCases` en Mocha Test Explorer cuando se ejecuta el conjunto de pruebas:

![BookingDialogTests](media/how-to-unit-test/js/BookingDialogTestsResults.png)

---

## <a name="using-mocks"></a>Uso de elementos ficticios

Puede usar elementos ficticios para los aspectos que no está probando actualmente. Como referencia, este nivel puede considerarse generalmente como unidad y prueba de integración.

La simulación de tantos elementos como pueda permite un mejor aislamiento de la pieza que está probando. Los candidatos para los elementos ficticios incluyen el almacenamiento, el adaptador, el software intermedio, la canalización de actividades, los canales y cualquier otra cosa que no forme parte del bot directamente. También se podrían quitar ciertos aspectos temporalmente, como el software intermedio no implicado en la parte del bot que está probando, para aislar cada fragmento. Sin embargo, si va a probar su software intermedio, es posible que quiera simular el bot en su lugar.

La simulación de elementos puede asumir formas diferentes, desde el reemplazo de un elemento por otro objeto conocido a la implementación de una funcionalidad básica de Hola mundo. También puede consistir en la eliminación del elemento simplemente, en caso de que no sea necesario, así como en forzarlo a no hacer nada.

Los elementos ficticios nos permiten configurar las dependencias de un diálogo y asegurarnos de que su estado es conocido durante la ejecución de la prueba sin tener que depender de recursos externos como bases de datos, modelos LUIS u otros objetos.

Para facilitar la prueba del diálogo y reducir las dependencias de objetos externos, es posible que tenga que insertar las dependencias externas en el constructor de diálogos.

Por ejemplo, en lugar de crear instancias de `BookingDialog` en `MainDialog`:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public MainDialog()
    : base(nameof(MainDialog))
{
    ...
    AddDialog(new BookingDialog());
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor() {
    super('MainDialog');
    ...
    this.addDialog(new BookingDialog('bookingDialog'));
    ...
}
```

---

Pasamos una instancia de `BookingDialog` como un parámetro de constructor:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public MainDialog(BookingDialog bookingDialog)
    : base(nameof(MainDialog))
{
    ...
    AddDialog(bookingDialog);
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(bookingDialog) {
    super('MainDialog');
    ...
    this.addDialog(bookingDialog);
    ...
}
```

---

Esto nos permite reemplazar la instancia `BookingDialog` con un objeto ficticio y escribir pruebas unitarias para `MainDialog` sin tener que llamar a la clase `BookingDialog` real.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Create the mock object
var mockDialog = new Mock<BookingDialog>();

// Use the mock object to instantiate MainDialog
var sut = new MainDialog(mockDialog.Object);

var testClient = new DialogTestClient(Channels.Test, sut);
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Create the mock object
const mockDialog = new MockBookingDialog();

// Use the mock object to instantiate MainDialog
const sut = new MainDialog(mockDialog);

const testClient = new DialogTestClient('test', sut);
```

---

### <a name="mocking-dialogs"></a>Diálogos ficticios

Como se describió anteriormente, `MainDialog` invoca a `BookingDialog` para obtener el objeto `BookingDetails`. Implementamos y configuramos una instancia ficticia de `BookingDialog` de la siguiente manera:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Create the mock object for BookingDialog.
var mockDialog = new Mock<BookingDialog>();
mockDialog
    .Setup(x => x.BeginDialogAsync(It.IsAny<DialogContext>(), It.IsAny<object>(), It.IsAny<CancellationToken>()))
    .Returns(async (DialogContext dialogContext, object options, CancellationToken cancellationToken) =>
    {
        // Send a generic activity so we can assert that the dialog was invoked.
        await dialogContext.Context.SendActivityAsync($"{mockDialogNameTypeName} mock invoked", cancellationToken: cancellationToken);

        // Create the BookingDetails instance we want the mock object to return.
        var expectedBookingDialogResult = new BookingDetails()
        {
            Destination = "Seattle",
            Origin = "New York",
            TravelDate = $"{DateTime.UtcNow.AddDays(1):yyyy-MM-dd}"
        };

        // Return the BookingDetails we need without executing the dialog logic.
        return await dialogContext.EndDialogAsync(expectedBookingDialogResult, cancellationToken);
    });

// Create the sut (System Under Test) using the mock booking dialog.
var sut = new MainDialog(mockDialog.Object);
```

En este ejemplo, usamos [Moq](https://github.com/moq/moq) para crear el diálogo ficticio, y los métodos `Setup` y `Returns` para configurar su comportamiento.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
class MockBookingDialog extends BookingDialog {
    constructor() {
        super('bookingDialog');
    }

    async beginDialog(dc, options) {
        // Send a generic activity so we can assert that the dialog was invoked.
        await dc.context.sendActivity(`${ this.id } mock invoked`);

        // Create the BookingDetails instance we want the mock object to return.
        const bookingDetails = {
            origin: 'New York',
            destination: 'Seattle',
            travelDate: '2025-07-08'
        };

        // Return the BookingDetails we need without executing the dialog logic.
        return await dc.endDialog(bookingDetails);
    }
}
...
// Create the sut (System Under Test) using the mock booking dialog.
const sut = new MainDialog(new MockBookingDialog());
...

```

En este ejemplo, para implementar el diálogo ficticio, derivamos de `BookingDialog` e invalidamos el método `beginDialog` para omitir la lógica de diálogo subyacente.

---

### <a name="mocking-luis-results"></a>Resultados de LUIS ficticios

En escenarios sencillos, puede implementar resultados de LUIS ficticios mediante código de la siguiente manera:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var mockRecognizer = new Mock<IRecognizer>();
mockRecognizer
    .Setup(x => x.RecognizeAsync<FlightBooking>(It.IsAny<ITurnContext>(), It.IsAny<CancellationToken>()))
    .Returns(() =>
    {
        var luisResult = new FlightBooking
        {
            Intents = new Dictionary<FlightBooking.Intent, IntentScore>
            {
                { FlightBooking.Intent.BookFlight, new IntentScore() { Score = 1 } },
            },
            Entities = new FlightBooking._Entities(),
        };
        return Task.FromResult(luisResult);
    });
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript

// Create a mock class for the recognizer that overrides executeLuisQuery.
class MockFlightBookingRecognizer extends FlightBookingRecognizer {
    constructor(mockResult) {
        super();
        this.mockResult = mockResult;
    }

    async executeLuisQuery(context) {
        return this.mockResult;
    }
}
...
// Create a mock result from a string
const mockLuisResult = JSON.parse(`{"intents": {"BookFlight": {"score": 1}}, "entities": {"$instance": {}}}`);
// Use the mock result with the mock recognizer.
const mockRecognizer = new MockFlightBookingRecognizer(mockLuisResult);
...
```

---

Pero los resultados de LUIS pueden ser complejos y, cuando lo son, es más fácil capturar el resultado en un archivo JSON, agregarlo como un recurso al proyecto y deserializarlo en un resultado de LUIS. Este es un ejemplo:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var mockRecognizer = new Mock<IRecognizer>();
mockRecognizer
    .Setup(x => x.RecognizeAsync<FlightBooking>(It.IsAny<ITurnContext>(), It.IsAny<CancellationToken>()))
    .Returns(() =>
    {
        // Deserialize the LUIS result from embedded json file in the TestData folder.
        var bookingResult = GetEmbeddedTestData($"{GetType().Namespace}.TestData.FlightToMadrid.json");

        // Return the deserialized LUIS result.
        return Task.FromResult(bookingResult);
    });
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Create a mock result from a json file
const mockLuisResult = require(`./testData/FlightToMadrid.json`);
// Use the mock result with the mock recognizer.
const mockRecognizer = new MockFlightBookingRecognizer(mockLuisResult);
```

---

## <a name="additional-information"></a>Información adicional

- [Ejemplo de prueba CoreBot (C#)](https://aka.ms/cs-core-test-sample)
- [Ejemplo de prueba CoreBot (JavaScript)](https://aka.ms/js-core-test-sample)
- [Pruebas de bots](https://github.com/microsoft/botframework-sdk/blob/master/specs/testing/testing.md)
