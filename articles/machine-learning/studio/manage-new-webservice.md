---
title: 'ML Studio (clásico): Administración de servicios web: Azure'
description: Administre los servicios web nuevos y clásicos de Machine Learning mediante el portal de servicios web de Microsoft Azure Machine Learning. Dado que los servicios web clásicos y nuevos se basan en tecnologías subyacentes distintas, cada uno de ellos presenta funciones de administración ligeramente diferentes.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: previous-ms.author=yahajiza, previous-author=YasinMSFT
ms.date: 02/28/2017
ms.openlocfilehash: d40d67b40d9e269f9ed6552467918cc878ffd197
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87430779"
---
# <a name="manage-a-web-service-using-the-azure-machine-learning-studio-classic-web-services-portal"></a>Administración de un servicio web mediante el portal de servicios web de Azure Machine Learning Studio (clásico)

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


Puede administrar los servicios web Machine Learning nuevos y clásicos mediante el portal Servicios web Microsoft Azure Machine Learning. Dado que los servicios web clásicos y nuevos se basan en tecnologías subyacentes distintas, cada uno de ellos presenta funciones de administración ligeramente diferentes.

En el portal Servicios web Machine Learning, puede realizar las siguientes acciones:

* Supervisar el uso del servicio web.
* Configurar la descripción, actualizar las claves del servicio web (solo en los nuevos), actualizar la clave de la cuenta de almacenamiento (solo en los nuevos), habilitar el registro y habilitar o deshabilitar datos de ejemplo
* Eliminar el servicio web.
* Crear, eliminar o actualizar planes de facturación (solo en los nuevos)
* Agregar y eliminar puntos de conexión (solo en los clásicos)

