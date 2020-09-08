---
title: Conexión de Azure Information Protection a Azure Sentinel
description: Transmita la información de registro de  Azure Information Protection  a Azure Sentinel mediante a configuración del conector de datos de Azure Information Protection. 
services: sentinel
author: yelevin
manager: rkarlin
ms.assetid: bfa2eca4-abdc-49ce-b11a-0ee229770cdd
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: conceptual
ms.date: 09/24/2019
ms.author: yelevin
ms.openlocfilehash: 160912ef53b5c079d026c5d16230f49eadf66366
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88794445"
---
# <a name="connect-data-from-azure-information-protection"></a>Conectar datos de Azure Information Protection

> [!IMPORTANT]
> El conector de datos de Azure Information Protection en Azure Sentinel se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Puede transmitir la información de registro de [Azure Information Protection](https://azure.microsoft.com/services/information-protection/) a Azure Sentinel mediante la configuración del conector de datos de Azure Information Protection. Azure Information Protection le ayuda a controlar y proteger los datos confidenciales, tanto si están almacenados en la nube o de forma local.

Si los [informes centrales de Azure Information Protection](https://docs.microsoft.com/azure/information-protection/reports-aip) ya están configurados para que la información de registro de este servicio se almacene en el mismo área de trabajo de Log Analytics que ha seleccionado actualmente para Azure Sentinel, puede omitir la configuración de este conector de datos. La información de registro de Azure Information Protection ya está disponible para Azure Sentinel.

Sin embargo, si la información de registro de Azure Information Protection va a un área de trabajo de Log Analytics diferente de la que ha seleccionado actualmente para Azure Sentinel, realice una de las siguientes acciones:

- Cambie el área de trabajo seleccionada en Azure Sentinel.

- Cambie el área de trabajo de Azure Information Protection; para ello, configure este conector de datos.
    
    Si cambia el área de trabajo, los nuevos datos de informes de Azure Information Protection ahora se almacenarán en el área de trabajo que se usa para Azure Sentinel y los datos históricos no estarán disponibles para esta solución. Además, si el área de trabajo anterior está configurada para consultas personalizadas, alertas o API REST, se deben volver a configurar para el área de trabajo de Azure Sentinel si quiere seguir usándolas en Azure Information Protection. No es necesaria ninguna reconfiguración para los clientes y servicios que usan Azure Information Protection.

## <a name="prerequisites"></a>Requisitos previos

- Uno de los siguientes roles de administrador de Azure AD para su inquilino: 
    - Administrador de Azure Information Protection
    - Administrador de seguridad
    - Administrador de cumplimiento
    - Administrador de datos de cumplimiento
    - Administrador global
    
    > [!NOTE]
    > No puede usar el rol de administrador de Azure Information Protection si su inquilino está en la [plataforma unificada de etiquetado](/information-protection/faqs#how-can-i-determine-if-my-tenant-is-on-the-unified-labeling-platform).
    
    Estos roles de administrador solo son necesarios para configurar el conector de Azure Information Protection y no son necesarios si Azure Sentinel está conectado a Azure Information Protection.

- Permisos para leer y escribir en el área de trabajo de Log Analytics que se usa para Azure Sentinel y Azure Information Protection.

- Azure Information Protection se ha agregado a Azure Portal. Si necesita ayuda con este paso, consulte [Adición de Azure Information Protection a Azure Portal.](https://docs.microsoft.com/azure/information-protection/quickstart-viewpolicy#add-azure-information-protection-to-the-azure-portal)

## <a name="connect-to-azure-information-protection"></a>Conectar a Azure Information Protection

Siga las instrucciones que se indican a continuación si no ha configurado un área de trabajo de Log Analytics para Azure Information Protection o si necesita cambiar el área de trabajo que almacena la información de registro de Azure Information Protection.

1. En Azure Sentinel, seleccione **Data connectors** (Conectores de datos)  > **Azure Information Protection (versión preliminar)** .

2. Seleccione **Open connector page** (Abrir página del conector).

3. En **Configuración**, seleccione **Connect Azure Information Protection logs** (Registros de conexión de Azure Information Protection).

4. En la hoja **Configurar análisis (versión preliminar)** , seleccione el área de trabajo que usa actualmente para Azure Sentinel. Si selecciona un área de trabajo diferente, los datos de informes de Azure Information Protection no estarán disponibles para Azure Sentinel.

5. Después de seleccionar un área de trabajo, elija **Aceptar**. El campo **ESTADO** del conector cambia a **Conectado**.

6. Los datos de informes de Azure Information Protection se almacenan en la tabla **InformationProtectionLogs_CL** en el área de trabajo seleccionada. 
    
    Para usar el esquema correspondiente en Azure Monitor para estos datos de informes, busque **InformationProtectionEvents**. Para información sobre estas funciones de evento, consulte la sección [Referencia de esquema descriptivo para funciones de evento](https://docs.microsoft.com/azure/information-protection/reports-aip#friendly-schema-reference-for-event-functions) de la documentación de Azure Information Protection.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a conectar Azure Information Protection a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
