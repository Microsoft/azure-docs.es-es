---
title: 'Registrar el flujo del tráfico de red desde y hacia una VM (tutorial): Azure Portal | Microsoft Docs'
description: Obtenga información acerca de cómo registrar el flujo de tráfico de red desde y hacia una VM mediante la funcionalidad de registro de flujos de NSG de Network Watcher.
services: network-watcher
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I need to log the network traffic to and from a VM so I can analyze it for anomalies.
ms.assetid: 01606cbf-d70b-40ad-bc1d-f03bb642e0af
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/30/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: 3c62ad66a29943e26d1cb2f15ca71631d2feabe3
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467423"
---
# <a name="tutorial-log-network-traffic-to-and-from-a-virtual-machine-using-the-azure-portal"></a>Tutorial: Registro del tráfico de red de entrada y salida de una máquina virtual mediante Azure Portal

Un grupo de seguridad de red (NSG) le permite filtrar el tráfico entrante y el tráfico saliente en una máquina virtual (VM). Puede registrar el tráfico de red que fluye a través de un NSG gracias a la capacidad de registro de flujos de NSG de Network Watcher. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un grupo de seguridad de red con una máquina virtual.
> * Habilitar Network Watcher y registrar un proveedor de Microsoft.Insights.
> * Habilitar un registro de flujo de tráfico para un NSG, mediante la capacidad de registro de flujos de NSG de Network Watcher.
> * Descargar los datos registrados.
> * Ver los datos registrados.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-a-vm"></a>Crear una VM

1. Seleccione **+ Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Seleccione **Proceso** y, luego, **Windows Server 2016 Datacenter** o una versión de **Ubuntu Server**.
3. Escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Aceptar**:

    |Configuración|Valor|
    |---|---|
    |NOMBRE|myVm|
    |Nombre de usuario| Escriba un nombre de usuario de su elección.|
    |Password| Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    |Subscription| Seleccione su suscripción.|
    |Grupos de recursos| Haga clic en **Crear nuevo** y escriba **myResourceGroup**.|
    |Ubicación| Seleccione **Este de EE. UU**.|

4. Seleccione un tamaño para la máquina virtual y luego **Seleccionar**.
5. En **Configuración**, acepte todos los valores predeterminados y seleccione **Aceptar**.
6. En **Crear** de la página **Resumen**, seleccione **Crear** para iniciar la implementación de la máquina virtual. La maquina virtual tarda unos minutos en implementarse. Espere a que la VM finalice la implementación antes de continuar con los pasos restantes.

La máquina virtual tarda en crearse unos minutos. No continúe con los pasos restantes hasta que la VM haya finalizado el proceso de creación. Además de crear la máquina virtual, el portal también crea un grupo de seguridad de red denominado **myVm nsg** y lo asocia a la interfaz de red de la máquina virtual.

## <a name="enable-network-watcher"></a>Habilitación de Network Watcher

