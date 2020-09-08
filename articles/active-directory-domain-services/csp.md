---
title: Azure AD Domain Services para proveedores de soluciones en la nube | Microsoft Docs
description: Aprenda a habilitar y administrar dominios administrados de Azure Active Directory Domain Services para proveedores de soluciones en la nube de Azure.
services: active-directory-ds
author: iainfoulds
ms.assetid: 56ccb219-11b2-4e43-9f07-5a76e3cd8da8
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: iainfou
ms.openlocfilehash: 524e3682e39b6d9153beeef781deeb20e6ea8750
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88749648"
---
# <a name="azure-active-directory-domain-services-deployment-and-management-for-azure-cloud-solution-providers"></a>Implementación y administración de Azure Active Directory Domain Services para proveedores de soluciones en la nube de Azure

Proveedores de soluciones en la nube (CSP) de Azure es un programa para asociados de Microsoft y proporciona un canal de licencias para varios servicios en la nube de Microsoft. Azure CSP permite a los asociados administrar ventas, controlar la relación de facturación, proporcionar soporte técnico y asistencia para la facturación y ser el único punto de contacto de los clientes. Además, Azure CSP proporciona un conjunto completo de herramientas, incluido un portal de autoservicio y API complementarias. Estas herramientas permiten a los asociados de CSP aprovisionar y administrar recursos de Azure fácilmente, así como proporcionar una facturación para sus clientes y suscripciones.

El [portal del Centro de partners](/partner-center/azure-plan-lp) es el punto de entrada para todos los asociados de CSP de Azure y proporciona funciones de administración de clientes enriquecidas, procesamiento automatizado y mucho más. Los asociados de Azure CSP pueden utilizar las funciones de Centro de partners mediante una UI basada en web o mediante PowerShell y diversas llamadas API.

El siguiente diagrama ilustra cómo funciona el modelo de CSP en un nivel superior. Aquí, Contoso tiene un inquilino de Azure Active Directory (Azure AD). Tienen una asociación con un CSP, que implementa y administra los recursos de su suscripción a Azure CSP. Contoso también puede tener suscripciones regulares (directas) a Azure, que se facturan directamente a Contoso.

![Información general sobre el modelo de CSP](./media/csp/csp_model_overview.png)

El inquilino del asociado de CSP tiene tres grupos de agentes especiales: agentes de *administración*, agentes del *departamento de soporte técnic*o y agentes de *ventas*.

El grupo de agentes de *administración* se asigna al rol de administrador de inquilinos en el inquilino de Azure AD de Contoso. Como resultado, un usuario que pertenezca al grupo de agentes de administración del asociado de CSP tiene privilegios de administrador de inquilinos en el inquilino de Azure AD de Contoso.

Cuando el asociado de CSP aprovisiona una suscripción a Azure CSP para Contoso, su grupo de agentes de administración se asigna al rol de propietario de dicha suscripción. Como resultado, los agentes de administración del asociado de CSP tienen los privilegios necesarios para aprovisionar recursos de Azure, como máquinas virtuales, redes virtuales y Azure AD Domain Services en nombre de Contoso.

Para más información, consulte [Azure CSP overview](/partner-center/azure-plan-lp) (Introducción a Azure CSP)

## <a name="benefits-of-using-azure-ad-ds-in-an-azure-csp-subscription"></a>Ventajas de usar Azure AD DS en una suscripción a Azure CSP

Azure Active Directory Domain Services (Azure AD DS) proporciona servicios de dominio administrados como, por ejemplo, unión a un dominio, directiva de grupo, LDAP o autenticación Kerberos/NTLM, que son totalmente compatibles con Windows Server Active Directory Domain Services. A lo largo de décadas, muchas aplicaciones se han compilado para funcionar con AD con estas funcionalidades. Muchos proveedores de software independientes (ISV) han creado e implementado aplicaciones en las instalaciones de sus clientes. El soporte técnico de estas aplicaciones resulta difícil, ya que, a menudo, requiere acceso a los distintos entornos en que están implementadas las aplicaciones. Con las suscripciones a Azure CSP, tiene una alternativa más sencilla gracias al escalado y la flexibilidad de Azure.

Azure AD DS admite suscripciones a Azure CSP. Puede implementar la aplicación en una suscripción a Azure CSP enlazada al inquilino de Azure AD de su cliente. Como resultado, los empleados (personal de soporte técnico) pueden administrar y realizar el mantenimiento de las máquinas virtuales en las que se implementa la aplicación mediante credenciales corporativas de la organización.

También puede implementar un dominio administrado de Azure AD DS en el inquilino de Azure AD de su cliente. La aplicación está conectada entonces al dominio administrado del cliente. Las funciones de la aplicación que dependen de Kerberos/NTLM, LDAP o la [API System.DirectoryServices](/dotnet/api/system.directoryservices) funcionan sin problemas en el dominio de su cliente. Los clientes finales se beneficiarán de consumir la aplicación como servicio sin tener que preocuparse del mantenimiento de la infraestructura en que se implementó la aplicación.

Se le cobrará la facturación de todos los recursos de Azure que consuma en dicha suscripción, incluido Azure AD DS. Tiene control completo sobre la relación con el cliente en lo que respecta a ventas, facturación, soporte técnico, etc. Con la flexibilidad de la plataforma Azure CSP, un pequeño equipo de agentes de soporte técnico puede ofrecer servicio a muchos clientes que tengan instancias de la aplicación implementadas.

## <a name="csp-deployment-models-for-azure-ad-ds"></a>Modelos de implementación de CSP para Azure AD DS

