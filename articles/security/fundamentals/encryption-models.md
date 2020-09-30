---
title: Modelos de cifrado de datos en Microsoft Azure
description: Este artículo proporciona información general sobre los modelos de cifrado de datos en Microsoft Azure.
services: security
documentationcenter: na
author: msmbaldwin
manager: rkarlin
ms.assetid: 9dcb190e-e534-4787-bf82-8ce73bf47dba
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/13/2020
ms.author: mbaldwin
ms.openlocfilehash: 51d7cb32ef2d1d7750a0bb53cfa3ccc111bd1302
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89143740"
---
# <a name="data-encryption-models"></a>Modelos de cifrado de datos

La comprensión de los distintos modelos de cifrado y sus ventajas y desventajas es fundamental para entender cómo los distintos proveedores de recursos en Azure implementan el cifrado en reposo. Estas definiciones se comparten entre todos los proveedores de recursos en Azure para asegurar la taxonomía y el idioma común.

Hay tres escenarios para el cifrado del lado servidor:

- Cifrado del lado servidor mediante claves administradas del servicio
  - Los proveedores de recursos de Azure realizan las operaciones de cifrado y descifrado
  - Microsoft administra las claves
  - Funcionalidad de nube completa

- Cifrado del lado servidor mediante claves administradas por el cliente en Azure Key Vault
  - Los proveedores de recursos de Azure realizan las operaciones de cifrado y descifrado
  - El cliente controla las claves mediante Azure Key Vault
  - Funcionalidad de nube completa

- Cifrado del lado servidor mediante claves administradas por el cliente en el hardware controlado por el cliente
  - Los proveedores de recursos de Azure realizan las operaciones de cifrado y descifrado
  - Claves de controles de cliente en el hardware controlado por el cliente
  - Funcionalidad de nube completa

Los modelos de cifrado del lado servidor hacen referencia al cifrado que se realiza mediante el servicio de Azure. En este modelo, el proveedor de recursos realiza las operaciones de cifrado y descifrado. Por ejemplo, Azure Storage puede recibir datos en las operaciones de texto sin formato y llevará a cabo el cifrado y descifrado internamente. El proveedor de recursos podría utilizar claves de cifrado que están administradas por Microsoft o por el cliente en función de la configuración proporcionada.

![Server](./media/encryption-models/azure-security-encryption-atrest-fig3.png)

Cada uno de los modelos de cifrado en reposo del lado servidor implica características distintivas de administración de claves. Esto incluye dónde y cómo se crean y almacenan las claves de cifrado, así como los modelos de acceso y los procedimientos de rotación de claves.  

Para el cifrado del lado cliente, tenga en cuenta lo siguiente:

- Los servicios de Azure no pueden ver los datos descifrados
- Los clientes administran y almacenan las claves en ubicaciones locales (o en otras ubicaciones seguras). Las claves no están disponibles para los servicios de Azure
- Funcionalidad de nube reducida

Los modelos de cifrado admitidos en Azure se dividen en dos grupos principales: "Cifrado del cliente" y "Cifrado del servidor" como se mencionó anteriormente. Independientemente del modelo de cifrado en reposo utilizado, los servicios de Azure siempre recomiendan el uso de un transporte seguro como TLS o HTTPS. Por lo tanto, el cifrado de transporte debe tratarse con el protocolo de transporte y no debe ser un factor importante para determinar qué modelo de cifrado en reposo se utilizará.

## <a name="client-encryption-model"></a>Modelo de cifrado del cliente

El modelo de cifrado del cliente hace referencia al cifrado que se realiza fuera del proveedor de recursos o Azure mediante el servicio o la aplicación que realiza la llamada. El cifrado puede realizarse mediante la aplicación de servicio de Azure o por una aplicación que se ejecuta en el centro de datos del cliente. En cualquier caso, cuando se saca provecho de este modelo de cifrado, el proveedor de recursos de Azure recibe un blob cifrado de datos sin la capacidad de descifrar los datos de ninguna forma ni tener acceso a las claves de cifrado. En este modelo, la administración de claves se realiza mediante el servicio o aplicación que realiza la llamada y es opaca para el servicio de Azure.

