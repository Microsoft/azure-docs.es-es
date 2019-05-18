---
title: Integración de la escritura diferida de contraseñas con Azure AD SSPR - Azure Active Directory local
description: Escriba en diferido las contraseñas de la nube en la infraestructura de AD local
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 05/06/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahenry
ms.collection: M365-identity-device-management
ms.openlocfilehash: 749216d3fe9164857bd4abce7ba7c766e466e7d3
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65823295"
---
# <a name="what-is-password-writeback"></a>¿Qué es la escritura diferida de contraseñas?

Tener una utilidad de restablecimiento de contraseña basada en la nube es genial, pero la mayoría de las empresas todavía tienen un directorio local donde se encuentran sus usuarios. ¿Cómo mantiene Microsoft el soporte técnico tradicional de Active Directory (AD) local sincronizado con los cambios de contraseña en la nube? La escritura diferida de contraseñas es una característica que se habilita con [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) y que permite que los cambios de contraseña en la nube se escriban en diferido en un directorio local existente en tiempo real.

La escritura diferida de contraseñas se admite en entornos que usan:

* [Servicios de federación de Active Directory](../hybrid/how-to-connect-fed-management.md)
* [Sincronización de hash de contraseñas](../hybrid/how-to-connect-password-hash-synchronization.md)
* [Autenticación de paso a través](../hybrid/how-to-connect-pta.md)

> [!WARNING]
> La escritura diferida de contraseñas dejará de funcionar para los clientes que usan versiones de Azure AD Connect 1.0.8641.0 y anteriores cuando el [servicio de Azure Access Control (ACS) se retire el 7 de noviembre de 2018](../develop/active-directory-acs-migration.md). Las versiones de Azure AD Connect 1.0.8641.0 y anteriores ya no permitirán la escritura diferida de contraseñas a partir de ese momento porque dependen de ACS para esa funcionalidad.
>
> Para evitar una interrupción en el servicio y actualizar desde una versión anterior de Azure AD Connect a una versión más reciente, consulte el artículo [Azure AD Connect: actualización de una versión anterior a la versión más reciente](../hybrid/how-to-upgrade-previous-version.md)
>

La escritura diferida de contraseñas ofrece:

