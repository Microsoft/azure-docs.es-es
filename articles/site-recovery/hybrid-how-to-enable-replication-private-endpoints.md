---
title: Habilitación de la replicación de máquinas locales con puntos de conexión privados
description: En este artículo se describe cómo configurar la replicación de máquinas locales mediante el uso de puntos de conexión privados en Site Recovery.
author: mayurigupta13
ms.author: mayg
ms.service: site-recovery
ms.topic: article
ms.date: 07/14/2020
ms.openlocfilehash: 13c19f07ac21f986a5523407e46c59c050ebf96d
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88142084"
---
# <a name="replicate-on-premises-machines-by-using-private-endpoints"></a>Replicación de máquinas locales mediante puntos de conexión privados

Azure Site Recovery permite usar puntos de conexión privados de [Azure Private Link](../private-link/private-endpoint-overview.md) para replicar máquinas locales en una red virtual de Azure. El acceso de un punto de conexión privado a un almacén de recuperación se admite en estas regiones:

- Azure Commercial: Centro y Sur de EE. UU., Oeste de EE. UU. 2, Este de EE. UU.
- Azure Government: US Gov Virginia, US Gov Arizona, US Gov Texas, US DoD (este), US DoD (centro)

En este artículo se describe cómo completar los pasos siguientes:

- Crear un almacén de Recovery Services de Azure Backup para proteger sus máquinas.
- Habilitar una identidad administrada para el almacén. Conceda los permisos necesarios para acceder a las cuentas de almacenamiento con el fin de habilitar la replicación del tráfico desde las ubicaciones locales a las de destino de Azure. Para que Private Link acceda al almacén, se necesita el acceso a la identidad administrada para el almacenamiento.

- Realizar cambios en el DNS que son necesarios para los puntos de conexión privados.
- Crear y aprobar puntos de conexión privados para un almacén que esté en una red virtual.
- Crear puntos de conexión privados para las cuentas de almacenamiento. Puede seguir permitiendo el acceso público o con firewall para el almacenamiento según sea necesario. Azure Site Recovery no necesita que se cree ningún punto de conexión privado para acceder al almacenamiento.
  
En el diagrama siguiente se muestra el flujo de trabajo de la replicación en una recuperación ante desastres híbrida con puntos de conexión privados. No se pueden crear puntos de conexión privados en una red local. Para usar vínculos privados es preciso crear una red virtual de Azure (que en este artículo se denomina *red de derivación*), establecer la conectividad privada entre el entorno local y la red de derivación y, seguidamente, crear puntos de conexión privados en la red de derivación. Puede elegir cualquier forma de conectividad privada.

:::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/architecture.png" alt-text="Diagrama que muestra la arquitectura de Azure Site Recovery y los puntos de conexión privados.":::

## <a name="prerequisites-and-caveats"></a>Requisitos previos y advertencias

- Los vínculos privados se admiten tanto en Site Recovery 9.35 como en las versiones posteriores.
- Solo se pueden crear puntos de conexión privados para almacenes de Recovery Services nuevos que no tengan elementos registrados. Por consiguiente, deben crearse antes de agregar ningún elemento al almacén. Para más información sobre los precios, consulte [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/).
- Cuando se crea un punto de conexión privado para un almacén, el almacén se bloquea. Solo se puede acceder a él desde aquellas redes que tengan puntos de conexión privados.
- Actualmente, Azure Active Directory no admite puntos de conexión privados. Por consiguiente, es preciso permitir el acceso saliente desde la red virtual de Azure protegida a las direcciones IP y los nombres de dominio completos necesarios para que Azure Active Directory funcione en una región.
  Según corresponda, también puede usar la etiqueta de grupo de seguridad de red "Azure Active Directory" y las etiquetas de Azure Firewall para permitir el acceso a Azure Active Directory.
