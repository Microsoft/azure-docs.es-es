---
title: Guía de introducción para operadores de TI de Azure | Microsoft Docs
description: Guía de introducción para operadores de TI de Azure
author: RicksterCDN
ms.author: rclaus
tags: azure-resource-manager
ms.service: azure
ms.topic: overview
ms.workload: infrastructure
ms.date: 08/24/2018
ms.openlocfilehash: 0a383e8e2c38dbd00e0a55f9debb9b4daf40fa9c
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87832763"
---
# <a name="get-started-for-azure-it-operators"></a>Introducción para operadores de TI de Azure

Esta guía presenta los conceptos básicos relacionados con la implementación y administración de una infraestructura de Microsoft Azure. Si no tiene experiencia con la informática en la nube o Azure, esta guía le ayuda a rápidamente empezar con los conceptos, implementación y detalles de administración. Varias secciones de esta guía presentan una operación, como la implementación de una máquina virtual y, después, se proporciona un vínculo a información técnica más detallada.

## <a name="cloud-computing-overview"></a>Información general sobre la informática en la nube

La informática en la nube proporciona una alternativa moderna a los centros de datos locales tradicionales. Los proveedores de la nube pública proporcionan y administran toda la infraestructura informática y el software de administración subyacente. Estos proveedores ofrecen una amplia variedad de servicios en la nube. Un servicio en la nube en este caso podría ser una máquina virtual, un servidor web o un motor de base de datos hospedado en la nube. Los clientes de estos proveedores alquilan los servicios en la nube en función de sus necesidades. Esto convierte los gastos de capital de mantenimiento de hardware en gastos de operación. Un servicio en la nube también ofrece estas ventajas:

- Implementación rápida de entornos de proceso de gran tamaño

- Desasignación rápida de los sistemas que ya no sean necesarios

- Implementación sencilla de sistemas tradicionalmente complejos, como los equilibradores de carga

- Posibilidad de proporcionar capacidad de proceso flexible o escalar cuando sea necesario

- Entornos informáticos más rentables

- Acceso desde cualquier lugar mediante un portal basado en web y automatización mediante programación

- Servicios en la nube para satisfacer la mayoría de las necesidades de aplicaciones y proceso

Con la infraestructura local, tiene un control completo sobre el hardware y software que se implementa. Históricamente, esto ha llevado a tomar decisiones de adquisición de hardware centradas en el escalado. Un ejemplo es la adquisición de un servidor con más núcleos para satisfacer las necesidades de rendimiento en puntos de máximos. Desafortunadamente, esta infraestructura podría estar infrautilizada fuera de las ventanas de alta demanda. Con Azure, puede implementar únicamente la infraestructura que necesita y ajustarla hacia arriba o hacia abajo en cualquier momento. Esto conduce a un enfoque en el escalado horizontal mediante la implementación de nodos de proceso adicionales para satisfacer las necesidades de rendimiento. El escalado horizontal de servicios en la nube es más rentable que el escalado vertical mediante hardware de alto costo.

Microsoft ha implementado muchos centros de datos de Azure todo el mundo y hay más previstos. Además, Microsoft está aumentando las nubes independientes en regiones como China y Alemania. Solo las empresas más grandes a nivel global pueden implementar centros de datos de esta manera, por lo que el uso de Azure facilita a las empresas de cualquier tamaño la implementación de los servicios cerca de sus clientes.

Para las pequeñas empresas, Azure permite un punto de entrada de bajo costo, con la capacidad de escalar rápidamente a medida que aumenta la demanda de proceso. Esto evita una gran inversión de capital inicial en infraestructura y proporciona la flexibilidad para diseñar y volver a diseñar sistemas según sea necesario. El uso de la informática en la nube encaja bien con los modelos de escalado rápido y de fracaso y respuesta rápida a los errores de las startups.

