---
title: 'Capítulo 2: Instalación y uso del agente de SNMP de Azure RTOS NetX Duo'
description: Este capítulo contiene una descripción de varios problemas relacionados con la instalación, la configuración y el uso del componente de agente de SNMP de NetX Duo.
author: philmea
ms.author: philmea
ms.date: 06/04/2020
ms.topic: article
ms.service: rtos
ms.openlocfilehash: 6e18906b6356bd8ff4efdc1ab0f2809d75493ad027c3d3e27e0536ee4b80f43b
ms.sourcegitcommit: 93d716cf7e3d735b18246d659ec9ec7f82c336de
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2021
ms.locfileid: "116798289"
---
# <a name="chapter-2---installation-and-use-of-the-azure-rtos-netx-duo-snmp-agent"></a>Capítulo 2: Instalación y uso del agente de SNMP de Azure RTOS NetX Duo

Este capítulo contiene una descripción de varios problemas relacionados con la instalación, la configuración y el uso del componente de agente de SNMP de Azure RTOS NetX Duo.

## <a name="product-distribution"></a>Distribución del producto

El agente de SNMP para NetX Duo está disponible en [https://github.com/azure-rtos/netxduo](https://github.com/azure-rtos/netxduo). El paquete incluye cuatro archivos de código fuente, un archivo de inclusión y un archivo PDF que contiene este documento, como se indica a continuación:

- **nxd_snmp.h**: archivo de encabezado para SNMP para NetX Duo
- **demo_snmp_helper.h**: archivo de encabezado para datos MIB de SNMP
- **nxd_snmp.c**: archivo de código fuente de C para el agente de SNMP para NetX Duo
- **nx_md5.c**: algoritmos hash MD5
- **nx_sha.c**: algoritmos hash SHA
- **nx_des.c**: algoritmos hash DES
- **nxd_snmp.pdf**: guía del usuario para el agente de SNMP para NetX Duo
- **demo_netxduo_snmp.c**: demostración simple de SNMP
- **demo_netxduo_mib2.c**: demostración simple de MIB2 (MIB tiene elementos de dirección IPv6).
- **demo_snmp_helper.h**: archivo de encabezado que define elementos MIB.

## <a name="netx-duo-snmp-agent-installation"></a>Instalación del agente de SNMP de NetX Duo

Para usar el protocolo SNMP de NetX Duo, la distribución completa que se ha mencionado anteriormente debe copiarse en el mismo directorio en el que está instalado NetX Duo. Por ejemplo, si NetX Duo está instalado en el directorio “ *\threadx\arm7\green*”, los archivos *nxd_snmp.h*, *nxd_snmp.c*, *nx_md5.c, nx_sha.c* y nx_ *des.c* deben copiarse en este directorio.

## <a name="using-the-netx-duo-snmp-agent"></a>Uso del agente de SNMP de NetX Duo

La aplicación debe tener *nxd_snmp.c*, *nx_md5.c, nx_sha.c* y *nx_des.c* en el proyecto de compilación. El código de aplicación también debe incluir *nxd_snmp.h* después de *nx_api.h* para poder invocar los servicios SNMP. Estos archivos deben compilarse de la misma manera que otros archivos de aplicación y su forma de objeto debe vincularse a la biblioteca de NetX Duo. Esto es todo lo que se necesita para usar SNMP de NetX Duo.

> [!NOTE]
> *Si se especifica **NX_SNMP_NO_SECURITY** en el proceso de compilación, los archivos nx_md5.c, nx_sha.c y nx_des.c no son necesarios.*

> [!NOTE]
> Dado que NetX Duo SNMP usa los servicios de UDP, UDP debe habilitarse con la llamada *nx_udp_enable* antes de utilizar SNMP.

## <a name="small-example-system"></a>Sistema de ejemplo pequeño

En la figura 1.0 que aparece a continuación, se muestra un ejemplo de cómo usar el agente de SNMP de NetX Duo. En este ejemplo, el archivo de inclusión SNMP *nxd_snmp.h* se incluye en la línea 6. El archivo de encabezado que define los elementos de base de datos MIB, *demo_snmp_helper.h,* se introduce en la línea 8. MIB se define a partir de la línea 32. A continuación, el agente SNMP se crea en “*tx_application_define*” en la línea 129. Tenga en cuenta que el bloque de control del agente SNMP "*my_agent*" se definió anteriormente como una variable global en la línea 18. Si IPv6 está habilitado, las direcciones IPv6 se registran con la instancia de IP en las líneas 166-223. El agente de SNMP se inicia en la línea 229. Las definiciones de devoluciones de llamada de objetos de SNMP para las solicitudes GET, GETNEXT y SET del administrador de SNMP, así como las solicitudes de actualización de nombre de usuario y MIB, se procesan a partir de la línea 250. En este ejemplo no se realiza ninguna autenticación.

> [!NOTE]
> *La tabla MIB2 que se muestra a continuación es solo un ejemplo. La aplicación puede usar otro MIB e incluirlo en archivos independientes, así como definir el proceso GET, GETNEXT o SET en función de los requisitos de su aplicación.*

```c
/* This is a small demo of the NetX SNMP Agent on the high-performance NetX TCP/IP  
   stack. This demo relies on ThreadX and NetX to show simple SNMP the SNMP
   GET/GETNEXT/SET requests on MIB-2 objects.  */

#include  "tx_api.h"
#include  "nx_api.h"
#include  "nxd_snmp.h"
#include  "demo_snmp_helper.h"

#define     DEMO_STACK_SIZE         4096
#define     AGENT_PRIMARY_ADDRESS   IP_ADDRESS(192, 2, 2, 66)

/* Define the ThreadX and NetX object control blocks...  */

TX_THREAD               thread_0;
NX_PACKET_POOL          pool_0;
NX_IP                   ip_0;
NX_SNMP_AGENT           my_agent;


/* Indicate if using IPv6 to communicate with SNMP servers. Note that
   IPv6 must be enabled in the NetX Duo library first. Further, IPv6
   and ICMPv6 services are enabled before starting the SNMP agent. */
#define USE_IPV6


/* Define authentication and privacy keys.  */

#ifdef AUTHENTICATION_REQUIRED
NX_SNMP_SECURITY_KEY    my_authentication_key;
#endif

#ifdef PRIVACY_REQUIRED
NX_SNMP_SECURITY_KEY    my_privacy_key;
#endif

/* Define an error counter variable. */
UINT error_counter = 0;

/* This binds a secondary interfaces to the primary IP network interface 
   if SNMP is required for required for that interface. */
/* #define  MULTI_HOMED_DEVICE */

/* Define function prototypes.  A generic ram driver is used in this demo.  However
   to properly run an SNMP agent demo, a real driver should be substituted. */

VOID    thread_agent_entry(ULONG thread_input);
VOID    _nx_ram_network_driver(NX_IP_DRIVER *driver_req_ptr);
UINT    mib2_get_processing(NX_SNMP_AGENT *agent_ptr, UCHAR *object_requested, 
                            NX_SNMP_OBJECT_DATA *object_data);
UINT    mib2_getnext_processing(NX_SNMP_AGENT *agent_ptr, UCHAR *object_requested, 
                            NX_SNMP_OBJECT_DATA *object_data);
UINT    mib2_set_processing(NX_SNMP_AGENT *agent_ptr, UCHAR *object_requested, 
                            NX_SNMP_OBJECT_DATA *object_data);
UINT    mib2_username_processing(NX_SNMP_AGENT *agent_ptr, UCHAR *username);
VOID    mib2_variable_update(NX_IP *ip_ptr, NX_SNMP_AGENT *agent_ptr);


UCHAR context_engine_id[] = {0x80, 0x00, 0x0d, 0xfe, 0x03, 0x00, 0x11, 0x23, 0x23, 
                             0x44, 0x55};
UINT  context_engine_size = 11;
UCHAR context_name[] = {0x69, 0x6e, 0x69, 0x74, 0x69, 0x61, 0x6c};
UINT  context_name_size = 7;

/* Define main entry point.  */

int main()
{

   /* Enter the ThreadX kernel.  */
   tx_kernel_enter();
}


/* Define what the initial system looks like.  */
void    tx_application_define(void *first_unused_memory)
{

UCHAR   *pointer;
UINT    status;


   /* Setup the working pointer.  */
   pointer =  (UCHAR *) first_unused_memory;

   status = tx_thread_create(&thread_0, "agent thread", thread_agent_entry, 0,  
            pointer, DEMO_STACK_SIZE, 
            4, 4, TX_NO_TIME_SLICE, TX_AUTO_START);
   if (status != NX_SUCCESS)
   {
      return;
   }

   pointer =  pointer + DEMO_STACK_SIZE;


   /* Initialize the NetX system.  */
   nx_system_initialize();

   /* Create packet pool.  */
   status = nx_packet_pool_create(&pool_0, "NetX Packet Pool 0", 2048, 
                                   pointer, 20000);

   if (status != NX_SUCCESS)
   {
      return;
   }
  
   pointer = pointer + 20000;
  
   /* Create an IP instance.  */
   status = nx_ip_create(&ip_0, "SNMP Agent IP Instance", AGENT_PRIMARY_ADDRESS, 
                        0xFFFFFF00UL, &pool_0, _nx_ram_network_driver,
                        pointer, 4096, 1);

   if (status != NX_SUCCESS)
   {
      return;
   }
  
   pointer =  pointer + 4096;
  
   /* Enable ARP and supply ARP cache memory for IP Instance 0.  */
   nx_arp_enable(&ip_0, (void *) pointer, 1024);
   pointer = pointer + 1024;

   /* Enable UPD processing for IP instance.  */
   nx_udp_enable(&ip_0);
  
   /* Enable ICMP for ping.  */
   nx_icmp_enable(&ip_0);
  
   /* Create an SNMP agent instance.  */
   status = nx_snmp_agent_create(&my_agent, "SNMP Agent", &ip_0, pointer, 4096, 
                                 &pool_0, 
                                 mib2_username_processing, mib2_get_processing, 
                                 mib2_getnext_processing, 
                                 mib2_set_processing);


  
   if (status != NX_SUCCESS)
   {
      return;
   }
  
   pointer =  pointer + 4096;
  
   status = nx_snmp_agent_context_engine_set(&my_agent, context_engine_id, 
                                             context_engine_size);
  
   if (status != NX_SUCCESS)
   {
      error_counter++;
   }
  
   return;
}
  
VOID thread_agent_entry(ULONG thread_input)
{
  
#ifdef USE_IPV6
UINT        iface_index, address_index;
UINT        status;
NXD_ADDRESS agent_ipv6_address;
#endif
  
  
      /* Allow NetX time to get initialized. */
      tx_thread_sleep(100);
  
      /* If using IPv6, enable IPv6 and ICMPv6 services and get IPv6 addresses 
         registered with NetX Dou. */
  
#ifdef USE_IPV6
  
      /* Enable IPv6 on the IP instance. */
      status = nxd_ipv6_enable(&ip_0);
   
      /* Check for enable errors.  */
      if (status)
      {
  
         error_counter++;
         return;
      }
      /* Enable ICMPv6 on the IP instance. */
      status = nxd_icmp_enable(&ip_0);
  
      /* Check for enable errors.  */
      if (status)
      {
  
         error_counter++;
         return;
      }
  
      agent_ipv6_address.nxd_ip_address.v6[3] = 0x101;
      agent_ipv6_address.nxd_ip_address.v6[2] = 0x0;
      agent_ipv6_address.nxd_ip_address.v6[1] = 0x0000f101;
      agent_ipv6_address.nxd_ip_address.v6[0] = 0x20010db8;
      agent_ipv6_address.nxd_ip_version = NX_IP_VERSION_V6;
  
      /* Set the primary interface for our DNS IPv6 addresses. */
      iface_index = 0;
  
      /* This assumes we are using the primary network interface (index 0). */
      status = nxd_ipv6_address_set(&ip_0, iface_index, NX_NULL, 10, &address_index);
  
      /* Check for link local address set error.  */
      if (status) 
      {
  
         error_counter++;
         return;
      }
  
      /* Set the host global IP address. We are assuming a 64 
         bit prefix here but this can be any value (< 128). */
      status = nxd_ipv6_address_set(&ip_0, iface_index, &agent_ipv6_address, 64, 
                                    &address_index);
  
      /* Check for global address set error.  */
      if (status) 
      {
  
         error_counter++;
         return;
      }

      /* Wait while NetX Duo validates the link local and global address. */
      tx_thread_sleep(500);
#endif
  
#ifdef AUTHENTICATION_REQUIRED

      /* Create an authentication key.  */
      nx_snmp_agent_md5_key_create(&my_agent, "authpassword", &my_authentication_key);
  
      /* Use the authentication key.  */
      nx_snmp_agent_authenticate_key_use(&my_agent, &my_authentication_key);
#endif
  
#ifdef PRIVACY_REQUIRED
  
      /* Create a privacy key.  */
      nx_snmp_agent_md5_key_create(&my_agent, "privpassword", &my_privacy_key);
  
      /* Use the privacy key.  */
      nx_snmp_agent_privacy_key_use(&my_agent, &my_privacy_key);
#endif
  
      /* Start the SNMP instance.  */
      nx_snmp_agent_start(&my_agent);
  
}
  
/* Define the application's GET processing routine.  */
  
UINT    mib2_get_processing(NX_SNMP_AGENT *agent_ptr, UCHAR *object_requested, 
                            NX_SNMP_OBJECT_DATA *object_data)
{
  
UINT    i;
UINT    status;
  
  
      printf("SNMP Manager GET Request For:  %s", object_requested);
  
      /* Loop through the sample MIB to see if we have information for the supplied 
         variable.  */
      i =  0;
      status =  NX_SNMP_ERROR;
      while (mib2_mib[i].object_name)
      {
  
         /* See if we have found the matching entry.  */
         status =  nx_snmp_object_compare(object_requested, mib2_mib[i].object_name);
  
         /* Was it found?  */
         if (status == NX_SUCCESS)
         {
  
            /* Yes it was found.  */
            break;
         }
  
         /* Move to the next index.  */
         i++;
      }
  
      /* Determine if a not found condition is present.  */
      if (status != NX_SUCCESS)
      {
  
         printf(" NO SUCH NAME!\n");
  
         /* The object was not found - return an error.  */
         return(NX_SNMP_ERROR_NOSUCHNAME);
      }
  
      /* Determine if the entry has a get function.  */
      if (mib2_mib[i].object_get_callback)
      {
  
         /* Yes, call the get function.  */
         status =  (mib2_mib[i].object_get_callback)(mib2_mib[i].object_value_ptr, 
                                                     object_data);
      }
      else
      {
  
         printf(" NO GET FUNCTION!");
  
         /* No get function, return no access.  */
         status =  NX_SNMP_ERROR_NOACCESS;
      }
  
      printf("\n");

      /* Return the status.  */
      return(status);
}
  
  
/* Define the application's GETNEXT processing routine.  */
  
UINT    mib2_getnext_processing(NX_SNMP_AGENT *agent_ptr, UCHAR *object_requested, 
                                NX_SNMP_OBJECT_DATA *object_data)
{
  
UINT    i;
UINT    status;
  
  
   printf("SNMP Manager GETNEXT Request For:  %s", object_requested);
  
   /* Loop through the sample MIB to see if we have information for the supplied 
      variable.  */
      i =  0;
      status =  NX_SNMP_ERROR;
      while (mib2_mib[i].object_name)
      {
  
         /* See if we have found the next entry.  */
         status =  nx_snmp_object_compare(object_requested, mib2_mib[i].object_name);
  
         /* Is the next entry the mib greater?  */
         if (status == NX_SNMP_NEXT_ENTRY)
         {
  
            /* Yes it was found.  */
            break;
         }
  
         /* Move to the next index.  */
         i++;
      }
  
      /* Determine if a not found condition is present.  */
      if (status != NX_SNMP_NEXT_ENTRY)
      {
  
         printf(" NO SUCH NAME!\n");
  
         /* The object was not found - return an error.  */
         return(NX_SNMP_ERROR_NOSUCHNAME);
      }
  
  
      /* Copy the new name into the object.  */
      nx_snmp_object_copy(mib2_mib[i].object_name, object_requested);
  
      printf(" Next Name is: %s", object_requested);
  
      /* Determine if the entry has a get function.  */
      if (mib2_mib[i].object_get_callback)
      {
  
         /* Yes, call the get function.  */
         status =  (mib2_mib[i].object_get_callback)(mib2_mib[i].object_value_ptr, 
                                                     object_data);
  
         /* Determine if the object data indicates an end-of-mib condition.  */
         if (object_data -> nx_snmp_object_data_type == NX_SNMP_END_OF_MIB_VIEW)
         {
  
            /* Copy the name supplied in the mib table.  */
            nx_snmp_object_copy(mib2_mib[i].object_value_ptr, object_requested);
         }
      }
      else
      {
  
         printf(" NO GET FUNCTION!");
  
         /* No get function, return no access.  */
         status =  NX_SNMP_ERROR_NOACCESS;
      }
  
      printf("\n");

      /* Return the status.  */
      return(status);
}
  
  
/* Define the application's SET processing routine.  */
  
UINT    mib2_set_processing(NX_SNMP_AGENT *agent_ptr, UCHAR *object_requested, 
                            NX_SNMP_OBJECT_DATA *object_data)
{
  
UINT    i;
UINT    status;
  
  
   printf("SNMP Manager SET Request For:  %s", object_requested);
  
   /* Loop through the sample MIB to see if we have information for the supplied variable.  */
      i =  0;
      status =  NX_SNMP_ERROR;
      while (mib2_mib[i].object_name)
      {
  
         /* See if we have found the matching entry.  */
         status =  nx_snmp_object_compare(object_requested, mib2_mib[i].object_name);
  
         /* Was it found?  */
         if (status == NX_SUCCESS)
         {
  
            /* Yes it was found.  */
            break;
         }
  
         /* Move to the next index.  */
         i++;
      }
  
      /* Determine if a not found condition is present.  */
      if (status != NX_SUCCESS)
      {
  
         printf(" NO SUCH NAME!\n");
  
         /* The object was not found - return an error.  */
         return(NX_SNMP_ERROR_NOSUCHNAME);
      }
  
  
      /* Determine if the entry has a set function.  */
      if (mib2_mib[i].object_set_callback)
      {
  
         /* Yes, call the set function.  */
         status =  (mib2_mib[i].object_set_callback)(mib2_mib[i].object_value_ptr, 
                                                     object_data);
      }
      else
      {
  
         printf(" NO SET FUNCTION!");
  
         /* No get function, return no access.  */
         status =  NX_SNMP_ERROR_NOACCESS;
      }
  
      printf("\n");

      /* Return the status.  */
      return(status);
}
  
  
/* Define the application's authentication routine.  */
  
UINT  mib2_username_processing(NX_SNMP_AGENT *agent_ptr, UCHAR *username)
{
  
      printf("Username is:  %s\n", username);
  
      /* Update MIB-2 objects. In this example, it is only the SNMP objects.  */
      mib2_variable_update(&ip_0, &my_agent);
  
      /* No authentication is done, just return success!  */
      return(NX_SUCCESS);
}
  
  
/* Define the application's update routine.  */ 
  
VOID  mib2_variable_update(NX_IP *ip_ptr, NX_SNMP_AGENT *agent_ptr)
{
  
      /* Update the snmp parameters.  */
      snmpInPkts =                agent_ptr -> nx_snmp_agent_packets_received;
      snmpOutPkts =               agent_ptr -> nx_snmp_agent_packets_sent;
      snmpInBadVersions =         agent_ptr -> nx_snmp_agent_invalid_version;
      snmpInBadCommunityNames =   agent_ptr -> nx_snmp_agent_authentication_errors;
      snmpInBadCommunityUsers =   agent_ptr -> nx_snmp_agent_username_errors; 
      snmpInASNParseErrs =        agent_ptr -> nx_snmp_agent_internal_errors;
      snmpInTotalReqVars =        agent_ptr -> nx_snmp_agent_total_get_variables;
      snmpInTotalSetVars =        agent_ptr -> nx_snmp_agent_total_set_variables;
      snmpInGetRequests =         agent_ptr -> nx_snmp_agent_get_requests;
      snmpInGetNexts =            agent_ptr -> nx_snmp_agent_getnext_requests;
      snmpInSetRequests =         agent_ptr -> nx_snmp_agent_set_requests;
      snmpOutTooBigs =            agent_ptr -> nx_snmp_agent_too_big_errors;
      snmpOutNoSuchNames =        agent_ptr -> nx_snmp_agent_no_such_name_errors;
      snmpOutBadValues =          agent_ptr -> nx_snmp_agent_bad_value_errors;
      snmpOutGenErrs =            agent_ptr -> nx_snmp_agent_general_errors;
      snmpOutTraps =              agent_ptr -> nx_snmp_agent_traps_sent;
}   
```
Figura 1.0 Ejemplo de uso del agente de SNMP con NetX Duo

## <a name="configuration-options"></a>Opciones de configuración

Hay varias opciones de configuración para compilar SNMP para NetX Duo. A continuación, se muestra una lista de todas las opciones, donde se describe con detalle cada una de ellas:  
  
| Definir                     | Significado                                                                                                                                                                                                                                                                        |
|----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **NX_SNMP_AGENT_PRIORITY**     | Prioridad del subproceso del agente de SNMP. De forma predeterminada, este valor se define como 16 para especificar la prioridad 16.                                                                                                                                                                         |
| **NX_SNMP_TYPE_OF_SERVICE**    | Tipo de servicio necesario para las respuestas UDP de SNMP. De forma predeterminada, este valor se define como NX_IP_NORMAL para indicar el servicio de paquetes de IP normal. La aplicación puede establecer esta definición antes de la inclusión de *nxd_snmp.h.*                                                       |
| **NX_SNMP_FRAGMENT_OPTION**    | Habilitación del fragmento para solicitudes de UDP de SNMP. De forma predeterminada, este valor es NX_DONT_FRAGMENT para deshabilitar la fragmentación de UDP de SNMP. La aplicación puede establecer esta definición antes de la inclusión de *nxd_snmp.h.*                                                                                 |
| **NX_SNMP_TIME_TO_LIVE**       | Especifica el período de vida antes de que expire. El valor predeterminado se establece en 0x80, pero se puede redefinir antes de la inclusión de *nxd_snmp.h.*                                                                                                                                         |
| **NX_SNMP_AGENT_TIMEOUT**      | Especifica el número de tics de ThreadX durante los que se suspenderán los servicios internos. El valor predeterminado se establece en 100, pero se puede redefinir antes de la inclusión de *nxd_snmp.h.*                                                                                                         |
| **NX_SNMP_MAX_OCTET_STRING**   | Especifica el número máximo de bytes permitidos en una cadena de octetos en el agente de SNMP. El valor predeterminado se establece en 255, pero se puede redefinir antes de la inclusión de *nxd_snmp.h.*                                                                                                    |
| **NX_SNMP_MAX_CONTEXT_STRING** | Especifica el número máximo de bytes de una cadena de motor de contexto en el agente de SNMP. El valor predeterminado se establece en 32, pero se puede redefinir antes de la inclusión de *nxd_snmp.h.*                                                                                                    |
| **NX_SNMP_MAX_USER_NAME**      | Especifica el número máximo de bytes de un nombre de usuario (incluidas las cadenas de comunidad). El valor predeterminado se establece en 64, pero se puede redefinir antes de la inclusión de *nxd_snmp.h.*                                                                                                      |
| **NX_SNMP_MAX_SECURITY_KEY**   | Especifica el número de bytes permitidos en una cadena de clave de seguridad. El valor predeterminado se establece en 64, pero se puede redefinir antes de la inclusión de *nxd_snmp.h.*                                                                                                                          |
| **NX_SNMP_PACKET_SIZE**        | Especifica el tamaño mínimo de los paquetes del grupo especificado al crear el agente de SNMP. El tamaño mínimo es necesario para asegurarse de que la carga de SNMP completa puede estar incluida en un paquete. El valor predeterminado se establece en 560, pero se puede redefinir antes de la inclusión de *nxd_snmp.h.* |
| **NX_SNMP_AGENT_PORT**         | Especifica el puerto UDP en el que se atenderán las solicitudes de administrador de SNMP. El puerto predeterminado es el de UDP 161, pero se puede redefinir antes de la inclusión de *nxd_snmp.h.*                                                                                                                             |
| **NX_SNMP_MANAGER_TRAP_PORT**  | Especifica el puerto UDP al que se enviarán las solicitudes de captura del agente de SNMP. El puerto predeterminado es UDP 162, pero se puede redefinir antes de la inclusión de *nxd_snmp.h.*                                                                                                                           |
| **NX_SNMP_MAX_TRAP_NAME**      | Especifica el tamaño de la matriz que contiene el nombre de usuario enviado con mensajes de captura. El valor predeterminado es 64.                                                                                                                                                                         |
| **NX_SNMP_MAX_TRAP_KEY**       | Especifica el tamaño de las claves de privacidad y autenticación de los mensajes de captura. El valor predeterminado es 64.                                                                                                                                                                          |
| **NX_SNMP_TIME_INTERVAL**      | Determina el intervalo de suspensión en tics del temporizador que realiza la tarea de subproceso de SNMP entre el procesamiento de paquetes de SNMP recibidos. El valor predeterminado es 100. Durante dicho intervalo, la aplicación host tiene acceso a los servicios de la API de SNMP.                                           |
| **NX_SNMP_DISABLE_V1**         | Si se define, elimina todo el procesamiento de la versión 1 de SNMP en *nxd_snmp.c.* De manera predeterminada, no está definido.                                                                                                                                                                         |
| **NX_SNMP_DISABLE_V2**         | Si se define, elimina todo el procesamiento de la versión 2 de SNMP en *nxd_snmp.c.* De manera predeterminada, no está definido.                                                                                                                                                                         |
| **NX_SNMP_DISABLE_V3**         | Si está definido, elimina todo el procesamiento de SNMPv3 en *nxd_snmp.c.* De manera predeterminada, no está definido.                                                                                                                                                                                 |