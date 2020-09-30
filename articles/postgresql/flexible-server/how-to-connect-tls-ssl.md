---
title: 'Conectividad cifrada mediante TLS/SSL en Azure Database for PostgreSQL: Servidor flexible'
description: 'Instrucciones e información sobre cómo conectarse mediante TSL/SSL en Azure Database for PostgreSQL: Servidor flexible.'
author: ambhatna
ms.author: ambhatna
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 0ba04ff14e62cd411515a765dc95ef7a3e72b709
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932189"
---
# <a name="encrypted-connectivity-using-transport-layer-security-in-azure-database-for-postgresql---flexible-server"></a>Conectividad cifrada con Seguridad de la capa de transporte en Azure Database for PostgreSQL: Servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible en versión preliminar

Azure Database for PostgreSQL: Servidor flexible admite la conexión de las aplicaciones cliente al servicio PostgreSQL mediante Seguridad de la capa de transporte (TLS), que anteriormente se denominaba Capa de sockets seguros (SSL). TLS es un protocolo estándar del sector que garantiza conexiones de red cifradas entre el servidor de bases de datos y las aplicaciones cliente, lo que le permite ajustarse a los requisitos de cumplimiento.

Azure Database for PostgreSQL: Servidor flexible admite conexiones cifradas mediante la Seguridad de la capa de transporte (TLS 1.2+) y se denegarán todas las conexiones entrantes con TLS 1.0 y TLS 1.1. Para todos los servidores flexibles, la aplicación de las conexiones TLS está habilitada y no se puede deshabilitar TLS/SSL para conectarse a un servidor flexible.

## <a name="applications-that-require-certificate-verification-for-tlsssl-connectivity"></a>Aplicaciones que requieren la verificación de certificados para la conectividad TLS/SSL
En algunos casos, las aplicaciones requieren un archivo de certificado local generado a partir de un archivo de certificado de una entidad de certificación (CA) de confianza para conectarse de forma segura. Azure Database for PostgreSQL: Servidor flexible usa *DigiCert Global Root CA*. Descargue este certificado necesario para comunicarse a través de SSL desde [DigiCert Global Root CA](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) y guarde el archivo de certificado en su ubicación preferida. Por ejemplo, en este tutorial se usa `c:\ssl`.


### <a name="connect-using-psql"></a>Conexión mediante psql
Si ha creado el servidor flexible con *acceso privado (Integración con VNet)* , deberá conectarse a su servidor desde un recurso en la misma red virtual que el servidor. Puede crear una máquina virtual y agregarla a la red virtual creada con el servidor flexible.

Si ha creado el servidor flexible con *acceso público (direcciones IP permitidas)* , puede agregar la dirección IP local a la lista de reglas de firewall del servidor.

En el ejemplo siguiente se muestra cómo conectarse al servidor mediante la interfaz de la línea de comandos psql. Use la configuración de la cadena de conexión `sslmode=verify-full` para aplicar la comprobación del certificado TLS/SSL. Pase la ruta de acceso al archivo del certificado local al parámetro `sslrootcert`.

```bash
 psql "sslmode=verify-full sslrootcert=c:\ssl\DigiCertGlobalRootCA.crt.pem host=mydemoserver.postgres.database.azure.com dbname=postgres user=myadmin"
```
> [!Note]
> Confirme que el valor pasado a `sslrootcert` coincide con la ruta de acceso al archivo del certificado que guardó.

## <a name="ensure-your-application-or-framework-supports-tls-connections"></a>Comprobación de que la aplicación o el marco de trabajo admiten conexiones TLS

Algunos marcos de trabajo de aplicaciones que usan PostgreSQL en los servicios de bases de datos no habilitan TLS de manera predeterminada durante la instalación. El servidor PostgreSQL establece conexiones TLS, pero si la aplicación no está configurada para dicho protocolo, esta puede presentar un error al conectarse al servidor de bases de datos. Consulte la documentación de la aplicación para aprender a habilitar las conexiones TLS.

## <a name="next-steps"></a>Pasos siguientes
- [Creación y administración de redes virtuales en Azure Database for PostgreSQL: Servidor flexible mediante la CLI de Azure](./how-to-manage-virtual-network-cli.md).
- Obtenga más información acerca de las [redes en Azure Database for PostgreSQL: Servidor flexible](./concepts-networking.md).
- Información más detallada sobre las [Reglas de firewall de Azure Database for PostgreSQL: Servidor flexible](./concepts-networking.md#public-access-allowed-ip-addresses).