![Remoto](./media/encryption-models/azure-security-encryption-atrest-fig2.png)

## <a name="server-side-encryption-using-service-managed-keys"></a>Cifrado del lado servidor mediante claves administradas del servicio

Para muchos clientes, el requisito esencial es asegurarse de que los datos se cifran siempre que estén en reposo. El cifrado del lado servidor mediante las claves administradas del servicio habilita este modelo al permitir a los clientes marcar el recurso específico (cuenta de almacenamiento, SQL Database, etc.) para el cifrado y dejar todos los aspectos de la administración de claves, como la emisión de claves, la rotación y la copia de seguridad a Microsoft. La mayoría de los servicios de Azure que admiten cifrado en reposo normalmente admiten este modelo de descarga de la administración de las claves de cifrado de Azure. El proveedor de recursos de Azure crea las claves, las coloca en un almacenamiento seguro y las recupera cuando es necesario. Esto significa que el servicio tiene acceso completo a las claves y el servicio tiene control total sobre la administración del ciclo de vida de las credenciales.

![administrado](./media/encryption-models/azure-security-encryption-atrest-fig4.png)

Por lo tanto, el cifrado del lado servidor mediante las claves administradas del servicio satisface rápidamente la necesidad de que tengan el cifrado en reposo con poca sobrecarga al cliente. Cuando esté disponible, un cliente abrirá con normalidad Azure Portal para la suscripción de destino y el proveedor de recursos y comprobará un cuadro que indica si desearía que los datos se cifraran. El cifrado del lado servidor de algunas instancias de Resource Manager con las claves administradas del servicio se encuentra activado de forma predeterminada.

El cifrado del lado servidor con las claves de Microsoft administradas implica que el servicio tiene acceso completo para almacenar y administrar las claves. Aunque algunos clientes podrían desear administrar las claves porque creen que pueden conseguir mayor seguridad, se deben tener en cuenta los costos y riesgos asociados a una solución de almacenamiento de claves personalizadas al evaluar este modelo. En muchos casos, una organización podría determinar que los riesgos o restricciones de recursos de una solución local pueden ser mayores que el riesgo de administración en la nube de las claves de cifrado en reposo.  Sin embargo, este modelo podría no ser suficiente para las organizaciones que tienen requisitos para controlar la creación o el ciclo de vida de las claves de cifrado o tener personal diferente para administrar las claves de cifrado de un servicio al que administra el servicio (es decir, la segregación de la administración de claves de todo el modelo de administración para el servicio).

### <a name="key-access"></a>Acceso a la clave

Cuando se usa el cifrado del lado servidor con las claves administradas del servicio, el servicio administra la creación de claves, el almacenamiento y el acceso al servicio. Normalmente, los proveedores fundamentales de recursos de Azure almacenarán las claves de cifrado de datos en un almacén que se encuentra cerca de los datos y está rápidamente disponible y accesible mientras las claves de cifrado de clave se almacenan en un almacén interno seguro.

**Ventajas**

- Instalación simple
- Microsoft administra la rotación de claves, la copia de seguridad y la redundancia
- El cliente no tiene el costo asociado con la implementación o el riesgo de un esquema personalizado de administración de claves.

**Desventajas**

- No hay control al cliente sobre las claves de cifrado (revocación, ciclo de vida, especificación de clave, etc.)
- La administración de la clave no se puede separar del modelo de administración global para el servicio

## <a name="server-side-encryption-using-customer-managed-keys-in-azure-key-vault"></a>Cifrado del lado servidor mediante claves administradas por el cliente en Azure Key Vault

Para escenarios donde el requisito es cifrar los datos en reposo y controlar los clientes de las claves de cifrado, los clientes pueden usar el cifrado de lado servidor mediante las claves almacenadas por el cliente en Key Vault. Algunos servicios solo pueden almacenar la clave de cifrado de clave de raíz en Azure Key Vault y almacenar la clave de cifrado de datos cifrada en una ubicación interna cercana a los datos. En este escenario, los clientes pueden aportar sus propias claves a Key Vault (BYOK: aportar su propia clave), o generar nuevas y usarlas para cifrar los recursos deseados. Mientras que el proveedor de recursos realiza las operaciones de cifrado y descifrado, usa la clave de cifrado de claves configurada como clave raíz para todas las operaciones de cifrado.

