---
title: Protección de datos del cliente en Azure
description: Obtenga información sobre cómo Azure protege los datos de clientes mediante la segregación, redundancia y destrucción de datos.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/10/2020
ms.author: terrylan
ms.openlocfilehash: 14589e4efe22d89468b069bf6ff7e3d9babcc714
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543799"
---
# <a name="azure-customer-data-protection"></a>Protección de datos de cliente de Azure   
El personal de operaciones y soporte técnico de Microsoft no tiene libre acceso a los datos de los clientes, ya que se les deniega de forma predeterminada. Cuando se concede el acceso a los datos relacionados con un caso de soporte técnico, solo se concede mediante un modelo Just-in-Time (JIT) usando directivas que se auditan y aprueban en función de las directivas de cumplimiento y privacidad.  Los requisitos de control de acceso se establecen mediante la directiva de seguridad de Azure siguiente:

- No hay acceso a los datos de clientes de manera predeterminada.
- Ninguna cuenta de usuario o administrador en las máquinas virtuales de cliente.
- Concesión del privilegio mínimo necesario para completar la tarea; auditoría y registro de las solicitudes de acceso.

Azure asigna al personal de soporte técnico cuentas de Active Directory corporativa únicas. Azure se basa en Microsoft Corporate Active Directory, administrado por Microsoft Information Technology (MSIT), para controlar el acceso a los sistemas de información clave. Se requiere la autenticación multifactor y solo se concede el acceso desde consolas seguras.

Todos los intentos de acceso se supervisan y se pueden mostrar a través de un conjunto básico de informes.

## <a name="data-protection"></a>Protección de los datos
Azure proporciona a los clientes una fuerte seguridad de los datos, tanto de manera predeterminada como opcional del cliente.

**Segregación de datos**: Azure es un servicio multiinquilino, lo que significa que las implementaciones y máquinas virtuales de varios clientes se almacenan en el mismo hardware físico. Azure usa un aislamiento lógico para separar los datos de cada cliente de los del resto. La segregación ofrece las ventajas de escala económicas de los servicios multiinquilino, a la vez que evita rigurosamente que los clientes tengan acceso a los datos de otros.

**Protección de datos en reposo**: los clientes son responsables de garantizar que los datos almacenados en Azure se cifren según sus propios estándares. Azure facilita una amplia gama de funcionalidades de cifrado, por lo que ofrece a los clientes la flexibilidad de poder elegir la solución que mejor se ajusta a sus necesidades. Azure Key Vault le permite a los clientes mantener de forma fácil el control de las claves que usan los servicios y las aplicaciones en la nube para cifrar datos. Azure Disk Encryption permite a los clientes cifrar máquinas virtuales. El cifrado del servicio Azure Storage permite cifrar todos los datos que se colocan en la cuenta de almacenamiento de un cliente.

**Protección de datos en tránsito**: Microsoft ofrece una serie de opciones que los clientes pueden usar para proteger los datos en tránsito internamente, dentro de la red de Azure, y externamente, a través de Internet, dirigidos al usuario final.  Entre ellas se incluyen la comunicación a través de redes privadas virtuales (mediante el cifrado IPsec/IKE), Seguridad de la capa de transporte (TLS) 1.2 o posterior (a través de componentes de Azure, como Application Gateway o Azure Front Door), protocolos directamente en las máquinas virtuales de Azure (como Windows IPsec o SMB) y mucho más. 

Además, el "cifrado de forma predeterminada" con MACsec (un estándar de IEEE en la capa de enlace de datos) está habilitado para todo el tráfico de Azure que viaja entre los centros de datos de Azure con el fin de garantizar la confidencialidad y la integridad de los datos de los clientes. 

**Data redundancy** (Redundancia de datos): Microsoft ayuda a garantizar que los datos están protegidos si se produce un ciberataque o daños físicos en un centro de datos. Los clientes pueden optar por:

- Consideraciones sobre el almacenamiento en el país o región por motivos de cumplimiento o latencia.
- Almacenamiento fuera del país o la región por motivos de seguridad o de recuperación ante desastres.

