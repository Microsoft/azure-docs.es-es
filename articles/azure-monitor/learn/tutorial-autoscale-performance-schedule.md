---
title: Escalado automático de recursos de Azure en función de los datos o la programación
description: Creación de una configuración de escalado automático para un plan de servicio de aplicaciones con datos de métricas y una programación
author: anirudhcavale
services: azure-monitor
ms.topic: tutorial
ms.date: 12/11/2017
ms.author: ancav
ms.custom: mvc
ms.subservice: autoscale
ms.openlocfilehash: 8a55ab9b6b87a379fbdebcead4467ec47507d67a
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327521"
---
# <a name="create-an-autoscale-setting-for--azure-resources-based-on-performance-data-or-a-schedule"></a>Creación de una configuración de escalado automático de recursos de Azure basado en los datos de rendimiento o una programación

La configuración de escalado automático le permite agregar o quitar instancias de servicio según condiciones predefinidas. Esta configuración puede crearse a través del portal. Este método proporciona una interfaz de usuario basada en explorador para crear y establecer una configuración de escalado automático. 

En este tutorial, aprenderá lo siguiente: 
> [!div class="checklist"]
> * Creación de una aplicación web y un plan de App Service
> * Configuración de reglas de escalado automático para reducción horizontal y escalado horizontal de solicitudes que recibe una aplicación web
> * Activación de una acción de escalado horizontal y visualización del número de aumento de instancias
> * Activación de una acción de reducción horizontal y visualización del número de reducción de instancias
> * Limpiar los recursos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="log-in-to-the-azure-portal"></a>Iniciar sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="create-a-web-app-and-app-service-plan"></a>Creación de una aplicación web y un plan de App Service
1. Haga clic en la opción **Crear un recurso** en el panel de navegación izquierdo.
2. Busque y seleccione el elemento *Aplicación web* y haga clic en **Crear**.
3. Seleccione un nombre de aplicación, como *MyTestScaleWebApp*. Cree un nuevo grupo de recursos *myResourceGroup" y colóquelo en un grupo de recursos de su elección.

En el plazo de unos minutos, los recursos deberían aprovisionarse. Utilice la aplicación web y el plan de App Service correspondiente a lo largo de este tutorial.

   ![Creación de un nuevo servicio de aplicación en el portal](./media/tutorial-autoscale-performance-schedule/Web-App-Create.png)

## <a name="navigate-to-autoscale-settings"></a>Navegación a la configuración de escalado automático
1. En el panel de navegación izquierdo, seleccione la opción **Monitor**. Una vez que la página se cargue, seleccione la pestaña **Escalado automático**.
2. Aquí se muestra una lista de los recursos de la suscripción que admiten el escalado automático. Busque el plan de App Service que se creó anteriormente en el tutorial y haga clic en él.

    ![Navegación a la configuración de escalado automático](./media/tutorial-autoscale-performance-schedule/monitor-blade-autoscale.png)

3. En la configuración de escalado automático, haga clic en el botón **Habilitar escalado automático**.

Los pasos siguientes le ayudarán a rellenar la pantalla de escalado automático para que tenga un aspecto como el de la imagen siguiente:

   ![Guardado de la configuración de escalado automático](./media/tutorial-autoscale-performance-schedule/Autoscale-Setting-Save.png)

## <a name="configure-default-profile"></a>Configuración del perfil predeterminado
1. Escriba un **Nombre** para la configuración de escalado automático.
2. En el perfil predeterminado, asegúrese de que el **Modo de escala** esté establecido en “Escalar a un número específico de instancias”.
3. Establezca el recuento de instancias en **1**. Este valor garantiza que, cuando ningún otro perfil esté activo o en vigor, el perfil predeterminado devuelve el recuento de instancias a 1.

   ![Navegación a la configuración de escalado automático](./media/tutorial-autoscale-performance-schedule/autoscale-setting-profile.png)


## <a name="create-recurrence-profile"></a>Creación del perfil de periodicidad

1. Haga clic en el vínculo **Add a scale condition** (Agregar una condición de escalado) bajo el perfil predeterminado.

2. Edite el **nombre** de este perfil para que sea “Monday to Friday profile”.

3. Asegúrese de que en **Modo de escala** se haya seleccionado "Escalado basado en una métrica".

4. Para **Límites de instancia**, establezca el **Mínimo** en 1, el **Máximo** en 2 y el valor **Predeterminado** en 1. Esta configuración garantiza que este perfil no escale automáticamente el plan de servicio para que tenga menos de 1 instancia ni más de 2 instancias. Si el perfil no tiene suficientes datos para tomar una decisión, utiliza el número predeterminado de instancias (en este caso, 1).

5. Para **Programación**, seleccione "Repetir en días específicos".

6. Establezca el perfil para repetirse de lunes a viernes, de 09:00 PST a 18:00 PST. Esta configuración garantiza que este perfil solo esté activo y se aplique de lunes a viernes de 09:00 18:00 h. Durante el resto del tiempo, el perfil “Predeterminado” será la configuración de escalado automático que utilice el perfil.

## <a name="create-a-scale-out-rule"></a>Creación de una regla de escalado horizontal

1. En “Monday to Friday profile”.

2. Haga clic en el vínculo **Agregar una regla**.

3. Establezca el **Origen de métrica** en “Otro recurso”. Establezca **Tipo de recurso** en “App Services” y **Recurso** en la aplicación web creada anteriormente en este tutorial.

4. Establezca **Agregación de tiempo** en “Total”, **Nombre de métrica** en “Solicitudes” y la **Estadística de intervalo de agregación** en “Suma”.

