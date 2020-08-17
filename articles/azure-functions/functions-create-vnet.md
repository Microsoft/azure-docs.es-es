---
title: Integración de Azure Functions con una red virtual de Azure
description: Tutorial paso a paso donde se explica cómo conectarse a una función en una red virtual de Azure
ms.topic: article
ms.date: 4/23/2020
ms.openlocfilehash: f50c923104fdfcf26f400f20f0de66a82eb3d245
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387530"
---
# <a name="tutorial-integrate-functions-with-an-azure-virtual-network"></a>Tutorial: integración de Functions con una red virtual de Azure

En este tutorial se explica cómo se utiliza Azure Functions para conectarse a recursos de una red virtual de Azure. Creará una función que tenga acceso a Internet y a una máquina virtual que ejecute WordPress en una red virtual.

> [!div class="checklist"]
> * Creación de una aplicación de funciones en el plan Premium
> * Implementación de un sitio de WordPress en la máquina virtual en una red virtual
> * Conexión de la aplicación de funciones a la red virtual
> * Creación de un proxy de función para acceder a recursos de WordPress
> * Solicitud de un archivo de WordPress desde dentro de la red virtual

## <a name="topology"></a>Topología

El siguiente diagrama muestra la arquitectura de la solución que se ha creado:

 ![Interfaz de usuario para la integración de red virtual](./media/functions-create-vnet/topology.png)

La ejecución de Functions en el plan Premium tiene las mismas capacidades de hospedaje que las aplicaciones web en Azure App Service, que incluye la característica de integración de red virtual. Para obtener más información sobre la integración de red virtual, incluida la solución de problemas y la configuración avanzada, vea [Integración de su aplicación con una red virtual de Azure](../app-service/web-sites-integrate-with-vnet.md).

## <a name="prerequisites"></a>Prerrequisitos

