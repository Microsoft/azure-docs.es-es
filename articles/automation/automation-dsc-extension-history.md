---
title: Trabajo con el historial de versiones de la extensión Desired State Configuration de Azure
description: En este artículo se explica cómo trabajar con el historial de versiones para la extensión Desired State Configuration (DSC) de Azure.
ms.date: 07/22/2020
keywords: dsc, powershell, azure, extensión
services: automation
ms.subservice: dsc
ms.topic: conceptual
ms.openlocfilehash: b45512faf09cfe745023d29d32f89a4432cc3b2b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079764"
---
# <a name="work-with-azure-desired-state-configuration-extension-version-history"></a>Trabajo con el historial de versiones de la extensión Desired State Configuration de Azure

La extensión para máquinas virtuales Desired State Configuration (DSC) de Azure se actualiza cuando es necesario para incluir compatibilidad con mejoras y nuevas funciones de Azure, Windows Server y Windows Management Framework (WMF), que incluye Windows PowerShell.

Este artículo ofrece información sobre cada versión de la extensión DSC de Azure para máquinas virtuales y los entornos con los que es compatible, así como comentarios sobre nuevas características o cambios.

## <a name="latest-version"></a>La versión más reciente

### <a name="version-280"></a>Versión 2.80

- **Fecha de publicación:**
  - 26 de septiembre de 2019 (Azure) | 6 de julio de 2020 (Azure China ViaNet 21) | 20 de julio de 2020 (Azure Government)
- **Sistemas operativos admitidos:**
  - Windows Server 2019
  - Windows Server 2016
  - Windows Server 2012 R2
  - Windows Server 2012
  - Windows Server 2008 R2 SP1
  - Cliente de Windows 7/8.1/10
  - Nano Server
- **Compatibilidad con WMF:**
  - WMF 5.1
  - WMF 5.0 RTM
  - Actualización de WMF 4.0
  - WMF 4.0
- **Entorno:**
  - Azure
  - Azure China Vianet 21
  - Azure Government
- **Observaciones**: En esta versión no se incluye ninguna característica nueva.

### <a name="version-276"></a>Versión 2.76

- **Fecha de publicación:**
  - 9 de mayo de 2018 (Azure) | 21 de junio de 2018 (Azure China Vianet 21 y Azure Government)
- **Sistemas operativos admitidos:**
  - Windows Server 2016
  - Windows Server 2012 R2
  - Windows Server 2012
  - Windows Server 2008 R2 SP1
  - Cliente de Windows 7/8.1/10
  - Nano Server
- **Compatibilidad con WMF:**
  - WMF 5.1
  - WMF 5.0 RTM
  - Actualización de WMF 4.0
  - WMF 4.0
- **Entorno:**
  - Azure
  - Azure China Vianet 21
  - Azure Government
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/). Después de instalarlo, tendrá que reiniciar el equipo. En el caso de Nano Server, el rol de DSC se instala en la máquina virtual.
- **Nuevas características:**
  - Se han mejorado los metadatos de extensión de los subestados y se han aplicado otras correcciones de errores.

## <a name="supported-versions"></a>Versiones compatibles

