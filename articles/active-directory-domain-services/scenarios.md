---
title: Escenarios de implementación comunes para Azure AD Domain Services | Microsoft Docs
description: Obtenga información sobre algunos de los escenarios comunes y casos de uso de Azure Active Directory Domain Services para proporcionar valor y satisfacer las necesidades empresariales.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: c5216ec9-4c4f-4b7e-830b-9d70cf176b20
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 08/14/2020
ms.author: iainfou
ms.openlocfilehash: a27d97e7e030da216b2296f11b6876ccf28c8ad0
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88722762"
---
# <a name="common-use-cases-and-scenarios-for-azure-active-directory-domain-services"></a>Escenarios y casos de uso comunes para Azure Active Directory Domain Services

Azure Active Directory Domain Services (Azure AD DS) proporciona servicios de dominio administrados como, por ejemplo, unión a un dominio, directivas de grupo, protocolo ligero de acceso a directorios (LDAP) y autenticación Kerberos o NTLM. Azure AD DS se integra con el inquilino de Azure AD existente, lo que posibilita que los usuarios inicien sesión con sus credenciales existentes. Estos servicios de dominio se usan sin necesidad de implementar, administrar y aplicar revisiones a controladores de dominio en la nube, lo que proporciona una migración mediante lift-and-shift más suave de los recursos locales a Azure.

En este artículo se describen algunos escenarios empresariales comunes en los que Azure AD DS proporciona valor y satisface esas necesidades.

## <a name="common-ways-to-provide-identity-solutions-in-the-cloud"></a>Formas comunes de proporcionar soluciones de identidad en la nube

Al migrar las cargas de trabajo existentes a la nube, las aplicaciones que tienen en cuenta el directorio pueden usar LDAP para el acceso de lectura o escritura a un directorio de AD DS local. Las aplicaciones que se ejecutan en Windows Server suelen implementarse en máquinas virtuales unidas a un dominio, de modo que se pueden administrar de forma segura mediante directivas de grupo. Para autenticar a los usuarios finales, las aplicaciones también pueden confiar en la autenticación integrada de Windows, como la autenticación Kerberos o NTLM.

A menudo, los administradores de TI usan una de las siguientes soluciones para proporcionar un servicio de identidad a las aplicaciones que se ejecutan en Azure:

* Configure una conexión VPN de sitio a sitio entre las cargas de trabajo que se ejecuten en Azure y en un entorno de AD DS local.
    * Los controladores de dominio locales proporcionan autenticación a través de la conexión VPN.
* Cree controladores de dominio de réplica con máquinas virtuales de Azure para extender el dominio o bosque de AD DS.
    * Los controladores de dominio que se ejecutan en máquinas virtuales de Azure proporcionan autenticación y replican información de directorio entre el entorno de AD DS local.
* Implemente un entorno de AD DS independiente en Azure mediante controladores de dominio que se ejecuten en máquinas virtuales Azure.
    * Los controladores de dominio que se ejecutan en máquinas virtuales de Azure proporcionan autenticación, pero no se replica la información de directorio desde un entorno de AD DS local.

Con estos métodos, las conexiones VPN con el directorio en el entorno local hacen que las aplicaciones sean vulnerables a interrupciones o problemas de red transitorios. Si implementa controladores de dominio mediante máquinas virtuales en Azure, el equipo de TI deben ocuparse de las tareas de administración, protección, revisión, supervisión, copia de seguridad y solución de problemas de las máquinas virtuales.

Azure AD DS ofrece alternativas a la necesidad de volver a crear las conexiones VPN en un entorno de AD DS local o ejecutar y administrar máquinas virtuales en Azure para proporcionar servicios de identidad. Como servicio administrado, Azure AD DS reduce la complejidad de crear una solución de identidad integrada para entornos tanto híbridos como solo en la nube.

> [!div class="nextstepaction"]
> [Comparar Azure AD DS con Azure AD y AD DS autoadministrado en VM de Azure o locales][compare]

## <a name="azure-ad-ds-for-hybrid-organizations"></a>Azure AD DS para organizaciones híbridas

Muchas organizaciones ejecutan una infraestructura híbrida que incluye cargas de trabajo de aplicaciones en el entorno local y en la nube. Las aplicaciones heredadas migradas a Azure como parte de una migración mediante lift-and-shift pueden usar las conexiones LDAP tradicionales para proporcionar información de identidad. Para admitir esta infraestructura híbrida, la información de identidad de un entorno de AD DS local se puede sincronizar con un inquilino de Azure AD. A continuación, Azure AD DS proporciona estas aplicaciones heredadas en Azure con un origen de identidad, sin necesidad de configurar y administrar la conectividad de la aplicación de nuevo en los servicios de directorio en el entorno local.

