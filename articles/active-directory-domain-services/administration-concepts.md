---
title: Administración de conceptos para Azure AD Domain Services | Microsoft Docs
description: Obtenga información sobre cómo administrar un dominio administrado de Azure Active Directory Domain Services y el comportamiento de las cuentas de usuario y las contraseñas
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 06/05/2020
ms.author: iainfou
ms.openlocfilehash: 6266248b817485562c7ed2643b3dda5f32cecc53
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489680"
---
# <a name="management-concepts-for-user-accounts-passwords-and-administration-in-azure-active-directory-domain-services"></a>Conceptos de administración para cuentas de usuario, contraseñas y administración en Azure Active Directory Domain Services

Al crear y ejecutar un dominio administrado de Azure Active Directory Domain Services (AD DS), hay algunas diferencias en el comportamiento en comparación con un entorno de AD DS local tradicional. Puede usar las mismas herramientas administrativas en Azure AD DS como dominio autoadministrado, pero no puede obtener acceso directo a los controladores de dominio (DC). Hay algunas diferencias en el comportamiento para las directivas de contraseña y los valores hash de contraseñas dependiendo del origen de la creación de cuentas de usuario.

En este artículo conceptual se detalla cómo gestionar un dominio administrado y el comportamiento diferente de las cuentas de usuario dependiendo de cómo se crean.

## <a name="domain-management"></a>Administración de dominios

Un dominio administrado es un espacio de nombres DNS y un directorio coincidente. En un dominio administrado, los controladores de dominio (DC) que contienen todos los recursos, como usuarios y grupos, credenciales y directivas, forman parte del servicio administrado. Para ofrecer redundancia, se crean dos DC como parte de un dominio administrado. No puede iniciar sesión en estos DC para realizar tareas de administración. En su lugar, cree una máquina virtual de administración unida al dominio administrado y, a continuación, instale sus herramientas de administración de AD DS regulares. Puede usar los complementos del Centro de administración de Active Directory o Microsoft Management Console (MMC) como, por ejemplo, objetos de directiva de grupo o DNS.

## <a name="user-account-creation"></a>Creación de una cuenta de usuario

Las cuentas de usuario se pueden crear en un dominio administrado de varias formas. La mayoría de las cuentas de usuario se sincronizan desde Azure AD, lo que también puede incluir cuentas de usuario sincronizadas desde un entorno de AD DS local. También puede crear de forma manual cuentas directamente en el dominio administrado. Algunas características, como la directiva de contraseñas o la sincronización de contraseña inicial, se comportan de forma diferente dependiendo de cómo y dónde se crean las cuentas de usuario.

* La cuenta de usuario se puede sincronizar desde Azure AD. Esto incluye las cuentas de usuario solo en la nube creadas directamente en Azure AD y las cuentas de usuario híbridas sincronizadas desde un entorno de AD DS local mediante Azure AD Connect.
    * La mayoría de las cuentas de usuario de un dominio administrado se crean a través del proceso de sincronización de Azure AD.
* La cuenta de usuario se puede crear manualmente en un dominio administrado y no existe en Azure AD.
    * Si tiene que crear cuentas de servicio para aquellas aplicaciones que solo se ejecutan en el dominio administrado, puede crearlas manualmente en el dominio administrado. Como la sincronización desde Azure AD es unidireccional, las cuentas de usuario que se crearon en el dominio administrado no se vuelven a sincronizar con Azure AD.

## <a name="password-policy"></a>Directiva de contraseñas

Azure AD DS incluye una directiva de contraseñas predeterminada que define la configuración para elementos como el bloqueo de cuenta, la antigüedad máxima de contraseña y la complejidad de la contraseña. Opciones de configuración como la directiva de bloqueo de cuentas se aplican a todos los usuarios en un dominio administrado, independientemente de cómo se creó el usuario tal como se señala en la sección anterior. Algunos valores de configuración, como la longitud mínima de contraseña y la complejidad de la contraseña, solo se aplican a los usuarios creados directamente en un dominio administrado.

Puede crear sus propias directivas de contraseñas personalizadas para reemplazar la directiva predeterminada en un dominio administrado. Estas directivas personalizadas se pueden aplicar a continuación a grupos de usuarios específicos según sea necesario.

Para obtener más información sobre las diferencias en el modo en que se aplican las directivas de contraseñas dependiendo del origen de la creación de usuarios, consulte [Directivas de bloqueo de cuenta y contraseña en dominios administrados][password-policy].

## <a name="password-hashes"></a>Valores hash de contraseñas

