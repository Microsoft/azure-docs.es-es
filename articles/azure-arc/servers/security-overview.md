---
title: Introducción a la seguridad
description: Información de seguridad sobre los servidores habilitados para Azure Arc (versión preliminar).
ms.topic: conceptual
ms.date: 08/31/2020
ms.openlocfilehash: 17641fab9933d9d6a60c2b21912f755acc01a6dd
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89447662"
---
# <a name="azure-arc-for-servers-preview-security-overview"></a>Introducción a la seguridad de Azure Arc para servidores (versión preliminar)

En este artículo se describen la configuración y consideraciones de seguridad que debe evaluar antes de implementar los servidores habilitados para Azure Arc en su empresa.

## <a name="identity-and-access-control"></a>Control de identidades y acceso

Cada servidor habilitado para Azure Arc tiene una identidad administrada como parte de un grupo de recursos dentro de una suscripción a Azure que representa al servidor que se ejecuta en el entorno local o en otro entorno de nube. El acceso a este recurso se controla mediante el [control de acceso basado en roles de Azure](../../role-based-access-control/overview.md) estándar. En la página [**Access Control (IAM)** ](../../role-based-access-control/role-assignments-portal.md#access-control-iam) de Azure Portal, puede comprobar quién tiene acceso a su servidor habilitado para Azure Arc.

:::image type="content" source="./media/security-overview/access-control-page.png" alt-text="Control de acceso del servidor habilitado para Azure Arc" border="false" lightbox="./media/security-overview/access-control-page.png":::

Los usuarios y aplicaciones a los que se ha concedido el acceso de rol [Colaborador](../../role-based-access-control/built-in-roles.md#contributor) o Administrador en el recurso pueden realizar cambios en el recurso, incluida la implementación o eliminación de [extensiones](manage-vm-extensions.md) en la máquina. Las extensiones pueden incluir scripts arbitrarios que se ejecutan en un contexto con privilegios, por lo que se aconseja que todos los colaboradores en el recurso de Azure sean un administrador indirecto del servidor que no es de Azure.

El rol **Incorporación de Azure Connected Machine** está disponible para la incorporación a escala y solo puede leer o crear nuevos servidores habilitados para Arc en Azure. No se puede usar para eliminar servidores ya registrados, ni para administrar extensiones. Como procedimiento recomendado, se recomienda asignar este rol a la entidad de servicio de Azure Active Directory (Azure AD) que se usa para incorporar máquinas a escala.

Cuando los usuarios son miembros del rol **Administrador de recursos de Azure Connected Machine** pueden leer, modificar, volver a incorporar y eliminar una máquina. Este rol está diseñado para admitir la administración de servidores habilitados para Arc, pero no otros recursos del grupo de recursos o la suscripción.

## <a name="agent-security-and-permissions"></a>Seguridad y permisos del agente

Para administrar el agente de Azure Connected Machine (azcmagent) en Windows, la cuenta de usuario debe ser miembro del grupo local de administradores; en el caso de Linux, debe tener permisos de acceso raíz.

El agente de Azure Connected Machine se compone de tres servicios que se ejecutan en su equipo.

* El servicio Hybrid Instance Metadata Service (himds) es responsable de toda la funcionalidad básica de Arc. Esto incluye el envío de latidos a Azure, la exposición del servicio de metadatos de la instancia local para que otras aplicaciones obtengan información sobre el identificador de recurso de Azure de la máquina, y la recuperación de tokens de Azure AD para autenticarse en otros servicios de Azure. Este servicio se ejecuta como una cuenta de servicio virtual sin privilegios en Windows, y como el usuario **himds** en Linux.

* El servicio de configuración de invitados (GCService) se encarga de evaluar Azure Policy en el equipo.

* El servicio de extensión de configuración de invitados (ExtensionService) se encarga de la instalación, actualización y eliminación de extensiones (agentes, scripts u otro software) en el equipo.

Los servicios de la extensión y configuración de invitados se ejecutan como sistema local en Windows y como raíz en Linux.

## <a name="using-a-managed-identity-with-arc-enabled-servers"></a>Uso de una identidad administrada con servidores habilitados para Arc

De manera predeterminada, la identidad asignada por el sistema de Azure Active Directory que usa Arc solo se puede emplear para actualizar el estado del servidor habilitado para Arc en Azure. Por ejemplo, el *último* estado de latido. También puede asignar roles adicionales a la identidad si una aplicación en el servidor usa la identidad asignada por el sistema para acceder a otros servicios de Azure.

Mientras que todas las aplicaciones que se ejecutan en el equipo pueden acceder a Hybrid Instance Metadata Service, solo las aplicaciones autorizadas pueden solicitar un token de Azure AD para la identidad asignada por el sistema. En el primer intento de acceder al URI del token, el servicio generará un blob criptográfico generado de forma aleatoria en una ubicación del sistema de archivos que solo puedan leer los autores de llamada de confianza. A continuación, el autor de la llamada debe leer el archivo (al demostrar que tiene el permiso adecuado) y reintentar la solicitud con el contenido del archivo en el encabezado de autorización para recuperar correctamente un token de Azure AD.

* En Windows, el autor de la llamada debe ser miembro del grupo local **Administradores** o del grupo **Aplicaciones de la extensión de agente híbrido** para leer el blob.

* En Linux, el autor de la llamada debe ser miembro del grupo **himds** para leer el blob.

## <a name="using-disk-encryption"></a>Uso del cifrado de disco

El agente de Azure Connected Machine usa la autenticación de clave pública para comunicarse con el servicio de Azure. Después de incorporar un servidor a Azure Arc, se guarda una clave privada en el disco y se usa siempre que el agente se comunica con Azure. Si se descarta, la clave privada se puede usar en otro servidor para comunicarse con el servicio y actuar como si fuera el servidor original. Esto incluye acceder a la identidad asignada por el sistema y a todos los recursos a los que tiene acceso la identidad. El archivo de clave privada está protegido para permitir que solo la cuenta **himds** acceda para leerlo. Para evitar los ataques sin conexión, recomendamos el uso del cifrado de disco completo (por ejemplo, BitLocker, dm-crypt, etc.) en el volumen del sistema operativo del servidor.

## <a name="next-steps"></a>Pasos siguientes

Antes de evaluar o habilitar los servidores habilitados para Arc (versión preliminar) en varias máquinas híbridas, consulte el artículo [Introducción al agente de Connected Machine ](agent-overview.md) para conocer los requisitos, los detalles técnicos sobre el agente y los métodos de implementación.