En este tutorial, es importante que comprenda el direccionamiento IP y las subredes. Puede empezar con [este artículo que trata los aspectos básicos del direccionamiento y las subredes](https://support.microsoft.com/help/164015/understanding-tcp-ip-addressing-and-subnetting-basics). Hay muchos más artículos y vídeos disponibles en línea.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-a-function-app-in-a-premium-plan"></a>Creación de una aplicación de funciones en un plan Premium

En primer lugar, cree una aplicación de funciones en el [plan Premium]. Este plan ofrece escala sin servidor, al mismo tiempo que admite la integración de red virtual.

[!INCLUDE [functions-premium-create](../../includes/functions-premium-create.md)]  

Puede anclar la aplicación de funciones al panel seleccionando el icono de anclaje situado en la esquina superior derecha. Dicho anclaje facilita la vuelta a esta aplicación de funciones una vez creada la máquina virtual.

## <a name="create-a-vm-inside-a-virtual-network"></a>Creación de una máquina virtual dentro de una red virtual

Luego, cree una máquina virtual preconfigurada que ejecute WordPress en una red virtual ([WordPress LEMP7 Max Performance](https://jetware.io/appliances/jetware/wordpress4_lemp7-170526/profile?us=azure) por Jetware). Se usa una máquina virtual de WordPress debido a su bajo costo y la comodidad. Este mismo escenario funciona con cualquier recurso en una red virtual, como las API REST, App Service Environment y otros servicios de Azure. 

1. En el portal, elija **+ Crear un recurso** en el panel de navegación izquierdo, en el tipo de campo de búsqueda `WordPress LEMP7 Max Performance`, y presione Entrar.

1. Elija **Wordpress LEMP Max Performance** en los resultados de búsqueda. Seleccione un plan de software de **Wordpress LEMP Max Performance para CentOS** como el **Plan de Software** y seleccione **Crear**.

1. En la pestaña **Básico**, utilice la configuración de máquina virtual que se especifica en la tabla debajo de la imagen:

    ![Pestaña Básico para crear una máquina virtual](./media/functions-create-vnet/create-vm-1.png)

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Suscripción** | Su suscripción | Suscripción en la que se han creado los recursos. | 
    | **[Grupo de recursos](../azure-resource-manager/management/overview.md)**  | myResourceGroup | Elija `myResourceGroup` o el grupo de recursos que ha creado con la aplicación de funciones. Con el uso del mismo grupo de recursos para la aplicación de funciones, la máquina virtual de WordPress y el plan de hospedaje, resulta más sencillo limpiar los recursos cuando haya acabado este tutorial. |
    | **Nombre de la máquina virtual** | Wordpress de red virtual | El nombre de la máquina virtual debe ser único en el grupo de recursos |
    | **[Región](https://azure.microsoft.com/regions/)** | (Europa) Oeste de Europa | Elija una región cerca de usted o cerca de las funciones que tienen acceso a la máquina virtual. |
    | **Tamaño** | B1s | Elija **Cambiar tamaño** y luego seleccione la imagen estándar B1s, que tiene 1 vCPU y 1 GB de memoria. |
    | **Tipo de autenticación** | Contraseña | Para usar la autenticación de contraseña, también debe especificar un **Nombre de usuario**, una **Contraseña** segura y luego **Confirmar contraseña**. En este tutorial, no es necesario iniciar sesión en la máquina virtual a menos que deba solucionar algún problema. |

1. Elija la pestaña **Redes** y, en Configurar redes virtuales, seleccione **Crear nueva**.

1. En **Crear red virtual**, use la configuración de la tabla que aparece debajo de la imagen:

    ![Pestaña Redes de creación de máquina virtual](./media/functions-create-vnet/create-vm-2.png)

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Nombre** | myResourceGroup-vnet | Puede usar el nombre predeterminado generado para la red virtual. |
    | **Intervalo de direcciones** | 10.10.0.0/16 | Use un rango de dirección único para la red virtual. |
    | **Nombre de subred** | Tutorial-Net | Nombre de la subred. |
    | **Rango de dirección** (subred) | 10.10.1.0/24   | El tamaño de la subred define la cantidad de interfaces que se pueden agregar a la subred. Esta subred la usa el sitio de WordPress.  Una subred de `/24` proporciona 254 direcciones de host. |

1. Seleccione **Aceptar** para crear la red virtual.

1. Vuelva a la pestaña **Redes** y, en la **IP pública**, elija **Ninguna**.

1. Elija la pestaña **Administración**, después, en **Cuenta de almacenamiento de diagnóstico**, elija la cuenta de almacenamiento que se ha creado con la aplicación de funciones.

1. Seleccione **Revisar + crear**. Una vez completada la validación, seleccione **Crear**. El proceso de creación de la máquina virtual tarda unos minutos. La máquina virtual creada solo puede tener acceso a la red virtual.

1. Una vez creada la máquina virtual, elija **Ir al recurso** para ver la página de la nueva máquina virtual y luego, en **Configuración**, elija **Redes**.

1. Compruebe que no haya ninguna **IP pública**. Tome nota de la **IP privada**, que se utiliza para conectarse a la máquina virtual desde la aplicación de funciones.

    ![Configuración de Redes en la máquina virtual](./media/functions-create-vnet/vm-networking.png)

Ahora, tendrá un sitio de WordPress implementado por completo dentro de la red virtual. Este sitio no será accesible desde la red pública de Internet.

## <a name="connect-your-function-app-to-the-virtual-network"></a>Conexión de la aplicación de funciones a la red virtual

Gracias a un sitio de WordPress que se ejecuta en una máquina virtual en una red virtual, ahora puede conectar la aplicación de funciones a esa red virtual.

1. En la nueva aplicación de funciones, seleccione **Redes** en el menú de la izquierda.

1. En **Integración de red virtual**, seleccione **Haga clic aquí para configurar**.

    :::image type="content" source="./media/functions-create-vnet/networking-0.png" alt-text="Elección de una red en la aplicación de funciones":::

1. En la página de **Integración con red virtual**, seleccione **Agregar VNET**.

    :::image type="content" source="./media/functions-create-vnet/networking-2.png" alt-text="Adición de la versión preliminar de integración con red virtual":::

1. En **Estado de la característica de red**, use la configuración de la tabla que aparece debajo de la imagen:

    ![Definición de la red virtual de la aplicación de funciones](./media/functions-create-vnet/networking-3.png)

    | Configuración      | Valor sugerido  | Descripción      |
    | ------------ | ---------------- | ---------------- |
    | **Virtual Network** | MyResourceGroup-vnet | Esta red virtual es la que se ha creado anteriormente. |
    | **Subred** | Creación de una subred nueva | Cree una subred en la red virtual para aplicación de funciones que se va a usar. La integración de red virtual se debe configurar para usar una subred vacía. No importa que las funciones usen una subred diferente a la máquina virtual. La red virtual enruta automáticamente el tráfico entre las dos subredes. |
    | **Nombre de subred** | Function-Net | Nombre de la nueva subred. |
    | **Bloque de direcciones de la red virtual** | 10.10.0.0/16 | Elija el mismo bloque de direcciones que ha usado el sitio de WordPress. Solo debe tener un bloque de direcciones definido. |
    | **Intervalo de direcciones** | 10.10.2.0/24   | El tamaño de la subred restringe el número total de instancias a las que la aplicación de funciones del plan Premium puede escalar horizontalmente. En este ejemplo se usa una subred de `/24` con 254 direcciones disponibles en el host. Esta subred tiene un aprovisionamiento excesivo, pero es fácil de calcular. |

1. Seleccione **Aceptar** para agregar la subred. Cierre las páginas de **Integración con red virtual** y de **Estado de característica de red** para volver a la página de aplicación de funciones.

Ahora la aplicación de funciones puede tener acceso a la red virtual donde se ejecuta el sitio de WordPress. Luego, use [Azure Functions Proxies](functions-proxies.md) para devolver un archivo desde el sitio de WordPress.

## <a name="create-a-proxy-to-access-vm-resources"></a>Creación de un proxy para acceder a los recursos de máquina virtual

Con la integración de red virtual habilitada, puede crear a un proxy en la aplicación de funciones para reenviar solicitudes a la máquina virtual que se ejecuta en la red virtual.

1. En la aplicación de funciones, seleccione **Proxies** en el menú de la izquierda y, a continuación, seleccione **Agregar**. Use la configuración de proxy de la tabla debajo de la imagen:

    :::image type="content" source="./media/functions-create-vnet/create-proxy.png" alt-text="Definición de la configuración del proxy":::

    | Configuración  | Valor sugerido  | Descripción      |
    | -------- | ---------------- | ---------------- |
    | **Nombre** | Plant | El nombre puede ser cualquier valor. Sirve para identificar al proxy |
    | **Plantilla de ruta** | /plant | Ruta que se asigna a un recurso de máquina virtual. |
    | **Dirección URL de back-end** | http://<LA_IP_DE_SU_VM>/wp-content/themes/twentyseventeen/assets/images/header.jpg | Reemplace `<YOUR_VM_IP>` por la dirección IP de la máquina virtual de WordPress que se ha creado anteriormente. Esta asignación devuelve un único archivo desde el sitio. |

1. Seleccione **Crear** para agregar el proxy a la aplicación de funciones.

## <a name="try-it-out"></a>Prueba

1. En el explorador, intente obtener acceso a la dirección URL que se ha usado como la **dirección URL de back-end**. Según lo previsto, la solicitud agota el tiempo de espera. Se produce un tiempo de expiración porque el sitio de WordPress se conecta únicamente a la red virtual y no a Internet.

1. Copia el valor de la **dirección URL del proxy** desde el nuevo proxy y péguelo en la barra de direcciones del explorador. La imagen devuelta corresponde al sitio de WordPress que se ejecuta en la red virtual.

    ![Archivo de imagen Plant devuelto desde el sitio de WordPress.](./media/functions-create-vnet/plant.png)

La aplicación de funciones está conectada a Internet y a la red virtual. El proxy recibe una solicitud a través de la red pública de Internet y, después, actúa como un proxy HTTP y reenvía la solicitud a la red virtual conectada. Luego, el proxy transmite la respuesta de vuelta de forma pública a través de Internet.

[!INCLUDE [clean-up-section-portal](../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, el sitio de WordPress actúa como una API a la que se llama mediante el uso de un proxy en la aplicación de funciones. Este escenario representa un buen tutorial porque es fácil de configurar y visualizar. Puede usar cualquier otra API implementada en una red virtual. También se podría haber creado una función con código que llamara a las API implementadas en la red virtual. Un escenario más realista sería una función que usa las API de cliente de datos para llamar a una instancia de SQL Server implementada en la red virtual.

Las funciones que se ejecutan en un plan Premium comparten la misma infraestructura subyacente de App Service que las aplicaciones web en los planes PremiumV2. Toda la documentación de las [aplicaciones web en Azure App Service](../app-service/overview.md) se aplica a las funciones del plan Premium.

> [!div class="nextstepaction"]
> [Más información sobre las opciones de redes en Functions](./functions-networking-options.md)

[Plan Premium]: functions-scale.md#premium-plan