Los datos pueden replicarse en un área geográfica seleccionada para redundancia, pero no se transmitirán fuera de ella. Los clientes tienen varias opciones para la replicación de datos, incluido el número de copias y el número y la ubicación de los centros de datos de replicación.

Al crear la cuenta de almacenamiento, seleccione una de las siguientes opciones de replicación:

- **Almacenamiento con redundancia local (LRS)** :  El almacenamiento con redundancia local mantiene tres copias de sus datos. LRS se replica tres veces dentro de una única instalación de una sola región. LRS protege los datos frente a errores comunes del hardware, pero no frente a errores de una única instalación.
- **Almacenamiento con redundancia de zona (ZRS)** :  El almacenamiento con redundancia de zona mantiene tres copias de los datos. ZRS se replica tres veces entre dos o tres instalaciones para proporcionar mayor durabilidad que LRS. La replicación se produce en una única región o entre dos regiones. ZRS ayuda a garantizar la durabilidad de sus datos dentro de una sola región.
- **Almacenamiento con redundancia geográfica (GRS)** : El almacenamiento con redundancia geográfica está habilitado para su cuenta de almacenamiento de manera predeterminada cuando la crea. GRS mantiene seis copias de sus datos. Con GRS, los datos se replican tres veces dentro de la región primaria. Los datos se replican también tres veces en una región secundaria a cientos de kilómetros de distancia de la región primaria, lo que proporciona el nivel más alto de durabilidad. En caso de que se produzca un error en la región primaria, Azure Storage conmuta por error a la región secundaria. GRS ayuda a garantizar la durabilidad de sus datos en dos regiones distintas.

**Destrucción de datos**: cuando los clientes eliminan datos o abandonan Azure, Microsoft sigue estándares estrictos para sobrescribir los recursos de almacenamiento antes de reutilizarlos, así como la destrucción física del hardware retirado. Microsoft ejecuta una eliminación completa de los datos a solicitud del cliente y a la finalización del contrato.

## <a name="customer-data-ownership"></a>Propiedad de los datos del cliente
Microsoft no inspecciona, aprueba ni supervisa las aplicaciones que los clientes implementan en Azure. Además, Microsoft no conoce qué tipo de datos los clientes eligen almacenar en Azure. Microsoft no reclama la propiedad de los datos a través de la información del cliente escrita en Azure.

## <a name="records-management"></a>Administración de registros
Azure ha establecido requisitos internos de retención de registros para los datos de back-end. Los clientes son responsables de identificar sus propios requisitos de retención de registros. Para los registros almacenados en Azure, el cliente es responsable de extraer sus datos y conservar el contenido fuera de Azure durante un período de retención especificado por el cliente.

Azure permite a los clientes exportar datos e informes de auditoría desde el producto. Las exportaciones se guardan localmente para conservar la información durante un período de retención definido por el cliente.

## <a name="electronic-discovery-e-discovery"></a>Detección electrónica (eDiscovery)
Los clientes de Azure son responsables de cumplir con requisitos de eDiscovery en su uso de los servicios de Azure. Si los clientes de Azure deben conservar sus datos de cliente, los datos se pueden exportar y guardar localmente. Además, los clientes pueden solicitar las exportaciones de los datos al departamento de Asistencia al cliente de Azure. Además de permitir a los clientes exportar sus datos, Azure lleva a cabo tareas internas exhaustivas de registro y supervisión.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre lo que hace Microsoft para proteger la infraestructura de Azure, consulte:

- [Instalaciones de Azure, entornos locales y seguridad física](physical-security.md)
- [Disponibilidad de la infraestructura de Azure](infrastructure-availability.md)
- [Componentes y límites del sistema de información de Azure](infrastructure-components.md)
- [Arquitectura de red de Azure](infrastructure-network.md)
- [Red de producción de Azure](production-network.md)
- [Características de seguridad de Azure SQL Database](infrastructure-sql.md)
- [Operaciones de producción y administración de Azure](infrastructure-operations.md)
- [Supervisión de la infraestructura de Azure](infrastructure-monitoring.md)
- [Integridad de la infraestructura de Azure](infrastructure-integrity.md)
