---
title: 'Tutorial: Configuración de inventarios dinámicos de los recursos de Azure con Ansible | Microsoft Docs'
description: Más información acerca del uso de Ansible para administrar sus inventarios dinámicos de Azure
keywords: ansible, azure, devops, bash, cloudshell, inventario dinámico
ms.topic: tutorial
ms.service: ansible
author: tomarchermsft
manager: jeconnoc
ms.author: tarcher
ms.date: 04/30/2019
ms.openlocfilehash: 46b13fae437a555edf0bdd0b0d4c1496d7596e0f
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230700"
---
# <a name="tutorial-configure-dynamic-inventories-of-your-azure-resources-using-ansible"></a>Tutorial: Configuración de inventarios dinámicos de los recursos de Azure con Ansible

Ansible puede utilizarse para extraer información de inventario de varios orígenes (incluidos orígenes en la nube como Azure) en un *inventario dinámico*. 

[!INCLUDE [ansible-tutorial-goals.md](../../includes/ansible-tutorial-goals.md)]

> [!div class="checklist"]
>
> * Configurar dos máquinas virtuales de prueba. 
> * Etiquetar una de las máquinas virtuales.
> * Instalar Nginx en las máquinas virtuales etiquetadas.
> * Configurar un inventario dinámico que incluya los recursos de Azure configurados.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../../includes/open-source-devops-prereqs-azure-subscription.md)]
[!INCLUDE [open-source-devops-prereqs-create-service-principal.md](../../includes/open-source-devops-prereqs-create-service-principal.md)]
[!INCLUDE [ansible-prereqs-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-cloudshell-use-or-vm-creation2.md)]

## <a name="create-the-test-vms"></a>Creación de las máquinas virtuales de prueba

