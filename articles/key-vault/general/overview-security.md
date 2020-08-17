---
title: Seguridad de Azure Key Vault | Microsoft Docs
description: Administración de los permisos de acceso para Azure Key Vault, claves y secretos. Trata sobre el modelo de autenticación y autorización de Key Vault y cómo proteger un almacén de claves.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 04/18/2019
ms.author: mbaldwin
Customer intent: As a key vault administrator, I want to learn the options available to secure my vaults
ms.openlocfilehash: 3170b51b8e580a625b2dbb4d83229c59bafa900e
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534811"
---
# <a name="azure-key-vault-security"></a>Seguridad de Azure Key Vault

Necesita proteger las claves de cifrado y los secretos, como certificados, cadenas de conexión y contraseñas en la nube para usar Azure Key Vault. Ya que almacena datos confidenciales y críticos para la empresa, debe tomar medidas para maximizar la seguridad de los almacenes y de los datos almacenados en ellos. En este artículo se tratan algunos de los conceptos que se deben considerar al diseñar la seguridad de Azure Key Vault.

## <a name="identity-and-access-management"></a>Administración de identidades y acceso

Cuando se crea un almacén de claves en una suscripción de Azure, se asocia automáticamente al inquilino de Azure AD de dicha suscripción. Cualquier persona que intente administrar o recuperar el contenido desde un almacén debe autenticarse con Azure AD.

- La autenticación establece la identidad del llamador.
- La autorización determina las operaciones que puede realizar quien envía la llamada. La autorización en Key Vault usa una combinación de [control de acceso basado en roles](../../role-based-access-control/overview.md) (RBAC) y directivas de acceso de Azure Key Vault.

### <a name="access-model-overview"></a>Introducción al modelo acceso

El acceso a los almacenes se realiza a través de dos interfaces o planos. Estos son el plano de administración y el plano de datos.

- El *plano de administración* es donde se administra Key Vault y es la interfaz utilizada para crear y eliminar almacenes. También puede leer las propiedades del almacén de claves y administrar las directivas de acceso.
- El *plano de datos* es donde se trabaja con los datos almacenados en un almacén de claves. Puede agregar, eliminar y modificar claves, secretos y certificados.

Para obtener acceso a un almacén de claves en cualquier plano, quienes realizan la llamada (usuarios o aplicaciones) deben estar autenticados y autorizados. Ambos planos usan Azure Active Directory (Azure AD) para la autenticación. Para la autorización, el plano de administración usa el control de acceso basado en rol (RBAC) y el plano de datos utiliza la directiva de acceso de Key Vault.

El modelo de un único mecanismo de autenticación para ambos planos tiene varias ventajas:

- Las organizaciones pueden controlar el acceso de forma centralizada a todos los almacenes de claves de su organización.
- Si un usuario abandona la organización, al instante pierde el acceso a todos los almacenes de claves de la organización.
- Las organizaciones pueden personalizar la autenticación mediante las opciones de Azure AD, como la habilitación de Multi-Factor Authentication para aumentar la seguridad.

### <a name="managing-administrative-access-to-key-vault"></a>Administración del acceso administrativo a Key Vault

Cuando crea un almacén de claves en un grupo de recursos, administra el acceso mediante Azure AD. Puede conceder a usuarios o grupos la capacidad de administrar los almacenes de claves en un grupo de recursos. Puede conceder acceso a un nivel de ámbito específico si asigna los roles de Azure apropiados. Para conceder acceso a un usuario para administrar almacén de claves, debe asignar un rol `key vault Contributor` predefinido al usuario en un ámbito específico. Los siguientes niveles de ámbitos se pueden asignar a un rol de Azure:

- **Suscripción**: Un rol de Azure asignado al nivel de suscripción se aplica a todos los recursos y grupos de recursos de esa suscripción.
- **Grupo de recursos**: Un rol de Azure asignado al nivel de grupo de recursos se aplica a todos los recursos de dicho grupo de recursos.
- **Recurso específico**: un rol de Azure asignado a un recurso concreto se aplica a dicho recurso. En este caso, el recurso es un almacén de claves específico.

Existen varios roles predefinidos. Si un rol predefinido no se ajusta a sus necesidades, puede definir uno propio. Para más información, vea [RBAC: roles integrados](../../role-based-access-control/built-in-roles.md).

> [!IMPORTANT]
> Si un usuario tiene permisos `Contributor` en un plano de administración de Key Vault, se puede conceder a sí mismo acceso al plano de datos estableciendo una directiva de acceso de Key Vault. Debe controlar estrechamente quién tiene el rol de acceso `Contributor` a los almacenes de claves. Asegúrese de que solo las personas autorizadas pueden acceder a los almacenes de claves, las claves, los secretos y los certificados y administrarlos.

