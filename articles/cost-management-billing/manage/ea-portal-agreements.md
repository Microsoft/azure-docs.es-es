---
title: Acuerdos y enmiendas de Contrato Enterprise de Azure
description: En este artículo se explica cómo los acuerdos y enmiendas del Contrato Enterprise de Azure afectan al uso del portal del Contrato Enterprise de Azure.
author: bandersmsft
ms.author: banders
ms.date: 08/04/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.reviewer: boalcsva
ms.openlocfilehash: bc1749ba8246aa2181890c4c846f9159a0557aff
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87563490"
---
# <a name="azure-ea-agreements-and-amendments"></a>Acuerdos y enmiendas de Contrato Enterprise de Azure

En este artículo se describe cómo los acuerdos y modificaciones del Contrato Enterprise de Azure pueden afectar al acceso a los servicios de Azure, a su uso y al pago por ellos.

## <a name="enrollment-provisioning-status"></a>Estado de aprovisionamiento de inscripciones

La fecha de inicio de un nuevo prepago de Azure se define por la fecha en que el centro de operaciones regional lo ha procesado. Como los pedidos de prepago de Azure a través de Azure EA Portal se procesan en la zona horaria UTC, puede producirse algún retraso si el pedido de compra de prepago de Azure se procesó en otra región. La fecha de inicio de la cobertura en el pedido de compra muestra el inicio del prepago de Azure. La fecha de inicio de la cobertura es cuando el prepago de Azure aparece en Azure EA Portal.

