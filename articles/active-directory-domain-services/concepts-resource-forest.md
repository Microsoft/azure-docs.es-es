---
title: Conceptos del bosque de recursos en Azure AD Domain Services | Microsoft Docs
description: Obtenga información sobre qué es un bosque de recursos en Azure Active Directory Domain Services y cómo beneficia a su organización en un entorno híbrido con opciones de autenticación de usuario limitadas o problemas de seguridad.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: 62a2ffeea1d15a16c4ec4aa6a2b88c8e34763064
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87480414"
---
# <a name="resource-forest-concepts-and-features-for-azure-active-directory-domain-services"></a>Conceptos y características del bosque de recursos en Azure Active Directory Domain Services

Azure Active Directory Domain Services (Azure AD DS) proporciona una experiencia de inicio de sesión para las aplicaciones de línea de negocio heredadas locales. Los usuarios, grupos y hash de contraseña de los usuarios locales y en la nube se sincronizan en el dominio administrado de Azure AD DS. Estos hash de contraseña sincronizados proporcionan a los usuarios un único conjunto de credenciales que pueden usar para AD DS, Office 365 y Azure Active Directory en el entorno local.

Si bien este modo es seguro y aporta ventajas de seguridad adicionales, algunas organizaciones no pueden sincronizar los hash de contraseñas de usuario en Azure AD o Azure AD DS. Es posible, por ejemplo, que los usuarios de una organización no conozcan su contraseña porque solo usan la autenticación de tarjeta inteligente. Estas limitaciones impiden que algunas organizaciones usen Azure AD DS para migrar mediante lift-and-shift las aplicaciones clásicas locales a Azure.

Para abordar estas necesidades y restricciones, puede crear un dominio administrado que use un bosque de recursos. En este artículo conceptual se explica qué son los bosques y cómo confían en otros recursos para proporcionar un método de autenticación seguro.

## <a name="what-are-forests"></a>¿Qué son los bosques?

Un *bosque* es una construcción lógica que Active Directory Domain Services (AD DS) usa para agrupar uno o más *dominios*. Estos dominios almacenan objetos para los usuarios o grupos, y proporcionan servicios de autenticación.

En un dominio administrado de Azure AD DS, el bosque solo contiene un dominio. Los bosques de entornos de AD DS locales suelen contener varios dominios. En organizaciones de gran tamaño, especialmente después de fusiones y adquisiciones, es posible que acabe con varios bosques locales y que cada uno contenga varios dominios.

De forma predeterminada, un dominio administrado se crea como un bosque de *usuario*. Este tipo de bosque sincroniza todos los objetos de Azure AD, incluidas las cuentas de usuario creadas en un entorno de AD DS local. Las cuentas de usuario se pueden autenticar directamente en el dominio administrado, por ejemplo, para iniciar sesión en una máquina virtual unida a un dominio. Un bosque de usuarios funciona cuando se pueden sincronizar los hash de contraseña y los usuarios no usan métodos de inicio de sesión exclusivos, como la autenticación de tarjeta inteligente.

En un bosque de *recursos* de Azure AD DS, los usuarios se autentican a través de una *confianza* de bosque unidireccional que procede del entorno de AD DS local. Con este enfoque, los objetos de usuario y los valores hash de contraseña no se sincronizan con el dominio administrado. Los objetos y las credenciales de usuario solo existen en el entorno de AD DS local. Esto permite que las empresas hospeden en Azure recursos y plataformas de aplicaciones que dependen de la autenticación clásica, como LDAPS, Kerberos o NTLM, y que se olviden todos los problemas o las preocupaciones de autenticación.

Los bosques de recursos también proporcionan la funcionalidad para migrar mediante lift-and-shift las aplicaciones, un componente a la vez. Muchas aplicaciones locales heredadas son de varios niveles, a menudo con un servidor web o front-end y muchos componentes relacionados con la base de datos. Estos niveles dificultan la migración mediante lift-and-shift de toda la aplicación a la nube en un solo paso. Gracias a los bosques de recursos, puede realizar esta operación en un enfoque por fases, lo que facilita el traslado de la aplicación a Azure.

## <a name="what-are-trusts"></a>¿Qué son las confianzas?