Hay dos maneras de usar Azure AD DS con una suscripción a Azure CSP. Seleccione el adecuado en función de las consideraciones de seguridad y simplicidad de los clientes.

### <a name="direct-deployment-model"></a>Modelo de implementación directa

En este modelo de implementación, Azure AD DS se habilita en una red virtual que pertenece a la suscripción a Azure CSP. Los agentes de administración del asociado de CSP tienen los privilegios siguientes:

* Privilegios de *administrador global* en el inquilino de Azure AD del cliente.
* Privilegios de *propietario de la suscripción* en la suscripción a Azure CSP.

![Modelo de implementación directa](./media/csp/csp_direct_deployment_model.png)

En este modelo de implementación, los agentes de administración del proveedor de CSP pueden administrar identidades para el cliente. Estos agentes de administración pueden realizar tareas como aprovisionar nuevos usuarios o grupos o agregar aplicaciones en el inquilino de Azure AD del cliente.

Este modelo de implementación puede ser adecuado para organizaciones pequeñas que no tienen un administrador de identidades dedicado o prefieren que el asociado de CSP administre identidades en su nombre.

### <a name="peered-deployment-model"></a>Modelo de implementación emparejado

En este modelo de implementación, Azure AD DS se habilita en una red virtual que pertenece al cliente, es decir, una suscripción directa a Azure que paga el cliente. El asociado de CSP puede implementar aplicaciones en una red virtual que pertenece a la suscripción a CSP del cliente. A continuación, las redes virtuales se pueden conectar mediante el emparejamiento de redes virtuales de Azure.

Con esta implementación, las cargas de trabajo o aplicaciones que implementa el asociado de CSP en la suscripción a Azure CSP pueden conectar con el dominio administrado del cliente aprovisionado en la suscripción a Azure directa del cliente.

![Modelo de implementación emparejado](./media/csp/csp_peered_deployment_model.png)

Este modelo de implementación proporciona una separación de privilegios y permite a los agentes del departamento de soporte técnico del asociado de CSP administrar la suscripción a Azure, e implementar y administrar los recursos que contiene. Sin embargo, los agentes del departamento de soporte técnico del asociado de CSP no necesitan tener privilegios de administrador global en el directorio de Azure AD del cliente. Los administradores de identidades del cliente pueden seguir administrando identidades para su organización.

Este modelo de implementación puede ser adecuado para escenarios en que un ISV proporciona una versión hospedada de su aplicación local, que también se debe poder conectar a la instancia de Azure AD del cliente.

## <a name="administer-azure-ad-ds-in-csp-subscriptions"></a>Administración de Azure AD DS en suscripciones de CSP

Cuando se administra un dominio administrado en una suscripción a Azure CSP, se aplican las siguientes consideraciones importantes:

* **Los agentes de administrador de CSP pueden aprovisionar un dominio administrado mediante sus credenciales:** Azure AD DS admite suscripciones a Azure CSP. Los usuarios que pertenecen al grupo de agentes de administración del asociado de CSP pueden aprovisionar un nuevo dominio administrado.

* **Los CSP pueden crear con scripts nuevos dominios administrados para sus clientes mediante PowerShell:** Consulte [cómo puede habilitar Azure AD DS mediante PowerShell](powershell-create-instance.md) para ver detalles.

* **Los agentes de administrador de CSP no pueden realizar tareas de administración continuas en el dominio administrado con sus credenciales:** Los usuarios administradores de CSP no pueden llevar a cabo tareas de administración de rutina dentro del dominio administrado mediante el uso de sus credenciales. Estos usuarios son externos al inquilino de Azure AD del cliente y sus credenciales no están disponibles en el inquilino de Azure AD del cliente. Azure AD DS no tiene acceso a los hash de contraseña de Kerberos y NTLM para estos usuarios, por lo que los usuarios no se pueden autenticar en dominios administrados.

  > [!WARNING]
  > Debe crear una cuenta de usuario en el directorio del cliente para realizar tareas de administración en curso en el dominio administrado.
  >
  > No puede iniciar sesión en el dominio administrado usando credenciales de usuario administrador de CSP. Para ello, use las credenciales de una cuenta de usuario que pertenezca al inquilino de Azure AD. Necesita estas credenciales para tareas como unir máquinas virtuales al dominio administrado, administrar DNS o administrar directivas de grupo.

* **La cuenta de usuario creada para la administración continua se debe agregar al grupo *Administradores de controladores de dominio de AAD*:** El grupo *Administradores de controladores de dominio de AAD* tiene privilegios para realizar ciertas tareas de administración delegada en el dominio administrado. Estas tareas incluyen la configuración de DNS, la creación de unidades organizativas y la administración de directivas de grupos.
    
    Para que un asociado de CSP realice estas tareas en un dominio administrado, es necesario crear una cuenta de usuario en el inquilino de Azure AD del cliente. Las credenciales para esta cuenta se deben compartir con los agentes de administración del asociado de CSP. Además, esta cuenta de usuario debe agregarse al grupo *Administradores de controladores de dominio de AAD* para permitir que las tareas de configuración del dominio administrado se realicen mediante esta cuenta de usuario.

## <a name="next-steps"></a>Pasos siguientes

Para empezaer, [inscríbase en el programa CSP de Azure](/partner-center/enrolling-in-the-csp-program). Luego puede habilitar Azure AD Domain Services mediante [Azure Portal](tutorial-create-instance.md) o [Azure PowerShell](powershell-create-instance.md).