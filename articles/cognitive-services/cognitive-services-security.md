---
title: Seguridad de Azure Cognitive Services
titleSuffix: Azure Cognitive Services
description: Conozca las distintas consideraciones relativas a la seguridad en el uso de Cognitive Services.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: erhopf
ms.custom: devx-track-python, devx-track-javascript, devx-track-csharp
ms.openlocfilehash: d74a2c45c1d2b2b03c35e7235994980a64cba979
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89072118"
---
# <a name="azure-cognitive-services-security"></a>Seguridad de Azure Cognitive Services

La seguridad se debe considerar una de las máximas prioridades cuando se desarrolla cualquier aplicación. Con la aparición de las aplicaciones habilitadas para la inteligencia artificial, la seguridad cobra aún mayor importancia. En este artículo se describen varios aspectos de la seguridad de Azure Cognitive Services, como el uso de la seguridad de la capa de transporte, la autenticación, la configuración segura de los datos confidenciales y la Caja de seguridad del cliente para el acceso a los datos de los clientes.

## <a name="transport-layer-security-tls"></a>Seguridad de la capa de transporte (TLS)

Todos los puntos de conexión de Cognitive Services expuestos a través de HTTP aplican TLS 1.2. Con un protocolo de seguridad aplicado, los consumidores que intentan llamar a un punto de conexión de Cognitive Services deben cumplir estas directrices:

* El sistema operativo (SO) del cliente debe admitir TLS 1.2
* El lenguaje (y la plataforma) que se usa para hacer la llamada HTTP tiene que especificar TLS 1.2 como parte de la solicitud
  * En función del lenguaje y la plataforma, la especificación de TLS se realiza de forma implícita o explícita

En el caso de los usuarios de .NET, tenga en cuenta los <a href="https://docs.microsoft.com/dotnet/framework/network-programming/tls" target="_blank">procedimientos recomendados de Seguridad de la capa de transporte <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

## <a name="authentication"></a>Authentication

Cuando se habla de autenticación, hay varias ideas equivocadas que son habituales. La autenticación y la autorización suelen confundirse. La identidad es también un componente importante de la seguridad. Una identidad es una colección de información sobre una <a href="https://en.wikipedia.org/wiki/Principal_(computer_security)" target="_blank">entidad de seguridad<span class="docon docon-navigate-external x-hidden-focus"></span></a>. Los proveedores de identidades (IdP) proporcionan identidades a los servicios de autenticación. La autenticación es el acto de comprobar la identidad de un usuario. La autorización es la especificación de los privilegios y derechos de acceso a los recursos de una identidad determinada. Varias de las ofertas de Cognitive Services incluyen el control de acceso basado en rol (RBAC). RBAC se puede usar para simplificar una parte del ceremonial implicado en la administración manual de entidades de seguridad. Para más detalles, consulte [Control de acceso basado en rol para recursos de Azure](../role-based-access-control/overview.md).

Para más información sobre la autenticación con claves de suscripción, tokens de acceso y Azure Active Directory (AAD), consulte <a href="https://docs.microsoft.com/azure/cognitive-services/authentication" target="_blank">Autenticación de solicitudes en Azure Cognitive Services<span class="docon docon-navigate-external x-hidden-focus"></span></a>.

## <a name="environment-variables-and-application-configuration"></a>Configuración de las variables de entorno y de la aplicación

Las variables de entorno son pares nombre-valor que se almacenan en un entorno específico. Son una alternativa más segura al uso de valores codificados de forma rígida para datos confidenciales. Los valores codificados de forma rígida no son seguros y deben evitarse.

> [!CAUTION]
> **No** use valores codificados de forma rígida para datos confidenciales, ya que representa una importante vulnerabilidad de la seguridad.

> [!NOTE]
> Aunque las variables de entorno se almacenan en texto sin formato, están aisladas en un entorno. Si un entorno está en peligro, también lo están sus variables.

### <a name="set-environment-variable"></a>Establecimiento de una variable de entorno

Para establecer las variables de entorno, use uno de los siguientes comandos (donde `ENVIRONMENT_VARIABLE_KEY` es la clave con nombre y `value` es el valor almacenado en la variable de entorno).

# <a name="command-line"></a>[Línea de comandos](#tab/command-line)

Cree y asigne una variable de entorno persistente según el valor.

```CMD
:: Assigns the env var to the value
setx ENVIRONMENT_VARIABLE_KEY="value"
```

En una nueva instancia del **símbolo del sistema**, lea la variable de entorno.

```CMD
:: Prints the env var value
echo %ENVIRONMENT_VARIABLE_KEY%
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Cree y asigne una variable de entorno persistente según el valor.

```powershell
# Assigns the env var to the value
[System.Environment]::SetEnvironmentVariable('ENVIRONMENT_VARIABLE_KEY', 'value', 'User')
```

En una nueva instancia de **Windows PowerShell**, lea la variable de entorno.

```powershell
# Prints the env var value
[System.Environment]::GetEnvironmentVariable('ENVIRONMENT_VARIABLE_KEY')
```

# <a name="bash"></a>[Bash](#tab/bash)

Cree y asigne una variable de entorno persistente según el valor.

```Bash
# Assigns the env var to the value
echo export ENVIRONMENT_VARIABLE_KEY="value" >> /etc/environment && source /etc/environment
```

En una nueva instancia de **Bash**, lea la variable de entorno.

```Bash
# Prints the env var value
echo "${ENVIRONMENT_VARIABLE_KEY}"