## <a name="support-for-enterprise-customers"></a>Soporte técnico para clientes de Enterprise

 La [oferta del plan de soporte técnico del Contrato Enterprise](https://azure.microsoft.com/offers/enterprise-agreement-support/) de Azure está disponible para algunos clientes.

## <a name="enrollment-status"></a>Estado de la inscripción

Una inscripción tiene uno de los siguientes valores de estado. Cada valor determina cómo puede utilizar y tener acceso a una inscripción. El estado de la inscripción determina en qué fase se encuentra la inscripción. Indica si es necesario activar la inscripción antes de poder usarla. O bien, si el período inicial ha expirado y se le cobra el uso por encima del límite.

**Pending** (Pendiente): el administrador de inscripción debe iniciar sesión en el portal del Contrato Enterprise de Azure. Una vez iniciada la sesión, la inscripción cambia a estado **activo**.

**Active** (Activo): la inscripción es accesible y se puede usar. Puede crear cuentas y suscripciones en el portal del Contrato Enterprise de Azure. La inscripción permanece activa hasta la fecha de finalización del Contrato Enterprise.

**Indefinite Extended Term** (Período extendido indefinido): este estado tiene lugar después de que se alcanza la fecha de finalización del Contrato Enterprise. Antes de que la inscripción del Contrato Enterprise alcance la fecha de finalización, el administrador de inscripciones debe decidir:

- Renovar la inscripción mediante la adición de un nuevo prepago de Azure
- Transferir la inscripción existente a una nueva inscripción
- Migrar al programa de suscripción en línea de Microsoft (MOSP)
- Confirmar la deshabilitación de todos los servicios asociados con la inscripción

**Expired** (Expirado): la inscripción del Contrato Enterprise expira cuando se alcanza la fecha de finalización de este contrato. El cliente del Contrato Enterprise no participa en el período extendido y todos sus servicios están deshabilitados.

A partir del 1 de agosto de 2019, no se aceptan nuevos formularios de no participación para los clientes comerciales de Azure. En su lugar, todas las inscripciones entran en un período extendido indefinido. Si desea dejar de usar los servicios de Azure, cierre la suscripción en [Azure Portal](https://portal.azure.com). O bien, el asociado puede enviar una solicitud de finalización. No hay ningún cambio para los clientes con los tipos de contrato de gobierno.

**Transferred** (Transferido): el estado transferido se aplica a las inscripciones que tienen las cuentas y servicios asociados transferidos a una nueva inscripción. Las inscripciones no se transfieren automáticamente si se genera un nuevo número de inscripción durante la renovación. El número de inscripción anterior debe incluirse en la solicitud de renovación del cliente para una transferencia automática.

## <a name="partner-markup"></a>Incremento del asociado

En el portal del Contrato Enterprise de Azure, el incremento de precios de asociados ayuda a habilitar un mejor informe de costos para los clientes. El portal del Contrato Enterprise de Azure muestra el uso y los precios configurados por los asociados para sus clientes.

El incremento permite a los administradores de los asociados agregar un margen de beneficio porcentual a sus Contratos Enterprise indirectos. El incremento porcentual se aplica a toda la información de los servicios de Microsoft en Azure EA Portal como, por ejemplo, tasas de los medidores, prepago de Azure y pedidos. Cuando el asociado publica el incremento, el cliente ve los costos de Azure en Azure EA Portal. Por ejemplo, resumen de uso, listas de precios e informes de uso descargados.

A partir de septiembre de 2019, los asociados pueden aplicar el incremento en cualquier momento durante un período. No es necesario esperar hasta el siguiente aniversario del período para aplicar el incremento.

Microsoft no tendrá acceso ni usará el incremento proporcionado ni los precios asociados con fin alguno, a menos que el asociado del canal lo autorice explícitamente.

### <a name="how-the-calculation-works"></a>Realización del cálculo

El Proveedor de soluciones de licencia proporciona un único número de porcentaje en EA Portal.  Toda la información comercial del portal se elevará según este porcentaje proporcionado. Ejemplo:

- El cliente firma un Contrato Enterprise con un prepago de Azure de 100 000 USD.
- La tasa de medidor para el servicio A es de 10 USD por hora.
- El Proveedor de soluciones de licencia establece un porcentaje de incremento del 10 % en EA Portal.
- En el ejemplo siguiente se muestra cómo verá el cliente la información comercial:
    - Saldo monetario: 110 000 USD.
    - Tasa de medidor para el servicio A: 11 USD por hora.
    - Información de uso u hospedaje del servicio A cuando se ha usado durante 100 horas: 1100 USD.
    - Saldo monetario disponible para el cliente tras la deducción del consumo del servicio A: 108 900 USD.

### <a name="when-to-use-a-markup"></a>Cuándo se usa el incremento

Use esta característica si establece el mismo porcentaje de incremento en TODAS las transacciones comerciales del Contrato Enterprise; es decir, si incrementa la información de prepago de Azure, las tasas de los medidores, la información de pedidos, etc.

No use la característica de incremento en los siguientes casos:
- Usa diferentes tasas entre el prepago de Azure y el medidor.
- Usa diferentes tasas para distintos medidores.

Si usa tasas diferentes para distintos medidores, se recomienda desarrollar una solución personalizada basada en la clave de API, que puede proporcionar el cliente, para extraer datos de consumo y disponer de informes.

### <a name="other-important-information"></a>Información importante adicional

Esta característica está pensada para proporcionar una estimación del costo que tiene Azure para el cliente final. El Proveedor de soluciones de licencia es responsable de todas las transacciones financieras con el cliente bajo el Contrato Enterprise.

Asegúrese de revisar la información comercial —información de saldo monetario, lista de precios, etc.— antes de publicar los precios incrementados para el cliente final.

### <a name="how-to-add-a-price-markup"></a>Incorporación de un incremento de precio

**Paso uno: Agregar el incremento de precio**

1. En Enterprise Portal, haga clic en **Reports** (Informes) en el panel de navegación izquierdo.
1. En _Usage Summary_ (Resumen de uso), haga clic en el texto **Markup** (Incremento) azul.
1. Escriba el porcentaje de incremento (entre -100 y 100) y haga clic en **Preview** (Vista previa).


**Paso dos: Revisar y validar**

Revise el incremento de precio en _Usage Summary_ (Resumen de uso) para el período de prepago en la vista de cliente. El precio de Microsoft seguirá estando disponible en la vista de asociado. Las vistas se pueden alternar mediante el texto de alternancia "People" (Personas) de incremento del asociado en la parte superior derecha.

1. Revise los precios de la hoja de precios.
1. Se pueden realizar cambios antes de la publicación seleccionando **Edit** (Editar) en la pestaña _View Usage Summary > Customer View_ (Ver resumen de uso > Vista de cliente). 
  Tanto los precios de los servicios como los saldos de prepago se incrementarán con los mismos porcentajes. Si cuenta con porcentajes diferentes para el saldo monetario y para las tasas de medidor, o bien diferentes porcentajes para diferentes servicios, no use esta característica.

**Paso tres: Publicar**

Después de revisar y validar los precios, haga clic en **Publish** (Publicar).
  
Los precios con el incremento estarán disponibles para los administradores de empresa inmediatamente después de seleccionar la opción de publicación. Ya no se pueden realizar modificaciones en el incremento. Deberá deshabilitarlo y comenzar en el paso uno.

### <a name="which-enrollments-have-a-markup-enabled"></a>¿Qué inscripciones tienen el incremento habilitado?

Para comprobar si una inscripción tiene publicado un incremento, haga clic en **Manage** (Administrar) en el panel de navegación izquierdo y, después, en la pestaña **Enrollment** (Inscripción). Seleccione la casilla de inscripción que desea comprobar y consulte el estado de la inscripción en _Enrollment Detail_ (Detalles de inscripción). Se mostrará el estado actual de la característica de incremento para ese Contrato Enterprise; el estado puede ser Disabled (Deshabilitado), Preview (Vista previa) o Published (Publicado).

### <a name="how-can-the-customer-download-usage-estimates"></a>¿Cómo puede descargar el cliente estimaciones de uso?

Una vez que se publique el incremento del asociado, el cliente indirecto tendrá acceso a los archivos mensuales .csv de saldo y cargos, y a los archivos .csv de detalles de uso. Los archivos de detalles de uso incluirán la tasa de los recursos y el costo extendido.

### <a name="how-can-i-as-partner-apply-markup-to-existing-ea-customers-that-was-earlier-with-another-partner"></a>En tanto que asociado, ¿cómo puedo aplicar el incremento a los clientes de Contrato Enterprise existentes que estaban con otro asociado?
Los asociados pueden usar la característica de incremento (en Azure EA) después de que se procese un cambio de asociado de canal; no es necesario esperar al siguiente aniversario del período.


## <a name="resource-prepayment-and-requesting-quota-increases"></a>Prepago de recursos y solicitud de aumentos de la cuota

**El sistema aplica las siguientes cuotas predeterminadas por suscripción:**

| **Recurso** | **Cuota predeterminada** | **Comentarios** |
| --- | --- | --- |
| Instancias de proceso de Microsoft Azure | 20 instancias de proceso pequeñas simultáneas o su equivalente en los demás tamaños de instancia de proceso. | En la tabla siguiente se describe cómo calcular el número equivalente de instancias pequeñas:<ul><li> Extra pequeña: equivale a 1 instancia pequeña </li><li> Pequeña: equivale a 1 instancia pequeña </li><li> Media: equivale a 2 instancias pequeñas </li><li> Grande: equivale a 4 instancias pequeñas </li><li> Extra grande: equivale a 8 instancias pequeñas </li> </ul>|
| Máquinas virtuales con instancias de proceso de Microsoft Azure v2 | Contrato Enterprise: 350 núcleos | Máquinas virtuales de IaaS de disponibilidad general v2:<ul><li> Familia A0\_A7: 350 núcleos </li><li> Familia B\_A0\_A4: 350 núcleos </li><li> Familia A8\_A9: 350 núcleos </li><li> Familia DF: 350 núcleos</li><li> GF: 350 núcleos </li></ul>|
| Servicios hospedados de Microsoft Azure | 6 servicios hospedados | Este límite de servicios hospedados no se puede aumentar más allá de seis para una suscripción individual. Si necesita servicios hospedados adicionales, agregue más suscripciones. |
| Microsoft Azure Storage | 5 cuentas de almacenamiento, cada una de ellas con un tamaño máximo de 100 TB. | Puede aumentar el número de cuentas de almacenamiento a un máximo de 20 por suscripción. Si necesita más cuentas de almacenamiento, agregue más suscripciones. |
| SQL Azure | 149 bases de datos de cualquier tipo (es decir, Web Edition o Business Edition). |   |
| Control de acceso | 50 espacios de nombres por cuenta. 100 millones de transacciones de control de acceso al mes |   |
| Azure Service Bus | 50 espacios de nombres por cuenta. 40 conexiones de Service Bus | Los clientes que compren conexiones de Service Bus mediante los paquetes de conexión tendrán cuotas iguales al punto medio entre el paquete de conexión que adquirieron y el importe del siguiente paquete de conexión más alto. Los clientes que elijan un paquete de 500 tendrán una cuota de 750. |

## <a name="resource-prepayment"></a>Prepago de recursos

Microsoft le ofrecerá servicios hasta, al menos, el nivel de uso asociado incluido en el prepago mensual que haya adquirido (el Prepago de servicio), pero el resto de aumentos en los niveles de uso de los recursos de servicio (por ejemplo, aumento del número de instancias de proceso en ejecución o aumento de la cantidad de almacenamiento en uso) estarán sujetos a la disponibilidad de estos recursos de servicio.

Todas las cuotas descritas anteriormente no suponen un Prepago de servicio. Para determinar el número de instancias de proceso pequeñas simultáneas (o su equivalente) que Microsoft ofrecerá como parte del Prepago de servicio, se divide el número de horas de instancias de proceso pequeñas comprometidas para un mes entre el número de horas del mes más corto del año (es decir, febrero, o lo que es lo mismo, 672 horas).

## <a name="requesting-a-quota-increase"></a>Solicitud de un aumento de la cuota

Puede solicitar un aumento de la cuota en cualquier momento mediante el envío de una [solicitud en línea](https://g.microsoftonline.com/0WAEP00en/6). Para que se procese la solicitud, indique la siguiente información:

- La cuenta de Microsoft o bien la cuenta profesional o educativa asociada con el propietario de la cuenta de la suscripción. Se trata de la dirección de correo electrónico utilizada para iniciar sesión en Microsoft Azure Portal y administrar las suscripciones. Identifique también que esta cuenta está asociada a una inscripción de Contrato Enterprise.
- Los recursos y la cantidad para los que desea aumentar la cuota.
- El identificador de suscripción del Portal para desarrolladores de Azure asociado al servicio.
  - Para información sobre cómo obtener el identificador de la suscripción, [póngase en contacto con el servicio de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

## <a name="plan-skus"></a>SKU de plan

Las SKU de plan ofrecen la posibilidad de adquirir unidos un conjunto de servicios integrados con un descuento en la tasa. Las SKU de plan se diseñan para complementarse entre sí mediante otras ofertas y conjuntos integrados, para un mayor ahorro de costos.

Un ejemplo sería la suscripción a Operations Management Suite (OMS). OMS ofrece una manera sencilla de acceder a un conjunto completo de funcionalidades de administración basadas en la nube, incluidas el análisis, la configuración, la automatización, la seguridad, la copia de seguridad y la recuperación ante desastres. Las suscripciones a OMS incluyen derechos a los componentes de System Center, con el fin de proporcionar una solución completa para entornos de nube híbrida.

Los administradores de empresa pueden asignar propietarios de la cuenta para aprovisionar las SKU de plan adquiridas previamente en Enterprise Portal siguiendo estos pasos:

### <a name="view-the-price-sheet-to-check-included-quantity"></a>Visualización de la hoja de precios para comprobar la cantidad incluida

1. Inicie sesión como administrador de empresa.
1. En el panel de navegación izquierdo, haga clic en **Reports** (Informes).
1. Haga clic en la pestaña **Price Sheet** (Hoja de precios).
1. Haga clic en el icono "Download" (Descargar) en la esquina superior derecha.
1. Busque los números de pieza de la SKU de plan correspondiente, con el filtro en la columna "Included Quantity" (Cantidad incluida) y la selección de valores mayores que "0".

### <a name="existingnew-account-owners-to-create-new-subscriptions"></a>Propietarios de la cuenta nuevos o existentes para crear nuevas suscripciones

**Paso uno: Iniciar sesión en la cuenta**
1. En Azure EA Portal, seleccione la pestaña **Manage** (Administrar) y vaya a **Subscription** (Suscripción) en el menú superior.
1. Compruebe que ha iniciado sesión como propietario de la cuenta.
1. Haga clic en **+Add Subscription** (Agregar suscripción).
1. Haga clic en **Comprar**.

La primera vez que se agrega una suscripción a una cuenta, es preciso especificar la información de contacto. Al agregar suscripciones posteriores, la información de contacto se rellenará automáticamente.

La primera vez que agregue una suscripción a una cuenta, se le pedirá que acepte tanto el Contrato Microsoft Online Subscription como un plan de tasas. Estas secciones NO son aplicables a los clientes de Contrato Enterprise, pero actualmente son necesarias para aprovisionar la suscripción. La modificación de la inscripción del Contrato Enterprise de Microsoft Azure sustituye a los elementos anteriores, por lo que la relación contractual no cambiará. Active la casilla para indicar que acepta los términos.

**Paso dos: Actualizar el nombre de la suscripción**

Las suscripciones nuevas se agregan con el nombre de suscripción predeterminado "Microsoft Azure Enterprise". Es importante actualizar el nombre de la suscripción, para diferenciarla de las demás suscripciones de la inscripción Enterprise y asegurarse de que es reconocible en los informes de empresa.

Haga clic en **Subscriptions** (Suscripciones), haga clic en la suscripción que ha creado y, después haga clic en **Edit Subscription Details** (Editar detalles de suscripción).

Actualice el nombre de la suscripción y el administrador de servicios y, después, haga clic en la marca de verificación. El nombre de la suscripción aparecerá en los informes y será también el nombre del proyecto asociado a la suscripción en el portal de desarrollo.
Las nuevas suscripciones pueden tardar hasta 24 horas en propagarse a la lista de suscripciones.

Solo los propietarios de la cuenta pueden ver y administrar las suscripciones.

### <a name="troubleshooting"></a>Solución de problemas

**El propietario de la cuenta se muestra con el estado pendiente**

Cuando se agregan por primera vez nuevos propietarios de la cuenta a una inscripción, su estado siempre aparece como "Pending" (Pendiente). Al recibir el correo electrónico de bienvenida de activación, el propietario de la cuenta puede iniciar sesión para activarla. Esta activación actualizará el estado de su cuenta de "Pending" (Pendiente) a "Active" (Activo).

**Se cobran usos después de adquirir SKU de plan**

Este escenario se produce cuando el cliente ha implementado servicios con un número de inscripción incorrecto o ha seleccionado los servicios incorrectos.

Para comprobar si está realizando la implementación en la inscripción adecuada, puede consultar la información de las unidades incluidas a través de la hoja de precios. Inicie sesión como administrador de empresa, haga clic en **Reports** (Informes) en el panel de navegación izquierdo y seleccione la pestaña **Price Sheet** (Hoja de precios). Haga clic en el icono de descarga en la esquina superior derecha y busque los números de pieza de la SKU de plan correspondiente, con el filtro en la columna "Included Quantity" (Cantidad incluida) y la selección de valores mayores que "0".

Asegúrese de que el plan de OMS aparece en la hoja de precios bajo las unidades incluidas. Si no hay ninguna unidad incluida para el plan de OMS en la inscripción, es posible que este plan esté bajo otra inscripción. Póngase en contacto con el soporte técnico de Azure Enterprise Portal en [https://aka.ms/AzureEntSupport](https://aka.ms/AzureEntSupport).

Si las unidades incluidas de los servicios que aparecen en la hoja de precios no coinciden con lo que ha implementado, por ejemplo Operational Insights Premium Data Analyzed frente a Operational Insights Standard Data Analyzed, significa que puede haber implementado servicios que no están incluidos en el plan. En este caso, póngase en contacto con el soporte técnico de Azure Enterprise Portal en [https://aka.ms/AzureEntSupport](https://aka.ms/AzureEntSupport) para que podamos ayudarle.

**Servicios de SKU de plan aprovisionados en una inscripción incorrecta**

Si tiene varias inscripciones y ha implementado servicios con el número de inscripción incorrecto, que no cuenta con un plan de OMS, póngase en contacto con el servicio de soporte técnico de Azure Enterprise Portal en [https://aka.ms/AzureEntSupport](https://aka.ms/AzureEntSupport).

## <a name="next-steps"></a>Pasos siguientes

- Para empezar a usar el portal del Contrato Enterprise de Azure, consulte [Introducción al portal del Contrato Enterprise de Azure](ea-portal-get-started.md).
- Los administradores del portal del Contrato Enterprise de Azure deben leer el artículo acerca de la [administración del portal del Contrato Enterprise de Azure](ea-portal-administration.md) para obtener información sobre las tareas administrativas comunes.
