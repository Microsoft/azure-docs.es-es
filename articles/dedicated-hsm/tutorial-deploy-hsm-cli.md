---
title: 'Tutorial sobre la implementación en una red virtual existente con la CLI de Azure: Azure Dedicated HSM | Microsoft Docs'
description: Tutorial que muestra cómo implementar un HSM dedicado con la CLI en una red virtual existente
services: dedicated-hsm
documentationcenter: na
author: msmbaldwin
manager: rkarlin
editor: ''
ms.service: key-vault
ms.topic: tutorial
ms.custom: mvc, seodec18, devx-track-azurecli
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/11/2019
ms.author: mbaldwin
ms.openlocfilehash: 02b15f88e05abc918100f5e6b1ad1cdd60954a9d
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87486993"
---
# <a name="tutorial-deploying-hsms-into-an-existing-virtual-network-using-cli"></a>Tutorial: Implementación de HSM en una red virtual existente con la CLI

Azure Dedicated HSM proporciona un dispositivo físico para uso exclusivo del cliente, con un control administrativo completo y una responsabilidad en la administración total. El uso de dispositivos físicos crea la necesidad de que Microsoft controle la asignación de dispositivos que garantice que la capacidad se administra de forma eficaz. Como resultado, en una suscripción de Azure, el servicio Dedicated HSM no será normalmente visible para el aprovisionamiento de recursos. Los clientes de Azure que requieran acceso al servicio Dedicated HSM, primero deben ponerse en contacto con el ejecutivo de cuentas de Microsoft para solicitar el registro en este servicio. Solo una vez que este proceso se complete correctamente el aprovisionamiento será posible. 

Este tutorial muestra un proceso de aprovisionamiento típico en el que:

- Un cliente ya tiene una red virtual
- Tienen una máquina virtual
- Tienen que agregar recursos HSM en ese entorno ya existente.

Una arquitectura de implementación típica, con alta disponibilidad y para varias regiones puede que tenga este aspecto:

![implementación de varias regiones](media/tutorial-deploy-hsm-cli/high-availability-architecture.png)

Este tutorial se centra en dos HSM y en la puerta de enlace de ExpressRoute necesaria (consulte la Subred 1 anterior) que se integra en una red virtual ya existente (consulte la red virtual 1 anterior).  Todos los demás recursos son recursos estándar de Azure. Se puede utilizar el mismo proceso de integración para HSM en la subred 4 de la red virtual 3 anterior.

## <a name="prerequisites"></a>Prerrequisitos

Azure Dedicated HSM no está disponible actualmente en Azure Portal. Toda interacción con el servicio se realizará a través de la línea de comandos o mediante PowerShell. Este tutorial usará la interfaz de la línea de comandos (CLI) en Azure Cloud Shell. Si no está familiarizado con la CLI de Azure, siga las instrucciones de inicio que se encuentran aquí: [Introducción a la CLI de Azure 2.0](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest).

Se supone que:

- Ha completado el proceso de registro de Azure Dedicated HSM
- Se le ha autorizado el uso del servicio. En caso contrario, póngase en contacto con su representante de cuenta Microsoft para más información.
- Ha creado un grupo de recursos para estos recursos y los nuevos recursos que se implementen en este tutorial se unirán más tarde a ese grupo.
- Ya ha creado la red virtual, la subred y las máquinas virtuales necesarias según el diagrama anterior y ahora desea integrar 2 HSM en esa implementación.

Todas las instrucciones que aparecen a continuación presuponen que ya ha ido a Azure Portal y ha abierto Cloud Shell (seleccione "\>\_" en la parte superior derecha del portal).

## <a name="provisioning-a-dedicated-hsm"></a>Aprovisionamiento de un HSM dedicado

El aprovisionamiento de HSM y su integración en una red virtual existente a través de la puerta de enlace de ExpressRoute se validará mediante SSH. Esta validación ayuda a garantizar la accesibilidad y disponibilidad básicas del dispositivo HSM para cualquier actividad de configuración adicional. Los siguientes comandos usarán una plantilla de Azure Resource Manager para crear los recursos HSM y los recursos de red asociados.