Si ya dispone de un monitor de red habilitado en la región este de EE. UU., vaya al [Registro del proveedor de Insights](#register-insights-provider).

1. En el portal, seleccione **Todos los servicios**. En el **cuadro Filtrar**, escriba *Network Watcher*. Cuando aparezca la opción **Network Watcher** en los resultados, selecciónela.
2. Seleccione **Regiones** para expandirla y, a continuación, seleccione **...**  a la derecha de la opción **Este de EE. UU.**, tal y como se muestra en la siguiente imagen:

    ![Habilitación de Network Watcher](./media/network-watcher-nsg-flow-logging-portal/enable-network-watcher.png)

3. Seleccione **Habilitar Network Watcher**.

## <a name="register-insights-provider"></a>Registro del proveedor de Insights

Para iniciar sesión en el flujo de NSG, es necesario recurrir al proveedor **Microsoft.Insights**. Para registrar el proveedor, realice los pasos siguientes:

1. En la esquina superior izquierda del portal, seleccione **Todos los servicios**. En el cuadro Filtro, escriba *Suscripciones*. Cuando aparezca la opción **Suscripciones** en los resultados de la búsqueda, selecciónela.
2. En la lista de suscripciones, seleccione la suscripción con la que quiera habilitar el proveedor.
3. Seleccione **Proveedores de recursos** en **CONFIGURACIÓN**.
4. Confirme que el **ESTADO** del proveedor **microsoft.insights** está **registrado**, tal y como se muestra en la figura siguiente. Si el estado está **sin registrar**, seleccione **Registrar** a la derecha del proveedor.

    ![Registrar el proveedor](./media/network-watcher-nsg-flow-logging-portal/register-provider.png)

## <a name="enable-nsg-flow-log"></a>Habilitar el registro de flujos de NSG

1. Los datos del registro de flujos de NSG se escriben en una cuenta de Azure Storage. Para crear una cuenta de Azure Storage, seleccione **+ Create a resource** (+ Crear un recurso) en la esquina superior izquierda del portal.
2. Seleccione **Storage** y, a continuación, seleccione **Storage account - blob, file, table, queue** (Cuenta de almacenamiento: blob, archivo, tabla, cola).
3. Escriba o seleccione la siguiente información, acepte los valores predeterminados restantes y haga clic en **Crear**:

    | Configuración        | Valor                                                        |
    | ---            | ---   |
    | NOMBRE           | Debe tener entre 3 y 24 caracteres de longitud y solo puede contener letras minúsculas y números; asimismo, debe ser único para todas las cuentas de Azure Storage.                                                               |
    | Ubicación       | Seleccione **Este de EE. UU**.                                           |
    | Grupos de recursos | Seleccione **Usar existente** y, luego, seleccione **myResourceGroup** |

    La cuenta de almacenamiento tardará unos minutos en crearse. No continúe con los pasos restantes hasta que haya creado la cuenta de almacenamiento. Si va a usar una cuenta de almacenamiento existente en vez de crear una, asegúrese de seleccionar la cuenta de almacenamiento que tenga la opción **Todas las redes** (valor predeterminado) seleccionada en **Firewalls y redes virtuales**, en la **CONFIGURACIÓN** de la cuenta de almacenamiento.
    
    > [!NOTE]
    > Aunque actualmente los proveedores de Microsoft.Insight y Microsoft.Network son compatibles con Azure Storage, los registros de Flow de NSG aún no se han incorporado totalmente. Para habilitar el registro de Flow de NSG, debe seleccionarse **Todas las redes** hasta que esta característica esté plenamente incorporada. 
4. En la esquina superior izquierda del portal, seleccione **Todos los servicios**. En el cuadro **Filtrar**, escriba *Network Watcher*. Cuando aparezca la opción **Network Watcher** en los resultados de búsqueda, selecciónela.
5. En **REGISTROS**, seleccione **Registro de flujos de NSG**, tal y como se muestra en la siguiente imagen:

    ![Grupos de seguridad de red](./media/network-watcher-nsg-flow-logging-portal/nsgs.png)

6. En la lista de NSG, seleccione el NSG denominado **myVm-nsg**.
7. En **Configuración de los registros de flujo**, seleccione **Activada**.
8. Seleccione la versión del registro de flujos. La versión 2 contiene estadísticas de la sesión de flujo (Bytes y paquetes)

   ![Seleccionar la versión de los registros de flujo](./media/network-watcher-nsg-flow-logging-portal/select-flow-log-version.png)

9. Seleccione la cuenta de almacenamiento que creó anteriormente en el paso 3:
10. Establezca el valor de **Retención (días)** en 5 y, a continuación, seleccione **Guardar**.

## <a name="download-flow-log"></a>Descargar el registro de flujos

1. En el portal de Network Watcher, seleccione **Registro de flujos de NSG** en **REGISTROS**.
2. Seleccione **You can download flow logs from configured storage accounts** (Puede descargar los registros de flujo desde cuentas de almacenamiento configuradas), tal como se muestra en la siguiente imagen.

   ![Descarga de registros de flujo](./media/network-watcher-nsg-flow-logging-portal/download-flow-logs.png)

3. Seleccione la cuenta de almacenamiento que configuró en el paso 2 de la opción [Habilitar los registros de flujo de NSG](#enable-nsg-flow-log).
4. En **Blob service** seleccione **Blobs** y, a continuación, seleccione el contenedor **insights-logs-networksecuritygroupflowevent**.
5. En el contenedor, explore la jerarquía de carpetas hasta llegar a un archivo PT1H.json, tal como se muestra en la imagen a continuación. Los archivos de registro se escriben en una jerarquía de carpetas que tiene la siguiente convención de nomenclatura: https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json

   ![Registro de flujo](./media/network-watcher-nsg-flow-logging-portal/log-file.png)

6. Seleccione **...** a la derecha del archivo PT1H.json de y **Descargar**.

## <a name="view-flow-log"></a>Ver el registro de flujos

El siguiente formato JSON es un ejemplo de lo que verá en el archivo PT1H.json para cada flujo en el que se registran los datos:

### <a name="version-1-flow-log-event"></a>Evento de registro de flujo de la versión 1
```json
{
    "time": "2018-05-01T15:00:02.1713710Z",
    "systemId": "<Id>",
    "category": "NetworkSecurityGroupFlowEvent",
    "resourceId": "/SUBSCRIPTIONS/<Id>/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/MYVM-NSG",
    "operationName": "NetworkSecurityGroupFlowEvents",
    "properties": {
        "Version": 1,
        "flows": [
            {
                "rule": "UserRule_default-allow-rdp",
                "flows": [
                    {
                        "mac": "000D3A170C69",
                        "flowTuples": [
                            "1525186745,192.168.1.4,10.0.0.4,55960,3389,T,I,A"
                        ]
                    }
                ]
            }
        ]
    }
}
```
### <a name="version-2-flow-log-event"></a>Evento de registro de flujo de la versión 2
```json
{
    "time": "2018-11-13T12:00:35.3899262Z",
    "systemId": "a0fca5ce-022c-47b1-9735-89943b42f2fa",
    "category": "NetworkSecurityGroupFlowEvent",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
    "operationName": "NetworkSecurityGroupFlowEvents",
    "properties": {
        "Version": 2,
        "flows": [
            {
                "rule": "DefaultRule_DenyAllInBound",
                "flows": [
                    {
                        "mac": "000D3AF87856",
                        "flowTuples": [
                            "1542110402,94.102.49.190,10.5.16.4,28746,443,U,I,D,B,,,,",
                            "1542110424,176.119.4.10,10.5.16.4,56509,59336,T,I,D,B,,,,",
                            "1542110432,167.99.86.8,10.5.16.4,48495,8088,T,I,D,B,,,,"
                        ]
                    }
                ]
            },
            {
                "rule": "DefaultRule_AllowInternetOutBound",
                "flows": [
                    {
                        "mac": "000D3AF87856",
                        "flowTuples": [
                            "1542110377,10.5.16.4,13.67.143.118,59831,443,T,O,A,B,,,,",
                            "1542110379,10.5.16.4,13.67.143.117,59932,443,T,O,A,E,1,66,1,66",
                            "1542110379,10.5.16.4,13.67.143.115,44931,443,T,O,A,C,30,16978,24,14008",
                            "1542110406,10.5.16.4,40.71.12.225,59929,443,T,O,A,E,15,8489,12,7054"
                        ]
                    }
                ]
            }
        ]
    }
}
```

El valor de **mac** en la salida anterior es la dirección MAC de la interfaz de red que se creó cuando creó la máquina virtual. La información separada por comas de **flowTuples** tiene el siguiente aspecto:

| Datos de ejemplo | Qué representan los datos   | Explicación                                                                              |
| ---          | ---                    | ---                                                                                      |
| 1542110377   | Marca de tiempo             | La marca de tiempo de cuando se produjo el flujo en formato UNIX EPOCH. En el ejemplo anterior, la fecha se convierte en el 1 de mayo de 2018 a las 2:59:05 P.M. GMT.                                                                                    |
| 10.0.0.4  | Dirección IP de origen      | La dirección IP de origen en la que se ha originado el flujo. 10.0.0.4 es la dirección IP privada de la VM que creó en el paso [Crear una máquina virtual](#create-a-vm).
| 13.67.143.118     | Dirección IP de destino | La dirección IP de destino a la que se destina el flujo.                                                                                  |
| 44931        | Puerto de origen            | El puerto de origen en el que se ha originado el flujo.                                           |
| 443         | Puerto de destino       | El puerto de destino al que se destina el flujo. Puesto que el tráfico se destinó al puerto 443, la regla denominada **UserRule_default-allow-rdp**, en el archivo de registro, es la que procesa el flujo.                                                |
| T            | Protocolo               | Si el protocolo del flujo es TCP (T) o UDP (U).                                  |
| O            | Dirección              | Si el tráfico es entrante (I) o saliente (O).                                     |
| Una             | .                 | Si el tráfico está permitido (A) o si está denegado (D).  
| C            | Estado de flujo **solo versión 2** | Captura el estado del flujo. Los estados posibles son**B**: Begin (Comienzo), cuando se crea el flujo. No se proporcionan las estadísticas. **C**: Continuación de un flujo en curso. Las estadísticas se proporcionan a intervalos de cinco minutos. **E**: End (Final), cuando termina el flujo. Se proporcionan las estadísticas. |
| 30 | Paquetes enviados: origen a destino **solo versión 2** | El número total de paquetes TCP o UDP enviados desde el origen al destino desde la última actualización. |
| 16978 | Bytes enviados: origen a destino **solo versión 2** | El número total de bytes de paquetes TCP o UDP enviados desde el origen al destino desde la última actualización. Los bytes de paquete incluyen el encabezado y la carga del paquete. | 
| 24 | Paquetes enviados: destino a origen **solo versión 2** | El número total de paquetes TCP o UDP enviados desde el destino al origen desde la última actualización. |
| 14008| Bytes enviados: destino a origen **solo versión 2** | El número total de bytes de paquetes TCP y UDP enviados desde el destino al origen desde la última actualización. Los bytes de paquete incluyen el encabezado y la carga del paquete.|

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a habilitar el registro de flujos de NSG para un grupo de seguridad de red o NSG. Asimismo, también aprendió a descargar y ver los datos registrados en un archivo. Los datos sin procesar del archivo json pueden ser difíciles de interpretar. Para visualizar los datos, puede usar el [análisis del tráfico](traffic-analytics.md) de Network Watcher, Microsoft [PowerBI](network-watcher-visualize-nsg-flow-logs-power-bi.md) y otras herramientas.
