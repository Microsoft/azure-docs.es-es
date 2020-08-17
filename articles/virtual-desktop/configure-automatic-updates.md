---
title: 'Configuración de Microsoft Endpoint Configuration Manager: Azure'
description: Cómo configurar Microsoft Endpoint Configuration Manager para implementar actualizaciones de software en la sesión múltiple de Windows 10 Enterprise en Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 06/12/2020
ms.author: helohr
ms.reviewer: v-cawood; clemr
manager: lizross
ms.openlocfilehash: bfc797e0c11a1804d27c6c97fea8cb2ba62dca7a
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88010130"
---
# <a name="configure-microsoft-endpoint-configuration-manager"></a>Configuración de Microsoft Endpoint Configuration Manager

En este artículo se explica cómo configurar Microsoft Endpoint Configuration Manager para aplicar automáticamente las actualizaciones a un host de Windows Virtual Desktop que ejecuta la sesión múltiple de Windows 10 Enterprise.

## <a name="prerequisites"></a>Requisitos previos

Para configurar esta opción, necesitará lo siguiente:

   - Asegúrese de que ha instalado Microsoft Endpoint Configuration Manager en las máquinas virtuales.
   - Asegúrese de que su versión de Microsoft Endpoint Configuration Manager se encuentre al menos en el nivel de rama 1906. Para obtener los mejores resultados, use el nivel de rama 1910 o superior.

## <a name="configure-the-software-update-point"></a>Configuración del punto de actualización de software

Para recibir actualizaciones para la sesión múltiple de Windows 10 Enterprise, debe habilitar Windows Server, versión 1903 y versiones posteriores como un producto en Microsoft Endpoint Configuration Manager. Esta opción de producto también se aplica si usa el servicio de actualización de Windows Server para implementar las actualizaciones en sus sistemas.

Para recibir las actualizaciones:

1. Abra Microsoft Endpoint Configuration Manager y seleccione **Sitios**.
2. Seleccione **Configurar componentes de sitio**.
3. Seleccione **Punto de actualización de software** en el menú desplegable.
4. Seleccione la pestaña **Products** (Productos).
5. Marque la casilla **Windows Server, versión 1903 y posteriores**.
6. Diríjase a **Biblioteca de software** > **Información general** > **Actualizaciones de software** > **Todas las actualizaciones de software** y seleccione **Sincronizar las actualizaciones de software**.
7. Compruebe el archivo wsyncmgr.log en **Archivos de programa** > **Microsoft Configuration Manager** > **Registros** para asegurarse de que se guardaron los cambios. La sincronización de las actualizaciones puede tardar unos minutos.

## <a name="create-a-query-based-collection"></a>Creación de una colección basada en consulta

Para crear una colección de máquinas virtuales de sesión múltiple de Windows 10 Enterprise, se puede usar una colección basada en consultas para identificar la SKU del sistema operativo específica.

Para crear una colección:

1. Seleccione **Activos y compatibilidad**.
2. Diríjase a **Información general** > **Colecciones de dispositivos** y haga clic con el botón derecho en **Colecciones de dispositivos** y seleccione **Crear colección de dispositivos** en el menú desplegable.
3. En la pestaña **General** del menú que se abre, escriba un nombre que describa la colección en el campo **Nombre**. En el campo **Comentario**, puede proporcionar información adicional que describa la colección. En **Limiting Collection** (Restricciones de colección), defina las máquinas que se van a incluir en la consulta de la colección.
4. En la pestaña **Reglas de pertenencia**, agregue una regla para la consulta; para ello, seleccione **Agregar regla** y, a continuación, **Regla de consulta**.
5. En **Query Rule Properties** (Propiedades de regla de consulta), escriba un nombre para la regla y, a continuación, defina los parámetros de la regla; para ello, seleccione **Edit Query Statement** (Editar instrucción de consulta).
6. Seleccione **Show Query Statement** (Mostrar instrucción de consulta).
7. En la instrucción, escriba la siguiente cadena:

    ```syntax
    select
    SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client
    from SMS_R_System inner join SMS_G_System_OPERATING_SYSTEM on
    SMS_G_System_OPERATING_SYSTEM.ResourceId = SMS_R_System.ResourceId where
    SMS_G_System_OPERATING_SYSTEM.OperatingSystemSKU = 175
    ```

8. Seleccione **Aceptar** para crear la colección.
9. Para comprobar si la colección se creó correctamente, diríjase a **Activos y compatibilidad** > **Información general** > **Colecciones de dispositivos**.
