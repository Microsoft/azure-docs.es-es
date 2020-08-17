---
title: Configuración de una instancia de Application Developers Controlled Distribution (ADCD) en IBM zD&T v1 | Microsoft Docs
description: Ejecute un entorno de IBM Z Development and Test Environment (zD&T) en máquinas virtuales (VM) de Azure.
services: virtual-machines-linux
ms.service: virtual-machines-linux
documentationcenter: ''
author: njray
manager: edprice
ms.author: edprice
editor: edprice
ms.topic: conceptual
ms.date: 02/22/2019
tags: ''
keywords: ''
ms.openlocfilehash: b2509539551b3991690e6d0313e069ae015eb892
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87052397"
---
# <a name="set-up-an-application-developers-controlled-distribution-adcd-in-ibm-zdt-v1"></a>Configuración de una instancia de Application Developers Controlled Distribution (ADCD) en IBM zD&T v1

Puede ejecutar un entorno de IBM Z Development and Test Environment (zD&T) en máquinas virtuales (VM) de Azure. Este entorno emula la arquitectura de IBM Z Series. Puede hospedar una variedad de sistemas operativos o instalaciones de Z Series (también denominadas instancias o paquetes de Z), que se ponen a disposición a través de conjuntos personalizados llamados Application Developers Controlled Distributions (ADCD) de IBM.

En este artículo se muestra cómo configurar una instancia de ADCD en un entorno de zD&T en Azure. Los ADCD crean implementaciones completas de sistema operativo de Z Series para entornos de desarrollo y pruebas que se ejecutan en zD&T.

