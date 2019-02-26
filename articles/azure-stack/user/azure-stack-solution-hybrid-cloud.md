---
title: Implementación de una nube híbrida con Azure y Azure Stack | Microsoft Docs
description: Obtenga información sobre cómo implementar una nube híbrida con Azure y Azure Stack.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/25/2019
ms.author: mabrigg
ms.reviewer: anajod
ms.lastreviewed: 01/25/2019
ms.openlocfilehash: 5f142192571bdd15a33575a425d75baf3e5caea2
ms.sourcegitcommit: 898b2936e3d6d3a8366cfcccc0fccfdb0fc781b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/30/2019
ms.locfileid: "55243494"
---
# <a name="tutorial-deploy-a-hybrid-cloud-solution-with-azure-and-azure-stack"></a>Tutorial: Implementación de una solución en la nube híbrida con Azure y Azure Stack

*Se aplica a: Sistemas integrados de Azure Stack y Kit de desarrollo de Azure Stack*

Este tutorial muestra cómo implementar una solución en la nube híbrida que usa la nube pública de Azure y la nube privada de Azure Stack.

Mediante el uso de una solución en la nube híbrida, puede combinar las ventajas de cumplimiento de una nube privada con la escalabilidad de la nube pública. Además, los desarrolladores pueden aprovechar el ecosistema de desarrolladores de Microsoft y aplicar sus conocimientos a los entornos en la nube y locales.

## <a name="overview-and-assumptions"></a>Introducción y supuestos

Puede seguir este tutorial para configurar un flujo de trabajo que permite a los desarrolladores implementar una aplicación web idéntica en una nube pública y una nube privada. Esta aplicación podrá acceder a una red enrutable sin conexión a Internet hospedada en la nube privada. Estas aplicaciones web se supervisan y cuando hay un pico de tráfico un programa modifica los registros DNS para redirigir el tráfico a la nube pública. Cuando el tráfico disminuye hasta el nivel anterior al pico, el tráfico se enruta de vuelta a la nube privada.

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> - Implementar un servidor de base de datos de SQL Server con conexión híbrida.
> - Conectar una aplicación web de Azure global a una red híbrida.
> - Configurar DNS para el escalado entre nubes.
> - Configurar certificados SSL para el escalado entre nubes.
> - Configurar e implementar la aplicación web.
> - Crear un perfil de Traffic Manager y configurarlo para el escalado entre nubes.
> - Configurar la supervisión y las alertas de Application Insights para un aumento del tráfico.
> - Configurar la conmutación automática del tráfico entre Azure global y Azure Stack.

### <a name="assumptions"></a>Supuestos