1. Inicie sesión en el [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Abra [Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

1. Cree un grupo de recursos de Azure para almacenar las máquinas virtuales para este tutorial.

    > [!IMPORTANT]  
    > El grupo de recursos de Azure que crea en este paso debe tener un nombre que está completamente en minúsculas. De lo contrario, se producirá un error en la generación del inventario dinámico.

    ```azurecli-interactive
    az group create --resource-group ansible-inventory-test-rg --location eastus
    ```

1. Cree dos máquinas virtuales Linux en Azure mediante una de las técnicas siguientes:

    - **Guión de procedimientos de Ansible**: el artículo [Creación de una máquina virtual básica en Azure con Ansible](/azure/virtual-machines/linux/ansible-create-vm) explica cómo crear una máquina virtual a partir de un guión de procedimientos de Ansible. Si usa un guión de procedimientos para definir una o las dos máquinas virtuales, asegúrese de que se utiliza la conexión SSH en lugar de una contraseña.

    - **CLI de Azure**: Use cada uno de los siguientes comandos en Cloud Shell para crear las dos máquinas virtuales:

        ```azurecli-interactive
        az vm create --resource-group ansible-inventory-test-rg \
                     --name ansible-inventory-test-vm1 \
                     --image UbuntuLTS --generate-ssh-keys
        ```

        ```azurecli-interactive
        az vm create --resource-group ansible-inventory-test-rg \
                     --name ansible-inventory-test-vm2 \
                     --image UbuntuLTS --generate-ssh-keys
        ```

## <a name="tag-a-vm"></a>Etiquetado de una máquina virtual

Puede [usar etiquetas para organizar los recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags#azure-cli) por categorías definidas por el usuario. 

Escriba el comando [az resource tag](/cli/azure/resource?view=azure-cli-latest.md#az-resource-tag) siguiente para etiquetar la máquina virtual `ansible-inventory-test-vm1` con la clave `nginx`:

```azurecli-interactive
az resource tag --tags nginx --id /subscriptions/<YourAzureSubscriptionID>/resourceGroups/ansible-inventory-test-rg/providers/Microsoft.Compute/virtualMachines/ansible-inventory-test-vm1
```
## <a name="generate-a-dynamic-inventory"></a>Generación de un inventario dinámico

Una vez definidas y etiquetadas las máquinas virtuales, es el momento de generar el inventario dinámico.

### <a name="using-ansible-version--28"></a>Uso de versiones de Ansible anteriores a 2.8

Ansible proporciona un script de Python llamado [azure_rm.py](https://github.com/ansible/ansible/blob/devel/contrib/inventory/azure_rm.py) que genera un inventario dinámico de sus recursos de Azure. Los pasos siguientes le guían en el uso del script `azure_rm.py` para conectarse a sus dos máquinas virtual de Azure de prueba:

1. Use el comando `wget` de GNU para recuperar el script `azure_rm.py`:

    ```azurecli-interactive
    wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/azure_rm.py
    ```

1. Use el comando `chmod` para cambiar los permisos de acceso al script `azure_rm.py`. El siguiente comando usa el parámetro `+x` para permitir la ejecución del archivo especificado (`azure_rm.py`):

    ```azurecli-interactive
    chmod +x azure_rm.py
    ```

1. Use el [comando ansible](https://docs.ansible.com/ansible/2.4/ansible.html) para conectarse al grupo de recursos: 

    ```azurecli-interactive
    ansible -i azure_rm.py ansible-inventory-test-rg -m ping 
    ```

1. Una vez conectado, debería ver resultados similares a lo siguiente:

    ```Output
    ansible-inventory-test-vm1 | SUCCESS => {
        "changed": false,
        "failed": false,
        "ping": "pong"
    }
    ansible-inventory-test-vm2 | SUCCESS => {
        "changed": false,
        "failed": false,
        "ping": "pong"
    }
    ```

### <a name="ansible-version--28"></a>Ansible versión 2.8 y posteriores

A partir de la versión 2.8, Ansible proporciona un [complemento de inventario dinámico de Azure](https://github.com/ansible/ansible/blob/devel/lib/ansible/plugins/inventory/azure_rm.py). Los siguientes pasos le guiarán en el uso del complemento:

1. El complemento de inventario requiere un archivo de configuración. El archivo de configuración debe terminar con `azure_rm` y tener la extensión `yml` o `yaml`. Para este ejemplo del tutorial, guarde el siguiente cuaderno de estrategias como `myazure_rm.yml`:

    ```yml
    plugin: azure_rm
    include_vm_resource_groups:
    - ansible-inventory-test-rg
    auth_source: auto
    ```

1. Ejecute el siguiente comando para hacer ping a las máquinas virtuales del grupo de recursos:

    ```bash
    ansible all -m ping -i ./myazure_rm.yml
    ```

1. Al ejecutar el comando anterior, podría recibir el siguiente error:

    ```Output
    Failed to connect to the host via ssh: Host key verification failed.
    ```
    
    Si recibe el error "host-key verification" (comprobación de la clave de host), agregue la siguiente línea al archivo de configuración de Ansible. El archivo de configuración de Ansible se encuentra en `/etc/ansible/ansible.cfg`.

    ```bash
    host_key_checking = False
    ```

1. Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a la siguiente salida:
  
    ```Output
    ansible-inventory-test-vm1_0324 : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ansible-inventory-test-vm2_8971 : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    ```

## <a name="enable-the-vm-tag"></a>Habilitación de la etiqueta de la máquina virtual
Una vez que ha establecido una etiqueta, debe "habilitarla". Una manera de habilitar una etiqueta es exportarla a una variable de entorno llamada `AZURE_TAGS` mediante el comando `export`:

```azurecli-interactive
export AZURE_TAGS=nginx
```

- Si usa versiones de Ansible anteriores a 2.8, ejecute el siguiente comando:

    ```bash
    ansible -i azure_rm.py ansible-inventory-test-rg -m ping
    ```

- Si usa Ansible 2.8 o versiones posteriores, ejecute el siguiente comando:
  
    ```bash
    ansible all -m ping -i ./myazure_rm.yml
    ```

Ahora verá solo una máquina virtual (aquella cuya etiqueta coincida con el valor exportado a la variable de entorno `AZURE_TAGS`):

```Output
ansible-inventory-test-vm1 | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}
```

## <a name="set-up-nginx-on-the-tagged-vm"></a>Configuración de Nginx en la máquina virtual etiquetada

El objetivo de las etiquetas es permitir que se pueda trabajar de forma rápida y sencilla con subgrupos de las máquinas virtuales. Por ejemplo, supongamos que desea instalar Nginx solo en las máquinas virtuales a las que ha asignado una etiqueta de `nginx`. Los siguientes pasos muestran lo fácil que puede hacerlo:

1. Cree un archivo llamado `nginx.yml`:

   ```azurecli-interactive
   code nginx.yml
   ```

1. Pegue el siguiente código de ejemplo en el editor:

    ```yml
    ---
    - name: Install and start Nginx on an Azure virtual machine
      hosts: all
      become: yes
      tasks:
      - name: install nginx
        apt: pkg=nginx state=installed
        notify:
        - start nginx

      handlers:
        - name: start nginx
          service: name=nginx state=started
    ```

1. Guarde el archivo y salga del editor.

1. Use el comando `ansible-playbook` para ejecutar el cuaderno de estrategias:

   - Ansible versiones anteriores a 2.8:

    ```bash
    ansible-playbook -i azure_rm.py nginx.yml
    ```

   - Ansible 2.8 y versiones posteriores:

    ```bash
     ansible-playbook  -i ./myazure_rm.yml  nginx.yml
    ```

1. Tras ejecutar el cuaderno de estrategias, debería ver resultados similares a los siguientes:

    ```Output
    PLAY [Install and start Nginx on an Azure virtual machine] 

    TASK [Gathering Facts] 
    ok: [ansible-inventory-test-vm1]

    TASK [install nginx] 
    changed: [ansible-inventory-test-vm1]

    RUNNING HANDLER [start nginx] 
    ok: [ansible-inventory-test-vm1]

    PLAY RECAP 
    ansible-inventory-test-vm1 : ok=3    changed=1    unreachable=0    failed=0
    ```

## <a name="test-nginx-installation"></a>Prueba de la instalación de Nginx

En esta sección se muestra una técnica para probar que Nginx está instalado en la máquina virtual.

1. Use el comando [az vm list-ip-addresses](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-list-ip-addresses) para recuperar la dirección IP de la máquina virtual `ansible-inventory-test-vm1`. El valor devuelto (dirección IP de la máquina virtual) se utiliza entonces como parámetro del comando SSH para conectarse a la máquina virtual.

    ```azurecli-interactive
    ssh `az vm list-ip-addresses \
    -n ansible-inventory-test-vm1 \
    --query [0].virtualMachine.network.publicIpAddresses[0].ipAddress -o tsv`
    ```

1. Mientras esté conectado a la máquina virtual `ansible-inventory-test-vm1`, ejecute el comando [nginx - v](https://nginx.org/en/docs/switches.html) comando para determinar si Nginx está instalado.

    ```azurecli-interactive
    nginx -v
    ```

1. Una vez que ejecute el comando `nginx -v`, verá la versión de Nginx (segunda línea) que indica que Nginx está instalado.

    ```Output
    tom@ansible-inventory-test-vm1:~$ nginx -v

    nginx version: nginx/1.10.3 (Ubuntu)

    tom@ansible-inventory-test-vm1:~$
    ```

1. Presione la combinación de teclado `<Ctrl>D` para desconectar la sesión de SSH.

1. Al realizar los anteriores pasos para la máquina virtual `ansible-inventory-test-vm2`, aparece un mensaje informativo que indica dónde puede obtener Nginx (lo que implica que no lo tiene instalado en este momento):

    ```Output
    tom@ansible-inventory-test-vm2:~$ nginx -v
    The program 'nginx' can be found in the following packages:
    * nginx-core
    * nginx-extras
    * nginx-full
    * nginx-lightTry: sudo apt install <selected package>
    tom@ansible-inventory-test-vm2:~$
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"] 
> [Inicio rápido: Configuración de máquinas virtuales Linux en Azure con Ansible](/azure/virtual-machines/linux/ansible-create-vm)