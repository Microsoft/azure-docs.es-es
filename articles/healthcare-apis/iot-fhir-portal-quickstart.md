---
title: 'Inicio rápido: Implementación del conector de IoT (versión preliminar) con Azure Portal'
description: En esta guía de inicio rápido aprenderá a implementar, configurar y usar la característica del conector IoT de Azure API for FHIR mediante Azure Portal.
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: quickstart
ms.date: 05/11/2020
ms.author: punagpal
ms.openlocfilehash: 95f5b5f13401c224ccf67c5f013deedf00379de7
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446812"
---
# <a name="quickstart-deploy-iot-connector-preview-using-azure-portal"></a>Inicio rápido: Implementación del conector de IoT (versión preliminar) con Azure Portal

El conector de IoT es una característica opcional de Azure API for FHIR que proporciona la funcionalidad de ingesta de datos de dispositivos de Internet de las cosas médicas (IoMT). Y durante la fase de versión preliminar, la característica del conector de IoT está disponible de forma gratuita. En esta guía de inicio rápido, aprenderá a hacer lo siguiente:
- Implementación y configuración del conector de IoT mediante Azure Portal
- Uso de un dispositivo simulado para enviar datos al conector de IoT
- Visualización de los recursos creados por el conector de IoT en Azure API for FHIR

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción de Azure activa: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- Recurso de Azure API for FHIR: [Implementación de Azure API for FHIR mediante Azure Portal](fhir-paas-portal-quickstart.md)

## <a name="go-to-azure-api-for-fhir-resource"></a>Navegación al recurso de Azure API for FHIR

