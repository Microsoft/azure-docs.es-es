---
title: Solución de problemas para los errores del trabajo de U-SQL de Azure Data Lake Analytics debidos a la actualización a .NET Framework 4.7.2
description: Solucione los errores de trabajos de U-SQL debidos a la actualización a .NET Framework 4.7.2.
services: data-lake-analytics
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: troubleshooting
ms.workload: big-data
ms.date: 10/11/2019
ms.openlocfilehash: f553da53f665676dd44ec581d082f9a39e242516
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132388"
---
# <a name="azure-data-lake-analytics-is-upgrading-to-the-net-framework-v472"></a>Azure Data Lake Analytics se está actualizando a .NET Framework v4.7.2

El entorno en tiempo de ejecución predeterminado de Azure Data Lake Analytics se está actualizando de .NET Framework v4.5.2 a .NET Framework v4.7.2. Este cambio supone un pequeño riesgo de cambios importantes si su código de U-SQL usa ensamblados personalizados y esos usan bibliotecas de .NET.

Esta actualización de .NET Framework 4.5.2 a la versión 4.7.2 significa que .NET Framework implementado en un entorno en tiempo de ejecución de U-SQL (el entorno en tiempo de ejecución predeterminado) ahora siempre será 4.7.2. No hay ninguna opción en paralelo para las versiones de .NET Framework.

Una vez finalizada la actualización a .NET Framework 4.7.2, el código administrado del sistema se ejecutará como la versión 4.7.2, las bibliotecas proporcionadas por el usuario, como los ensamblados personalizados de U-SQL, se ejecutarán en el modo compatible con versiones anteriores adecuado para la versión para la cual se generó el ensamblado.

- Si se generan bibliotecas de vínculos dinámicos de ensamblado para la versión 4.5.2, el marco implementado las tratará como bibliotecas 4.5.2, proporcionando (con algunas excepciones) semántica de la versión 4.5.2.
- Ahora puede usar ensamblados personalizados de U-SQL que usan las características de la versión 4.7.2, si tiene como destino .NET Framework 4.7.2.

Debido a esta actualización a .NET Framework 4.7.2, existe la posibilidad de introducir cambios importantes en los trabajos de U-SQL que utilizan ensamblados personalizados de .NET. Se recomienda usar el procedimiento siguiente para comprobar si hay problemas de compatibilidad con versiones anteriores.

## <a name="how-to-check-for-backwards-compatibility-issues"></a>Comprobación de problemas de compatibilidad con versiones anteriores

Compruebe la posibilidad de que haya problemas importantes de compatibilidad con versiones anteriores mediante la ejecución de las comprobaciones de compatibilidad de .NET en el código .NET en los ensamblados personalizados de U-SQL.

> [!Note]
> La herramienta no detecta cambios importantes reales, solo identifica las API .NET llamadas que pueden generar problemas (para ciertas entradas). Si recibe una notificación de un problema, es posible que el código siga siendo correcto, pero debe revisar más detalles.

