---
title: Creación de un punto de conexión privado para una conexión segura
titleSuffix: Azure Cognitive Search
description: Configuración de un punto de conexión privado en una red virtual para una conexión segura a un servicio Azure Cognitive Search
manager: nitinme
author: mrcarter8
ms.author: mcarter
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/11/2020
ms.openlocfilehash: 0cfa7b63d1ce9dd4d9b40cd0eedac247f9c56437
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935762"
---
# <a name="create-a-private-endpoint-for-a-secure-connection-to-azure-cognitive-search"></a>Creación de un punto de conexión privado para una conexión segura a Azure Cognitive Search

En este artículo, usará Azure Portal para crear una nueva instancia del servicio Azure Cognitive Search a la que no se pueda obtener acceso a través de Internet. A continuación, configurará una máquina virtual de Azure en la misma red virtual y la usará para obtener acceso al servicio de búsqueda a través de un punto de conexión privado.

Los puntos de conexión privados se proporcionan mediante [Azure Private Link](../private-link/private-link-overview.md), como un servicio independiente. Para más información sobre los costos, consulte la [página de precios](https://azure.microsoft.com/pricing/details/private-link/).

> [!Important]
> El soporte técnico del punto de conexión privado de Azure Cognitive Search se puede configurar con Azure Portal o la [API REST de administración, versión 2020-03-13](/rest/api/searchmanagement/). Cuando el punto de conexión de servicio es privado, se deshabilitan algunas características del portal. Podrá ver y administrar información de nivel de servicio, pero, por motivos de seguridad, se ha restringido el acceso del portal a los datos del índice y de los distintos componentes de este servicio, como índice, indizador y definiciones del conjunto de aptitudes.

## <a name="why-use-a-private-endpoint-for-secure-access"></a>¿Por qué usar un punto de conexión privado para obtener un acceso seguro?

Los [puntos de conexión privados](../private-link/private-endpoint-overview.md) para Azure Cognitive Search permiten a un cliente de una red virtual obtener acceso de forma segura a los datos de un índice de búsqueda a través de un [vínculo privado](../private-link/private-link-overview.md). El punto de conexión privado usa una dirección IP del [espacio de direcciones de la red virtual](../virtual-network/private-ip-addresses.md) para el servicio de búsqueda. El tráfico de red entre el cliente y el servicio de búsqueda atraviesa la red virtual y un vínculo privado de la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet. Para obtener una lista de otros servicios de PaaS que admiten Private Link, compruebe la [sección de disponibilidad](../private-link/private-link-overview.md#availability) en la documentación del producto.

Los puntos de conexión privados para su servicio de búsqueda le permiten:

- Bloquear todas las conexiones del punto de conexión público para su servicio de búsqueda.
- Aumentar la seguridad de la red virtual, ya que permite bloquear la filtración de datos de la red virtual.
- Conectarse de forma segura al servicio de búsqueda desde las redes locales que se conectan a la red virtual mediante [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) o [instancias de ExpressRoute](../expressroute/expressroute-locations.md) con emparejamiento privado.

## <a name="create-the-virtual-network"></a>Crear la red virtual

En esta sección, va a crear una red virtual y una subred para hospedar la máquina virtual que se usará para obtener acceso al punto de conexión privado del servicio de búsqueda.

1. En la pestaña Inicio de Azure Portal, seleccione **Crear un recurso** > **Redes** > **Red virtual**.

1. En **Creación de una red virtual**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Suscripción | Seleccione su suscripción.|
    | Resource group | Seleccione **Crear nuevo**, escriba *myResourceGroup* y, después, seleccione **Aceptar**. |
    | Nombre | Escriba *MyVirtualNetwork*. |
    | Region | Seleccione la región que quiera. |
    |||

1. En el resto de la configuración, deje los valores predeterminados. Haga clic en **Revisar y crear** y, a continuación, en **Crear**.

## <a name="create-a-search-service-with-a-private-endpoint"></a>Creación de un servicio de búsqueda con un punto de conexión privado

En esta sección, creará un nuevo servicio Azure Cognitive Search con un punto de conexión privado. 

1. En la parte superior izquierda de la pantalla en Azure Portal, seleccione **Crear un recurso** > **Web** > **Azure Cognitive Search**.

1. En **Nuevo servicio de búsqueda: Conceptos básicos**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | **DETALLES DEL PROYECTO** | |
    | Suscripción | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**. Lo creó en la sección anterior.|
    | **DETALLES DE INSTANCIA** |  |
    | URL | Escriba un nombre único. |
    | Location | Seleccione la región que quiera. |
    | Plan de tarifa | Seleccione la opción para **cambiar el plan de tarifa** y elija el nivel de servicio que quiera. (No se admite en el nivel **gratuito**. Debe ser **básico** o posterior). |
    |||
  
1. Seleccione **Siguiente: Escalado**.

1. Deje los valores tal como están y seleccione **Siguiente: Redes**.

1. En **Nuevo servicio de búsqueda: Redes**, seleccione **Privado** para **Conectividad de punto de conexión (datos)** .

1. En **Nuevo servicio de búsqueda: Redes**, seleccione **+ Agrear** en **Punto de conexión privado**. 

1. En **Crear un punto de conexión privado**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Suscripción | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**. Lo creó en la sección anterior.|
    | Location | Seleccione **Oeste de EE. UU.**|
    | Nombre | Escriba *myPrivateEndpoint*.  |
    | Recurso secundario de destino | Deje el valor predeterminado **searchService**. |
    | **REDES** |  |
    | Virtual network  | Seleccione *MyVirtualNetwork* en el grupo de recursos *myResourceGroup*. |
    | Subnet | Seleccione *mySubnet*. |
    | **INTEGRACIÓN DE DNS PRIVADO** |  |
    | Integración con una zona DNS privada  | Deje el valor predeterminado **Sí**. |
    | Zona DNS privada  | Deje el valor predeterminado ** (New) privatelink.blob.core.windows.net**. |
    |||

1. Seleccione **Aceptar**. 

1. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración. 

1. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**. 

1. Una vez completado del aprovisionamiento de su nuevo servicio, vaya al recurso que acaba de crear.

1. Seleccione **Claves** en el menú de contenido de la izquierda.

1. Copie la **clave de administrador principal** para más adelante, al conectarse al servicio.

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**.

1. En **Creación de una máquina virtual: conceptos básicos**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | **DETALLES DEL PROYECTO** | |
    | Suscripción | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**. Lo creó en la sección anterior.  |
    | **DETALLES DE INSTANCIA** |  |
    | Nombre de la máquina virtual | Escriba *myVm*. |
    | Region | Seleccione **Oeste de EE. UU.** o la región que esté usando. |
    | Opciones de disponibilidad | Deje el valor predeterminado **No se requiere redundancia de la infraestructura**. |
    | Imagen | Seleccione **Windows Server 2019 Datacenter**. |
    | Size | Deje el valor predeterminado **Estándar DS1 v2**. |
    | **CUENTA DE ADMINISTRADOR** |  |
    | Nombre de usuario | Escriba un nombre de usuario de su elección. |
    | Contraseña | Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    | Confirm Password | Vuelva a escribir la contraseña. |
    | **REGLAS DE PUERTO DE ENTRADA** |  |
    | Puertos de entrada públicos | Deje el valor predeterminado **Permitir los puertos seleccionados**. |
    | Selección de puertos de entrada | Deje el valor predeterminado **RDP (3389)** . |
    | **AHORRE DINERO** |  |
    | ¿Ya tiene una licencia de Windows? | Deje el valor predeterminado **No**. |
    |||

1. Seleccione **Siguiente: Discos**.

1. En **Creación de una máquina virtual: Discos**, deje los valores predeterminados y seleccione **Siguiente: Redes**.

1. En **Creación de una máquina virtual: Redes**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Virtual network | Deje el valor predeterminado **MyVirtualNetwork**.  |
    | Espacio de direcciones | Deje el valor predeterminado **10.1.0.0/24**.|
    | Subnet | Deje el valor predeterminado **mySubnet (10.1.0.0/24)** .|
    | Dirección IP pública | Deje el valor predeterminado **(new) myVm-ip**. |
    | Puertos de entrada públicos | Seleccione **Permitir los puertos seleccionados**. |
    | Selección de puertos de entrada | Seleccione **HTTP** y **RDP**.|
    ||

1. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración.

1. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**. 


## <a name="connect-to-the-vm"></a>Conexión a la máquina virtual

Descargue y, a continuación, conéctese a la máquina virtual *myVm* de la siguiente manera:

1. En la barra de búsqueda del portal, escriba *myVm*.

1. Seleccione el botón **Conectar**. Después de seleccionar el botón **Conectar**, se abre **Conectar a máquina virtual**.

1. Seleccione **Descargar archivo RDP**. Azure crea un archivo de Protocolo de Escritorio remoto ( *.rdp*) y lo descarga en su equipo.

1. Abra el archivo downloaded.rdp*.

    1. Cuando se le pida, seleccione **Conectar**.

    1. Escriba el nombre de usuario y la contraseña que especificó al crear la VM.

        > [!NOTE]
        > Es posible que tenga que seleccionar **Más opciones** > **Usar otra cuenta** para especificar las credenciales que escribió al crear la máquina virtual.

1. Seleccione **Aceptar**.

1. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe una advertencia de certificado, seleccione **Sí** o **Continuar**.

1. Una vez que aparezca el escritorio de la máquina virtual, minimícelo para volver a su escritorio local.  


## <a name="test-connections"></a>Prueba de conexiones

En esta sección, comprobará el acceso de la red privada al servicio de búsqueda y se conectará de forma privada a este mediante el punto de conexión privado.

Cuando el punto de conexión del servicio de búsqueda es privado, se deshabilitan algunas características del portal. Podrá ver y administrar la configuración del nivel de servicio, pero, por motivos de seguridad, se ha restringido el acceso del portal a los datos del índice y de los distintos componentes de este servicio, como el índice, el indexador y las definiciones del conjunto de aptitudes.

1. En el Escritorio remoto de *myVm*, abra PowerShell.

1. Escriba "nslookup [nombre del servicio de búsqueda].search.windows.net".

    Recibirá un mensaje similar a este:
    ```azurepowershell
    Server:  UnKnown
    Address:  168.63.129.16
    Non-authoritative answer:
    Name:    [search service name].privatelink.search.windows.net
    Address:  10.0.0.5
    Aliases:  [search service name].search.windows.net
    ```

1. En la máquina virtual, conéctese al servicio de búsqueda y cree un índice. Puede seguir este [inicio rápido](search-get-started-postman.md) para crear un nuevo índice de búsqueda en su servicio de Postman mediante la API de REST. La configuración de solicitudes de Postman requiere el punto de conexión del servicio de búsqueda (https://[nombre del servicio de búsqueda].search.windows.net) y la clave de API de administración.

1. La finalización del inicio rápido de la máquina virtual es su confirmación de que el servicio es totalmente operativo.

1. Cierre la conexión de Escritorio remoto a *myVM*. 

1. Para comprobar que su servicio no es accesible en un punto de conexión público, abra Postman en su estación de trabajo local e intente realizar las primeras tareas del inicio rápido. Si recibe un error que indica que el servidor remoto no existe, habrá configurado correctamente un punto de conexión privado para su servicio de búsqueda.

## <a name="clean-up-resources"></a>Limpieza de recursos 
Cuando haya terminado mediante el punto de conexión privado, el servicio de búsqueda y la máquina virtual, elimine el grupo de recursos y todos los recursos que contiene:
1. Escriba *myResourceGroup* en el cuadro **Buscar** de la parte superior del portal y seleccione *myResourceGroup* en los resultados de la búsqueda. 
1. Seleccione **Eliminar grupo de recursos**. 
1. Escriba *myResourceGroup* en **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS** y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes
En este artículo, creó una máquina virtual en una red virtual y un servicio de búsqueda con un punto de conexión privado. Se conectó a la máquina virtual desde Internet y se comunicó de forma segura con el servicio de búsqueda mediante Private Link. Para más información sobre el punto de conexión privado, consulte  [¿Qué es un punto de conexión privado de Azure? ](../private-link/private-endpoint-overview.md).