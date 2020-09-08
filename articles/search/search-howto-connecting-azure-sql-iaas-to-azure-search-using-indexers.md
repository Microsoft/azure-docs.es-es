---
title: Conexión de la VM de SQL de Azure para la indexación de búsqueda
titleSuffix: Azure Cognitive Search
description: Habilite conexiones cifradas y configure el firewall para permitir conexiones a SQL Server en una máquina virtual de Azure a partir de un indexador de Búsqueda cognitiva de Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/12/2020
ms.openlocfilehash: ec1e74c6a029ab0f8defc3ae783c9e974f387289
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88922980"
---
# <a name="configure-a-connection-from-an-azure-cognitive-search-indexer-to-sql-server-on-an-azure-vm"></a>Configuración de una conexión desde un indexador de Búsqueda cognitiva de Azure a SQL Server en una máquina virtual de Azure

Como se indicó en [Conexión de Azure SQL Database a Búsqueda cognitiva de Azure mediante indexadores](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md#faq), la creación de indexadores en **SQL Server on Azure VMs** (o **SQL Azure VMs** para abreviar) es compatible con Búsqueda cognitiva de Azure, pero hay varios requisitos previos relacionados con la seguridad de los que hay que ocuparse en primer lugar. 

La conexión de Búsqueda cognitiva de Azure a SQL Server en una máquina virtual es una conexión pública de Internet. Todas las medidas de seguridad que seguiría normalmente para este tipo de conexiones son aplicables aquí también:

+ Obtenga un certificado de una [entidad de certificación](https://en.wikipedia.org/wiki/Certificate_authority#Providers) para el nombre de dominio completo de la instancia de SQL Server de la máquina virtual de Azure.
+ Instale el certificado en la máquina virtual y, a continuación, habilite y configure las conexiones cifradas en la máquina virtual siguiendo las instrucciones de este artículo.

## <a name="enable-encrypted-connections"></a>Habilitación de conexiones cifradas
Búsqueda cognitiva de Azure requiere un canal cifrado para todas las solicitudes del indexador a través una conexión pública a Internet. En esta sección se enumeran los pasos necesarios para realizar este trabajo.

1. En las propiedades del certificado compruebe que el nombre de sujeto es el nombre de dominio completo (FQDN) de la máquina virtual de Azure. Para ver las propiedades, puede utilizar una herramienta como CertUtils o el complemento Certificados. El FQDN se puede obtener de la sección Essentials de la hoja del servicio VM, en el campo **Etiqueta de dirección IP pública/nombre de DNS** de [Azure Portal](https://portal.azure.com/).
   
   * En el caso de las máquinas virtuales creadas mediante la plantilla de **Resource Manager** más reciente, el nombre de dominio completo tiene el formato `<your-VM-name>.<region>.cloudapp.azure.com`.
   * En el caso de las VM anteriores creadas como una VM **clásica**, el FQDN tiene el formato `<your-cloud-service-name.cloudapp.net>`.

2. Configure SQL Server para que use el certificado mediante el Editor del registro (regedit). 
   
    Aunque el Administrador de configuración de SQL Server se usa a menudo para esta tarea, no se puede utilizar para este escenario. No encontrará el certificado importado, ya que el FQDN de la VM de Azure no coincide con el FQDN que determina la VM (identifica el dominio como el equipo local o el dominio de red al que está conectado). Si los nombres no coinciden, utilice regedit para especificar el certificado.
   
   * En regedit, vaya a esta clave del Registro: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\[MSSQL13.MSSQLSERVER]\MSSQLServer\SuperSocketNetLib\Certificate`.
     
     La parte `[MSSQL13.MSSQLSERVER]` varía en función de la versión y del nombre de la instancia. 
   * Establezca el valor de la clave del **Certificado** en la **huella digital** del certificado TLS/SSL que importó a la VM.
     
     Hay varias maneras de obtener la huella digital, y algunas de ellas son mejores que otras. Si la copia desde el complemento **Certificados** de MMC, es probable que elija un carácter inicial invisible [como se describe en este artículo](https://support.microsoft.com/kb/2023869/), lo que provoca un error al intentar establecer una conexión. Existen varias soluciones para corregir este problema. La más fácil es borrarlo y, luego, volver a escribir el primer carácter de la huella digital para quitar el carácter inicial del campo de valor de clave en regedit. Como alternativa, se puede utilizar otra herramienta para copiar la huella digital.

3. Otorgue permisos a la cuenta de servicio. 
   
    Asegúrese de que a la cuenta de servicio de SQL Server se le concede el permiso adecuado en la clave privada del certificado TLS/SSL. Si pasa por alto este paso, SQL Server no se iniciará. Puede usar el complemento **Certificados** o **CertUtils** para esta tarea.
    
4. Reinicie el servicio SQL Server.

## <a name="configure-sql-server-connectivity-in-the-vm"></a>Configuración de la conectividad de SQL Server en la máquina virtual
Después de configurar la conexión cifrada requerida por Búsqueda cognitiva de Azure, existen pasos adicionales de configuración intrínsecos a SQL Server en las máquinas virtuales de Azure. Si aún no lo ha hecho, el paso siguiente es finalizar la configuración mediante cualquiera de estos artículos:

* En el caso de una máquina virtual de **Resource Manager** , consulte [Conexión a una máquina virtual de SQL Server en Azure (Resource Manager)](../azure-sql/virtual-machines/windows/ways-to-connect-to-sql.md). 
* En el caso de una máquina virtual **clásica** , consulte [Conexión a una máquina virtual de SQL Server en Azure (implementación clásica)](/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-sql-connect).

En concreto, consulte en ambos artículos la sección dedicada a la "conexión a través de Internet".

## <a name="configure-the-network-security-group-nsg"></a>Configuración del grupo de seguridad de red (NSG)
No es extraño configurar el NSG y el correspondiente punto de conexión o lista de control de acceso (ACL) de Azure para que se pueda acceder a la máquina virtual de Azure desde otras partes. Lo más probable es que ya haya realizado esta operación para la lógica de su aplicación se conecte a la máquina virtual de SQL Azure. Esto es igual para una conexión de Búsqueda cognitiva de Azure a la máquina virtual de SQL Azure. 

Los vínculos siguientes proporcionan instrucciones para la configuración de NSG en las implementaciones de VM. Siga estas instrucciones para incluir en la ACL un punto de conexión de Búsqueda cognitiva de Azure por su dirección IP.

> [!NOTE]
> Para más información, consulte [¿Qué es un grupo de seguridad de red?](../virtual-network/security-overview.md)
> 
> 

* En el caso de una máquina virtual de **Resource Manager** , consulte [Administración de grupos de seguridad de red con Azure Portal](../virtual-network/tutorial-filter-network-traffic.md). 
* En el caso de una máquina virtual **clásica** , consulte [Creación de grupos de seguridad de red (clásicos) en PowerShell](/previous-versions/azure/virtual-network/virtual-networks-create-nsg-classic-ps).

La dirección IP puede plantear ciertos problemas, que se solucionan fácilmente si se conoce el problema y las posibles soluciones. En las restantes secciones encontrará recomendaciones para el control de los problemas relacionados con las direcciones IP de la ACL.

#### <a name="restrict-access-to-the-azure-cognitive-search"></a>Restricción del acceso a Azure Cognitive Search
Se recomienda encarecidamente restringir el acceso a la dirección IP del servicio de búsqueda y el rango de direcciones IP de la [etiqueta de servicio](../virtual-network/service-tags-overview.md#available-service-tags) `AzureCognitiveSearch` en la ACL en lugar de abrir las máquinas virtuales de SQL Azure a todas las solicitudes de conexión.

Puede averiguar la dirección IP haciendo ping en el FQDN (por ejemplo, `<your-search-service-name>.search.windows.net`) del servicio de búsqueda.

Puede averiguar el intervalo de direcciones IP de la [etiqueta de servicio](../virtual-network/service-tags-overview.md#available-service-tags) `AzureCognitiveSearch` mediante el uso de [archivos JSON descargables](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) o a través de la [API de detección de etiquetas de servicio](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview). El intervalo de direcciones IP se actualiza semanalmente.

#### <a name="managing-ip-address-fluctuations"></a>Administración de las fluctuaciones de dirección IP
Si el servicio de búsqueda tiene solo una unidad de búsqueda (es decir, una réplica y una partición), la dirección IP cambiará durante los reinicios rutinarios, lo que invalida una ACL existente con la dirección IP de su servicio de búsqueda.

Una forma de evitar el posterior error de conectividad es usar más de una réplica y una partición en Búsqueda cognitiva de Azure. Esto aumenta el costo, pero también soluciona el problema de las direcciones IP. En Búsqueda cognitiva de Azure, las direcciones IP no cambian cuando hay más de una unidad de búsqueda.

Un segundo enfoque es permitir que la conexión genere un error y, después, volver a configurar las ACL en el NSG. Como promedio, se puede esperar que las direcciones IP cambien cada pocas semanas. Para los clientes que realizan una indexación controlado muy de tarde en tarde, este enfoque puede ser viable.

Un tercer enfoque viable (aunque no especialmente seguro) es especificar el intervalo de direcciones IP de la región de Azure en la que se aprovisiona el servicio de búsqueda. La lista de intervalos IP desde la que se asignan direcciones IP públicas a recursos de Azure está publicada en [Intervalos IP del centro de datos de Azure](https://www.microsoft.com/download/details.aspx?id=41653). 

#### <a name="include-the-azure-cognitive-search-portal-ip-addresses"></a>Inclusión de las direcciones IP del portal de Búsqueda cognitiva de Azure
Si utiliza Azure Portal para crear un indexador, la lógica del portal de Búsqueda cognitiva de Azure también necesitará acceso a la máquina virtual de SQL Azure durante el tiempo de creación. Las direcciones IP del portal de Búsqueda cognitiva de Azure se pueden encontrar haciendo ping en `stamp2.search.ext.azure.com`.

## <a name="next-steps"></a>Pasos siguientes
Dejando a un lado la configuración, ya puede especificar un servicio SQL Server en la máquina virtual de Azure como origen de los datos de un indexador de Búsqueda cognitiva de Azure. Consulte [Conexión de Azure SQL Database a Búsqueda cognitiva de Azure mediante indexadores](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md) para más información.