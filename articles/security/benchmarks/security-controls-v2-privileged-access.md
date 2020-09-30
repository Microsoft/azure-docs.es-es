---
title: 'Azure Security Benchmark V2: acceso con privilegios'
description: 'Azure Security Benchmark V2: acceso con privilegios'
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/13/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 8727cbd07fad1960f4bdb33742729b6bda3a369e
ms.sourcegitcommit: 94c750edd4d755d6ecee50ac977328098a277479
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90059110"
---
# <a name="security-control-privileged-access"></a>Control de seguridad: Acceso con privilegios

El acceso con privilegios abarca los controles para proteger el acceso con privilegios a los recursos y al inquilino de Azure. Incluye una serie de controles para proteger el modelo administrativo, las cuentas administrativas y las estaciones de trabajo de acceso con privilegios frente a riesgos deliberados e involuntarios.

## <a name="pa-1-protect-and-limit-the-global-administrators"></a>PA-1: Protección y limitación de los administradores globales

| Identificador de Azure | Identificadores de CIS Controls v7.1 | Identificadores de NIST SP800-53 r4 |
|--|--|--|--|
| PA-1 | 4.3, 4.8 | AC-2 |

Los usuarios que tienen asignado el rol Administrador global pueden leer y modificar cada configuración administrativa de la organización de Azure AD. Limite el número de cuentas de administrador global de Azure a no más de dos en cada suscripción. Los roles integrados más críticos de Azure AD son administrador global y administrador de roles con privilegios, ya que los usuarios asignados a estos dos roles pueden delegar roles de administrador:
- Administrador global/administrador de empresa: los usuarios con este rol tienen acceso a todas las características administrativas de Azure AD, así como a los servicios que utilizan identidades de Azure AD.

- Administrador de roles con privilegios: los usuarios con este rol pueden administrar asignaciones de roles en Azure AD, y dentro de Azure AD Privileged Identity Management (PIM). Además, este rol permite administrar todos los aspectos de PIM y de las unidades administrativas.

Nota: Si usa roles personalizados con determinados permisos con privilegios asignados, es posible que tenga otros roles críticos que deban administrarse. Además, puede que también quiera aplicar controles similares a la cuenta de administrador de recursos empresariales críticos.  

Puede habilitar el acceso con privilegios Just-in-Time (JIT) a los recursos de Azure y Azure AD mediante Azure AD Privileged Identity Management (PIM). JIT concede permisos temporales para realizar tareas con privilegios solo cuando los usuarios lo necesitan. PIM también puede generar alertas de seguridad cuando hay actividades sospechosas o no seguras en la organización de Azure AD.

Nota: Algunos servicios de Azure, como Azure SQL, admiten la autenticación de usuario local además de la autenticación de Azure AD. Este tipo de autenticación no se administra mediante Azure AD. Estos usuarios deberán administrarse por separado.

- [Permisos de rol administrativo en Azure AD](../../active-directory/users-groups-roles/directory-assign-admin-roles.md)

- [Roles personalizados en los recursos de Azure](../../role-based-access-control/custom-roles.md)

- [Uso de alertas de seguridad de Azure Privileged Identity Management](../../active-directory/privileged-identity-management/pim-how-to-configure-security-alerts.md)

