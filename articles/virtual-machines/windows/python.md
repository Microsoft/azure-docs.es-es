---
title: Creación y administración de una máquina virtual Windows en Azure con Python
description: Aprenda a usar Python para crear y administrar máquinas virtuales Windows en Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure
ms.topic: how-to
ms.date: 06/22/2017
ms.author: cynthn
ms.custom: devx-track-python
ms.openlocfilehash: 1e2558074275f9b97bd4a2fd9a3d111c5de02948
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87853422"
---
# <a name="create-and-manage-windows-vms-in-azure-using-python"></a>Creación y administración de máquinas virtuales Windows en Azure con Python

Las [máquinas virtuales de Azure](overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) necesitan varios recursos de Azure compatibles. En este artículo se trata la creación, la administración y la eliminación de recursos de máquina virtual con Python. Aprenderá a:

> [!div class="checklist"]
> * Creación de un proyecto de Visual Studio
> * Instalar paquetes
> * Crear credenciales
> * Crear recursos
> * Realizar tareas de administración
> * Eliminar recursos
> * Ejecución de la aplicación

Tardará unos 20 minutos en realizar estos pasos.

## <a name="create-a-visual-studio-project"></a>Creación de un proyecto de Visual Studio

1. Si aún no lo ha hecho, instale [Visual Studio](/visualstudio/install/install-visual-studio). Seleccione **Desarrollo de Python** en la página Cargas de trabajo y haga clic en **Instalar**. En el resumen, verá que **Python 3 de 64 bits (3.6.0)** se selecciona automáticamente. Si ya ha instalado Visual Studio, puede agregar la carga de trabajo de Python con el selector de Visual Studio.
2. Después de instalar e iniciar Visual Studio, haga clic en **Archivo** > **Nuevo** > **Proyecto**.
3. Haga clic en **Plantillas** > **Python** > **Python Application** (Aplicación de Python), escriba *myPythonProject* en el nombre del proyecto, seleccione la su ubicación y haga clic en **Aceptar**.

## <a name="install-packages"></a>Instalar paquetes

1. En el Explorador de soluciones, bajo *myPythonProject*, haga clic con el botón derecho en **Python Environments** (Entornos de Python) y seleccione **Add virtual environment** (Agregar entorno virtual).
2. En la pantalla Add virtual environment (Agregar entorno virtual), acepte el nombre predeterminado de *env*, asegúrese de que *Python 3.6 (64 bits)* está seleccionada para el intérprete de base y haga clic en **Crear**.
3. Haga clic con el botón derecho en el entorno de *env* que creó, haga clic en **Install Python Package** (Instalar paquete de Python), escriba *azure* en el cuadro de búsqueda y presione Entrar.

Debería ver en las ventanas de salida que se han instalado correctamente los paquetes de azure. 

## <a name="create-credentials"></a>Crear credenciales

Antes de empezar este paso, asegúrese de que tiene una [entidad de servicio de Active Directory](../../active-directory/develop/howto-create-service-principal-portal.md). También debe registrar el identificador de aplicación, la clave de autenticación y el identificador del inquilino que necesitará en un paso posterior.

1. Abra el archivo *myPythonProject.py* que se creó y agregue este código para habilitar la ejecución de la aplicación:

    ```python
    if __name__ == "__main__":
    ```

2. Para importar el código necesario, agregue estas instrucciones a la parte superior del archivo .py:

    ```python
    from azure.common.credentials import ServicePrincipalCredentials
    from azure.mgmt.resource import ResourceManagementClient
    from azure.mgmt.compute import ComputeManagementClient
    from azure.mgmt.network import NetworkManagementClient
    from azure.mgmt.compute.models import DiskCreateOption
    ```

3. A continuación, en el archivo .py, agregue variables después de las instrucciones de importación para especificar los valores comunes que se usan en el código:
   
    ```
    SUBSCRIPTION_ID = 'subscription-id'
    GROUP_NAME = 'myResourceGroup'
    LOCATION = 'westus'
    VM_NAME = 'myVM'
    ```

    Reemplace **subscription-id** por el identificador de la suscripción.

