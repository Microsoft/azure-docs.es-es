---
title: Habilitación de la replicación para puntos de conexión privados en Azure Site Recovery
description: En este artículo se describe cómo configurar la replicación de VM con puntos de conexión privados de una región de Azure a otra mediante Site Recovery.
author: mayurigupta13
ms.author: mayg
ms.service: site-recovery
ms.topic: article
ms.date: 07/14/2020
ms.custom: references_regions
ms.openlocfilehash: 16cde1cf43c6463cbbe640d9e0a80a9ea88f1f1f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095153"
---
# <a name="replicate-machines-with-private-endpoints"></a>Replicación de máquinas con puntos de conexión privados

Azure Site Recovery permite usar puntos de conexión privados de [Azure Private Link](../private-link/private-endpoint-overview.md) para replicar las máquinas desde una red virtual aislada. La compatibilidad con el acceso de puntos de conexión privados a un almacén de recuperación es compatible con las siguientes regiones:

- Azure Commercial: Centro y Sur de EE. UU., Oeste de EE. UU. 2, Este de EE. UU.
- Azure Government: US Gov Virginia, US Gov Arizona, US Gov Texas, US DoD (este), US DoD (centro)

En este artículo se proporcionan instrucciones para seguir los pasos a continuación:

- Crear un almacén de Recovery Services de Azure Backup para proteger sus máquinas.
- Habilitar una identidad administrada para el almacén y conceder los permisos necesarios para tener acceso a las cuentas de almacenamiento de los clientes con el fin de replicar el tráfico desde las ubicaciones de origen a las de destino. El acceso a la identidad administrada para el almacenamiento es necesario cuando se configura el acceso de Private Link al almacén.
- Realizar cambios de DNS necesarios en puntos de conexión privados.
- Crear y aprobar puntos de conexión privados para un almacén dentro de una red virtual.
- Crear puntos de conexión privados para las cuentas de almacenamiento. Puede seguir permitiendo el acceso público o con firewall para el almacenamiento según sea necesario. La creación de un punto de conexión privado para el acceso al almacenamiento no es obligatoria en Azure Site Recovery.
  
A continuación se muestra una arquitectura de referencia sobre cómo el flujo de trabajo de replicación cambia con los puntos de conexión privados.

:::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/architecture.png" alt-text="Arquitectura de referencia para Site Recovery con puntos de conexión privados.":::

## <a name="prerequisites-and-caveats"></a>Requisitos previos y advertencias

