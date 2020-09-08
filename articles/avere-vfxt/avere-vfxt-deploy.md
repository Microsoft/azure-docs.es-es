---
title: Implementación de Avere vFXT for Azure
description: Obtenga información sobre cómo usar el asistente para la implementación disponible en Azure Marketplace para implementar un clúster con Avere vFXT for Azure.
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 01/13/2020
ms.author: rohogue
ms.openlocfilehash: 31d12466186bb7f66197218fbb9675888a35fef3
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88272780"
---
# <a name="deploy-the-vfxt-cluster"></a>Implementación del clúster de vFXT

Este procedimiento explica cómo usar el asistente para la implementación disponible en Azure Marketplace. El asistente implementa automáticamente el clúster mediante una plantilla de Azure Resource Manager. Después de escribir los parámetros en el formulario y hacer clic en **Crear**, Azure completa automáticamente estas tareas:

* Crea el controlador de clúster, que es una máquina virtual básica que contiene el software necesario para implementar y administrar el clúster.
* Configura el grupo de recursos y la infraestructura de red virtual, incluida la creación de elementos si es necesario.
* Crea las máquinas virtuales del nodo de clúster y las configura como el clúster de Avere.
* En caso de que se le solicite, crea un contenedor de blobs de Azure y lo configura como un archivador principal del clúster.

Después de seguir las instrucciones de este documento, tendrá una red virtual, una subred, un controlador de clúster y un clúster de vFXT, tal como se muestra en el diagrama siguiente. Este diagrama muestra el filtro principal de Azure Blob opcional, que incluye un nuevo contenedor de almacenamiento de blobs (en una nueva cuenta de almacenamiento, que no se muestra) y un punto de conexión de servicio para Microsoft Storage dentro de la subred.

![diagrama que muestra tres rectángulos concéntricos con componentes del clúster de Avere. El rectángulo exterior se etiqueta como "Resource group" (Grupo de recursos) y contiene un hexágono con la etiqueta "Blob storage (optional)" (Almacenamiento de blobs [opcional]). El siguiente rectángulo tiene la etiqueta "Red virtual: 10.0.0.0/16" y no contiene ningún componente único. El rectángulo más interno tiene la etiqueta "Subnet:10.0.0.0/24" (Subred:10.0.0.0/24) y contiene una máquina virtual con la etiqueta "Cluster controller" (Controlador de clúster), una pila de tres máquinas virtuales con la etiqueta "vFXT nodes (vFXT cluster)" (nodos vFXT [clúster de vFXT]) y un hexágono con la etiqueta "Service endpoint" (Punto de conexión de servicio). Hay una flecha que conecta el punto de conexión de servicio (que se encuentra dentro de la subred) y el almacenamiento de blobs (que está fuera de la subred y la red virtual en el grupo de recursos). La flecha pasa los límites de la red virtual y la subred.](media/avere-vfxt-deployment.png)

Antes de usar la plantilla de creación, asegúrese de haber abordado estos requisitos previos:  