### <a name="validating-feature-registration"></a>Validación del registro de características

Como ya se ha mencionado anteriormente, cualquier actividad de aprovisionamiento requiere que se registre el servicio Dedicated HSM para su suscripción. Para validar, ejecute los siguientes comandos en Cloud Shell de Azure Portal.

```azurecli
az feature show \
   --namespace Microsoft.HardwareSecurityModules \
   --name AzureDedicatedHSM
```

Los comandos deben devolver un estado "Registrado" (como se indica a continuación). Si los comandos no devuelven un estado "Registrado", tendrá que registrarse para este servicio. Para ello, póngase en contacto con su representante de cuenta de Microsoft.

![estado de la suscripción](media/tutorial-deploy-hsm-cli/subscription-status.png)

### <a name="creating-hsm-resources"></a>Creación de recursos HSM

Un dispositivo HSM se aprovisiona en la red virtual de un cliente, por lo que se necesita una red virtual y una subred. Se requiere una dependencia para que el HSM habilite la comunicación entre la red virtual y el dispositivo físico en una puerta de enlace de ExpressRoute y, finalmente, también se requiere una red virtual para acceder al dispositivo HSM mediante el software cliente Gemalto. Estos recursos se han recopilado en un archivo de plantilla, junto con el correspondiente archivo de parámetros, para facilitar su uso. Para obtener los archivos póngase en contacto con Microsoft directamente en HSMrequest@Microsoft.com.

Una vez que tenga los archivos, debe editar el archivo de parámetros para insertar los nombres preferidos de los recursos. Edite las líneas con "value": "".

- `namingInfix` Prefijo para los nombres de los recursos HSM
- `ExistingVirtualNetworkName` Nombre de la red virtual que se usa para los HSM
- `DedicatedHsmResourceName1` Nombre del recurso HSM en la marca del centro de datos 1
- `DedicatedHsmResourceName2` Nombre del recurso HSM en la marca del centro de datos 2
- `hsmSubnetRange` Intervalo de direcciones IP de la subred para los HSM
- `ERSubnetRange` Intervalo de direcciones IP de la subred para la puerta de enlace de la red virtual

A continuación puede ver un ejemplo de estos cambios:

```json
{
"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "namingInfix": {
      "value": "MyHSM"
    },
    "ExistingVirtualNetworkName": {
      "value": "MyHSM-vnet"
    },
    "DedicatedHsmResourceName1": {
      "value": "HSM1"
    },
    "DedicatedHsmResourceName2": {
      "value": "HSM2"
    },
    "hsmSubnetRange": {
      "value": "10.0.2.0/24"
    },
    "ERSubnetRange": {
      "value": "10.0.255.0/26"
    },
  }
}
```

El archivo de plantilla de Azure Resource Manager asociado creará 6 recursos con esta información:

- Una subred para los HSM en la red virtual especificada
- Una subred para la puerta de enlace de red virtual
- Una puerta de enlace de red virtual que se conecta la red virtual a los dispositivos HSM
- Una dirección IP pública para la puerta de enlace
- Un HSM en la marca 1
- Un HSM en la marca 2

Una vez que se establecen los valores de parámetro, los archivos tienen que cargarse al recurso compartido de archivos del Cloud Shell de Azure Portal para su uso. En Azure Portal, haga clic en el símbolo de Cloud Shell "\>\_" situado en la parte superior derecha, lo cual convertirá la parte inferior de la pantalla en un entorno de comandos. Las opciones para esto son BASH y PowerShell y debe seleccionar BASH si no está ya configurado.

El shell de comandos tiene una opción de carga y descarga en la barra de herramientas y debe seleccionar esta opción para cargar los archivos de plantilla y de parámetros en el recurso compartido de archivos:

![recurso compartido de archivos](media/tutorial-deploy-hsm-cli/file-share.png)

Una vez cargados los archivos, ya está listo para crear recursos. Antes de crear nuevos recursos HSM hay algunos requisitos previos que debe garantizar. Debe disponer de una red virtual con intervalos de subred para procesos, HSM y una puerta de enlace. Los siguientes comandos sirven como ejemplo de la creación de esa red virtual.

