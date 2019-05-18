---
title: Vista de los registros de Azure Monitor para contenedores en tiempo real | Microsoft Docs
description: Este artículo describe la vista de registros de contenedor (stdout y stderr) y eventos en tiempo real sin usar kubectl con Azure Monitor para contenedores.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/10/2019
ms.author: magoedte
ms.openlocfilehash: 376a7f3f83cc7fcf7490675d9c0aef1513862e8a
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65521736"
---
# <a name="how-to-view-logs-and-events-in-real-time-preview"></a>Cómo ver los registros y eventos en tiempo real (versión preliminar)
Supervisión de contenedores de Azure incluye una característica, que se encuentra actualmente en versión preliminar, que proporciona una vista en vivo en los registros de contenedor de Azure Kubernetes Service (AKS) (stdout y stderr) y los eventos sin tener que ejecutar los comandos de kubectl. Cuando se selecciona cualquiera de las opciones, aparece un nuevo panel debajo de la tabla de datos de rendimiento en el **nodos**, **controladores**, y **contenedores** vista. Muestra eventos generados por el motor de contenedor para ayudar a solucionar los problemas en tiempo real y registro en vivo. 

>[!NOTE]
>**Colaborador** se requiere acceso al recurso de clúster para esta característica funcione.
>

Los registros dinámicos admiten tres métodos diferentes para controlar el acceso a los registros:

1. AKS sin autorización RBAC de Kubernetes habilitada 
2. AKS habilitado con autorización de RBAC de Kubernetes
3. AKS habilitado con Azure Active Directory (AD) basado en SAML inicio de sesión único en 

## <a name="kubernetes-cluster-without-rbac-enabled"></a>Clúster de Kubernetes sin RBAC habilitado
 
Si tiene un clúster de Kubernetes que no está configurado con autorización de RBAC de Kubernetes ni integrado en el inicio de sesión único de Azure AD, no es necesario seguir estos pasos. Dado que la autorización de Kubernetes usa la api de kube, se requieren permisos de solo lectura.

## <a name="kubernetes-rbac-authorization"></a>Autorización de RBAC de Kubernetes
Si ha habilitado la autorización de RBAC de Kubernetes, deberá aplicar el enlace de rol de clúster. Los pasos de ejemplo siguientes muestran cómo configurar el enlace de rol de clúster desde esta plantilla de configuración de yaml. 

1. Copie el archivo yaml, péguelo y guárdelo como LogReaderRBAC.yaml.  

    ```
    apiVersion: rbac.authorization.k8s.io/v1 
    kind: ClusterRole 
    metadata: 
       name: containerHealth-log-reader 
    rules: 
       - apiGroups: [""] 
         resources: ["pods/log", "events"] 
         verbs: ["get", "list"]  
    --- 
    apiVersion: rbac.authorization.k8s.io/v1 
    kind: ClusterRoleBinding 
    metadata: 
       name: containerHealth-read-logs-global 
    roleRef: 
        kind: ClusterRole 
        name: containerHealth-log-reader 
        apiGroup: rbac.authorization.k8s.io 
    subjects: 
       - kind: User 
         name: clusterUser 
         apiGroup: rbac.authorization.k8s.io
    ```

2. Si se configuran por primera vez, se crea el enlace de regla de clúster, ejecute el comando siguiente: `kubectl create -f LogReaderRBAC.yaml`. Si habilitó anteriormente el soporte técnico para obtener una vista previa en registros activos antes de que se introdujeron en directo de los registros de eventos, para actualizar la configuración, ejecute el siguiente comando: `kubectl apply -f LiveLogRBAC.yml`. 

