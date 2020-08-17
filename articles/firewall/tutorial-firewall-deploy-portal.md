---
title: 'Tutorial: Implementación y configuración de Azure Firewall mediante Azure Portal'
description: En este tutorial, aprenderá a implementar y configurar Azure Firewall mediante Azure Portal.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: tutorial
ms.date: 07/15/2020
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 8b4d58163c28e00c30c5b0f9db3a6ff259fbf5ae
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86536939"
---
# <a name="tutorial-deploy-and-configure-azure-firewall-using-the-azure-portal"></a>Tutorial: Implementación y configuración de Azure Firewall mediante Azure Portal

El control del acceso de red saliente es una parte importante de un plan de seguridad de red de ámbito general. Por ejemplo, es posible que desee limitar el acceso a sitios web. O bien, que desee limitar las direcciones IP de salida y los puertos a los que se puede acceder.

Una manera de controlar el acceso de red saliente desde una subred de Azure es con Azure Firewall. Con Azure Firewall, puede configurar:

* Reglas de aplicación que definen los nombres de dominio completos (FQDN) a los que se puede acceder desde una subred.
* Reglas de red que definen la dirección de origen, el protocolo, el puerto de destino y la dirección de destino.

El tráfico está sujeto a las reglas de firewall configuradas cuando enruta el tráfico al firewall como puerta de enlace predeterminada de la subred.

En este tutorial, creará una red virtual única simplificada con dos subredes para facilitar la implementación.

Para las implementaciones de producción, se recomienda un [modelo de concentrador y radio](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke), en el que el firewall está en su propia red virtual. Los servidores de las cargas de trabajo están en redes virtuales emparejadas en la misma región con una o varias subredes.

* **AzureFirewallSubnet**: el firewall está en esta subred.
* **Workload-SN**: el servidor de carga de trabajo está en esta subred. El tráfico de red de esta subred va a través del firewall.

![Infraestructura de red del tutorial](media/tutorial-firewall-deploy-portal/tutorial-network.png)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar un entorno de red de prueba
> * Implementar un firewall
> * Crear una ruta predeterminada
> * Configurar una regla de aplicación para permitir el acceso a www.google.com
> * Configuración de una regla de red para permitir el acceso a los servidores DNS externos
> * Configurar una regla NAT para permitir un escritorio remoto en el servidor de prueba
> * Probar el firewall

