---
title: 'Tutorial: Protección de un servidor web con Linux con certificados TLS/SSL en Azure'
description: En este tutorial, aprenderá a usar la CLI de Azure para proteger una máquina virtual Linux que ejecuta el servidor web NGINX con certificados SSL almacenados en Azure Key Vault.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/30/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 7964f96afc59464c28cabb9e1d5c7961fc765a3d
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86501779"
---
# <a name="tutorial-secure-a-web-server-on-a-linux-virtual-machine-in-azure-with-tlsssl-certificates-stored-in-key-vault"></a>Tutorial: Protección de un servidor web en una máquina virtual Linux en Azure con certificados TLS/SSL almacenados en Key Vault
Para proteger los servidores web, se puede usar un certificado de seguridad de la capa de transporte (TLS), conocido anteriormente como Capa de sockets seguros (SSL), para cifrar el tráfico web. Estos certificados TLS/SSL pueden almacenarse en Azure Key Vault y permiten implementaciones seguras de certificados en máquinas virtuales Linux en Azure. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una instancia de Azure Key Vault
> * Generar o cargar un certificado en Key Vault
> * Creación de una máquina virtual e instalación del servidor web NGINX
> * Insertar el certificado en la máquina virtual y configurar NGINX con un enlace TLS

En este tutorial se usa la CLI dentro de [Azure Cloud Shell](../../cloud-shell/overview.md), que se actualiza constantemente a la versión más reciente. Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior de cualquier bloque de código.

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.30, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).


## <a name="overview"></a>Información general
Azure Key Vault protege claves y secretos criptográficos, como certificados y contraseñas. Key Vault ayuda a agilizar el proceso de administración de certificados y le permite mantener el control de las claves que acceden a ellos. Puede crear un certificado autofirmado en Key Vault o cargar un certificado de confianza existente que ya posea.

En lugar de usar una imagen de máquina virtual personalizada que incluya los certificados preparados, inserta los certificados en una máquina virtual en ejecución. Este proceso garantiza que los certificados más actualizados se instalan en un servidor web durante la implementación. Si renueva o reemplaza un certificado, no tiene también que crear una nueva imagen de máquina virtual personalizada. Los certificados más recientes se insertan automáticamente a medida que se crean máquinas virtuales adicionales. Durante el proceso completo, el certificado nunca deja la plataforma de Azure ni se expone en un script, historial de la línea de comandos o una plantilla.


## <a name="create-an-azure-key-vault"></a>Crear una instancia de Azure Key Vault
Para poder crear una instancia de Key Vault y certificados, cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroupSecureWeb* en la ubicación *eastus*:

```azurecli-interactive 
az group create --name myResourceGroupSecureWeb --location eastus
```

Después, cree una instancia de Key Vault con [az keyvault create](/cli/azure/keyvault) y habilítela para usarla al implementar una máquina virtual. Cada almacén Key Vault requiere un nombre único, que debe estar todo en minúsculas. Reemplace *\<mykeyvault>* en el siguiente ejemplo por su propio nombre único de Key Vault:

```azurecli-interactive 
keyvault_name=<mykeyvault>
az keyvault create \
    --resource-group myResourceGroupSecureWeb \
    --name $keyvault_name \
    --enabled-for-deployment
```

## <a name="generate-a-certificate-and-store-in-key-vault"></a>Generación de un certificado y almacenamiento en Key Vault
Para usarlo en producción, debe importar un certificado válido firmado por un proveedor de confianza con [az keyvault certificate import](/cli/azure/keyvault/certificate). En este tutorial, el ejemplo siguiente muestra cómo puede generar un certificado autofirmado con [az keyvault certificate create](/cli/azure/keyvault/certificate) que usa la directiva de certificado predeterminada:

```azurecli-interactive 
az keyvault certificate create \
    --vault-name $keyvault_name \
    --name mycert \
    --policy "$(az keyvault certificate get-default-policy)"
```

