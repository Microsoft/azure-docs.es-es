---
title: Introducción a Cloudyn en Azure
description: Cloudyn es una solución de administración de costos en varias nubes que le permitirá usar Azure y otros recursos en la nube.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: overview
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.reviewer: benshy
ms.custom: seodec18
ROBOTS: NOINDEX
ms.openlocfilehash: 3acc13ca535808f14cb01d50e38f6bd4d12902fc
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684446"
---
# <a name="what-is-the-cloudyn-service"></a>¿Qué es el servicio Cloudyn?

Cloudyn, una subsidiaria de Microsoft, le permite realizar un seguimiento de los gastos y el uso de la nube para los recursos de Azure y otros proveedores de servicios en la nube, como AWS y Google. Los sencillos informes del panel proporcionan ayuda con la asignación de costos, los contracargos y la visibilidad de los gastos. Cloudyn le ayuda a optimizar los gastos en la nube mediante la identificación de los recursos que no está aprovechando para que pueda administrarlos y ajustarlos.

Para ver un vídeo de introducción, consulte [Introducción a Cloudyn](https://azure.microsoft.com/resources/videos/azure-cost-management-overview-and-demo/).
 
Azure Cost Management ofrece una funcionalidad similar a Cloudyn. Azure Cost Management es una solución de administración de costos de Azure nativa. Le ayuda a analizar costos, crear y administrar presupuestos, exportar datos y revisar y actuar sobre recomendaciones de optimización para ahorrar dinero. Para más información, consulte [Azure Cost Management](../cost-management-billing-overview.md).
 
[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

Vea el [vídeo de Azure Cost Management y Cloudyn](https://www.youtube.com/watch?v=15DzKPMBRxM) para obtener recomendaciones de cuándo debe usar Azure Cost Management o Cloudyn, según sus necesidades empresariales.
 
>[!VIDEO https://www.youtube.com/embed/15DzKPMBRxM]

## <a name="monitor-usage-and-spending"></a>Supervisión del uso y del gasto

Supervisar el uso y el gasto es crucial para las infraestructuras en la nube porque las organizaciones pagan por los recursos que consumen a lo largo del tiempo. Cuando el uso supera los umbrales del contrato, pueden producirse gastos inesperados rápidamente. Algunos factores importantes pueden dificultar la supervisión ad hoc. En primer lugar, proyectar los costos según el uso medio da por supuesto que el consumo es coherente durante un determinado período de facturación. En segundo lugar, cuando los costos están a punto de superar el presupuesto, es importante recibir notificaciones de forma proactiva para ajustar los gastos. Además, es posible que los proveedores de servicios en la nube no puedan ofrecer informes de comparación entre proyección de costos y umbrales, o entre períodos.

Los informes ayudan a supervisar los gastos para analizar y realizar un seguimiento del uso de la nube, los costos y las tendencias. Los informes de uso a lo largo del tiempo permiten detectar anomalías que difieren de las tendencias normales. Los informes de optimización también permiten detectar las deficiencias en la implementación de la nube. También puede detectar deficiencias en informes de análisis de costos.

## <a name="manage-costs"></a>Administrar costos

Los datos históricos pueden ayudar a administrar los costos al analizar el uso y los costos a lo largo del tiempo para identificar tendencias. Después, las tendencias se usan para hacer una previsión de los gastos futuros. Cloudyn también incluye informes útiles de costos previstos.

La asignación de costos administra los costos mediante el análisis basado en la directiva de etiquetado. Puede utilizar etiquetas en sus cuentas, recursos y entidades personalizadas para refinar la asignación de costos. El administrador de categorías organiza las etiquetas para proporcionarle gobernanza adicional. Así mismo, puede usar la asignación de costos para el proceso de visualización completa de los gastos y contracargo, y para mostrar el uso de recursos y los costos asociados para influir en los comportamientos de consumo o cobrar a los clientes del inquilino.

El control de acceso ayuda a administrar los costos al garantizar que los equipos y los usuarios solo tienen acceso a los datos de administración de costos que necesitan. Puede usar la estructura de la entidad, la administración de usuarios e informes programados con las listas de destinatarios para asignar acceso.

Las alertas ayudan a administrar los costos mediante la notificación automática cuando se producen gastos adicionales o inusuales. También pueden notificar automáticamente a otras partes interesadas sobre anomalías de gastos y riesgos de gastos adicionales. Existen varios informes que admiten alertas en función de los umbrales de costo y presupuesto. Sin embargo, las alertas no se admiten actualmente para las suscripciones o las cuentas de asociados de CSP.

## <a name="improve-efficiency"></a>Mejorar la eficiencia

Puede determinar el uso óptimo de las VM e identificar o eliminar VM inactivas, así como detectar discos sin conectar con Cloudyn. Con la información de los informes acerca de la optimización de tamaño y la falta de eficiencia, puede crear un plan para reducir o eliminar las VM inactivas. Sin embargo, los informes sobre optimización no se admiten actualmente para las suscripciones o las cuentas de asociados de CSP.

Si aprovisionó instancias reservadas de AWS, puede mejorar el uso de las instancias reservadas con informes de optimización que le permiten consultar recomendaciones de compra, modificar reservas sin usar y planear el aprovisionamiento.


## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con Cloudyn, el paso siguiente es registrar el entorno de nube y empezar a explorar los datos.

- [Registrarse con el programa de asociados de CSP y ver los datos de costo](quick-register-csp.md)