En este tutorial se da por supuesto que tiene conocimientos básicos de Azure global y Azure Stack. Para más información antes de iniciar el tutorial, consulte los siguientes artículos:

 - [Introducción a Azure](https://azure.microsoft.com/overview/what-is-azure/)
 - [Conceptos clave de Azure Stack](https://docs.microsoft.com/azure/azure-stack/azure-stack-key-features)

En este tutorial también se supone que tiene una suscripción de Azure. Si no tiene una suscripción, puede [crear una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar este tutorial, asegúrese de que se cumplen los requisitos siguientes:

- Un Kit de desarrollo de Azure Stack (ASDK) o una suscripción a un sistema integrado de Azure Stack. Para implementar un Kit de desarrollo de Azure Stack, siga las instrucciones de [Implementación de ASDK mediante el instalador](../asdk/asdk-deploy.md).
- La instalación de Azure Stack debe tener instalado lo siguiente:
  - Azure App Service. Trabaje con su operador de Azure Stack para implementar y configurar Azure App Service en su entorno. Este tutorial requiere que App Service tenga al menos un (1) rol de trabajo dedicado disponible.
  - Una imagen de Windows Server 2016
  - Windows Server 2016 con una imagen de Microsoft SQL Server
  - Los planes y ofertas adecuados
 - Un nombre de dominio para la aplicación web. Si no tiene un nombre de dominio, puede comprar uno a un proveedor de dominios como GoDaddy, Bluehost o InMotion.
- Un certificado SSL para el dominio de una entidad de certificación de confianza como LetsEncrypt.
- Una aplicación web que se comunica con una base de datos de SQL Server y es compatible con Application Insights. Puede descargar la aplicación de ejemplo [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) de GitHub.
- Una red híbrida entre una red virtual de Azure y la red virtual de Azure Stack. Para instrucciones detalladas, consulte [Configuración de la conectividad de la nube híbrida con Azure y Azure Stack](azure-stack-solution-hybrid-connectivity.md).

- Una canalización de integración continua e implementación continua (CI/CD) con un agente de compilación privado en Azure Stack. Para instrucciones detalladas, consulte [Configuración de la identidad de nube híbrida con aplicaciones de Azure y Azure Stack](azure-stack-solution-hybrid-identity.md)

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Implementación de un servidor de base de datos de SQL Server con conexión híbrida

1. Inicie sesión en el portal de usuarios de Azure Stack.

2. En **Dashboard** (Panel), seleccione **Marketplace**.

    ![Marketplace de Azure Stack](media/azure-stack-solution-hybrid-cloud/image1.png)

3. En **Marketplace**, seleccione **Compute** (Proceso) y, a continuación, elija **More** (Más). En **Más**, seleccione **Free SQL Server License: SQL Server 2017 Developer en Windows Server**.

    ![Selección de una imagen de máquina virtual](media/azure-stack-solution-hybrid-cloud/image2.png)

4. En **Free SQL Server License: SQL Server 2017 Developer en Windows Server** seleccione **Crear**.

5. En **Basics>Configure basic settings** (Aspectos básicos>Configurar), escriba un nombre para la máquina virtual (VM) en **Name** (Nombre), escriba un nombre de usuario para el usuario SA de SQL Server SA en **User name** (Nombre de usuario) y escriba la contraseña del usuario SA en **Password** (Contraseña).  En la lista desplegable **Subscription** (Suscripción), seleccione la suscripción en la que se va a realizar la implementación. Para **Resource group** (Grupo de recursos), utilice **Choose existing** (Elegir existente) y coloque la máquina virtual en el mismo grupo de recursos que la aplicación web de Azure Stack.

    ![Configuración de las opciones básicas para la máquina virtual](media/azure-stack-solution-hybrid-cloud/image3.png)

6. En **Size** (Tamaño), elija un tamaño de máquina virtual. Para este tutorial se recomienda A2_Standard o DS2_V2_Standard.

7. En **Settings>Configure optional features** (Configuración>Configurar características opcionales), configure las siguientes opciones:

    - **Cuenta de almacenamiento**. Si necesita una, cree una nueva cuenta.
    - **Red virtual**

      > [!Important]  
      > Asegúrese de que la máquina virtual de SQL Server se implementa en la misma red virtual que las puertas de enlace de VPN.

    - **Dirección IP pública**. Puede usar la configuración predeterminada.
    - **Network security group** (Grupo de seguridad de red) (NSG). Cree un nuevo grupo de seguridad de red.
    - **Extensions (Extensiones) y Monitoring (Supervisión)**. Mantenga la configuración predeterminada.
    - **Diagnostics storage account** (Cuenta de almacenamiento de información de diagnóstico). Si necesita una, cree una nueva cuenta.
    - Seleccione **OK** (Aceptar) para guardar la configuración.

    ![Configuración de características opcionales](media/azure-stack-solution-hybrid-cloud/image4.png)

1. En **SQL Server settings** (Configuración de SQL Server), configure las siguientes opciones:
   - En **SQL connectivity** (Conectividad SQL), seleccione **Public (Internet)** (Público [Internet]).
   - En **Port** (Puerto), mantenga el valor predeterminado, **1433**.
   - En **SQL authentication** (Autenticación SQL), seleccione **Enable** (Habilitar).

     > [!Note]  
     > Al habilitar la autenticación SQL, se rellena de forma automática con la información de "SQLAdmin" que configuró en **Aspectos básicos**.

   - Para el resto de la configuración, conserve los valores predeterminados. Seleccione **Aceptar**.

    ![Configuración de SQL Server](media/azure-stack-solution-hybrid-cloud/image5.png)

9. En **Summary** (Resumen), revise la configuración de la máquina virtual y, a continuación, seleccione **OK** (Aceptar) para iniciar la implementación.

    ![Resumen de la configuración](media/azure-stack-solution-hybrid-cloud/image6.png)

10. La nueva máquina virtual tardará algún tiempo en crearse. Puede ver el estado de las máquinas virtuales en **Virtual machines** (Máquinas virtuales).

    ![Máquinas virtuales](media/azure-stack-solution-hybrid-cloud/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack"></a>Creación de aplicaciones web en Azure y Azure Stack

Azure App Service simplifica la ejecución y administración de una aplicación web. Dado que Azure Stack es coherente con Azure, App Service se puede ejecutar en ambos entornos. Usará App Service para hospedar la aplicación.

### <a name="create-web-apps"></a>Creación de aplicaciones web

1. Cree una aplicación web en Azure siguiendo las instrucciones de [Administración de un plan de App Service en Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan). Debe asegurarse de colocar la aplicación web en la misma suscripción y grupo de recursos que la red híbrida.

2. Repita el paso anterior (1) en Azure Stack.

### <a name="add-route-for-azure-stack"></a>Adición de una ruta para Azure Stack

App Service en Azure Stack debe ser enrutable desde la red pública de Internet para permitir que los usuarios accedan a la aplicación. Si la instancia de Azure Stack es accesible desde Internet, anote la dirección IP pública o la dirección URL de la aplicación web de Azure Stack.

Si usa un ASDK, puede [configurar una asignación NAT estática](https://docs.microsoft.com/azure/azure-stack/azure-stack-create-vpn-connection-one-node#configure-the-nat-virtual-machine-on-each-azure-stack-development-kit-for-gateway-traversal) para exponer App Service fuera del entorno virtual.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Conexión de una aplicación web de Azure a una red híbrida

Para proporcionar conectividad entre el front-end web en Azure y la base de datos de SQL Server en Azure Stack, la aplicación web debe estar conectada a la red híbrida entre Azure y Azure Stack. Para habilitar la conectividad, tendrá que:

- Configurar la conectividad de punto a sitio
- Configuración de la aplicación web
- Modificar la puerta de enlace de red local en Azure Stack.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Configuración de la red virtual de Azure para la conectividad de punto a sitio

La puerta de enlace de red virtual en el lado de Azure de la red híbrida debe permitir conexiones de punto a sitio para la integración con Azure App Service.

1. En Azure, vaya a la página de la puerta de enlace de red virtual. En **Configuración**, seleccione **Configuración de punto a sitio**.

    ![Opción de punto a sitio](media/azure-stack-solution-hybrid-cloud/image8.png)

2. Seleccione **Configurar ahora** para la configuración de punto a sitio.

    ![Inicio de la configuración de punto a sitio](media/azure-stack-solution-hybrid-cloud/image9.png)

3. En la página de configuración **De punto a sitio**, escriba el intervalo de direcciones IP privadas que desea usar en **Grupo de direcciones**.

   > [!Note]  
   > Asegúrese de que el intervalo que especifique no se superpone con ninguno de los intervalos de direcciones ya utilizados por las subredes de Azure global o los componentes de Azure Stack d la red híbrida.

   En **Tipo de túnel**, desactive la opción **VPN IKEv2**. Seleccione **Guardar** para finalizar la configuración de punto a sitio.

   ![Configuración de punto a sitio](media/azure-stack-solution-hybrid-cloud/image10.png)

### <a name="integrate-the-azure-app-service-application-with-the-hybrid-network"></a>Integración de la aplicación de Azure App Service con la red híbrida

1. Para conectar la aplicación a la red virtual de Azure, siga las instrucciones de [Habilitación de la integración de red virtual](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#enabling-vnet-integration).

2. Vaya a **Configuración** en el plan de App Service que hospeda la aplicación web. En **Configuración**, seleccione **Redes**.

    ![Configuración de red](media/azure-stack-solution-hybrid-cloud/image11.png)

3. En **Integración de VNET**, seleccione **Haga clic aquí para administrar**.

    ![Administración de la integración con red virtual](media/azure-stack-solution-hybrid-cloud/image12.png)

4. Seleccione la red virtual que desea configurar. En **DIRECCIONES IP ENRUTADAS A RED VIRTUAL**, escriba el intervalo de direcciones IP de la red virtual de Azure, la red virtual de Azure Stack y los espacios de direcciones de punto a sitio. Seleccione **Guardar** para validar y guardar la configuración.

    ![Intervalos de direcciones IP que se van a enrutar](media/azure-stack-solution-hybrid-cloud/image13.png)

Para más información acerca de cómo se integra App Service con las redes virtuales de Azure, consulte [Integración de la aplicación con una red virtual de Azure](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).

### <a name="configure-the-azure-stack-virtual-network"></a>Configuración de la red virtual de Azure Stack

La puerta de enlace de red local de la red virtual de Azure Stack se debe configurar para enrutar el tráfico desde el intervalo de direcciones de punto a sitio de App Service.

1. En Azure Stack, vaya a **Local network gateway** (Puerta de enlace de red local). En **Settings** (Configuración), seleccione **Configuration** (Configuración).

    ![Opción de configuración de puerta de enlace](media/azure-stack-solution-hybrid-cloud/image14.png)

2. En **Address space** (Espacio de direcciones), escriba el intervalo de direcciones de punto a sitio para la puerta de enlace de red virtual de Azure y seleccione **Save** (Guardar) para validar y guardar esta configuración.

    ![Espacio de direcciones de punto a sitio](media/azure-stack-solution-hybrid-cloud/image15.png)

## <a name="configure-dns-for-cross-cloud-scaling"></a>Configuración de DNS para el escalado entre nubes

Mediante la configuración adecuada de DNS para las aplicaciones entre nubes, los usuarios pueden acceder a las instancias de Azure global y de Azure Stack de la aplicación web. La configuración de DNS de este tutorial también permite que Azure Traffic Manager enrute el tráfico cuando la carga aumenta o disminuye.

Este tutorial usa Azure DNS para administrar el sistema DNS. (Los dominios de App Service no funcionarán).

### <a name="create-subdomains"></a>Creación de subdominios

Dado que Traffic Manager se basa en registros CNAME de DNS, se necesita un subdominio para enrutar correctamente el tráfico a los puntos de conexión. Para más información acerca de la asignación de dominios y registros DNS, consulte [Asignación de dominios con Traffic Manager](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name)

Para el punto de conexión de Azure, creará un subdominio que los usuarios pueden utilizar para acceder a la aplicación web. Para este tutorial, puede usar **app.northwind.com**, pero este valor se debe personalizar en función de su propio dominio.

También deberá crear un subdominio con un registro A para el punto de conexión de Azure Stack. Puede usar **azurestack.northwind.com**.

### <a name="configure-a-custom-domain-in-azure"></a>Configuración de un dominio personalizado en Azure

1. Agregue el nombre de host **app.northwind.com** a la aplicación web de Azure mediante la [Asignación de un registro CNAME a Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).

### <a name="configure-custom-domains-in-azure-stack"></a>Configuración de dominios personalizados en Azure Stack

1. Agregue el nombre de host **azurestack.northwind.com** a la aplicación web de Azure Stack mediante la [Asignación de un registro A para Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record). Use la dirección IP enrutable por Internet de la aplicación de App Service.

2. Agregue el nombre de host **app.northwind.com** a la aplicación web de Azure Stack mediante la [Asignación de un registro CNAME a Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record). Utilice el nombre de host que configuró en el paso anterior (1) como destino para el registro CNAME.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>Configuración de certificados SSL para el escalado entre nubes

Debe asegurarse de que los datos confidenciales recopilados por la aplicación web están seguros en reposo y en tránsito hacia la base de datos SQL.

Debe configurar las aplicaciones web de Azure y Azure Stack para que usen certificados SSL para todo el tráfico entrante.

### <a name="add-ssl-to-azure-and-azure-stack"></a>Adición de SSL en Azure y Azure Stack

Para agregar SSL en Azure:

1. Asegúrese de que el certificado SSL que ha obtenido es válido para el subdominio que ha creado. (Es correcto usar certificados con caracteres de comodín).

2. En Azure, siga las instrucciones de las secciones **Preparación de la aplicación web** y **Enlace del certificado SSL** del artículo [Enlace de un certificado SSL personalizado existente con Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl). Seleccione **SSL basado en SNI** como el **Tipo de SSL**.

3. Redirija todo el tráfico al puerto HTTPS. Siga las instrucciones de la sección **Aplicación de HTTPS** del artículo [Enlace de un certificado SSL personalizado existente con Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl).

Para agregar SSL en Azure Stack:

- Repita los pasos 1 a 3 que usó para Azure.

## <a name="configure-and-deploy-the-web-application"></a>Configuración e implementación de la aplicación web

Deberá configurar el código de la aplicación para notificar los datos de telemetría a la instancia correcta de Application Insights y configurar las aplicaciones web con las cadenas de conexión correctas. Para más información sobre Application Insights, consulte [¿Qué es Application Insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)

### <a name="add-application-insights"></a>Adición de Application Insights

1. Abra la aplicación web en Microsoft Visual Studio.

2. [Agregue Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net-core#add-application-insights-telemetry) al proyecto para transmitir los datos de telemetría que utiliza Application Insights para crear alertas cuando el tráfico web aumenta o disminuye.

### <a name="configure-dynamic-connection-strings"></a>Configuración de cadenas de conexión dinámicas

Cada instancia de la aplicación web usará un método diferente para conectarse a la base de datos SQL. La aplicación de Azure utiliza la dirección IP privada de la máquina virtual (VM) de SQL Server y la aplicación de Azure Stack usa la dirección IP pública de la máquina virtual de SQL Server.

> [!Note]  
> En un sistema integrado de Azure Stack, la dirección IP pública no debe ser enrutable en Internet. En un Kit de desarrollo de Azure Stack (ASDK), la dirección IP pública no es enrutable fuera del ASDK.

Puede usar variables de entorno de App Service para pasar una cadena de conexión distinta a cada instancia de la aplicación.

1. Abra la aplicación en Visual Studio.

2. Abra Startup.cs y busque el bloque de código siguiente:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Reemplace el bloque de código anterior por el código siguiente, que usa una cadena de conexión definida en el archivo appsettings.json:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-application-settings"></a>Configuración de la aplicación de App Service

1. Cree cadenas de conexión para Azure y Azure Stack. Las cadenas deben ser iguales, excepto las direcciones IP utilizadas.

2. En Azure y Azure Stack, agregue la cadena de conexión adecuada [como una configuración de la aplicación](https://docs.microsoft.com/azure/app-service/web-sites-configure) en la aplicación web, utilizando `SQLCONNSTR\_` como prefijo en el nombre.

3. **Guarde** la configuración de la aplicación web y reinicie la aplicación.

## <a name="enable-automatic-scaling-in-global-azure"></a>Habilitación del escalado automático en Azure global

Cuando se crea la aplicación web en un entorno de App Service, esta se inicia con una instancia. Puede escalar horizontalmente de manera automática para agregar instancias y proporcionar más recursos de proceso a la aplicación. De forma similar, puede reducir horizontalmente de manera automática y reducir el número de instancias que necesita la aplicación.

> [!Note]  
> Debe tener un plan de App Service para configurar el escalado y la reducción horizontal. Si no tiene un plan, cree uno antes de iniciar los pasos siguientes.

### <a name="enable-automatic-scale-out"></a>Habilitación del escalado horizontal automático

1. En Azure, busque el plan de App Service para los sitios que desea escalar horizontalmente y, a continuación, seleccione **Escalar horizontalmente (plan de App Service)**.

    ![Escalado horizontal](media/azure-stack-solution-hybrid-cloud/image16.png)

2. Seleccione **Habilitar escalado automático**.

    ![Habilitar escalado automático](media/azure-stack-solution-hybrid-cloud/image17.png)

3. Escriba un nombre para **Nombre de la configuración de escalado automático**. Para la regla de escalado automático **Predeterminada**, seleccione **Escalado basado en una métrica**. Establezca el valor de **Límites de instancia** en **Mínimo: 1**, **Máximo: 10** y **Predeterminado: 1**.

    ![Configuración del escalado automático](media/azure-stack-solution-hybrid-cloud/image18.png)

4. Seleccione **+Agregar una regla**.

5. En **Origen de métrica**, seleccione **Recurso actual**. Utilice los siguientes criterios y acciones para la regla.

**Criterios**

1. En **Agregación de tiempo**, seleccione **Media**.

2. En **Nombre de métrica**, seleccione **Porcentaje de CPU**.

3. En **Operador**, seleccione **Mayor que**.

   - Establezca **Umbral** en **50**.
   - Establezca **Duración** en **10**.

**Acción**

1. En **Operación**, seleccione **Aumentar recuento en**.

2. Establezca **Recuento de instancias** en **2**.

3. Establezca **Tiempo de finalización** en **5**.

4. Seleccione **Agregar**.

5. Seleccione **+Agregar una regla**.

6. En **Origen de métrica**, seleccione **Recurso actual**.

   > [!Note]  
   > El recurso actual contendrá el nombre o el GUID del plan de App Service y las listas desplegables **Tipo de recurso** y **Recurso** se atenuarán.

### <a name="enable-automatic-scale-in"></a>Habilitación de la reducción horizontal automática

Cuando el tráfico disminuye, la aplicación web de Azure puede reducir automáticamente el número de instancias activas para reducir los costos. Esta acción es menos agresiva que el escalado horizontal con el fin de minimizar el impacto en los usuarios de la aplicación.

1. Vaya a la condición de escalado horizontal **Predeterminada** y seleccione **+ Agregar una regla**. Utilice los siguientes criterios y acciones para la regla.

**Criterios**

1. En **Agregación de tiempo**, seleccione **Media**.

2. En **Nombre de métrica**, seleccione **Porcentaje de CPU**.

3. En **Operador**, seleccione **Menor que**.

   - Establezca **Umbral** en **30**.
   - Establezca **Duración** en **10**.

**Acción**

1. En **Operación**, seleccione **Reducir recuento en**.

   - Establezca **Recuento de instancias** en **1**.
   - Establezca **Tiempo de finalización** en **5**.

2. Seleccione **Agregar**.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Creación de un perfil de Traffic Manager y configuración del escalado entre nubes

Creará un perfil de Traffic Manager en Azure y, a continuación, configurará puntos de conexión para habilitar el escalado entre nubes.

### <a name="create-traffic-manager-profile"></a>Creación de un perfil de Traffic Manager

1. Seleccione **Crear un recurso**
2. Seleccione **Redes**
3. Seleccione **Perfil de Traffic Manager** y configure lo siguiente:

   - En **Nombre**, escriba un nombre para el perfil. Este nombre **debe** ser único en la zona trafficmanager.net y se utiliza para crear un nuevo nombre de DNS (por ejemplo, northwindstore.trafficmanager.net).
   - En **Método de enrutamiento**, seleccione **Ponderado**.
   - En **Suscripción**, seleccione la suscripción en la que desea crear este perfil.
   - En **Grupo de recursos**, cree un grupo de recursos nuevo para este perfil.
   - En **Ubicación del grupo de recursos**, seleccione la ubicación del grupo de recursos. Esta configuración se refiere a la ubicación del grupo de recursos y no tiene efecto alguno sobre el perfil de Traffic Manager que se implementará globalmente.

4. Seleccione **Crear**.

    ![Creación de un perfil de Traffic Manager](media/azure-stack-solution-hybrid-cloud/image19.png)

 Una vez que se completa la implementación global del perfil de Traffic Manager, aparece en la lista de recursos del grupo de recursos en el que se creó.

### <a name="add-traffic-manager-endpoints"></a>Incorporación de puntos de conexión de Traffic Manager

1. Busque el perfil de Traffic Manager que creó. (Si estaba en el grupo de recursos del perfil, seleccione el perfil).

2. En **Perfil de Traffic Manager**, en **Configuración**, seleccione **Puntos de conexión**.

3. Seleccione **Agregar**.

4. En **Agregar punto de conexión**, utilice la siguiente configuración para Azure Stack:

   - En **Tipo**, seleccione **Punto de conexión externo**.
   - Escriba un **Nombre** para este punto de conexión.
   - En **Nombre de dominio completo (FQDN) o dirección IP**, escriba la dirección URL externa de la aplicación web de Azure Stack.
   - En **Peso**, mantenga el valor predeterminado, **1**. Este peso hace que todo el tráfico vaya a este punto de conexión si funciona correctamente.
   - No active la opción **Agregar como deshabilitado**.

5. Seleccione **Aceptar** para guardar el punto de conexión de Azure Stack.

A continuación, configurará el punto de conexión de Azure.

1. En **Perfil de Traffic Manager**, seleccione **Puntos de conexión**.
2. Seleccione **+Agregar**.
3. En **Agregar punto de conexión**, utilice la siguiente configuración para Azure:

   - En **Tipo**, seleccione **Punto de conexión de Azure**.
   - Escriba un **Nombre** para este punto de conexión.
   - En **Tipo de recurso de destino**, seleccione **App Service**.
   - En **Recurso de destino**, seleccione **Elegir un servicio de aplicaciones** para mostrar la lista de Web Apps en la misma suscripción.
   - En **Recursos**, elija el servicio de aplicación que quiere agregar como el primer punto de conexión.
   - En **Peso**, seleccione **2**. Esto hace que todo el tráfico vaya a este punto de conexión si el punto de conexión principal está en mal estado o si tiene una regla o alerta que redirige el tráfico cuando se desencadena.
   - No active la opción **Agregar como deshabilitado**.

4. Seleccione **Aceptar** para guardar el punto de conexión de Azure.

Después de que ambos puntos de conexión están configurados, aparecerán en el **Perfil de Traffic Manager** al seleccionar **Puntos de conexión**. El ejemplo de la captura de pantalla siguiente muestra dos puntos de conexión con la información de estado y configuración de cada uno de ellos.

![Puntos de conexión](media/azure-stack-solution-hybrid-cloud/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a>Configuración de la supervisión y las alertas de Application Insights

Azure Application Insights permite supervisar la aplicación y enviar alertas basadas en las condiciones que configure. Algunos ejemplos son: la aplicación no está disponible, experimenta errores o muestra problemas de rendimiento.

Usará las métricas de Application Insights para crear alertas. Cuando se desencadenan estas alertas, la instancia de la aplicación web cambiará automáticamente de Azure Stack a Azure para escalar horizontalmente y volverá a Azure Stack para reducir horizontalmente.

### <a name="create-an-alert-from-metrics"></a>Creación de una alerta a partir de métricas

Vaya al grupo de recursos de este tutorial y, a continuación, seleccione la instancia de Application Insights para abrir **Application Insights**.

![Application Insights](media/azure-stack-solution-hybrid-cloud/image21.png)

Usará esta vista para crear una alerta de escalado horizontal y una alerta de reducción horizontal.

### <a name="create-the-scale-out-alert"></a>Creación de la alerta de escalado horizontal

1. En **Configurar**, seleccione **Alertas (clásica)**.
2. Seleccione **Agregar una alerta de métrica (clásica)**.
3. En **Agregar regla**, configure lo siguiente:

   - En **Nombre**, escriba **Burst into Azure Cloud**.
   - La **Descripción** es opcional.
   - En **Origen**, en **Alerta sobre**, seleccione **Métricas**.
   - En **Criterios**, seleccione la suscripción, el grupo de recursos del perfil de Traffic Manager y el nombre del perfil de Traffic Manager para Recurso.

4. En **Métrica**, seleccione **Velocidad de solicitudes de**.
5. En **Condición**, seleccione **Mayor que**.
6. En **Umbral**, escriba **2**.
7. En **Período**, seleccione **En los últimos 5 minutos**.
8. En **Notificar mediante**:
   - Active la casilla de verificación para **Correo electrónico a propietarios, colaboradores y lectores**.
   - En **Correos electrónicos adicionales del administrador**, escriba su dirección de correo electrónico.

9. En la barra de menús, seleccione **Guardar**.

### <a name="create-the-scale-in-alert"></a>Creación de la alerta de reducción horizontal

1. En **Configurar**, seleccione **Alertas (clásica)**.
2. Seleccione **Agregar una alerta de métrica (clásica)**.
3. En **Agregar regla**, configure lo siguiente:

   - En **Nombre**, escriba **Scale back into Azure Stack**.
   - La **Descripción** es opcional.
   - En **Origen**, en **Alerta sobre**, seleccione **Métricas**.
   - En **Criterios**, seleccione la suscripción, el grupo de recursos del perfil de Traffic Manager y el nombre del perfil de Traffic Manager para Recurso.

4. En **Métrica**, seleccione **Velocidad de solicitudes de**.
5. En **Condición**, seleccione **Menor que**.
6. En **Umbral**, escriba **2**.
7. En **Período**, seleccione **En los últimos 5 minutos**.
8. En **Notificar mediante**:
   - Active la casilla de verificación para **Correo electrónico a propietarios, colaboradores y lectores**.
   - En **Correos electrónicos adicionales del administrador**, escriba su dirección de correo electrónico.

9. En la barra de menús, seleccione **Guardar**.

La captura de pantalla siguiente muestra las alertas de escalado horizontal y reducción horizontal.

   ![Alertas (clásica)](media/azure-stack-solution-hybrid-cloud/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack"></a>Redirección del tráfico entre Azure y Azure Stack

Puede configurar la conmutación manual o automática del tráfico de la aplicación web entre Azure y Azure Stack.

### <a name="configure-manual-switching-between-azure-and-azure-stack"></a>Configuración de la conmutación manual entre Azure y Azure Stack

Cuando el sitio web llegue a los umbrales que configuró, recibirá una alerta. Utilice los pasos siguientes para redirigir el tráfico a Azure manualmente.

1. En Azure Portal, seleccione el perfil de Traffic Manager.

    ![Puntos de conexión del Administrador de tráfico](media/azure-stack-solution-hybrid-cloud/image20.png)

2. Seleccione **Puntos de conexión**.
3. Seleccione el **punto de conexión de Azure**.
4. En **Estado**, seleccione **Habilitado** y, a continuación, seleccione **Guardar**.

    ![Habilitación del punto de conexión de Azure](media/azure-stack-solution-hybrid-cloud/image23.png)

5. En **Puntos de conexión** del perfil de Traffic Manager, seleccione **Punto de conexión externo**.
6. En **Estado**, seleccione **Deshabilitado** y, a continuación, seleccione **Guardar**.

    ![Deshabilitación del punto de conexión de Azure Stack](media/azure-stack-solution-hybrid-cloud/image24.png)

Después de configurar los puntos de conexión, el tráfico de la aplicación va a la aplicación web de escalabilidad horizontal de Azure en lugar de la aplicación web de Azure Stack.

 ![Puntos de conexión modificados](media/azure-stack-solution-hybrid-cloud/image25.png)

Para invertir el flujo de vuelta a Azure Stack, use los pasos anteriores para:

- Habilitar el punto de conexión de Azure Stack
- Deshabilitar el punto de conexión de Azure

### <a name="configure-automatic-switching-between-azure-and-azure-stack"></a>Configuración de la conmutación automática entre Azure y Azure Stack

También puede usar la supervisión de Application Insights si la aplicación se ejecuta en un entorno [sin servidor](https://azure.microsoft.com/overview/serverless-computing/) proporcionado por Azure Functions.

En este escenario, puede configurar Application Insights para que use un webhook que llama a una aplicación de función. Esta aplicación habilita o deshabilita automáticamente un punto de conexión en respuesta a una alerta.

Use los pasos siguientes como guía para configurar la conmutación automática del tráfico.

1. Cree una aplicación de función de Azure.
2. Cree una función desencadenada por HTTP.
3. Importe los SDK de Azure para Resource Manager, Web Apps y Traffic Manager.
4. Desarrolle código para:

   - Autenticarse en la suscripción de Azure.
   - Usar un parámetro que alterna los puntos de conexión de Traffic Manager para dirigir el tráfico a Azure o Azure Stack.

5. Guarde el código y agregue la dirección URL de la aplicación de función con los parámetros adecuados en la sección **Webhook** de la configuración de la regla de alertas de Application Insights.
6. El tráfico se redirige automáticamente cuando se desencadena una alerta de Application Insights.

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los patrones de nube de Azure, consulte [Patrones de diseño en la nube](https://docs.microsoft.com/azure/architecture/patterns).