Para más información sobre las regiones de Azure disponibles, consulte [Regiones de Azure](https://azure.microsoft.com/regions/).

### <a name="cloud-computing-model"></a>Modelo de informática en la nube

Azure usa una modelo de informática en la nube basado en categorías de servicio proporcionado a los clientes. Las tres categorías de servicio incluyen: infraestructura como servicio (IaaS), plataforma como servicio (PaaS) y software como servicio (SaaS). Los proveedores comparten alguna o toda la responsabilidad por los componentes de la pila informática en cada una de estas categorías. Echemos un vistazo a cada una de las categorías de informática en la nube.
![Comparación de pilas de informática en la nube](./media/cloud-computing-comparison.png)

#### <a name="iaas-infrastructure-as-a-service"></a>IaaS: infraestructura como servicio

Un proveedor en la nube de IaaS ejecuta y administra todos los recursos de proceso físicos y el software necesario para habilitar la virtualización de equipos. Un cliente de este servicio implementa máquinas virtuales en centros de datos hospedados. Aunque las máquinas virtuales se encuentran en un centro de datos remoto, el consumidor de IaaS tiene control sobre la configuración y administración del sistema operativo dejando la infraestructura subyacente al proveedor de la nube.

Azure ofrece varias soluciones de IaaS que incluyen máquinas virtuales, conjuntos de escalado de máquinas virtuales y la infraestructura de red relacionada. Las máquinas virtuales son una elección popular para la migración inicial de los servicios a Azure porque posibilitan un modelo de migración "levantar y mover". Puede configurar una máquina virtual similar a la infraestructura que ejecuta los servicios actualmente en su centro de datos y, a continuación, migrar el software a la nueva máquina virtual. Tendrá que realizar cambios de configuración, como las direcciones URL para otros servicios o almacenamiento, pero puede migrar muchas aplicaciones de esta manera.

Los conjuntos de escalado de máquinas virtuales se crean sobre Azure Virtual Machines y proporcionan una manera fácil de implementar clústeres de máquinas virtuales idénticas. Los conjuntos de escalado de máquinas virtuales también admiten el escalado automático para que se pueden implementar automáticamente nuevas máquinas virtuales cuando sea necesario. Esto hace de los conjuntos de escalado de máquinas virtuales una plataforma ideal para hospedar clústeres de proceso de microservicios del más alto nivel como, por ejemplo, Azure Service Fabric y Azure Container Service.

#### <a name="paas-platform-as-a-service"></a>PaaS: plataforma como servicio

Con PaaS, se implementa la aplicación en un entorno proporcionado por el proveedor de servicios en la nube. El proveedor lleva a cabo toda la administración de la infraestructura de forma que el usuario puede centrarse en el desarrollo de aplicaciones y la administración de datos.

Azure proporciona varias ofertas de proceso PaaS, incluida la característica Web Apps de Azure App Service y Azure Cloud Services (roles web y de trabajo). En cualquier caso, los desarrolladores tienen varias maneras de implementar su aplicación sin necesidad de saber nada acerca de la plataforma que la soporta. Los desarrolladores no necesitan crear máquinas virtuales, usar el protocolo de escritorio remoto (RDP) para iniciar sesión en cada una de ellas ni instalar la aplicación. Basta con pulsar un botón (o casi) y las herramientas proporcionadas por Microsoft aprovisionan las máquinas virtuales y, a continuación, implementan e instalan la aplicación en ellas.

#### <a name="saas-software-as-a-service"></a>SaaS: software como servicio

SaaS es un software que se hospeda y administra de forma centralizada. Normalmente se basa en una arquitectura multiinquilino: se usa una única versión de la aplicación para todos los clientes. Puede escalar horizontalmente a varias instancias para garantizar el máximo rendimiento en todas las ubicaciones. El software SaaS normalmente se licencia a través de una suscripción mensual o anual. Los proveedores de software SaaS son responsables de todos los componentes de la pila de software, por lo que el usuario solo tiene que administrar los servicios proporcionados.

Microsoft Office 365 es un buen ejemplo de una oferta de SaaS. Los suscriptores pagan una cuota de suscripción mensual o anual y obtienen Microsoft Exchange, Microsoft OneDrive y el resto de Microsoft Office Suite como un servicio. Los suscriptores reciben siempre la versión más reciente y el servidor de Exchange se administra para ellos. En comparación con la instalación y actualización de Office cada año, esto resulta más económico y requiere menos esfuerzo.

## <a name="azure-services"></a>Servicios de Azure

Azure ofrece muchos servicios en su plataforma de informática en la nube. Estos servicios incluyen los siguientes.

### <a name="compute-services"></a>Compute Services

Servicios para hospedar y ejecutar cargas de trabajo de aplicación:

- Azure Virtual Machines, tanto Linux como Windows

- App Services (Web Apps, Mobile Apps, Logic Apps, API Apps y Function Apps)

- Azure Batch (para trabajos de proceso en paralelo a gran escala y trabajos de proceso por lotes)

- Azure Service Fabric

- Azure Container Service

### <a name="data-services"></a>Servicios de datos

Servicios para almacenar y administrar datos:

- Azure Storage (consta de los servicios Azure Blob, Queue, Table y File)

- Azure SQL Database

- Azure Cosmos DB

- Microsoft Azure StorSimple

- Azure Cache for Redis

### <a name="application-services"></a>Servicios de aplicación

Servicios para la creación y operación de aplicaciones:

- Azure Active Directory (Azure AD)

- Azure Service Bus para conectar sistemas distribuidos

- Azure HDInsight para el proceso de macrodatos

- Azure Logic Apps para flujos de trabajo de integración y orquestación

- Azure Media Services

### <a name="network-services"></a>Servicios de red

Servicios de red dentro de Azure y entre los centros de datos de Azure y locales:

- Azure Virtual Network

- Azure ExpressRoute

- DNS proporcionado por Azure

- Administrador de tráfico de Azure

- Azure Content Delivery Network

Para obtener documentación detallada sobre los servicios de Azure, consulte la [documentación del servicio Azure](https://docs.microsoft.com/azure).

## <a name="azure-key-concepts"></a>Conceptos clave de Azure

### <a name="datacenters-and-regions"></a>Centros de datos y regiones

Azure es una plataforma en la nube global que está disponible con carácter general en muchas regiones de todo el mundo. Al aprovisionar un servicio, una aplicación o una máquina virtual de Azure, deberá seleccionar una región. La región seleccionada representa un centro de datos específico en el que se ejecuta la aplicación. Para más información, consulte [Regiones de Azure](https://azure.microsoft.com/regions/).

Una de las ventajas del uso de Azure es que puede implementar aplicaciones en distintos centros de datos de todo el mundo. La región que elija puede afectar al rendimiento de la aplicación. Resulta óptimo elegir la región más cercana a la mayoría de sus clientes, para reducir la latencia de las solicitudes de red. También puede seleccionar una región para cumplir los requisitos legales para distribuir la aplicación en determinados países y regiones.

### <a name="azure-portal"></a>Azure portal

Azure Portal es una aplicación basada en web que puede usarse para crear, administrar y eliminar los servicios y recursos de Azure. Azure Portal se encuentra en [portal.azure.com](https://portal.azure.com). Incluye un panel personalizable y herramientas para administrar los recursos de Azure. También proporciona información de suscripciones y facturación. Para más información, consulte [Información general sobre Microsoft Azure Portal](https://azure.microsoft.com/documentation/articles/azure-portal-overview/) y [Administración de los recursos de Azure a través del Portal](https://docs.microsoft.com/azure/azure-portal/resource-group-portal).

### <a name="resources"></a>Recursos

Los recursos de Azure son servicios individuales de proceso, redes, datos u hospedaje de aplicaciones que se han implementado en una suscripción de Azure. Algunos recursos comunes son máquinas virtuales, cuentas de almacenamiento o bases de datos SQL. Los servicios de Azure a menudo constan de varios recursos de Azure relacionados. Por ejemplo, una máquina virtual de Azure puede incluir una máquina virtual, la cuenta de almacenamiento, el adaptador de red y la dirección IP pública. Estos recursos se pueden crear, administrar y eliminar individualmente o como un grupo. Los recursos de Azure se tratan con más detalle más adelante en esta guía.

### <a name="resource-groups"></a>Grupos de recursos

Un grupo de recursos de Azure es un contenedor que almacena los recursos relacionados con una solución de Azure. El grupo de recursos puede incluir todos los recursos de la solución o solo aquellos que se desean administrar como grupo. Los grupos de recursos de Azure se tratan con más detalle más adelante en esta guía.

### <a name="resource-manager-templates"></a>Plantillas de Resource Manager

Una plantilla de Azure Resource Manager es un archivo de notación de objetos JavaScript (JSON) que define uno o más recursos para implementar en un grupo de recursos. También define las dependencias entre los recursos implementados. Las plantillas de Resource Manager se tratan con más detalle más adelante en esta guía.

### <a name="automation"></a>Automation

Además de crear, administrar y eliminar recursos mediante Azure Portal, puede automatizar estas actividades con PowerShell o con la interfaz de línea de comandos (CLI) de Azure.

#### <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell es un conjunto de módulos que ofrece varios cmdlet para administrar Azure. Puede usar los cmdlet para crear, administrar y eliminar servicios de Azure. Con los cmdlet puede lograr implementaciones coherentes, repetibles y sin intervención humana. Para obtener más información, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/install-Az-ps).

#### <a name="azure-command-line-interface"></a>Interfaz de la línea de comandos de Azure

La interfaz de línea de comandos de Azure es una herramienta que puede usar para crear, administrar y quitar recursos de Azure desde la línea de comandos. La CLI de Azure está disponible para Windows, Linux y Mac OS X. Para más información y detalles técnicos, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

#### <a name="rest-apis"></a>API de REST

Azure se basa en un conjunto de API de REST que dan soporte a la interfaz de usuario de Azure Portal. También se admite la mayoría de estas API de REST para aprovisionar y administrar los recursos y aplicaciones de Azure desde cualquier dispositivo con acceso a Internet mediante programación. Para más información, consulte la [Referencia de SDK de REST de Azure](https://docs.microsoft.com/rest/api/index).

### <a name="azure-cloud-shell"></a>Azure Cloud Shell

Los administradores pueden acceder a Azure PowerShell y la CLI de Azure mediante una experiencia accesible desde el explorador llamada Azure Cloud Shell. Esta interfaz interactiva proporciona una herramienta flexible para que los administradores de Windows y Linux usen la interfaz de línea de comandos que prefieran, Bash o PowerShell. Se puede acceder a Azure Cloud Shell a través del portal, como una interfaz web independiente en [shell.azure.com](https://shell.azure.com), o desde una serie de otros puntos de acceso. Para más información, vea [Introducción a Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

## <a name="azure-subscriptions"></a>Suscripciones de Azure

Una suscripción es una agrupación lógica de servicios de Azure que está vinculada a una cuenta de Azure. Una única cuenta de Azure puede contener varias suscripciones. La facturación de los servicios de Azure se realiza por suscripción. Las suscripciones de Azure tienen un administrador de cuenta, que tiene control total sobre la suscripción y un administrador de servicios, que tiene control sobre todos los servicios de la suscripción. Para saber más sobre los administradores de suscripción clásica, vea [Agregar o cambiar los administradores de la suscripción de Azure](../../cost-management-billing/manage/add-change-subscription-administrator.md). Además de a los administradores, se puede conceder control detallado de los recursos de Azure con [control de acceso basado en rol de Azure (Azure RBAC)](../../role-based-access-control/overview.md) a cuentas individuales.

### <a name="select-and-enable-an-azure-subscription"></a>Seleccionar y habilitar una suscripción de Azure

Para poder trabajar con los servicios de Azure, necesita una suscripción. Existen varios tipos de suscripción disponibles.

**Cuentas gratuita**: el vínculo para registrarse para una cuenta gratuita se encuentra en el [sitio web de Azure](https://azure.microsoft.com/). Esto proporciona un crédito a lo largo de 30 días para probar cualquier combinación de recursos de Azure. Si se supera la cantidad de crédito, la cuenta será suspendida. Al final del período de prueba, los servicios se retiran y dejará de funcionar. Puede actualizar a una suscripción de pago por uso en cualquier momento.

**Suscripciones de MSDN**: si tiene una suscripción de MSDN, obtendrá una cantidad específica de crédito de Azure cada mes. Por ejemplo, si tiene Microsoft Visual Studio Enterprise con una suscripción de MSDN, obtendrá \$150 al mes de crédito de Azure.

Si se supera la cantidad de crédito, los servicios se deshabilitan hasta que se inicie el mes siguiente. Puede desactivar el límite de gasto y agregar una tarjeta de crédito que se usará para los costos adicionales. Algunos de estos costos tienen descuento para las cuentas de MSDN. Por ejemplo, se paga el precio de Linux por las máquinas virtuales con Windows Server y no hay ningún cargo adicional para los servidores de Microsoft, como Microsoft SQL Server. Esto hace que las cuentas de MSDN sean ideales para escenarios de desarrollo y pruebas.

**Cuentas de BizSpark**: el programa Microsoft BizSpark ofrece numerosas ventajas para startups. Una de esas ventajas es el acceso a todo el software de Microsoft para entornos de desarrollo y pruebas para hasta cinco de las cuentas de MSDN. Obtendrá 150 dólares en crédito de Azure por cada una de las cinco cuentas MSDN y pagará tarifas reducidas por algunos de los servicios de Azure, como las máquinas virtuales.

**Pago por uso**: con esta suscripción, se asocia una tarjeta de crédito o débito a la cuenta y se paga por lo que se usa. Si se trata de una organización, también puede aprobarse la facturación.

**Contratos Enterprise**: con un contrato Enterprise, se compromete a utilizar un número determinado de servicios de Azure durante el año siguiente y deberá pagar esa cantidad por anticipado. El compromiso que realiza se consume durante el año. Si se supera la cantidad del compromiso, el uso por encima del límite se paga a período vencido. En función de la cantidad del compromiso, obtendrá un descuento en los servicios de Azure.

### <a name="grant-administrative-access-to-an-azure-subscription"></a>Concesión acceso administrativo a una suscripción de Azure

RBAC tiene varios roles integrados que puede usar para asignar permisos. Para que un usuario sea administrador de una suscripción de Azure, asígnele el rol [Propietario](../../role-based-access-control/built-in-roles.md#owner) en el ámbito de la suscripción. El rol Propietario da al usuario acceso completo a todos los recursos de la suscripción, incluido el derecho a delegar este acceso a otros.

Para saber más, vea [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).

### <a name="view-billing-information-in-the-azure-portal"></a>Visualización de la información de facturación en Azure Portal

Un componente importante del uso de Azure es la funcionalidad para ver información de facturación. Azure Portal proporciona una visión detallada de la información de facturación de Azure.

Para más información, consulte [Cómo descargar las datos de uso diario y de factura de facturación de Azure](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md).

### <a name="get-billing-information-from-billing-apis"></a>Obtención de la información de facturación desde las API de facturación

Además de ver la facturación en el portal, puede tener acceso a la información de facturación mediante un script o programa a través de las API de REST de facturación de Azure:

- Puede utilizar la API de uso de Azure para recuperar los datos de uso. Puede ajustar la información de uso de facturación etiquetando los recursos de Azure relacionados. Por ejemplo, puede etiquetar cada uno de los recursos de un grupo de recursos con un nombre de departamento o un nombre de proyecto y, a continuación, realizar un seguimiento de los costos específicamente para dicha etiqueta.

- Puede usar la API Rate Card de Azure para enumerar todos los recursos disponibles, junto con los metadatos y la información de precios de cada uno de esos recursos.

Para más información, consulte [Obtención de información sobre el consumo de recursos de Microsoft Azure](../../cost-management-billing/manage/usage-rate-card-overview.md).

### <a name="forecast-cost-with-the-pricing-calculator"></a>Previsión del costo con la calculadora de precios

El precio de cada servicio en Azure es diferente. Muchos servicios de Azure proporcionan niveles Basic, Standard y Premium. Normalmente, cada nivel tiene varios niveles de precio y rendimiento. Mediante el uso de la [Calculadora de precios en línea](https://azure.microsoft.com/pricing/calculator), puede crear estimaciones de precios. La calculadora incluye flexibilidad para calcular el costo en un único recurso o un grupo de recursos.

## <a name="azure-resource-manager"></a>Azure Resource Manager

Azure Resource Manager es un mecanismo de implementación, administración y organización de los recursos de Azure. Mediante Resource Manager, puede colocar muchos recursos individuales juntos en un grupo de recursos.

Resource Manager también incluye funcionalidades de implementación que permiten la implementación personalizable y la configuración de los recursos relacionados. Por ejemplo, mediante Resource Manager, puede implementar una aplicación que consta de varias máquinas virtuales, un equilibrador de carga y una base de datos en Azure SQL Database como una sola unidad. Estas implementaciones se desarrollan usando una plantilla de Resource Manager.

Administrador de recursos ofrece varias ventajas:

- Puede implementar, administrar y supervisar todos los recursos de la solución en grupo, en lugar de controlarlos individualmente.

- Puede implementar la solución repetidamente a lo largo del ciclo de vida del desarrollo y tener la seguridad de que los recursos se implementan de forma coherente.

- Puede administrar la infraestructura mediante plantillas declarativas en lugar de scripts.

- Puede definir las dependencias entre recursos de modo que se implementen en el orden correcto.

- Puede aplicar control de acceso a todos los servicios del grupo de recursos al integrarse RBAC de forma nativa en la plataforma de administración.

- Puede aplicar etiquetas a los recursos para organizar de manera lógica todos los recursos de la suscripción.

- Puede clarificar la facturación de su organización viendo los costos de un grupo de recursos que compartan la misma etiqueta.

### <a name="tips-for-creating-resource-groups"></a>Sugerencias para crear grupos de recursos

Cuando vaya a tomar decisiones acerca de los grupos de recursos, tenga en cuenta estas sugerencias:

- Todos los recursos de un grupo de recursos deben tener el mismo ciclo de vida.

- Solo es posible asignar un recurso a un grupo cada vez.

- Puede agregar o quitar un recurso de un grupo de recursos en cualquier momento. Todos los recursos deben pertenecer a un grupo de recursos. Por lo que si quita un recurso de un grupo, debe agregarlo a otro.

- Puede mover la mayoría de los tipos de recursos a otro grupo de recursos en cualquier momento.

- Los recursos de un grupo de recursos pueden estar en regiones diferentes.

- Puede utilizar un grupo de recursos para controlar el acceso a los recursos del mismo.

### <a name="building-resource-manager-templates"></a>Creación de plantillas de Resource Manager

Las plantillas de Resource Manager definen mediante declaración los recursos y las configuraciones de recursos que se implementarán en un único grupo de recursos. Puede utilizar las plantillas de Resource Manager para coordinar implementaciones complejas sin necesidad de scripting o configuración manual excesivas. Después de desarrollar una plantilla, puede implementarla varias veces, cada vez con idéntico resultado.

Una plantilla de Resource Manager está formada por cuatro secciones:

- **Parámetros**: son las entradas a la implementación. Los valores de los parámetros pueden ser proporcionados por una persona o por un proceso automatizado. Un ejemplo de parámetro podría ser un nombre de usuario administrador y una contraseña para una máquina virtual de Windows. Los valores de los parámetros se utilizan a lo largo de la implementación cuando se especifican.

- **Variables**: se utilizan para contener valores que se usan en toda la implementación. A diferencia de los parámetros, los valores de las variables no se proporcionan en tiempo de implementación. En su lugar, se define en código o se genera dinámicamente.

- **Recursos:** esta sección de la plantilla define los recursos que se van a implementar, como máquinas virtuales, cuentas de almacenamiento y redes virtuales.

- **Salida**: una vez finalizada una implementación, Resource Manager puede devolver datos como, por ejemplo, cadenas de conexión generadas dinámicamente.

Los mecanismos siguientes están disponibles para la automatización de una implementación:

- **Funciones**: puede usar varias funciones en las plantillas de Resource Manager. Estas incluyen operaciones como convertir una cadena a minúsculas, implementar varias instancias de un recurso definido o devolver dinámicamente el grupo de recursos de destino. Las funciones de Resource Manager ayudan a crear implementaciones dinámicas.

- **Dependencias de recursos**: si va a implementar varios recursos, algunos recursos tendrán una dependencia de otros. Para facilitar la implementación, puede usar una declaración de dependencia para que se implementen los recursos dependientes antes que los demás.

- **Vinculación de plantilla**: desde dentro de una plantilla de Resource Manager, puede vincular a otra plantilla. Esto permite la descomposición de la implementación en un conjunto de plantillas de propósito específico.

Puede crear plantillas de Resource Manager en cualquier editor de texto. Sin embargo, el SDK de Azure para Visual Studio incluye herramientas para ayudarle. Con Visual Studio, puede agregar recursos a la plantilla mediante un asistente y, a continuación, implementar y depurar la plantilla directamente desde Visual Studio. Para más información, consulte [Creación de plantillas de Azure Resource Manager](../../resource-group-authoring-templates.md).

Por último, puede convertir grupos de recursos existentes en una plantilla reutilizable desde Azure Portal. Esto puede resultar útil si desea crear una plantilla de implementación de un grupo de recursos existente o si simplemente desea examinar el JSON subyacente. Para exportar un grupo de recursos, seleccione el botón **Script de automatización** desde la configuración del grupo de recursos.

## <a name="security-of-azure-resources-rbac"></a>Seguridad de los recursos de Azure (RBAC)

Puede conceder acceso operativo a cuentas de usuario en un ámbito especificado: suscripción, grupo de recursos o un recurso individual. Esto significa que puede implementar un conjunto de recursos en un grupo de recursos, como una máquina virtual y todos los recursos relacionados y conceder permisos a un usuario o grupo específico. Este enfoque limita el acceso únicamente a los recursos que pertenecen al grupo de recursos de destino. También puede conceder acceso a un único recurso, por ejemplo, una máquina virtual o una red virtual.

Para conceder acceso, asigne un rol para el usuario o grupo de usuarios. Existen muchos roles predefinidos. También puede definir sus propios roles personalizados.

Estos son algunos ejemplos de [roles integrados en Azure](../../role-based-access-control/built-in-roles.md):

- **Propietario**: un usuario con este rol puede administrarlo todo, incluido el acceso.

- **Lector**: un usuario con este rol puede leer recursos de todos los tipos (excepto los secretos) pero no puede realizar cambios.

- **Colaborador de la máquina virtual**: un usuario con este rol puede administrar máquinas virtuales, pero no puede administrar la red virtual a la que está conectado o la cuenta de almacenamiento donde reside el archivo de disco duro virtual.

- **Colaborador de base de datos SQL**: un usuario con este rol puede administrar bases de datos SQL, pero no sus directivas relacionadas con la seguridad.

- **Administrador de seguridad SQL**: un usuario con este rol puede administrar las directivas relacionadas con la seguridad de servidores SQL y bases de datos.

- **Colaborador de la cuenta de almacenamiento**: un usuario con este rol puede administrar las cuentas de almacenamiento pero no puede administrar el acceso a las cuentas de almacenamiento.

Para saber más, vea [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).

## <a name="azure-virtual-machines"></a>Azure Virtual Machines

Azure Virtual Machines es uno de los servicios centrales de IaaS en Azure. Azure Virtual Machines permite la implementación de máquinas virtuales de Windows o Linux en un centro de datos de Microsoft Azure. Con Azure Virtual Machines, tiene control total sobre la configuración de la máquina virtual y es responsable de la instalación, configuración y mantenimiento de todo el software.

Al implementar una máquina virtual de Azure, puede seleccionar una imagen desde Azure Marketplace o puede proporcionar su propia imagen personalizada. Esta imagen se usa para aplicar el sistema operativo y la configuración inicial. Durante la implementación, Resource Manager también controlará algunos valores de configuración, como la asignación del nombre del equipo, las credenciales administrativas y la configuración de red. Puede usar las extensiones de máquina virtual de Azure para automatizar aún más configuraciones como la instalación de software, configuración de antivirus y soluciones de supervisión.

Puede crear máquinas virtuales de muchos tamaños diferentes. El tamaño de la máquina virtual determina la asignación de recursos, como la capacidad de procesamiento, memoria y almacenamiento. En algunos casos, características específicas como los adaptadores de red habilitados para RDMA y los discos SSD están disponibles solo con determinados tamaños de máquina virtual. Para obtener una lista completa de funcionalidades y tamaños de máquina virtual, consulte "Tamaños de máquinas virtuales en Azure" para [Windows](../../virtual-machines/windows/sizes.md) y [Linux](../../virtual-machines/linux/sizes.md).

### <a name="use-cases"></a>Casos de uso

Dado que las máquinas virtuales de Azure ofrecen un control completo sobre la configuración, son ideales para una amplia variedad de cargas de trabajo de servidor que no encajan en un modelo de PaaS. Las cargas de trabajo de servidor como los servidores de base de datos (SQL Server, Oracle o MongoDB), Windows Server Active Directory, Microsoft SharePoint y muchas más pueden ejecutarse en la plataforma de Microsoft Azure. Si lo desea, puede mover estas cargas de trabajo desde un centro de datos local a una o varias regiones de Azure, sin una gran cantidad de reconfiguración.

### <a name="deployment-of-virtual-machines"></a>Implementación de máquinas virtuales

Puede implementar máquinas virtuales de Azure mediante Azure Portal, mediante automatización con el módulo de PowerShell de Azure o mediante automatización con la CLI multiplataforma.

#### <a name="portal"></a>Portal

La implementación de una máquina virtual mediante Azure Portal requiere solo una suscripción activa de Azure y acceso a un explorador web. Puede seleccionar muchas imágenes de sistema operativo diferentes con distintas configuraciones. Todos los requisitos de almacenamiento y redes se configuran durante la implementación. Para más información, consulte "Creación de una máquina virtual en Azure Portal" para [Windows](../../virtual-machines/windows/quick-create-portal.md) y [Linux](../../virtual-machines/linux/quick-create-portal.md).

Además de implementar una máquina virtual desde Azure Portal, puede implementar una plantilla de Azure Resource Manager desde el portal. Esto implementará y configurará todos los recursos, tal y como se define en la plantilla. Para más información, consulte [Implementación de recursos con plantillas de Resource Manager y Azure Portal](../../azure-resource-manager/templates/deploy-portal.md).

#### <a name="powershell"></a>PowerShell

La implementación de una máquina virtual de Azure mediante PowerShell permite la automatización completa de la implementación de todos los recursos de máquina virtual relacionados, incluidos el almacenamiento y las redes. Para más información, consulte [Creación de una máquina virtual Windows mediante Resource Manager y PowerShell](../../virtual-machines/windows/quick-create-powershell.md).

Además de implementar individualmente los recursos de proceso de Azure, puede usar el módulo Azure PowerShell para implementar una plantilla de Azure Resource Manager. Para más información, consulte [Implementación de recursos con plantillas de Resource Manager y Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md).

#### <a name="command-line-interface-cli"></a>Interfaz de la línea de comandos (CLI)

Al igual que con el módulo de PowerShell, la interfaz de línea de comandos de Azure proporciona automatización de la implementación y puede usarse en sistemas Windows, OS X o Linux. Cuando se usa el comando **vm quick-create** de la CLI de Azure, se implementan todos los recursos relacionados con la máquina virtual (incluidos almacenamiento y redes) y la propia máquina virtual. Para más información, consulte [Creación de una máquina virtual Linux en Azure mediante la CLI](../../virtual-machines/linux/quick-create-cli.md).

Del mismo modo, puede utilizar la CLI de Azure para implementar una plantilla de Azure Resource Manager. Para más información, consulte [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../../azure-resource-manager/templates/deploy-cli.md).

### <a name="access-and-security-for-virtual-machines"></a>Acceso y seguridad para máquinas virtuales

Tener acceso a una máquina virtual desde Internet requiere que la interfaz de red asociada, o el equilibrador de carga en su caso, estén configurados con una dirección IP pública. La dirección IP pública incluye un nombre DNS que se resolverá en la máquina virtual o el equilibrador de carga. Para más información, consulte [Direcciones IP en Azure](../../virtual-network/virtual-network-ip-addresses-overview-arm.md).

El acceso a la máquina virtual por la dirección IP pública se administra utilizando un recurso de grupo de seguridad de red (NSG). Un NSG actúa como un firewall y permite o deniega el tráfico a través de la interfaz de red o la subred en un conjunto de puertos definidos. Por ejemplo, para crear una sesión de escritorio remoto con una máquina virtual de Azure, debe configurar el NSG para permitir el tráfico entrante en el puerto 3389. Para más información, consulte [Apertura de puertos para una máquina virtual en Azure mediante Azure Portal](../../virtual-machines/windows/nsg-quickstart-portal.md).

Por último, al igual que con la administración de cualquier sistema informático, debe proporcionar seguridad para una máquina virtual de Azure en el sistema operativo mediante el uso de credenciales de seguridad y firewalls de software.

## <a name="azure-storage"></a>Azure Storage

Azure Storage es un servicio administrado por Microsoft que proporciona almacenamiento redundante, escalable y duradero. Puede agregar una cuenta de Azure Storage como un recurso a cualquier grupo de recursos mediante cualquier método de implementación de recursos. Azure incluye cuatro tipos de almacenamiento: Blob Storage, File Storage, Table Storage y Queue Storage. Al implementar una cuenta de almacenamiento, están disponibles dos tipos de cuenta, de propósito general y de almacenamiento de blobs. Una cuenta de almacenamiento de propósito general proporciona acceso a los cuatro tipos de almacenamiento. Las cuentas de almacenamiento de blobs son similares a las cuentas de uso general, pero contienen blobs especializados que incluyen niveles de acceso activos e inactivos. Para más información sobre el almacenamiento de blobs, consulte [Azure Blob Storage](../../storage/blobs/storage-blob-storage-tiers.md).

Las cuentas de Azure Storage pueden configurarse con distintos niveles de redundancia:

- **Almacenamiento con redundancia local** ofrece una alta disponibilidad al garantizar que se realizan tres copias de todos los datos de forma sincrónica antes de una operación de escritura se considere correcta. Estas copias se almacenan en una única instalación de una única región. Las réplicas residen en dominios de error y dominios de actualización independientes. Esto significa que los datos están disponibles incluso si un nodo de almacenamiento que contiene los datos falla o es puesto en modo sin conexión para ser actualizado.

- **Almacenamiento con redundancia geográfica** realiza tres copias sincrónicas de los datos en la región principal para lograr alta disponibilidad y, a continuación, hace tres réplicas de forma asincrónica en una región emparejada para recuperación ante desastres.

- **Almacenamiento con redundancia geográfica con acceso de lectura** es un almacenamiento con redundancia geográfica más la funcionalidad de leer los datos desde la región secundaria. Esta funcionalidad resulta adecuada para una recuperación ante desastres parcial. Si hay un problema con la región principal, puede cambiar la aplicación y tener acceso de solo lectura a la región emparejada.

### <a name="use-cases"></a>Casos de uso

Cada tipo de almacenamiento tiene un caso de uso diferente.

#### <a name="blob-storage"></a>Blob Storage

La palabra *blob* es un acrónimo de *objeto binario grande*. Los blobs son archivos no estructurados, como los que se almacenan en el equipo. El Almacenamiento de blobs puede almacenar cualquier tipo de datos binarios o texto, como un documento, un archivo multimedia o un instalador de aplicación. El Almacenamiento de blobs a veces se conoce como "almacenamiento de objetos". Azure Blob Storage también contiene los discos de datos de Azure Virtual Machines.

Azure Storage admite tres tipos de blobs:

- Los **Blobs en bloques** se utilizan para contener archivos normales de hasta 195 GB de tamaño (4 MB × 50.000 bloques). El caso de uso principal de los blobs en bloques es el almacenamiento de archivos que se leen desde el principio hasta el final, como los archivos multimedia o los archivos de imagen para sitios web. Se denominan blobs en bloques porque los archivos mayores de 64 MB se deben cargar como bloques pequeños. Estos bloques, a continuación, se consolidan (o confirman) en el blob final.

- Los **Blobs en páginas** se utilizan para almacenar archivos de acceso aleatorio de hasta 1 TB de tamaño. Los blobs en páginas se utilizan principalmente como almacenamiento de copias de seguridad de los discos duros virtuales que proporcionan discos duraderos para Azure Virtual Machines, el servicio de proceso de la IaaS de Azure. Se denominan blobs en páginas porque proporcionan acceso de lectura y escritura aleatoria a páginas de 512 bytes.

- Los **blobs en anexos** consisten en bloques, como los blobs en bloques, pero están optimizados para operaciones de anexión. Con frecuencia, estos se utilizan para registrar información de uno o más orígenes en el mismo blob. Por ejemplo, puede escribir todo el registro de seguimiento en el mismo blob en anexos para una aplicación que se ejecuta en varias máquinas virtuales. Un blob en anexos solo puede tener hasta 195 GB.

Consulte [Introducción a Azure Blob Storage con .NET](../../storage/blobs/storage-dotnet-how-to-use-blobs.md) para más información.

#### <a name="file-storage"></a>File Storage

Azure File Storage es un servicio que ofrece recursos compartidos de archivos en la nube mediante el protocolo estándar Bloque de mensajes del servidor (SMB). El servicio admite SMB 2.1 y SMB 3.0. Con Azure File Storage puede migrar aplicaciones basadas en recursos compartidos de archivos a Azure con rapidez y sin necesidad de costosas reescrituras. Las aplicaciones que se ejecutan en máquinas virtuales de Azure, en servicios en la nube o desde clientes locales pueden montar un recurso compartido de archivos en la nube. Esto es similar a cómo una aplicación de escritorio monta un recurso compartido SMB típico. Cualquier número de componentes de aplicación puede montar y acceder simultáneamente al recurso compartido de Almacenamiento de archivos.

Puesto que un recurso compartido de almacenamiento de archivos es un recurso compartido de archivos SMB estándar, las aplicaciones que se ejecutan en Azure pueden tener acceso a los datos del recurso compartido a través de las API de E/S del sistema de archivos. Por tanto, los desarrolladores pueden aprovechar el código y los conocimientos que ya tienen para migrar las aplicaciones actuales. Los profesionales de TI pueden usar cmdlets de PowerShell para crear, montar y administrar recursos compartidos de almacenamiento de archivos como parte de la administración de las aplicaciones de Azure.

Para más información, vea [Introducción a Azure File Storage en Windows](../../storage/files/storage-how-to-use-files-windows.md) o [Uso de Azure File Storage con Linux](../../storage/files/storage-how-to-use-files-linux.md).

#### <a name="table-storage"></a>Almacenamiento de tablas

Almacenamiento de tablas de Azure es un servicio que almacena datos de NoSQL estructurados en la nube. Table Storage es un almacén de claves y atributos con un diseño sin esquema. Como Table Storage carece de esquema, es fácil adaptar los datos a medida que evolucionan las necesidades de la aplicación. El acceso a los datos es rápido y rentable para todos los tipos de aplicaciones y, además, su coste es muy inferior al del SQL tradicional para volúmenes de datos similares.

Table Storage se puede usar para almacenar conjuntos de datos flexibles, como datos de usuarios para aplicaciones web, libretas de direcciones, información de dispositivos y cualquier otro tipo de metadatos requerido por el servicio. Puede almacenar cualquier número de entidades en una tabla. Una cuenta de almacenamiento puede contener cualquier número de tablas, hasta el límite de capacidad de la cuenta de almacenamiento.

Para más información, consulte [Introducción a Azure Table Storage](../../cosmos-db/table-storage-how-to-use-dotnet.md).

#### <a name="queue-storage"></a>Queue Storage

El almacenamiento en cola de Azure proporciona mensajería en la nube entre componentes de aplicaciones. A la hora de diseñar aplicaciones para escala, los componentes de las mismas suelen desacoplarse para poder escalarlos de forma independiente. El almacenamiento en cola ofrece mensajería asincrónica para la comunicación entre los componentes de las aplicaciones, independientemente de si se ejecutan en la nube, en el escritorio, en un servidor local o en un dispositivo móvil. Además, este tipo de almacenamiento admite la administración de tareas asincrónicas y la creación de flujos de trabajo de procesos.

Para más información, consulte [Introducción a Azure Queue Storage](../../storage/queues/storage-dotnet-how-to-use-queues.md).

### <a name="deploying-a-storage-account"></a>Implementación de una cuenta de almacenamiento

Hay varias opciones para implementar una cuenta de almacenamiento.

#### <a name="portal"></a>Portal

La implementación de una cuenta de almacenamiento mediante Azure Portal requiere solo una suscripción activa de Azure y acceso a un explorador web. Puede implementar una nueva cuenta de almacenamiento en un grupo de recursos nuevo o existente. Después de crear la cuenta de almacenamiento, puede crear un recurso compartido de archivo o contenedor de blobs mediante el portal. Puede crear entidades de Table Storage y de Queue Storage mediante programación. Para obtener más información, consulte [Creación de una cuenta de almacenamiento](../../storage/common/storage-account-create.md).

Además de implementar una cuenta de almacenamiento desde Azure Portal, puede implementar una plantilla de Azure Resource Manager desde el portal. Esto implementará y configurará todos los recursos tal como se define en la plantilla, incluida cualquier cuenta de almacenamiento. Para más información, consulte [Implementación de recursos con plantillas de Resource Manager y Azure Portal](../../azure-resource-manager/templates/deploy-portal.md).

#### <a name="powershell"></a>PowerShell

La implementación de una cuenta de Azure Storage mediante PowerShell permite la automatización de la implementación completa de la cuenta de almacenamiento. Para más información, consulte [Uso de Azure PowerShell con Azure Storage](../../storage/common/storage-powershell-guide-full.md).

Además de implementar individualmente los recursos de Azure, puede usar el módulo Azure PowerShell para implementar una plantilla de Azure Resource Manager. Para más información, consulte [Implementación de recursos con plantillas de Resource Manager y Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md).

#### <a name="command-line-interface-cli"></a>Interfaz de la línea de comandos (CLI)

Al igual que con el módulo de PowerShell, la interfaz de línea de comandos de Azure proporciona automatización de la implementación y puede usarse en sistemas Windows, OS X o Linux. Puede utilizar el comando **storage account create** de la CLI de Azure para crear una cuenta de almacenamiento. Para más información, consulte [Uso de la CLI de Azure con Azure Storage](../../storage/common/storage-azure-cli.md)

Del mismo modo, puede utilizar la CLI de Azure para implementar una plantilla de Azure Resource Manager. Para más información, consulte [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../../resource-group-template-deploy-cli.md).

### <a name="access-and-security-for-azure-storage"></a>Acceso y seguridad en Azure Storage

Hay acceso a Azure Storage de varias maneras, por ejemplo desde Azure Portal durante la creación y operación de máquinas virtuales o desde bibliotecas de cliente de Storage.

#### <a name="virtual-machine-disks"></a>Discos de máquinas virtuales

Si va a implementar una máquina virtual, debe crear una cuenta de almacenamiento para almacenar el disco de sistema operativo de la máquina virtual y los discos de datos adicionales. Puede seleccionar una cuenta de almacenamiento existente o crear una nueva. Dado que el tamaño máximo de un blob es 1.024 GB, un único disco de máquina virtual tiene un tamaño máximo de 1023 GB. Para configurar un disco de datos más grande, puede asignar varios discos de datos a la máquina virtual y agruparlos como un único disco lógico. Para más información, consulte "Administración de discos de Azure" para [Windows](../../virtual-machines/windows/tutorial-manage-data-disk.md) y [Linux](../../virtual-machines/linux/tutorial-manage-disks.md).

#### <a name="storage-tools"></a>Herramientas de Azure Storage

El acceso a las cuentas de Azure Storage es posible a través de varios exploradores de almacenamiento diferentes, como Cloud Explorer de Visual Studio. Estas herramientas le permiten examinar las cuentas de almacenamiento y los datos. Consulte [Herramientas de cliente de Azure Storage](../../storage/common/storage-explorers.md) para ver una lista de exploradores de almacenamiento disponible.

#### <a name="storage-api"></a>API de Storage

Es posible acceder a los recursos de Azure Storage por medio de cualquier lenguaje que pueda realizar solicitudes HTTP/HTTPS. Además, Azure Storage ofrece bibliotecas de programación para varios lenguajes de amplio uso. Estas bibliotecas simplifican el uso de Azure Storage, ya que abordan detalles como la invocación sincrónica y asincrónica, el procesamiento por lotes de las operaciones, la administración de excepciones o los reintentos automáticos. Para más información, consulte [Referencia de la API de REST del servicio Azure Storage](/rest/api/storageservices/Azure-Storage-Services-REST-API-Reference).

#### <a name="storage-access-keys"></a>Claves de acceso a Azure Storage

Cada cuenta de almacenamiento tiene dos claves de autenticación, una principal y una secundaria. Puede utilizar cualquiera de ellas para las operaciones de acceso al almacenamiento. Estas claves de almacenamiento se usan para ayudar a proteger una cuenta de almacenamiento y son necesarias para el acceso mediante programación a los datos. Hay dos claves para permitir la sustitución ocasional de las claves para mejorar la seguridad. Es fundamental mantener las claves seguras porque su posesión, junto con el nombre de cuenta, permite acceso ilimitado a todos los datos de la cuenta de almacenamiento.

#### <a name="shared-access-signatures"></a>Firmas de acceso compartido

Si necesita un acceso controlado de los usuarios a los recursos de almacenamiento, puede crear una firma de acceso compartido. Una firma de acceso compartido es un token que se puede anexar a una dirección URL que permite el acceso delegado a un recurso de almacenamiento. Cualquier usuario que tenga el token puede obtener acceso al recurso correspondiente con los permisos que especifica durante el plazo válido correspondiente. Para más información, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../../storage/common/storage-dotnet-shared-access-signature-part-1.md).

## <a name="azure-virtual-network"></a>Azure Virtual Network

Las redes virtuales son necesarias para las comunicaciones entre máquinas virtuales. Puede definir subredes, direcciones IP personalizadas, configuración de DNS, filtrado de seguridad y equilibrio de carga. Azure es compatible con distintos casos de uso: redes solo de nube o redes virtuales híbridas.

### <a name="cloud-only-virtual-networks"></a>Redes virtual solo en la nube

Una red virtual de Azure, de forma predeterminada, es accesible solo para los recursos almacenados en Azure. Los recursos conectados a la misma red virtual pueden comunicarse entre sí. Puede asociar las interfaces de red de la máquina virtual y los equilibradores de carga con una dirección IP pública para que la máquina virtual sea accesible a través de Internet. Puede ayudar a proteger el acceso a los recursos expuestos públicamente mediante el uso de un grupo de seguridad de red.

![Azure Virtual Network para una aplicación web de nivel 2](https://docs.microsoft.com/azure/load-balancer/media/load-balancer-internal-overview/ic744147.png)

### <a name="hybrid-virtual-networks"></a>Redes virtuales híbridas

Puede conectar una red local con una red virtual de Azure con una conexión VPN de sitio a sitio o mediante ExpressRoute. En esta configuración, la red virtual de Azure es esencialmente una extensión basada en la nube de su red local.

Dado que la red virtual de Azure está conectada a la red local, las redes virtuales con entorno local deben utilizar una parte única del espacio de direcciones que usa la organización. De la misma manera que diferentes ubicaciones de la empresa se asignan a una subred IP específica, Azure se convierte en otra ubicación cuando la red se amplía.
Hay varias opciones para implementar una red virtual.

- [Portal](../..//virtual-network/quick-create-portal.md)

- [PowerShell](../../virtual-network/quick-create-powershell.md)

- [Interfaz de la línea de comandos (CLI)](../../virtual-network/quick-create-cli.md)

- Plantillas del Administrador de recursos de Azure

> **Cuándo se debe usar**: siempre que trabaje con máquinas virtuales en Azure, trabajará con redes virtuales. Esto permite segmentar las máquinas virtuales en subredes públicas y privadas de forma similar a los centros de datos en el entorno local.
>
> **Introducción**: La implementación de una red virtual de Azure mediante Azure Portal requiere solo una suscripción activa de Azure y acceso a un explorador web. Puede implementar una nueva red virtual en un grupo de recursos nuevo o existente. Al crear una nueva máquina virtual desde el portal, puede seleccionar una red virtual existente o crear una nueva. Introducción y [Creación de una red virtual mediante Azure Portal](../../virtual-network/quick-create-portal.md).

### <a name="access-and-security-for-virtual-networks"></a>Acceso y seguridad para redes virtuales

Puede ayudar a proteger las redes virtuales Azure mediante el uso de un grupo de seguridad de red. Los NSG contienen reglas de la lista de control de acceso (ACL) que permiten o deniegan el tráfico de red a las instancias de máquina virtual en una red virtual. Los NSG se pueden asociar con subredes o con instancias de una máquina virtual individual dentro de esa subred. Cuando un NSG está asociado a una subred, las reglas de la ACL se aplican a todas las instancias de máquinas virtuales de esa subred. Además, puede restringir más el tráfico a una máquina virtual individual asociando un NSG directamente a esa maquina virtual. Para más información, consulte [Filtrado del tráfico de red con grupos de seguridad de red](../../virtual-network/security-overview.md).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una máquina virtual Windows](../../virtual-machines/windows/quick-create-portal.md)
- [Creación de una máquina virtual Linux](../../virtual-machines/linux/quick-create-portal.md)