1. Ejecute el comprobador de compatibilidad con versiones anteriores en las bibliotecas de vínculos dinámicos de .NET mediante:
   1. El uso de la extensión de Visual Studio de [Extensión de Visual Studio para el Analizador de portabilidad de .NET](https://marketplace.visualstudio.com/items?itemName=ConnieYau.NETPortabilityAnalyzer).
   1. La descarga y el uso de la herramienta independiente desde [dotnetapiport de GitHub](https://github.com/microsoft/dotnet-apiport). Las instrucciones para ejecutar la herramienta independiente están en los [cambios importantes de dotnetapiport de GitHub](https://github.com/microsoft/dotnet-apiport/blob/dev/docs/HowTo/BreakingChanges.md).
   1. Para la versión 4.7.2, compatibilidad, `read isRetargeting == True` identifica posibles problemas.
2. Si la herramienta indica si el código puede verse afectado por cualquiera de las posibles incompatibilidades con versiones anteriores (a continuación se enumeran algunos ejemplos comunes de las incompatibilidades), puede realizar una comprobación adicional al:
   1. Analizar el código e identificar si pasa valores a las API afectadas.
   1. Realizar una comprobación en tiempo de ejecución. La implementación en tiempo de ejecución no se hace en paralelo en ADLA. Puede realizar una comprobación en tiempo de ejecución antes de la actualización, mediante la ejecución local de Visual Studio con un .NET Framework  4.7.2 local en un conjunto de datos representativo.
3. Si realmente se ve afectado por una incompatibilidad con versiones anteriores, siga los pasos necesarios para corregirla (por ejemplo, corregir los datos o la lógica de código).

En la mayoría de los casos, no debería verse afectado por la incompatibilidad con versiones anteriores.

## <a name="timeline"></a>Escala de tiempo

Puede comprobar la implementación del nuevo entorno en tiempo de ejecución aquí, en [la solución de problemas del entorno en tiempo de ejecución](runtime-troubleshoot.md), y observando cualquier trabajo anterior correcto.

### <a name="what-if-i-cant-get-my-code-reviewed-in-time"></a>¿Qué ocurre si no puedo hacer que se revise el código a tiempo?

Puede enviar el trabajo a la versión de entorno en tiempo de ejecución anterior (creada con 4.5.2 como destino), sin embargo, debido a la falta de funcionalidades en paralelo de .NET Framework, solo se ejecutará en modo de compatibilidad de 4.5.2. Todavía se pueden producir algunos problemas de compatibilidad con versiones anteriores debido a este comportamiento.

### <a name="what-are-the-most-common-backwards-compatibility-issues-you-may-encounter"></a>¿Cuáles son los problemas más comunes de compatibilidad con versiones anteriores que puede encontrar?

Las incompatibilidades con versiones anteriores más comunes que es probable que identifique el comprobador son (generamos esta lista al ejecutar el comprobador en nuestros propios trabajos de ADLA internos), las bibliotecas que se ven afectadas (tenga en cuenta que puede llamar a las bibliotecas solo de manera indirecta, por lo que es importante realizar la primera acción necesaria para comprobar si los trabajos se ven afectados) y las posibles acciones para solucionarlo. Nota: En casi todos los casos para nuestros propios trabajos, las advertencias han resultado ser falsos positivos debido a la naturaleza limitada de los cambios más importantes.

- La propiedad IAsyncResult. CompletedSynchronously debe ser correcta para que se complete la tarea resultante
  - Al llamar a TaskFactory.FromAsync, la implementación de la propiedad IAsyncResult.CompletedSynchronously debe ser correcta para que se complete la tarea resultante. Es decir, la propiedad debe devolver el valor True si, y solo si, la implementación se completó de manera sincrónica. Anteriormente, no se comprobó la propiedad.
  - Bibliotecas afectadas: mscorlib, System.Threading.Tasks
  - Acción sugerida: asegúrese de que TaskFactory.FromAsync devuelva correctamente el valor True

- DataObject.GetData ahora recupera datos como UTF-8
  - En el caso de las aplicaciones que tienen como destino .NET Framework 4 o que se ejecutan en .NET Framework 4.5.1 o versiones anteriores, DataObject.GetData recupera datos con formato HTML como una cadena ASCII. Como resultado, los caracteres que no son ASCII (los caracteres cuyos códigos ASCII son mayores que 0x7F) se representan mediante dos caracteres aleatorios.#N##N#En el caso de las aplicaciones que tienen como destino .NET Framework 4.5 o versiones posteriores y que se ejecutan en .NET Framework 4.5.2, `DataObject.GetData` recupera los datos con formato HTML como UTF-8, que representa correctamente los caracteres mayores que 0x7F.
  - Bibliotecas afectadas: Glo
  - Acción sugerida: asegúrese de que los datos recuperados tengan el formato que desea

- XmlWriter se inicia en pares suplentes no válidos
  - En el caso de las aplicaciones que tienen como destino .NET Framework 4.5.2 o versiones anteriores, escribir un par suplente no válido mediante el control de reserva de excepción no siempre genera una excepción. En el caso de las aplicaciones que tienen como destino .NET Framework 4.6, si se intenta escribir un par suplente no válido, se genera una `ArgumentException`.
  - Bibliotecas afectadas: System.Xml, System.Xml.ReaderWriter
  - Acción sugerida: asegúrese de no escribir un par suplente no válido que generará una excepción de argumento

- HtmlTextWriter no representa correctamente el elemento `<br/>`
  - A partir de .NET Framework 4.6, llamar a `HtmlTextWriter.RenderBeginTag()` y `HtmlTextWriter.RenderEndTag()` con un elemento `<BR />` insertará correctamente solo un `<BR />` (en lugar de dos)
  - Bibliotecas afectadas: System.Web
  - Acción sugerida: asegúrese de que inserta la cantidad de `<BR />` que espera ver, por lo que no se ve ningún comportamiento aleatorio en el trabajo de producción

- La llamada a CreateDefaultAuthorizationContext con un argumento NULL ha cambiado
  - La implementación de AuthorizationContext devuelta por una llamada a `CreateDefaultAuthorizationContext(IList<IAuthorizationPolicy>)` con un argumento authorizationPolicies NULL ha cambiado su implementación en .NET Framework 4.6.
  - Bibliotecas afectadas: System.IdentityModel
  - Acción sugerida: asegúrese de controlar el nuevo comportamiento esperado cuando hay una directiva de autorización nula
  
- RSACng ahora carga correctamente las claves RSA de tamaño de clave no estándar
  - En las versiones de .NET Framework anteriores a 4.6.2, los clientes con tamaños de clave no estándar para certificados RSA no pueden tener acceso a esas claves a través de los métodos de extensión `GetRSAPublicKey()` y `GetRSAPrivateKey()`. Se genera una `CryptographicException` con el mensaje "The requested key size is not supported" ("No se admite el tamaño de clave solicitado"). Este problema se corrigió con .NET Framework 4.6.2. Del mismo modo, `RSA.ImportParameters()` y `RSACng.ImportParameters()` ahora funcionan con tamaños de clave no estándar sin generar `CryptographicException`.
  - Bibliotecas afectadas: mscorlib, System.Core
  - Acción sugerida: asegúrese de que las claves RSA funcionan según lo previsto

- Las comprobaciones de dos puntos en las rutas de acceso son más estrictas
  - En .NET Framework 4.6.2, se realizaron varios cambios para admitir rutas de acceso que anteriormente no se admitían (tanto en longitud como en formato). Las comprobaciones de la sintaxis correcta del separador de unidad (dos puntos) se hicieron más estrictas, lo que tuvo el efecto secundario de bloquear algunas rutas de acceso de URI en algunas API de ruta de acceso seleccionadas donde solían ser toleradas.
  - Bibliotecas afectadas: mscorlib, System.Runtime.Extensions
  - Acción sugerida:

- llamadas a constructores ClaimsIdentity
  - A partir de .NET Framework 4.6.2, hay un cambio en la forma en que los constructores `T:System.Security.Claims.ClaimsIdentity` con un parámetro `T:System.Security.Principal.IIdentity` establecen la propiedad `P:System.Security.Claims.ClaimsIdentify.Actor`. Si el argumento `T:System.Security.Principal.IIdentity` es un objeto `T:System.Security.Claims.ClaimsIdentity` y la propiedad `P:System.Security.Claims.ClaimsIdentify.Actor` de ese objeto `T:System.Security.Claims.ClaimsIdentity` no es `null`, la propiedad `P:System.Security.Claims.ClaimsIdentify.Actor` se adjunta mediante el método `M:System.Security.Claims.ClaimsIdentity.Clone`. En .NET Framework 4.6.1 y versiones anteriores, la propiedad `P:System.Security.Claims.ClaimsIdentify.Actor` se adjunta como una referencia existente. Debido a este cambio, a partir de .NET Framework 4.6.2, la propiedad `P:System.Security.Claims.ClaimsIdentify.Actor` del objeto `T:System.Security.Claims.ClaimsIdentity` nuevo no es igual a la propiedad `P:System.Security.Claims.ClaimsIdentify.Actor` del argumento `T:System.Security.Principal.IIdentity` del constructor. En .NET Framework 4.6.1 y versiones anteriores, es igual.
  - Bibliotecas afectadas: mscorlib
  - Acción sugerida: asegúrese de que ClaimsIdentity funciona según lo previsto en el nuevo entorno en tiempo de ejecución

- La serialización de los caracteres de control con DataContractJsonSerializer ahora es compatible con ECMAScript v6 y v8
  - En .NET Framework 4.6.2 y versiones anteriores, DataContractJsonSerializer no serializaba algunos caracteres de control especiales, como \b, \f y \t, de manera que fuera compatible con los estándares de ECMAScript v6 y v8. A partir de .NET Framework 4.7, la serialización de estos caracteres de control es compatible con ECMAScript v6 y v8.
  - Bibliotecas afectadas: System.Runtime.Serialization.Json
  - Acción sugerida: garantice el mismo comportamiento con DataContractJsonSerializer
