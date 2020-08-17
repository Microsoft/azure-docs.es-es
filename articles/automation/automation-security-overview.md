---
title: Introducción a la autenticación de cuentas de Azure Automation
description: En este artículo se ofrece información general sobre la autenticación de cuentas de Azure Automation.
keywords: seguridad de automatización, automatización segura; autenticación de automatización
services: automation
ms.subservice: process-automation
ms.date: 04/23/2020
ms.topic: conceptual
ms.openlocfilehash: 61aec3936ffef488b989f54894568d206c759b12
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87014990"
---
# <a name="automation-account-authentication-overview"></a>Introducción a la autenticación de cuentas de Automation

Azure Automation le permite automatizar tareas en recursos de Azure, locales y de otros proveedores de servicios en la nube como Amazon Web Services (AWS). Puede usar runbooks para automatizar las tareas o bien, una instancia de Hybrid Runbook Worker si tiene que administrar tareas que no son de Azure. Cualquier entorno requiere permisos para acceder de forma segura a los recursos con los derechos mínimos necesarios en la suscripción de Azure.

En este artículo se tratan los distintos escenarios de autenticación que se admiten en Azure Automation y se indica cómo empezar a trabajar según los entornos que haya que administrar.

## <a name="automation-account"></a>Cuenta de Automation 

Cuando inicia Azure Automation por primera vez, debe crear al menos una cuenta de Automation. Las cuentas de Automation le permiten aislar sus recursos, runbooks, activos y configuraciones de Automation de los recursos de otras cuentas. Puede usar cuentas de Automation para separar los recursos en entornos lógicos independientes. Por ejemplo, puede usar una cuenta para desarrollo, otra para producción y otra para su entorno local. Una cuenta de Azure Automation es diferente de su cuenta Microsoft o de las cuentas creadas en su suscripción de Azure. Para obtener una introducción a la creación de una cuenta de Automation, consulte [Creación de una cuenta independiente de Azure Automation](automation-quickstart-create-account.md).

## <a name="automation-resources"></a>Recursos de Automation

Los recursos de Automation de cada cuenta de Automation están asociados a una sola región de Azure, pero la cuenta puede administrar todos los recursos de la suscripción de Azure. El principal motivo para crear cuentas de Automation en distintas regiones es si tiene directivas que requieren que los datos y recursos se aíslen en una región específica.

Todas las tareas que se crean en los recursos con Azure Resource Manager y los cmdlets de PowerShell en Azure Automation deben autenticarse en Azure mediante la autenticación basada en credenciales de la identidad organizativa de Azure Active Directory (Azure AD). 

## <a name="run-as-account"></a>cuenta de identificación

Las cuentas de ejecución de Azure Automation proporcionan autenticación para administrar los recursos de Azure con cmdlets de PowerShell. Al crear una cuenta de ejecución, se crea un usuario de la entidad de servicio en Azure AD y se le asigna el rol Colaborador en el nivel de suscripción. Con los runbooks que utilizan instancias de Hybrid Runbook Worker en máquinas virtuales de Azure, puede utilizar la [autenticación de runbook con identidades administradas](automation-hrw-run-runbooks.md#runbook-auth-managed-identities) en lugar de cuentas de ejecución para autenticarse en sus recursos de Azure.

## <a name="service-principal-for-run-as-account"></a>Entidad de servicio para la cuenta de ejecución

La entidad de servicio para una cuenta de ejecución no tiene los permisos para leer Azure AD de manera predeterminada. Si quiere agregar permisos para leer o administrar Azure AD, tiene que conceder los permisos a la entidad de servicio en **Permisos de API**. Para más información, consulte [Adición de permisos para acceder a las API web](../active-directory/develop/quickstart-configure-app-access-web-apis.md#add-permissions-to-access-web-apis).

## <a name="role-based-access-control"></a>Control de acceso basado en rol

El control de acceso basado en rol está disponible en Azure Resource Manager para conceder las acciones permitidas a una cuenta de usuario de Azure AD y a una cuenta de ejecución, y para autenticar la entidad de servicio. Para más información que le ayude a desarrollar su modelo de administración de permisos de Automation, consulte el [artículo Control de acceso basado en rol en Azure Automation](automation-role-based-access-control.md).  

## <a name="runbook-authentication-with-hybrid-runbook-worker"></a>Autenticación de Runbook con Hybrid Runbook Worker 

Los runbooks que se ejecutan en una instancia de Hybrid Runbook Worker en su centro de datos o en los servicios de computación de otros entornos de nube como AWS no pueden usar el mismo método que se usa normalmente para los runbooks que se autentican en recursos de Azure. Esto se debe a que esos recursos se ejecutan fuera de Azure y, por lo tanto, requieren sus propias credenciales de seguridad definidas en Automatización para la autenticación en los recursos a los que tienen acceso localmente. Para más información sobre la autenticación de runbooks con trabajos de runbook, consulte [Ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md). 

## <a name="next-steps"></a>Pasos siguientes

* Para crear una cuenta de Automation desde Azure Portal, consulte [Creación de una cuenta independiente de Azure Automation](automation-create-standalone-account.md).
* Si prefiere crear su cuenta mediante una plantilla, consulte [Creación de una cuenta de Automation con una plantilla de Azure Resource Manager](quickstart-create-automation-account-template.md).
* Para la autenticación con Amazon Web Services, consulte [Autenticación de runbooks de Azure Automation con Amazon Web Services](automation-config-aws-account.md).