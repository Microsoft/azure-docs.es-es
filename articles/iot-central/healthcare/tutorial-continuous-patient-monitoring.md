---
title: Creación de una aplicación de supervisión continua de pacientes con Azure IoT Central | Microsoft Docs
description: Aprenda a crear una aplicación de supervisión continua de pacientes mediante plantillas de aplicación de Azure IoT Central.
author: philmea
ms.author: philmea
ms.date: 09/24/2019
ms.topic: tutorial
ms.service: iot-central
services: iot-central
manager: eliotgra
ms.openlocfilehash: 704c56745ad89e9ed2f79e8a863f1d0bc9845bf9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001832"
---
# <a name="tutorial-deploy-and-walkthrough-a-continuous-patient-monitoring-app-template"></a>Tutorial: Implementación y tutorial de una plantilla de aplicación de supervisión continua de pacientes



En este tutorial se muestra cómo puede, como creador de soluciones, empezar a implementar una plantilla de aplicación de supervisión continua de pacientes de IoT Central. Aprenderá a implementar la plantilla, qué elementos se incluyen de fábrica y qué puede hacer a continuación.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Elija una plantilla de aplicación.
> * Recorrido por la plantilla de aplicación

## <a name="create-an-application-template"></a>Elija una plantilla de aplicación.

