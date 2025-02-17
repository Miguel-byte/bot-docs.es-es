---
title: Referencia de la Interfaz de la línea de comandos (CLI) de Azure Bot Framework | Microsoft Docs
description: Obtenga información sobre la interfaz de la línea de comandos (CLI) de Bot Framework.
keywords: Interfaz de la línea de comandos de Bot Framework, CLI de Bot Framework
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/25/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b516f20e74ac364a1ec0e953983701d1ab4b1e33
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/02/2019
ms.locfileid: "73443153"
---
# <a name="bot-framework-cli-reference"></a>Referencia de la CLI de Bot Framework

[!INCLUDE[applies-to](../includes/applies-to.md)]

Uso

```sh-session
npm install -g @microsoft/botframework-cli
```

## <a name="commands"></a>Comandos:

* [`bf chatdown`](#bf-chatdown)
* [`bf chatdown:convert`](#bf-chatdownconvert)
* [`bf config`](#bf-config)
* [`bf config:set:qnamaker`](#bf-configsetqnamaker)
* [`bf config:set:telemetry`](#bf-configsettelemetry)
* [`bf config:show`](#bf-configshow)
* [`bf config:show:qnamaker`](#bf-configshowqnamaker)
* [`bf config:show:telemetry`](#bf-configshowtelemetry)
* [`bf help [COMMAND]`](#bf-help-command)
* [`bf luis`](#bf-luis)
* [`bf luis:convert`](#bf-luisconvert)
* [`bf luis:generate:cs`](#bf-luisgeneratecs)
* [`bf luis:generate:ts`](#bf-luisgeneratets)
* [`bf luis:translate`](#bf-luistranslate)
* [`bf qnamaker`](#bf-qnamaker)
* [`bf qnamaker:alterations`](#bf-qnamakeralterations)
* [`bf qnamaker:alterations:list`](#bf-qnamakeralterationslist)
* [`bf qnamaker:alterations:replace`](#bf-qnamakeralterationsreplace)
* [`bf qnamaker:convert`](#bf-qnamakerconvert)
* [`bf qnamaker:endpointkeys`](#bf-qnamakerendpointkeys)
* [`bf qnamaker:endpointkeys:list`](#bf-qnamakerendpointkeyslist)
* [`bf qnamaker:endpointkeys:refresh`](#bf-qnamakerendpointkeysrefresh)
* [`bf qnamaker:endpointsettings`](#bf-qnamakerendpointsettings)
* [`bf qnamaker:endpointsettings:get`](#bf-qnamakerendpointsettingsget)
* [`bf qnamaker:endpointsettings:update`](#bf-qnamakerendpointsettingsupdate)
* [`bf qnamaker:init`](#bf-qnamakerinit)
* [`bf qnamaker:kb`](#bf-qnamakerkb)
* [`bf qnamaker:kb:create`](#bf-qnamakerkbcreate)
* [`bf qnamaker:kb:delete`](#bf-qnamakerkbdelete)
* [`bf qnamaker:kb:export`](#bf-qnamakerkbexport)
* [`bf qnamaker:kb:get`](#bf-qnamakerkbget)
* [`bf qnamaker:kb:list`](#bf-qnamakerkblist)
* [`bf qnamaker:kb:publish`](#bf-qnamakerkbpublish)
* [`bf qnamaker:kb:replace`](#bf-qnamakerkbreplace)
* [`bf qnamaker:kb:update`](#bf-qnamakerkbupdate)
* [`bf qnamaker:operationdetails`](#bf-qnamakeroperationdetails)
* [`bf qnamaker:operationdetails:get`](#bf-qnamakeroperationdetailsget)
* [`bf qnamaker:query`](#bf-qnamakerquery)
* [`bf qnamaker:train`](#bf-qnamakertrain)
* [`bf qnamaker:translate`](#bf-qnamakertranslate)

## `bf chatdown`

Convierte los archivos de diálogos de chat en formato \<nombre de archivo>.chat en archivos de transcripción. Escribe el archivo \<nombre de archivo>.transcript correspondiente de cada archivo .chat.

```text
USAGE
  $ bf chatdown

OPTIONS
  -h, --help  Chatdown command help
```

_Ver el código: [@microsoft/bf-chatdown](https://github.com/microsoft/botframework-cli/tree/master/packages/chatdown/src/commands/chatdown/index.ts)_

## `bf chatdown:convert`

Convierte los archivos de diálogos de chat en formato \<nombre de archivo>.chat en archivos de transcripción. Escribe el archivo \<nombre de archivo>.transcript correspondiente de cada archivo .chat.

```text
USAGE
  $ bf chatdown:convert

OPTIONS
  -f, --force    If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help     Chatdown command help

  -i, --in=in    The path of the chat file or directory to be parsed. A glob expression may be passed containing chat
                 files to be processed all at once, ex. ./**/*.chat. If flag is omitted, stdin will be used. If an
                 output directory is not present (-o), it will default the output to the current working directory.

  -o, --out=out  Path to the directory where the output of the multiple chat file processing (-o) will be placed.

  -p, --prefix   Prefix stdout with package name.

  -s, --stamp    Use static timestamps when generating timestamps on activities.

EXAMPLE

     $ bf chatdown
     $ bf chatdown --in=./path/to/file/sample.chat
     $ bf chatdown --in ./test/utils/*.sample.chat -o ./
     $ (echo user=Joe && [ConversationUpdate=MembersAdded=Joe]) | bf chatdown --static
```

_Ver el código: [@microsoft/bf-chatdown](https://github.com/microsoft/botframework-cli/tree/master/packages/chatdown/src/commands/chatdown/convert.ts)_

## `bf config`

Configura varias opciones dentro de la CLI.

```text
USAGE
  $ bf config

OPTIONS
  -h, --help  config help
```

_Ver el código: [@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/index.ts)_

## `bf config:set:qnamaker`

Establece los datos de configuración de QnA Maker.

```text
USAGE
  $ bf config:set:qnamaker

OPTIONS
  -h, --help                         config:set:qnamaker help
  --endpointKey=endpointKey          QnAMaker endpointKey to be set
  --hostname=hostname                QnAMaker hostname to be set
  --kbId=kbId                        QnAMaker kbId to be set
  --subscriptionKey=subscriptionKey  QnAMaker subscriptionkey to be set

EXAMPLE

     {
       "qnamaker_kbId": "3bda64af-dddd-dddd-dddd-021906b093b1",
       "qnamaker_subscriptionKey": "nnnnnnnnnnnnnnnnnnnnnnnnn",
       "qnamaker_endpointKey": "6b5ecf9c-kkkk-kkkk-kkkk-761489817e5f",
       "qnamaker_hostname": "https://{qnaservice-hostname}.azurewebsites.net"
     }
```

_Ver el código: [@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/set/qnamaker.ts)_

## `bf config:set:telemetry`

Habilita o deshabilita la recopilación de datos anónimos para mejorar los productos. (Llamadas a grupos de comandos y uso de marcas)

```text
USAGE
  $ bf config:set:telemetry

OPTIONS
  -d, --disable  Disable tlemetry
  -e, --enable   Enable tlemetry
  -h, --help     config:set:telemetry help
```

_Ver el código: [@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/set/telemetry.ts)_

## `bf config:show`

Muestra el archivo de configuración.

```text
USAGE
  $ bf config:show

OPTIONS
  -h, --help  config:show help
```

_Ver el código: [@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/show.ts)_

## `bf config:show:qnamaker`

Muestra la configuración de QnA Maker.

```text
USAGE
  $ bf config:show:qnamaker

OPTIONS
  -h, --help  config:show:qnamaker help
```

_Ver el código: [@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/show/qnamaker.ts)_

## `bf config:show:telemetry`

Muestra la configuración de telemetría.

```text
USAGE
  $ bf config:show:telemetry

OPTIONS
  -h, --help  config:show:telemetry help
```

_Ver el código: [@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/show/telemetry.ts)_

## `bf help [COMMAND]`

Muestra la ayuda del comando bf.

```text
USAGE
  $ bf help [COMMAND]

ARGUMENTS
  COMMAND  command to show help for

OPTIONS
  --all  see all commands in CLI
```

_Ver el código: [@oclif/plugin-help](https://github.com/oclif/plugin-help/blob/v2.1.6/src/commands/help.ts)_

## `bf luis`

Convierte y traduce archivos luis/lu o genera código fuente.

```text
USAGE
  $ bf luis

OPTIONS
  -h, --help  Display Luis available commands
```

_Ver el código: [@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/luis/index.ts)_

## `bf luis:convert`

Convierte archivos .lu en un modelo JSON de aplicación de LUIS o viceversa.

```text
USAGE
  $ bf luis:convert

OPTIONS
  -f, --force                    If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help                     luis:convert help
  -i, --in=in                    Source .lu file(s) or LUIS application JSON model
  -o, --out=out                  Output file or folder name. If not specified stdout will be used as output
  -r, --recurse                  Indicates if sub-folders need to be considered to file .lu file(s)
  --culture=culture              Lang code for the LUIS application
  --description=description      Text describing the LUIS application
  --log                          Enables log messages
  --name=name                    Name of the LUIS application
  --schemaversion=schemaversion  Schema version of the LUIS application
  --sort                         When set, intent, utterances, entities are alphabetically sorted in .lu files
  --versionid=versionid          Version ID of the LUIS application
```

_Ver el código: [@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/luis/convert.ts)_

## `bf luis:generate:cs`

Genera un código fuente en C# fuertemente tipado a partir de un modelo de LUIS exportado (JSON).

```text
USAGE
  $ bf luis:generate:cs

OPTIONS
  -f, --force            If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help             luis:generate:cs help
  -i, --in=in            Path to the file containing the LUIS application JSON model
  -o, --out=out          Output file or folder name. If not specified stdout will be used as output
  --className=className  Name of the autogenerated class (can include namespace)
```

_Ver el código: [@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/luis/generate/cs.ts)_

## `bf luis:generate:ts`

Genera un código fuente en typescript fuertemente tipado a partir de un modelo de LUIS exportado (JSON).

```text
USAGE
  $ bf luis:generate:ts

OPTIONS
  -f, --force            If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help             luis:generate:ts help
  -i, --in=in            Path to the file containing the LUIS application JSON model
  -o, --out=out          Output file or folder name. If not specified stdout will be used as output
  --className=className  Name of the autogenerated class
```

_Ver el código: [@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/luis/generate/ts.ts)_

## `bf luis:translate`

Traduce un modelo JSON de la aplicación de LUIS determinado o archivos .lu.

```text
USAGE
  $ bf luis:translate

OPTIONS
  -f, --force                  If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help                   luis:translate help
  -i, --in=in                  Source .lu file(s) or LUIS application JSON model
  -o, --out=out                Output folder name. If not specified stdout will be used as output
  -r, --recurse                Indicates if sub-folders need to be considered to file .lu file(s)
  --srclang=srclang            Source lang code. Auto detect if missing.
  --tgtlang=tgtlang            (required) Comma separated list of target languages.
  --translate_comments         When set, machine translate comments found in .lu file
  --translate_link_text        When set, machine translate link description in .lu file
  --translatekey=translatekey  (required) Machine translation endpoint key.
```

_Ver el código: [@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/luis/translate.ts)_

## `bf qnamaker`

QnA Maker

```text
USAGE
  $ bf qnamaker

OPTIONS
  -h, --help  Display QnA Maker CLI available commands
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/index.ts)_

## `bf qnamaker:alterations`

Comandos para reemplazar y enumerar las modificaciones.

```text
USAGE
  $ bf qnamaker:alterations

OPTIONS
  -h, --help  display qnamaker:alterations available commands
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/alterations/index.ts)_

## `bf qnamaker:alterations:list`

Descarga todas las modificaciones de palabras (sinónimos) agregadas por el usuario.

```text
USAGE
  $ bf qnamaker:alterations:list

OPTIONS
  -h, --help                         qnamaker:alterations:list command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/alterations/list.ts)_

## `bf qnamaker:alterations:replace`

Reemplaza las modificaciones de palabras (sinónimos) de la base de conocimiento por los registros especificados.

```text
USAGE
  $ bf qnamaker:alterations:replace

OPTIONS
  -h, --help                         qnamaker:alterations:replace command help
  -i, --in=in                        File path to the WordAlterationsDTO object to send in the body of the request
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/alterations/replace.ts)_

## `bf qnamaker:convert`

Convierte archivos .lu en un modelo JSON de aplicación de QnA o viceversa.

```text
USAGE
  $ bf qnamaker:convert

OPTIONS
  -f, --force    If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help     qnamaker:convert help
  -i, --in=in    Source .qna file(s) or QnA KB JSON file
  -o, --out=out  Output file or folder name. If not specified stdout will be used as output
  -r, --recurse  Indicates if sub-folders need to be considered to file .qna file(s)
  --alterations  Indicates if files is QnA Alterations
  --log          Enables log messages
  --name=name    Name of the QnA KB
  --sort         When set, questions collections are alphabetically sorted are alphabetically sorted in .qna files
```

_Ver el código: [@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/qnamaker/convert.ts)_

## `bf qnamaker:endpointkeys`

Comandos para actualizar y enumerar claves.

```text
USAGE
  $ bf qnamaker:endpointkeys

OPTIONS
  -h, --help  display qnamaker:endpointkeys available commands
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointkeys/index.ts)_

## `bf qnamaker:endpointkeys:list`

Enumera todas las claves de punto de conexión válidas actualmente para consultar el punto de conexión privado.

```text
USAGE
  $ bf qnamaker:endpointkeys:list

OPTIONS
  -h, --help                         qnamaker:endpointkeys:list command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointkeys/list.ts)_

## `bf qnamaker:endpointkeys:refresh`

Vuelve a generar una clave de punto de conexión, en caso de que sospeche que las claves se han puesto en peligro.

```text
USAGE
  $ bf qnamaker:endpointkeys:refresh

OPTIONS
  -h, --help                         qnamaker:endpoints:refresh command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --keyType=keyType                  (required) Type of Key. (PrimaryKey/SecondaryKey)

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointkeys/refresh.ts)_

## `bf qnamaker:endpointsettings`

Comandos para obtener y actualizar la configuración del punto de conexión.

```text
USAGE
  $ bf qnamaker:endpointsettings

OPTIONS
  -h, --help  display qnamaker:update available commands
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointsettings/index.ts)_

## `bf qnamaker:endpointsettings:get`

Obtiene la configuración de punto de conexión de un punto de conexión.

```text
USAGE
  $ bf qnamaker:endpointsettings:get

OPTIONS
  -h, --help                         qnamaker:endpointsettings:get command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --kbId=kbId                        Knowledgebase id to get metadata.

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointsettings/get.ts)_

## `bf qnamaker:endpointsettings:update`

Actualiza la configuración de punto de conexión de un punto de conexión.

```text
USAGE
  $ bf qnamaker:endpointsettings:update

OPTIONS
  -h, --help                         qnamaker:endpointsettings:update command help
  --activelearning                   Enable active learning. Disables if flag not set
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointsettings/update.ts)_

## `bf qnamaker:init`

Inicializa el archivo de configuración con valores.

```text
USAGE
  $ bf qnamaker:init

OPTIONS
  -h, --help           qnamaker:init command help
  --endpoint=endpoint  Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/init.ts)_

## `bf qnamaker:kb`

Comandos para manipular la base de conocimiento.

```text
USAGE
  $ bf qnamaker:kb

OPTIONS
  -h, --help  display qnamaker:kb available commands
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/index.ts)_

## `bf qnamaker:kb:create`

Crea una nueva base de conocimiento.

```text
USAGE
  $ bf qnamaker:kb:create

OPTIONS
  -h, --help                         qnamaker:kb:create command help
  -i, --in=in                        File path to the CreateKbDTO object to send in the body of the request.
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --name=name                        Name of the kb you want to create. This will override the name of KB that might be
                                     present in the CreateKb DTO

  --save                             Save the kbId in config.

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/create.ts)_

## `bf qnamaker:kb:delete`

Elimina una base de conocimiento por identificador.

```text
USAGE
  $ bf qnamaker:kb:delete

OPTIONS
  -h, --help                         qnamaker:kb:delete command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --force                            Do not prompt for confirmation, force the operation

  --kbId=kbId                        Knowledgebase id to be deleted. Overrides the knowledge base id present in the
                                     config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/delete.ts)_

## `bf qnamaker:kb:export`

Devuelve un archivo JSON de la base de conocimiento a stdout.

```text
USAGE
  $ bf qnamaker:kb:export

OPTIONS
  -h, --help                         qnamaker:kb:export command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --environment=environment          (required) Specifies whether environment is Test or Prod.

  --kbId=kbId                        Knowledgebase id to be exported. Overrides the knowledge base id present in the
                                     config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/export.ts)_

## `bf qnamaker:kb:get`

Obtiene los metadatos de una base de conocimiento.

```text
USAGE
  $ bf qnamaker:kb:get

OPTIONS
  -h, --help                         qnamaker:kb:get command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --kbId=kbId                        Knowledgebase id to get metadata. Overrides the knowledge base id present in the
                                     config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/get.ts)_

## `bf qnamaker:kb:list`

Enumera todas las bases de conocimiento.

```text
USAGE
  $ bf qnamaker:kb:list

OPTIONS
  -h, --help                         qnamaker:kb:list command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/list.ts)_

## `bf qnamaker:kb:publish`

Publica todo lo no publicado en la base de conocimiento en el punto de conexión de producción.

```text
USAGE
  $ bf qnamaker:kb:publish

OPTIONS
  -h, --help                         qnamaker:kb:publish command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --kbId=kbId                        Knowledgebase id to pubish. Overrides the knowledge base id present in the config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/publish.ts)_

## `bf qnamaker:kb:replace`

Reemplaza el contenido de la base de conocimiento por contenido nuevo.

```text
USAGE
  $ bf qnamaker:kb:replace

OPTIONS
  -h, --help                         qnamaker:kb:replace command help
  -i, --in=in                        File path to the ReplaceKbDTO object to send in the body of the request
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --kbId=kbId                        Knowledgebase id. Overrides the knowledge base id present in the config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/replace.ts)_

## `bf qnamaker:kb:update`

Agrega o elimina pares o direcciones URL de QnA en una base de conocimiento existente.

```text
USAGE
  $ bf qnamaker:kb:update

OPTIONS
  -h, --help                         qnamaker:kb:update command help

  -i, --in=in                        The file path to the UpdateKbOperationDTO object to send in the body of the
                                     request.

  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --kbId=kbId                        Knowledgebase id. Overrides the knowledge base id present in the config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config

  --wait                             Wait for the operation to complete.
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/update.ts)_

## `bf qnamaker:operationdetails`

Comando para obtener los detalles de la operación.

```text
USAGE
  $ bf qnamaker:operationdetails

OPTIONS
  -h, --help  display qnamaker:operationdetails available commands
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/operationdetails/index.ts)_

## `bf qnamaker:operationdetails:get`

Obtiene los detalles de una operación de ejecución prolongada específica.

```text
USAGE
  $ bf qnamaker:operationdetails:get

OPTIONS
  -h, --help                         qnamaker:operationdetails:get command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --operationId=operationId          (required) Operation id.

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/operationdetails/get.ts)_

## `bf qnamaker:query`

Consulta una respuesta en una base de conocimiento.

```text
USAGE
  $ bf qnamaker:query

OPTIONS
  -h, --help                       qnamaker:query command help
  --context=context                Path to Context object json file with previous QnA

  --endpointKey=endpointKey        Specifies the endpoint key for your private QnA service (From qnamaker.ai portal user
                                   settings page). Overrides the value present in config

  --hostname=hostname              Specifies the url for your private QnA service. Overrides the value present in config

  --kbId=kbId                      Specifies the active qnamaker knowledgebase id. Overrides the value present in the
                                   config

  --qnaId=qnaId                    Exact qnaId to fetch from the knowledgebase, this field takes priority over question

  --question=question              (required) Query to get a prediction for

  --scorethreshold=scorethreshold  Specifies the confidence score threshold for the returned answer.

  --strictfilters=strictfilters    Path to json file with MetadataDTO[] e.g {"strictfilters": MetadataDTO[]}

  --test                           Query against the test index

  --top=top                        Specifies the number of matching results
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/query.ts)_

## `bf qnamaker:train`

Agrega sugerencias a la base de conocimiento.

```text
USAGE
  $ bf qnamaker:train

OPTIONS
  -h, --help                         qnamaker:get:kb command help

  --endpointKey=endpointKey          Specifies the endpoint key for your private QnA service.(from qnamaker.ai portal
                                     user settings page). Overrides the value present in config.

  --hostname=hostname                Specifies the url for your private QnA service. Overrides the value present in
                                     config.

  --in=in                            File path to the FeedbackRecordDTO object to send in the body of the request.

  --kbId=kbId                        Specifies the active qnamaker knowledgebase id. Overrides the value present in the
                                     config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in config
```

_Ver el código: [@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/train.ts)_

## `bf qnamaker:translate`

Traduce un modelo JSON de la aplicación de QnA Maker determinado o archivos .qna.

```text
USAGE
  $ bf qnamaker:translate

OPTIONS
  -f, --force                  If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help                   qnamaker:translate help
  -i, --in=in                  Source .qna file(s) or QnA maker application JSON model
  -o, --out=out                Output folder name. If not specified stdout will be used as output
  -r, --recurse                Indicates if sub-folders need to be considered to find .qna file(s)
  --srclang=srclang            Source lang code. Auto detect if missing.
  --tgtlang=tgtlang            (required) Comma separated list of target languages.
  --translate_comments         When set, machine translate comments found in .qna file
  --translate_link_text        When set, machine translate link description in .qna file
  --translatekey=translatekey  (required) Machine translation endpoint key.
```

_Ver el código: [@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/qnamaker/translate.ts)_
<!-- commandsstop -->
