---
title: Habilitación de la replicación de máquinas locales con puntos de conexión privados en Azure Site Recovery
description: En este artículo se describe cómo configurar la replicación de máquinas locales con puntos de conexión privados mediante Site Recovery.
author: mayurigupta13
ms.author: mayg
ms.service: site-recovery
ms.topic: article
ms.date: 07/14/2020
ms.openlocfilehash: c91c92a18570f569b6364b646e6fdf7bf534802f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095322"
---
# <a name="replicate-on-premises-machines-with-private-endpoints"></a>Replicación de máquinas locales con puntos de conexión privados

Azure Site Recovery permite usar puntos de conexión privados de [Azure Private Link](../private-link/private-endpoint-overview.md) para replicar las máquinas locales en una red virtual de Azure. La compatibilidad con el acceso de puntos de conexión privados a un almacén de recuperación es compatible con las siguientes regiones:

- Azure Commercial: Centro y Sur de EE. UU., Oeste de EE. UU. 2, Este de EE. UU.
- Azure Government: US Gov Virginia, US Gov Arizona, US Gov Texas, US DoD (este), US DoD (centro)

En este artículo se proporcionan instrucciones para seguir los pasos a continuación:

- Crear un almacén de Recovery Services de Azure Backup para proteger sus máquinas.
- Habilitar una identidad administrada para el almacén y conceder los permisos necesarios para tener acceso a las cuentas de almacenamiento de los clientes con el fin de replicar el tráfico del entorno local en ubicaciones de destino de Azure. El acceso a la identidad administrada para el almacenamiento es necesario cuando se configura el acceso de Private Link al almacén.
- Realizar cambios de DNS necesarios en puntos de conexión privados.
- Crear y aprobar puntos de conexión privados para un almacén dentro de una red virtual.
- Crear puntos de conexión privados para las cuentas de almacenamiento. Puede seguir permitiendo el acceso público o con firewall para el almacenamiento según sea necesario. La creación de un punto de conexión privado para el acceso al almacenamiento no es obligatoria en Azure Site Recovery.
  
A continuación se muestra una arquitectura de referencia sobre cómo el flujo de trabajo de replicación cambia para la recuperación ante desastres híbrida con los puntos de conexión privados. No se pueden crear puntos de conexión privados en la red local. Para usar vínculos privados, debe crear una red virtual de Azure (denominada "red de derivación" en este artículo), establecer la conectividad privada entre el entorno local y la red de derivación y, a continuación, crear puntos de conexión privados en la red de derivación. La elección de la conectividad privada queda a su criterio.

:::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/architecture.png" alt-text="Arquitectura de referencia para Site Recovery con puntos de conexión privados.":::

## <a name="prerequisites-and-caveats"></a>Requisitos previos y advertencias

