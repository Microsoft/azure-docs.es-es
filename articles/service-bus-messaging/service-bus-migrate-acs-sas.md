---
title: 'Azure Service Bus: Migración a la autorización de firma de acceso compartido'
description: Aprenda a migrar desde Azure Active Directory Access Control Service a la autorización de firma de acceso compartido.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: e8cd12ac97020417f9958beded1fd198dd485fff
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88064628"
---
# <a name="service-bus---migrate-from-azure-active-directory-access-control-service-to-shared-access-signature-authorization"></a>Service Bus: Migración desde Azure Active Directory Access Control Service a la autorización de firma de acceso compartido

Las aplicaciones de Service Bus tenían la posibilidad anteriormente de usar dos modelos de autorización diferentes: el modelo de token de [Firma de acceso compartido (SAS)](service-bus-sas.md), proporcionado directamente por Service Bus, y un modelo federado donde la administración de las reglas de autorización ocurría de manera interna por medio del servicio de recopilación de auditorías (ACS) de [Azure Active Directory](../active-directory/index.yml), y los tokens obtenidos de ACS se pasaban a Service Bus para autorizar el acceso a las funciones que se querían.

Hace tiempo que el modelo de autorización ACS se sustituyó por la [autorización de SAS](service-bus-authentication-and-authorization.md) como modelo preferido y en toda la documentación, instrucciones y ejemplos de hoy día se usa SAS única y exclusivamente. Es más, ya no se pueden crear espacios de nombres de Service Bus que se emparejen con ACS.

SAS reporta la ventaja de que no depende inmediatamente de otro servicio, sino que se puede usar directamente desde un cliente sin intermediarios, dando acceso al cliente en cuestión al nombre de regla y la clave de regla de SAS. SAS también se integra fácilmente con un método por el que un cliente tiene que superar primero una comprobación de autorización con otro servicio y, seguidamente, se le emite un token. El último método es similar al patrón de uso de ACS, pero permite emitir tokens de acceso según condiciones específicas de la aplicación que son difíciles de expresar en ACS.

En el caso de todas aquellas aplicaciones existentes que dependan de ACS, instamos a los clientes a que migren sus aplicaciones para que se basen en SAS.

## <a name="migration-scenarios"></a>Escenarios de migración

ACS y Service Bus se integran por medio del conocimiento compartido de una *clave de firma*. La clave de firma se usa en un espacio de nombres de ACS para firmar tokens de autorización, y también se usa en Service Bus para comprobar que el token lo ha emitido el espacio de nombres de ACS emparejado. El espacio de nombres de ACS contiene las reglas de autorización y las identidades de servicio. Las reglas de autorización definen qué identidad de servicio o qué token emitido por un proveedor de identidades obtiene un tipo de acceso determinado a una parte del gráfico del espacio de nombres de Service Bus, con la forma de coincidencia del prefijo más largo.

Por ejemplo, una regla de ACS podría conceder la notificación **Enviar** en el prefijo de ruta de acceso `/` a una identidad de servicio, lo que significa que un token emitido por ACS según dicha regla concederá al cliente derechos para realizar envíos a todas las entidades en el espacio de nombres. Si el prefijo de ruta de acceso es `/abc`, la identidad quedará limitada a envíos a entidades denominadas `abc` u organizadas bajo ese prefijo. Se da por hecho que los lectores de esta guía de migración ya están familiarizados con estos conceptos.

Los escenarios de migración se dividen en tres categorías generales:

1.  **Valores predeterminados sin modificar**. Algunos clientes usan un objeto [SharedSecretTokenProvider](/dotnet/api/microsoft.servicebus.sharedsecrettokenprovider), que pasa la identidad de servicio de **propietario** generada automáticamente y su clave secreta relativa el espacio de nombres de ACS, emparejado con el espacio de nombres de Service Bus, y no se agregan nuevas reglas.

2.  **Identidades de servicio personalizadas con reglas simples**. Algunos clientes agregan nuevas identidades de servicio y conceden a cada identidad de servicio nueva los permisos **Enviar**, **Escuchar** y **Administrar** en relación con una entidad determinada.

3.  **Identidades de servicio personalizadas con reglas complejas**. Algunos clientes (muy pocos) tienen conjuntos de reglas complejas en los que los tokens emitidos externamente se asignan a derechos en Relay, o en los que a una identidad de servicio única se le asignan derechos distintos en varias rutas de acceso del espacio de nombres a través de varias reglas.

Para obtener ayuda con la migración de conjuntos de reglas complejas, puede ponerse en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/options/). Los otros dos escenarios permiten realizar la migración directamente.

### <a name="unchanged-defaults"></a>Valores predeterminados sin modificar

Si la aplicación no ha cambiado los valores predeterminados de ACS, puede reemplazar todo el uso de [SharedSecretTokenProvider](/dotnet/api/microsoft.servicebus.sharedsecrettokenprovider) por un objeto [SharedAccessSignatureTokenProvider](/dotnet/api/microsoft.servicebus.sharedaccesssignaturetokenprovider) y usar el espacio de nombres preconfigurado **RootManageSharedAccessKey** en lugar de la cuenta de **propietario** de ACS. Recuerde que, aun cuando se use la cuenta de **propietario** de ACS, esta configuración no solía ser (ni es) recomendable, ya que esta cuenta/regla proporciona plena autoridad de administración por encima del espacio de nombres, incluido el permiso para eliminar cualquier entidad.

### <a name="simple-rules"></a>Reglas simples

Si la aplicación usa identidades de servicio personalizadas con reglas simples, la migración es sencilla en caso de que se haya creado una identidad de servicio de ACS para proporcionar control de acceso en una cola específica. Este escenario sucede con frecuencia en las soluciones de tipo SaaS donde cada cola sirve de puente a un sitio de inquilino o a una sucursal, y la identidad de servicio se crea para ese sitio en particular. En este caso, la identidad de servicio correspondiente se puede migrar a una regla de Firma de acceso compartido, directamente en la cola. El nombre de la identidad de servicio puede convertirse en el nombre de la regla de SAS y la clave de identidad de servicio, en la clave de la regla de SAS. Los derechos de la regla de SAS se configuran de forma equivalente a la correspondiente regla de ACS aplicable de la entidad.

Esta configuración nueva y adicional de SAS se puede establecer localmente en cualquier espacio de nombres existente que esté federado con ACS, y la migración lejos de ACS se realiza posteriormente con [SharedAccessSignatureTokenProvider](/dotnet/api/microsoft.servicebus.sharedaccesssignaturetokenprovider) en lugar de con [SharedSecretTokenProvider](/dotnet/api/microsoft.servicebus.sharedsecrettokenprovider). El espacio de nombres no tiene que desvincularse de ACS.

### <a name="complex-rules"></a>Reglas complejas

Las reglas de SAS no están diseñadas para ser cuentas, pero se denominan claves de firma asociadas a derechos. Por tanto, los escenarios en los que la aplicación crea varias identidades de servicio y les concede derechos de acceso para varias entidades o el espacio de nombres completo siguen requiriendo un intermediario que emita tokens. [Póngase en contacto con el soporte técnico](https://azure.microsoft.com/support/options/) si necesita instrucciones relativas a este tipo intermediario.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la autenticación de Service Bus, vea los siguientes temas:

* [Autenticación y autorización de Service Bus](service-bus-authentication-and-authorization.md)
* [Autenticación en Service Bus con Firmas de acceso compartido](service-bus-sas.md)