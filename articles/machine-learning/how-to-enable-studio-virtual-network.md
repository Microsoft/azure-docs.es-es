---
title: Habilitación de Azure Machine Learning Studio en una red virtual
titleSuffix: Azure Machine Learning
description: Use Azure Machine Learning Studio para tener acceso a los datos almacenados en una red virtual.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 07/16/2020
ms.custom: contperfq4, tracking-python
ms.openlocfilehash: 5dce7cde3c46fbcf3f764819f730f42cace4a74c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90897533"
---
# <a name="use-azure-machine-learning-studio-in-an-azure-virtual-network"></a>Habilitación de Azure Machine Learning Studio en una Azure Virtual Network

En este artículo, aprenderá a usar Azure Machine Learning Studio en una red virtual. Aprenderá a:

> [!div class="checklist"]
> - Obtener acceso a Studio desde un recurso dentro de una red virtual.
> - Proporcionar a Studio acceso a los datos almacenados dentro de una red virtual.
> - Comprender cómo Studio afecta a la seguridad del almacenamiento.

Este artículo es la quinta parte de una serie de cinco capítulos que le guía a través de la protección de un flujo de trabajo de Azure Machine Learning. Le recomendamos encarecidamente que lea [Parte uno: Introducción a las redes virtuales](how-to-network-security-overview.md) para comprender la arquitectura general en primer lugar. 

Consulte los demás artículos de esta serie:

[1. Introducción a las redes virtuales](how-to-network-security-overview.md) > [2. Protección del área de trabajo ](how-to-secure-workspace-vnet.md) > [3. Protección del entorno de entrenamiento](how-to-secure-training-vnet.md) > [4. Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md) > [5. Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)


> [!IMPORTANT]
> Aunque la mayor parte de Studio funciona con datos almacenados en una red virtual, __no__ es así en el caso de los cuadernos integrados. Los notebooks integrados no admiten el uso de almacenamiento que se encuentra en una red virtual. En su lugar, puede usar cuadernos de Jupyter Notebook en una instancia de Compute. Para obtener más información, consulte la sección [Acceso a los datos en un cuaderno de instancia de Compute]().


## <a name="prerequisites"></a>Requisitos previos

+ Lea el artículo [Introducción a la seguridad de red](how-to-network-security-overview.md) para comprender los escenarios comunes de redes virtuales y la arquitectura de red virtual general.

+ Una red virtual y una subred preexistentes que se usarán.

