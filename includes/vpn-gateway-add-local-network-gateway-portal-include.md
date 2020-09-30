---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 09/17/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: bc9e18427bb3b8094911d2ac7f285d271ecd1a21
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "91025304"
---
1. En el menú de [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso**.

   ![Recurso](./media/vpn-gateway-add-local-network-gateway-portal-include/azure-portal-create-resource.png)
2. En el campo **Buscar en Marketplace**, escriba **Puerta de enlace de red local** y presione **Entrar** para comenzar la búsqueda. Se devolverá una lista de resultados. Haga clic en **Puerta de enlace de red local** y haga clic en el botón **Crear** para abrir la página **Crear puerta de enlace de red local**.

   :::image type="content" source="./media/vpn-gateway-add-local-network-gateway-portal-include/create-local-network-gateway-ip.png" alt-text="Creación de una puerta de enlace de red local con una dirección IP":::

3. En la página **Crear puerta de enlace de red local**, especifique los valores de la puerta de enlace de red local.

   - **Nombre:** especifique el nombre del objeto de puerta de enlace de red local.
   - **Punto de conexión:** Seleccione el tipo de punto de conexión para el dispositivo VPN local: **dirección IP** o **FQDN (nombre de dominio completo)** .
      - **Dirección IP**: si tiene asignada una dirección IP pública estática de su proveedor de servicios de Internet para el dispositivo VPN, seleccione la opción Dirección IP y rellene la dirección IP como se indica en el ejemplo. Esta es la dirección IP pública del dispositivo VPN al que desea que Azure VPN Gateway se conecte. Si no tiene la dirección IP en este momento, puede usar los valores que se muestran en el ejemplo, pero deberá volver y reemplazar la dirección IP del marcador de posición por la dirección IP pública de su dispositivo VPN. Si no lo hace, Azure no podrá conectarse.
      - **Nombre de dominio completo**: si tiene una dirección IP pública dinámica que podría cambiar después de un cierto período de tiempo, que normalmente determina el proveedor de servicios de Internet, puede usar un nombre DNS constante con un servicio DNS dinámico que apunte a la dirección IP pública actual del dispositivo VPN. Azure VPN Gateway resolverá el nombre de dominio completo para determinar la dirección IP pública a la que se va a conectar. En la captura de pantalla siguiente se muestra un ejemplo del uso de un nombre de dominio completo en lugar de la dirección IP.
   - **Espacio de direcciones** hace referencia a los intervalos de direcciones de la red que representa esta red local. Puede agregar varios intervalos de espacios de direcciones. Asegúrese de que los intervalos que especifique aquí no se superpongan con los de otras redes a las que quiera conectarse. Azure enrutará el intervalo de direcciones que especifique a la dirección IP del dispositivo VPN local. *Use sus propios valores aquí, y no los mostrados en el ejemplo, si quiere conectarse a su sitio local*.
   - **Configurar BGP:** usar solo al configurar BGP. En caso contrario, no seleccione esta opción.
   - **Subscription** (Suscripción): compruebe que se muestra la suscripción correcta.
   - **Grupos de recursos:** seleccione el grupo de recursos que quiere usar. Puede crear un grupo de recursos nuevo o seleccionar uno ya creado.
   - **Ubicación:** La ubicación es la misma que **Región** en otros valores. seleccione la ubicación en la que se creará este objeto. Puede seleccionar la misma ubicación en la que reside la red, pero no es obligatorio.

   Esta es la misma página pero con el nombre de dominio completo resaltado:

   :::image type="content" source="./media/vpn-gateway-add-local-network-gateway-portal-include/create-local-gateway-fqdn.png" alt-text="Creación de una puerta de enlace de red local con un nombre de dominio completo":::

   > [!NOTE]
   >
   > * Azure VPN Gateway solo admite una dirección IPv4 para cada nombre de dominio completo. Si el nombre de dominio se resuelve en varias direcciones IP, Azure VPN Gateway usará la primera dirección IP que devuelvan los servidores DNS. Para eliminar la incertidumbre, se recomienda que el nombre de dominio completo siempre se resuelva en una sola dirección IPv4. No se admite IPv6.
   > * Azure VPN Gateway mantiene una caché DNS que se actualiza cada 5 minutos. La puerta de enlace intenta resolver los nombres de dominio completos solo para los túneles desconectados. Al restablecer la puerta de enlace también se desencadenará la resolución del nombre de dominio completo.
   >

4. Cuando haya terminado de especificar los valores, seleccione el botón **Crear** en la parte inferior de la página para crear la puerta de enlace de red local.