- Se necesitan cinco direcciones IP en la red de derivación en la que se crea el punto de conexión privado. Cuando se crea un punto de conexión privado para el almacén, Site Recovery crea cinco vínculos privados para el acceso a sus microservicios.
- Se requiere una dirección IP adicional en la red de derivación para la conectividad de los puntos de conexión privados a una cuenta de almacenamiento en caché. Puede usar cualquier método de conectividad entre el entorno local y su punto de conexión de la cuenta de almacenamiento. Por ejemplo, puede usar Internet o Azure [ExpressRoute](../expressroute/index.yml). El establecimiento de un vínculo privado es opcional. Solo se pueden crear puntos de conexión privados para el almacenamiento en cuentas De uso general v2. Para más información sobre los precios de la transferencia de datos en cuentas De uso general v2, consulte los [precios de blobs en páginas de Azure](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

 ## <a name="create-and-use-private-endpoints-for-site-recovery"></a>Creación y uso de puntos de conexión privados para la recuperación del sitio

 En las secciones siguientes se describen los pasos que hay que dar para crear y usar puntos de conexión privados para la recuperación del sitio en las redes virtuales.

> [!NOTE]
> Se recomienda seguir estos pasos en el orden que se indica. Si no lo hace, es posible que no pueda usar puntos de conexión privados en el almacén y que tenga que reiniciar el proceso con un nuevo almacén.

### <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Los almacenes de Recovery Services contienen la información de replicación de las máquinas. Se usan para desencadenar las operaciones de Site Recovery. Para más información sobre cómo crear un almacén de Recovery Services en la región de Azure en la que desea realizar la conmutación por error si se produce un desastre, consulte [Creación de un almacén de Recovery Services](./azure-to-azure-tutorial-enable-replication.md#create-a-recovery-services-vault).

### <a name="enable-the-managed-identity-for-the-vault"></a>Habilitación de la identidad administrada del almacén

Las [identidades administradas](../active-directory/managed-identities-azure-resources/overview.md) permiten a los almacenes acceder a las cuentas de almacenamiento. En función de los requisitos, es posible que Site Recovery tenga que acceder a las cuentas de almacenamiento de destino y de almacenamiento en caché/registro. El acceso a la identidad administrada es necesario cuando se usa el servicio Private Link para el almacén.

1. Vaya al almacén de Recovery Services. Seleccione **Identidad** en **Configuración**:

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/enable-managed-identity-in-vault.png" alt-text="Captura de pantalla que muestra la página de configuración de la identidad.":::

1. Cambie el **Estado** a **Activado** y seleccione **Guardar**.

   Se genera un identificador de objeto. El almacén ya está registrado en Azure Active Directory.

### <a name="create-private-endpoints-for-the-recovery-services-vault"></a>Creación de puntos de conexión privados para el almacén de Recovery Services

Para proteger las máquinas de la red de origen local, necesitará un punto de conexión privado para el almacén en la red de derivación. Cree el punto de conexión privado mediante Private Link Center en Azure Portal o [Azure PowerShell](../private-link/create-private-endpoint-powershell.md).

1. En el cuadro de búsqueda de Azure Portal, busque "private link". Seleccione **Private Link** para ir a Private Link Center:

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/search-private-links.png" alt-text="Captura de pantalla que muestra la búsqueda de Private Link Center en Azure Portal.":::

1. En el panel izquierdo, seleccione **Puntos de conexión privados**. En la página **Puntos de conexión privados**, seleccione **Agregar** para empezar a crear un punto de conexión privado para el almacén:

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints.png" alt-text="Captura de pantalla que muestra cómo crear un punto de conexión privado en Private Link Center.":::

1. En la página **Crear un punto de conexión privado**, especifique los detalles necesarios para crear una conexión con un punto de conexión privado.

   1. **Conceptos básicos**. Proporcionan los detalles básicos de los puntos de conexión privados. Use la región que utilizó para la red de derivación:

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints-basic-tab.png" alt-text="Captura de pantalla que muestra la pestaña Conceptos básicos para crear un punto de conexión privado.":::

   1. **Recurso**. En esta pestaña debe especificar el recurso de plataforma como servicio para el que desea crear la conexión. En **Tipo de recurso** para la suscripción que ha seleccionado, seleccione **Microsoft.RecoveryServices/vaults**. Elija el nombre del almacén de Recovery Services en **Recurso**. Seleccione **Azure Site Recovery** en **Subrecurso de destino**.

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints-resource-tab.png" alt-text="Captura de pantalla que muestra la pestaña Recurso para realizar la vinculación a un punto de conexión privado.":::

   1. **Configuración**. En esta pestaña, especifique la red de derivación y la subred en que desea que se cree el punto de conexión privado. 

      Habilite la integración en una zona DNS privada, seleccione **Sí**.
      Elija una zona DNS existente o cree una. Si selecciona **Sí**, la zona se vincula automáticamente a la red de derivación. Esta acción también agrega los registros DNS necesarios para la resolución DNS de nuevas direcciones IP y nombres de dominio completos creados para el punto de conexión privado.

      Asegúrese de elegir crear una nueva zona DNS para cada nuevo punto de conexión privado que se conecta al mismo almacén. Si elige una zona DNS privada existente, se sobrescriben los registros CNAME anteriores. Consulte la [guía de los puntos de conexión privados](../private-link/private-endpoint-overview.md#private-endpoint-properties) antes de continuar.

      Si su entorno tiene un modelo del tipo hub-and-spoke, solo necesita un punto de conexión privado y solo una zona DNS privada para toda la instalación, ya que todas las redes virtuales ya tienen habilitado el emparejamiento entre ellas. Para obtener más información, consulte [Integración de DNS de punto de conexión privado](../private-link/private-endpoint-dns.md#virtual-network-workloads-without-custom-dns-server).

      Para crear manualmente la zona DNS privada, siga los pasos descritos en [Creación de una zona DNS privada y adición de registros DNS manualmente](#create-private-dns-zones-and-add-dns-records-manually).

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-endpoints-configuration-tab.png" alt-text="Captura de pantalla que muestra la pestaña Configuración para la configuración de un punto de conexión privado.":::

   1. **Etiquetas**. Opcionalmente, puede agregar etiquetas para el punto de conexión privado.

   1. **Revisar \+ y crear**. Al finalizar la validación, seleccione **Crear** para crear el punto de conexión privado.

Una vez creado el punto de conexión privado, se agregan cinco nombres de dominio completos (FQDN) al punto de conexión privado. Estos vínculos permiten a las máquinas de la red local acceder a todos los microservicios de Site Recovery necesarios en el contexto del almacén a través de la red de derivación. Se puede usar el mismo punto de conexión privado para la protección de cualquier máquina de Azure tanto en la red de derivación como en todas las redes emparejadas.

Para dar formato a los cinco nombres de dominio se usa este patrón:

`{Vault-ID}-asr-pod01-{type}-.{target-geo-code}.siterecovery.windowsazure.com`

### <a name="approve-private-endpoints-for-site-recovery"></a>Aprobación de puntos de conexión privados para la recuperación del sitio

Si crea el punto de conexión privado y también es el propietario del almacén de Recovery Services, el punto de conexión privado que creó anteriormente se aprueba automáticamente a los pocos minutos. De no ser así, el propietario del almacén debe aprobar el punto de conexión privado para poder usarlo. Para aprobar o rechazar una conexión de punto de conexión privado solicitada, vaya a **Conexiones de punto de conexión privado** en **Configuración** en la página del almacén de recuperación.

Puede ir al recurso de punto de conexión privado para examinar el estado de la conexión antes de continuar:

:::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/vault-private-endpoint-connections.png" alt-text="Captura de pantalla que muestra la página Conexiones de punto de conexión privado del almacén y la lista de conexiones.":::

### <a name="optional-create-private-endpoints-for-the-cache-storage-account"></a><a name="create-private-endpoints-for-the-cache-storage-account"></a>(Opcional) Creación de puntos de conexión privados para la cuenta de almacenamiento de caché

Puede usar un punto de conexión privado a Azure Storage. La creación de puntos de conexión privados para el acceso al almacenamiento es opcional para la replicación de Azure Site Recovery. Si crea un punto de conexión privado para el almacenamiento, necesita un punto de conexión privado para la cuenta de almacenamiento en caché o registro en la red virtual de derivación.

> [!NOTE]
> Los puntos de conexión privados para el almacenamiento solo se pueden crear en cuentas de almacenamiento del tipo De uso general v2. Para más información sobre los precios, consulte [Precios de blobs en páginas de Azure](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

Para crear una cuenta de almacenamiento con un punto de conexión privado, siga la [guía para crear almacenamiento privado](../private-link/create-private-endpoint-storage-portal.md#create-your-private-endpoint). Asegúrese de seleccionar **Sí** en **Integrar con la zona DNS privada**. Seleccione una zona DNS existente o cree una.

### <a name="grant-required-permissions-to-the-vault"></a>Concesión de los permisos necesarios para el almacén

En función de la configuración, es posible que necesite una o varias cuentas de almacenamiento en la región de Azure de destino. A continuación, conceda a la identidad administrada permisos para todas las cuentas de almacenamiento en caché o registro necesarias para Site Recovery. En este caso, debe crear de antemano las cuentas de almacenamiento necesarias.

Para poder habilitar la replicación de máquinas virtuales, la identidad administrada del almacén debe tener los siguientes permisos de rol, en función del tipo de cuenta de almacenamiento.

- Cuentas de almacenamiento basadas en Resource Manager (tipo Estándar):
  - [Colaborador](../role-based-access-control/built-in-roles.md#contributor)
  - [Colaborador de datos de blobs de almacenamiento](../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)
- Cuentas de almacenamiento basadas en Resource Manager (tipo Premium):
  - [Colaborador](../role-based-access-control/built-in-roles.md#contributor)
  - [Propietario de datos de blobs de almacenamiento](../role-based-access-control/built-in-roles.md#storage-blob-data-owner)
- Cuentas de almacenamiento clásicas:
  - [Colaborador de cuentas de almacenamiento clásico](../role-based-access-control/built-in-roles.md#classic-storage-account-contributor)
  - [Rol de servicio de operador de claves de cuentas de almacenamiento clásicas](../role-based-access-control/built-in-roles.md#classic-storage-account-key-operator-service-role)

En estos pasos se indica cómo agregar una asignación de roles a una cuenta de almacenamiento:

1. Vaya a la cuenta de almacenamiento. Seleccione **Control de acceso (IAM)** en el panel izquierdo.

1. En la sección **Agregar una asignación de roles**, seleccione **Agregar**:

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/storage-role-assignment.png" alt-text="Captura de pantalla que muestra la página Control de acceso (IAM) de una cuenta de almacenamiento.":::

1. En la página **Agregar una asignación de roles**, en la lista **Rol**, seleccione un rol en la lista que se encuentra al principio de esta sección. Escriba el nombre del almacén y seleccione **Guardar**.

   :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/storage-role-assignment-select-role.png" alt-text="Captura de pantalla que muestra la página Agregar una asignación de roles.":::

Después de agregar estos permisos, debe permitir el acceso a los servicios de confianza de Microsoft. Vaya a **Firewalls y redes virtuales** y seleccione **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento** en **Excepciones**.

### <a name="protect-your-virtual-machines"></a>Protección de las máquinas virtuales

Tras finalizar las tareas anteriores, continúe con la configuración de la infraestructura local. Para continuar, complete una de las siguientes tareas: 

- [Implementación de un servidor de configuración para VMware y de las máquinas físicas](./vmware-azure-deploy-configuration-server.md)
- [Configuración del entorno de Hyper-V para la replicación](./hyper-v-azure-tutorial.md#set-up-the-source-environment)

Tras completar la configuración, habilite la replicación de las máquinas de origen. No configure la infraestructura hasta que se creen los puntos de conexión privados para el almacén en la red de derivación.

### <a name="create-private-dns-zones-and-add-dns-records-manually"></a>Creación de una zona DNS privada y adición de registros DNS manualmente

Si no seleccionó la opción de integración en una zona DNS privada al crear el punto de conexión privado para el almacén, siga los pasos de esta sección.

Cree una zona DNS privada para que el proveedor de Site Recovery (en el caso de máquinas de Hyper-V) o el servidor de procesos (en el caso de VMware o máquinas físicas) puedan resolver los nombres de dominio completos privados en direcciones IP privadas.

1. Cree una zona DNS privada.

   1. Busque "zona DNS privada" en el cuadro de búsqueda **Todos los servicios** y seleccione **Zona DNS privada** en los resultados:

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/search-private-dns-zone.png" alt-text="Captura de pantalla que muestra una zona DNS privada en la página de nuevos recursos de Azure Portal.":::

   1. En la página **Zonas DNS privadas**, seleccione el botón **Agregar** para empezar a crear una zona.

   1. En la página **Crear una zona DNS privada**, escriba los detalles necesarios. Escriba **privatelink.siterecovery.windowsazure.com** como nombre de la zona DNS privada. Puede elegir cualquier grupo de recursos y cualquier suscripción.

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/create-private-dns-zone.png" alt-text="Captura de pantalla que muestra la pestaña Conceptos básicos de la página Crear una zona DNS privada.":::

   1. Vaya a la pestaña **Revisar \+ crear** para revisar y crear la zona DNS.

1. Vincule la zona DNS privada a su red virtual.

   Ahora necesita vincular la zona DNS privada que ha creado a la red de derivación.

   1. Vaya a la zona DNS privada que creó en el paso anterior y, después, a **Vínculos de la red virtual** en el panel de la izquierda. Seleccione **Agregar**.

   1. Escriba los detalles necesarios. En las listas **Suscripción** y **Red virtual**, seleccione los detalles correspondientes a la red de derivación. Deje los valores predeterminados en los demás campos.

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/add-virtual-network-link.png" alt-text="Captura de pantalla que muestra la página Agregar vínculo de red virtual.":::

1. Agregue registros DNS.

   Ahora que ha creado la zona DNS privada necesaria y el punto de conexión privado, debe agregar los registros DNS a la zona DNS.

   > [!NOTE]
   > Si utiliza una zona DNS privada personalizada, asegúrese de crear entradas similares, como se describe en el siguiente paso.

   En este paso, necesita crear entradas para cada nombre de dominio completo del punto de conexión privado en su zona DNS privada.

   1. Vaya a su zona DNS privada y, después, a la sección **Información general** del panel de la izquierda. Seleccione **Conjunto de registros** para empezar a agregar registros.

   1. En la página **Agregar conjunto de registros**, agregue una entrada para cada nombre de dominio completo y una dirección IP privada como un registro de tipo **A**. En la página **Punto de conexión privado** en **Información general**, puede obtener una lista de los nombres de dominio completos y de las direcciones IP. Como se puede ver en la siguiente captura de pantalla, el primer nombre de dominio completo del punto de conexión privado se agrega al conjunto de registros de la zona DNS privada.

      Estos nombres de dominio completos coinciden con este patrón: `{Vault-ID}-asr-pod01-{type}-.{target-geo-code}.siterecovery.windowsazure.com`

      :::image type="content" source="./media/hybrid-how-to-enable-replication-private-endpoints/add-record-set.png" alt-text="Captura de pantalla que muestra la página Agregar conjunto de registros.":::

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha habilitado los puntos de conexión privados para la replicación de máquinas virtuales, consulte estos otros artículos para obtener información adicional y relacionada:

- [Implementación de un servidor de configuración local](./vmware-azure-deploy-configuration-server.md)
- [Configuración de la recuperación ante desastres de máquinas virtuales de Hyper-V locales en Azure](./hyper-v-azure-tutorial.md)