5. Establezca **Operador** en “Mayor que”, **Umbral** en 10 y **Duración** en 5 minutos.

6. Seleccione la **Operación** como "Aumentar el número en", **Recuento de instancias** en 1 y **Tiempo de finalización** en 5 minutos.

7. Haga clic en el botón **Agregar**.

Esta regla garantiza que, si la aplicación web recibe más de 10 solicitudes en 5 minutos o menos, se agrega una instancia adicional al plan de App Service para administrar la carga.

   ![Creación de una regla de escalado horizontal](./media/tutorial-autoscale-performance-schedule/Scale-Out-Rule.png)

## <a name="create-a-scale-in-rule"></a>Creación de una regla de reducción horizontal
Se recomienda tener siempre una regla de reducción horizontal que acompañe la regla de escalado horizontal. La existencia de ambas garantiza que los recursos no se aprovisionen en exceso. El aprovisionamiento en exceso significa tener más instancias en ejecución de las necesarias para administrar la carga actual. 

1. En “Monday to Friday profile”.

2. Haga clic en el vínculo **Agregar una regla**.

3. Establezca el **Origen de métrica** en “Otro recurso”. Establezca **Tipo de recurso** en “App Services” y **Recurso** en la aplicación web creada anteriormente en este tutorial.

4. Establezca **Agregación de tiempo** en “Total”, **Nombre de métrica** en “Solicitudes” y la **Estadística de intervalo de agregación** en “Promedio”.

5. Establezca **Operador** en “Menor que”, **Umbral** en 5 y **Duración** en 5 minutos.

6. Establezca la **Operación** en"Reducir el recuento en", el **Recuento de instancias** en 1 y el **Tiempo de finalización** en 5 minutos.

7. Haga clic en el botón **Agregar**.

    ![Creación de una regla de reducción horizontal](./media/tutorial-autoscale-performance-schedule/Scale-In-Rule.png)

8. **Guarde** la configuración de escalado automático.

    ![Guardado de la configuración de escalado automático](./media/tutorial-autoscale-performance-schedule/Autoscale-Setting-Save.png)

## <a name="trigger-scale-out-action"></a>Desencadenamiento de una acción de escalado horizontal
Para desencadenar la condición de escalado horizontal en la configuración de escalado automático que acaba de crear, la aplicación web debe tener más de 10 solicitudes en menos de 5 minutos.

1. Abra una ventana del explorador y navegue a la aplicación web que creó anteriormente en este tutorial. Para encontrar la dirección URL de la aplicación web en Azure Portal, vaya al recurso de la aplicación web y haga clic en el botón **Examinar** en la pestaña “Información general”.

2. En rápida sucesión, vuelva a cargar la página más de 10 veces.

3. En el panel de navegación izquierdo, seleccione la opción **Monitor**. Una vez que la página se cargue, seleccione la pestaña **Escalado automático**.

4. En la lista, seleccione el plan de App Service usado a lo largo de este tutorial.

5. En la configuración de escalado automático, haga clic en la pestaña **Historial de ejecución**.

6. Verá un gráfico que refleja el recuento de instancias del plan de App Service a lo largo del tiempo.

7. En unos minutos, el recuento de instancias debería aumentar de 1 a 2.

8. Debajo del gráfico, verá las entradas del registro de actividad para cada acción de escalado realizada por esta configuración de escalado automático.

## <a name="trigger-scale-in-action"></a>Desencadenamiento de una acción de reducción horizontal
La condición de reducción horizontal en la configuración de escalado automático se desencadena si hay menos de 5 solicitudes a la aplicación web durante un período de 10 minutos. 

1. Asegúrese de que no se envíe ninguna solicitud a la aplicación web.

2. Cargue Azure Portal.

3. En el panel de navegación izquierdo, seleccione la opción **Monitor**. Una vez que la página se cargue, seleccione la pestaña **Escalado automático**.

4. En la lista, seleccione el plan de App Service usado a lo largo de este tutorial.

5. En la configuración de escalado automático, haga clic en la pestaña **Historial de ejecución**.

6. Verá un gráfico que refleja el recuento de instancias del plan de App Service a lo largo del tiempo.

7. En unos minutos, el recuento de instancias debería disminuir de 2 a 1. El proceso tarda al menos 100 minutos.  

8. Debajo del gráfico, verá el conjunto correspondiente de entradas del registro de actividad para cada acción de escalado realizada por esta configuración de escalado automático.

    ![Vista de las acciones de reducción horizontal](./media/tutorial-autoscale-performance-schedule/Scale-In-Chart.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

1. Desde el menú de la izquierda en Azure Portal, haga clic en **Todos los recursos** y luego seleccione la aplicación web creada en este tutorial.

2. En la página de recursos, haga clic en **Eliminar**, escriba **sí** en el cuadro de texto para confirmar la eliminación y haga clic en **Eliminar**.

3. A continuación, seleccione el recurso del plan de App Service y haga clic en **Eliminar**.

4. Escriba **sí** en el cuadro de texto para confirmar la eliminación y haga clic en **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:  
> [!div class="checklist"]
> * Creó una aplicación web y un plan de App Service.
> * Configuró reglas de escalado automático para reducción horizontal y escalado horizontal de solicitudes que recibe una aplicación web.
> * Activó una acción de escalado horizontal y visualizó el número de aumento de instancias.
> * Activó una acción de reducción horizontal y visualizó el número reducción de instancias.
> * Limpió los recursos.


Para obtener más información sobre la configuración del escalado automático, continúe con la [información general del escalado automático](../platform/autoscale-overview.md).

> [!div class="nextstepaction"]
> [Archivado de los datos de supervisión](../platform/platform-logs-overview.md)