- [Obtención de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [Uso de Azure Security Center para supervisar la identidad y el acceso](../../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Partes interesadas de seguridad del cliente**:

- [Identidad y claves](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Seguridad de las aplicaciones y DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="pa-2-review-and-reconcile-user-access-regularly"></a>PA-2: Revisión y conciliación de manera periódica del acceso de los usuarios

| Identificador de Azure | Identificadores de CIS Controls v7.1 | Identificadores de NIST SP800-53 r4 |
|--|--|--|--|
| PA-2 | 4.1, 16.9, 16.10 | AC-2 |

Revise las cuentas de usuario y los derechos de acceso con regularidad para asegurarse de que sean válidos. 

Use las revisiones de identidad y acceso de Azure AD para administrar las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. Los informes de Azure AD pueden proporcionar registros para ayudar a detectar cuentas obsoletas. También puede usar Azure AD Privileged Identity Management para crear un flujo de trabajo de informes de revisión de acceso para facilitar el proceso de revisión.

En el caso de usuarios administrativos en el nivel de carga de trabajo y servicio de Azure, se debe realizar una revisión más frecuente de los usuarios y el acceso. También puede usar las alertas de seguridad de Azure Privileged Identity Management para detectar cuándo se crean las cuentas de administrador y para buscar cuentas de administrador que hayan quedado obsoletas o que no estén configuradas correctamente. 

Nota: Algunos servicios de Azure, como Azure SQL, admiten usuarios locales que no se administran mediante Azure AD. Estos usuarios deberán administrarse por separado.

- [Obtención de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

- [Uso de Azure Security Center para supervisar la identidad y el acceso](../../security-center/security-center-identity-access.md)

- [Uso de alertas de seguridad de Azure Privileged Identity Management](../../active-directory/privileged-identity-management/pim-how-to-configure-security-alerts.md)

- [Alineación de las responsabilidades administrativas entre los equipos](/azure/cloud-adoption-framework/organize/raci-alignment) 

- [Descripción de los informes de Azure AD](/azure/active-directory/reports-monitoring/)

- [Procedimiento para usar las revisiones de acceso e identidades de Azure AD](../../active-directory/governance/access-reviews-overview.md)

- [Privileged Identity Management: revisión del acceso a los roles de Azure AD](../../active-directory/privileged-identity-management/pim-how-to-start-security-review.md)

- [Azure Security Center: supervisión de la identidad y el acceso](../../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Partes interesadas de seguridad del cliente**:

- [Identidad y claves](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Seguridad de las aplicaciones y DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Administración del cumplimiento de la seguridad](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="pa-3-set-up-an-emergency-access-account-in-azure-ad"></a>PA-3: Configuración de cuentas de acceso de emergencia en Azure AD

| Identificador de Azure | Identificadores de CIS Controls v7.1 | Identificadores de NIST SP800-53 r4 |
|--|--|--|--|
| PA-3 | 12.3 | AC-2 |

Para evitar que se le bloquee por accidente el acceso a la organización de Azure AD, configure una cuenta de acceso de emergencia para cuando no se puedan usar cuentas administrativas normales. Las cuentas de acceso de emergencia tienen normalmente privilegios elevados y no se asignan a usuarios específicos. Las cuentas de acceso de emergencia se limitan a situaciones "excepcionales" o de emergencia en las que no se pueden usar las cuentas administrativas normales.

Debe asegurarse de que las credenciales (como contraseña, certificado o tarjeta inteligente) de las cuentas de acceso de emergencia estén protegidas y solo las conozcan aquellas personas que estén autorizadas a usarlas solo en caso de emergencia.

- [Administración de cuentas de acceso de emergencia en Azure AD](../../active-directory/users-groups-roles/directory-emergency-access.md)

**Responsabilidad**: Customer

**Partes interesadas de seguridad del cliente**:

- [Identidad y claves](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Seguridad de las aplicaciones y DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Administración del cumplimiento de la seguridad](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Centro de operaciones de seguridad (SOC)](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

## <a name="pa-4-automate-azure-identity-and-access-request-workflow"></a>PA-4: Automatización del flujo de trabajo de solicitud de identidad y acceso de Azure

| Identificador de Azure | Identificadores de CIS Controls v7.1 | Identificadores de NIST SP800-53 r4 |
|--|--|--|--|
| PA-4 | N/D | AC-2, AC-5, PM-10 |

Use las características de administración de derechos de Azure AD para automatizar los flujos de trabajo de solicitud de acceso de Azure, como las asignaciones, las revisiones y la expiración del acceso. También se admite la aprobación en dos o varias fases.  

- [¿Qué es la administración de derechos de Azure AD?](../../active-directory/governance/entitlement-management-overview.md)

- [Configuración del proceso de solicitud y aprobación de acceso](../../active-directory/governance/entitlement-management-access-package-request-policy.md)

**Responsabilidad**: Customer

**Partes interesadas de seguridad del cliente**:

- [Identidad y claves](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Seguridad de las aplicaciones y DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Administración del cumplimiento de la seguridad](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="pa-5-use-highly-secured-machines-for-administrative-tasks"></a>PA-5: Uso de máquinas de alta seguridad para tareas administrativas

| Identificador de Azure | Identificadores de CIS Controls v7.1 | Identificadores de NIST SP800-53 r4 |
|--|--|--|--|
| PA-5 | 4.6, 11.6, 12.12 | AC-2, SC-7 |

Las estaciones de trabajo seguras y aisladas son de una importancia vital para la seguridad de los roles con acceso a información confidencial como administradores, desarrolladores y operadores de servicios críticos. Use estaciones de trabajo de usuario de alta seguridad o Azure Bastion para las tareas administrativas:
- Use Azure Active Directory, Protección contra amenazas avanzada de Microsoft Defender (ATP) o Microsoft Intune para implementar una estación de trabajo de usuario segura y administrada para las tareas administrativas. Las estaciones de trabajo protegidas se pueden administrar de forma centralizada para aplicar una configuración segura, como autenticación sólida, líneas de base de software y hardware y acceso lógico y de red restringido. 

- Use la característica Azure Bastion para tener una ruta de acceso segura a las máquinas virtuales mediante RDP o SSH. Azure Bastion es un servicio PaaS totalmente administrado que se puede aprovisionar por red virtual sin necesidad de crear otra red virtual. 

- [Descripción de las estaciones de trabajo seguras administradas por Azure](../../active-directory/devices/concept-azure-managed-workstation.md)

- [Implementación de una estación de trabajo segura administrada por Azure](../../active-directory/devices/howto-azure-managed-workstation.md)

- [Uso del host de Azure Bastion](../../bastion/bastion-create-host-portal.md)

**Responsabilidad**: Customer

**Partes interesadas de seguridad del cliente**:

- [Seguridad de las aplicaciones y DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Centro de operaciones de seguridad (SOC)](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

- [Identidad y claves](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

## <a name="pa-6-assign-privileges-to-resources-using-azure-rbac"></a>PA-6: Asignación de privilegios a recursos mediante Azure RBAC

| Identificador de Azure | Identificadores de CIS Controls v7.1 | Identificadores de NIST SP800-53 r4 |
|--|--|--|--|
| PA-6 | 14.6 | AC-2, AC-3 |

El control de acceso basado en rol de Azure (RBAC) permite administrar los privilegios de acceso a los recursos de Azure mediante asignaciones de roles. Puede asignar estos roles a usuarios, grupos de entidades de servicio e identidades administradas. Hay roles integrados predefinidos para determinados recursos, y estos roles se pueden inventariar o consultar mediante herramientas como la CLI de Azure, Azure PowerShell o el Azure Portal. 

- [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../../role-based-access-control/overview.md)

- [Configuración de RBAC en Azure](../../role-based-access-control/role-assignments-portal.md)

**Responsabilidad**: Customer

**Partes interesadas de seguridad del cliente**:

- [Seguridad de las aplicaciones y DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Administración del cumplimiento de la seguridad](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Administración de la posición](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Identidad y claves](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

## <a name="pa-7-choose-approval-process-for-microsoft-support"></a>PA-7: Selección del proceso de aprobación para el soporte técnico de Microsoft

| Identificador de Azure | Identificadores de CIS Controls v7.1 | Identificadores de NIST SP800-53 r4 |
|--|--|--|--|
| PA-7 | 16 | AC-2, AC-3, AC-4 |

En escenarios de soporte técnico en los que Microsoft necesita acceder a los datos de los clientes, la Caja de seguridad del cliente proporciona una funcionalidad para que pueda revisar y aprobar o rechazar explícitamente las solicitudes de acceso a tales datos.

- [Descripción de la Caja de seguridad del cliente](../fundamentals/customer-lockbox-overview.md)

**Responsabilidad**: Customer

**Partes interesadas de seguridad del cliente**:

- [Seguridad de las aplicaciones y DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Administración del cumplimiento de la seguridad](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Identidad y claves](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

