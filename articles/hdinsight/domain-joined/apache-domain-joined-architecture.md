---
title: Arquitectura de Azure HDInsight con Enterprise Security Package
description: Aprenda a planear la seguridad de HDInsight con Enterprise Security Package.
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: omidm
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 069b3fe89183d7897cea39e2a8a1fe1bbed80bb0
ms.sourcegitcommit: 9e8dfa1169a55c3c8af93a6c5f4e0dace4de48b2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65556348"
---
# <a name="use-enterprise-security-package-in-hdinsight"></a>Uso de Enterprise Security Package en HDInsight

El clúster de Azure HDInsight estándar es de un único usuario. Es apropiado para la mayor parte de las empresas que tienen equipos de aplicaciones pequeños que generan grandes cargas de trabajo de datos. Cada usuario puede crear un clúster dedicado a petición y destruirlo cuando ya no sea necesario. 

Muchas empresas han migrado a un modelo en el que los equipos de TI administran los clústeres y varios equipos de aplicaciones los comparten. Estas empresas más grandes necesitan acceso multiusuario a cada clúster de Azure HDInsight.

HDInsight se basa en un proveedor de identidades conocido (Active Directory) de una manera administrada. Mediante la integración de HDInsight con [Azure Active Directory Domain Services (Azure AD DS)](../../active-directory-domain-services/active-directory-ds-overview.md), puede acceder a los clústeres con sus credenciales de dominio. 

Las máquinas virtuales (VM) de HDInsight están unidas al dominio proporcionado. De este modo, todos los servicios que se ejecutan en HDInsight (Apache Ambari, servidor de Apache Hive, Apache Ranger, servidor Thrift de Apache Spark, etc.) funcionan perfectamente para el usuario autenticado. Los administradores pueden crear entonces directivas de autorización seguras con Apache Ranger para proporcionar control de acceso basado en rol para los recursos del clúster.

## <a name="integrate-hdinsight-with-active-directory"></a>Integración de HDInsight con Active Directory

Apache Hadoop de código abierto se basa en el protocolo Kerberos para proporcionar autenticación y seguridad. Por consiguiente, los nodos del clúster de HDInsight con Enterprise Security Package (ESP) se combinan en un dominio que administra Azure AD DS. Se configura la seguridad de Kerberos para los componentes de Hadoop en el clúster. 

Las siguientes cosas se crean automáticamente:

- Una entidad de servicio para cada componente de Hadoop.
- Una entidad de seguridad de máquina para cada máquina unida al dominio.
- Una unidad organizativa (UO) para cada clúster para almacenar estas entidades de servicio y de máquina.

En resumen, es necesario configurar un entorno con:

- Un dominio de Active Directory (administrado por Azure AD DS).
- LDAP seguro (LDAPS) habilitado en Azure AD DS.
- Conectividad de red adecuada desde la red virtual de HDInsight a la red virtual de Azure AD DS, si elige redes virtuales independientes para ellas. Una máquina virtual dentro de la red virtual de HDInsight debe tener una línea de visión a Azure AD DS a través del emparejamiento de la red virtual. Si HDInsight y Azure AD DS se implementan en la misma red virtual, la conectividad se proporciona de manera automática y no es necesaria ninguna otra acción.

## <a name="set-up-different-domain-controllers"></a>Configuración de distintos controladores de dominio
HDInsight actualmente solo admite Azure AD DS como el controlador de dominio principal que el clúster usa para la comunicación de Kerberos. Pero hay otras configuraciones complejas de Active Directory que son posibles, siempre que dicha configuración lleve a habilitar Azure AD DS para el acceso de HDInsight.

### <a name="azure-active-directory-domain-services"></a>Azure Active Directory Domain Services
[Azure AD DS](../../active-directory-domain-services/active-directory-ds-overview.md) proporciona un dominio administrado que es totalmente compatible con Windows Server Active Directory. Microsoft se encarga de administrar y supervisar el dominio, y de aplicarle los parches, en una configuración de alta disponibilidad (HA). Puede implementar el clúster sin preocuparse por mantener los controladores de dominio. 

