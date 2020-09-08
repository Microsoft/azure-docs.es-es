---
title: Instalación de MongoDB en una máquina virtual de Windows en Azure
description: Aprenda a instalar MongoDB en una máquina virtual de Azure en la que se ejecuta Windows Server 2012 R2 y que se ha creado con el modelo clásico de implementación de Resource Manager.
documentationcenter: ''
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: how-to
ms.date: 12/15/2017
ms.author: cynthn
ms.openlocfilehash: d02fa8fa23b587db06f3d2d1e08f0a8565471123
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510426"
---
# <a name="install-and-configure-mongodb-on-a-windows-vm-in-azure"></a>Instalación y configuración de MongoDB en una máquina virtual Windows en Azure
[MongoDB](https://www.mongodb.org) es una conocida base de datos NoSQL de código abierto y alto rendimiento. Este artículo le guía por la instalación y configuración de MongoDB en una máquina virtual (VM) Windows Server 2016 en Azure. También es posible [instalar MongoDB en una máquina virtual Linux en Azure](../linux/install-mongodb.md).

## <a name="prerequisites"></a>Requisitos previos
Antes de instalar y configurar MongoDB, es preciso crear una máquina virtual y lo ideal sería agregarle un disco de datos. Consulte los artículos siguientes para crear una máquina virtual y agregar un disco de datos:

* Cree una máquina virtual de Windows Server mediante [Azure Portal](quick-create-portal.md) o [Azure PowerShell](quick-create-powershell.md).
* Adjunte un disco de datos a una máquina virtual de Windows Server mediante [Azure Portal](attach-managed-disk-portal.md) o [Azure PowerShell](attach-disk-ps.md).

Para empezar a instalar y configurar MongoDB, [inicie sesión en la máquina virtual con Windows Server](connect-logon.md) mediante Escritorio remoto.

## <a name="install-mongodb"></a>Instalación de MongoDB
> [!IMPORTANT]
> Las características de seguridad de MongoDB, como la vinculación de direcciones IP y la autenticación, no se encuentran habilitadas de forma predeterminada. Las características de seguridad deben habilitarse antes de implementar MongoDB en un entorno de producción. Para más información, consulte [MongoDB Security and Authentication](https://www.mongodb.org/display/DOCS/Security+and+Authentication) (Seguridad y autenticación de MongoDB).


1. Después de conectarse a la máquina virtual mediante Escritorio remoto, abra Internet Explorer desde la barra de tareas.
2. Seleccione **Usar la configuración recomendada de compatibilidad, privacidad y seguridad** la primera vez que se abra Internet Explorer y haga clic en **Aceptar**.
3. De forma predeterminada está habilitada la configuración de seguridad mejorada de Internet Explorer. Agregue el sitio web de MongoDB a la lista de sitios permitidos:

   * Seleccione el icono **Herramientas** de la esquina superior derecha.
   * En **Opciones de Internet**, seleccione la pestaña **Seguridad** y, luego, el icono **Sitios de confianza**.
   * Haga clic en el botón **Sitios**. Agregue *https://\*.mongodb.com* a la lista de sitios de confianza y, luego, cierre el cuadro de diálogo.

     ![Configurar las opciones de seguridad de Internet Explorer](./media/install-mongodb/configure-internet-explorer-security.png)
4. Vaya a la página de [descargas de MongoDB](https://www.mongodb.com/downloads) (https://www.mongodb.com/downloads).
5. Si es necesario,, seleccione la edición **Community Server** y, luego, seleccione la versión estable más reciente para *Windows Server 2008 R2 de 64 bits y posterior*. Para descargar el instalador, haga clic en **DOWNLOAD (msi)** (DESCARGAR).

    ![Descargue el instalador de MongoDB](./media/install-mongodb/download-mongodb.png)

    Cuando se haya completado la descarga, ejecute al instalador.
6. Lee y acepte los términos de la licencia. Cuando se le pida, seleccione **Complete** (Completar) instalación.
7. Si quiere, puede elegir también instalar Compass, una interfaz gráfica para MongoDB.
8. En la última pantalla, haga clic en **Install** (Instalar).

## <a name="configure-the-vm-and-mongodb"></a>Configuración de la máquina virtual y MongoDB
1. El instalador de MongoDB no actualiza las variables path. Sin la ubicación `bin` de MongoDB en la variable path, será preciso especificar la ruta de acceso completa cada vez que se use un archivo ejecutable de MongoDB. Para agregar la ubicación a la variable path:

   * Haga clic con el botón derecho en el menú **Inicio** y seleccione **Sistema**.
   * Haga clic en la pestaña **Configuración avanzada del sistema** y luego en **Variables de entorno**.
   * En **Variables del sistema**, seleccione **Path**, y, luego, haga clic en **Editar**.

     ![Configurar variables PATH](./media/install-mongodb/configure-path-variables.png)

     Agregue la ruta de acceso a la carpeta `bin` de MongoDB. MongoDB se suele instalar en *C:\Archivos de programa\MongoDB*. Compruebe la ruta de instalación en la máquina v. En el ejemplo siguiente se agrega la ubicación de instalación de MongoDB predeterminada a la variable `PATH`:

     ```
     ;C:\Program Files\MongoDB\Server\3.6\bin
     ```

     > [!NOTE]
     > No olvide agregar el signo inicial de punto y coma (`;`) para indicar que va a agregar una ubicación a la variable `PATH`.

2. Cree los directorios de registro y datos de MongoDB en el disco de datos. En el menú **Inicio**, seleccione **Símbolo del sistema**. En los ejemplos siguientes los directorios se crean en la unidad F:

    ```
    mkdir F:\MongoData
    mkdir F:\MongoLogs
    ```
3. Inicie una instancia de MongoDB con el siguiente comando y ajuste la ruta de acceso a los directorios de registro y de datos en consecuencia:

    ```
    mongod --dbpath F:\MongoData\ --logpath F:\MongoLogs\mongolog.log
    ```

    MongoDB puede tardar varios minutos en asignar los archivos de diario y comenzar la escucha de conexiones. Todos los mensajes de registro se dirigen al archivo *F:\MongoLogs\mongolog.log* cuando el servidor `mongod.exe` se inicia y asigna los archivos de diario.

   > [!NOTE]
   > El símbolo del sistema permanece centrado en esta tarea mientras se ejecuta la instancia de MongoDB. Deje la ventana de símbolo del sistema abierta para continuar con la ejecución de MongoDB. O bien, instale MongoDB como servicio, como se detalla en el paso siguiente.

4. Para una experiencia más sólida de MongoDB, instale `mongod.exe` como servicio. La creación de un servicio significa que no es preciso dejar un símbolo de sistema en ejecución cada vez que se desee utilizar MongoDB. Cree el servicio como se indica a continuación y ajuste la ruta de acceso a los directorios de datos y de registro según en consecuencia:

    ```
    mongod --dbpath F:\MongoData\ --logpath F:\MongoLogs\mongolog.log --logappend  --install
    ```

    El comando anterior crea un servicio denominado MongoDB con la descripción "Mongo DB". También se especifican los parámetros siguientes:

   * La opción `--dbpath` especifica la ubicación del directorio de datos.
   * La opción `--logpath` debe usarse para especificar un archivo de registro, ya que el servicio en ejecución no dispone de una ventana de comandos que muestre la salida.
   * La opción `--logappend` especifica que un reinicio del servicio provoca que la salida se anexe al archivo de registro existente.

   Para iniciar el servicio de MongoDB, ejecute el siguiente comando:

    ```
    net start MongoDB
    ```

    Para más información acerca de cómo crear el servicio MongoDB, consulte [Configure a Windows Service for MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/#mongodb-as-a-windows-service) (Configuración de un servicio de Windows para MongoDB).

## <a name="test-the-mongodb-instance"></a>Prueba de la instancia de MongoDB
Con MongoDB ejecutándose como una instancia individual o instalado como un servicio, ya puede comenzar la creación y uso de las bases de datos. Para iniciar el shell administrativo de MongoDB, abra otra ventana del símbolo del sistema en el menú **Inicio** y escriba el siguiente comando:

```
mongo
```

Puede enumerar las bases de datos con el comando `db`. Inserte algunos datos como sigue:

```
db.foo.insert( { a : 1 } )
```

Busque datos como sigue:

```
db.foo.find()
```

La salida es similar a la del ejemplo siguiente:

```
{ "_id" : "ObjectId("57f6a86cee873a6232d74842"), "a" : 1 }
```

Salga de la consola de `mongo` como se indica a continuación:

```
exit
```

## <a name="configure-firewall-and-network-security-group-rules"></a>Configuración de reglas del grupo de seguridad de red y del firewall
Ahora que MongoDB ya está instalado y ejecutándose, abra un puerto en Firewall de Windows para poder conectarse de forma remota a MongoDB. Para crear una nueva regla de entrada para permitir el puerto TCP 27017, abra un símbolo del sistema administrativo de PowerShell y escriba el siguiente comando:

```powershell
New-NetFirewallRule `
    -DisplayName "Allow MongoDB" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 27017 `
    -Action Allow
```

También puede crear la regla mediante la herramienta de administración gráfica **Firewall de Windows con seguridad avanzada**. Cree una nueva regla de entrada para permitir el puerto TCP 27017.

Si es necesario, cree una regla de grupo de seguridad de red para permitir el acceso a MongoDB desde fuera de la subred de la red virtual de Azure existente. Las reglas del grupo de seguridad de red se pueden crear mediante [Azure Portal ](nsg-quickstart-portal.md) o [Azure PowerShell](nsg-quickstart-powershell.md). Igual que sucede con las reglas de Firewall de Windows, permita el puerto TCP 27017 en la interfaz de red virtual de la máquina virtual con MongoDB.

> [!NOTE]
> El puerto TCP 27017 es el puerto predeterminado que usa MongoDB. Para cambiarlo, use el parámetro `--port` al iniciar `mongod.exe` manualmente o desde un servicio. Si cambia el puerto, asegúrese de actualizar las reglas de Firewall de Windows y del grupo de seguridad de red en los pasos anteriores.


## <a name="next-steps"></a>Pasos siguientes
En este tutorial ha aprendido a instalar y configurar MongoDB en una máquina virtual Windows. Ahora puede acceder a MongoDB en una máquina virtual Windows siguiendo los temas avanzados de la [documentación de MongoDB](https://docs.mongodb.com/manual/).