- La compatibilidad con vínculos privados está habilitada para la infraestructura de Site Recovery con la versión **9.35** y versiones posteriores.
- Los puntos de conexión privados se pueden crear solo para almacenes de Recovery Services nuevos que no tienen elementos registrados. Por lo tanto, los puntos de conexión privados **deben crearse antes de agregar ningún elemento al almacén**. Revise la estructura de precios para los [puntos de conexión privados](https://azure.microsoft.com/pricing/details/private-link/).
- Cuando se crea un punto de conexión privado para un almacén, el almacén se bloquea y **no es posible acceder desde redes que no sean las redes que tienen puntos de conexión privados**.
- Actualmente, Azure Active Directory no admite puntos de conexión privados. Por lo tanto, las direcciones IP y los nombres de dominio completos necesarios para que Azure Active Directory funcione en una región deben tener permiso de acceso de salida desde la red virtual de Azure protegida. También puede usar la etiqueta de grupo de seguridad de red "Azure Active Directory" y etiquetas de Azure Firewall para permitir el acceso a Azure Active Directory, según corresponda.
- **Se necesitan cinco direcciones IP** en la red de derivación en la que se crea el punto de conexión privado. Cuando se crea un punto de conexión privado para el almacén, Site Recovery crea cinco vínculos privados para el acceso a sus microservicios.
- **Se requiere una dirección IP adicional** en la red de derivación para la conectividad de los puntos de conexión privados a una cuenta de almacenamiento en caché. El método de conectividad, como Internet o [ExpressRoute](../expressroute/index.yml), entre el sitio local y el punto de conexión de la cuenta de almacenamiento queda a su criterio. El establecimiento de un vínculo privado es opcional. Los puntos de conexión privados para el almacenamiento solo se pueden crear en un tipo De uso general v2. Revise la estructura de precios para la [transferencia de datos De uso general v2](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

 ## <a name="creating-and-using-private-endpoints-for-site-recovery"></a>Creación y uso de puntos de conexión privados para Site Recovery

 En esta sección se tratan los pasos necesarios para crear y usar puntos de conexión privados para Azure Site Recovery dentro de las redes virtuales.

> [!NOTE]
> Se recomienda encarecidamente seguir estos pasos en la misma secuencia que se mencionan. Si no lo hace, puede que el almacén sea incapaz de usar puntos de conexión privados y que usted tenga que reiniciar el proceso con un nuevo almacén.

## <a name="create-a-recovery-services-vault"></a>Cree un almacén de Recovery Services.

Un almacén de Recovery Services es una entidad que contiene la información de replicación de las máquinas y se usa para desencadenar operaciones de Site Recovery. Para conocer los pasos para crear un almacén de Recovery Services en la región de Azure en la que desea realizar la conmutación por error si se produce un desastre, consulte [Creación de un almacén de Recovery Services](./azure-to-azure-tutorial-enable-replication.md#create-a-recovery-services-vault).

## <a name="enable-the-managed-identity-for-the-vault"></a>Habilite la identidad administrada del almacén.

Una [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) permite que el almacén obtenga acceso a las cuentas de almacenamiento del cliente. Site Recovery necesita acceder a las cuentas de almacenamiento de destino y de almacenamiento en caché/registro según el requisito del escenario. El acceso con la identidad administrada es esencial cuando se usa el servicio de vínculos privados para el almacén.

1. Vaya al almacén de Recovery Services. Seleccione **Identidad** en _Configuración_.

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/enable-managed-identity-in-vault.png" alt-text="Muestra Azure Portal y la página Recovery Services.":::

1. Cambie el **Estado** a _Activado_ y seleccione **Guardar**.

1. Se genera un **Id. de objeto** que indica que el almacén se ha registrado en Azure Active Directory.

## <a name="create-private-endpoints-for-the-recovery-services-vault"></a>Creación de puntos de conexión privados para el almacén de Recovery Services

Necesitará un punto de conexión privado para el almacén en la red de derivación para la protección de las máquinas en la red de origen local. Cree el punto de conexión privado mediante Private Link Center en Azure Portal o a través de [Azure PowerShell](../private-link/create-private-endpoint-powershell.md).

1. En la barra de búsqueda de Azure Portal, busque y seleccione "Private Link". Esta acción le llevará a Private Link Center.

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/search-private-links.png" alt-text="Muestra la búsqueda de Private Link Center en Azure Portal.":::

1. En la barra de navegación de la izquierda, seleccione **Puntos de conexión privados**. Una vez que esté en el panel Puntos de conexión privados, seleccione **\+Agregar** para empezar a crear un punto de conexión privado para el almacén.

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints.png" alt-text="Muestra la creación de un punto de conexión privado en Private Link Center.":::

1. Una vez que esté en la experiencia de "creación de un punto de conexión privado", se le pedirá que especifique los detalles para crear la conexión del punto de conexión privado.

   1. **Aspectos básicos**: Rellene los detalles básicos de los puntos de conexión privados. La región debe ser la misma que la de la red de derivación.

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints-basic-tab.png" alt-text="Muestra la pestaña Básico, los detalles del proyecto, la suscripción y otros campos relacionados para crear un punto de conexión privado en Azure Portal.":::

   1. **Recursos**: Esta pestaña requiere que mencione el recurso de plataforma como servicio para el que desea crear la conexión. Seleccione _Microsoft.RecoveryServices/vaults_ de **Tipo de recurso** para la suscripción seleccionada. Luego, elija el nombre del almacén de Recovery Services para **Recurso** y establezca _Azure Site Recovery_ como **Subrecurso de destino**.

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints-resource-tab.png" alt-text="Muestra la pestaña Recurso, los campos de tipo de recurso, recurso y subrecurso de destino para la vinculación a un punto de conexión privado en Azure Portal.":::

   1. **Configuración**: En Configuración, especifique la red de derivación y la subred en la que desea que se cree el punto de conexión privado. Para habilitar la integración en la zona DNS privada, seleccione **Sí**.
      Elija una zona DNS ya creada o cree una nueva. Al seleccionar **Sí**, se vincula automáticamente la zona a la red de derivación y se agregan los registros DNS necesarios para la resolución DNS de nuevas direcciones IP y nombres de dominio completos creados para el punto de conexión privado.

      Asegúrese de elegir crear una nueva zona DNS para cada nuevo punto de conexión privado que se conecta al mismo almacén. Si elige una zona DNS privada existente, se sobrescriben los registros CNAME anteriores. Consulte la [guía de puntos de conexión privados](../private-link/private-endpoint-overview.md#private-endpoint-properties) antes de continuar.

      Si su entorno tiene un modelo de tipo hub-and-spoke, solo necesita un punto de conexión privado y solo una zona DNS privada para toda la instalación, ya que todas las redes virtuales ya tienen habilitado el emparejamiento entre ellas. Para obtener más información, consulte [Integración de DNS de punto de conexión privado](../private-link/private-endpoint-dns.md#virtual-network-workloads-without-custom-dns-server).

      Para crear manualmente la zona DNS privada, siga los pasos descritos en [Creación de una zona DNS privada y adición de registros DNS manualmente](#create-private-dns-zones-and-add-dns-records-manually).

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints-configuration-tab.png" alt-text="Muestra la pestaña Configuración con los campos de redes e integración de DNS para la configuración de un punto de conexión privado en Azure Portal.":::

   1. **Etiquetas**: Opcionalmente, puede agregar etiquetas para el punto de conexión privado.

   1. **Revisar \+ crear**: Una vez finalizada la validación, seleccione **Crear** para crear el punto de conexión privado.

Una vez creado el punto de conexión privado, se agregan cinco nombres de dominio completos al punto de conexión privado. Estos vínculos permiten a las máquinas de la red local acceder a todos los microservicios de Site Recovery necesarios en el contexto del almacén a través de la red de derivación. Se puede usar el mismo punto de conexión privado para la protección de cualquier máquina de Azure en la red de derivación y todas las redes emparejadas.

Los cinco nombres de dominio reciben el formato con el siguiente patrón:

`{Vault-ID}-asr-pod01-{type}-.{target-geo-code}.siterecovery.windowsazure.com`

## <a name="approve-private-endpoints-for-site-recovery"></a>Aprobación de puntos de conexión privados para Site Recovery

Si el usuario que crea el punto de conexión privado también es el propietario del almacén de Recovery Services, el punto de conexión privado creado anteriormente se aprueba automáticamente a los pocos minutos. De lo contrario, el propietario del almacén debe aprobar el punto de conexión privado antes de que lo use. Para aprobar o rechazar una conexión de punto de conexión privado solicitada, vaya a **Conexiones de punto de conexión privado** en "Configuración" en la página del almacén de recuperación.

Puede ir al recurso de punto de conexión privado para revisar el estado de la conexión antes de continuar.

:::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/vault-private-endpoint-connections.png" alt-text="Muestra la página Conexiones de punto de conexión privado del almacén y la lista de conexiones en Azure Portal.":::

## <a name="optional-create-private-endpoints-for-the-cache-storage-account"></a><a name="create-private-endpoints-for-the-cache-storage-account"></a>(Opcional) Creación de puntos de conexión privados para la cuenta de almacenamiento de caché

Se puede usar un punto de conexión privado para Azure Storage. La creación de puntos de conexión privados para el acceso al almacenamiento es _opcional_ para la replicación de Azure Site Recovery. Al crear un punto de conexión privado para el almacenamiento, necesita un punto de conexión privado para la cuenta de almacenamiento en caché o registro en la red virtual de derivación.

> [!NOTE]
> Los puntos de conexión privados para el almacenamiento solo se pueden crear en cuentas de almacenamiento de tipo **De uso general v2**. Para obtener información sobre los precios, consulte [Precios de blobs en páginas estándar](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

Siga la [guía para la creación de almacenamiento privado](../private-link/create-private-endpoint-storage-portal.md#create-your-private-endpoint) para crear una cuenta de almacenamiento con un punto de conexión privado. Asegúrese de seleccionar **Sí** para la integración en una zona DNS privada. Seleccione una zona DNS ya creada o cree una nueva.

## <a name="grant-required-permissions-to-the-vault"></a>Concesión de los permisos necesarios para el almacén

En función de la configuración, es posible que necesite una o varias cuentas de almacenamiento en la región de Azure de destino. A continuación, conceda a la identidad administrada permisos para todas las cuentas de almacenamiento en caché o registro necesarias para Site Recovery. En este caso, debe crear de antemano las cuentas de almacenamiento necesarias.

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

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/storage-role-assignment.png" alt-text="Muestra la página Control de acceso (IAM) en una cuenta de almacenamiento y el botón "Agregar una asignación de roles" en Azure Portal.":::

1. En la página lateral "Agregar una asignación de roles", elija el rol en la lista anterior en la lista desplegable **Role**. Escriba el **nombre** del almacén y seleccione **Guardar**.

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/storage-role-assignment-select-role.png" alt-text="Muestra la página Control de acceso (IAM) en una cuenta de almacenamiento y las opciones para seleccionar un rol y la entidad de seguridad a la que se va a conceder ese rol en Azure Portal.":::

Además de estos permisos, también es necesario permitir el acceso a los servicios de confianza de MS. Vaya a "Firewalls y redes virtuales" y seleccione la casilla "Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento" en **Excepciones**.

## <a name="protect-your-virtual-machines"></a>Protección de las máquinas virtuales

Una vez que complete todas las configuraciones anteriores, continúe con la configuración de la infraestructura local.

- [Implementación del servidor de configuración de VMware y de las máquinas físicas](./vmware-azure-deploy-configuration-server.md)
- O bien [Configuración del entorno de Hyper-V para la replicación](./hyper-v-azure-tutorial.md#set-up-the-source-environment)

Una vez que complete la configuración, habilite la replicación de las máquinas de origen. Asegúrese de que la configuración de la infraestructura se realice solo después de que ya se hayan creado los puntos de conexión privados para el almacén en la red de derivación.

## <a name="create-private-dns-zones-and-add-dns-records-manually"></a>Creación de una zona DNS privada y adición de registros DNS manualmente

Si no seleccionó la opción de integración en la zona DNS privada en el momento de crear un punto de conexión privado para el almacén, siga los pasos de esta sección.

Cree una zona DNS privada para permitir que el proveedor de Site Recovery (para máquinas de Hyper-V) o el servidor de procesos (para VMware o máquinas físicas) resuelvan los nombres de dominio completos de vínculos privados en direcciones IP privadas.

1. Crear una zona DNS privada

   1. Busque "Zona DNS privada" en la barra de búsqueda **Todos los servicios** y seleccione "Zona DNS privada" en la lista desplegable.

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/search-private-dns-zone.png" alt-text="Muestra la búsqueda "zona DNS privada" en la página de nuevos recursos en Azure Portal.":::

   1. Una vez que se encuentre en la página "Zona DNS privada", haga clic en el botón **\+Agregar** para empezar a crear una nueva zona.

   1. En la página "Crear zona DNS privada", rellene los detalles necesarios. Escriba el nombre de la zona DNS privada como `privatelink.siterecovery.windowsazure.com`. Puede elegir cualquier grupo de recursos y cualquier suscripción para crearla.

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-dns-zone.png" alt-text="Muestra la pestaña Básico de la página Crear zona DNS privada y los detalles relacionados del proyecto en Azure Portal.":::

   1. Vaya a la pestaña **Revisar \+ crear** para revisar y crear la zona DNS.

1. Vincular la zona DNS privada a la red virtual

   La zona DNS privada creada anteriormente debe estar vinculada ahora a la derivación.

   1. Vaya a la zona DNS privada que creó en el paso anterior y diríjase a **Vínculos de red virtual** en la barra de la izquierda de la página. Una vez allí, seleccione el botón **\+Agregar**.

   1. Ingrese todos los detalles obligatorios. Los campos **Suscripción** y **Red virtual** deben rellenarse con los detalles correspondientes de la red de derivación. Los demás campos deben dejarse tal cual.

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/add-virtual-network-link.png" alt-text="Muestra la página para agregar un vínculo de red virtual con el nombre del vínculo, la suscripción y la red virtual relacionada en Azure Portal.":::

1. Agregar registros DNS

   Una vez que haya creado la zona DNS privada necesaria y el punto de conexión privado, deberá agregar los registros DNS a la zona DNS.

   > [!NOTE]
   > En caso de que use una zona DNS privada personalizada, asegúrese de que se realicen entradas similares, tal como se describe a continuación.

   Este paso requiere que se creen entradas para cada nombre de dominio completo del punto de conexión privado en la zona DNS privada.

   1. Vaya a la zona DNS privada y navegue hasta la sección **Información general** a la izquierda de la página. Una vez allí, seleccione **\+Conjunto de registros** para empezar a agregar registros.

   1. En el panel "Agregar conjunto de registros" que se abre, agregue una entrada para cada nombre de dominio completo y dirección IP privada como un registro de tipo _A_. La lista de nombres de dominio completos y direcciones IP puede obtenerse de la página "Punto de conexión privado" en **Información general**. Como se muestra en el ejemplo siguiente, el primer nombre de dominio completo del punto de conexión privado se agrega al conjunto de registros de la zona DNS privada.

      Estos nombres de dominio completos coinciden con el patrón: `{Vault-ID}-asr-pod01-{type}-.{target-geo-code}.siterecovery.windowsazure.com`

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/add-record-set.png" alt-text="Muestra la página para agregar un registro DNS de tipo A para el nombre de dominio completo al punto de conexión privado en Azure Portal.":::

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha habilitado los puntos de conexión privados para la replicación de máquinas virtuales, consulte estas otras páginas para obtener información adicional y relacionada:

- [Implementación de un servidor de configuración local](./vmware-azure-deploy-configuration-server.md)
- [Configuración de la recuperación ante desastres de máquinas virtuales de Hyper-V locales en Azure](./hyper-v-azure-tutorial.md)