### <a name="prepare-a-certificate-for-use-with-a-vm"></a>Preparación del certificado para usarlo con una máquina virtual
Para usar el certificado durante el proceso de creación de la máquina virtual, obtenga el id. del certificado con [az keyvault secret list-versions](/cli/azure/keyvault/secret). Convierta el certificado con [az vm secret format](/cli/azure/vm/secret#az-vm-secret-format). En el ejemplo siguiente se asigna la salida de estos comandos a las variables para facilitar su uso en los pasos siguientes:

```azurecli-interactive 
secret=$(az keyvault secret list-versions \
          --vault-name $keyvault_name \
          --name mycert \
          --query "[?attributes.enabled].id" --output tsv)
vm_secret=$(az vm secret format --secrets "$secret" -g myResourceGroupSecureWeb --keyvault $keyvault_name)
```

### <a name="create-a-cloud-init-config-to-secure-nginx"></a>Creación de una configuración de cloud-init para proteger NGINX
[cloud-init](https://cloudinit.readthedocs.io) es un enfoque ampliamente usado para personalizar una máquina virtual Linux la primera vez que se arranca. Puede usar cloud-init para instalar paquetes y escribir archivos o para configurar los usuarios y la seguridad. Como cloud-init se ejecuta durante el proceso de arranque inicial, no hay pasos adicionales o agentes requeridos que aplicar a la configuración.

Cuando crea una máquina virtual, los certificados y las claves se almacenan en el directorio */var/lib/waagent/* protegido. Para automatizar la adición del certificado a la máquina virtual y configurar el servidor web, utilice cloud-init. En este ejemplo, instalará y configurará el servidor web NGINX. Puede usar el mismo proceso para instalar y configurar Apache. 

Cree un archivo denominado *cloud-init-web-server.txt* y pegue la siguiente configuración:

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 443 ssl;
        ssl_certificate /etc/nginx/ssl/mycert.cert;
        ssl_certificate_key /etc/nginx/ssl/mycert.prv;
      }
runcmd:
  - secretsname=$(find /var/lib/waagent/ -name "*.prv" | cut -c -57)
  - mkdir /etc/nginx/ssl
  - cp $secretsname.crt /etc/nginx/ssl/mycert.cert
  - cp $secretsname.prv /etc/nginx/ssl/mycert.prv
  - service nginx restart
```

### <a name="create-a-secure-vm"></a>Creación de una máquina virtual segura
Ahora cree una máquina virtual con el comando [az vm create](/cli/azure/vm). Los datos del certificado se insertan desde Key Vault con el parámetro `--secrets`. Pase la configuración cloud-init con el parámetro `--custom-data`:

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroupSecureWeb \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init-web-server.txt \
    --secrets "$vm_secret"
```

Transcurren unos minutos hasta que la máquina virtual se crea, los paquetes se instalan y la aplicación se inicia. Cuando se haya creado la máquina virtual, anote el valor `publicIpAddress` mostrado por la CLI de Azure. Esta dirección se usa para acceder al sitio en un explorador web.

Para permitir que el tráfico web llegue a la máquina virtual, abra el puerto 443 desde Internet con el comando [az vm open-port](/cli/azure/vm):

```azurecli-interactive 
az vm open-port \
    --resource-group myResourceGroupSecureWeb \
    --name myVM \
    --port 443
```


### <a name="test-the-secure-web-app"></a>Prueba de la aplicación web segura
Ahora puede abrir un explorador web y escribir *https:\/\/\<publicIpAddress>* en la barra de direcciones. Proporcione su propia dirección IP pública obtenida del proceso de creación de la máquina virtual. Acepte la advertencia de seguridad si usó un certificado autofirmado:

![Aceptar la advertencia de seguridad del explorador web](./media/tutorial-secure-web-server/browser-warning.png)

A continuación, se muestra el sitio de NGINX protegido, como en el ejemplo siguiente:

![Ver el sitio de NGINX seguro en funcionamiento](./media/tutorial-secure-web-server/secured-nginx.png)


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, protegió un servidor web NGINX con un certificado TLS/SSL almacenado en Azure Key Vault. Ha aprendido a:

> [!div class="checklist"]
> * Crear una instancia de Azure Key Vault
> * Generar o cargar un certificado en Key Vault
> * Creación de una máquina virtual e instalación del servidor web NGINX
> * Insertar el certificado en la máquina virtual y configurar NGINX con un enlace TLS

Siga este vínculo para ver ejemplos de scripts de máquina virtual creados previamente.

> [!div class="nextstepaction"]
> [Ejemplos de scripts de máquina virtual Linux](./cli-samples.md)