```azurecli
az network vnet create \
  --name myHSM-vnet \
  --resource-group myRG \
  --address-prefix 10.2.0.0/16
  --subnet-name compute
  --subnet-prefix 10.2.0.0/24
```

```azurecli
az network vnet create \
  --vnet-name myHSM-vnet \
  --resource-group myRG \
  --name hsmsubnet \
  --address-prefixes 10.2.1.0/24 \
  --delegations Microsoft.HardwareSecurityModules/dedicatedHSMs
```

```azurecli
az network vnet subnet create \
  --vnet-name myHSM-vnet \
  --resource-group myRG \
  --name GatewaySubnet \
  --address-prefixes 10.2.255.0/26
```

>[!NOTE]
>La configuración más importante a tener en cuenta para la red virtual, es que la subred del dispositivo HSM debe tener las delegaciones establecidas en "Microsoft.HardwareSecurityModules/dedicatedHSMs".  El aprovisionamiento de HSM no funcionará si no establece esta opción.

Una vez que se cumplen todos los requisitos previos, ejecute el siguiente comando para usar la plantilla de Azure Resource Manager asegurándose de que se han actualizado los valores con sus nombres únicos (al menos el nombre del grupo de recursos):

```azurecli
az group deployment create \
   --resource-group myRG  \
   --template-file ./Deploy-2HSM-toVNET-Template.json \
   --parameters ./Deploy-2HSM-toVNET-Params.json \
   --name HSMdeploy \
   --verbose
```

Esta implementación tardará aproximadamente 25 o 30 minutos en completarse empleando la mayor parte de ese tiempo en los dispositivos HSM

![estado de aprovisionamiento](media/tutorial-deploy-hsm-cli/progress-status.png)

Si la implementación se completa correctamente se mostrará "provisioningState": "Correcto". Puede conectarse a la máquina virtual existente y usar SSH para garantizar la disponibilidad del dispositivo HSM.

## <a name="verifying-the-deployment"></a>Comprobación de la implementación

Para comprobar que los dispositivos se hayan aprovisionado y ver los atributos del dispositivo, ejecute el siguiente conjunto de comandos. Asegúrese de que el grupo de recursos se ha establecido adecuadamente y de que el nombre del recurso es exactamente el mismo que aparece en el archivo de parámetros.

```azurecli
subid=$(az account show --query id --output tsv)
az resource show \
   --ids /subscriptions/$subid/resourceGroups/myRG/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/HSM1
az resource show \
   --ids /subscriptions/$subid/resourceGroups/myRG/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/HSM2
```

![salida de aprovisionamiento](media/tutorial-deploy-hsm-cli/progress-status2.png)