* [Suscripción nueva](avere-vfxt-prereqs.md#create-a-new-subscription)
* [Permisos de propietario de la suscripción](avere-vfxt-prereqs.md#configure-subscription-owner-permissions)
* [Cuota del clúster de vFXT](avere-vfxt-prereqs.md#quota-for-the-vfxt-cluster)
* [Punto de conexión de servicio de almacenamiento (si es necesario)](avere-vfxt-prereqs.md#create-a-storage-service-endpoint-in-your-virtual-network-if-needed): necesario para implementaciones que utilizan una red virtual existente y que crean un almacenamiento de blobs

Para más información sobre los pasos de planeación e implementación del clúster, lea [Planeamiento de un sistema de Avere vFXT](avere-vfxt-deploy-plan.md) e [Descripción general de la implementación](avere-vfxt-deploy-overview.md).

## <a name="create-the-avere-vfxt-for-azure"></a>Creación del clúster de Avere vFXT for Azure

Acceda a la plantilla de creación en Azure Portal; para ello, busque Avere y seleccione "Plantilla de Avere vFXT for Azure ARM".

![Ventana del explorador que muestra Azure Portal con la ruta "Nuevo > Marketplace > Todo". En la página Todo, el campo de búsqueda tiene el término "avere" y el segundo resultado, "Plantilla de Avere vFXT for Azure ARM", está subrayado en rojo para resaltarlo.](media/avere-vfxt-template-choose.png)

Después de leer los detalles de la página Plantilla de Avere vFXT for Azure ARM, haga clic en su botón **Crear** para comenzar.

![Azure Marketplace con la visualización de la primera página de la plantilla de implementación](media/avere-vfxt-deploy-first.png)

La plantilla se divide en cuatro pasos: dos páginas de recopilación de información más los pasos de validación y confirmación.

* En la primera página se recopila la configuración de la máquina virtual del controlador del clúster.
* En la segunda página se recopilan parámetros para crear el clúster y recursos adicionales, como subredes y almacenamiento.
* En la tercera página se resumen las elecciones y se valida la configuración.
* La cuarta página explica los términos y condiciones del software y permite iniciar el proceso de creación del clúster.

## <a name="page-one-parameters---cluster-controller-information"></a>Parámetros de la primera página: información sobre el controlador de clúster

La primera página de la plantilla de implementación se centra en el controlador de clúster.

![Primera página de la plantilla de implementación](media/avere-vfxt-deploy-1.png)

Rellene la información siguiente:

* **Nombre del controlador de clúster**: establezca el nombre de la máquina virtual del controlador de clúster.

* **Controller username** (Nombre de usuario del controlador): establezca el nombre de usuario raíz de la máquina virtual del controlador de clúster.

* **Tipo de autenticación**: elija la contraseña o la autenticación con clave pública SSH para conectarse al controlador. Se recomienda el método de clave pública SSH; lea [Creación y uso de claves SSH](https://docs.microsoft.com/azure/virtual-machines/linux/ssh-from-windows) si necesita ayuda.

* **Contraseña** o **clave pública SSH**: según el tipo de autenticación seleccionado, debe proporcionar una clave pública RSA o una contraseña en los campos siguientes. Esta credencial se usa con el nombre de usuario proporcionado anteriormente.

* **Suscripción**: seleccione la suscripción para Avere vFXT.

* **Grupo de recursos**: seleccione un grupo de recursos vacío existente para el clúster de Avere vFXT o haga clic en "Crear" y escriba un nombre para el nuevo grupo de recursos.

* **Ubicación**: seleccione la ubicación de Azure para el clúster y los recursos.

Haga clic en **Aceptar** cuando haya finalizado.

> [!NOTE]
> Si desea que el controlador de clúster tenga una dirección IP pública, cree una red virtual para el clúster en lugar de seleccionar una red existente. Esta configuración se encuentra en la segunda página.

## <a name="page-two-parameters---vfxt-cluster-information"></a>Parámetros de la segunda página: información del clúster de vFXT

La segunda página de la plantilla de implementación permite establecer el tamaño del clúster, el tipo de nodo, el tamaño de la caché y los parámetros de almacenamiento, entre otras opciones.

![Segunda página de la plantilla de implementación](media/avere-vfxt-deploy-2.png)

* **Avere vFXT cluster node count** (Número de nodos del clúster de Avere vFXT): elija el número de nodos del clúster. El valor mínimo es tres nodos y el máximo es doce.

* **Contraseña de administración del clúster**: crea la contraseña para la administración del clúster. Esta contraseña se usará con el nombre de usuario ```admin``` para iniciar sesión en el panel de control del clúster, donde puede supervisar el clúster y configurarlo.

* **Nombre del clúster de Avere vFXT**: asigne al clúster un nombre exclusivo.

* **Tamaño**: esta sección muestra el tipo de máquina virtual que se usará para los nodos del clúster. Aunque hay solo una opción recomendada, el vínculo **Cambiar tamaño** abre una tabla con detalles sobre este tipo de instancia y un vínculo a una calculadora de precios.

* **Cache size per node** (Tamaño de la caché por nodo): la caché del clúster se reparte entre los nodos del clúster, por lo que el tamaño total de la caché del clúster de Avere vFXT será el tamaño multiplicado por el número de nodos.

  Configuración recomendada: Use 4 TB por nodo para los nodos Standard_E32s_v3.

* **Virtual network** (Red virtual): defina una nueva red virtual para alojar el clúster o seleccione una red virtual existente que cumpla los requisitos previos descritos en [Planeamiento de un sistema de Avere vFXT](avere-vfxt-deploy-plan.md#subscription-resource-group-and-network-infrastructure).

  > [!NOTE]
  > Si crea una red virtual, el controlador de clúster tendrá una dirección IP pública, de forma que puede acceder a la nueva red privada. Si elige una red virtual existente, se configura el controlador de clúster sin una dirección IP pública.
  >
  > Una dirección IP visible públicamente en el controlador de clúster facilita el acceso al clúster de vFXT, pero plantea un pequeño riesgo de seguridad.
  >* Una dirección IP pública en el controlador del clúster le permite usarla como host de salto para conectarse al clúster de Avere vFXT desde fuera de la subred privada.
  >* Si no tiene una dirección IP pública en el controlador, debe usar otro host de salto, una conexión VPN o ExpressRoute para acceder al clúster. Por ejemplo, use una red virtual existente que ya tenga configurada una conexión VPN.
  >* Si crea un controlador con una dirección IP pública, debe proteger la VM de controlador con un grupo de seguridad de red. De forma predeterminada, la implementación de Avere vFXT for Azure crea un grupo de seguridad de red que restringe el acceso entrante solo al puerto 22 para los controladores con direcciones IP públicas. Puede proteger aún más el sistema al bloquear el acceso al intervalo de direcciones IP de origen, es decir, solo permitir las conexiones desde las máquinas que pretende usar para el acceso al clúster.

  La nueva red virtual también se configura con un punto de conexión de servicio de almacenamiento para Azure Blob Storage y con control de acceso de red bloqueado a solo las direcciones IP de la subred del clúster.

* **Subnet** (Subred): elija una subred o cree una.

* **Creación y uso de Blob Storage**: elija **true** para crear un contenedor de blobs de Azure y configurarlo como almacenamiento back-end para el nuevo clúster de Avere vFXT. Con esta opción también se crea una cuenta de almacenamiento dentro del grupo de recursos del clúster y un punto de conexión de servicio de almacenamiento de Microsoft dentro de la subred del clúster.
  
  Si se proporciona una red virtual existente, debe tener un punto de conexión de servicio de almacenamiento antes de crear el clúster. Para obtener más información, consulte [Planeamiento de un sistema de Avere vFXT](avere-vfxt-deploy-plan.md).

  Establezca este campo en **false** si no desea crear un contenedor. En este caso, debe asociar y configurar el almacenamiento después de crear el clúster. Puede encontrar las instrucciones en [Configurar el almacenamiento](avere-vfxt-add-storage.md).

* **(Nueva) Cuenta de almacenamiento**: si crea un contenedor de blobs de Azure, escriba el nombre de la nueva cuenta de almacenamiento.

## <a name="validation-and-purchase"></a>Compra y validación

En la tercera página se resume la configuración y se validan los parámetros. Una vez finalizada la validación, compruebe el resumen y haga clic en el botón **Aceptar**.

> [!TIP]
> Para guardar la configuración de creación de este clúster, haga clic en el vínculo **Descargar plantilla y parámetros** junto al botón **Aceptar**. Esta información puede ser útil si necesita crear un clúster parecido más adelante, por ejemplo, para crear un clúster de reemplazo en un escenario de recuperación ante desastres. (Lea la [guía de recuperación ante desastres](disaster-recovery.md) para más información).

![Tercera página de la plantilla de implementación: validación](media/avere-vfxt-deploy-3.png)

En la página cuatro se proporcionan las condiciones de uso y los vínculos a la información de privacidad y precios.

Especifique la información de contacto que falte y haga clic en el botón **Crear** para aceptar los términos y crear el clúster de Avere vFXT for Azure.

![Cuarta página de la plantilla de implementación: términos y condiciones y botón Crear](media/avere-vfxt-deploy-4.png)

La implementación del clúster tarda entre 15 y 20 minutos.

## <a name="gather-template-output"></a>Recopilación de la salida de la plantilla

Cuando la plantilla de Avere vFXT termina de crear el clúster, genera información importante sobre el nuevo clúster.

> [!TIP]
> Asegúrese de copiar la **dirección IP de administración** de la salida de la plantilla. Necesita esta dirección para administrar el clúster.

Para encontrar la información:

1. Vaya al grupo de recursos del controlador del clúster.

1. En el lado izquierdo, haga clic en **Implementaciones** y, luego, en **microsoft-avere.vfxt-template**.

   ![Página del portal del grupo de recursos con Implementaciones seleccionada a la izquierda y microsoft-avere.vfxt-template que aparece en una tabla en el Nombre de la implementación](media/avere-vfxt-outputs-deployments.png)

1. En el lado izquierdo, haga clic en **Salidas**. Copie los valores en cada uno de los campos.

   ![página de resultados en la que se muestran los valores SSHSTRING, RESOURCE_GROUP, LOCATION, NETWORK_RESOURCE_GROUP, NETWORK, SUBNET, SUBNET_ID, VSERVER_IPs y MGMT_IP en los campos situados a la derecha de las etiquetas](media/avere-vfxt-outputs-values.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que el clúster se está ejecutando y conoce su dirección IP de administración, [conéctese a la herramienta de configuración del clúster](avere-vfxt-cluster-gui.md).

Use la interfaz de configuración para personalizar el clúster, incluidas las siguientes tareas de configuración:

* [Habilitar el soporte técnico](avere-vfxt-enable-support.md)
* [Agregar almacenamiento](avere-vfxt-add-storage.md) (si es necesario)