Las organizaciones que tienen más de un dominio suelen necesitar que los usuarios accedan a recursos compartidos de un dominio diferente. El acceso a estos recursos compartidos requiere que los usuarios de un dominio se autentiquen en otro dominio. Para disponer de estas características de autenticación y autorización entre clientes y servidores de dominios diferentes, debe haber *confianza* entre los dos dominios.

Gracias a la confianza de dominio, los mecanismos de autenticación de un dominio confían en las autenticaciones que proceden del otro dominio. La confianza ayuda a proporcionar acceso controlado a los recursos compartidos de un dominio de recursos (el dominio *que confía*) asegurando que las solicitudes de autenticación entrantes proceden de una entidad de confianza (el dominio *de confianza*). La confianza actúa como un puente que solo permite que pasen entre los dominios las solicitudes de autenticación validadas.

La forma en que la confianza pasa las solicitudes de autenticación depende de cómo esté configurada. La confianza se puede configurar de una de las siguientes maneras:

* **Unidireccional**: proporciona acceso desde el dominio de confianza a los recursos del dominio que confía.
* **Bidireccional**: proporciona acceso desde cada dominio a los recursos del otro dominio.

La confianza también se configura para administrar relaciones de confianza adicionales de una de las siguientes maneras:

* **No transitiva**: la confianza solo existe entre los dos dominios asociados de confianza.
* **Transitiva**: la confianza se extiende automáticamente a cualquier otro dominio en el que cada uno de los asociados confíe.

En algunos casos, las relaciones de confianza se establecen automáticamente cuando se crean los dominios. Otras veces, es necesario elegir un tipo de confianza y establecer explícitamente las relaciones adecuadas. Los tipos específicos de confianza que se usan y la estructura de esas relaciones de confianza dependen de cómo esté organizado el directorio de AD DS y de si las distintas versiones de Windows coexisten en la red.

## <a name="trusts-between-two-forests"></a>Confianza entre dos bosques

Puede extender las confianzas de dominio disponibles en un bosque a otro, mediante la creación manual de una confianza de bosque unidireccional o bidireccional. Una confianza de bosque es una confianza transitiva que solo existe entre el dominio raíz de un bosque y el dominio raíz de un segundo bosque.

* Una confianza de bosque unidireccional permite que todos los usuarios de un bosque confíen en todos los dominios del otro bosque.
* Una confianza de bosque bidireccional forma una relación de confianza transitiva entre todos los dominios de ambos bosques.

La transitividad de las confianzas de bosque se limita a los dos bosques asociados. La confianza de bosque no se extiende a otros bosques de confianza de cualquiera de los asociados.

![Diagrama de la confianza de bosque desde Azure AD DS a un entorno local de AD DS](./media/concepts-resource-forest/resource-forest-trust-relationship.png)

Puede crear diferentes configuraciones de confianza de dominio y de bosque en función de la estructura de AD DS de la organización. Azure AD DS solo admite la confianza de bosque unidireccional. En esta configuración, los recursos del dominio administrado pueden confiar en todos los dominios de un bosque local.

## <a name="supporting-technology-for-trusts"></a>Tecnología compatible para confianzas

Las confianzas usan diversos servicios y características, como DNS, para buscar controladores de dominio en los bosques asociados. Las confianzas también dependen de los protocolos de autenticación NTLM y Kerberos así como de los mecanismos de autorización y control de acceso basados en Windows para ayudar a proporcionar una infraestructura de comunicaciones segura entre los dominios y bosques de AD DS. Los siguientes servicios y características ayudan a establecer relaciones de confianza correctas.

### <a name="dns"></a>DNS

AD DS necesita DNS para la ubicación y la nomenclatura de los controladores de dominio. Para que AD DS funcione correctamente, se proporciona la siguiente compatibilidad con DNS:

* Un servicio de resolución de nombres que permite a los hosts y servicios de red localizar controladores de dominio.
* Una estructura de nomenclatura que permite a una empresa reflejar su estructura organizativa en los nombres de sus dominios del servicio de directorio.

Normalmente se implementa un espacio de nombres de dominio DNS que refleja el espacio de nombres de dominio de AD DS. Si ya existe un espacio de nombres DNS anterior a la implementación de AD DS, normalmente se particiona el espacio de nombres DNS para AD DS y se crea un subdominio DNS y una delegación para la raíz del bosque AD DS. Después se agregan nombres de dominio DNS adicionales para cada dominio secundario AD DS.