>[!NOTE]
>También puede administrar los servicios web clásicos en [Machine Learning Studio (clásico)](https://studio.azureml.net), en la pestaña de **servicios web**.

## <a name="permissions-to-manage-new-resources-manager-based-web-services"></a>Permisos para administrar el administrador de nuevos recursos basados en servicios web

Los nuevos servicios web se implementan como recursos de Azure. Por lo tanto, debe tener los permisos correctos para implementar y administrar nuevos servicios web.  Para implementar o administrar nuevos servicios web, es preciso que se le haya asignado un rol de colaborador o administrador en la suscripción en la que se implementa el servicio web. Si invita a otro usuario a un área de trabajo de Machine Learning, debe asignarle un rol de colaborador o administrador en la suscripción para que pueda implementar o administrar servicios web. 

Si el usuario no tiene los permisos correctos para acceder a recursos en el portal de servicios web Machine Learning de Azure, verá el siguiente error al tratar de implementar un servicio web:

*Error de implementación de servicio web. Esta cuenta no tiene suficientes derechos de acceso a la suscripción de Azure que contiene el área de trabajo. Para implementar un servicio web en Azure, la misma cuenta debe estar invitada al área de trabajo y tener acceso a la suscripción de Azure que contiene el área de trabajo*.

Para más información sobre cómo crear un área de trabajo, consulte cómo [crear y compartir un área de trabajo de Azure Machine Learning Studio (clásico)](create-workspace.md).

Para más información acerca de cómo establecer los permisos de acceso, consulte [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).


## <a name="manage-new-web-services"></a>Administración de servicios web nuevos
Instrucciones para la administración de servicios web nuevos:

1. Inicie sesión en el portal [Servicios web Microsoft Azure Machine Learning](https://services.azureml.net/quickstart) con su cuenta de Microsoft Azure (use la que está asociada a la suscripción de Azure).
2. En el menú, haga clic en **Servicios web**.

Se mostrará una lista de los servicios web implementados de la suscripción. 

Para administrar un servicio web, haga clic en Servicios web. En la página Servicios web, puede llevar a cabo estas acciones:

* Hacer clic en el servicio web para administrarlo
* Hacer clic en el plan de facturación del servicio web para actualizarlo
* Eliminar un servicio web
* Copiar un servicio web e implementarlo en otra región

Al hacer clic en un servicio web, se abrirá la correspondiente página Inicio rápido. Esta página tiene dos opciones de menú que le permiten administrar el servicio web:

* **PANEL**: permite ver el uso del servicio web.
* **CONFIGURAR**: permite agregar texto descriptivo, actualizar la clave de la cuenta de almacenamiento asociada con el servicio web, y habilitar y deshabilitar datos de ejemplo.

### <a name="monitoring-how-the-web-service-is-being-used"></a>Supervisión del uso del servicio web
Haga clic en la pestaña **Panel** .

En el panel puede ver el uso general del servicio web durante un periodo. Puede seleccionar el periodo para verlo en el menú desplegable Periodo situado en la esquina superior derecha de los gráficos de uso. El panel muestra la siguiente información:

* **Requests Over Time** (Solicitudes a lo largo de un periodo) muestra un gráfico de escalera del número de solicitudes durante el periodo seleccionado. Puede ayudarlo a identificar si se están experimentando picos de uso.
* **Request-Response Requests** (Solicitudes de solicitud-respuesta) muestra el número total de llamadas de solicitud-respuesta que ha recibido el servicio durante el periodo seleccionado y cuántas tienen errores.
* **Average Request-Response Compute Time** (Tiempo de proceso medio de solicitud-respuesta) muestra un promedio del tiempo necesario para ejecutar las solicitudes recibidas.
* **Batch Requests** (Solicitudes por lotes) muestra el número total de solicitudes por lotes que ha recibido el servicio en el periodo seleccionado y cuántas tienen errores.
* **Average Job Latency** (Latencia media de los trabajos) muestra un promedio del tiempo necesario para ejecutar las solicitudes recibidas.
* **Errores** muestra el número total de errores que se han producido en las llamadas al servicio web.
* **Services Costs** (Costos de los servicios) muestra los cargos correspondientes al plan de facturación asociado con el servicio.

### <a name="configuring-the-web-service"></a>Configuración del servicio web
Haga clic en la opción de menú **CONFIGURAR** .

Puede actualizar las propiedades siguientes:

* **Descripción** permite escribir una descripción del servicio web.
* **Título** permite escribir un título para el servicio web
* **Claves** permite alternar las claves de API primarias y secundarias.
* **Clave de cuenta de almacenamiento** permite actualizar la clave de la cuenta de almacenamiento asociada con los cambios del servicio web. 
* **Habilitar datos de ejemplo** permite ofrecer datos de ejemplo que pueden usarse para probar el servicio de solicitud-respuesta. Si ha creado el servicio web en Machine Learning Studio (clásico) de Microsoft Azure, los datos de ejemplo se toman de los utilizados para entrenar el modelo. Si ha creado el servicio mediante programación, los datos se extraen de los datos de ejemplo ofrecidos como parte del paquete JSON.

### <a name="managing-billing-plans"></a>Administración de planes de facturación
Haga clic en la opción de menú **Planes** de la página Inicio rápido de los servicios web. También puede hacer clic en el plan asociado con el servicio web específico para administrar dicho plan.

* **Nuevo** permite crear un nuevo plan.
* **Add/Remove Plan instance** (Agregar/eliminar instancia de plan) permite escalar horizontalmente un plan existente para agregar capacidad.
* **Upgrade/DownGrade** (Actualizar/cambiar a una versión anterior) permite escalar verticalmente un plan existente para agregar capacidad.
* **Eliminar** permite eliminar un plan.

Haga clic en un plan para ver su panel. El panel proporciona una instantánea o el uso del plan durante un periodo seleccionado. Para seleccionar el periodo que desee ver, haga clic en el menú desplegable **Período** situado en la esquina superior derecha del panel. 

El panel de planes proporciona la siguiente información:

* **Descripción del plan** muestra información sobre los costos y la capacidad asociados con el plan.
* **Plan Usage** (Uso del plan) muestra el número de transacciones y las horas de proceso que se han cobrado en el plan.
* **Servicios web** muestra el número de servicios web que usan este plan.
* **Top web Service By Calls** (Principales servicios web por llamadas) muestra los cuatro principales servicios web que realizan llamadas que se cobran del plan.
* **Top web Services by Compute Hrs** (Principales servicios web por horas de proceso) muestra los cuatro principales servicios web que usan recursos de proceso que se cobran del plan.

## <a name="manage-classic-web-services"></a>Administración de servicios web clásicos
> [!NOTE]
> Los procedimientos de esta sección son pertinentes para la administración de los servicios web clásicos a través del portal Servicios web Azure Machine Learning. Para más información sobre cómo administrar servicios web clásicos mediante Machine Learning Studio (clásico) y Azure Portal, consulte [Administración de un área de trabajo de Azure Machine Learning Studio (clásico)](manage-workspace.md).
> 
> 

Instrucciones para administrar los servicios web clásicos:

1. Inicie sesión en el portal [Servicios web Microsoft Azure Machine Learning](https://services.azureml.net/quickstart) con su cuenta de Microsoft Azure (use la que está asociada a la suscripción de Azure).
2. En el menú, haga clic en **Classic Web Services** (Servicios web clásicos).

Para administrar un servicio web clásico, haga clic en **Classic Web Services** (Servicios web clásicos). Desde la página Classic Web Services (Servicios web clásicos), puede llevar a cabo estas acciones:

* Hacer clic en el servicio web para ver los puntos de conexión asociados
* Eliminar un servicio web

Al administrar un servicio web clásico, cada uno de los puntos de conexión se administra por separado. Al hacer clic en un servicio web en la página Servicios web, se abre la lista de puntos de conexión asociados con el servicio. 

En la página de puntos de conexión del servicio web clásico puede agregar y eliminar los puntos de conexión del servicio. Para más información sobre la incorporación de puntos de conexión, consulte [Creación de puntos de conexión](create-endpoint.md).

Haga clic en uno de los puntos de conexión para abrir la página Inicio rápido del servicio web. En la página Inicio rápido hay dos opciones de menú que permiten administrar el servicio web:

* **PANEL**: permite ver el uso del servicio web.
* **CONFIGURAR**: permite agregar texto descriptivo, activar y desactivar el registro de errores, actualizar la clave de la cuenta de almacenamiento asociada con el servicio web, y habilitar y deshabilitar datos de ejemplo.

### <a name="monitoring-how-the-web-service-is-being-used"></a>Supervisión del uso del servicio web
Haga clic en la pestaña **Panel** .

En el panel puede ver el uso general del servicio web durante un periodo. Puede seleccionar el periodo para verlo en el menú desplegable Periodo situado en la esquina superior derecha de los gráficos de uso. El panel muestra la siguiente información:

* **Requests Over Time** (Solicitudes a lo largo de un periodo) muestra un gráfico de escalera del número de solicitudes durante el periodo seleccionado. Puede ayudarlo a identificar si se están experimentando picos de uso.
* **Request-Response Requests** (Solicitudes de solicitud-respuesta) muestra el número total de llamadas de solicitud-respuesta que ha recibido el servicio durante el periodo seleccionado y cuántas tienen errores.
* **Average Request-Response Compute Time** (Tiempo de proceso medio de solicitud-respuesta) muestra un promedio del tiempo necesario para ejecutar las solicitudes recibidas.
* **Batch Requests** (Solicitudes por lotes) muestra el número total de solicitudes por lotes que ha recibido el servicio en el periodo seleccionado y cuántas tienen errores.
* **Average Job Latency** (Latencia media de los trabajos) muestra un promedio del tiempo necesario para ejecutar las solicitudes recibidas.
* **Errores** muestra el número total de errores que se han producido en las llamadas al servicio web.
* **Services Costs** (Costos de los servicios) muestra los cargos correspondientes al plan de facturación asociado con el servicio.

### <a name="configuring-the-web-service"></a>Configuración del servicio web
Haga clic en la opción de menú **CONFIGURAR** .

Puede actualizar las propiedades siguientes:

* **Descripción** permite escribir una descripción del servicio web. La descripción es un campo obligatorio.
* **Registro** permite habilitar o deshabilitar el registro de errores en el punto de conexión. Para obtener más información sobre el registro, consulte [Habilitación del registro para los servicios web Machine Learning](web-services-logging.md).
* **Habilitar datos de ejemplo** permite ofrecer datos de ejemplo que pueden usarse para probar el servicio de solicitud-respuesta. Si ha creado el servicio web en Machine Learning Studio (clásico) de Microsoft Azure, los datos de ejemplo se toman de los utilizados para entrenar el modelo. Si ha creado el servicio mediante programación, los datos se extraen de los datos de ejemplo ofrecidos como parte del paquete JSON.