Para autenticar a los usuarios en el dominio administrado, Azure AD DS necesita los hash de las contraseñas en un formato adecuado para la autenticación de NT LAN Manager (NTLM) y Kerberos. A menos que habilite Azure AD DS para el inquilino, Azure AD no genera ni almacena los hash de las contraseñas en el formato necesario para la autenticación NTLM o Kerberos. Por motivos de seguridad, Azure AD tampoco almacena las credenciales de contraseñas en forma de texto sin cifrar. Por consiguiente, Azure AD no tiene forma de generar automáticamente estos hash de las contraseñas de NTLM o Kerberos basándose en las credenciales existentes de los usuarios.

En el caso de las cuentas de usuario solo de nube, los usuarios deben cambiar sus contraseñas para poder usar el dominio administrado. Este proceso de cambio de contraseña hace que los valores hash de contraseña para la autenticación Kerberos y NTLM se generen y almacenen en Azure AD. La cuenta no se sincroniza de Azure AD a Azure AD DS hasta que se cambia la contraseña.

Para los usuarios sincronizados desde un entorno de AD DS local mediante Azure AD Connect, [habilite la sincronización de los valores hash de contraseñas][hybrid-phs].

> [!IMPORTANT]
> Azure AD Connect solo sincroniza los hashes de contraseña heredados al habilitar Azure AD DS para su inquilino de Azure AD. Los valores hash heredados no se usan si solo utiliza Azure AD Connect para sincronizar un entorno de AD DS local con Azure AD.
>
> Si las aplicaciones heredadas no utilizan la autenticación NTLM o los enlaces simples LDAP, se recomienda deshabilitar la sincronización de hash de contraseña de NTLM para Azure AD DS. Para más información, consulte [Deshabilitación de la sincronización de hash de credenciales NTLM y de conjuntos de cifrado débil][secure-domain].

Una vez configurados correctamente, los hash de las contraseñas que se pueden usar se almacenan en el dominio administrado. Si elimina el dominio administrado, también se eliminarán los hash de las contraseñas que haya almacenados en ese momento. La información de credenciales sincronizada en Azure AD no se puede volver a usar si posteriormente crea otro dominio administrado: debe volver a configurar la sincronización de los hash de contraseña para almacenarlos de nuevo. Los usuarios o las máquinas virtuales anteriormente unidos a un dominio no podrán autenticarse de inmediato, ya que Azure AD debe generar y almacenar los hash de contraseña en el nuevo dominio administrado. Para más información, consulte [Proceso de sincronización de hash de contraseña para Azure AD DS y Azure AD Connect][azure-ad-password-sync].

> [!IMPORTANT]
> Azure AD Connect solo debe instalarse y configurarse para la sincronización con entornos de AD DS locales. No se admite la instalación de Azure AD Connect en un dominio administrado para volver a sincronizar los objetos con Azure AD.

## <a name="forests-and-trusts"></a>Bosques y confianzas

Un *bosque* es una construcción lógica que Active Directory Domain Services (AD DS) usa para agrupar uno o más *dominios*. Estos dominios almacenan objetos para los usuarios o grupos, y proporcionan servicios de autenticación.

En Azure AD DS, el bosque solo contiene un dominio. Los bosques de entornos de AD DS locales suelen contener varios dominios. En organizaciones de gran tamaño, especialmente después de fusiones y adquisiciones, es posible que acabe con varios bosques locales y que cada uno contenga varios dominios.

De forma predeterminada, un dominio administrado se crea como un bosque de *usuario*. Este tipo de bosque sincroniza todos los objetos de Azure AD, incluidas las cuentas de usuario creadas en un entorno de AD DS local. Las cuentas de usuario se pueden autenticar directamente en el dominio administrado, por ejemplo, para iniciar sesión en una máquina virtual unida a un dominio. Un bosque de usuarios funciona cuando se pueden sincronizar los hash de contraseña y los usuarios no usan métodos de inicio de sesión exclusivos, como la autenticación de tarjeta inteligente.

En un bosque de *recursos* de Azure AD DS, los usuarios se autentican a través de una *confianza* de bosque unidireccional que procede del entorno de AD DS local. En esta estrategia, los objetos y hash de contraseña de los usuarios no se sincronizan con Azure AD DS. Los objetos y las credenciales de usuario solo existen en el entorno de AD DS local. Esto permite que las empresas hospeden en Azure recursos y plataformas de aplicaciones que dependen de la autenticación clásica, como LDAPS, Kerberos o NTLM, y que se olviden todos los problemas o las preocupaciones de autenticación.