> [!WARNING]
> De la versión 2.4 a la 2.13 se usa WMF 5.0 Public Preview, cuyos certificados de firma expiraron en agosto.
> 2016. Para obtener más información sobre este problema, vea la [entrada de blog](https://devblogs.microsoft.com/powershell/azure-dsc-extension-versions-2-4-up-to-2-13-will-retire-in-august/).

### <a name="version-275"></a>Versión 2.75

- **Fecha de publicación:** 5 de marzo de 2018
- **Sistemas operativos admitidos:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Windows Client 7/8.1/10 y Nano Server
- **Compatibilidad con WMF:** WMF 5.1, WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/). Después de instalarlo, tendrá que reiniciar el equipo. En el caso de Nano Server, el rol de DSC se instala en la máquina virtual.
- **Nuevas características:**
  - Debido a que ahora GitHub usa TLS 1.2, ya no puede incorporar una máquina virtual a DSC de Azure Automation con las plantillas personalizadas de Resource Manager disponibles en Azure Marketplace ni usar la extensión DSC para obtener cualquier configuración hospedada en GitHub. Verá un error similar al siguiente durante la implementación de la extensión:

    ```json
    {
        "code": "DeploymentFailed",
        "message": "At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.",
        "details": [{
            "code": "Conflict",
            "message": "{
                \"status\": \"Failed\",
                \"error\": {
                    \"code\": \"ResourceDeploymentFailure\",
                    \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.\",
                    \"details\": [ {
                        \"code\": \"VMExtensionProvisioningError\",
                        \"message\": \"VM has reported a failure when processing extension 'Microsoft.Powershell.DSC'.
                        Error message: \\\"The DSC Extension failed to execute: Error downloading
                        https://github.com/Azure/azure-quickstart-templates/raw/master/dsc-extension-azure-automation-pullserver/UpdateLCMforAAPull.zip
                        after 29 attempts: The request was aborted: Could not create SSL/TLS secure channel..\\nMore information about the failure can
                        be found in the logs located under 'C:\\\\WindowsAzure\\\\Logs\\\\Plugins\\\\Microsoft.Powershell.DSC\\\\2.74.0.0' on the VM.\\\".\"
                    } ]
                }
            }"
        }]
    }
    ```

  - En la nueva versión de la extensión se usa obligatoriamente TLS 1.2. Al implementar la extensión, si ya tenía AutoUpgradeMinorVersion establecido en "true" en la plantilla de Resource Manager, la extensión se actualizará automáticamente a la versión 2.75. En el caso de las actualizaciones manuales, especifique `TypeHandlerVersion = 2.75` en la plantilla de Resource Manager.

### <a name="version-270---272"></a>Versiones 2.70 - 2.72

- **Fecha de publicación:** 13 de noviembre de 2017
- **Sistemas operativos admitidos:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Windows Client 7/8.1/10 y Nano Server
- **Compatibilidad con WMF:** WMF 5.1, WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/). Después de instalarlo, tendrá que reiniciar el equipo. En el caso de Nano Server, el rol de DSC se instala en la máquina virtual.
- **Nuevas características:**
  - Se han aplicado correcciones de errores y mejoras que simplifican el uso de DSC con Azure Automation mediante la interfaz de usuario del portal, así como la plantilla de Resource Manager. Para obtener más información, vea [Script de configuración predeterminada](../virtual-machines/extensions/dsc-overview.md) en la documentación de la extensión DSC.

### <a name="version-226"></a>Versión 2.26

- **Fecha de publicación:** 9 de junio de 2017
- **Sistemas operativos admitidos:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Windows Client 7/8.1/10 y Nano Server
- **Compatibilidad con WMF:** WMF 5.1, WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/). Después de instalarlo, tendrá que reiniciar el equipo. En el caso de Nano Server, el rol de DSC se instala en la máquina virtual.
- **Nuevas características:**
  - Se han realizado mejoras de telemetría.

### <a name="version-225"></a>Versión 2.25

- **Fecha de publicación:** 2 de junio de 2017
- **Sistemas operativos admitidos:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Windows Client 7/8.1/10 y Nano Server
- **Compatibilidad con WMF:** WMF 5.1, WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/). Después de instalarlo, tendrá que reiniciar el equipo. En el caso de Nano Server, el rol de DSC se instala en la máquina virtual.
- **Nuevas características:**
  - Se han corregido varios errores y se han agregado mejoras menores.

### <a name="version-224"></a>Versión 2.24

- **Fecha de publicación:** 13 de abril de 2017
- **Sistemas operativos admitidos:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Nano Server
- **Compatibilidad con WMF:** WMF 5.1, WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/). Después de instalarlo, tendrá que reiniciar el equipo. En el caso de Nano Server, el rol de DSC se instala en la máquina virtual.
- **Nuevas características:**
  - Expone el id. de agente de DSC y el UUID de la máquina virtual como metadatos de la extensión. Se han agregado otras mejoras menores.

### <a name="version-223"></a>Versión 2.23

- **Fecha de publicación:** 15 de marzo de 2017
- **Sistemas operativos admitidos:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Nano Server
- **Compatibilidad con WMF:** WMF 5.1, WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/). Después de instalarlo, tendrá que reiniciar el equipo. En el caso de Nano Server, el rol de DSC se instala en la máquina virtual.
- **Nuevas características:**
  - Se han corregido muchos errores y se han agregado otras mejoras.

### <a name="version-222"></a>Versión 2.22