* **Cumplimiento de las directivas de contraseña de Active Directory locales**: cuando un usuario restablece su contraseña, se comprueba que cumple la directiva de Active Directory local antes de confirmarla en ese directorio. En esta revisión se incluye la comprobación del historial, la complejidad, la antigüedad, los filtros de contraseñas y otras restricciones de contraseña que ha definido en el entorno local de Active Directory.
* **Comentarios sin retraso**:  La escritura diferida de contraseñas es una operación sincrónica. Si la contraseña de un usuario no cumple la directiva o no se puede restablecer o modificar por algún motivo, a dicho usuario se le envía una notificación inmediatamente.
* **Permite el cambio de contraseña en el panel de acceso y Office 365**: si los usuarios con federación o sincronización de hash de contraseñas modifican las contraseñas expiradas o no expiradas, estas se escriben en diferido en el entorno local de Active Directory.
* **Admite la escritura diferida de contraseñas cuando un administrador las restablece desde Azure Portal**: siempre que un administrador restablece la contraseña de un usuario en [Azure Portal](https://portal.azure.com) y se trata de un usuario con federación o sincronización de hash de contraseñas, la contraseña se escribe en diferido en el entorno local. Esta funcionalidad no se admite en el portal de administración de Office.
* **No requiere ninguna regla de firewall de entrada**: la escritura diferida de contraseñas usa una retransmisión de Azure Service Bus como canal de comunicación subyacente. Toda la comunicación es de salida a través del puerto 443.

> [!NOTE]
> Las cuentas de usuario que se encuentran dentro de grupos protegidos en Active Directory local no se pueden utilizar con la escritura diferida de contraseñas. Las cuentas de administrador que se encuentran dentro de grupos protegidos en AD local no se pueden utilizar con la escritura diferida de contraseñas. Para más información sobre los grupos protegidos, vea [Cuentas y grupos protegidos en Active Directory](https://technet.microsoft.com/library/dn535499.aspx).

## <a name="licensing-requirements-for-password-writeback"></a>Requisitos de licencia para la escritura diferida de contraseñas

**El restablecimiento de contraseñas de autoservicio/cambio/desbloqueo con escritura diferida local es una característica premium de Azure AD**. Para más información sobre licencias, consulte la [página de precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

Para poder usar la escritura diferida de contraseñas, debe tener una de las siguientes licencias asignadas en el inquilino:

* Azure AD Premium P1
* Azure AD Premium P2
* Enterprise Mobility + Security E3 o A3
* Enterprise Mobility + Security E5 o A5
* Microsoft 365 E3 o A3
* Microsoft 365 E5 o A5
* Microsoft 365 F1
* Microsoft 365 Empresa

> [!WARNING]
> Los planes de licencias de Office 365 independientes *no admiten "Self-Service Password Reset/Change/Unlock with on-premises writeback"* (Autoservicio de restablecimiento/modificación/desbloqueo de contraseñas con escritura en diferido local) y requieren que tenga uno de los planes anteriores para que sirva esta funcionalidad.

## <a name="how-password-writeback-works"></a>Funcionamiento de la escritura diferida de contraseñas

Si un usuario con federación o sincronización de hash de contraseñas cambia o restablece su contraseña en la nube, ocurre lo siguiente:

1. Se comprueba qué tipo de contraseña tiene el usuario. Si la contraseña del usuario se administra en el entorno local:
   * Se comprueba si el servicio de escritura diferida está en funcionamiento. Si es así, el usuario puede continuar.
   * Si el servicio de escritura diferida no está activo, se informa al usuario de que la contraseña no se puede restablecer en este momento.
1. A continuación, el usuario realiza los pasos de autenticación adecuados y llega a la página **Restablecer contraseña**.
1. El usuario selecciona una nueva contraseña y la confirma.
1. Al hacer clic en **Enviar**, se cifra la contraseña de texto no cifrado con una clave simétrica creada durante el proceso de configuración de la escritura diferida.
1. La contraseña cifrada se incluye en una carga que se envía a través de un canal HTTPS a la instancia de Service Bus Relay específica del inquilino (que se establece automáticamente durante el proceso de configuración de la escritura diferida). Esta retransmisión está protegida por una contraseña generada de forma aleatoria que sólo conoce la instalación local.
1. Una vez que el mensaje llega a Service Bus, el punto de conexión de restablecimiento de contraseña se activa automáticamente y comprueba que tiene una solicitud de restablecimiento pendiente.
1. A continuación, el servicio busca al usuario mediante el atributo delimitador de la nube. Para que esta búsqueda se realice correctamente:

   * El objeto de usuario debe existir en el espacio del conector de Active Directory.
   * El objeto de usuario debe estar vinculado al objeto de metaverso (MV) correspondiente.
   * El objeto de usuario debe estar vinculado al objeto de conector de Azure Active Directory correspondiente.
   * El vínculo del objeto de conector de Active Directory que lleva al objeto de MV debe tener la regla de sincronización `Microsoft.InfromADUserAccountEnabled.xxx` en ese vínculo.
   
   Cuando se recibe la llamada de la nube, el motor de sincronización usa el atributo **cloudAnchor** para buscar el objeto de espacio de conector de Azure Active Directory. A continuación, sigue el vínculo de vuelta al objeto de MV para volver a seguir el vínculo al objeto de Active Directory. Dado que podría haber varios objetos de Active Directory (bosque múltiple) para el mismo usuario, el motor de sincronización se basa en el vínculo `Microsoft.InfromADUserAccountEnabled.xxx` para seleccionar el objeto correcto.

1. Una vez que se encuentra la cuenta de usuario, se intenta restablecer la contraseña directamente en el bosque de Active Directory correspondiente.
1. Si la operación de establecimiento de la contraseña se realiza correctamente, se notifica al usuario que se ha cambiado su contraseña.
   > [!NOTE]
   > Si la contraseña del usuario se sincroniza con Azure AD mediante la sincronización de hash de contraseñas, existe la posibilidad de que la directiva de contraseñas local sea menos segura que la directiva de contraseñas en la nube. En este caso, se aplica la directiva local. Esta directiva garantiza que la directiva local se aplique en la nube, sin importar si se usa la federación o sincronización de hash de contraseñas para proporcionar el inicio de sesión único.

1. Si la operación de establecimiento de la contraseña no se realiza, un mensaje de error pide al usuario que vuelva a intentarla. La operación puede producir un error debido a lo siguiente:
    * El servicio estaba inactivo.
    * La contraseña seleccionada no cumple las directivas de la organización.
    * No se encuentra el usuario en Active Directory local.

      Los mensajes de error ofrecen instrucciones a los usuarios para que pueden intentar resolver la situación sin intervención del administrador.

## <a name="password-writeback-security"></a>Seguridad de la escritura diferida de contraseñas

La escritura diferida de contraseñas es un servicio muy seguro. Para garantizar que su información esté protegida, se habilita un modelo de seguridad de cuatro niveles tal y como se describe a continuación:

* **Service Bus Relay específico del inquilino**
   * Al configurar el servicio, se configura una instancia de Service Bus Relay específica del inquilino que está protegida por una contraseña segura generada aleatoriamente y a la que Microsoft nunca tiene acceso.
* **Clave de cifrado de contraseñas bloqueadas y criptográficamente segura**
   * Una vez creada la instancia de Service Bus Relay, se crea una clave simétrica segura que se usa para cifrar la contraseña tal cual llega de la red. Esta clave solo reside en el almacén secreto en la nube de la compañía, el cual se bloquea y audita de forma estricta como se haría con cualquier contraseña del directorio.
* **Seguridad de la capa de transporte (TLS) estándar del sector**
   1. Cuando se produce una operación de restablecimiento o cambio de contraseña, la contraseña de texto no cifrado se cifra con la clave pública.
   1. La contraseña cifrada se inserta en un mensaje HTTPS que se envía a Service Bus Relay a través de un canal cifrado con certificados SSL de Microsoft.
   1. Una vez que el mensaje llega a Service Bus, el agente local se activa y se autentica en Service Bus con la contraseña segura que se generó anteriormente.
   1. El agente local selecciona el mensaje cifrado y lo descifra con la clave privada.
   1. El agente local intenta definir la contraseña con SetPassword API de AD DS. Este paso permite aplicar la directiva de contraseñas local de Active Directory (como, por ejemplo, complejidad, antigüedad, historial y filtros) en la nube.
* **Directivas de expiración de mensajes**
   * Si el mensaje se queda en Service Bus porque el servicio local está inactivo, se agotará el tiempo de espera y se eliminará después de varios minutos. El tiempo de espera y la eliminación del mensaje aumenta aún más la seguridad.

### <a name="password-writeback-encryption-details"></a>Detalles de cifrado de la escritura diferida de contraseñas

Después de que un usuario envíe un restablecimiento de contraseña, la solicitud de restablecimiento pasa por varias fases de cifrado antes de que llegue al entorno local. Estos pasos de cifrado garantizan una seguridad y confiabilidad máximas del servicio. A continuación se detalla la descripción de estos pasos:

* **Paso 1: Cifrado de contraseña con clave RSA de 2048 bits**: cuando el usuario envía una contraseña para que se escriba en diferido en el entorno local, se cifra la propia contraseña enviada con una clave RSA de 2048 bits.
* **Paso 2: cifrado a nivel de paquete con AES-GCM**: todo el paquete (la contraseña y los metadatos necesarios) se cifra mediante AES-GCM. Este cifrado evita que cualquier persona con acceso directo al canal de Service Bus subyacente vea o manipule el contenido.
* **Paso 3: Toda la comunicación se realiza a través de TLS/SSL**: toda comunicación con Service Bus tiene lugar en un canal SSL/TLS. Este cifrado protege el contenido de terceras personas no autorizadas.
* **Sustitución de clave automática cada seis meses**: todas las claves se sustituyen cada seis meses, o cada vez que la escritura diferida de contraseñas se deshabilita y luego se vuelve a habilitar en Azure AD Connect, para garantizar la máxima seguridad y protección del servicio.

### <a name="password-writeback-bandwidth-usage"></a>Uso de ancho de banda de la escritura diferida de contraseñas

La escritura diferida de contraseñas es un servicio de bajo consumo de ancho de banda que solo envía solicitudes al agente local si se dan las siguientes circunstancias:

* Se envían dos mensajes al habilitar o deshabilitar la característica a través de Azure AD Connect.
* Se envía un mensaje cada cinco minutos como un latido del servicio siempre que el servicio está en ejecución.
* Se envían dos mensajes cada vez que se envía una nueva contraseña:
   * El primer mensaje es una solicitud para realizar la operación.
   * El segundo mensaje, que contiene el resultado de la operación, se envía en las siguientes circunstancias:
      * Cada vez que se envía una nueva contraseña durante un restablecimiento de la contraseña de autoservicio de un usuario.
      * Cada vez que se envía una nueva contraseña durante una operación de cambio de la contraseña de un usuario.
      * Cada vez que se envía una nueva contraseña durante el restablecimiento de contraseña de usuario que inició el administrador (solo desde los portales de administración de Azure).

#### <a name="message-size-and-bandwidth-considerations"></a>Consideraciones sobre el ancho de banda y el tamaño de mensaje

El tamaño de cada uno de los mensajes que se ha descrito anteriormente suele ser inferior a 1 KB. Incluso bajo cargas extremas, el propio servicio de escritura diferida de contraseñas solo usa como máximo unos kilobits por segundo del ancho de banda. Puesto que cada mensaje se envía en tiempo real solo cuando lo requiere una operación de actualización de contraseña, y dado que el tamaño del mensaje es tan pequeño, el uso de ancho de banda de la funcionalidad de escritura diferida es demasiado pequeño para tener ningún impacto cuantificable real.

## <a name="supported-writeback-operations"></a>Operaciones de reescritura admitidas

Las contraseñas se escriben en diferido en todas las situaciones siguientes:

* **Operaciones de usuario final admitidas**
   * Cualquier operación de cambio de contraseña voluntaria de autoservicio del usuario final
   * Cualquier operación exigida de cambio de contraseña de autoservicio del usuario final (por ejemplo, la expiración de contraseña)
   * Cualquier restablecimiento de contraseña de autoservicio del usuario final que se origina en el [portal de restablecimiento de contraseñas](https://passwordreset.microsoftonline.com)
* **Operaciones de administrador admitidas**
   * Cualquier operación de cambio de contraseña voluntaria de autoservicio del administrador
   * Cualquier operación exigida de cambio de contraseña de autoservicio del administrador (por ejemplo, la expiración de contraseña)
   * Cualquier restablecimiento de contraseña de autoservicio del administrador que se origina en el [portal de restablecimiento de contraseñas](https://passwordreset.microsoftonline.com)
   * Cualquier restablecimiento de contraseña del usuario final iniciado por el administrador desde [Azure Portal](https://portal.azure.com)
   * Cualquier usuario final iniciado por el Administrador de restablecimiento de contraseña desde el [centro de administración de Microsoft 365](https://admin.microsoft.com)

## <a name="unsupported-writeback-operations"></a>Operaciones de reescritura no admitidas

Las contraseñas *no* se escriben en diferido en ninguna de las situaciones siguientes:

* **Operaciones de usuario final no admitidas**
   * Cualquier usuario final que restablezca su propia contraseña mediante PowerShell v1, v2 o Graph API de Azure AD
* **Operaciones de administrador no admitidas**
   * Cualquier restablecimiento de contraseña del usuario final que inicie el administrador desde PowerShell v1, v2 o Graph API de Azure AD

> [!WARNING]
> No se admite el uso de la casilla de verificación "el usuario debe cambiar la contraseña en el siguiente inicio de sesión" en Herramientas administrativas de Active Directory local, como usuarios de Active Directory y los equipos o el centro de administración de Active Directory. Al cambiar una contraseña en el entorno local no Active esta opción.

## <a name="next-steps"></a>Pasos siguientes

Habilitar la escritura diferida de contraseñas siguiendo el tutorial: [Habilitación de la escritura diferida de contraseñas](tutorial-enable-writeback.md)
