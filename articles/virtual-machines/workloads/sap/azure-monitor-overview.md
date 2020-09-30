---
title: Información general y arquitectura de Azure Monitor para soluciones de SAP | Microsoft Docs
description: En este artículo se ofrecen respuestas a las preguntas más frecuentes sobre Azure Monitor para soluciones de SAP.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 06/30/2020
ms.author: radeltch
ms.openlocfilehash: 8d97c713dfec574a8ef7f3f0dde4701ddea0e98b
ms.sourcegitcommit: 5d7f8c57eaae91f7d9cf1f4da059006521ed4f9f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89669020"
---
# <a name="azure-monitor-for-sap-solutions-preview"></a>Azure Monitor para soluciones de SAP (versión preliminar)

## <a name="overview"></a>Información general

Azure Monitor para soluciones de SAP es un producto de supervisión nativo de Azure para los clientes que ejecutan entornos de SAP en Azure. El producto funciona con [SAP en Azure Virtual Machines](./hana-get-started.md) y con [SAP en Azure (instancias grandes)](./hana-overview-architecture.md).
Con Azure Monitor para las soluciones de SAP, los clientes pueden recopilar datos de telemetría de la infraestructura y las bases de datos de Azure en una ubicación central y correlacionar visualmente dichos datos para una solución de problemas más rápida.

Azure Monitor para soluciones de SAP se ofrece a través de Azure Marketplace. Proporciona una experiencia de instalación sencilla e intuitiva y solo se requieren unos clics para implementar el recurso de Azure Monitor para soluciones de SAP (conocido como **recurso de supervisión de SAP**).

Los clientes pueden supervisar distintos componentes de un entorno de SAP, como Azure Virtual Machines, un clúster de alta disponibilidad, una base de datos de SAP HANA, etc., para lo que solo deben agregar el **proveedor** correspondiente al componente.

Infraestructura admitida:

- Máquina virtual de Azure
- Azure (instancias grandes)

Bases de datos admitidas:
- Base de datos de SAP HANA
- Microsoft SQL Server