4. Para crear las credenciales de Active Directory que necesita para realizar solicitudes, agregue esta función después de las variables en el archivo .py:

    ```python
    def get_credentials():
        credentials = ServicePrincipalCredentials(
            client_id = 'application-id',
            secret = 'authentication-key',
            tenant = 'tenant-id'
        )

        return credentials
    ```

    Reemplace **application-id**, **authentication-key** y **tenant-id** por los valores que recopiló anteriormente al crear la entidad de servicio de Azure Active Directory.

5. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    credentials = get_credentials()
    ```

## <a name="create-resources"></a>Crear recursos
 
### <a name="initialize-management-clients"></a>Inicialización de los clientes de administración

Se necesitan clientes de administración para crear y administrar los recursos con el SDK de Python en Azure. Para crear los clientes de administración, agregue este código bajo la instrucción **if** al final del archivo .py:

```python
resource_group_client = ResourceManagementClient(
    credentials,
    SUBSCRIPTION_ID
)
network_client = NetworkManagementClient(
    credentials,
    SUBSCRIPTION_ID
)
compute_client = ComputeManagementClient(
    credentials,
    SUBSCRIPTION_ID
)
```

### <a name="create-the-vm-and-supporting-resources"></a>Creación de la máquina virtual y los recursos compatibles

Todos los recursos deben encontrarse en un [grupo de recursos](../../azure-resource-manager/management/overview.md).

1. Para crear un grupo de recursos, agregue esta función después de las variables en el archivo .py:

    ```python
    def create_resource_group(resource_group_client):
        resource_group_params = { 'location':LOCATION }
        resource_group_result = resource_group_client.resource_groups.create_or_update(
            GROUP_NAME, 
            resource_group_params
        )
    ```

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    create_resource_group(resource_group_client)
    input('Resource group created. Press enter to continue...')
    ```

Los [conjuntos de disponibilidad](tutorial-availability-sets.md) facilitan el mantenimiento de las máquinas virtuales que utiliza la aplicación.

