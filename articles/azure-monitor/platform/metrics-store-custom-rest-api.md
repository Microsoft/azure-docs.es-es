---
title: Envío de métricas a la base de datos de métricas de Azure Monitor mediante la API de REST
description: Envío de métricas personalizadas de un recurso de Azure al almacén de métricas de Azure Monitor mediante la API REST
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ancav
ms.subservice: metrics
ms.openlocfilehash: c315cc8c9dfcfa66999ff263fab95f414061e54e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87321265"
---
# <a name="send-custom-metrics-for-an-azure-resource-to-the-azure-monitor-metric-store-by-using-a-rest-api"></a>Envío de métricas personalizadas de un recurso de Azure al almacén de métricas de Azure Monitor mediante la API REST

En este artículo muestra cómo enviar métricas personalizadas de los recursos de Azure al almacén de métricas Azure Monitor a través de una API de REST. Una vez que las métricas estén en Azure Monitor, puede hacer con ellas lo mismo que con las métricas estándar. Por ejemplo, puede crear gráficos o alertas, y enrutarlos a otras herramientas externas.  

>[!NOTE]  
>La API de REST solo permite el envío de métricas personalizadas de los recursos de Azure. Para enviar métricas de recursos en distintos entornos o de forma local, puede usar [Application Insights](../app/api-custom-events-metrics.md).    


## <a name="create-and-authorize-a-service-principal-to-emit-metrics"></a>Creación y autorización de una entidad de servicio para la emisión de métricas 

Cree una entidad de servicio en el inquilino de Azure Active Directory según las instrucciones que encontrará en [Creación de una entidad de servicio](../../active-directory/develop/howto-create-service-principal-portal.md). 

Tenga en cuenta lo siguiente al realizar este proceso: 

- Puede escribir cualquier dirección URL para la dirección URL de inicio de sesión.  
- Cree un nuevo secreto de cliente para esta aplicación.  
- Guarde la clave y el identificador de cliente para usarlos en pasos posteriores.  

Asigne a la aplicación que creó como parte del paso 1, Supervisión del publicador de métricas, permisos para el recurso frente al cual desea emitir métricas. Si va a usar la aplicación para emitir métricas personalizadas frente a muchos recursos, puede conceder estos permisos en el nivel de suscripción o de grupo de recursos. 

## <a name="get-an-authorization-token"></a>Obtención de un token de autorización
Abra un símbolo del sistema y ejecute el comando siguiente:

```shell
curl -X POST https://login.microsoftonline.com/<yourtenantid>/oauth2/token -F "grant_type=client_credentials" -F "client_id=<insert clientId from earlier step>" -F "client_secret=<insert client secret from earlier step>" -F "resource=https://monitoring.azure.com/"
```
Guarde el token de acceso de la respuesta.

![Access token](./media/metrics-store-custom-rest-api/accesstoken.png)

## <a name="emit-the-metric-via-the-rest-api"></a>Emisión de la métrica a través de la API de REST 

1. Pegue el siguiente código JSON en un archivo y guárdelo como  **custommetric.json** en el equipo local. Actualice el parámetro de tiempo en el archivo JSON: 
    
    ```json
    { 
        "time": "2018-09-13T16:34:20", 
        "data": { 
            "baseData": { 
                "metric": "QueueDepth", 
                "namespace": "QueueProcessing", 
                "dimNames": [ 
                  "QueueName", 
                  "MessageType" 
                ], 
                "series": [ 
                  { 
                    "dimValues": [ 
                      "ImagesToProcess", 
                      "JPEG" 
                    ], 
                    "min": 3, 
                    "max": 20, 
                    "sum": 28, 
                    "count": 3 
                  } 
                ] 
            } 
        } 
    } 
    ``` 

1. En la ventana del símbolo del sistema, publique los datos de la métrica: 
   - **azureRegion**. Debe coincidir con la región de implementación del recurso para el que se van a emitir las métricas. 
   - **resourceID**.  El identificador de recurso del recurso de Azure con respecto al que realiza el seguimiento de la métrica.  
   - **AccessToken**. Pegue el token que haya adquirido anteriormente.

     ```Shell 
     curl -X POST https://<azureRegion>.monitoring.azure.com/<resourceId>/metrics -H "Content-Type: application/json" -H "Authorization: Bearer <AccessToken>" -d @custommetric.json 
     ```
1. Cambie la marca de tiempo y los valores del archivo JSON. 
1. Repita los dos pasos anteriores varias veces para tener datos durante varios minutos.

## <a name="troubleshooting"></a>Solución de problemas 
Si recibe un mensaje de error en alguna parte del proceso, tenga en cuenta la siguiente información de solución de problemas:

1. No se pueden emitir métricas con respecto a una suscripción o grupo de recursos como el recurso de Azure. 
1. No se puede colocar en el almacén una métrica si pasan más de 20 minutos desde su creación. El almacén de métricas está optimizado para las alertas y los gráficos en tiempo real. 
2. El número de nombres de dimensión debe coincidir con los valores y viceversa. Compruebe los valores. 
2. Se pueden emitir métricas con respecto a una región que no es compatible con las métricas personalizadas. Consulte las [regiones admitidas](./metrics-custom-overview.md#supported-regions). 



## <a name="view-your-metrics"></a>Visualización de las métricas 

1. Inicie sesión en Azure Portal. 

1. En el menú de la izquierda, seleccione **Monitor**. 

1. En la página **Monitor**, seleccione **Métricas**. 

   ![Seleccione Métricas](./media/metrics-store-custom-rest-api/metrics.png) 

1. Cambie el período de agregación a **Últimos 30 minutos**.  

1. En el menú desplegable **Recursos**, seleccione el recurso con respecto al que emitió la métrica.  

1. En el menú desplegable **Espacios de nombres**, seleccione **QueueProcessing**. 

1. En el menú desplegable **Métricas**, seleccione **QueueDepth**.  

 
## <a name="next-steps"></a>Pasos siguientes
- Más información acerca de las [métricas personalizadas](./metrics-custom-overview.md).