Vaya al [sitio web del administrador de aplicaciones de Azure IoT Central](https://apps.azureiotcentral.com/). Seleccione **Crear** en la barra de navegación de la izquierda y, a continuación, haga clic en la pestaña **Healthcare** (Asistencia sanitaria). 

>[!div class="mx-imgBorder"] 
>![Administrador de aplicaciones de asistencia sanitaria](media/app-manager-health.png)

Haga clic en el botón **Crear aplicación** para empezar a crear la aplicación y, a continuación, inicie sesión con una cuenta personal, profesional o educativa de Microsoft. Esto le llevará a la página **Nueva aplicación**.

![Crear aplicación de asistencia sanitaria](media/app-manager-health-create.png)

![Crear información de facturación de una aplicación de asistencia sanitaria](media/app-manager-health-create-billinginfo.png)

Para crear la aplicación:

1. Azure IoT Central sugiere automáticamente un nombre de aplicación basado en la plantilla que haya seleccionado. Puede aceptar este nombre o escribir su propio nombre descriptivo de aplicación, como **Supervisión continua de pacientes**. Azure IoT Central también genera un prefijo de dirección URL único, en función del nombre de la aplicación. Cambiar si lo desea este prefijo de dirección URL por algo más fácil de recordar.

2. Puede elegir si desea crear la aplicación mediante el plan de precios *gratuito* o mediante uno de los planes de precios *estándar*. Las aplicaciones que crea mediante el plan de precios gratuito no tienen costo alguno durante siete días y luego expiran, y permiten hasta cinco dispositivos gratuitos. Puede cambiar una aplicación del plan de precios gratuito a un plan estándar en cualquier momento antes de que expire. Si elige el plan de precios gratuito, debe escribir su información de contacto y elegir si desea recibir información y sugerencias de Microsoft. Las aplicaciones que crea mediante un plan de tarifa estándar admiten hasta dos dispositivos gratuitos y requieren que se incluya la información de suscripción de Azure para la facturación.

3. Seleccione **Crear** en la parte inferior de la página para implementar la aplicación.

## <a name="walk-through-the-application-template"></a>Recorrido por la plantilla de aplicación

### <a name="dashboards"></a>Paneles

Después de implementar la plantilla de la aplicación, irá en primer lugar al **panel de supervisión de pacientes in situ de Lamna**. Lamna Healthcare es un sistema hospitalario ficticio que contiene dos hospitales: de Woodgrove y de Burkville. En este panel del operador del hospital de Woodgrove podrá ver información y telemetría sobre los dispositivos de esta plantilla junto con un conjunto de comandos, trabajos y acciones que puede realizar. Desde el panel se puede:

* Consultar la telemetría de los dispositivos y propiedades como el **nivel de batería** del dispositivo o su estado de **conectividad**.

* Vea el **plan de las plantas** y la ubicación para el dispositivo inteligente de revisión de constantes vitales (Smart Vitals Patch).

* **Vuelva a aprovisionarlo** para un nuevo paciente.

* Vea un ejemplo del **panel de proveedor** que un equipo de atención hospitalaria podría ver para realizar el seguimiento de sus pacientes.

* Cambie el **estado del paciente** del dispositivo para indicar si se está usando para un escenario remoto o con pacientes in situ.

>[!div class="mx-imgBorder"] 
>![Pacientes in situ Lamna](media/lamna-in-patient.png)

También puede hacer clic en **Go to remote patient dashboard** (Ir al panel de pacientes remotos) para ver el segundo panel del operador usado para el hospital de Burkville. Este panel contiene un conjunto similar de acciones, telemetría e información. Además, puede ver la utilización de varios dispositivos y tener la capacidad de **actualizar el firmware** en cada uno.

>[!div class="mx-imgBorder"] 
>![Lamna remoto](media/lamna-remote.png)

Siempre puede volver a vincular esta documentación en ambos paneles.

### <a name="device-templates"></a>Plantillas de dispositivo

Si hace clic en la pestaña **Plantillas de dispositivo**, verá que hay dos tipos de dispositivo diferentes que forman parte de la plantilla:

* **Smart Vitals Patch** (Revisión inteligente de constantes vitales): Este dispositivo realiza una revisión que mide distintos tipos de constantes vitales. Se puede usar para supervisar pacientes dentro y fuera del hospital. Si hace clic en la plantilla, verá que, además de enviar datos del dispositivo, como el nivel de batería y la temperatura del dispositivo, la revisión también envía datos del estado de los pacientes, como la frecuencia respiratoria y la presión arterial.

* **Smart Knee Brace** (Rodillera inteligente): este dispositivo representa una rodillera que los pacientes pueden usar al recuperarse de una cirugía de reemplazo de una rodilla. Si hace clic en esta plantilla, verá funcionalidades como el intervalo de movimiento y la aceleración, además de los datos del dispositivo.

>[!div class="mx-imgBorder"] 
>![Plantilla de dispositivo Smart Vitals Patch](media/smart-vitals-device-template.png)

### <a name="device-groups"></a>Grupos de dispositivos 
Los grupos de dispositivos permiten agrupar lógicamente un conjunto de dispositivos para así realizar consultas u operaciones masivas sobre ellos. 

Si hace clic en la pestaña de grupos de dispositivos, verá que hemos creado algunos grupos de dispositivos predeterminados para cada una de las plantillas de dispositivo de la aplicación. Observe que también hemos creado dos grupos de dispositivos de ejemplo adicionales, denominados "Provision devices" (Aprovisionamiento de dispositivos) y "Devices with outdated firmware" (Dispositivos con firmware sin actualizar). Usaremos estos grupos de dispositivos de ejemplo como entradas para ejecutar algunos [Trabajos](#jobs).

### <a name="rules"></a>Reglas

Al saltar a la pestaña de reglas, verá tres reglas que existen en la plantilla de aplicación:

* **Brace temperature high**(Temperatura alta de rodillera): esta regla se desencadena cuando la temperatura del dispositivo de la rodillera es mayor que 35&deg;C en un período de 5 minutos. Puede usar esta regla para avisar al equipo de asistencia sanitaria y al paciente, y enfriar el dispositivo de forma remota.

* **Fall detected** (Caída detectada): esta regla se desencadena si se detecta que un paciente se cayó. Puede configurar una acción con el fin de implementar un equipo operativo para ayudar al paciente que se cayera.

* **Patch battery low** (Revisión de poca batería): esta regla se desencadena cuando el nivel de batería del dispositivo desciende por debajo del 10 %. Puede usar esta regla para desencadenar una notificación al paciente para que cargue su dispositivo.

>[!div class="mx-imgBorder"] 
>![Brace temperature high rule](media/brace-temp-rule.png) (Regla de temperatura alta de rodillera)

### <a name="jobs"></a>Trabajos

Los trabajos le permiten ejecutar operaciones masivas en un conjunto de dispositivos con los [grupos de dispositivos](#device-groups) como entrada. Hemos propagado la plantilla de aplicación con dos trabajos de ejemplo que un operador de solución puede tener que ejecutar en algún momento del ciclo de vida de los dispositivos:
* **Update knee brace firmware** (Actualización del firmware de la rodillera): Este trabajo buscará dispositivos en el grupo de dispositivos "devices with outdated firmware" (dispositivos con firmware sin actualizar), y ejecutará un comando para actualizar dichos dispositivos a la versión de firmware más reciente de la rodillera. En este trabajo de ejemplo se supone que los dispositivos tienen funcionalidades para recibir un comando "update" y capturar los archivos de firmware directamente de la nube.  

* **Re-provision devices** (Reaprovisionamiento de dispositivos): Si tiene un conjunto de dispositivos que se han devuelto recientemente al hospital y tienen que volver a aprovisionarse para el siguiente conjunto de pacientes, puede ejecutar este trabajo para actualizar de forma masiva los dispositivos para aprovisionar. En este caso, vamos a tomar todos los dispositivos de un grupo de dispositivos denominado "provision device" (aprovisionamiento de dispositivos) y vamos a ejecutar un comando para aprovisionarlos. 

### <a name="devices"></a>Dispositivos

Haga clic en la pestaña **Dispositivos**  y, a continuación, seleccione una instancia de la **Smart Knee Brace** (Rodillera inteligente). Verá que hay tres vistas para explorar la información sobre el dispositivo concreto que ha seleccionado. Estas vistas se crean y publican al compilar la plantilla del dispositivo, lo que significa que serán coherentes en todos los dispositivos que se conecten o simulen.

La vista **Panel** ofrece información general sobre la telemetría y las propiedades que proceden del dispositivo que está orientada a los operadores.

La pestaña **Propiedades** le permitirá editar las propiedades de la nube y las propiedades del dispositivo de lectura/escritura.

La pestaña **Comandos** le permitirá ejecutar los comandos que se hayan modelado como parte de la plantilla del dispositivo.

>[!div class="mx-imgBorder"] 
>![Vistas de la rodillera](media/knee-brace-dashboard.png)

### <a name="data-export"></a>Exportación de datos

La exportación de datos permite exportar los datos del dispositivo de IoT Central de forma continua a otros servicios de Azure, [Azure API for FHIR](concept-continuous-patient-monitoring-architecture.md#export-to-azure-api-for-fhir) entre otros.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine su plantilla. Para ello, vaya a **Administración > Configuración de la aplicación** y haga clic en **Eliminar**.

>[!div class="mx-imgBorder"] 
>![Eliminar la aplicación](media/admin-delete.png)

## <a name="next-steps"></a>Pasos siguientes

Vaya al siguiente artículo para aprender a crear un panel de proveedor que se conecte a la aplicación IoT Central.

> [!div class="nextstepaction"]
> [Creación de un panel de proveedor](howto-health-data-triage.md)