- **Fecha de publicación:** 8 de febrero de 2017
- **Sistemas operativos admitidos:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Nano Server
- **Compatibilidad con WMF:** WMF 5.1, WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/). Después de instalarlo, tendrá que reiniciar el equipo. En el caso de Nano Server, el rol de DSC se instala en la máquina virtual.
- **Nuevas características:**
  - La extensión DSC ahora es compatible con WMF 5.1.
  - Se han agregado otras mejoras menores.

### <a name="version-221"></a>Versión 2.21

- **Fecha de publicación:** 2 de diciembre de 2016
- **Sistemas operativos admitidos:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Nano Server
- **Compatibilidad con WMF:** WMF 5.1 Preview, WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/). Después de instalarlo, tendrá que reiniciar el equipo. En el caso de Nano Server, el rol de DSC se instala en la máquina virtual.
- **Nuevas características:**
  - La extensión DSC ahora está disponible en Nano Server. Esta versión contiene principalmente cambios de código para ejecutar la extensión en Nano Server.
  - Se han agregado otras mejoras menores.

### <a name="version-220"></a>Versión 2.20

- **Fecha de publicación:** 2 de agosto de 2016
- **Sistemas operativos admitidos:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Compatibilidad con WMF:** WMF 5.1 Preview, WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016 Technical Preview. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/). Después de instalarlo, tendrá que reiniciar el equipo.
- **Nuevas características:**
  - Se ha agregado la compatibilidad con WMF 5.1 Preview. Cuando se publicó por primera vez, esta versión era una actualización opcional y tenía que especificar Wmfversion = '5.1PP' en las plantillas de Resource Manager para instalar la versión preliminar de WMF 5.1.
    Wmfversion = 'latest' todavía instala [WMF 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/).
    Para obtener más información sobre la versión preliminar de WMF 5.1, vea [este blog](https://devblogs.microsoft.com/powershell/announcing-windows-management-framework-wmf-5-1-preview/).
  - Se han agregado correcciones y otras mejoras menores.

### <a name="version--219"></a>Versión 2.19

- **Fecha de publicación:** 3 de junio de 2016
- **Sistemas operativos admitidos:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Compatibilidad con WMF:** WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure, Azure China Vianet 21, Azure Government
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016 Technical Preview. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/). Después de instalarlo, tendrá que reiniciar el equipo.
- **Nuevas características:**
  - Ahora la extensión DSC está incorporada en Azure China Vianet 21. Esta versión contiene principalmente correcciones para ejecutar la extensión en Azure China Vianet 21.

### <a name="version-218"></a>Versión 2.18

- **Fecha de publicación:** 3 de junio de 2016
- **Sistemas operativos admitidos:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Compatibilidad con WMF:** WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016 Technical Preview. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/). Después de instalarlo, tendrá que reiniciar el equipo.
- **Nuevas características:**
  - La telemetría no bloquea contenido cuando se produce un error durante la descarga de la revisión de telemetría (problema conocido de Azure DNS) ni durante la instalación.
  - Se ha corregido el problema intermitente que producía que la extensión dejase de procesar la configuración después de un reinicio.
    Esto provocaba que la extensión DSC se mantuviese en estado de "transición".
  - Se han agregado correcciones y otras mejoras menores.

### <a name="version-217"></a>Versión 2.17

- **Fecha de publicación:** 26 de abril de 2016
- **Sistemas operativos admitidos:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Compatibilidad con WMF:** WMF 5.0 RTM, actualización de WMF 4.0 y WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016 Technical Preview. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/). Después de instalarlo, tendrá que reiniciar el equipo.
- **Nuevas características:**
  - Se ha agregado la compatibilidad con la actualización de WMF 4.0. Para obtener más información sobre la actualización de WMF 4.0, vea [este blog](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-4-0-update-now-available-for-windows-server-2012-windows-server-2008-r2-sp1-and-windows-7-sp1/).
  - Lógica de reintentos en los errores que se producen durante la instalación de la extensión DSC o durante la aplicación de una configuración de DSC posterior a la instalación de la extensión. Como parte de este cambio, la extensión volverá a intentar la instalación en caso de errores en una instalación anterior o volverá a establecer una configuración de DSC si se produjo un error con esta, durante un máximo de tres veces hasta alcanzar el estado de finalización (correcto o error) o si se recibe una solicitud nueva. Si se produce un error en la extensión debido a una entrada de usuario o a configuración de usuario no válidas, no se producirá el reintento. En este caso, será necesario volver a invocar la extensión con una solicitud y configuración de usuario correcta nuevas. Nota: La extensión DSC depende del agente de máquinas virtuales de Azure para los reintentos. El agente de máquinas virtuales de Azure invoca la extensión con la última solicitud con errores hasta que se consiga un estado correcto o con errores.