Los usuarios, los grupos y las contraseñas se sincronizan desde Azure AD. La sincronización unidireccional desde la instancia de Azure AD a Azure AD DS permite que los usuarios inicien sesión en el clúster con las mismas credenciales corporativas. 

Para más información, consulte [Configuración de clústeres de HDInsight con Enterprise Security Package mediante Azure Active Directory Domain Services](./apache-domain-joined-configure-using-azure-adds.md).

### <a name="on-premises-active-directory-or-active-directory-on-iaas-vms"></a>Active Directory local o Active Directory en máquinas virtuales de IaaS

Si tiene una instancia de Active Directory local o configuraciones más complejas de Active Directory para el dominio, puede sincronizar esas identidades con Azure AD mediante Azure AD Connect. Luego puede habilitar Azure AD DS en ese inquilino de Active Directory. 

Como Kerberos se basa en valores hash de contraseña, debe [habilitar la sincronización de hash de contraseñas en Azure AD DS](../../active-directory-domain-services/active-directory-ds-getting-started-password-sync.md). 

Si está usando la federación con los Servicios de federación de Active Directory (AD FS), debe habilitar la sincronización del hash de contraseña. (Para ver una configuración recomendada, consulte [este vídeo](https://youtu.be/qQruArbu2Ew).) La sincronización del hash contraseña le permite llevar a cabo la recuperación ante desastres en caso de que su infraestructura de AD FS falle; asimismo, también le ayudará a proporcionar protección para las credenciales filtradas. Para más información, consulte [Implementación de la sincronización de hash de contraseñas con la sincronización de Azure AD Connect](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md). 

El uso de una instancia de Active Directory local o de Active Directory solo en máquinas virtuales de IaaS, sin Azure AD ni Azure AD DS, no es una configuración compatible con los clústeres de HDInsight con ESP.

Si se está usando la federación y los hash de contraseña se sincronizan correctamente, pero recibe errores de autenticación, compruebe si está habilitada la autenticación de contraseña en la nube para la entidad de servicio de PowerShell. De lo contrario, debe establecer una [directiva para la Detección del dominio principal (HRD)](../../active-directory/manage-apps/configure-authentication-for-federated-users-portal.md) en el inquilino de Azure AD. Para comprobar y establecer la directiva de HRD:

1. Instale el módulo de PowerShell de Azure AD.

   ```
   Install-Module AzureADPreview
   ```

2. Escriba `Connect-AzureAD` usando las credenciales de un administrador global (administrador de inquilinos).

3. Compruebe si ya se ha creado la entidad de servicio "Microsoft Azure PowerShell".

   ```
   $powershellSPN = Get-AzureADServicePrincipal -SearchString "Microsoft Azure Powershell"
   ```

4. Si no existe (es decir, si `($powershellSPN -eq $null)`), cree la entidad de servicio.

   ```
   $powershellSPN = New-AzureADServicePrincipal -AppId 1950a258-227b-4e31-a9cf-717495945fc2
   ```

5. Cree y asocie la directiva a la entidad de servicio.

   ```
   $policy = New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AllowCloudPasswordValidation`":true}}") -DisplayName EnableDirectAuth -Type HomeRealmDiscoveryPolicy

   Add-AzureADServicePrincipalPolicy -Id $powershellSPN.ObjectId -refObjectID $policy.ID
   ```

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de clústeres de HDInsight con Enterprise Security Package](apache-domain-joined-configure-using-azure-adds.md)
* [Configuración de directivas de Apache Hive en clústeres de HDInsight con Enterprise Security Package](apache-domain-joined-run-hive.md)
* [Administración de clústeres de HDInsight con Enterprise Security Package](apache-domain-joined-manage.md) 