<a id="data-plane-access-control"></a>
### <a name="controlling-access-to-key-vault-data"></a>Control del acceso a datos de Key Vault

Las directivas de acceso de Key Vault conceden permisos independientes a las claves, los secretos o los certificados. Puede conceder acceso a un usuario solo a las claves y no a los secretos. Los permisos para acceder a las claves, los secretos y los certificados se administran en el nivel del almacén.

> [!IMPORTANT]
> Las directivas de acceso de Key Vault no admiten permisos de nivel de objeto pormenorizados, como una clave, un secreto o un certificado específicos. Cuando se concede a un usuario permiso para crear y eliminar claves, este puede realizar dichas operaciones en todas las claves de dicho almacén de claves.

Para establecer directivas de acceso de un almacén de claves, use [Azure Portal](https://portal.azure.com/), la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest), [Azure PowerShell](/powershell/azure/) o las [API de REST de administración de Key Vault](/rest/api/keyvault/).

Puede restringir el acceso al plano de datos mediante el uso de [puntos de conexión de servicio de red virtual para Azure Key Vault](overview-vnet-service-endpoints.md). Puede configurar [firewall y reglas de red virtual](network-security.md) para una capa adicional de seguridad.

## <a name="network-access"></a>Acceso de red

Puede reducir la exposición de los almacenes si especifica las direcciones IP que tienen acceso a ellos. Los puntos de conexión de servicio de red virtual para Azure Key Vault permiten restringir el acceso a una red virtual especificada. También permiten restringir el acceso a una lista de intervalos de direcciones IPv4 (protocolo de Internet, versión 4). A todos los usuarios que se conecten a su almacén de claves desde fuera de esos orígenes se les negará el acceso.

Una vez que las reglas del firewall están en vigor, los usuarios solo pueden leer datos de Key Vault cuando las solicitudes se originan desde redes virtuales o rangos de direcciones IPv4 permitidos. Esto también se aplica al acceso de Key Vault desde Azure Portal. Aunque los usuarios pueden ir a un almacén de claves desde Azure Portal, es posible que no pueda enumerar las claves, los secretos o los certificados si su equipo cliente no está en la lista de dispositivos permitidos. Esto también afecta al selector de Key Vault de otros servicios de Azure. Los usuarios podrían ver la lista de almacenes de claves, pero no enumerar las claves si las reglas del firewall limitan su equipo cliente.

Para obtener más información sobre las direcciones de red de Azure Key Vault, revise [Puntos de conexión de servicio de red virtual para Azure Key Vault](overview-vnet-service-endpoints.md))

## <a name="monitoring"></a>Supervisión

Los registros de Key Vault guardan información acerca de las actividades realizadas en el almacén. Key Vault registra lo siguiente:

- Todas las solicitudes autenticadas de API REST, incluidas las solicitudes con error.
  - Las operaciones en el almacén de claves. Estas operaciones incluyen la creación, eliminación y definición de políticas de acceso, y la actualización de los atributos del almacén de claves, como las etiquetas.
  - Las operaciones en claves y secretos del almacén de claves, incluido lo siguiente:
    - Crear, modificar o eliminar estas claves o secretos.
    - Firmar, comprobar, cifrar, descifrar, encapsular y desencapsular claves, obtener secretos y elaborar listados de claves y secretos (y sus versiones).
- Solicitudes no autenticadas que dan como resultado una respuesta 401. Por ejemplo, las solicitudes que no tienen un token de portador, cuyo formato es incorrecto o está caducado o que tienen un token no válido.

Puede acceder a la información de registro 10 minutos después de la operación del almacén de claves. Es decisión suya administrar los registros en la cuenta de almacenamiento.

- Utilice los métodos de control de acceso de Azure estándar para proteger los registros mediante la restricción de quién puede tener acceso a ellos.
- Elimine los registros que ya no desee mantener en la cuenta de almacenamiento.

Para obtener recomendaciones sobre la administración segura de las cuentas de almacenamiento, revise la [guía de seguridad de Azure Storage](../../storage/blobs/security-recommendations.md)

## <a name="next-steps"></a>Pasos siguientes

- [Puntos de conexión de servicio de red virtual para Azure Key Vault](overview-vnet-service-endpoints.md)
- [RBAC: roles integrados](../../role-based-access-control/built-in-roles.md)
- [Puntos de conexión de servicio de red virtual para Azure Key Vault](overview-vnet-service-endpoints.md)
