---
title: Conexión de Configuration Manager con Azure Monitor | Microsoft Docs
description: En este artículo se muestran los pasos para conectar Configuration Manager con un área de trabajo en Azure Monitor y empezar a analizar datos.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/28/2019
ms.openlocfilehash: 6d6431aa26637e4b956d5c334a2862f689f845bf
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319327"
---
# <a name="connect-configuration-manager-to-azure-monitor"></a>Conexión de Configuration Manager con Azure Monitor
Puede conectar el entorno de Microsoft Endpoint Configuration Manager con Azure Monitor para sincronizar los datos de recopilación del dispositivo y hacer referencia a estas recopilaciones en Azure Monitor y Azure Automation.  

## <a name="prerequisites"></a>Prerrequisitos

Azure Monitor es compatible con la rama actual de Configuration Manager, versión 1606 y posteriores.

>[!NOTE]
>La característica para conectar Configuration Manager con un área de trabajo de Log Analytics es opcional y no está habilitada de forma predeterminada. Para poder usarla, debe habilitarla primero. Para más información, consulte [Habilitar características opcionales de las actualizaciones](/configmgr/core/servers/manage/install-in-console-updates#bkmk_options).

## <a name="configuration-overview"></a>Información general sobre la configuración

Los pasos siguientes son un resumen de los pasos necesarios para configurar la integración de Configuration Manager con Azure Monitor.  

1. En Azure Active Directory, registre Configuration Manager como una aplicación web o una aplicación de API web y asegúrese de que tiene el identificador de cliente y la clave secreta de cliente del registro de Azure Active Directory. Consulte [Uso del portal para crear una aplicación de Active Directory y una entidad de servicio con acceso a los recursos](../../active-directory/develop/howto-create-service-principal-portal.md) para obtener información detallada sobre cómo realizar este paso.

2. En Azure Portal, [conceda a Configuration Manager (la aplicación web registrada) permiso para acceder a Azure Monitor](#grant-configuration-manager-with-permissions-to-log-analytics).

3. En Configuration Manager, agregue una conexión utilizando el asistente para **servicios de Azure**.

4. [Descargue e instale el agente de Log Analytics para Windows](#download-and-install-the-agent) en el equipo en el que se ejecuta el rol del sistema del sitio de punto de conexión del servicio Configuration Manager. El agente envía datos de Configuration Manager al área de trabajo de Log Analytics en Azure Monitor.

5. En Azure Monitor, [importe recopilaciones de Configuration Manager](#import-collections) como grupos de equipos.

6. En Azure Monitor, visualice datos de Configuration Manager como [grupos de equipos](computer-groups.md).

## <a name="grant-configuration-manager-with-permissions-to-log-analytics"></a>Concesión de permisos a Configuration Manager para acceder a Log Analytics

En el procedimiento siguiente, concede el rol *Colaborador* del área de trabajo de Log Analytics a la aplicación de AD y a la entidad de servicio que creó anteriormente para Configuration Manager. Si no dispone de un área de trabajo, consulte [Creación de un área de trabajo en Azure Monitor](../learn/quick-create-workspace.md) antes de continuar. Esto permite a Configuration Manager autenticarse y conectarse a su área de trabajo de Log Analytics.  

> [!NOTE]
> Debe especificar permisos en el área de trabajo de Log Analytics para Configuration Manager. De lo contrario, recibirá un mensaje de error cuando utilice el Asistente para configuración de Configuration Manager.
>

1. En Azure Portal, haga clic en **Todos los servicios**, en la esquina superior izquierda. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Log Analytics**.

2. En la lista de áreas de trabajo de Log Analytics, seleccione el área de trabajo que desea modificar.

3. En el panel izquierdo, seleccione **Access Control (IAM)** .

4. En la página de Access Control (IAM), haga clic en **Agregar asignación de roles** y aparecerá el panel **Agregar asignación de roles**.

5. En el panel **Agregar asignación de roles**, en la lista desplegable **Rol**, seleccione el rol **Colaborador**.  

6. En la lista desplegable **Asignar acceso a**, seleccione la aplicación de Configuration Manager que creó en AD anteriormente y, a continuación, haga clic en **Aceptar**.  

## <a name="download-and-install-the-agent"></a>Descarga e instalación del agente

Consulte el artículo [Conexión de equipos Windows a Azure Monitor en Azure](agent-windows.md) si desea conocer los métodos disponibles para instalar el agente de Log Analytics en el equipo que hospeda el rol del sistema del sitio de punto de conexión del servicio Configuration Manager.  

## <a name="connect-configuration-manager-to-log-analytics-workspace"></a>Conexión de Configuration Manager con el área de trabajo de Log Analytics

>[!NOTE]
> Para agregar una conexión a Log Analytics, el entorno de Configuration Manager debe tener un [punto de conexión de servicio](/configmgr/core/servers/deploy/configure/about-the-service-connection-point) configurado en el modo en línea.

> [!NOTE]
> Debe conectar el sitio de nivel superior de la jerarquía a Azure Monitor. Si conecta un sitio principal independiente a Azure Monitor y, después, agrega un sitio de administración central a su entorno, tendrá que eliminar y volver a crear la conexión dentro de la nueva jerarquía.

1. En el área de trabajo **Administración** de Configuration Manager, seleccione **Cloud Services** y **Servicios de Azure**. 

2. Haga clic con el botón derecho en **Servicios de Azure** y seleccione **Configurar servicios de Azure**. Aparecerá la página **Configurar servicios de Azure**. 
   
3. En la pantalla **General**, confirme que ha realizado las siguientes acciones y que tiene los detalles de cada elemento; después, seleccione **Siguiente**.

4. En la página Servicios de Azure del Asistente para servicios de Azure:

    1. Especifique un **nombre** para el objeto de Configuration Manager.
    2. Especifique una **descripción** opcional que le ayude a identificar el servicio.
    3. Seleccione el **conector de OMS** de los servicios de Azure.

    >[!NOTE]
    >OMS ahora se conoce como Log Analytics, que es una característica de Azure Monitor.

5. Seleccione **Siguiente** para ir a la página de propiedades de la aplicación de Azure del Asistente para servicios de Azure.

6. En la página **Aplicación** del Asistente para servicios de Azure, seleccione primero el entorno de Azure en la lista y haga clic en **Importar**.

7. En la página **Importar aplicaciones**, especifique lo siguiente:

    1. Especifique el **nombre de inquilino de Azure AD** de la aplicación.

    2. Especifique el inquilino de Azure AD en **Id. de inquilino de Azure AD**. Encontrará esta información en la página **Propiedades** de Azure Active Directory. 

    3. En **Nombre de la aplicación**, especifique el nombre de la aplicación.

    4. En **Id. de cliente**, especifique el identificador de la aplicación de Azure AD que creó anteriormente.

    5. En **Clave secreta**, especifique la clave secreta del cliente de la aplicación de Azure AD que creó.

    6. En **Expiración de la clave secreta**, especifique la fecha de vencimiento de la clave.

    7. En **URI de id. de aplicación**, especifique el URI del identificador de la aplicación de Azure AD que creó anteriormente.

    8. Seleccione **Comprobar** y en los resultados de la derecha debería aparecer **Verificación realizada correctamente**.

8. En la página **Configuración**, revise los datos para comprobar que los campos **Suscripciones de Azure**, **Grupo de recursos de Azure** y **Operations Management Suite workspace** (Área de trabajo de Operations Management Suite) tengan valores que indiquen que la aplicación de Azure AD tiene permisos suficientes en el grupo de recursos. Si los campos están vacíos, significa que la aplicación no tiene los derechos necesarios. Seleccione los dispositivos que desea recopilar y reenviar al área de trabajo. A continuación, seleccione **Agregar**.

9. Revise las opciones de la página **Confirmar la configuración** y seleccione **Siguiente** para empezar a crear y configurar la conexión.

10. Una vez finalizada la configuración, aparecerá la página **Completion (Finalización)** . Seleccione **Cerrar**. 

Después de haber vinculado Configuration Manager a Azure Monitor, puede agregar o quitar recopilaciones y ver las propiedades de la conexión.

## <a name="update-log-analytics-workspace-connection-properties"></a>Actualización de las propiedades de conexión del área de trabajo de Log Analytics

Si expira o se pierde la contraseña o la clave secreta de cliente, tendrá que actualizar manualmente las propiedades de conexión de Log Analytics.

1. En el área de trabajo **Administración** de Configuration Manager, seleccione **Cloud Services** y después **Conector de OMS** para abrir la página **Propiedades de Conexión a OMS**.
2. En esta página, haga clic en la pestaña **Azure Active Directory** para ver su **inquilino**, **identificador de cliente** y **expiración de clave secreta de cliente**. **Compruebe** si ha expirado la **clave secreta de cliente**.

## <a name="import-collections"></a>Importación de recopilaciones

Después de agregar una conexión de Log Analytics en Configuration Manager y de instalar el agente en el equipo que ejecuta el rol de sistema de sitio del punto de conexión de servicio de Configuration Manager, el paso siguiente es importar las recopilaciones de Configuration Manager en Azure Monitor como grupos de equipos.

Una vez completada la configuración inicial para importar las recopilaciones de dispositivos de la jerarquía, la información de la recopilación se recupera cada tres horas para mantener actualizada la pertenencia. Puede deshabilitar esta opción en cualquier momento.

1. En Azure Portal, haga clic en **Todos los servicios**, en la esquina superior izquierda. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Áreas de trabajo de Log Analytics**.
2. En la lista de áreas de trabajo de Log Analytics, seleccione el área de trabajo en la que se ha registrado Configuration Manager.  
3. Seleccione **Configuración avanzada**.
4. Seleccione **Grupos de equipos** y, a continuación, **SCCM**.  
5. Seleccione **Importar pertenencias de la recopilación de Configuration Manager** y, después, haga clic en **Guardar**.  
   
    ![Grupos de equipos: pestaña SCCM](./media/collect-sccm/sccm-computer-groups01.png)

## <a name="view-data-from-configuration-manager"></a>Visualización de datos de Configuration Manager

Después de agregar una conexión de Log Analytics en Configuration Manager y de instalar el agente en el equipo que ejecuta el rol de sistema de sitio del punto de conexión de servicio de Configuration Manager, se envían datos desde el agente al área de trabajo de Log Analytics en Azure Monitor. En Azure Monitor, las recopilaciones de Configuration Manager aparecen como [grupos de equipos](./computer-groups.md). Puede ver los grupos de la página **Configuration Manager** en **Configuración\Grupos de equipos**.

Después de importar las recopilaciones, puede ver cuántos equipos con pertenencias a las recopilaciones se han detectado. También puede ver el número de recopilaciones que se han importado.

![Grupos de equipos: pestaña SCCM](./media/collect-sccm/sccm-computer-groups02.png)

Al hacer clic en cualquiera de ellos, se abre el editor de consultas de registro con todos los grupos importados o todos los equipos que pertenecen a cada grupo. Con [Búsqueda de registros](../log-query/log-query-overview.md), puede analizar con más profundidad los datos de pertenencia de las colecciones.

## <a name="next-steps"></a>Pasos siguientes

Use [Búsqueda de registros](../log-query/log-query-overview.md) para ver información detallada sobre los datos de Configuration Manager.