Ahora también podrá ver los recursos mediante [Azure Resource Explorer](https://resources.azure.com/).   Una vez que esté en el explorador, expanda "suscripciones" a la izquierda, expanda la suscripción específica de Dedicated HSM, expanda "grupos de recursos", expanda el grupo de recursos que usó y, por último, seleccione el elemento "recursos".

## <a name="testing-the-deployment"></a>Prueba de la implementación

Para probar la implementación es necesario conectarse a una máquina virtual que pueda acceder a los HSM y, posteriormente, conectarse directamente al dispositivo HSM. Estas acciones confirmarán que el HSM está disponible.
La herramienta SSH se usa para conectarse a la máquina virtual. El comando será parecido al siguiente, pero con el nombre del administrador y el nombre DNS que especificó en el parámetro.

`ssh adminuser@hsmlinuxvm.westus.cloudapp.azure.com`

La dirección IP de la máquina virtual también se puede usar en lugar del nombre DNS del comando anterior. Si el comando es correcto, se le solicitará una contraseña y deberá especificarla. Una vez que ha iniciado sesión en la máquina virtual, puede iniciar sesión en el dispositivo HSM con la dirección IP privada que se encuentra en el portal para el recurso de interfaz de red asociado con el HSM.

![lista de componentes](media/tutorial-deploy-hsm-cli/resources.png)

>[!NOTE]
>Observe que la casilla "Mostrar tipos ocultos" muestra los recursos HSM cuando se selecciona.

En la captura de pantalla anterior, si hace clic en "HSM1_HSMnic" o "HSM2_HSMnic" se debería mostrar la dirección IP privada adecuada. En caso contrario, el comando `az resource show` que se ha usado anteriormente es una manera de identificar la dirección IP correcta. 

Cuando tenga la dirección IP correcta, ejecute el siguiente comando sustituyendo esa dirección:

`ssh tenantadmin@10.0.2.4`

Si se ejecuta correctamente se le pedirá una contraseña. La contraseña predeterminada es PASSWORD y el HSM le pedirá que cambie la contraseña, así que establezca una contraseña segura y use el mecanismo que prefiera su organización para almacenarla e impedir su pérdida.

>[!IMPORTANT]
>Si pierde esta contraseña, tendrá que restablecer el HSM lo cual implica la pérdida de las claves.

Cuando esté conectado al dispositivo HSM mediante SSH, ejecute el siguiente comando para asegurarse de que el HSM está operativo.

`hsm show`

La salida será similar a la de la imagen que se muestra a continuación:

![lista de componentes](media/tutorial-deploy-hsm-cli/hsm-show-output.png)

Hasta este momento ha asignado todos los recursos necesarios para una implementación de alta disponibilidad, con dos HSM, y ha validado el acceso y el estado operativo. Ya no hay ninguna configuración o prueba que implique más tareas con el dispositivo HSM en sí. Para esto, debe seguir las instrucciones que aparecen en la guía de administración de Gemalto Luna Network HSM 7, capítulo 7, para inicializar el HSM y crear particiones. Toda la documentación y el software estarán disponibles directamente desde Gemalto para su descarga una vez que se haya registrado en el portal de atención al cliente de Gemalto y haya obtenido un identificador de cliente. Descargue la versión 7.2 del software cliente para obtener todos los componentes necesarios.

## <a name="delete-or-clean-up-resources"></a>Eliminación y limpieza de los recursos

Si ya ha terminado con el dispositivo HSM, puede eliminarlo como un recurso y devolverlo al grupo disponible. La preocupación más obvia a la hora de hacer esto es la información confidencial del cliente que se encuentra en el dispositivo. La mejor manera de "poner a cero" un dispositivo es escribir la contraseña de administrador de HSM equivocada 3 veces (nota: no el administrador de la aplicación, el administrador de HSM real). Como medida de seguridad para proteger el material clave, el dispositivo no se puede eliminar como recurso de Azure hasta que se encuentre en estado "a cero".

> [!NOTE]
> Si tiene algún problema con la configuración de cualquier dispositivo de Gemalto, debe ponerse en contacto con el departamento de [asistencia al cliente de Gemalto](https://safenet.gemalto.com/technical-support/).


Si ya ha terminado con todos los recursos de este grupo de recursos, puede quitarlos con el siguiente comando:

```azurecli
az group deployment delete \
   --resource-group myRG \
   --name HSMdeploy \
   --verbose

```

## <a name="next-steps"></a>Pasos siguientes

Después de completar los pasos del tutorial, se habrán aprovisionado recursos de HSM dedicados y tendrá una red virtual con los dispositivos HSM necesarios y los componentes de red adicionales para habilitar la comunicación con el HSM.  Ahora ya puede complementar esta implementación con más recursos según sea necesario mediante la arquitectura de implementación que prefiera. Para más información sobre cómo ayudar a planear la implementación, consulte la documentación sobre conceptos.
Se recomienda un diseño con dos HSM en una región primaria que controle la disponibilidad en el nivel de bastidor y dos HSM en una región secundaria que se encarguen de la disponibilidad regional. El archivo de plantilla que ha utilizado en este tutorial se puede usar fácilmente como punto de partida para una implementación de dos HSM, pero debe modificar sus parámetros para cumplir los requisitos.

* [Alta disponibilidad](high-availability.md)
* [Seguridad física](physical-security.md)
* [Redes](networking.md)
* [Compatibilidad](supportability.md)
* [Supervisión](monitoring.md)