DNS también se usa para admitir la ubicación de los controladores de AD DS. Las zonas DNS se rellenan con registros de recursos DNS que permiten a los hosts y servicios de red ubicar AD DS DC.

### <a name="applications-and-net-logon"></a>Aplicaciones y Net Logon

Tanto las aplicaciones como el servicio de Net Logon son componentes del modelo de canal de seguridad distribuida de Windows. Las aplicaciones integradas con Windows Server y AD DS usan protocolos de autenticación para comunicarse con el servicio de inicio de sesión de red, de forma que se pueda establecer una ruta de acceso segura a través de la cual se puede producir la autenticación.

### <a name="authentication-protocols"></a>Protocolos de autenticación

Los controladores de aplicación AD DS autentican a los usuarios y las aplicaciones mediante uno de los siguientes protocolos:

* **Protocolo de autenticación Kerberos versión 5**
    * El protocolo Kerberos versión 5 es el protocolo de autenticación predeterminado que usan los equipos locales que ejecutan Windows y admiten sistemas operativos de terceros. Este protocolo se especifica en RFC 1510 y está totalmente integrado con AD DS, bloque de mensajes del servidor (SMB), HTTP y llamada a procedimiento remoto (RPC), así como las aplicaciones cliente y servidor que utilizan estos protocolos.
    * Cuando se usa el protocolo Kerberos, el servidor no tiene que ponerse en contacto con el controlador de dominio. En su lugar, el cliente obtiene un vale para un servidor solicitándolo a un controlador de dominio del dominio correspondiente a la cuenta del servidor. Después, el servidor valida el vale sin consultar con ninguna otra entidad.
    * Si algún equipo implicado en una transacción no es compatible con el protocolo Kerberos versión 5, se usa el protocolo NTLM.

* **Protocolo de autenticación NTLM**
    * El protocolo NTLM es un protocolo de autenticación de red clásico usado por sistemas operativos anteriores. Por motivos de compatibilidad, los dominios de AD DS usan para procesar solicitudes de autenticación de red que proceden de aplicaciones diseñadas para clientes y servidores anteriores basados en Windows y sistemas operativos de terceros.
    * Cuando se usa el protocolo NTLM entre un cliente y un servidor, el servidor debe ponerse en contacto con un servicio de autenticación de dominios en un controlador de dominio, para comprobar las credenciales del cliente. El servidor autentica al cliente mediante el reenvío de las credenciales de este a un controlador de dominio en el dominio de la cuenta del cliente.
    * Cuando dos dominios o bosques de AD DS están conectados con una confianza, se pueden enrutar las solicitudes de autenticación realizadas con estos protocolos para proporcionar acceso a los recursos de ambos bosques.

## <a name="authorization-and-access-control"></a>Control de autorización y acceso

Las tecnologías de autorización y de confianza colaboran para proporcionar una infraestructura de comunicaciones segura en dominios o bosques de AD DS. La autorización determina el nivel de acceso de un usuario a los recursos de un dominio. La confianza facilita la autorización entre dominios de los usuarios, al proporcionar una ruta de acceso para autenticar a los usuarios de otros dominios, de modo que se puedan autorizar sus solicitudes para recursos compartidos en esos dominios.

Cuando el dominio de confianza valida una solicitud de autenticación realizada en un dominio que confía, la solicitud se pasa al recurso de destino. A continuación, el recurso de destino determina si debe autorizar la solicitud específica realizada por el usuario, el servicio o el equipo en el dominio de confianza, en función de su configuración de control de acceso.

La confianza proporciona este mecanismo para validar las solicitudes de autenticación que se pasan a un dominio que confía. Los mecanismos de control de acceso en el equipo del recurso determinan el nivel final de acceso concedido al solicitante en el dominio de confianza.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de la confianza, consulte [Funcionamiento de la confianza de bosque en Azure AD DS][concepts-trust].

Para empezar a crear un dominio administrado con un bosque de recursos, consulte [Crear y configurar un dominio administrado de Azure AD DS][tutorial-create-advanced]. Después puede continuar en [Creación de una confianza de bosque de salida a un dominio local][create-forest-trust].

<!-- LINKS - INTERNAL -->
[concepts-trust]: concepts-forest-trust.md
[tutorial-create-advanced]: tutorial-create-instance-advanced.md
[create-forest-trust]: tutorial-create-forest-trust.md