La pérdida de claves de cifrado de claves significa también la pérdida de los datos. Por esta razón, no se deben eliminar las claves. Se debe realizar una copia de seguridad de las claves cada vez que se creen o giren. La [eliminación temporal](https://docs.microsoft.com/azure/key-vault/key-vault-ovw-soft-delete) debe estar habilitada en cualquier almacén que almacene claves de cifrado de claves. En lugar de eliminar una clave, establezca la opción Habilitado en "false" o defina la fecha de expiración.

### <a name="key-access"></a>Acceso a la clave

El modelo de cifrado del lado servidor con claves administradas del cliente en Azure Key Vault implica el servicio de acceso a las claves para cifrar y descifrar según sea necesario. Las claves del cifrado en reposo son accesibles para un servicio a través de una directiva de control de acceso. Esta directiva concede el acceso de identidad de servicio para recibir la clave. Un servicio de Azure que se ejecuta en nombre de una suscripción asociada puede configurarse con una identidad dentro de esa suscripción. El servicio puede realizar la autenticación de Azure Active Directory y recibir un token de autenticación que se identifica como el servicio que actúa en nombre de la suscripción. A continuación, se puede presentar ese token al Key Vault para obtener una clave a la que se le haya dado acceso.

Para las operaciones con claves de cifrado, una identidad de servicio puede tener acceso a cualquiera de las siguientes operaciones: descifrar, cifrar, unwrapKey, wrapKey, comprobar, iniciar sesión, obtener, enumerar, actualizar, crear, importar, eliminar, backup y restaurar.

Para obtener una clave para usar al cifrar o descifrar datos en reposo, la identidad de servicio con la que se ejecutará la instancia de Resource Manager debe tener UnwrapKey (para obtener la clave de descifrado) y WrapKey (para insertar una clave en el almacén de claves al crear una nueva clave).

>[!NOTE]
>Para obtener más detalles sobre la autorización del Key Vault, vea la protección de la página del almacén de claves en la [documentación de Azure Key Vault](../../key-vault/general/secure-your-key-vault.md).

**Ventajas**

- Control total sobre las claves usadas: las claves de cifrado se administran en la instancia de Key Vault del cliente bajo el control del cliente.
- Capacidad de cifrar varios servicios en un patrón
- Puede separar la administración de la clave del modelo de administración global para el servicio
- Puede definir el servicio y la ubicación de la clave en regiones

**Desventajas**

- El cliente tiene responsabilidad total para la administración de acceso a las claves
- El cliente tiene responsabilidad total para la administración del ciclo de vida de las claves
- Sobrecarga de configuración e instalación adicional

## <a name="server-side-encryption-using-customer-managed-keys-in-customer-controlled-hardware"></a>Cifrado del lado servidor mediante claves administradas por el cliente en el hardware controlado por el cliente

Algunos servicios de Azure permiten el modelo de administración de claves Host Your Own Key (HYOK). Este modo de administración es útil en escenarios donde hay una necesidad para cifrar los datos en reposo y administrar las claves en un repositorio patentado fuera del control de Microsoft. En este modelo, el servicio debe recuperar la clave de un sitio externo. Las garantías de rendimiento y disponibilidad se ven afectadas y la configuración es más compleja. Además, puesto que el servicio tiene acceso a la DEK durante las operaciones de cifrado y descifrado de las garantías de seguridad general de este modelo son similares a cuando las claves son administradas en Azure Key Vault por el cliente.  Como resultado, este modelo no es adecuado para la mayoría de las organizaciones a menos que tengan requisitos específicos de administración de claves. Debido a estas limitaciones, la mayoría de los servicios de Azure no admiten el cifrado del lado del servidor mediante claves de servidor administradas en el hardware controlado por el cliente.

### <a name="key-access"></a>Acceso a la clave

Cuando se usa el cifrado del lado servidor mediante las claves administradas del servicio en el hardware controlado del cliente, las claves se mantienen en un sistema configurado por el cliente. Los servicios de Azure que admiten este modelo proporcionan un medio para establecer una conexión segura en un almacén de claves proporcionado por el cliente.

**Ventajas**

- Control total sobre la clave raíz usada: una tienda proporcionada por el un cliente administra las claves de cifrado
- Capacidad de cifrar varios servicios en un patrón
- Puede separar la administración de la clave del modelo de administración global para el servicio
- Puede definir el servicio y la ubicación de la clave en regiones

**Desventajas**

- Responsabilidad total para la disponibilidad, rendimiento, seguridad y almacenamiento de claves
- Responsabilidad total para la administración de acceso a las claves
- Responsabilidad total para la administración del ciclo de vida de las claves
- Costos significativos de mantenimiento en curso, instalación y configuración
- Dependencia aumentada sobre la disponibilidad de la red entre el centro de datos del cliente y los centros de datos de Azure.

## <a name="supporting-services"></a>Servicios compatibles
Los servicios de Azure que admiten cada modelo de cifrado:

| Producto, característica o servicio | Lado servidor con clave administrada por el servicio   | Lado servidor con clave administrada por el cliente | Lado cliente con clave administrada por el cliente  |
|----------------------------------|--------------------|-----------------------------------------|--------------------|
| **Inteligencia artificial y aprendizaje automático**      |                    |                    |                    |
| Azure Cognitive Search           | Sí                | Sí                | -                  |
| Azure Cognitive Services         | Sí                | Sí                | -                  |
| Azure Machine Learning           | Sí                | Sí                | -                  |
| Azure Machine Learning Studio    | Sí                | Versión preliminar, RSA de 2048 bits | -               |
| Content Moderator                | Sí                | Sí                | -                  |
| Caras                             | Sí                | Sí                | -                  |
| Language Understanding           | Sí                | Sí                | -                  |
| Personalizer                     | Sí                | Sí                | -                  |
| QnA Maker                        | Sí                | Sí                | -                  |
| Speech Services                  | Sí                | Sí                | -                  |
| Translator Text                  | Sí                | Sí                | -                  |
| Power BI                         | Sí                | Versión preliminar, RSA de 2048 bits | -                  |
| **Analytics**                    |                    |                    |                    |
| Azure Stream Analytics           | Sí                | N/D\*              | -                  |
| Event Hubs                       | Sí                | Sí                | -                  |
| Functions                        | Sí                | Sí                | -                  |
| Azure Analysis Services          | Sí                | -                  | -                  |
| Azure Data Catalog               | Sí                | -                  | -                  |
| HDInsight de Azure                  | Sí                | All                | -                  |
| Azure Monitor Application Insights | Sí                | Sí                | -                  |
| Azure Monitor Log Analytics      | Sí                | Sí                | -                  |
| Explorador de datos de Azure              | Sí                | Sí                | -                  |
| Azure Data Factory               | Sí                | Sí                | -                  |
| Azure Data Lake Store            | Sí                | Sí, RSA de 2048 bits  | -                  |
| **Contenedores**                   |                    |                    |                    |
| Azure Kubernetes Service         | Sí                | Sí                | -                  |
| Azure Container Instances              | Sí                | Sí                | -                  |
| Container Registry               | Sí                | Sí                | -                  |
| **Proceso**                      |                    |                    |                    |
| Virtual Machines                 | Sí                | Sí                | -                  |
| Conjunto de escalado de máquinas virtuales        | Sí                | Sí                | -                  |
| SAP HANA                         | Sí                | Sí                | -                  |
| App Service                      | Sí                | Sí\*\*            | -                  |
| Automation                       | Sí                | Sí\*\*            | -                  |
| Azure Functions                  | Sí                | Sí\*\*            | -                  |
| Azure portal                     | Sí                | Sí\*\*            | -                  |
| Logic Apps                       | Sí                | Sí                | -                  |
| Aplicaciones administradas por Azure       | Sí                | Sí\*\*            | -                  |
| Azure Service Bus                      | Sí                | Sí                | -                  |
| Site Recovery                    | Sí                | Sí                | -                  |
| **Bases de datos**                    |                    |                    |                    |
| SQL Server en máquinas virtuales   | Sí                | Sí                | Sí                |
| Azure SQL Database               | Sí                | Sí, RSA de 3072 bits  | Sí                |
| Azure SQL Database for MariaDB   | Sí                | -                  | -                  |
| Azure SQL Database for MySQL     | Sí                | Sí                | -                  |
| Azure SQL Database for PostgreSQL | Sí               | Sí                | -                  |
| Azure Synapse Analytics          | Sí                | Sí, RSA de 3072 bits  | -                  |
| SQL Server Stretch Database      | Sí                | Sí, RSA de 3072 bits  | Sí                |
| Table Storage                    | Sí                | Sí                | Sí                |
| Azure Cosmos DB                  | Sí                | Sí                | -                  |
| Azure Databricks                 | Sí                | Sí                | -                  |
| Azure Database Migration Service | Sí                | N/D\*              | -                  |
| **DevOps**                       |                    |                    |                    |
| Azure DevOps Services            | Sí                | -                  | Sí                |
| Azure Repos                      | Sí                | -                  | Sí                |
| **Identidad**                     |                    |                    |                    |
| Azure Active Directory           | Sí                | -                  | -                  |
| Azure Active Directory Domain Services | Sí          | Sí                | -                  |
| **Integración**                  |                    |                    |                    |
| Azure Service Bus                      | Sí                | Sí                | Sí                |
| Event Grid                       | Sí                | -                  | -                  |
| API Management                   | Sí                | -                  | -                  |
| **Servicios IoT**                 |                    |                    |                    |
| IoT Hub                          | Sí                | Sí                | Sí                |
| IoT Hub Device Provisioning      | Sí                | Sí                | -                  |
| **Administración y gobernanza**    |                    |                    |                    |
| Azure Site Recovery              | Sí                | -                  | -                  |
| Azure Migrate                    | Sí                | Sí                | -                  |
| **Elementos multimedia**                        |                    |                    |                    |
| Media Services                   | Sí                | -                  | Sí                |
| **Seguridad**                     |                    |                    |                    |
| Preguntas más frecuentes de Azure Security Center for IoT    | Sí                | Sí                | -                  |
| Azure Sentinel                   | Sí                | Sí                | -                  |
| **Storage**                      |                    |                    |                    |
| Blob Storage                     | Sí                | Sí                | Sí                |
| Premium Blob Storage             | Sí                | Sí                | Sí                |
| Disk Storage                     | Sí                | Sí                | -                  |
| Almacenamiento en disco Ultra               | Sí                | Sí                | -                  |
| Disk Storage administrado             | Sí                | Sí                | -                  |
| File Storage                     | Sí                | Sí                | -                  |
| Premium Storage de archivos             | Sí                | Sí                | -                  |
| File Sync                        | Sí                | Sí                | -                  |
| Queue Storage                    | Sí                | Sí                | Sí                |
| Avere vFXT                       | Sí                | -                  | -                  |
| Azure Cache for Redis            | Sí                | N/D\*              | -                  |
| Azure NetApp Files               | Sí                | Sí                | -                  |
| Archive Storage                  | Sí                | Sí                | -                  |
| StorSimple                       | Sí                | Sí                | Sí                |
| Azure Backup                     | Sí                | Sí                | Sí                |
| Data Box                         | Sí                | -                  | Sí                |
| Data Box Edge                    | Sí                | Sí                | -                  |

\* Este servicio no conserva los datos. Las memorias caché transitorias, si las hay, se cifran con una clave de Microsoft.

\*\* Este servicio admite el almacenamiento de datos en su propio Key Vault, cuenta de almacenamiento u otro servicio de persistencia de datos que ya admita el cifrado del lado servidor con una clave administrada por el cliente.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda [cómo se usa el cifrado de en Azure](encryption-overview.md).
- Obtenga información sobre cómo usa Azure el [cifrado doble](double-encryption.md) para mitigar las amenazas que incluye el cifrado de datos.
