---
title: Cómo solicitar un aumento de la cuota de recursos de Azure Cosmos DB
description: Obtenga información sobre cómo solicitar un aumento de la cuota de recursos de Azure Cosmos DB. También aprenderá a habilitar una suscripción para acceder a una región.
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.topic: how-to
ms.date: 07/17/2020
ms.openlocfilehash: cd2bc3ec7e70049ee7e2c700731515a272e541dc
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095171"
---
# <a name="how-to-request-quota-increase-for-azure-cosmos-db-resources"></a>Cómo solicitar un aumento de la cuota de recursos de Azure Cosmos DB

Los recursos de Azure Cosmos DB tienen [cuotas/límites predeterminados](concepts-limits.md). Sin embargo, puede haber un caso en el que la carga de trabajo necesite más cuota que el valor predeterminado. En tal caso, debe ponerse en contacto con el equipo de Azure Cosmos DB para solicitar un aumento de la cuota. En este artículo se explica cómo solicitar un aumento de la cuota de recursos de Azure Cosmos DB. También aprenderá a habilitar una suscripción para acceder a una región.

## <a name="create-a-new-support-request"></a>Creación de una solicitud de soporte técnico

Para solicitar un aumento de la cuota, debe crear una nueva solicitud de soporte técnico con los detalles de la carga de trabajo. A continuación, el equipo de Azure Cosmos DB evaluará la solicitud y la aprobará. Use los pasos siguientes para crear una nueva solicitud de soporte técnico desde Azure Portal:

1. Inicie sesión en Azure Portal.

1. En el menú de la izquierda, seleccione **Ayuda y soporte técnico** y, a continuación, **Nueva solicitud de soporte técnico**.

1. Escriba la siguiente información en la pestaña **Aspectos básicos**:

   * En **Tipo de problema**, seleccione **Límites de servicio y suscripción (cuotas)** .
   * En **Suscripción**, seleccione la suscripción para la que quiere aumentar la cuota.
   * En **Tipo de cuota**, seleccione **Cosmos DB**.

   :::image type="content" source="./media/create-support-request-quota-increase/create-quota-increase-request.png" alt-text="Crear una nueva solicitud de soporte técnico de Cosmos DB para aumentar la cuota":::

1. En la pestaña **Detalles**, escriba los detalles correspondientes a la solicitud de cuota. La información proporcionada en esta pestaña se usarán para evaluar de forma más detallada el problema y ayudará al ingeniero de soporte técnico a solucionar el problema.

1. Rellene los siguientes detalles en este formulario:

   * **Descripción**: proporcione una breve descripción de la solicitud, como la carga de trabajo y el motivo por el que los valores predeterminados no son suficientes. En función del tipo de recurso para el que desea aumentar la cuota, es obligatorio proporcionar los detalles siguientes en el campo **Descripción**:

     **Regions requests** (Solicitudes de regiones). Si la solicitud es para agregar una región a la lista de permitidos, asegúrese de proporcionar los valores siguientes:

        * Nombre de la región
        * Id. de suscripción

     **Throughput limit requests** (Solicitudes de límite de rendimiento). Si la solicitud es para aumentar la cuota de rendimiento, asegúrese de proporcionar los valores siguientes:

        * Nombre de la base de datos
        * Id. de suscripción
        * New throughput limit (Nuevo límite de rendimiento)

     **Database account limit requests** (Solicitudes de límite de cuentas de la base de datos). Si la solicitud es para aumentar la cuota del número de cuentas de la base de datos de una suscripción, asegúrese de proporcionar los valores siguientes:

       * Id. de suscripción
       * New database account limit (Nuevo límite de cuentas de la base de datos)

   * **Carga de archivos**: cargue los archivos de diagnóstico o cualquier otro archivo que considere pertinente para la solicitud de soporte técnico. Para obtener más información sobre la guía de carga de archivos, consulte el artículo [Soporte técnico de Azure]( ../azure-portal/supportability/how-to-manage-azure-support-request.md#upload-files).

   * **Gravedad**: elija uno de los niveles de gravedad disponibles en función del impacto en el negocio.

   * **Método de contacto preferido**: las opciones de contacto disponibles son **Correo electrónico** y **Teléfono**.

1. Rellene los detalles restantes, como la disponibilidad, el idioma de soporte técnico, la información de contacto, el correo electrónico y el número de teléfono en el formulario.

1. Seleccione **Siguiente: Revisar y crear**. Valide la información proporcionada y seleccione **Crear** para crear una solicitud de soporte técnico.

En un plazo de 24 horas, el equipo de soporte técnico de Azure Cosmos DB evaluará su solicitud y se pondrá en contacto con usted.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Cuotas de servicio predeterminadas de Azure Cosmos DB](concepts-limits.md).