+ Un [área de trabajo existente de Azure Machine Learning con Private Link habilitado](how-to-secure-workspace-vnet.md#secure-the-workspace-with-private-endpoint).

+ Una [cuenta de Azure Storage existente agregada a la red virtual](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts).

## <a name="access-the-studio-from-a-resource-inside-the-vnet"></a>Acceso a Studio desde un recurso dentro de una red virtual

Si accede a Studio desde un recurso dentro de una red virtual (por ejemplo, una instancia de proceso o una máquina virtual), tendrá que permitir el tráfico de salida desde la red virtual a Studio. 

Por ejemplo, si usa grupos de seguridad de red (NSG) para restringir el tráfico de salida, agregue una regla a un destino de __etiqueta de servicio__ de __AzureFrontDoor.Frontend__.

## <a name="access-data-using-the-studio"></a>Acceso a los datos mediante Studio

Si los datos están almacenados en una red virtual, debe configurar las cuentas de almacenamiento para usar una [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) para conceder a Studio acceso a los datos.


Si no habilita la identidad administrada, recibirá este error: `Error: Unable to profile this dataset. This might be because your data is stored behind a virtual network or your data does not support profile.`. Además, se deshabilitarán las siguientes operaciones:

* Vista previa de los datos en Studio.
* Visualización de los datos en el diseñador.
* Envío de un experimento de AutoML.
* Inicio de un proyecto de etiquetado.

Studio admite la lectura de datos de los siguientes tipos de almacén de datos en una red virtual:

* Blob de Azure
* Azure Data Lake Storage Gen1
* Azure Data Lake Storage Gen2
* Azure SQL Database


### <a name="configure-datastores-to-use-managed-identity"></a>Configuración de almacenes de datos para usar identidad administrada

Azure Machine Learning usa [almacenes de datos](concept-data.md#datastores) para conectarse a las cuentas de almacenamiento. Siga los pasos que se indican a continuación para configurar los almacenes de datos para que usen identidad administrada. 

1. En Studio, seleccione __Almacenes de datos__.

1. Para crear un almacén de datos, seleccione __+ Nuevo almacén de datos__.

    Para actualizar un almacén de datos existente, seleccione el almacén de datos y después __Actualizar credenciales__.

1. En la configuración del almacén de datos, seleccione __Sí__ para __Permitir que Azure Machine Learning Service acceda al almacenamiento mediante la identidad administrada del área de trabajo__.


En estos pasos se agrega la identidad administrada del área de trabajo como __Lector__ al servicio de almacenamiento mediante el control de acceso basado en recursos (RBAC) de Azure. El acceso __Lector__ permite que el área de trabajo recupere la configuración del firewall y se asegura de que los datos no salgan de la red virtual.

> [!NOTE]
> Estos cambios pueden tardar hasta 10 minutos en surtir efecto.

## <a name="technical-notes-for-managed-identity"></a>Notas técnicas de identidad administrada

El uso de identidad administrada para tener acceso a los servicios de almacenamiento afecta a algunas consideraciones de seguridad. Estas consideraciones son únicas en cuanto al tipo de cuenta de almacenamiento al que se obtiene acceso. En esta sección se describen los cambios para cada tipo de cuenta de almacenamiento.

### <a name="azure-blob-storage"></a>Azure Blob Storage

Para __Azure Blob Storage__, la identidad administrada del área de trabajo también se agrega como un [lector de datos de blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader) para que pueda leer datos del almacenamiento de blobs.

### <a name="azure-data-lake-storage-gen2-access-control"></a>Control de acceso de Azure Data Lake Storage Gen2

Puede usar listas de control de acceso (ACL) de tipo RBAC y POSIX para controlar el acceso a los datos dentro de una red virtual.

Para usar RBAC, agregue la identidad administrada del área de trabajo al rol [Lector de datos de blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Para más información, consulte [Control de acceso basado en rol](../storage/blobs/data-lake-storage-access-control.md#role-based-access-control).

Para usar ACL, se puede asignar el acceso de la identidad administrada del área de trabajo como cualquier otra entidad de seguridad. Para obtener más información, vea [Listas de control de acceso en archivos y directorios](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories).

### <a name="azure-data-lake-storage-gen1-access-control"></a>Control de acceso de Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 solo admite listas de control de acceso de estilo POSIX. Puede asignar el acceso de la identidad administrada del área de trabajo a los recursos como cualquier otra entidad de seguridad. Para obtener más información, vea [Control de acceso en Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-access-control.md).

### <a name="azure-sql-database-contained-user"></a>Usuario independiente de Azure SQL Database

Para acceder a los datos almacenados en una instancia de Azure SQL Database mediante la identidad administrada, debe crear un usuario independiente de SQL que se asigne a la identidad administrada. Para obtener más información sobre cómo crear un usuario desde un proveedor externo, vea [Creación de usuarios independientes asignados a identidades de Azure AD](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities).

Después de crear un usuario independiente de SQL, utilice el [comando GRANT de T-SQL](https://docs.microsoft.com/sql/t-sql/statements/grant-object-permissions-transact-sql) para concederle permisos.

### <a name="azure-machine-learning-designer-default-datastore"></a>Almacén de datos predeterminado del diseñador de Azure Machine Learning

El diseñador usa la cuenta de almacenamiento asociada al área de trabajo para almacenar la salida de forma predeterminada. Sin embargo, puede especificarla para almacenar la salida en cualquier almacén de información al que tenga acceso. Si su entorno usa redes virtuales, puede usar estos controles para asegurarse de que los datos se mantienen seguros y son accesibles.

Para configurar un nuevo almacenamiento predeterminado para una canalización:

1. En un borrador de canalización, seleccione el **icono de engranaje de la configuración** junto al título de la canalización.
1. Seleccione la opción **Seleccionar almacén de datos predeterminado**.
1. Especifique un nuevo almacén de datos.

También puede invalidar el almacén de datos predeterminado por cada módulo. Esto le ofrece control sobre la ubicación de almacenamiento de cada módulo individual.

1. Seleccione el módulo cuya salida quiere especificar.
1. Expanda la sección **Configuración de salida**.
1. Seleccione la opción **Override default output settings** (Invalidar configuración de salida predeterminada).
1. Seleccione **Set output settings** (Establecer configuración de salida).
1. Especifique un nuevo almacén de datos.

## <a name="next-steps"></a>Pasos siguientes

Este artículo es una parte opcional de una serie de cuatro capítulos sobre redes virtuales. Vea el resto de los artículos para obtener información sobre cómo proteger una red virtual:

* [Parte 1: Introducción a las redes virtuales](how-to-network-security-overview.md)
* [Parte 2: Protección de los recursos de un área de trabajo](how-to-secure-workspace-vnet.md)
* [Parte 3: Protección del entorno de entrenamiento](how-to-secure-training-vnet.md)
* [Parte 4: Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md)