# Or use printenv:
# printenv ENVIRONMENT_VARIABLE_KEY
```

---

> [!TIP]
> Después de establecer una variable de entorno, reinicie su entorno de desarrollo integrado (IDE) para asegurarse de que las variables de entorno recién agregadas están disponibles.

### <a name="get-environment-variable"></a>Obtención de una variable de entorno

Para obtener una variable de entorno, se debe leer en memoria. En función del lenguaje que se utilice, plantéese la posibilidad de usar los siguientes fragmentos de código. Estos fragmentos de código muestran cómo obtener una variable de entorno dada la clave `ENVIRONMENT_VARIABLE_KEY` y asignarla a una variable denominada `value`.

# <a name="c"></a>[C#](#tab/csharp)

Para más información, consulte <a href="https://docs.microsoft.com/dotnet/api/system.environment.getenvironmentvariable" target="_blank">`Environment.GetEnvironmentVariable` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```csharp
using static System.Environment;

class Program
{
    static void Main()
    {
        // Get the named env var, and assign it to the value variable
        var value =
            GetEnvironmentVariable(
                "ENVIRONMENT_VARIABLE_KEY");
    }
}
```

# <a name="c"></a>[C++](#tab/cpp)

Para más información, consulte <a href="https://docs.microsoft.com/cpp/c-runtime-library/reference/getenv-wgetenv" target="_blank">`getenv` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```cpp
#include <stdlib.h>

int main()
{
    // Get the named env var, and assign it to the value variable
    auto value =
        getenv("ENVIRONMENT_VARIABLE_KEY");
}
```

# <a name="java"></a>[Java](#tab/java)

Para más información, consulte <a href="https://docs.oracle.com/javase/7/docs/api/java/lang/System.html#getenv(java.lang.String)" target="_blank">`System.getenv` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```java
import java.lang.*;

public class Program {
   public static void main(String[] args) throws Exception {
    // Get the named env var, and assign it to the value variable
    String value =
        System.getenv(
            "ENVIRONMENT_VARIABLE_KEY")
   }
}
```

# <a name="nodejs"></a>[Node.js](#tab/node-js)

Para más información, consulte <a href="https://nodejs.org/api/process.html#process_process_env" target="_blank">`process.env` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```javascript
// Get the named env var, and assign it to the value variable
const value =
    process.env.ENVIRONMENT_VARIABLE_KEY;
```

# <a name="python"></a>[Python](#tab/python)

Para más información, consulte <a href="https://docs.python.org/2/library/os.html#os.environ" target="_blank">`os.environ` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```python
import os

# Get the named env var, and assign it to the value variable
value = os.environ['ENVIRONMENT_VARIABLE_KEY']
```

# <a name="objective-c"></a>[Objective-C](#tab/objective-c)

Para más información, consulte <a href="https://developer.apple.com/documentation/foundation/nsprocessinfo/1417911-environment?language=objc" target="_blank">`environment` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

```objectivec
// Get the named env var, and assign it to the value variable
NSString* value =
    [[[NSProcessInfo processInfo]environment]objectForKey:@"ENVIRONMENT_VARIABLE_KEY"];
```

---

## <a name="customer-lockbox"></a>Caja de seguridad del cliente

[La Caja de seguridad del cliente de Microsoft Azure](../security/fundamentals/customer-lockbox-overview.md) proporciona una interfaz para los clientes y así permitirles revisar y aprobar o rechazar las solicitudes de acceso de datos de cliente. Se utiliza en casos donde un ingeniero de Microsoft necesita obtener acceso a los datos del cliente durante una solicitud de soporte técnico. En este artículo se indica la manera de iniciar, seguir y almacenar las solicitudes de la Caja de seguridad del cliente para realizar revisiones y auditorías posteriores; consulte [Caja de seguridad del cliente](../security/fundamentals/customer-lockbox-overview.md). 

La Caja de seguridad del cliente está disponible para esta instancia de Cognitive Services:

* Traductor

En cuanto a los siguientes servicios, los ingenieros de Microsoft no obtendrán acceso a los datos de clientes del nivel E0: 

* Language Understanding
* Caras
* Content Moderator
* Personalizer

> [!IMPORTANT]
> En el caso de **Form Recognizer**, los ingenieros de Microsoft no tendrán acceso a los datos de los clientes en los recursos creados después del 10 de julio de 2020.

Para solicitar la capacidad de usar la SKU de E0, rellene y envíe este  [formulario de solicitud](https://aka.ms/cogsvc-cmk). Tardará de tres a cinco días hábiles aproximadamente en recibir una respuesta sobre el estado de la solicitud. En función de la demanda, es posible que se coloque en una cola y se apruebe a medida que haya espacio disponible. Una vez aprobado el uso de E0 SKU con LUIS, deberá crear un nuevo recurso desde Azure Portal y seleccionar E0 como el plan de tarifa. Los usuarios no podrán realizar la actualización de F0 a la nueva SKU de E0.

El servicio de voz no admite actualmente la Caja de seguridad del cliente. Sin embargo, los datos del cliente se pueden almacenar con Traiga su propio almacenamiento (BYOS), lo que le permite lograr controles de datos similares para la Caja de seguridad del cliente. Tenga en cuenta que los datos del servicio de voz permanecen y se procesan en la región en la que se creó el recurso de voz. Esto se aplica a los datos en reposo y a los datos en tránsito. Cuando se usan características de personalización, como el Habla personalizada y la Voz personalizada, todos los datos del cliente se transfieren, almacenan y procesan en la misma región donde residen el recurso de BYOS (si se usa) y el servicio de voz.

> [!IMPORTANT]
> Microsoft **no** usa los datos del cliente para mejorar sus modelos de voz. Además, si el registro de puntos de conexión está deshabilitado y no se usa ninguna personalización, no se almacenan los datos de clientes. 

## <a name="next-steps"></a>Pasos siguientes

* Exploración de los distintos servicios de [Cognitive Services](welcome.md)
* Más información sobre [redes virtuales de Cognitive Services](cognitive-services-virtual-networks.md)
