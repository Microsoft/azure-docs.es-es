---
title: 'Tutorial: Personalización de una máquina virtual Linux con cloud-init en Azure | Microsoft Docs'
description: En este tutorial, aprenderá a usar cloud-init y Key Vault para personalizar las máquinas virtuales Linux la primera vez que arrancan en Azure
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/30/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 4f4456ed81fba4648f5bc3efaa415b1e44e5e415
ms.sourcegitcommit: 8115c7fa126ce9bf3e16415f275680f4486192c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/24/2019
ms.locfileid: "54852123"
---
# <a name="tutorial---how-to-use-cloud-init-to-customize-a-linux-virtual-machine-in-azure-on-first-boot"></a>Tutorial: Uso de cloud-init para personalizar una máquina virtual Linux en Azure durante el primer arranque

En el tutorial anterior, aprendió a cómo acceder mediante SSH a una máquina virtual y tambien a instalar NGINX de forma manual. Para crear máquinas virtuales de manera rápida y coherente, algunas formas de automatización suelen ser deseables. Un enfoque común para personalizar una máquina virtual en el primer arranque es usar el comando [cloud-init](https://cloudinit.readthedocs.io). En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un archivo de configuración cloud-init
> * Crear una máquina virtual que usa un archivo cloud-init
> * Ver una aplicación Node.js en ejecución una vez creada la máquina virtual
> * Usar Key Vault para almacenar de forma segura certificados
> * Automatizar implementaciones seguras de NGINX con cloud-init

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.30, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="cloud-init-overview"></a>Introducción a cloud-init
[cloud-init](https://cloudinit.readthedocs.io) es un enfoque ampliamente usado para personalizar una máquina virtual Linux la primera vez que se arranca. Puede usar cloud-init para instalar paquetes y escribir archivos o para configurar los usuarios y la seguridad. Como cloud-init se ejecuta durante el proceso de arranque inicial, no hay pasos adicionales o agentes requeridos que aplicar a la configuración.

cloud-init también funciona entre distribuciones. Por ejemplo, no use **apt-get install** o **yum install** para instalar un paquete. En su lugar, puede definir una lista de paquetes que se van a instalar. Cloud-init usará automáticamente la herramienta de administración de paquetes nativos para la distribución de Linux (distro) que seleccione.

Trabajamos con nuestros asociados para que cloud-init se incluya y funcione en las imágenes que estos proporcionan a Azure. En la tabla siguiente se describe la disponibilidad actual de cloud-init en imágenes de la plataforma de Azure:

| Alias | Publicador | Oferta | SKU | Versión |
|:--- |:--- |:--- |:--- |:--- |:--- |
| UbuntuLTS |Canonical |UbuntuServer |16.04-LTS |más reciente |
| UbuntuLTS |Canonical |UbuntuServer |14.04.5-LTS |más reciente |
| CoreOS |CoreOS |CoreOS |Stable |más reciente |
| | OpenLogic | CentOS | 7-CI | más reciente |
| | Redhat | RHEL | 7-RAW-CI | más reciente |


## <a name="create-cloud-init-config-file"></a>Creación de un archivo de configuración cloud-init
Para ver cloud-init en acción, cree una máquina virtual que instale NGINX y ejecute una aplicación Node.js sencilla "Hello World". Con la siguiente configuración de cloud-init se instalan los paquetes necesarios, se crea una aplicación Node.js y luego se inicializa e inicia la aplicación.

En el shell actual, cree un archivo denominado "*cloud-init.txt*" y pegue la siguiente configuración. Por ejemplo, cree el archivo en Cloud Shell, no en la máquina local. Puede utilizar el editor que prefiera. Escriba `sensible-editor cloud-init.txt` para crear el archivo y ver una lista de editores disponibles. Asegúrese de que todo el archivo cloud-init se copia correctamente, especialmente la primera línea:

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

Para más información sobre las opciones de configuración de cloud-init, consulte los [ejemplos de configuración de cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/examples.html).

## <a name="create-virtual-machine"></a>Crear máquina virtual
Antes de poder crear una máquina virtual, cree un grupo de recursos con [az group create](/cli/azure/group#az-group-create). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroupAutomate* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroupAutomate --location eastus
```

Ahora cree una máquina virtual con el comando [az vm create](/cli/azure/vm#az-vm-create). Use el parámetro `--custom-data` para pasar su archivo de configuración cloud-init. Proporcione la ruta de acceso completa a la configuración de *cloud-init.txt* si guardó el archivo fuera de su directorio de trabajo actual. En el ejemplo siguiente se crea una VM denominada *myAutomatedVM*:

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupAutomate \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
```

Transcurren unos minutos hasta que la máquina virtual se crea, los paquetes se instalan y la aplicación se inicia. Hay tareas en segundo plano que continúan ejecutándose después de que la CLI de Azure vuelve a abrir el símbolo del sistema. Es posible que tenga que esperar otros dos minutos antes de poder acceder a la aplicación. Cuando se haya creado la máquina virtual, anote el valor `publicIpAddress` mostrado por la CLI de Azure. Esta dirección se usa para acceder a la aplicación Node.js mediante un explorador web.

Para permitir que el tráfico web llegue a la máquina virtual, abra el puerto 80 desde Internet con el comando [az vm open-port](/cli/azure/vm#az-vm-open-port):

```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroupAutomate --name myVM
```

## <a name="test-web-app"></a>Prueba de la aplicación web
Ahora puede abrir un explorador web y escribir *http://<publicIpAddress>* en la barra de direcciones. Proporcione su propia dirección IP pública obtenida del proceso de creación de la máquina virtual. Su aplicación Node.js se muestra como en el ejemplo siguiente:

![Ver sitio de NGINX en funcionamiento](./media/tutorial-automate-vm-deployment/nginx.png)


## <a name="inject-certificates-from-key-vault"></a>Inserción de certificados desde Key Vault
En esta sección opcional se muestra cómo puede almacenar certificados de forma segura en Azure Key Vault e insertarlos durante la implementación de máquinas virtuales. En lugar de usar una imagen personalizada que incluye los certificados ya preparados, este proceso garantiza que los certificados más actualizados se insertan en una máquina virtual en el primer arranque. Durante el proceso, el certificado nunca deja la plataforma de Azure ni se expone en un script, el historial de la línea de comandos o una plantilla.

Azure Key Vault protege claves y secretos criptográficos, como certificados y contraseñas. Key Vault ayuda a agilizar el proceso de administración de claves y le permite mantener el control de las claves que acceden a sus datos y los cifran. En este escenario se presentan algunos conceptos de Key Vault para crear y usar un certificado, si bien no es una introducción exhaustiva sobre cómo usar Key Vault.

Los pasos siguientes muestran cómo puede:

- Crear una instancia de Azure Key Vault
- Generar o cargar un certificado en Key Vault
- Crear un secreto a partir del certificado para insertarlo en una máquina virtual
- Crear una máquina virtual e insertar el certificado

### <a name="create-an-azure-key-vault"></a>Crear una instancia de Azure Key Vault
En primer lugar, cree una instancia de Key Vault con [az keyvault create](/cli/azure/keyvault#az-keyvault-create) y habilítela para su uso al implementar una máquina virtual. Cada instancia de Key Vault requiere un nombre único, que debe estar todo en minúsculas. Reemplace *mykeyvault* en el siguiente ejemplo por su propio nombre único de Key Vault:

```azurecli-interactive
keyvault_name=mykeyvault
az keyvault create \
    --resource-group myResourceGroupAutomate \
    --name $keyvault_name \
    --enabled-for-deployment
```

### <a name="generate-certificate-and-store-in-key-vault"></a>Generación de un certificado y su almacenamiento en Key Vault
Para usarlo en producción, debe importar un certificado válido firmado por un proveedor de confianza con [az keyvault certificate import](/cli/azure/keyvault/certificate#az-keyvault-certificate-import). En este tutorial, el ejemplo siguiente muestra cómo puede generar un certificado autofirmado con [az keyvault certificate create](/cli/azure/keyvault/certificate#az-keyvault-certificate-create) que usa la directiva de certificado predeterminada:

```azurecli-interactive
az keyvault certificate create \
    --vault-name $keyvault_name \
    --name mycert \
    --policy "$(az keyvault certificate get-default-policy)"
```


### <a name="prepare-certificate-for-use-with-vm"></a>Preparación del certificado para usarlo con la máquina virtual
Para usar el certificado durante el proceso de creación de la máquina virtual, obtenga el id. del certificado con [az keyvault secret list-versions](/cli/azure/keyvault/secret#az-keyvault-secret-list-versions). La máquina virtual necesita el certificado en un formato determinado para insertarlo en el arranque, por lo que convierte el certificado con [az vm secret format](/cli/azure/vm). En el ejemplo siguiente se asigna la salida de estos comandos a las variables para facilitar su uso en los pasos siguientes:

```azurecli-interactive
secret=$(az keyvault secret list-versions \
          --vault-name $keyvault_name \
          --name mycert \
          --query "[?attributes.enabled].id" --output tsv)
vm_secret=$(az vm secret format --secret "$secret")
```


### <a name="create-cloud-init-config-to-secure-nginx"></a>Creación de la configuración de cloud-init para proteger NGINX
Cuando crea una máquina virtual, los certificados y las claves se almacenan en el directorio */var/lib/waagent/* protegido. Para automatizar el proceso de agregar el certificado a la máquina virtual y configurar NGINX, puede utilizar la configuración de cloud-init actualizada del ejemplo anterior.

Cree un archivo denominado "*cloud-init-secured.txt*" y pegue la siguiente configuración. De nuevo, si se utiliza Cloud Shell, cree ahí el archivo de configuración de cloud-init y no en la máquina local. Use `sensible-editor cloud-init-secured.txt` para crear el archivo y ver una lista de editores disponibles. Asegúrese de que todo el archivo cloud-init se copia correctamente, especialmente la primera línea:

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        listen 443 ssl;
        ssl_certificate /etc/nginx/ssl/mycert.cert;
        ssl_certificate_key /etc/nginx/ssl/mycert.prv;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - secretsname=$(find /var/lib/waagent/ -name "*.prv" | cut -c -57)
  - mkdir /etc/nginx/ssl
  - cp $secretsname.crt /etc/nginx/ssl/mycert.cert
  - cp $secretsname.prv /etc/nginx/ssl/mycert.prv
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

### <a name="create-secure-vm"></a>Creación de una máquina virtual segura
Ahora cree una máquina virtual con el comando [az vm create](/cli/azure/vm#az-vm-create). Los datos del certificado se insertan desde Key Vault con el parámetro `--secrets`. Como en el ejemplo anterior, se pasa la configuración de cloud-init con el parámetro `--custom-data`:

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupAutomate \
    --name myVMSecured \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init-secured.txt \
    --secrets "$vm_secret"
```

Transcurren unos minutos hasta que la máquina virtual se crea, los paquetes se instalan y la aplicación se inicia. Hay tareas en segundo plano que continúan ejecutándose después de que la CLI de Azure vuelve a abrir el símbolo del sistema. Es posible que tenga que esperar otros dos minutos antes de poder acceder a la aplicación. Cuando se haya creado la máquina virtual, anote el valor `publicIpAddress` mostrado por la CLI de Azure. Esta dirección se usa para acceder a la aplicación Node.js mediante un explorador web.

Para permitir que el tráfico web llegue a la máquina virtual, abra el puerto 443 desde Internet con el comando [az vm open-port](/cli/azure/vm#az-vm-open-port):

```azurecli-interactive
az vm open-port \
    --resource-group myResourceGroupAutomate \
    --name myVMSecured \
    --port 443
```

### <a name="test-secure-web-app"></a>Prueba de la aplicación web segura
Ahora puede abrir un explorador web y escribir *https://<publicIpAddress>* en la barra de direcciones. Proporcione su propia dirección IP pública, como se muestra en la salida del proceso anterior de creación de máquinas virtuales. Acepte la advertencia de seguridad si usó un certificado autofirmado:

![Aceptar la advertencia de seguridad del explorador web](./media/tutorial-automate-vm-deployment/browser-warning.png)

Se muestran el sitio de NGINX protegido y la aplicación Node.js, como en el ejemplo siguiente:

![Ver el sitio de NGINX seguro en funcionamiento](./media/tutorial-automate-vm-deployment/secured-nginx.png)


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha configurado las máquinas virtuales en el primer inicio con cloud-init. Ha aprendido a:

> [!div class="checklist"]
> * Crear un archivo de configuración cloud-init
> * Crear una máquina virtual que usa un archivo cloud-init
> * Ver una aplicación Node.js en ejecución una vez creada la máquina virtual
> * Usar Key Vault para almacenar de forma segura certificados
> * Automatizar implementaciones seguras de NGINX con cloud-init

Avanzar al siguiente tutorial para aprender a crear imágenes de máquina virtual personalizadas.

> [!div class="nextstepaction"]
> [Creación de imágenes personalizadas de máquinas virtuales](./tutorial-custom-images.md)
