---
title: 'Tutorial: implementación de LAMP en una máquina virtual Linux en Azure'
description: En este tutorial, aprenderá a instalar la pila LAMP en una máquina virtual Linux en Azure.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: 6c12603a-e391-4d3e-acce-442dd7ebb2fe
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 01/30/2019
ms.author: cynthn
ms.openlocfilehash: 5a3b3d7c0bf61ea0aa9b85965c11e572e9e2d999
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085408"
---
# <a name="tutorial-install-a-lamp-web-server-on-a-linux-virtual-machine-in-azure"></a>Tutorial: Instalación de un servidor web LAMP en una máquina virtual Linux en Azure

En este artículo se ofrecen instrucciones paso a paso para implementar un servidor web Apache, MySQL y PHP (la pila LAMP) en una máquina virtual de Ubuntu en Azure. Si prefiere el servidor web NGINX, consulte el tutorial [Pila LEMP](). Para ver el servidor LAMP en acción, si lo desea, puede instalar y configurar un sitio de WordPress. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una máquina virtual de Ubuntu (la "L" de la pila LAMP)
> * Apertura del puerto 80 para el tráfico web
> * Instalación de Apache, MySQL y PHP
> * Comprobación de la instalación y configuración
> * Instalación de WordPress en el servidor LAMP

Esta configuración es para pruebas rápidas o prueba de concepto. Para más información sobre la pila LAMP, incluyendo recomendaciones para un entorno de producción, consulte la [documentación de Ubuntu](https://help.ubuntu.com/community/ApacheMySQLPHP).

En este tutorial se usa la CLI dentro de [Azure Cloud Shell](../../cloud-shell/overview.md), que se actualiza constantemente a la versión más reciente. Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior de cualquier bloque de código.

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.30, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

[!INCLUDE [virtual-machines-linux-tutorial-stack-intro.md](../../../includes/virtual-machines-linux-tutorial-stack-intro.md)]

## <a name="install-apache-mysql-and-php"></a>Instalación de Apache, MySQL y PHP

Ejecute el siguiente comando para actualizar los orígenes de paquetes de Ubuntu e instalar Apache, MySQL y PHP. Tenga en cuenta el símbolo de intercalación (^) al final del comando, que forma parte del nombre del paquete `lamp-server^`. 


```bash
sudo apt update && sudo apt install lamp-server^
```

Se le pide que instale los paquetes y otras dependencias. Este proceso instala las extensiones PHP mínimas necesarias para utilizar PHP con MySQL.  

## <a name="verify-installation-and-configuration"></a>Comprobación de la instalación y configuración


### <a name="verify-apache"></a>Comprobación de Apache

Compruebe la versión de Apache con el comando siguiente:
```bash
apache2 -v
```

Con Apache instalado y el puerto 80 abierto para la máquina virtual, ahora se puede acceder al servidor web desde Internet. Para ver la página predeterminada de Apache2 Ubuntu, abra un explorador web y escriba la dirección IP pública de la máquina virtual. Use la dirección IP pública utilizada para SSH en la máquina virtual:

![Página predeterminada de Apache][3]


### <a name="verify-and-secure-mysql"></a>Comprobación y protección de MySQL

Compruebe la versión de MySQL con el siguiente comando (tenga en cuenta el parámetro `V` en mayúsculas):

```bash
mysql -V
```

Para ayudar a proteger la instalación de MySQL, incluido el establecimiento de una contraseña raíz, ejecute el script `mysql_secure_installation`. 

```bash
sudo mysql_secure_installation
```

Si lo desea, puede configurar el complemento para validar contraseña (se recomienda). A continuación, establezca una contraseña para el usuario raíz de MySQL y configure las opciones de seguridad restantes para su entorno. Se recomienda que responda "Y" (Yes o Sí) para todas las preguntas.

Si desea crear probar características de MySQL (crear una base de datos MySQL, agregar usuarios o cambiar la configuración), inicie sesión en MySQL. Este paso no es necesario para completar este tutorial.

```bash
sudo mysql -u root -p
```

Cuando haya terminado, escriba `\q` para salir del símbolo del sistema de mysql.

### <a name="verify-php"></a>Comprobación de PHP

Compruebe la versión de PHP con el comando siguiente:

```bash
php -v
```

Si desea realizar más pruebas, cree una página PHP de información rápida para verla en un explorador. El siguiente comando crea la página de información de PHP:

```bash
sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
```

Ahora puede comprobar la página de información de PHP que creó. Abra un explorador y vaya a `http://yourPublicIPAddress/info.php`. Sustituya la dirección por la dirección IP pública de la máquina virtual. Debe tener un aspecto similar a esta imagen.

![Página de información de PHP][2]

[!INCLUDE [virtual-machines-linux-tutorial-wordpress.md](../../../includes/virtual-machines-linux-tutorial-wordpress.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, implementó un servidor LAMP en Azure. Ha aprendido a:

> [!div class="checklist"]
> * Creación de una máquina virtual de Ubuntu
> * Apertura del puerto 80 para el tráfico web
> * Instalación de Apache, MySQL y PHP
> * Comprobación de la instalación y configuración
> * Instalación de WordPress en el servidor LAMP

Pase al siguiente tutorial para aprender a proteger servidores web con certificados TLS/SSL.

> [!div class="nextstepaction"]
> [Protección de servidores web con TLS](tutorial-secure-web-server.md)

[2]: ./media/tutorial-lamp-stack/phpsuccesspage.png
[3]: ./media/tutorial-lamp-stack/apachesuccesspage.png
