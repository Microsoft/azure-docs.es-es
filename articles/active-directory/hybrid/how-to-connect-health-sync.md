---
title: Uso de Azure AD Connect Health con sincronización | Microsoft Docs
description: Esta es la página de Azure AD Connect Health donde se describe cómo supervisar la sincronización de Azure AD Connect.
services: active-directory
documentationcenter: ''
author: zhiweiwangmsft
manager: daveba
ms.assetid: 1dfbeaba-bda2-4f68-ac89-1dbfaf5b4015
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/18/2017
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: ef1e988bef3876fafdee4f7f6f109858e85d75d0
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89278741"
---
# <a name="monitor-azure-ad-connect-sync-with-azure-ad-connect-health"></a>Supervisión de Azure AD Connect sync con Azure AD Connect Health
La siguiente documentación es específica de la supervisión de sincronización de Azure AD Connect (Sync) con Azure AD Connect Health.  Para obtener información sobre la supervisión de AD FS con Azure AD Connect Health, consulte [Uso de Azure AD Connect Health con AD FS](how-to-connect-health-adfs.md). Para obtener información adicional sobre la supervisión de Active Directory Domain Services con Azure AD Connect Health, consulte [Using Azure AD Connect Health with AD DS](how-to-connect-health-adds.md)(Uso de Azure AD Connect Health con AD DS).

![Azure AD Connect Health para sincronización](./media/how-to-connect-health-sync/syncsnapshot.png)

## <a name="alerts-for-azure-ad-connect-health-for-sync"></a>Alertas de Azure AD Connect Health para sincronización
La sección Alertas de Azure AD Connect Health para sincronización proporciona la lista de alertas activas. Cada alerta incluye información pertinente, pasos de resolución y vínculos a documentación relacionada. Al seleccionar una alerta activa o una alerta resulta, verá una hoja nueva con información adicional, así como los pasos que puede seguir para resolver la alerta y vínculos a documentación adicional. También puede ver datos históricos sobre las alertas resueltas en el pasado.

Al seleccionar una alerta, recibirá información adicional, así como los pasos que puede seguir para resolver la alerta y vínculos a documentación adicional.

![Error de sincronización de Azure AD Connect](./media/how-to-connect-health-sync/alert.png)

### <a name="limited-evaluation-of-alerts"></a>Evaluación limitada de alertas
Si Azure AD Connect no usa la configuración predeterminada (por ejemplo, si el filtrado de atributos se cambia de la configuración predeterminada a una configuración personalizada), el agente de Azure AD Connect Health no cargará los eventos de error relacionados con Azure AD Connect.

Esto limita la evaluación de las alertas por parte del servicio. Verá un banner que indica esta condición en Azure Portal en su servicio.

![Azure AD Connect Health para sincronización](./media/how-to-connect-health-sync/banner.png)

Para cambiar esto, haga clic en "Configuración" y permita que el agente de Azure AD Connect Health cargue todos los registros de errores.

![Azure AD Connect Health para sincronización](./media/how-to-connect-health-sync/banner2.png)

## <a name="sync-insight"></a>Recomendación de sincronización
Con frecuencia, los administradores desean conocer el tiempo que se tarda en sincronizar cambios en Azure AD y la cantidad de cambios que se están implantando. Esta característica proporciona una manera fácil de visualizar esta información mediante los gráficos siguientes:   

* Latencia de operaciones de sincronización
* Tendencia de cambio de objeto

### <a name="sync-latency"></a>Latencia de sincronización
Esta característica proporciona una tendencia gráfica de la latencia de las operaciones de sincronización (importación, exportación, etc.) para los conectores.  Esto proporciona una forma rápida y fácil de entender no solo la latencia de las operaciones (más grande si se produce un gran conjunto de cambios), sino también una manera de detectar anomalías en la latencia que pueden requerir más investigación.

![Latencia de sincronización](./media/how-to-connect-health-sync/synclatency02.png)

De manera predeterminada, solo se muestra la latencia de la operación de exportación del conector de Azure AD.  Para ver más operaciones del conector o para ver las operaciones de otros conectores, haga clic con el botón derecho en el gráfico, seleccione Editar gráfico o haga clic en el botón "Editar gráfico de latencia" y elija la operación específica y los conectores.

### <a name="sync-object-changes"></a>Cambios en el objeto de sincronización
Esta característica proporciona una tendencia gráfica del número de cambios que se evalúan y exportan a Azure AD.  Hoy en día, es difícil intentar recopilar esta información de los registros de sincronización.  El gráfico le proporciona, no solo una manera más sencilla de supervisar el número de cambios que se producen en su entorno, sino también una vista de los errores que ocurren.

![Latencia de sincronización](./media/how-to-connect-health-sync/syncobjectchanges02.png)