## <a name="configure-aks-with-azure-active-directory"></a>Configuración de AKS con Azure Active Directory
Es posible configurar AKS para que utilice Azure Active Directory (AD) para la autenticación de usuarios. Si va a configurar esto por primera vez, consulte [Integración de Azure Active Directory con Azure Kubernetes Service](../../aks/azure-ad-integration.md). Durante los pasos para crear la [aplicación cliente](../../aks/azure-ad-integration.md#create-client-application) y especificar el **URI de redirección**, deberá agregar otro URI a la lista `https://ininprodeusuxbase.microsoft.com/*`.  

>[!NOTE]
>La configuración de la autenticación con Azure Active Directory para el inicio de sesión único solo puede lograrse durante la implementación inicial de un nuevo clúster de AKS. No puede configurar el inicio de sesión único en un clúster de AKS ya implementado. Debe configurar la autenticación desde **registro de la aplicación (heredada)** opción en Azure AD con el fin de admitir el uso de un carácter comodín en el URI y mientras éste se agrega a la lista, regístrelo como un **nativo** app.
> 

## <a name="view-live-logs-and-events"></a>Eventos y registros de vista en vivo

Puede ver eventos de registro en tiempo real de medida que se generan por el motor de contenedor desde el **nodos**, **controladores**, y **contenedores** vista. En el panel Propiedades seleccione **ver datos activos (versión preliminar)** se presenta la opción y un panel debajo de la tabla de datos de rendimiento donde puede ver el registro y eventos en un flujo continuo. 

![Opción de nodo propiedades panel Ver registros activos](./media/container-insights-live-logs/node-properties-live-logs-01.png)  

Los mensajes de eventos y registros están limitados según el tipo de recurso está seleccionado en la vista.

| Vista | Tipo de recurso | Registro o evento | Datos presentados |
|------|---------------|--------------|----------------|
| Nodos | Nodo | Evento | Cuando se selecciona un nodo de eventos no se filtran y mostrar los eventos de todo el clúster Kubernetes. Título del panel muestra el nombre del clúster. |
| Nodos | Pod | Evento | Cuando se selecciona un pod se filtran los eventos a su espacio de nombres. Título del panel muestra el espacio de nombres de lo pod. | 
| Controladores | Pod | Evento | Cuando se selecciona un pod se filtran los eventos a su espacio de nombres. Título del panel muestra el espacio de nombres de lo pod. |
| Controladores | Controlador | Evento | Cuando se selecciona un controlador de eventos se filtran a su espacio de nombres. Título del panel muestra el espacio de nombres del controlador. |
| Nodos/controladores o contenedores | Contenedor | Registros | Título del panel muestra que el nombre del pod del contenedor se agrupa con. |

Si el clúster de AKS se configura con inicio de sesión único mediante AAD, deberá autenticarse en el primer uso durante esa sesión del explorador. Seleccione la cuenta y complete la autenticación con Azure.  

Después de autenticarse correctamente, el panel de registros dinámicos aparecerá en la sección inferior del panel central. Si el indicador de estado de captura muestra una marca de verificación verde, que se encuentra en el extremo derecho del panel, significa que puede recuperar los datos.
    
  ![Datos recuperados del panel de registros dinámicos](./media/container-insights-live-logs/live-logs-pane-01.png)  

En la barra de búsqueda, puede filtrar por palabra clave para resaltar texto en el registro o el evento y en la barra de búsqueda en el extremo derecho, muestra cuántos resultados coinciden con horizontalmente el filtro.   

  ![Ejemplo de filtro del panel de registros dinámicos](./media/container-insights-live-logs/live-logs-pane-filter-example-01.png)

Para suspender el desplazamiento automático y controlar el comportamiento del panel y permitirle desplazarse manualmente por los nuevos datos de lectura, haga clic en el **desplazamiento** opción. Para volver a habilitar desplazamiento automático, simplemente haga clic en el **desplazamiento** opción de nuevo. También puede pausar la recuperación de datos de registro o un evento, haga clic en el **pausar** opción y cuando esté listo para reanudar, simplemente haga clic en **reproducir**.  

![Vista activa de pausa en el panel de registros dinámicos](./media/container-insights-live-logs/live-logs-pane-pause-01.png)

Puede ir a registros de Azure Monitor para ver los registros de contenedor históricos seleccionando **ver registros de contenedor** en la lista desplegable **ver en análisis**.

## <a name="next-steps"></a>Pasos siguientes
- Para continuar aprendiendo a usar Azure Monitor y supervisar otros aspectos de su clúster de AKS, consulte [Comprender el rendimiento del clúster de AKS con Azure Monitor para contenedores](container-insights-analyze.md).
- Vista [ejemplos de consultas de registro](container-insights-log-search.md#search-logs-to-analyze-data) para ver consultas predefinidas y ejemplos para evaluar o personalizar para las alertas, visualizar o analizar los clústeres.
