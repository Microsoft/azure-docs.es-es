---
title: Supervisión de los servicios y aplicaciones de Azure mediante Grafana
description: Enrute los datos de Azure Monitor y Application Insights para que puedan verse en Grafana.
ms.subservice: ''
ms.topic: conceptual
ms.date: 11/06/2017
ms.openlocfilehash: 23bba091628eee767fbf292a8a8d772ffab674cb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073471"
---
# <a name="monitor-your-azure-services-in-grafana"></a>Supervisar los servicios de Azure en Grafana
Ahora, los servicios y las aplicaciones de Azure se pueden supervisar desde [Grafana](https://grafana.com/) gracias al [complemento de origen de datos de Azure Monitor](https://grafana.com/plugins/grafana-azure-monitor-datasource). El complemento recopila datos de rendimiento de aplicación recabados por Azure Monitor, incluidos varios registros y métricas. De este modo, podrá ver estos datos en el panel de Grafana.

Haga lo siguiente para configurar un servidor de Grafana y confeccionar paneles para las métricas y los registros desde Azure Monitor.

## <a name="set-up-a-grafana-server"></a>Configuración de un servidor de Grafana

### <a name="set-up-grafana-locally"></a>Configuración de Grafana localmente
Para configurar un servidor local de Grafana, [descargue e instale Grafana en su entorno local](https://grafana.com/grafana/download). Para usar la integración de Azure Monitor del complemento, instale Grafana versión 5.3 o posterior.

### <a name="set-up-grafana-on-azure-through-the-azure-marketplace"></a>Configuración de Grafana en Azure a través de Azure Marketplace
1. Vaya a Azure Marketplace y seleccione Grafana de Grafana Labs.

2. Rellene los nombres y los detalles pertinentes. Cree un nuevo grupo de recursos. Lleve un seguimiento de los valores de nombre de usuario de máquina virtual, contraseña de máquina virtual y contraseña de administrador del servidor de Grafana que haya elegido.  

3. Elija un tamaño de máquina virtual y una cuenta de almacenamiento.

4. Configure la red.

5. Vea el resumen y seleccione **Crear** después de aceptar los términos de uso.

6. Una vez finalizada la implementación, seleccione **Ir al grupo de recursos**. Verá una lista de los recursos recién creados.

    ![Objetos del grupo de recursos de Grafana](media/grafana-plugin/grafana1.png)

    Si selecciona el grupo de seguridad de red (*grafana-nsg* en este caso), verá que se usa el puerto 3000 para tener acceso al servidor Grafana.

7. Obtenga la dirección IP pública del servidor de Grafana; vuelva a la lista de recursos y seleccione **Dirección IP pública**.

## <a name="sign-in-to-grafana"></a>Inicio de sesión en Grafana

1. Con la dirección IP del servidor, abra la página de inicio de sesión en *http://\<IP address\>:3000* o *\<DNSName>\:3000* en el explorador. Aunque 3000 es el puerto predeterminado, tenga en cuenta que es posible que haya seleccionado un puerto diferente durante la instalación. Debería aparecer una página de inicio de sesión del servidor de Grafana que creó.

    ![Pantalla de inicio de sesión de Grafana](./media/grafana-plugin/grafana-login-screen.png)

2. Inicie sesión con el nombre de usuario *admin* y la contraseña de administrador del servidor de Grafana que creó anteriormente. Si usa una instalación local, la contraseña predeterminada sería *admin*, y se le pedirá que la cambie en el primer inicio de sesión.

## <a name="configure-data-source-plugin"></a>Configurar el complemento de origen de datos

Tras haber iniciado sesión correctamente, deberá ver que el complemento de origen de datos de Azure Monitor ya aparece incluido.

![Grafana incluye el complemento de Azure Monitor](./media/grafana-plugin/grafana-includes-azure-monitor-plugin-dark.png)

1. Seleccione **Agregar origen de datos** para agregar y configurar el origen de datos de Azure Monitor.

2. Elija un nombre para el origen de datos y seleccione **Azure Monitor** como tipo en la lista desplegable.

3. Cree una entidad de servicio: Grafana usa una entidad de servicio de Azure Active Directory para conectarse a las API de Azure Monitor y recopilar datos. Debe crear una entidad de servicio, o usar una existente, para administrar el acceso a los recursos de Azure.
    * Vea [estas instrucciones](../../active-directory/develop/howto-create-service-principal-portal.md) para crear una entidad de servicio. Copie y guarde el identificador de inquilino (id. de directorio), el identificador de cliente (id. de aplicación) y el secreto de cliente (valor de clave de aplicación).
    * Consulte [Asignación de la aplicación a un rol](../../active-directory/develop/howto-create-service-principal-portal.md) para asignar el rol Lector a la aplicación de Azure Active Directory en la suscripción, el grupo de recursos o el recurso que quiere supervisar. 
    La API de Log Analytics requiere el [rol Lector de Log Analytics](../../role-based-access-control/built-in-roles.md#log-analytics-reader), que incluye los permisos del rol Lector y se agrega a él.

4. Proporcione los detalles de conexión a las API que le gustaría usar. Puede conectarse a todas o a algunas de ellas. 
    * Si se conecta tanto a las métricas como a los registros de Azure Monitor, puede volver a usar las mismas credenciales; para ello, seleccione **Same details as Azure Monitor API** (Mismos detalles que la API de Azure Monitor).
    * Al configurar el complemento, puede indicar qué nube de Azure (pública, Azure US Gov, Azure Alemania o Azure China) quiere que supervise el complemento.
    * Si usa Application Insights, también puede incluir la API de Application Insights y el identificador de la aplicación para recopilar métricas basadas en Application Insights. Para más información, vea [Getting your API key and Application ID](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID) (Obtener la clave de API y el identificador de aplicación).

        > [!NOTE]
        > Algunos campos del origen de datos se denominan de forma diferente a su configuración de Azure correlacionada:
        > * El id. de inquilino es el identificador de Azure Directory
        > * El id. de cliente es el id. de la aplicación de Azure Active Directory
        > * El secreto de cliente es el valor de clave de la aplicación de Azure Active Directory

5. Si usa Application Insights, también puede incluir la API de Application Insights y el identificador de la aplicación para recopilar métricas basadas en Application Insights. Para más información, vea [Getting your API key and Application ID](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID) (Obtener la clave de API y el identificador de aplicación).

6. Seleccione **Guardar**, y Grafana probará las credenciales para cada API. Debería aparecer un mensaje similar al siguiente.  
    ![Configuración aprobada del origen de datos de Grafana](./media/grafana-plugin/grafana-data-source-config-approved-dark.png)

## <a name="build-a-grafana-dashboard"></a>Crear un panel de Grafana

1. Vaya a la página principal de Grafana y seleccione **New Dashboard**.

2. En el nuevo panel, seleccione el **gráfico**. Puede probar con otras opciones de gráfico, pero en este artículo usaremos *Gráfico* como ejemplo.

3. Se abre un gráfico en blanco en el panel. Haga clic en el título del panel y seleccione **Editar** para especificar los detalles de los datos que quiere trazar en este gráfico.
    ![Nuevo gráfico de Grafana](./media/grafana-plugin/grafana-new-graph-dark.png)

4. Seleccione el origen de datos de Azure Monitor que ha configurado.
   * Recopilación de métricas de Azure Monitor: seleccione **Azure Monitor** en la lista desplegable de servicio. Aparece una lista de selectores, donde puede seleccionar los recursos y las métricas que quiere supervisar en este gráfico. Para recopilar métricas de una VM, use el espacio de nombres **Microsoft.Compute/VirtualMachines**. Cuando haya seleccionado las VM y las métricas, puede empezar a ver los datos en el panel.
     ![Configuración del gráfico de Grafana para Azure Monitor](./media/grafana-plugin/grafana-graph-config-for-azure-monitor-dark.png)
   * Recopilación de datos de registro de Azure Monitor: seleccione **Azure Log Analytics** en la lista desplegable de servicio. Seleccione el área de trabajo que le gustaría consultar y establezca el texto de consulta. Puede copiar aquí cualquier consulta de registro que ya tenga o puede crear una. A medida que escribe la consulta, aparecerá IntelliSense y sugerirá las opciones de Autocompletar. Seleccione el tipo de visualización, **Serie temporal** y **Tabla**, y ejecute la consulta.
    
     > [!NOTE]
     >
     > La consulta predeterminada que se proporciona con el complemento usa dos macros: "$__timeFilter() y $__interval. 
     > Estas macros permiten a Grafana calcular dinámicamente el intervalo de tiempo y el intervalo de agregación, al acercarse a una parte de un gráfico. Puede quitar estas macros y usar un filtro de tiempo estándar, como *TimeGenerated > ago(1h)* , pero significaría que el gráfico no admitiese la característica de acercamiento.
    
     ![Configuración del gráfico de Grafana para Azure Log Analytics](./media/grafana-plugin/grafana-graph-config-for-azure-log-analytics-dark.png)

5. Este es un sencillo panel con dos gráficos. El de la izquierda muestra el porcentaje de CPU de dos máquinas virtuales. El de la derecha muestra las transacciones de una cuenta de almacenamiento de Azure desglosadas por el tipo de la API de transacción.
    ![Ejemplo de dos gráficos en Grafana](media/grafana-plugin/grafana6.png)


## <a name="optional-monitor-your-custom-metrics-in-the-same-grafana-server"></a>Opcional: Supervisar las métricas personalizadas en el mismo servidor de Grafana

También puede instalar Telegraf e InfluxDB para recopilar y trazar métricas tanto basadas en agente como personalizadas en la misma instancia de Grafana. Existe un amplio abanico de complementos de origen de datos que se pueden usar para reunir estas métricas en un panel.

También puede reutilizar esta configuración para incluir métricas desde su servidor de Prometheus. Use el complemento de origen de datos de Prometheus de la galería de complementos de Grafana.

Aquí tiene algunos artículos de referencia interesantes sobre cómo usar Telegraf, InfluxDB, Prometheus y Docker:
 - [How To Monitor System Metrics with the TICK Stack on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-monitor-system-metrics-with-the-tick-stack-on-ubuntu-16-04) (Cómo supervisar métricas de sistema con la pila TICK en Ubuntu 16.04)

 - [A monitoring solution for Docker hosts, containers, and containerized services](https://stefanprodan.com/2016/a-monitoring-solution-for-docker-hosts-containers-and-containerized-services/) (Solución de supervisión para hosts, contenedores y servicios en contenedores de Docker)

Esta es una imagen de un panel de Grafana completo con métricas de Azure Monitor y Application Insights.
![Métricas de ejemplo de Grafana](media/grafana-plugin/grafana8.png)

## <a name="advanced-grafana-features"></a>Características avanzadas de Grafana

### <a name="variables"></a>variables
Algunos valores de consulta pueden seleccionarse a través de las listas desplegables de la interfaz de usuario y se actualizan en la consulta. Tenga en cuenta la siguiente consulta como ejemplo:
```
Usage 
| where $__timeFilter(TimeGenerated) 
| summarize total_KBytes=sum(Quantity)*1024 by bin(TimeGenerated, $__interval) 
| sort by TimeGenerated
```

Puede configurar una variable que enumerará todos los valores de **Solution** disponibles y, luego, actualice la consulta para usarla.
Para crear una variable, haga clic en botón de configuración del panel en la parte superior derecha, seleccione **Variables** y luego **New**.
En la página de la variable, defina el origen de datos y la consulta que se ejecutará con el fin de obtener la lista de valores.
![Variable de configuración de Grafana](./media/grafana-plugin/grafana-configure-variable-dark.png)

Una vez creado, ajuste la consulta para usar los valores seleccionados, y sus gráficos responderán en consecuencia:
```
Usage 
| where $__timeFilter(TimeGenerated) and Solution in ($Solutions)
| summarize total_KBytes=sum(Quantity)*1024 by bin(TimeGenerated, $__interval) 
| sort by TimeGenerated
```
    
![Variables de uso de Grafana](./media/grafana-plugin/grafana-use-variables-dark.png)

### <a name="create-dashboard-playlists"></a>Creación de listas de reproducción de panel

Una de las muchas características útiles de Grafana es la lista de reproducción de panel. Puede crear varios paneles y agregarlos a una lista de reproducción, configurando un intervalo para que cada panel se muestre. Seleccione **Reproducir** para ir viendo los paneles uno a uno. Puede que quiera que se muestren en un monitor de pared de grandes dimensiones a modo de panel de estado del grupo.

![Ejemplo de lista de reproducción de Grafana](./media/grafana-plugin/grafana7.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha configurado un entorno de Grafana en Azure, se le cobrará cuando se ejecutan las VM, así las esté usando o no. Para evitar estos cargos extra, limpie el grupo de recursos creado en este artículo.

1. En el menú izquierdo de Azure Portal, haga clic en **Grupos de recursos** y en **Grafana**.
2. En la página del grupo de recursos, haga clic en **Eliminar**, escriba **Grafana** en el cuadro de texto y haga clic en **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes
* [Información general sobre las métricas en Microsoft Azure](data-platform.md)