Azure Monitor para soluciones de SAP aprovecha la eficacia de las capacidades de [Azure Monitor](../../../azure-monitor/overview.md), como Log Analytics y [Workbooks](../../../azure-monitor/platform/workbooks-overview.md), para proporcionar funciones de supervisión adicionales. Los clientes pueden crear [visualizaciones personalizadas](../../../azure-monitor/platform/workbooks-overview.md#getting-started) mediante la modificación de los libros predeterminados de Workbooks que proporciona Azure Monitor para soluciones de SAP, escribir [consultas personalizadas](../../../azure-monitor/log-query/get-started-portal.md) y crear [alertas personalizadas](../../../azure-monitor/learn/tutorial-response.md) mediante el uso del área de trabajo de Log Analytics de Azure, aprovechar el [período de retención flexible](../../../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period) y conectar datos de supervisión con el sistema de vales.

## <a name="what-data-does-azure-monitor-for-sap-solutions-collect"></a>¿Qué datos recopila Azure Monitor para soluciones de SAP?

La recopilación de datos en Azure Monitor para soluciones de SAP depende de los proveedores que hayan configurado los clientes. Durante la versión preliminar pública, se recopilan los datos siguientes.

Telemetría del clúster de Pacemaker de alta disponibilidad:
- Estado del nodo, del recurso y del dispositivo SBD
- Restricciones de ubicación de Pacemaker
- Votos de cuórum y estado del anillo
- [Otros](https://github.com/ClusterLabs/ha_cluster_exporter/blob/master/doc/metrics.md)

Telemetría de SAP HANA:
- Uso de CPU, memoria, disco y red
- Replicación del sistema HANA (HSR)
- Copia de seguridad de HANA
- Estado del host de HANA
- Roles del servidor de nombres e indexación

Telemetría de Microsoft SQL Server:
- Uso de CPU, memoria y disco
- Nombre del host, nombre de la instancia de SQL e identificador del sistema SAP
- Solicitudes por lotes, compilaciones y duración prevista de la página a lo largo del tiempo
- Las 10 instrucciones SQL más caras a lo largo del tiempo
- Las 12 tablas más grandes del sistema SAP
- Problemas detectados en los registros de errores de SQL Server
- Procesos de bloqueo y estadísticas de espera de SQL a lo largo del tiempo

## <a name="data-sharing-with-microsoft"></a>Uso compartido de datos con Microsoft

Azure Monitor para soluciones de SAP recopila metadatos del sistema para proporcionar una compatibilidad mejorada a los clientes de SAP en Azure. No se recopila información de identificación personal ni información de identificación del usuario final.
Los clientes pueden habilitar el uso compartido de datos con Microsoft en el momento de crear el recurso de Azure Monitor para soluciones de SAP si seleccionan *Compartir* en el menú desplegable.
Se recomienda encarecidamente a los clientes habilitar el uso compartido de datos, ya que suministra a los equipos de ingeniería y soporte técnico de Microsoft más información sobre el entorno del cliente y proporciona una asistencia mejorada a los clientes críticos de SAP en Azure.

## <a name="architecture-overview"></a>Información general sobre la arquitectura

En el diagrama siguiente se explica a grandes rasgos la manera en que Azure Monitor para soluciones de SAP recopila telemetría a partir de una base de datos de SAP HANA. La arquitectura no depende de si SAP HANA está implementado en Azure Virtual Machines o en Azure (instancias grandes).

![Arquitectura de Azure Monitor para soluciones de SAP](./media/azure-monitor-sap/azure-monitor-architecture.png)

Los componentes clave de la arquitectura son:
- Azure Portal: el punto de partida de los clientes. Los clientes pueden ir a Marketplace desde Azure Portal y descubrir Azure Monitor para soluciones de SAP.
- Recurso de Azure Monitor para soluciones de SAP: sitio de aterrizaje donde los clientes pueden ver la telemetría de supervisión.
- Grupo de recursos administrado: se implementa automáticamente como parte de la implementación del recurso de Azure Monitor para soluciones de SAP. Los recursos implementados en el grupo de recursos administrado ayudan a recopilar telemetría. Los recursos clave implementados y su finalidad son:
   - Azure Virtual Machines: también se conoce como *máquina virtual del recopilador*. Se trata de una máquina virtual Standard_B2ms. El propósito principal de esta máquina virtual consiste en hospedar la *carga de supervisión*. La carga de supervisión hace referencia a la lógica de recopilación de telemetría de los sistemas de origen y a la transferencia de los datos recopilados al marco de supervisión. En el diagrama anterior, la carga de supervisión contiene la lógica para conectarse a la base de datos de SAP HANA a través del puerto de SQL.
   - [Azure Key Vault](../../../key-vault/general/basic-concepts.md): este recurso se implementa para conservar de forma segura las credenciales de la base de datos de SAP HANA y para almacenar información sobre los [proveedores](./azure-monitor-providers.md).
   - Área de trabajo de Log Analytics: destino en el que residen los datos de telemetría.
      - La visualización se basa en la telemetría de Log Analytics mediante [Azure Workbooks](../../../azure-monitor/platform/workbooks-overview.md). Los clientes pueden personalizar la visualización. Además, pueden anclar la visualización de Workbooks u otra específica en el panel de Azure para aplicar la función de actualización automática con la granularidad mínima de 30 minutos.
      - Los clientes pueden usar el área de trabajo existente dentro de la misma suscripción como recurso de supervisión de SAP si eligen esta opción en el momento de la implementación.
      - Los clientes pueden usar el lenguaje de consulta Kusto (KQL) para ejecutar [consultas](../../../azure-monitor/log-query/log-query-overview.md) en las tablas sin procesar dentro del área de trabajo de Log Analytics. Consulte los *registros personalizados*.

> [!Note]
> Los clientes son responsables de las revisiones y el mantenimiento de la máquina virtual implementada en el grupo de recursos administrado.

> [!Tip]
> Los clientes pueden optar por usar un área de trabajo de Log Analytics existente para la recopilación de telemetría, si está implementada en la misma suscripción de Azure que el recurso de Azure Monitor para soluciones de SAP.

### <a name="architecture-highlights"></a>Aspectos destacados de la arquitectura

A continuación se indican los principales aspectos de la arquitectura:
 - **Instancias múltiples**: los clientes pueden crear la supervisión para varias instancias de un tipo de componente determinado (por ejemplo, la base de datos de HANA, el clúster de alta disponibilidad, Microsoft SQL Server) en varios SID de SAP dentro de una red virtual con un único recurso de Azure Monitor para soluciones de SAP.
 - **Proveedores múltiples**: en el diagrama de arquitectura anterior se muestra el proveedor de SAP HANA a modo de ejemplo. De forma similar, los clientes pueden configurar proveedores adicionales para los componentes correspondientes (por ejemplo, la base de datos de HANA, el clúster de alta disponibilidad, Microsoft SQL Server) para recopilar datos de esos componentes.
 - **Código abierto**: el código fuente de Azure Monitor para soluciones de SAP está disponible en [GitHub](https://github.com/Azure/AzureMonitorForSAPSolutions). Los clientes pueden consultar el código del proveedor y obtener más información sobre el producto, realizar contribuciones o compartir sus comentarios.
 - **Marco de consulta extensible**: las consultas SQL para recopilar datos de telemetría se escriben en [JSON](https://github.com/Azure/AzureMonitorForSAPSolutions/blob/master/sapmon/content/SapHana.json). Se pueden agregar fácilmente consultas SQL adicionales para recopilar más datos de telemetría. Los clientes pueden solicitar la incorporación de datos de telemetría específicos a Azure Monitor para soluciones de SAP. Para ello, deben dejar sus comentarios a través del vínculo que aparece al final de este documento o ponerse en contacto con el equipo de cuentas.

## <a name="pricing"></a>Precios
Azure Monitor para soluciones de SAP es un producto gratuito (sin cuota de licencias). Los clientes son responsables de pagar el costo de los componentes subyacentes en el grupo de recursos administrados.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre los proveedores y cree su primer recurso de Azure Monitor para soluciones de SAP.
 - Obtenga más información sobre los [proveedores](./azure-monitor-providers.md).
 - [Implementación de Azure Monitor para soluciones de SAP con Azure PowerShell](azure-monitor-sap-quickstart-powershell.md)
 - ¿Tiene alguna pregunta sobre Azure Monitor para soluciones de SAP? Consulte la sección de [preguntas más frecuentes](./azure-monitor-faq.md).