### <a name="version-216"></a>Versión 2.16

- **Fecha de publicación:** 21 de abril de 2016
- **Sistemas operativos admitidos:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Compatibilidad con WMF:** WMF 5.0 RTM, WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016 Technical Preview. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/). Después de instalarlo, tendrá que reiniciar el equipo.
- **Nuevas características:**
  - Se ha mejorado el control de errores y se han agregado otras correcciones de errores menores.
  - Se ha agregado una nueva propiedad de configuración de la extensión DSC. Se ha agregado "ForcePullAndApply" en AdvancedOptions para permitir que la extensión DSC establezca configuraciones de DSC cuando el modo de actualización está configurado en Extracción, es decir, el modo contrario al modo Inserción. Consulte [este blog](https://devblogs.microsoft.com/powershell/arm-dsc-extension-settings/) para obtener más información sobre la configuración de la extensión DSC.

### <a name="version-215"></a>Versión 2.15

- **Fecha de publicación:** 14 de marzo de 2016
- **Sistemas operativos admitidos:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Compatibilidad con WMF:** WMF 5.0 RTM, WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016 Technical Preview. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/). Después de instalarlo, tendrá que reiniciar el equipo.
- **Nuevas características:**
  - En la versión 2.14 de la extensión se incluyen cambios para instalar WMF RTM. Durante la actualización de la versión 2.13.2.0 a la 2.14.0.0 de la extensión, puede que observe que se producen errores en algunos cmdlets de DSC o en la configuración con el siguiente error: "No Instance found with given property values" (No se ha encontrado ninguna instancia con los valores de propiedad especificados). Para obtener más información, vea las [notas de la versión de DSC](/powershell/scripting/wmf/known-issues/known-issues-dsc). Las soluciones alternativas para estos problemas se han agregado en la versión 2.15.
  - Si ha instalado la versión 2.14 y se encuentra con alguno de los problemas anteriores, tendrá que realizar los pasos manualmente. En una sesión de PowerShell con privilegios elevados:
    - `Remove-Item -Path $env:SystemRoot\system32\Configuration\DSCEngineCache.mof`
    - `mofcomp $env:windir\system32\wbem\DscCoreConfProv.mof`

### <a name="version-214"></a>Versión 2.14

- **Fecha de publicación:** 25 de febrero de 2016
- **Sistemas operativos admitidos:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Compatibilidad con WMF:** WMF 5.0 RTM, WMF 4.0
- **Entorno:** Azure
- **Observaciones**: en esta versión se usa DSC tal y como se incluye en Windows Server 2016 Technical Preview. Para otros sistemas operativos Windows, instale [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/). Después de instalarlo, tendrá que reiniciar el equipo.
- **Nuevas características:**
  - Usa WMF RTM.
  - Habilita la recopilación de datos con el fin de mejorar la calidad de la extensión DSC. Para obtener más información, vea [el blog](https://devblogs.microsoft.com/powershell/azure-dsc-extension-data-collection-2/).
  - Proporciona un formato de configuración que se ha actualizado para la extensión en una plantilla de Resource Manager. Para obtener más información, vea [el blog](https://devblogs.microsoft.com/powershell/arm-dsc-extension-settings/).
  - Se han aplicado correcciones de errores y otras mejoras.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre DSC de PowerShell, vea el [Centro de documentación de PowerShell](/powershell/scripting/dsc/overview/overview).
- Examine la [plantilla de Resource Manager para la extensión DSC](../virtual-machines/extensions/dsc-template.md).
- Para más funcionalidad y recursos que pueda administrar con DSC de PowerShell, [examine la Galería de PowerShell](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0).
- Para detalles sobre cómo pasar parámetros confidenciales a configuraciones, consulte [Pasar credenciales al controlador de extensiones DSC de Azure](../virtual-machines/extensions/dsc-credentials.md).