Al igual que zD&T, los ADCD solo están disponibles para los socios y clientes de IBM, y son exclusivamente para fines de desarrollo y prueba. No deben usarse para entornos de producción. Hay disponibles varios paquetes de instalación de IBM para su descarga a través de [Passport Advantage](https://www.ibm.com/support/knowledgecenter/en/SSTQBD_12.0.0/com.ibm.zsys.rdt.guide.adcd.doc/topics/installation_ps.html) o [IBM PartnerWorld](https://www.ibm.com/partnerworld/public).

## <a name="prerequisites"></a>Prerrequisitos

- Suscripción a Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- El [entorno de zD&T][ibm-install-z] configurado anteriormente en Azure. En este artículo se supone que usa la misma imagen de VM de Ubuntu 16.04 creada anteriormente.

- Acceso a los medios de ADCD a través de IBM PartnerWorld o Passport Advantage.

- Un [servidor de licencias](https://www.ibm.com/support/knowledgecenter/en/SSTQBD_12.0.0/com.ibm.zsys.rdt.tools.user.guide.doc/topics/zdt_ee.html). Esto es necesario para ejecutar IBM zD&T. La forma en que se cree depende de cómo se autorice el software desde IBM:

  - **Servidor de licencias basado en hardware**: requiere un dispositivo de hardware USB que contenga la licencia Rational Token necesaria para acceder a todas las partes del software. Se la debe proporciona IBM.

  - **Servidor de licencias basado en software**: requiere que configure un servidor centralizado para la administración de las claves de licencia. Este método es preferible y requiere que configure las claves que recibe de IBM en el servidor de administración.

## <a name="download-the-installation-packages-from-passport-advantage"></a>Descargar los paquetes de instalación de Passport Advantage

Se requiere acceso a los medios de ADCD. En los pasos siguientes se supone que es cliente de IBM y puede usar Passport Advantage. Los partners de IBM pueden usar [IBM PartnerWorld](https://www.ibm.com/partnerworld/public).

> [!NOTE]
> En este artículo se supone que se usa un equipo con Windows para acceder a Azure Portal y descargar los medios de IBM. Si usa un equipo de escritorio Mac o Ubuntu, los comandos y el proceso para obtener el medio de IBM pueden diferir ligeramente.

1. Inicie sesión en [Passport Advantage](https://www.ibm.com/software/howtobuy/passportadvantage/paocustomer).

2. Seleccione **Software Downloads** y **Media Access**.

3. Seleccione **Program offering and agreement number** y haga clic en **Continue**.

4. Escriba la descripción de la pieza o el número de pieza y haga clic en **Finder**.

5. Si lo desea, haga clic en la lista por orden alfabético para mostrar y ver el producto por su nombre.

6. Seleccione **All Operating Systems** en el campo **Operating system**, y **All Languages** en el campo **Languages**. Luego, haga clic en **Go**.

7. Haga clic en **Select individual files** para expandir la lista y mostrar los medios individuales para descargar.

8. Verifique los paquetes que quiere descargar, seleccione **Download** y, luego, descargue los archivos en el directorio que desee.

## <a name="upload-the-adcd-packages"></a>Cargar los paquetes de ADCD

Ahora que tiene los paquetes, debe cargarlos en la VM en Azure.

1. En Azure Portal, inicie una sesión **ssh** con la VM de Ubuntu que creó. Vaya a la VM, seleccione la hoja **Introducción** hoja y, luego, seleccione **Conectar**.

2. Seleccione la pestaña **SSH** y copie el comando ssh en el Portapapeles.

3. Inicie sesión en la VM con las credenciales y el [cliente ssh ](../../../linux/use-remote-desktop.md) de su elección. En esta demostración se usan las extensiones de Linux para Windows 10, lo que agrega un shell de bash en el símbolo del sistema de Windows. PuTTY funciona igual de bien.

4. Cuando haya iniciado sesión, cree un directorio para cargar los paquetes de IBM. Tenga en cuenta Linux distingue mayúsculas de minúsculas. Por ejemplo, en esta demostración se supone que los paquetes se han cargado en:

    `/home/MyUserID/ZDT/adcd/nov2017/volumes`

5. Cargue los archivos con un cliente SSH, como [WinSCP](https://winscp.net/eng/index.php). Dado que SCP es parte de SSH, usa el puerto 22, que es el que usa SSH. Si el equipo local no es Windows, puede escribir el [comando scp](http://man7.org/linux/man-pages/man1/scp.1.html) en la sesión SSH.

6. Inicie la carga en el directorio de la VM de Azure que creó, que se convierte en el almacenamiento de imágenes para zD&T.

    > [!NOTE]
    > Asegúrese de que **ADCDTOOLS. XML** se incluya en la carga al directorio **home/MiIDdeUsuario/ZDT/adcd/nov2017**. Lo necesitará más adelante.

7. Espere a que se carguen los archivos, lo que puede tardar algún tiempo según la conexión a Azure.

8. Cuando las cargas se hayan completado, navegue hasta el directorio de volúmenes y descomprima todos los volúmenes **gz**:

    ```console
    gunzip \*.gz
    ```
    
![Explorador de archivos que muestra los volúmenes gz descomprimidos](media/01-gunzip.png)

## <a name="configure-the-image-storage"></a>Configurar el almacenamiento de imágenes

El siguiente paso es configurar zD&T para usar los paquetes cargados. El proceso del almacenamiento de imágenes dentro de zD&T le permite montar y usar las imágenes. Puede usar SSH o FTP.

1. Inicie **zDTServer**. Para ello, debe estar en el nivel raíz. Escriba los dos comandos siguientes en orden:
    ```console
    sudo su -
    /opt/ibm/zDT/bin/startServer
    ```
2. Anote la dirección URL que devuelve el comando y úsela para acceder al servidor web. Es similar a la siguiente:
     > https://(nombre o dirección IP de la VM):9443/ZDTMC/index.HTML
     >
     > Recuerde que el acceso web usa el puerto 9443. Use esto para iniciar sesión en el servidor web. El identificador de usuario para ZD&T es **zdtadmin** y la contraseña es **password**.

    ![Pantalla de bienvenida de IBM zD&T Enterprise Edition](media/02-welcome.png)

3. En la página **Quick Start**, en **Configure**, seleccione **Image Storage**.

     ![Pantalla de inicio rápido de IBM zD&T Enterprise Edition](media/03-quickstart.png)

4. En la página **Configure image storage**, seleccione **SSH File Transfer Protocol**.

5. Para **Host name**, escriba **Localhost** y la ruta del directorio donde cargó las imágenes. Por ejemplo, /home/MiIDdeUsuario/ZDT/adcd/nov2017/volumes.

6. Escriba el **id. de usuario** y la **contraseña** para la VM. No use el id. de usuario ni la contraseña de ZD&T.

7. Pruebe la conexión para asegurarse de tener acceso y, luego, seleccione **Save** para guardar la configuración.

## <a name="configure-the-target-environments"></a>Configurar los entornos de destino

El siguiente paso es configurar el entorno de destino zD&T. Este entorno emulado hospedado es donde se ejecutan las imágenes.

1. En la página **Quick Start**, en **Configure**, seleccione **Target environments**.

2. En la página **Configure target environments**, seleccione **Add Target**.

3. Seleccione **Linux**. IBM admite dos tipos de entornos, Linux y Cloud(OpenStack), pero esta demostración se ejecuta en Linux.

4. En la página **Add target environment**, para **Host name**, escriba **localhost**. Mantenga **SSH port** establecido en **22**.

5. En el cuadro **Target Environment label**, escriba una etiqueta, como **MyCICS.**

     ![Pantalla para agregar el entorno de destino](media/04-add-target.png)

## <a name="configure-adcd-and-deploy"></a>Configurar ADCD e implementar

Después de completar los pasos de configuración anteriores, debe configurar zD&T para usar los paquetes y el entorno de destino. Nuevamente, use el proceso del almacenamiento de imágenes dentro de zD&T, que le permite montar y usar las imágenes. Puede usar SSH o FTP.

1. En la página **Quick Start**, en **Configure**, seleccione **ADCD**. Aparece un conjunto de instrucciones, que le indica los pasos que debe seguir antes de que se pueda montar un paquete de ADCD. Esto explica por qué hemos nombrado al directorio de destino como lo hemos hecho.

2. Suponiendo que todas las imágenes se han cargado en los directorios correctos, haga clic en el vínculo **IMAGE from ADCD** que se muestra en la esquina inferior derecha (se muestra en el paso 7 de la captura de pantalla siguiente).

     ![Pantalla Configure ADCD de IBM zD&T Enterprise Edition](media/05-adcd.png)

## <a name="create-the-image"></a>Crear la imagen

Cuando se complete el paso de configuración anterior, aparecerá la página **Create an image using ADCD Components**.

1. Seleccione el volumen (noviembre de 2017 en este caso) para mostrar los distintos paquetes que hay en ese volumen.

2. Para esta demostración, seleccione **Customer Information Control System (CICS) - 5.3**.

3. En el cuadro **Image name**, escriba un nombre para la imagen, como **MyCICS Image**.

4. Seleccione el botón **Create Image** situado en la esquina inferior derecha.

     ![Pantalla Create an image using ADCD Components de IBM zD&T Enterprise Edition](media/06-adcd.png)

5. En la ventana que aparece, que le indica que la imagen se ha implementado correctamente, elija **Deploy images**.

6. En la página **Deploy an image to a target environment**, seleccione la imagen que ha creado en la página anterior (**MyCICS Image**) y el entorno de destino que creó anteriormente (**MyCICS**).

7. En la siguiente pantalla, proporcione sus credenciales para la VM (es decir, no la credencial ztadmin).

8. En el panel Properties, escriba el número de **Central processors (CPs)** , la cantidad de **System memory (GB)** y **deployment directory** para la imagen en ejecución. Puesto que esta es una demostración, mantenga un tamaño reducido.

9. Asegúrese de que la casilla **Automatically issue IPL command to z/OS after deploy** esté seleccionada.

     ![Pantalla Properties](media/07-properties.png)

10. Seleccione **Complete**.

11. Seleccione **Deploy Image** en la página **Deploy an image to a target environment**.

La imagen ahora se puede implementar y está lista para que un emulador de terminal 3270 la monte.

> [!NOTE]
> Si recibe un error que indica que no tiene suficiente espacio en disco, tenga en cuenta que la región requiere 151 Gb.

Felicidades. Ahora está ejecutando un entorno de sistema central de IBM en Azure.

## <a name="learn-more"></a>Más información

- [Mitos y verdades del sistema central](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/myths-and-facts)
- [IBM DB2 pureScale en Azure](../../../linux/ibm-db2-purescale-azure.md)
- [Solución de problemas](../../../troubleshooting/index.yml)
- [Demystifying mainframe to Azure migration](https://azure.microsoft.com/resources/demystifying-mainframe-to-azure-migration/) (Desmitificación de la migración del sistema central a Azure)

<!-- INTERNAL LINKS -->
[microfocus-get-started]: /microfocus/get-started.md
[microfocus-setup]: /microfocus/set-up-micro-focus-on-azure.md
[microfocus-demo]: /microfocus/demo.md
[ibm-get-started]: /ibm/get-started.md
[ibm-install-z]: install-ibm-z-environment.md
[ibm-demo]: /ibm/demo.md