Si lo prefiere, puede seguir los pasos de este tutorial mediante [Azure PowerShell](deploy-ps.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="set-up-the-network"></a>Configuración de la red

En primer lugar, cree un grupo de recursos para que contenga los recursos necesarios para implementar el firewall. A continuación, cree una red virtual, subredes y un servidor de prueba.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

El grupo de recursos contiene todos los recursos necesarios para el tutorial.

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. En el menú de Azure Portal, seleccione **Grupos de recursos** o busque y seleccione *Grupos de recursos* desde cualquier página. A continuación, seleccione **Agregar**.
3. En **Nombre del grupo de recursos**, escriba *Test-FW-RG*.
4. En **Suscripción**, seleccione la suscripción.
5. En **Ubicación del grupo de recursos**: seleccione una ubicación. Los demás recursos que cree deben estar en la misma ubicación.
6. Seleccione **Crear**.

### <a name="create-a-vnet"></a>Creación de una red virtual

Esta red virtual contiene tres subredes.

> [!NOTE]
> El tamaño de la subred AzureFirewallSubnet es /26. Para más información sobre el tamaño de la subred, consulte [Preguntas más frecuentes sobre Azure Firewall](firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size).

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.
1. Seleccione **Redes** > **Red virtual**.
2. En **Suscripción**, seleccione la suscripción.
3. En **Grupo de recursos**, seleccione **Test-FW-RG**.
4. En **Nombre**, escriba **Test-FW-VN**.
5. En **Región**, seleccione la misma ubicación que usó anteriormente.
6. Seleccione **Siguiente: Direcciones IP**.
7. En **Espacio de direcciones IPv4**, escriba **10.0.0.0/16**.
8. En **Subred**, seleccione **predeterminada**.
9. En **Nombre de subred**, escriba **AzureFirewallSubnet**. El firewall estará en esta subred y el nombre de la subred **debe** ser AzureFirewallSubnet.
10. En **Intervalo de direcciones**, escriba **10.0.1.0/26**.
11. Seleccione **Guardar**.

   A continuación, cree una subred para el servidor de la carga de trabajo.

1. Haga clic en **Agregar subred**.
4. En **Nombre de subred**, escriba **Workload-SN**.
5. En **Intervalo de direcciones de subred**, escriba **10.0.2.0/24**.
6. Seleccione **Agregar**.
7. Seleccione **Revisar y crear**.
8. Seleccione **Crear**.

### <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Ahora cree la máquina virtual de la carga de trabajo y colóquela en la subred **Workload-SN**.

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.
2. Seleccione **Proceso** y, luego, **Máquina virtual**.
3. **Windows Server 2016 Datacenter** en la lista de destacados.
4. Especifique estos valores para la máquina virtual:

   |Configuración  |Valor  |
   |---------|---------|
   |Resource group     |**Test-FW-RG**|
   |Nombre de la máquina virtual     |**Srv-Work**|
   |Region     |Igual que la anterior|
   |Imagen|Windows Server 2019 Datacenter|
   |Nombre de usuario del administrador     |Escriba un nombre de usuario.|
   |Contraseña     |Escriba una contraseña.|

4. En **Reglas de puerto de entrada**, en **Puertos de entrada públicos**, seleccione **Ninguno**.
6. Acepte los restantes valores predeterminados y seleccione **Siguiente: Discos**.
7. Acepte los valores predeterminados del disco y seleccione **Siguiente: Redes**.
8. Asegúrese de que **Test-FW-VN** está seleccionada como red virtual y que la subred es **Workload-SN**.
9. En **IP pública**, seleccione **Ninguno**.
11. Acepte los restantes valores predeterminados y seleccione **Siguiente: Administración**.
12. Seleccione **Desactivar** para deshabilitar los diagnósticos de arranque. Acepte los restantes valores predeterminados y seleccione **Revisar y crear**.
13. Revise la configuración en la página de resumen y seleccione **Crear**.

## <a name="deploy-the-firewall"></a>Implementación del firewall

Implemente el firewall en la red virtual.

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.
2. Escriba **firewall** en el cuadro de búsqueda y presione **Entrar**.
3. Seleccione **Firewall** y después **Crear**.
4. En la página **Creación de un firewall**, utilice la tabla siguiente para configurar el firewall:

   |Configuración  |Valor  |
   |---------|---------|
   |Suscripción     |\<your subscription\>|
   |Resource group     |**Test-FW-RG** |
   |Nombre     |**Test-FW01**|
   |Location     |Seleccione la misma ubicación que usó anteriormente.|
   |Elegir una red virtual     |**Usar existente**: **Test-FW-VN**|
   |Dirección IP pública     |**Agregar nueva**<br>**Nombre**: **fw-pip**|

5. Seleccione **Revisar + crear**.
6. Revise el resumen y seleccione **Crear** para crear el firewall.

   La implementación tardará varios minutos.
7. Una vez finalizada la implementación, vaya al grupo de recursos **Test-FW-RG** y seleccione el firewall **Test-FW01**.
8. Anote las direcciones IP privadas y públicas del firewall. Usará estas direcciones más adelante.

## <a name="create-a-default-route"></a>Crear una ruta predeterminada

En la subred **Workload-SN**, configure la ruta predeterminada de salida que pase por el firewall.

1. En el menú de Azure Portal, seleccione **Todos los servicios** o busque y seleccione *Todos los servicios* desde cualquier página.
2. En **Redes**, seleccione **Tablas de rutas**.
3. Seleccione **Agregar**.
4. En **Nombre**, escriba **Firewall-route**.
5. En **Suscripción**, seleccione la suscripción.
6. En **Grupo de recursos**, seleccione **Test-FW-RG**.
7. En **Ubicación**, seleccione la misma ubicación que usó anteriormente.
8. Seleccione **Crear**.
9. Seleccione **Actualizar**y, después, seleccione la tabla de rutas **Firewall-route**.
10. Seleccione **Subredes** y, después, seleccione **Asociar**.
11. Seleccione **Red virtual** > **FW-Test-VN**.
12. En **Subred**, seleccione **Workload-SN**. Asegúrese de seleccionar únicamente la subred **Workload-SN** para esta ruta o el firewall no funcionará correctamente.

13. Seleccione **Aceptar**.
14. Seleccione **Rutas** y después **Agregar**.
15. En **Nombre de ruta**, escriba **fw-dg**.
16. En **Prefijo de dirección** escriba **0.0.0.0/0**.
17. En **Tipo del próximo salto**, seleccione **Aplicación virtual**.

    Azure Firewall es realmente un servicio administrado, pero una aplicación virtual funciona en esta situación.
18. En **Dirección del próximo salto**, escriba la dirección IP privada del firewall que anotó anteriormente.
19. Seleccione **Aceptar**.

## <a name="configure-an-application-rule"></a>Configuración de una regla de aplicación

Esta es la regla de aplicación que permite el acceso de salida a `www.google.com`.

1. Abra **Test-FW-RG**y seleccione el firewall **Test-FW01**.
2. En la página **Test-FW01**, en **Configuración**, seleccione **Reglas**.
3. Seleccione la pestaña **Recopilación de reglas de aplicación**.
4. Seleccione **Agregar una colección de reglas de aplicación**.
5. En **Nombre**, escriba **App-Coll01**.
6. En **Priority**, escriba **200**.
7. En **Acción**, seleccione **Permitir**.
8. En **Reglas**, **FQDN de destino**, como **Nombre** escriba **Allow-Google**.
9. Como **Tipo de origen**, seleccione **Dirección IP**.
10. Como **Origen**, escriba **10.0.2.0/24**.
11. En **Protocolo:Puerto**, escriba **http, https**.
12. En **FQDN de destino**, escriba **`www.google.com`** .
13. Seleccione **Agregar**.

Azure Firewall incluye una colección de reglas integradas para FQDN de infraestructura que están permitidos de forma predeterminada. Estos FQDN son específicos para la plataforma y no se pueden usar para otros fines. Para más información, consulte [Nombres de dominio completos de infraestructura](infrastructure-fqdns.md).

## <a name="configure-a-network-rule"></a>Configurar una regla de red

Se trata de la regla de red que permite el acceso saliente a dos direcciones IP en el puerto 53 (DNS).

1. Seleccione la pestaña **Recopilación de reglas de red**.
2. Seleccione **Agregar una colección de reglas de red** .
3. En **Nombre**, escriba **Net-Coll01**.
4. En **Priority**, escriba **200**.
5. En **Acción**, seleccione **Permitir**.
6. En **Reglas**, **Direcciones IP**, como **Nombre**, escriba **Allow-DNS**.
7. En **Protocolo**, seleccione **UDP**.
9. Como **Tipo de origen**, seleccione **Dirección IP**.
1. Como **Origen**, escriba **10.0.2.0/24**.
2. En **Tipo de destino**, seleccione **Dirección IP**.
3. Como **Dirección de destino**, escriba **209.244.0.3,209.244.0.4**.

   Estos son servidores DNS públicos ofrecidos por CenturyLink.
1. En **Puertos de destino**, escriba **53**.
2. Seleccione **Agregar**.

## <a name="configure-a-dnat-rule"></a>Configurar una regla de DNAT

Esta regla le permite conectar un escritorio remoto a la máquina virtual Srv-Work a través del firewall.

1. Seleccione la pestaña **Colección de reglas NAT**.
2. Seleccione **Agregar una colección de reglas NAT**.
3. En **Nombre**, escriba **rdp**.
4. En **Priority**, escriba **200**.
5. En **Reglas**, en **Nombre**, escriba **rdp-nat**.
6. En **Protocolo**, seleccione **TCP**.
7. Como **Tipo de origen**, seleccione **Dirección IP**.
8. Para **Origen**, escriba **\*** .
9. En **Dirección de destino**, escriba la dirección IP pública del firewall.
10. En **Puertos de destino**, escriba **3389**.
11. En **Dirección traducida**, escriba la dirección IP privada de **Srv-Work**.
12. En **Puerto traducido**, escriba **3389**.
13. Seleccione **Agregar**.


### <a name="change-the-primary-and-secondary-dns-address-for-the-srv-work-network-interface"></a>Cambio de la dirección DNS principal y secundaria para la interfaz de red **Srv-Work**

Con fines de prueba para este tutorial, configure las direcciones DNS principal y secundaria del servidor. Esto no es un requisito general de Azure Firewall.

1. En el menú de Azure Portal, seleccione **Grupos de recursos** o busque y seleccione *Grupos de recursos* desde cualquier página. Seleccione el grupo de recursos **Test-FW-RG**.
2. Seleccione la interfaz de red de la máquina virtual **Srv-Work**.
3. En **Configuración**, seleccione **Servidores DNS**.
4. En **Servidores DNS**, seleccione **Personalizado**.
5. Escriba **209.244.0.3** en el cuadro de texto **Agregar servidor DNS** y **209.244.0.4** en el siguiente cuadro de texto.
6. Seleccione **Guardar**.
7. Reinicie la máquina virtual **Srv-Work**.

## <a name="test-the-firewall"></a>Probar el firewall

Ahora, pruebe el firewall para confirmar que funciona según lo previsto.

1. Conecte un escritorio remoto a la dirección IP pública del firewall e inicie sesión en la máquina virtual **Srv-Work**. 
3. Abra Internet Explorer y vaya a `https://www.google.com`.
4. Seleccione **Aceptar** > **Cerrar** en las alertas de seguridad de Internet Explorer.

   Debería ver la página principal de Google.

5. Vaya a `https://www.microsoft.com`.

   El firewall debería bloquearle.

Con ello, ha comprobado que las reglas de firewall funcionan:

* Puede navegar al FQDN permitido pero no a ningún otro.
* Puede resolver nombres DNS con el servidor DNS externo configurado.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede conservar los recursos relacionados con el firewall para el siguiente tutorial o, si ya no los necesita, eliminar el grupo de recursos **Test-FW-RG** para eliminarlos todos.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Supervisión de los registros de Azure Firewall](./tutorial-diagnostics.md)