Echemos un vistazo a un ejemplo de Litware Corporation, una organización híbrida que ejecuta recursos locales y de Azure:

![Azure Active Directory Domain Services para una organización híbrida que incluye sincronización en el entorno local](./media/overview/synced-tenant.png)

* Las aplicaciones y cargas de trabajo de servidor que requieren servicios de dominio se implementan en una red virtual en Azure.
    * Puede que se incluyan aplicaciones heredadas migradas a Azure como parte de una estrategia de migración mediante lift-and-shift.
* Para sincronizar la información de identidad desde el directorio en el entorno local a su inquilino de Azure AD, Litware Corporation implementa [Azure AD Connect][azure-ad-connect].
    * La información de identidad que se sincroniza incluye las cuentas de usuario y las pertenencias a grupos.
* El equipo de TI de Litware habilita Azure AD DS para su inquilino de Azure AD en esta red virtual o en una emparejada.
* Las aplicaciones y las máquinas virtuales implementadas en la red virtual pueden usar características de Azure AD DS como, por ejemplo, la unión a un dominio, la lectura LDAP, el enlace LDAP, la autenticación NTLM y Kerberos, y directiva de grupo.

> [!IMPORTANT]
> Azure AD Connect solo debe instalarse y configurarse para la sincronización con entornos de AD DS locales. No se admite la instalación de Azure AD Connect en un dominio administrado para volver a sincronizar los objetos con Azure AD.

## <a name="azure-ad-ds-for-cloud-only-organizations"></a>Azure AD DS para organizaciones solo en la nube

Un inquilino de Azure AD solo en la nube no tiene un origen de identidad en el entorno local. Las cuentas de usuario y las pertenencias a grupos, por ejemplo, se crean y administran directamente en Azure AD.

Ahora veamos un ejemplo de Contoso, una organización solo en la nube que usa Azure AD para la identidad. Todas las identidades de usuario, sus credenciales y las pertenencias a grupos se crean y administran en Azure AD. No hay ninguna configuración adicional de Azure AD Connect para sincronizar cualquier información de identidad desde un directorio en el entorno local.

![Azure Active Directory Domain Services para una organización solo en la nube sin sincronización fuera del entorno local](./media/overview/cloud-only-tenant.png)

* Las aplicaciones y cargas de trabajo de servidor que requieren servicios de dominio se implementan en una red virtual en Azure.
* El equipo de TI de Litware Contoso habilita Azure AD DS para su inquilino de Azure AD en esta red virtual o en una emparejada.
* Las aplicaciones y las máquinas virtuales implementadas en la red virtual pueden usar características de Azure AD DS como, por ejemplo, la unión a un dominio, la lectura LDAP, el enlace LDAP, la autenticación NTLM y Kerberos, y directiva de grupo.

## <a name="secure-administration-of-azure-virtual-machines"></a>Administración segura de máquinas virtuales de Azure

Para permitir el uso de un solo conjunto de credenciales de AD, las máquinas virtuales de Azure se pueden unir a un dominio administrado de Azure AD DS. Este enfoque reduce los problemas de administración de credenciales, como el mantenimiento de cuentas de administrador local en cada máquina virtual o la separación de cuentas y contraseñas entre entornos.

Las máquinas virtuales que están unidas a un dominio administrado también se pueden administrar y proteger con la directiva de grupo. Las líneas de base de seguridad necesarias se pueden aplicar a las máquinas virtuales para bloquearlas con arreglo a las directrices de seguridad corporativa. Por ejemplo, puede utilizar las funcionalidades de administración de directiva de grupo para restringir los tipos de aplicaciones que pueden iniciarse en la máquina virtual.

![Administración simplificada de máquinas virtuales de Azure](./media/active-directory-domain-services-scenarios/streamlined-vm-administration.png)

Echemos un vistazo a un escenario de ejemplo común. A medida que los servidores y otras infraestructuras alcanzan el final de su ciclo de vida, Contoso quiere trasladar a la nube muchas aplicaciones que actualmente se hospedan de forma local. Su norma actual de TI dictamina que los servidores que hospedan aplicaciones corporativas se deben unir a un dominio y administrarse mediante la directiva de grupo.

El administrador de TI de Contoso preferiría unirse a un dominio de máquinas virtuales implementadas en Azure para facilitar la administración, ya que los usuarios pueden iniciar sesión con sus credenciales corporativas. Cuando están unidas a un dominio, las máquinas virtuales pueden configurarse también para cumplir con las líneas de base de seguridad necesarias mediante objetos de directiva de grupo. Contoso preferiría no tener que implementar, supervisar y administrar sus propios controladores en Azure.