## <a name="object-level-synchronization-error-report"></a>Informe de errores de sincronización de nivel de objeto
Esta característica proporciona un informe sobre los errores de sincronización que pueden producirse cuando se sincronizan datos de identidad entre Windows Server AD y Azure AD con Azure AD Connect.

* El informe contiene los errores registrados por el cliente de sincronización (Azure AD Connect versión 1.1.281.0 o superior).
* Incluye los errores producidos en la última operación de sincronización en el motor de sincronización ("Exportación" en Azure AD Connector).
* El agente de Azure AD Connect Health para la sincronización debe tener conectividad saliente a los puntos de conexión necesarios para que el informe incluya los datos más recientes.
* El informe se **actualiza cada 30 minutos** con los datos cargados por el agente de Azure AD Connect Health para la sincronización. Ofrece las siguientes funcionalidades principales:

  * Categorización de errores
  * Lista de objetos con error por categoría
  * Todos los datos sobre los errores en un solo lugar
  * Comparación en paralelo de objetos con error debido a un conflicto
  * Descarga del informe de errores como archivo CVS

### <a name="categorization-of-errors"></a>Categorización de errores
El informe clasifica los errores de sincronización existentes en las siguientes categorías:

| Category | Descripción |
| --- | --- |
| Atributo duplicado |Errores que se producen cuando Azure AD Connect intenta crear o actualizar objetos con valores duplicados de uno o más atributos en Azure AD que deben ser únicos en un inquilino como, por ejemplo, proxyAddresses o UserPrincipalName. |
| Error de coincidencia de datos |Errores que se producen cuando la coincidencia parcial no coincide con objetos, lo que da lugar a errores de sincronización. |
| Error de validación de datos |Errores debidos a datos no válidos, como caracteres no admitidos en atributos críticos como UserPrincipalName o errores de formato que no superan la validación antes de escribirse en Azure AD. |
| Cambio de dominio federado | Errores cuando las cuentas usan un dominio federado diferente. |
| Atributo grande |Errores que se producen cuando el recuento, el tamaño o la longitud de uno o varios atributos es superior al máximo permitido. |
| Otros |Todos los demás errores que no se ajustan a las categorías anteriores. A raíz de los comentarios recibidos, esta categoría se dividirá en varias subcategorías. |

![Resumen del informe de errores de sincronización](./media/how-to-connect-health-sync/errorreport01.png)
![Categorías del informe de errores de sincronización](./media/how-to-connect-health-sync/SyncErrorByTypes.PNG)

### <a name="list-of-objects-with-error-per-category"></a>Lista de objetos con error por categoría
Al desglosar cada categoría, se mostrará la lista de objetos que presentan errores en esa categoría.
![Lista de informe de errores de sincronización](./media/how-to-connect-health-sync/errorreport03.png)

### <a name="error-details"></a>Detalles del error
Los siguientes datos están disponibles en la vista detallada de cada error.

* Atributo en conflicto resaltado
* Identificadores para el *objeto de AD* implicado
* Identificadores para el *objeto de Azure AD* implicado (si procede)
* Descripción y resolución del error

![Detalles del informe de errores de sincronización](./media/how-to-connect-health-sync/duplicateAttributeSyncError.png)

### <a name="download-the-error-report-as-csv"></a>Descarga del informe de errores como archivo CVS
Si selecciona el botón "Exportar" puede descargar un archivo CSV con todos los detalles acerca de todos los errores.

### <a name="diagnose-and-remediate-sync-errors"></a>Diagnóstico y corrección de errores de sincronización 
En escenarios específicos de errores de sincronización de atributos duplicados relacionados con la actualización del delimitador de origen de usuario, puede corregir estos errores directamente desde el portal. Más información sobre el [diagnóstico y la corrección de errores de sincronización de atributos duplicados](how-to-connect-health-diagnose-sync-errors.md)

## <a name="related-links"></a>Vínculos relacionados
* [Solución de problemas de errores durante la sincronización](tshoot-connect-sync-errors.md)
* [Resistencia de atributos duplicados](how-to-connect-syncservice-duplicate-attribute-resiliency.md)
* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Instalación del agente de Azure AD Connect Health](how-to-connect-health-agent-install.md)
* [Operaciones de Azure AD Connect Health](how-to-connect-health-operations.md)
* [Uso de Azure AD Connect Health con AD FS](how-to-connect-health-adfs.md)
* [Uso de Azure AD Connect Health con AD DS](how-to-connect-health-adds.md)
* [Preguntas más frecuentes de Azure AD Connect Health](reference-connect-health-faq.md)
* [Historial de versiones de Azure AD Connect Health](reference-connect-health-version-history.md)