- Los puntos de conexión privados se pueden crear solo para almacenes de Recovery Services nuevos que no tienen elementos registrados. Por lo tanto, los puntos de conexión privados **deben crearse antes de agregar ningún elemento al almacén**. Revise la estructura de precios para los [puntos de conexión privados](https://azure.microsoft.com/pricing/details/private-link/).
- Cuando se crea un punto de conexión privado para un almacén, el almacén se bloquea y **no es posible acceder desde redes que no sean las redes que tienen puntos de conexión privados**.
- Actualmente, Azure Active Directory no admite puntos de conexión privados. Por lo tanto, las direcciones IP y los nombres de dominio completos necesarios para que Azure Active Directory funcione en una región deben tener permiso de acceso de salida desde la red protegida. También puede usar la etiqueta de grupo de seguridad de red "Azure Active Directory" y etiquetas de Azure Firewall para permitir el acceso a Azure Active Directory, según corresponda.
- **Se necesitan al menos siete direcciones IP** en las subredes tanto de las máquinas de origen como de las máquinas de recuperación. Cuando se crea un punto de conexión privado para el almacén, Site Recovery crea cinco vínculos privados para el acceso a sus microservicios. Además, al habilitar la replicación, se agregan dos vínculos privados adicionales para el emparejamiento de la región de origen y de destino.
- **Se requiere una dirección IP adicional** en las subredes de origen y de recuperación. Esta dirección IP solo es necesaria cuando necesita usar puntos de conexión privados que se conectan a cuentas de almacenamiento en caché.
  Los puntos de conexión privados para el almacenamiento solo se pueden crear en un tipo De uso general v2. Revise la estructura de precios para la [transferencia de datos De uso general v2](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

 ## <a name="creating-and-using-private-endpoints-for-site-recovery"></a>Creación y uso de puntos de conexión privados para Site Recovery

 En esta sección se tratan los pasos necesarios para crear y usar puntos de conexión privados para Azure Site Recovery dentro de las redes virtuales.

> [!NOTE]
> Se recomienda encarecidamente seguir estos pasos en la misma secuencia que se mencionan. Si no lo hace, puede que el almacén sea incapaz de usar puntos de conexión privados y que usted tenga que reiniciar el proceso con un nuevo almacén.

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Un almacén de Recovery Services es una entidad que contiene la información de replicación de las máquinas y se usa para desencadenar operaciones de Site Recovery. Para obtener más información, consulte [Creación de un almacén de Recovery Services](./azure-to-azure-tutorial-enable-replication.md#create-a-recovery-services-vault).

## <a name="enable-the-managed-identity-for-the-vault"></a>Habilite la identidad administrada del almacén.

Una [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) permite que el almacén obtenga acceso a las cuentas de almacenamiento del cliente. Site Recovery necesita acceder a las cuentas de almacenamiento de origen, almacenamiento de destino y almacenamiento en caché/registro según el requisito del escenario.
El acceso con la identidad administrada es esencial cuando se usa el servicio de vínculos privados para el almacén.

1. Vaya al almacén de Recovery Services. Seleccione **Identidad** en _Configuración_.

   :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/enable-managed-identity-in-vault.png" alt-text="Muestra Azure Portal y la página Recovery Services.":::

1. Cambie el **Estado** a _Activado_ y seleccione **Guardar**.

1. Se genera un **Id. de objeto** que indica que el almacén se ha registrado en Azure Active Directory.

## <a name="create-private-endpoints-for-the-recovery-services-vault"></a>Creación de puntos de conexión privados para el almacén de Recovery Services

Para habilitar la conmutación por error y la conmutación por recuperación en máquinas virtuales de Azure, necesitará dos puntos de conexión privados para el almacén. Un punto de conexión privado para la protección de máquinas en la red de origen y otro para la reprotección de las máquinas con conmutación por error en la red de recuperación.

Asegúrese de crear también una red virtual de recuperación en la región de destino durante este proceso de instalación.

Cree el primer punto de conexión privado para el almacén dentro de la red virtual de origen mediante Private Link Center en el portal o a través de [Azure PowerShell](../private-link/create-private-endpoint-powershell.md). Cree el segundo punto de conexión privado para el almacén dentro de la red de recuperación. A continuación se indican los pasos para crear el punto de conexión privado en la red de origen. Repita la misma guía para crear el segundo punto de conexión privado.

1. En la barra de búsqueda de Azure Portal, busque y seleccione "Private Link". Esta acción le llevará a Private Link Center.

   :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/search-private-links.png" alt-text="Muestra la búsqueda de Private Link Center en Azure Portal.":::

1. En la barra de navegación de la izquierda, seleccione **Puntos de conexión privados**. Una vez que esté en el panel Puntos de conexión privados, seleccione **\+Agregar** para empezar a crear un punto de conexión privado para el almacén.

   :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/create-private-endpoints.png" alt-text="Muestra la creación de un punto de conexión privado en Private Link Center.":::

1. Una vez que esté en la experiencia de "creación de un punto de conexión privado", se le pedirá que especifique los detalles para crear la conexión del punto de conexión privado.

   1. **Aspectos básicos**: Rellene los detalles básicos de los puntos de conexión privados. La región debe ser la misma que la de las máquinas de origen.

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/create-private-endpoints-basic-tab.png" alt-text="Muestra la pestaña Básico, los detalles del proyecto, la suscripción y otros campos relacionados para crear un punto de conexión privado en Azure Portal.":::

   1. **Recursos**: Esta pestaña requiere que mencione el recurso de plataforma como servicio para el que desea crear la conexión. Seleccione _Microsoft.RecoveryServices/vaults_ de **Tipo de recurso** para la suscripción seleccionada. Luego, elija el nombre del almacén de Recovery Services para **Recurso** y establezca _Azure Site Recovery_ como **Subrecurso de destino**.

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/create-private-endpoints-resource-tab.png" alt-text="Muestra la pestaña Recurso, los campos de tipo de recurso, recurso y subrecurso de destino para la vinculación a un punto de conexión privado en Azure Portal.":::

   1. **Configuración**: En Configuración, especifique la red virtual y la subred en la que desea que se cree el punto de conexión privado. Esta red virtual es la red en la que se encuentra la máquina virtual. Para habilitar la integración en la zona DNS privada, seleccione **Sí**. Elija una zona DNS ya creada o cree una nueva. Al seleccionar **Sí**, se vincula automáticamente la zona a la red virtual de origen y se agregan los registros DNS necesarios para la resolución DNS de nuevas direcciones IP y nombres de dominio completos creados para el punto de conexión privado.

      Asegúrese de elegir crear una nueva zona DNS para cada nuevo punto de conexión privado que se conecta al mismo almacén. Si elige una zona DNS privada existente, se sobrescriben los registros CNAME anteriores. Consulte la [guía de puntos de conexión privados](../private-link/private-endpoint-overview.md#private-endpoint-properties) antes de continuar.

      Si su entorno tiene un modelo de tipo hub-and-spoke, solo necesita un punto de conexión privado y solo una zona DNS privada para toda la instalación, ya que todas las redes virtuales ya tienen habilitado el emparejamiento entre ellas. Para obtener más información, consulte [Integración de DNS de punto de conexión privado](../private-link/private-endpoint-dns.md#virtual-network-workloads-without-custom-dns-server).

      Para crear manualmente la zona DNS privada, siga los pasos descritos en [Creación de una zona DNS privada y adición de registros DNS manualmente](#create-private-dns-zones-and-add-dns-records-manually).

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/create-private-endpoints-configuration-tab.png" alt-text="Muestra la pestaña Configuración con los campos de redes e integración de DNS para la configuración de un punto de conexión privado en Azure Portal.":::

   1. **Etiquetas**: Opcionalmente, puede agregar etiquetas para el punto de conexión privado.

   1. **Revisar \+ crear**: Una vez finalizada la validación, seleccione **Crear** para crear el punto de conexión privado.

Una vez creado el punto de conexión privado, se agregan cinco nombres de dominio completos al punto de conexión privado. Estos vínculos permiten a las máquinas de la red virtual acceder a todos los microservicios de Site Recovery necesarios en el contexto del almacén. Más adelante, cuando se habilita la replicación, se agregan dos nombres de dominio completos adicionales al mismo punto de conexión privado.

Los cinco nombres de dominio reciben el formato con el siguiente patrón:

`{Vault-ID}-asr-pod01-{type}-.{target-geo-code}.siterecovery.windowsazure.com`

## <a name="approve-private-endpoints-for-site-recovery"></a>Aprobación de puntos de conexión privados para Site Recovery

Si el usuario que crea el punto de conexión privado también es el propietario del almacén de Recovery Services, el punto de conexión privado creado anteriormente se aprueba automáticamente a los pocos minutos. De lo contrario, el propietario del almacén debe aprobar el punto de conexión privado antes de que lo use. Para aprobar o rechazar una conexión de punto de conexión privado solicitada, vaya a **Conexiones de punto de conexión privado** en "Configuración" en la página del almacén de recuperación.

Puede ir al recurso de punto de conexión privado para revisar el estado de la conexión antes de continuar.

:::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/vault-private-endpoint-connections.png" alt-text="Muestra la página Conexiones de punto de conexión privado del almacén y la lista de conexiones en Azure Portal.":::

## <a name="optional-create-private-endpoints-for-the-cache-storage-account"></a><a name="create-private-endpoints-for-the-cache-storage-account"></a>(Opcional) Creación de puntos de conexión privados para la cuenta de almacenamiento de caché

Se puede usar un punto de conexión privado para Azure Storage. La creación de puntos de conexión privados para el acceso al almacenamiento es _opcional_ para la replicación de Azure Site Recovery. Al crear un punto de conexión privado para el almacenamiento, se aplican los siguientes requisitos:

- Necesita un punto de conexión privado para la cuenta de almacenamiento de caché o de registro en la red virtual de origen.
- Necesita un segundo punto de conexión privado en el momento de la reprotección de las máquinas con conmutación por error en la red de recuperación. Este punto de conexión privado es para la nueva cuenta de almacenamiento creada en la región de destino.

> [!NOTE]
> Los puntos de conexión privados para el almacenamiento solo se pueden crear en cuentas de almacenamiento de tipo **De uso general v2**. Para obtener información sobre los precios, consulte [Precios de blobs en páginas estándar](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

Siga la [guía para la creación de almacenamiento privado](../private-link/create-private-endpoint-storage-portal.md#create-your-private-endpoint) para crear una cuenta de almacenamiento con un punto de conexión privado. Asegúrese de seleccionar **Sí** para la integración en una zona DNS privada. Seleccione una zona DNS ya creada o cree una nueva.

## <a name="grant-required-permissions-to-the-vault"></a>Concesión de los permisos necesarios para el almacén

Si las máquinas virtuales usan discos administrados, debe conceder los permisos de identidad administrada solo a las cuentas de almacenamiento en caché. En caso de que las máquinas virtuales usen discos no administrados, debe conceder los permisos de identidad administrada para las cuentas almacenamiento de origen, en caché y de destino. En este caso, debe crear de antemano la cuenta de almacenamiento de destino.

Antes de habilitar la replicación de máquinas virtuales, la identidad administrada del almacén debe tener los siguientes permisos de rol en función del tipo de cuenta de almacenamiento:

- Cuentas de almacenamiento basadas en Resource Manager (tipo Estándar):
  - [Colaborador](../role-based-access-control/built-in-roles.md#contributor)
  - [Colaborador de datos de blobs de almacenamiento](../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)
- Cuentas de almacenamiento basadas en Resource Manager (tipo Premium):
  - [Colaborador](../role-based-access-control/built-in-roles.md#contributor)
  - [Propietario de datos de blobs de almacenamiento](../role-based-access-control/built-in-roles.md#storage-blob-data-owner)
- Cuentas de almacenamiento clásicas:
  - [Colaborador de cuentas de almacenamiento clásico](../role-based-access-control/built-in-roles.md#classic-storage-account-contributor)
  - [Rol de servicio de operador de claves de cuentas de almacenamiento clásicas](../role-based-access-control/built-in-roles.md#classic-storage-account-key-operator-service-role)

En los pasos siguientes se describe cómo agregar una asignación de roles a las cuentas de almacenamiento, de una en una:

1. Vaya a la cuenta de almacenamiento y navegue a **Control de acceso (IAM)** en el lado izquierdo de la página.

1. Cuando esté en **Control de acceso (IAM)** , seleccione **Agregar** en "Agregar una asignación de roles".

   :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/storage-role-assignment.png" alt-text="Muestra la página Control de acceso (IAM) en una cuenta de almacenamiento y el botón Agregar una asignación de roles en Azure Portal.":::

1. En la página lateral "Agregar una asignación de roles", elija el rol en la lista anterior en la lista desplegable **Role**. Escriba el **nombre** del almacén y seleccione **Guardar**.

   :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/storage-role-assignment-select-role.png" alt-text="Muestra la página Control de acceso (IAM) en una cuenta de almacenamiento y las opciones para seleccionar un rol y la entidad de seguridad a la que se va a conceder ese rol en Azure Portal.":::

Además de estos permisos, también es necesario permitir el acceso a los servicios de confianza de MS. Vaya a "Firewalls y redes virtuales" y seleccione la casilla "Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento" en **Excepciones**.

## <a name="protect-your-virtual-machines"></a>Protección de las máquinas virtuales

Una vez que se completen todas las configuraciones anteriores, continúe con la habilitación de la replicación de las máquinas virtuales. Todas las operaciones de Site Recovery funcionan sin pasos adicionales si se usó la integración de DNS al crear puntos de conexión privados en el almacén. Sin embargo, si las zonas DNS se crean y configuran manualmente, se necesitan pasos adicionales para agregar registros DNS específicos en las zonas DNS de origen y destino después de habilitar la replicación. Para obtener más información y conocer los pasos, consulte [Creación de una zona DNS privada y adición de registros DNS manualmente](#create-private-dns-zones-and-add-dns-records-manually).

## <a name="create-private-dns-zones-and-add-dns-records-manually"></a>Creación de una zona DNS privada y adición de registros DNS manualmente

Si no seleccionó la opción de integración en la zona DNS privada en el momento de crear un punto de conexión privado para el almacén, siga los pasos de esta sección.

Cree una zona DNS privada para permitir que el agente de movilidad resuelva los nombres de dominio completos de los vínculos privados en direcciones IP privadas.

1. Crear una zona DNS privada

   1. Busque "Zona DNS privada" en la barra de búsqueda **Todos los servicios** y seleccione "Zona DNS privada" en la lista desplegable.

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/search-private-dns-zone.png" alt-text="Muestra la búsqueda zona DNS privada en la página de nuevos recursos en Azure Portal.":::

   1. Una vez que se encuentre en la página "Zona DNS privada", haga clic en el botón **\+Agregar** para empezar a crear una nueva zona.

   1. En la página "Crear zona DNS privada", rellene los detalles necesarios. Escriba el nombre de la zona DNS privada como `privatelink.siterecovery.windowsazure.com`. Puede elegir cualquier grupo de recursos y cualquier suscripción para crearla.

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/create-private-dns-zone.png" alt-text="Muestra la pestaña Básico de la página Crear zona DNS privada y los detalles relacionados del proyecto en Azure Portal.":::

   1. Vaya a la pestaña **Revisar \+ crear** para revisar y crear la zona DNS.

1. Vincular la zona DNS privada a la red virtual

   Ahora, las zonas DNS privadas creadas anteriormente deben vincularse a la red virtual en la que se encuentran los servidores actualmente. También debe vincular de antemano la zona DNS privada a la red virtual de destino.

   1. Vaya a la zona DNS privada que creó en el paso anterior y diríjase a **Vínculos de red virtual** en la barra de la izquierda de la página. Una vez allí, seleccione el botón **\+Agregar**.

   1. Ingrese todos los detalles obligatorios. Los campos **Suscripción** y **Red virtual** deben rellenarse con los detalles correspondientes de la red virtual en la que se encuentran los servidores. Los demás campos deben dejarse tal cual.

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/add-virtual-network-link.png" alt-text="Muestra la página para agregar un vínculo de red virtual con el nombre del vínculo, la suscripción y la red virtual relacionada en Azure Portal.":::

1. Agregar registros DNS

   Una vez que haya creado las zonas DNS privadas necesarias y los puntos de conexión privados, deberá agregar los registros DNS a las zonas DNS.

   > [!NOTE]
   > En caso de que use una zona DNS privada personalizada, asegúrese de que se realicen entradas similares, tal como se describe a continuación.

   Este paso requiere que se creen entradas para cada nombre de dominio completo del punto de conexión privado en la zona DNS privada.

   1. Vaya a la zona DNS privada y navegue hasta la sección **Información general** a la izquierda de la página. Una vez allí, seleccione **\+Conjunto de registros** para empezar a agregar registros.

   1. En el panel "Agregar conjunto de registros" que se abre, agregue una entrada para cada nombre de dominio completo y dirección IP privada como un registro de tipo _A_. La lista de nombres de dominio completos y direcciones IP puede obtenerse de la página "Punto de conexión privado" en **Información general**. Como se muestra en el ejemplo siguiente, el primer nombre de dominio completo del punto de conexión privado se agrega al conjunto de registros de la zona DNS privada.

      Estos nombres de dominio completos coinciden con el patrón: `{Vault-ID}-asr-pod01-{type}-.{target-geo-code}.siterecovery.windowsazure.com`

      :::image type="content" source="./media/azure-to-azure-how-to-enable-replication-private-endpoints/add-record-set.png" alt-text="Muestra la página para agregar un registro DNS de tipo A para el nombre de dominio completo al punto de conexión privado en Azure Portal.":::

   > [!NOTE]
   > Después de habilitar la replicación, se crean dos nombres de dominio completos adicionales en los puntos de conexión privados en ambas regiones. Asegúrese de agregar también los registros DNS para los nombres de dominio completos recién creados.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha habilitado los puntos de conexión privados para la replicación de máquinas virtuales, consulte estas otras páginas para obtener información adicional y relacionada:

- [Replicación de máquinas virtuales de Azure en otra región de Azure](./azure-to-azure-how-to-enable-replication.md)
- [Tutorial: Configuración de la recuperación ante desastres de máquinas virtuales de Azure](./azure-to-azure-tutorial-enable-replication.md)