Abra [Azure Portal](https://portal.azure.com) y vaya al recurso **Azure API for FHIR** para el que quiere crear la característica del conector de IoT.

[![Recurso de Azure API for FHIR](media/quickstart-iot-fhir-portal/portal-azure-api-fhir.jpg)](media/quickstart-iot-fhir-portal/portal-azure-api-fhir.jpg#lightbox)

En el menú de navegación de la izquierda, haga clic en **Conector de IoT (versión preliminar)** en la sección **Complementos** para abrir la página **Conectores de IoT**.

[![Característica Conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connectors.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connectors.jpg#lightbox)

## <a name="create-new-iot-connector-preview"></a>Creación de un nuevo conector de IoT (versión preliminar)

Haga clic en el botón **Agregar** para abrir la página **Crear conector de IoT**.

[![Adición de un conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connectors-add.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connectors-add.jpg#lightbox)

Especifique la configuración del nuevo conector de IoT. Haga clic en el botón **Crear** y espere la implementación del conector de IoT.

> [!NOTE]
> Debe seleccionar **Crear** como valor del menú desplegable **Tipo de resolución** de esta instalación. 

[![Creación de un conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-create.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-create.jpg#lightbox)

|Configuración|Value|Descripción |
|---|---|---|
|Nombre del conector|Un nombre único|Escriba un nombre para identificar el conector de IoT. Este nombre debe ser único dentro de un recurso de Azure API for FHIR. El nombre solo puede contener letras minúsculas, números y el carácter de guion (-). Debe comenzar y terminar con una letra o un número, y debe tener entre 3 y 24 caracteres de longitud.|
|Tipo de resolución|Lookup o Create|Seleccione **Lookup** si tiene un proceso fuera de banda para crear recursos FHIR de tipo [Device](https://www.hl7.org/fhir/device.html) y [Patient](https://www.hl7.org/fhir/patient.html) en su instancia de Azure API for FHIR. El conector de IoT usará la referencia a estos recursos al crear un recurso FHIR de tipo [Observation](https://www.hl7.org/fhir/observation.html) para representar los datos del dispositivo. Seleccione **Crear** si quiere que el conector de IoT cree recursos de tipo Device y Patient esenciales en Azure API for FHIR con los respectivos valores de identificador presentes en los datos del dispositivo.|

Una vez completada la instalación, el conector de IoT recién creado se mostrará en la página **Conectores de IoT**.

[![Conector de IoT creado](media/quickstart-iot-fhir-portal/portal-iot-connector-created.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-created.jpg#lightbox)

## <a name="configure-iot-connector-preview"></a>Configuración del conector de IoT (versión preliminar)

El conector de IoT necesita dos plantillas de asignación para transformar los mensajes del dispositivo en recursos de tipo Observation basados en FHIR: **Asignación de dispositivo** y **Asignación de FHIR**. El conector de IoT no funciona por completo hasta que se carguen estas asignaciones.

[![Asignaciones faltantes del conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-missing-mappings.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-missing-mappings.jpg#lightbox)

Para cargar plantillas de asignación, haga clic en el conector de IoT recién implementado para ir a la página **Conector de IoT**.

[![Clic en el conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-click.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-click.jpg#lightbox)

#### <a name="device-mapping"></a>Asignación de dispositivos

La plantilla de asignación de dispositivos transforma los datos del dispositivo en un esquema normalizado. En la página **Conector de IoT**, haga clic en el botón **Configure device mapping** (Configurar asignación de dispositivos) para ir a la página de **Asignación de dispositivos**. 

[![Clic en la opción Configurar asignación de dispositivos del conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-click-device-mapping.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-click-device-mapping.jpg#lightbox)

En la página **Asignación de dispositivos**, agregue el siguiente script al editor de JSON y haga clic en **Guardar**.

```json
{
  "templateType": "CollectionContent",
  "template": [
    {
      "templateType": "IotJsonPathContent",
      "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@Body.HeartRate)]",
        "patientIdExpression": "$.SystemProperties.iothub-connection-device-id",
        "values": [
          {
            "required": "true",
            "valueExpression": "$.Body.HeartRate",
            "valueName": "hr"
          }
        ]
      }
    }
  ]
}
```

[![Asignación de dispositivos del conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-device-mapping.jpg)](media/quickstart-iot-fhir-portal/portal-iot-device-mapping.jpg#lightbox)


#### <a name="fhir-mapping"></a>Asignación de FHIR

La plantilla de asignación de FHIR transforma un mensaje normalizado en un recurso de tipo Observation basado en FHIR. En la página **Conector de IoT**, haga clic en el botón **Configure FHIR mapping** (Configurar asignación de FHIR) para ir a la página de **Asignación de FHIR**.  

[![Clic en la opción Configurar asignación de FHIR del conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-click-fhir-mapping.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-click-fhir-mapping.jpg#lightbox)

En la página **Asignación de FHIR**, agregue el siguiente script al editor de JSON y haga clic en **Guardar**.

```json
{
  "templateType": "CollectionFhir",
  "template": [
    {
      "templateType": "CodeValueFhir",
      "template": {
        "codes": [
          {
            "code": "8867-4",
            "system": "http://loinc.org",
            "display": "Heart rate"
          }
        ],
        "periodInterval": 0,
        "typeName": "heartrate",
        "value": {
          "unit": "count/min",
          "valueName": "hr",
          "valueType": "Quantity"
        }
      }
    }
  ]
}
```

[![Asignación de FHIR del conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-fhir-mapping.jpg)](media/quickstart-iot-fhir-portal/portal-iot-fhir-mapping.jpg#lightbox)

## <a name="generate-a-connection-string"></a>Generación de una cadena de conexión

El dispositivo IoMT necesita una cadena de conexión para conectarse y enviar mensajes al conector de IoT. En la página **Conector de IoT** para el conector de IoT recién implementado, seleccione el botón **Manage client connections** (Administrar conexiones de cliente). 

[![Clic en la opción Administrar conexiones de cliente del conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-click-client-connections.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-click-client-connections.jpg#lightbox)

Una vez en la página **Conexiones**, haga clic en el botón **Agregar** para crear una nueva conexión. 

[![Conexiones del conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connections.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connections.jpg#lightbox)

Proporcione un nombre descriptivo para esta conexión en la ventana superpuesta y seleccione el botón **Crear**.

[![Nueva conexión del conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-new-connection.jpg)](media/quickstart-iot-fhir-portal/portal-iot-new-connection.jpg#lightbox)

Seleccione la conexión recién creada desde la página **Conexiones** y copie el valor del campo **Cadena de conexión principal** de la ventana superpuesta de la derecha.

[![Cadena del conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connection-string.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connection-string.jpg#lightbox)

Conserve esta cadena de conexión para usarla en un paso posterior. 

## <a name="connect-your-devices-to-iot"></a>Conexión de sus dispositivos a IoT

Azure ofrece un amplio conjunto de productos de IoT para conectar y administrar sus dispositivos de IoT. Puede crear su propia solución basada en PaaS mediante Azure IoT Hub o empezar con una plataforma de administración de aplicaciones de IoT con Azure IoT Central. En este tutorial, aprovecharemos Azure IoT Central, que tiene plantillas de soluciones centradas en el sector como introducción.

Implemente la [plantilla de aplicación de supervisión continua de pacientes](https://docs.microsoft.com/azure/iot-central/healthcare/tutorial-continuous-patient-monitoring#create-an-application-template). Esta plantilla incluye dos dispositivos simulados que producen datos en tiempo real para ayudarle a comenzar: **Smart Vitals Patch** (parche de signos vitales inteligente) y **Smart Knee Brace** (rodillera inteligente).

> [!NOTE]
> Cada vez que los dispositivos reales estén listos, puede usar la misma aplicación de IoT Central para [incorporar sus dispositivos](https://docs.microsoft.com/azure/iot-central/core/howto-set-up-template) y reemplazar los simuladores de dispositivos. Los datos del dispositivo también comenzarán a fluir automáticamente a FHIR. 

## <a name="connect-your-iot-data-with-the-iot-connector-preview"></a>Conexión de los datos de IoT al conector de IoT (versión preliminar)
Una vez que haya implementado la aplicación de IoT Central, los dos dispositivos simulados listos para usar comenzarán a generar telemetría. En este tutorial, se ingerirá la telemetría del simulador *Smart Vitals Patch* en FHIR a través del conector de IoT. Para exportar los datos de IoT al conector de IoT, queremos [configurar una exportación de datos continua en IoT Central](https://docs.microsoft.com/azure/iot-central/core/howto-export-data#set-up-data-export). En la página Exportación de datos continua:
- Elija *Azure Event Hubs* como destino de exportación.
- Seleccione el valor *Usar una cadena de conexión* para el campo **Espacio de nombres de Event Hubs**.
- Proporcione la cadena de conexión del conector de IoT que obtuvo en un paso anterior en el campo **Cadena de conexión**.
- Mantenga la opción **Telemetría** *activada* para el campo **Datos para exportar**.

## <a name="view-device-data-in-azure-api-for-fhir"></a>Visualización de los datos del dispositivo en Azure API for FHIR

Puede ver los recursos de tipo Observation basados en FHIR que creó el conector de IoT en Azure API for FHIR mediante Postman. Configure [Postman para acceder a Azure API for FHIR](access-fhir-postman-tutorial.md) y realice una solicitud `GET` a `https://your-fhir-server-url/Observation?code=http://loinc.org|8867-4` para ver los recursos de tipo Observation de FHIR con el valor de la frecuencia cardíaca. 

> [!TIP]
> Asegúrese de que el usuario tiene acceso adecuado al plano de datos de Azure API for FHIR. Use el [control de acceso basado en roles de Azure (Azure RBAC)](configure-azure-rbac.md) para asignar los bloques del plano de datos requeridos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sean necesarios, puede eliminar una instancia del conector de IoT al quitar el grupo de recursos asociado, el servicio de Azure API for FHIR asociado, o bien la propia instancia del conector de IoT. 

Para quitar directamente una instancia del conector de IoT, seleccione la instancia en la página **Conectores de IoT** para ir a la página **Conector de IoT** y haga clic en el botón **Eliminar**. Seleccione **Sí** cuando se solicite confirmación. 

[![Eliminación de una instancia del conector de IoT](media/quickstart-iot-fhir-portal/portal-iot-connector-delete.jpg)](media/quickstart-iot-fhir-portal/portal-iot-connector-delete.jpg#lightbox)

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha implementado la característica del conector de IoT en su recurso de Azure API for FHIR. Seleccione entre los pasos siguientes para más información sobre el conector de IoT:

Comprenda las distintas fases del flujo de datos en el conector de IoT.

>[!div class="nextstepaction"]
>[Flujo de datos del conector de IoT](iot-data-flow.md)

Aprenda a configurar el conector de IoT mediante plantillas de asignación de dispositivos y FHIR.

>[!div class="nextstepaction"]
>[Plantillas de asignación del conector de IoT](iot-mapping-templates.md)

FHIR es la marca registrada de HL7 y se usa con su permiso.