Azure AD DS es una buena opción para este caso de uso. Un dominio administrado le permite unirse a un dominio de máquinas virtuales, usar un único conjunto de credenciales y aplicar la directiva de grupo. Y dado que es un dominio administrado, no tiene que configurar y mantener los controladores de dominio.

### <a name="deployment-notes"></a>Notas de implementación

Las siguientes consideraciones de implementación se aplican a este caso de uso de ejemplo:

* Los dominios administrados usan de forma predeterminada una única estructura de unidad organizativa (OU) plana. Todas las máquinas virtuales unidas a un dominio se encuentran en una sola unidad organizativa. Si lo desea, puede crear [unidades organizativas personalizadas][custom-ou].
* Azure AD DS utiliza un GPO integrado para cada uno de los contenedores de equipos y usuarios. Para un control adicional, puede [crear GPO personalizados][create-gpo] y dirigirlos a unidades organizativas personalizadas.
* Azure AD DS admite el esquema base del objeto de equipo de AD. No puede extender el esquema del objeto de equipo.

## <a name="lift-and-shift-on-premises-applications-that-use-ldap-bind-authentication"></a>Migración mediante lift-and-shift de las aplicaciones locales que usan la autenticación de enlace LDAP

Como un escenario de ejemplo, Contoso cuenta con una aplicación local que se compró a un ISV hace muchos años. La aplicación está actualmente en modo de mantenimiento por el ISV y resulta excesivamente caro solicitar cambios en la aplicación. Esta aplicación tiene un front-end basado en web que recopila las credenciales de usuario mediante un formulario web y luego autentica a los usuarios mediante la realización de un enlace LDAP al entorno local de AD DS.

![Enlace LDAP](./media/active-directory-domain-services-scenarios/ldap-bind.png)

A Contoso le gustaría migrar esta aplicación a Azure. La aplicación debe continuar funcionando tal cual, sin necesidad de realizar cambios. Además, los usuarios deben poder autenticarse con las credenciales corporativas existentes y sin entrenamiento adicional. Debe ser transparente para los usuarios finales, donde que se ejecuta la aplicación.

En este escenario, Azure AD DS permite a las aplicaciones realizar enlaces LDAP como parte del proceso de autenticación. Las aplicaciones locales heredadas pueden migrar mediante lift-and-shift a Azure y continuar seguir autenticando a los usuarios sin ningún cambio en la configuración o la experiencia del usuario.

### <a name="deployment-notes"></a>Notas de implementación

Las siguientes consideraciones de implementación se aplican a este caso de uso de ejemplo:

* Asegúrese de que la aplicación no necesita modificar o escribir en el directorio. No se admite el acceso de escritura LDAP a un dominio administrado.
* No se pueden cambiar las contraseñas directamente en un dominio administrado. Los usuarios finales pueden cambiar su contraseña bien con el [mecanismo de autoservicio de cambio de contraseña de Azure AD][sspr] o en el directorio local. Estos cambios se sincronizan automáticamente y están disponibles en el dominio administrado.

## <a name="lift-and-shift-on-premises-applications-that-use-ldap-read-to-access-the-directory"></a>Migración mediante lift-and-shift de las aplicaciones locales que usan la lectura LDAP para acceder al directorio

Al igual que en el escenario de ejemplo anterior, supongamos que Contoso tiene una aplicación de línea de negocio (LOB) local que se desarrolló hace casi una década. Esta aplicación es compatible con directorios y se ha diseñado para usar LDAP para leer información o atributos acerca de los usuarios de AD DS. La aplicación no modifica los atributos ni escribe en el directorio.

Contoso quiere migrar esta aplicación a Azure y retirar el hardware local antiguo que actualmente hospeda esta aplicación. La aplicación no se puede volver a escribir para que use las modernas API de directorio, como Microsoft Graph API basada en REST. Se necesita una opción de migración mediante lift-and-shift mediante la cual se pueda migrar la aplicación para ejecutarse en la nube, sin modificar el código o volver a escribirla.

Para ayudar con este escenario, Azure AD DS permite a las aplicaciones realizar lecturas LDAP en el dominio administrado para obtener la información de los atributos que necesita. No es necesario volver a escribir la aplicación, por lo que una migración mediante lift-and-shift en Azure permite a los usuarios seguir usando la aplicación sin darse cuenta de que hay un cambio en el lugar donde se ejecuta.

### <a name="deployment-notes"></a>Notas de implementación

Las siguientes consideraciones de implementación se aplican a este caso de uso de ejemplo:

* Asegúrese de que la aplicación no necesita modificar o escribir en el directorio. No se admite el acceso de escritura LDAP a un dominio administrado.
* Asegúrese de que la aplicación no necesita un esquema de Active Directory ampliado o personalizado. No se admiten extensiones de esquema en Azure AD DS.

## <a name="migrate-an-on-premises-service-or-daemon-application-to-azure"></a>Migración de una aplicación de dominio o servicio local a Azure

Algunas aplicaciones incluyen varios niveles, donde uno de los niveles necesita realizar llamadas autenticadas a un nivel de back-end, como una base de datos. Las cuentas de servicio de AD se usan normalmente en estos escenarios. Al migrar mediante lift-and-shift las aplicaciones en Azure, Azure AD DS le permite seguir usando cuentas de servicio de la misma manera. Puede optar por usar la misma cuenta de servicio que se sincroniza desde el directorio local para Azure AD o crear una unidad organizativa personalizada y, a continuación, crear una cuenta de servicio independiente en esa unidad organizativa. Con cualquiera de estos enfoques, las aplicaciones siguen funcionando de la misma manera para realizar llamadas autenticadas a otros niveles y servicios.

![Cuenta de servicio mediante WIA](./media/active-directory-domain-services-scenarios/wia-service-account.png)

En este escenario de ejemplo, Contoso tiene una aplicación de almacén de software personalizada que incluye un front-end web, un servidor SQL y un servidor FTP de back-end. La autenticación integrada de Windows mediante las cuentas de servicio autentica el web front-end en el servidor FTP. El front-end web está configurado para ejecutarse como una cuenta de servicio. El servidor back-end está configurado para autorizar el acceso de la cuenta de servicio para el front-end web. Contoso no desea implementar y administrar sus propias máquinas virtuales de controlador de dominio en la nube para migrar esta aplicación a Azure.

En este escenario, los servidores que hospedan el front-end web, SQL Server y el servidor FTP se pueden migrar a máquinas virtuales de Azure y unirse a un dominio administrado. Las máquinas virtuales pueden usar la misma cuenta de servicio en su directorio local para los propósitos de autenticación de la aplicación, que se sincroniza a través de Azure AD con Azure AD Connect.

### <a name="deployment-notes"></a>Notas de implementación

Las siguientes consideraciones de implementación se aplican a este caso de uso de ejemplo:

* Asegúrese de que las aplicaciones usan un nombre de usuario y una contraseña para la autenticación. Azure AD DS no admite la autenticación basada en certificado o tarjeta inteligente.
* No se pueden cambiar las contraseñas directamente en un dominio administrado. Los usuarios finales pueden cambiar su contraseña bien con el [mecanismo de autoservicio de cambio de contraseña de Azure AD][sspr] o en el directorio local. Estos cambios se sincronizan automáticamente y están disponibles en el dominio administrado.

## <a name="windows-server-remote-desktop-services-deployments-in-azure"></a>Implementaciones de Servicios de Escritorio Remoto de Windows Server en Azure

Puede usar Azure AD DS para proporcionar servicios de dominio administrado a los servidores de escritorio remotos implementados en Azure.

Para más información acerca de este escenario de implementación, vea [cómo integrar Azure AD Domain Services con su implementación de RDS][windows-rds].

## <a name="domain-joined-hdinsight-clusters"></a>Usuarios de clústeres de HDInsight unidos a dominio

Puede configurar un clúster de Azure HDInsight que esté unido a un dominio administrado con Apache Ranger habilitado. Puede crear y aplicar directivas de Hive a través de Apache Ranger y permitir a los usuarios, como los científicos de datos, conectarse a Hive con herramientas basadas en ODBC, como Excel o Tableau. Seguimos trabajando para incluir otras cargas de trabajo, como HBase, Spark y Storm a HDInsight unido a un dominio.

Para más información sobre este escenario de implementación, consulte [cómo configurar clústeres de HDInsight unidos a un dominio][hdinsight]

## <a name="next-steps"></a>Pasos siguientes

Para comenzar, [Crear y configurar un dominio administrado de Azure Active Directory Domain Services][tutorial-create-instance].

<!-- INTERNAL LINKS -->
[hdinsight]: ../hdinsight/domain-joined/apache-domain-joined-configure-using-azure-adds.md
[tutorial-create-instance]: tutorial-create-instance.md
[custom-ou]: create-ou.md
[create-gpo]: manage-group-policy.md
[sspr]: ../active-directory/authentication/overview-authentication.md#self-service-password-reset
[compare]: compare-identity-solutions.md
[azure-ad-connect]: ../active-directory/hybrid/whatis-azure-ad-connect.md

<!-- EXTERNAL LINKS -->
[windows-rds]: /windows-server/remote/remote-desktop-services/rds-azure-adds