---
title: Actualización de Azure Load Balancer de básico interno a estándar interno
description: En este artículo se muestra cómo actualizar una instancia de Azure Load Balancer interno de una SKU básica a una estándar
services: load-balancer
author: irenehua
ms.service: load-balancer
ms.topic: how-to
ms.date: 08/07/2020
ms.author: irenehua
ms.openlocfilehash: a6d2b69b0b498601497c4b33fb6bdfede87002df
ms.sourcegitcommit: 206629373b7c2246e909297d69f4fe3728446af5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/06/2020
ms.locfileid: "89500256"
---
# <a name="upgrade-azure-internal-load-balancer--no-outbound-connection-required"></a>Actualización de Azure Load Balancer interno sin necesidad de conexión de salida
[Azure Standard Load Balancer](load-balancer-overview.md) ofrece un amplio conjunto de funcionalidades y alta disponibilidad gracias a la redundancia de zona. Para más información acerca de la SKU de Load Balancer, consulte la [tabla de comparación](https://docs.microsoft.com/azure/load-balancer/skus#skus).

En este artículo se presenta un script de PowerShell que crea una instancia de Standard Load Balancer con la misma configuración que la instancia básica de Load Balancer junto con la migración del tráfico desde la instancia básica hasta la estándar.

## <a name="upgrade-overview"></a>Información general sobre la actualización

Existe un script de Azure PowerShell que hace lo siguiente:

* Crea una instancia de Load Balancer interno de SKU estándar en la ubicación que se especifique. Tenga en cuenta que la instancia de Standard Load Balancer interno no proporciona ninguna [conexión de salida](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections).
* Copia perfectamente las configuraciones de Load Balancer de la SKU básica en la instancia de Standard Load Balancer recién creada.
* Mueve sin problemas las direcciones IP privadas de la instancia básica de Load Balancer a la instancia de Standard Load Balancer recién creada.
* Mueve sin problemas las máquinas virtuales del grupo de back-end de la instancia básica de Load Balancer al grupo de back-end de Standard Load Balancer.

### <a name="caveatslimitations"></a>Advertencias y limitaciones

* El script solo admite la actualización de Load Balancer interno si no se requiere ninguna conexión de salida. Si necesita [conexión de salida](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections) para alguna de las máquinas virtuales, vea esta [página](upgrade-InternalBasic-To-PublicStandard.md) para obtener instrucciones. 
* La instancia de Load Balancer básica debe estar en el mismo grupo de recursos que las NIC y VM de back-end.
* Si se crea la instancia de Standard Load Balancer en una región diferente, no podrá asociar las máquinas virtuales existentes de la región antigua a la instancia de Standard Load Balancer recién creada. Para solucionar esta limitación, asegúrese de crear una nueva máquina virtual en la nueva región.
* Si Load Balancer no tiene ninguna configuración de IP de front-end ni grupo de back-end, es probable que se produzca un error al ejecutar el script. Asegúrese de que no están vacíos.

## <a name="change-ip-allocation-method-to-static-for-frontend-ip-configuration-ignore-this-step-if-its-already-static"></a>Cambio del método de asignación de IP a Estática para la configuración de IP de front-end (ignore este paso si ya es estática)

1. Seleccione **Todos los servicios** en el menú de la izquierda, **Todos los recursos** y, después, en la lista de recursos, su instancia básica de Load Balancer.

2. En **Configuración**, seleccione **Configuración de IP de front-end** y seleccione la primera configuración de IP de front-end. 

3. Para **Asignación**, seleccione **Estática**.

4. Repita el paso 3 para todas las configuraciones de IP de front-end de la instancia básica de Load Balancer.


## <a name="download-the-script"></a>Descarga del script

Descargue el script de migración de la [Galería de PowerShell](https://www.powershellgallery.com/packages/AzureILBUpgrade/5.0).
## <a name="use-the-script"></a>Uso del script

Dispone de dos opciones en función de sus preferencias y de la configuración del entorno de PowerShell local:

* Si no tiene instalados los módulos de Azure Az, o si no le importa desinstalarlos, la mejor alternativa es usar la opción `Install-Script` para ejecutar el script.
* Si necesita conservar los módulos de Azure Az, lo mejor es que descargue el script y lo ejecute directamente.

Para determinar si tiene instalados los módulos de Azure Az, ejecute `Get-InstalledModule -Name az`. Si no ve ningún módulo de Az instalado, puede usar el método `Install-Script`.

### <a name="install-using-the-install-script-method"></a>Instalación con el método Install-Script

Para usar esta opción, los módulos de Azure Az no deben estar instalados en el equipo. En caso de que lo estén, el comando siguiente mostrará un error. Puede desinstalar los módulos de Azure Az o usar la otra opción para descargar manualmente el script y ejecutarlo.
  
Ejecute el script con el siguiente comando:

`Install-Script -Name AzureILBUpgrade`

Este comando también instala los módulos de Az necesarios.  

### <a name="install-using-the-script-directly"></a>Instalación directamente con el script

Si tiene instalados algunos módulos de Azure Az y no puede desinstalarlos (o no le interesa hacerlo), puede descargar manualmente el script mediante la pestaña **Descarga manual** en el vínculo de descarga del script. El script se descarga como un archivo nupkg sin procesar. Para instalar el script desde este archivo nupkg, consulte [Descarga manual del paquete](/powershell/scripting/gallery/how-to/working-with-packages/manual-download).

Para ejecutar el script:

1. Use `Connect-AzAccount` para conectarse a Azure.

1. Use `Import-Module Az` para importar los módulos de Az.

1. Examine los parámetros obligatorios:

   * **rgName: [String]: Required**: es el grupo de recursos de la instancia existente de Basic Load Balancer y la nueva instancia de Standard Load Balancer. Para encontrar este valor de cadena, vaya a Azure Portal, seleccione el origen Basic Load Balancer y haga clic en la sección **Información general** del equilibrador de carga. El grupo de recursos se encuentra en esa página.
   * **oldLBName: [String]: Required**: es el nombre de la instancia de Basic Load Balancer que desea actualizar. 
   * **newlocation: [String]: Required**: es la ubicación en la que se creará la instancia de Standard Load Balancer. Se recomienda heredar la misma ubicación de la instancia elegida de Basic Load Balancer para la instancia de Standard Load Balancer para una mejor asociación con otros recursos existentes.
   * **newLBName: [String]: Required**: es el nombre de la instancia de Standard Load Balancer que se va a crear.
1. Ejecute el script con los parámetros adecuados. Podría tardar entre cinco y siete minutos en finalizar.

    **Ejemplo**

   ```azurepowershell
   AzureILBUpgrade.ps1 -rgName "test_InternalUpgrade_rg" -oldLBName "LBForInternal" -newlocation "centralus" -newLbName "LBForUpgrade"
   ```

## <a name="common-questions"></a>Preguntas frecuentes

### <a name="are-there-any-limitations-with-the-azure-powershell-script-to-migrate-the-configuration-from-v1-to-v2"></a>¿Hay alguna limitación en el script de Azure PowerShell para migrar la configuración de v1 a v2?

Sí. Consulte [Advertencias y limitaciones](#caveatslimitations).

### <a name="does-the-azure-powershell-script-also-switch-over-the-traffic-from-my-basic-load-balancer-to-the-newly-created-standard-load-balancer"></a>¿Puede el script de Azure PowerShell cambiar el tráfico de la instancia de Basic Load Balancer a la instancia de Standard Load Balancer recién creada?

Sí, lo migra. Si quiere migrar el tráfico de forma personal, use [este script](https://www.powershellgallery.com/packages/AzureILBUpgrade/1.0) que no mueve las máquinas virtuales automáticamente.

### <a name="i-ran-into-some-issues-with-using-this-script-how-can-i-get-help"></a>Se han producido algunos problemas al usar este script. ¿Cómo puedo obtener ayuda?
  
Puede enviar un correo electrónico a slbupgradesupport@microsoft.com, abrir una incidencia con el Soporte técnico de Azure o hacer ambas cosas.

## <a name="next-steps"></a>Pasos siguientes

[Más información sobre Load Balancer estándar](load-balancer-overview.md)