1. Para crear un conjunto de disponibilidad, agregue esta función después de las variables en el archivo .py:
   
    ```python
    def create_availability_set(compute_client):
        avset_params = {
            'location': LOCATION,
            'sku': { 'name': 'Aligned' },
            'platform_fault_domain_count': 3
        }
        availability_set_result = compute_client.availability_sets.create_or_update(
            GROUP_NAME,
            'myAVSet',
            avset_params
        )
    ```

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    create_availability_set(compute_client)
    print("------------------------------------------------------")
    input('Availability set created. Press enter to continue...')
    ```

Se necesita una [dirección IP pública](../../virtual-network/public-ip-addresses.md) para la comunicación con la máquina virtual.

1. Para crear una dirección IP pública para la máquina virtual, agregue esta función después de las variables en el archivo .py:

    ```python
    def create_public_ip_address(network_client):
        public_ip_addess_params = {
            'location': LOCATION,
            'public_ip_allocation_method': 'Dynamic'
        }
        creation_result = network_client.public_ip_addresses.create_or_update(
            GROUP_NAME,
            'myIPAddress',
            public_ip_addess_params
        )

        return creation_result.result()
    ```

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    creation_result = create_public_ip_address(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

Debe haber una máquina virtual en una subred de una [red virtual](../../virtual-network/virtual-networks-overview.md).

1. Para crear una red virtual, agregue esta función después de las variables en el archivo .py:

    ```python
    def create_vnet(network_client):
        vnet_params = {
            'location': LOCATION,
            'address_space': {
                'address_prefixes': ['10.0.0.0/16']
            }
        }
        creation_result = network_client.virtual_networks.create_or_update(
            GROUP_NAME,
            'myVNet',
            vnet_params
        )
        return creation_result.result()
    ```

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:
   
    ```python
    creation_result = create_vnet(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

3. Para agregar una subred a la red virtual, agregue esta función después de las variables en el archivo .py:
    
    ```python
    def create_subnet(network_client):
        subnet_params = {
            'address_prefix': '10.0.0.0/24'
        }
        creation_result = network_client.subnets.create_or_update(
            GROUP_NAME,
            'myVNet',
            'mySubnet',
            subnet_params
        )

        return creation_result.result()
    ```
        
4. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:
   
    ```python
    creation_result = create_subnet(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

Una máquina virtual requiere una interfaz de red para comunicarse en la red virtual que acaba de crear.

1. Para crear una interfaz de red, agregue esta función después de las variables en el archivo .py:

    ```python
    def create_nic(network_client):
        subnet_info = network_client.subnets.get(
            GROUP_NAME, 
            'myVNet', 
            'mySubnet'
        )
        publicIPAddress = network_client.public_ip_addresses.get(
            GROUP_NAME,
            'myIPAddress'
        )
        nic_params = {
            'location': LOCATION,
            'ip_configurations': [{
                'name': 'myIPConfig',
                'public_ip_address': publicIPAddress,
                'subnet': {
                    'id': subnet_info.id
                }
            }]
        }
        creation_result = network_client.network_interfaces.create_or_update(
            GROUP_NAME,
            'myNic',
            nic_params
        )

        return creation_result.result()
    ```

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    creation_result = create_nic(network_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

Ahora que ha creado todos los recursos auxiliares, puede crear una máquina virtual.

1. Para crear la máquina virtual, agregue esta función después de las variables en el archivo .py:
   
    ```python
    def create_vm(network_client, compute_client):  
        nic = network_client.network_interfaces.get(
            GROUP_NAME, 
            'myNic'
        )
        avset = compute_client.availability_sets.get(
            GROUP_NAME,
            'myAVSet'
        )
        vm_parameters = {
            'location': LOCATION,
            'os_profile': {
                'computer_name': VM_NAME,
                'admin_username': 'azureuser',
                'admin_password': 'Azure12345678'
            },
            'hardware_profile': {
                'vm_size': 'Standard_DS1'
            },
            'storage_profile': {
                'image_reference': {
                    'publisher': 'MicrosoftWindowsServer',
                    'offer': 'WindowsServer',
                    'sku': '2012-R2-Datacenter',
                    'version': 'latest'
                }
            },
            'network_profile': {
                'network_interfaces': [{
                    'id': nic.id
                }]
            },
            'availability_set': {
                'id': avset.id
            }
        }
        creation_result = compute_client.virtual_machines.create_or_update(
            GROUP_NAME, 
            VM_NAME, 
            vm_parameters
        )
    
        return creation_result.result()
    ```

    > [!NOTE]
    > En este tutorial se crea una máquina virtual donde se ejecuta una versión del sistema operativo Windows Server. Para más información sobre cómo seleccionar otras imágenes, consulte [Seleccione y navegue por imágenes de máquina virtual de Azure con PowerShell y la CLI de Azure](../linux/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
    > 
    > 

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    creation_result = create_vm(network_client, compute_client)
    print("------------------------------------------------------")
    print(creation_result)
    input('Press enter to continue...')
    ```

## <a name="perform-management-tasks"></a>Realizar tareas de administración

Durante el ciclo de vida de una máquina virtual, puede ejecutar tareas de administración como iniciar, detener o eliminar una máquina virtual. Además, puede crear código para automatizar tareas repetitivas o complejas.

### <a name="get-information-about-the-vm"></a>Obtención de información acerca de la máquina virtual

1. Para obtener información acerca de la máquina virtual, agregue esta función después de las variables en el archivo .py:

    ```python
    def get_vm(compute_client):
        vm = compute_client.virtual_machines.get(GROUP_NAME, VM_NAME, expand='instanceView')
        print("hardwareProfile")
        print("   vmSize: ", vm.hardware_profile.vm_size)
        print("\nstorageProfile")
        print("  imageReference")
        print("    publisher: ", vm.storage_profile.image_reference.publisher)
        print("    offer: ", vm.storage_profile.image_reference.offer)
        print("    sku: ", vm.storage_profile.image_reference.sku)
        print("    version: ", vm.storage_profile.image_reference.version)
        print("  osDisk")
        print("    osType: ", vm.storage_profile.os_disk.os_type.value)
        print("    name: ", vm.storage_profile.os_disk.name)
        print("    createOption: ", vm.storage_profile.os_disk.create_option.value)
        print("    caching: ", vm.storage_profile.os_disk.caching.value)
        print("\nosProfile")
        print("  computerName: ", vm.os_profile.computer_name)
        print("  adminUsername: ", vm.os_profile.admin_username)
        print("  provisionVMAgent: {0}".format(vm.os_profile.windows_configuration.provision_vm_agent))
        print("  enableAutomaticUpdates: {0}".format(vm.os_profile.windows_configuration.enable_automatic_updates))
        print("\nnetworkProfile")
        for nic in vm.network_profile.network_interfaces:
            print("  networkInterface id: ", nic.id)
        print("\nvmAgent")
        print("  vmAgentVersion", vm.instance_view.vm_agent.vm_agent_version)
        print("    statuses")
        for stat in vm_result.instance_view.vm_agent.statuses:
            print("    code: ", stat.code)
            print("    displayStatus: ", stat.display_status)
            print("    message: ", stat.message)
            print("    time: ", stat.time)
        print("\ndisks");
        for disk in vm.instance_view.disks:
            print("  name: ", disk.name)
            print("  statuses")
            for stat in disk.statuses:
                print("    code: ", stat.code)
                print("    displayStatus: ", stat.display_status)
                print("    time: ", stat.time)
        print("\nVM general status")
        print("  provisioningStatus: ", vm.provisioning_state)
        print("  id: ", vm.id)
        print("  name: ", vm.name)
        print("  type: ", vm.type)
        print("  location: ", vm.location)
        print("\nVM instance status")
        for stat in vm.instance_view.statuses:
            print("  code: ", stat.code)
            print("  displayStatus: ", stat.display_status)
    ```
2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    get_vm(compute_client)
    print("------------------------------------------------------")
    input('Press enter to continue...')
    ```

### <a name="stop-the-vm"></a>Parada de la máquina virtual

Puede detener una máquina virtual y mantener toda su configuración, pero se le seguirá cobrando, o puede detener una máquina virtual y desasignarla. Cuando se desasigna una máquina virtual, todos los recursos asociados a ella también se desasignan y se le deja de cobrar por ellos.

1. Para detener la máquina virtual sin desasignarla, agregue esta función después de las variables en el archivo .py:

    ```python
    def stop_vm(compute_client):
        compute_client.virtual_machines.power_off(GROUP_NAME, VM_NAME)
    ```

    Si desea desasignar la máquina virtual, cambie la llamada de power_off por este código:

    ```python
    compute_client.virtual_machines.deallocate(GROUP_NAME, VM_NAME)
    ```

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    stop_vm(compute_client)
    input('Press enter to continue...')
    ```

### <a name="start-the-vm"></a>Inicio de la máquina virtual

1. Para iniciar la máquina virtual, agregue esta función después de las variables en el archivo .py:

    ```python
    def start_vm(compute_client):
        compute_client.virtual_machines.start(GROUP_NAME, VM_NAME)
    ```

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    start_vm(compute_client)
    input('Press enter to continue...')
    ```

### <a name="resize-the-vm"></a>Cambio de tamaño de la máquina virtual

Para decidir un tamaño de máquina virtual, se deben considerar muchos aspectos de la implementación. Para más información, consulte el artículo sobre los [tamaños de máquina virtual](../sizes.md).

1. Para cambiar el tamaño de la máquina virtual, agregue esta función después de las variables en el archivo .py:

    ```python
    def update_vm(compute_client):
        vm = compute_client.virtual_machines.get(GROUP_NAME, VM_NAME)
        vm.hardware_profile.vm_size = 'Standard_DS3'
        update_result = compute_client.virtual_machines.create_or_update(
            GROUP_NAME, 
            VM_NAME, 
            vm
        )

    return update_result.result()
    ```

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    update_result = update_vm(compute_client)
    print("------------------------------------------------------")
    print(update_result)
    input('Press enter to continue...')
    ```

### <a name="add-a-data-disk-to-the-vm"></a>Incorporación de un disco de datos a la máquina virtual

Las máquinas virtuales pueden tener uno o más [discos de datos](managed-disks-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) que se almacenen en discos duros virtuales.

1. Para agregar un disco de datos a la máquina virtual, agregue esta función después de las variables en el archivo .py: 

    ```python
    def add_datadisk(compute_client):
        disk_creation = compute_client.disks.create_or_update(
            GROUP_NAME,
            'myDataDisk1',
            {
                'location': LOCATION,
                'disk_size_gb': 1,
                'creation_data': {
                    'create_option': DiskCreateOption.empty
                }
            }
        )
        data_disk = disk_creation.result()
        vm = compute_client.virtual_machines.get(GROUP_NAME, VM_NAME)
        add_result = vm.storage_profile.data_disks.append({
            'lun': 1,
            'name': 'myDataDisk1',
            'create_option': DiskCreateOption.attach,
            'managed_disk': {
                'id': data_disk.id
            }
        })
        add_result = compute_client.virtual_machines.create_or_update(
            GROUP_NAME,
            VM_NAME,
            vm)

        return add_result.result()
    ```

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:

    ```python
    add_result = add_datadisk(compute_client)
    print("------------------------------------------------------")
    print(add_result)
    input('Press enter to continue...')
    ```

## <a name="delete-resources"></a>Eliminar recursos

Dado que se le cobrará por los recursos utilizados en Azure, siempre es conveniente eliminar los recursos que ya no sean necesarios. Si quiere eliminar las máquinas virtuales y todos los recursos auxiliares, lo único que tiene que hacer es eliminar el grupo de recursos.

1. Para eliminar un grupo de recursos y todos los recursos, agregue esta función después de las variables en el archivo .py:
   
    ```python
    def delete_resources(resource_group_client):
        resource_group_client.resource_groups.delete(GROUP_NAME)
    ```

2. Para llamar a la función que agregó anteriormente, agregue este código bajo la instrucción **if** al final del archivo .py:
   
    ```python
    delete_resources(resource_group_client)
    ```

3. Guarde *myPythonProject.py*.

## <a name="run-the-application"></a>Ejecución de la aplicación

1. Para ejecutar la aplicación de consola, haga clic en **Iniciar** en Visual Studio.

2. Después de la devolución de cada recurso, presione **Entrar**. En la información de estado, debería ver el estado de aprovisionamiento **Correcto**. Después de crear la máquina virtual, tendrá la oportunidad de eliminar todos los recursos creados. Antes de presionar **Entrar** para comenzar la eliminación de recursos, puede dedicarle unos minutos a comprobar si se han creado en Azure Portal. Si tiene abierto Azure Portal, tendrá que actualizar la hoja para ver los recursos nuevos.  

    Esta aplicación de consola tardará unos cinco minutos en ejecutarse completamente de principio a fin. Cuando termine la aplicación, la eliminación de los recursos y el grupo de recursos puede tardar unos minutos.


## <a name="next-steps"></a>Pasos siguientes

- Si se han producido problemas durante la implementación, el paso siguiente será consultar [Solución de problemas de implementaciones de grupo de recursos con Azure Portal](../../azure-resource-manager/templates/deployment-history.md)
- Más información acerca de la [biblioteca de Python de Azure](/python/api/overview/azure/?view=azure-python)