Para obtener más información sobre los tipos de bosque de Azure AD DS, consulte [¿Qué son los bosques de recursos?][concepts-forest] y [¿Cómo funcionan las confianzas de bosque en Azure AD DS?][concepts-trust]

## <a name="azure-ad-ds-skus"></a>SKU de Azure AD DS

En Azure AD DS, el rendimiento y las características disponibles dependen de la SKU. Puede seleccionar la SKU al crear el dominio administrado y cambiarla después si los requisitos del negocio varían una vez implementado el dominio administrado. En la tabla siguiente, se incluyen las SKU disponibles y las diferencias entre ellas:

| Nombre de SKU   | Número máximo de objetos | Frecuencia de copia de seguridad | Número máximo de relaciones de confianza entre bosques de salida |
|------------|----------------------|------------------|----|
| Estándar   | Sin límite            | Cada 7 días     | 0  |
| Enterprise | Sin límite            | Cada 3 días     | 5  |
| Premium    | Sin límite            | Diario            | 10 |

Antes de utilizar estas SKU de Azure AD DS, se empleaba un modelo de facturación basado en el número de objetos (cuentas de usuario y de equipo) del dominio administrado. Este modelo de precios variables que se basaban en el número de objetos del dominio administrado ya no está disponible.

Para más información, consulte la [página de precios de Azure AD DS][pricing].

### <a name="managed-domain-performance"></a>Rendimiento del dominio administrado

El rendimiento del dominio varía en función del modo en que la autenticación de una aplicación esté implementada. Además, puede haber otros recursos que mejoren el tiempo de respuesta de las consultas y reduzcan el tiempo de las operaciones de sincronización. A medida que el nivel de SKU aumenta, también lo hacen los recursos de proceso disponibles en el dominio administrado. Supervise el rendimiento de las aplicaciones y planee los recursos necesarios.

Si el negocio o la aplicación exigen cambios y necesita más capacidad de proceso para el dominio administrado, puede cambiar a otra SKU.

### <a name="backup-frequency"></a>Frecuencia de copia de seguridad

La frecuencia de copia de seguridad determina la asiduidad con la que se toma una instantánea del dominio administrado. Las copias de seguridad son un proceso automatizado administrado por la plataforma Azure. Si se produce un problema con el dominio administrado, el Soporte técnico de Azure puede ayudarle a restaurar el sistema desde la copia de seguridad. Como la sincronización es un proceso unidireccional que se realiza *desde* Azure AD, los problemas que se produzcan en el dominio administrado no afectarán a Azure AD ni a la funcionalidad y los entornos locales de AD DS.

A medida que el nivel de SKU aumente, lo hará también la frecuencia con la que se realizan instantáneas de copia de seguridad. Analice los requisitos empresariales y el objetivo de punto de recuperación (RPO) para determinar la frecuencia de copia de seguridad necesaria para el dominio administrado. Si los requisitos del negocio o de la aplicación cambian y necesita hacer copias de seguridad con mayor frecuencia, puede cambiar a una SKU diferente.

### <a name="outbound-forest-trusts"></a>Confianzas entre bosques de salida

En la sección anterior, vimos las relaciones de confianza de bosques de salida unidireccionales entre un dominio administrado y un entorno de AD DS local. La SKU determina el número máximo de relaciones de confianza entre bosques que puede crear para un dominio administrado. Analice los requisitos del negocio y la aplicación para determinar cuántas relaciones de confianza necesita realmente y elija la SKU de Azure AD DS adecuada. Como siempre, si los requisitos del negocio cambian y necesita crear otras relaciones de confianza entre bosques, puede cambiar a una SKU diferente.

## <a name="next-steps"></a>Pasos siguientes

Para empezar, [cree un dominio administrado de Azure AD DS][create-instance].

<!-- INTERNAL LINKS -->
[password-policy]: password-policy.md
[hybrid-phs]: tutorial-configure-password-hash-sync.md#enable-synchronization-of-password-hashes
[secure-domain]: secure-your-domain.md
[azure-ad-password-sync]: ../active-directory/hybrid/how-to-connect-password-hash-synchronization.md#password-hash-sync-process-for-azure-ad-domain-services
[create-instance]: tutorial-create-instance.md
[tutorial-create-instance-advanced]: tutorial-create-instance-advanced.md
[concepts-forest]: concepts-resource-forest.md
[concepts-trust]: concepts-forest-trust.md

<!-- EXTERNAL LINKS -->
[pricing]: https://azure.microsoft.com/pricing/details/active-directory-ds/
