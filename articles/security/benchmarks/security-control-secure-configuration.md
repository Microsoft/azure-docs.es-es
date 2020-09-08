---
title: 'Control de seguridad de Azure: configuración segura'
description: 'Control de seguridad de Azure: configuración segura'
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 347a63cc77c565d800328c19d1d543c2c9efafc0
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400099"
---
# <a name="security-control-secure-configuration"></a>Control de seguridad: Configuración segura

Establezca, implemente y administre activamente (realice un seguimiento, informe y corrija) la configuración de seguridad de los recursos de Azure con el fin de evitar que los atacantes aprovechen los servicios y las configuraciones vulnerables.

## <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.1 | 5,1 | Customer |

use alias de Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar la configuración de los recursos de Azure. También puede usar definiciones de Azure Policy integradas.

Además, Azure Resource Manager tiene la capacidad de exportar la plantilla en notación de objetos JavaScript (JSON), que se debe revisar para asegurarse de que las configuraciones cumplan o superen los requisitos de seguridad de la organización.

También puede usar las recomendaciones de Azure Security Center como línea de base de configuración segura para los recursos de Azure.

- [Visualización de los alias de Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Tutorial: Creación y administración de directivas para aplicar el cumplimiento](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Exportación de uno y varios recursos a una plantilla en Azure Portal](https://docs.microsoft.com/azure/azure-resource-manager/templates/export-template-portal)

- [Guía de referencia sobre las recomendaciones de seguridad](https://docs.microsoft.com/azure/security-center/recommendations-reference)

## <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.2 | 5,1 | Customer |

Use las recomendaciones de Azure Security Center para mantener las configuraciones de seguridad de todos los recursos de proceso.  Además, puede usar imágenes de sistema operativo personalizado o la configuración de estado de Azure Automation para establecer la configuración de seguridad del sistema operativo que requiere su organización.

- [Cómo supervisar las recomendaciones de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-recommendations)

- [Guía de referencia sobre las recomendaciones de seguridad](https://docs.microsoft.com/azure/security-center/recommendations-reference)

- [Introducción a Azure Automation State Configuration](https://docs.microsoft.com/azure/automation/automation-dsc-overview)

- [Carga de un VHD y su uso para crear máquinas virtuales Windows nuevas en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/upload-generalized-managed)

- [Creación de una VM Linux desde un disco personalizado con la CLI de Azure](https://docs.microsoft.com/azure/virtual-machines/linux/upload-vhd)

## <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.3 | 5.2 | Customer |

Utilice las directivas [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura en los recursos de Azure.  Además, puede usar plantillas de Azure Resource Manager para mantener la configuración de seguridad de los recursos de Azure que requiere su organización. 

- [Descripción de los efectos de Azure Policy](https://docs.microsoft.com/azure/governance/policy/concepts/effects)

- [Creación y administración de directivas para aplicar el cumplimiento](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Información general sobre plantillas de Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/templates/overview)

## <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.4 | 5.2 | Compartido |

Siga las recomendaciones de Azure Security Center sobre cómo realizar evaluaciones de vulnerabilidades en los recursos de proceso de Azure.  Además, puede usar plantillas de Azure Resource Manager, imágenes de sistema operativo personalizado o la configuración de estado de Azure Automation para mantener la configuración de seguridad del sistema operativo que requiere su organización.   Las plantillas de máquina virtual de Microsoft combinadas con Desired State Configuration de Azure Automation pueden ayudar a cumplir y mantener los requisitos de seguridad. 

Además, tenga en cuenta que las imágenes de máquinas virtuales de Azure Marketplace que publica Microsoft, las administra y mantiene Microsoft mismo. 

- [Implementación de las recomendaciones de evaluación de vulnerabilidades de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-vulnerability-assessment-recommendations)

- [Creación de una máquina virtual de Azure desde una plantilla de Azure Resource Manager](https://docs.microsoft.com/azure/virtual-machines/windows/ps-template)

- [Introducción a State Configuration de Azure Automation](https://docs.microsoft.com/azure/automation/automation-dsc-overview)

- [Creación de una máquina virtual de Windows en Azure Portal](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal)

- [Información sobre cómo descargar la plantilla de máquina virtual](https://docs.microsoft.com/azure/virtual-machines/windows/download-template)

- [Script de ejemplo para cargar un disco duro virtual en Azure y crear una máquina virtual nueva](https://docs.microsoft.com/azure/virtual-machines/scripts/virtual-machines-windows-powershell-upload-generalized-script)

## <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.5 | 5.3 | Customer |

Use Azure DevOps para almacenar y administrar de forma segura el código, como directivas de Azure personalizadas, plantillas de Azure Resource Manager y scripts de Desired State Configuration. Para acceder a los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps, o Active Directory si se integran con TFS.

- [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Acerca de los permisos y los grupos en Azure DevOps](https://docs.microsoft.com/azure/devops/organizations/security/about-permissions)

## <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.6 | 5.3 | Customer |

Si usa imágenes personalizadas, use el control de acceso basado en rol de Azure (Azure RBAC) para asegurarse de que solo los usuarios autorizados pueden acceder a las imágenes. Uso de una galería de imágenes compartidas para compartir imágenes con diferentes usuarios, entidades de servicio o grupos de AD dentro de su organización.  En el caso de las imágenes de contenedor, almacénelas en Azure Container Registry y aproveche Azure RBAC para asegurarse de que solo los usuarios autorizados puedan acceder a las imágenes.  

- [Descripción de Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/rbac-and-directory-admin-roles)

- [Descripción de Azure RBAC para Container Registry](https://docs.microsoft.com/azure/container-registry/container-registry-roles)

- [Configuración de Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/quickstart-assign-role-user-portal)

- [Introducción a la galería de imágenes compartidas](https://docs.microsoft.com/azure/virtual-machines/windows/shared-image-galleries)

## <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7,7 | 5.4 | Customer |

defina e implemente configuraciones de seguridad estándar para los recursos de Azure con Azure Policy. Use alias de Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de los recursos de Azure. También puede usar definiciones de directivas integradas relacionadas con recursos concretos.  Además, puede usar Azure Automation para implementar los cambios de configuración.

- [Configuración y administración de Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Uso de alias](https://docs.microsoft.com/azure/governance/policy/concepts/definition-structure#aliases)

## <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8: Implementación de herramientas de administración de la configuración para sistemas operativos

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.8 | 5.4 | Customer |

Azure Automation State Configuration es un servicio de administración de configuración para los nodos de Desired State Configuration (DSC) de cualquier centro de datos local o en la nube. Puede incorporar fácilmente máquinas, asignarles configuraciones declarativas y ver informes que muestran el cumplimiento por parte de cada máquina del estado deseado especificado. 

- [Incorporación de máquinas para su administración mediante Azure Automation State Configuration](https://docs.microsoft.com/azure/automation/automation-dsc-onboarding)

## <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.9 | 5.5 | Customer |

Use Azure Security Center para realizar análisis de base de referencia para los recursos de Azure.  Además, use Azure Policy para alertar y auditar las configuraciones de recursos de Azure.

- [Corrección de recomendaciones en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-remediate-recommendations)

## <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.10 | 5.5 | Customer |

Use Azure Security Center para realizar exámenes de base de referencia para la configuración del sistema operativo y de Docker para los contenedores.

- [Descripción de las recomendaciones sobre contenedores de Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-container-recommendations)

## <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.11 | 13.1 | Customer |

Use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos para las aplicaciones en la nube.

- [Integración con identidades administradas de Azure](https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity)

- [Creación de un almacén de claves](https://docs.microsoft.com/azure/key-vault/quick-create-portal)

- [Autenticación en Key Vault](../../key-vault/general/authentication.md)

- [Asignación de una directiva de acceso de Key Vault](../../key-vault/general/assign-access-policy-portal.md)

## <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.12 | 4,1 | Customer |

Use las identidades administradas para proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure AD. Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código.

- [Configuración de las identidades administradas](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm)

## <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

| Id. de Azure | Id. de CIS | Responsabilidad |
|--|--|--|
| 7.13 | 18.1, 18.7 | Customer |

Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault. 

- [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)


## <a name="next-steps"></a>Pasos siguientes

- Consulte el siguiente control de seguridad:  [Defensa contra malware](security-control-malware-defense.md)
