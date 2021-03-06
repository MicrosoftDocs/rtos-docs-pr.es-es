---
title: 'Capítulo 2: Instalación y uso de la AutoIP de Azure RTOS NetX'
description: Este capítulo contiene una descripción de varios problemas relacionados con la instalación, la configuración y el uso del componente AutoIP de Azure RTOS NetX.
author: philmea
ms.author: philmea
ms.date: 06/04/2020
ms.topic: article
ms.service: rtos
ms.openlocfilehash: 9bc5ce189980dbceaf12a2f2e8429d9267e7d37f559c88d10c54e399d01ec259
ms.sourcegitcommit: 93d716cf7e3d735b18246d659ec9ec7f82c336de
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2021
ms.locfileid: "116796895"
---
# <a name="chapter-2---installation-and-use-of-azure-rtos-netx-autoip"></a>Capítulo 2: Instalación y uso de la AutoIP de Azure RTOS NetX

Este capítulo contiene una descripción de varios problemas relacionados con la instalación, la configuración y el uso del componente AutoIP de Azure RTOS NetX.

## <a name="product-distribution"></a>Distribución del producto

AutoIP para NetX está disponible en [https://github.com/azure-rtos/netx](https://github.com/azure-rtos/netx). El paquete incluye tres archivos de código fuente, un archivo de inclusión y un archivo PDF que contiene este documento, como se indica a continuación:

- **nx_auto_ip.h**: archivo de encabezado para AutoIP de NetX
- **nx_auto_ip.c**: archivo de código fuente C para AutoIP de NetX
- **demo_netx_auto_ip.c**: demo de archivo de código fuente C para AutoIP de NetX
- **nx_auto_ip.pdf**: descripción en PDF de AutoIP de NetX

## <a name="autoip-installation"></a>Instalación de AutoIP

Para usar AutoIP de NetX, la distribución completa que se ha mencionado anteriormente debe copiarse en el mismo directorio en el que está instalado NetX. Por ejemplo, si NetX se ha instalado en el directorio " *\threadx\arm7\green*", los archivos *nx_auto_ip.h*, *nx_auto_ip.c* y *demo_netx_auto_ip.c* deben copiarse en ese mismo directorio.

## <a name="using-autoip"></a>Uso de AutoIP

El uso de AutoIP de NetX es sencillo. Básicamente, el código de la aplicación debe incluir *nx_auto_ip.h* después de incluir *tx_api.h* y *nx_api.h*, con el fin de usar ThreadX y NetX. Después de incluir *nx_auto_ip.h*, el código de la aplicación puede realizar las llamadas de función AutoIP que se especifican más adelante en este manual. La aplicación también debe incluir *nx_auto_ip.c* en el proceso de compilación. Estos archivos se deben compilar de la misma manera que otros archivos de aplicación y su forma de objeto debe vincularse junto con los archivos de la aplicación. Esto es todo lo que se necesita para usar AutoIP de NetX.

> [!NOTE]
> Dado que AutoIP usa servicios ARP de NetX, ARP debe estar habilitado con la llamada a *nx_arp_enable* antes de usar AutoIP.

## <a name="small-example-system"></a>Sistema de ejemplo pequeño

Un ejemplo de lo fácil que es usar AutoIP de NetX se describe en la figura 1.1, que aparece a continuación. En este ejemplo, el archivo de inclusión de AutoIP *nx_auto_ip.h* se agrega en la línea 002. A continuación, se crea la instancia de AutoIP de NetX en "*tx_application_define*" en la línea 090. Observe que el bloque de control de AutoIP de NetX "auto_ip_0" se definió anteriormente como una variable global en la línea 015. Después de la creación correcta, se inicia AutoIP de NetX en la línea 098. El procesamiento de la función de devolución de llamada de cambio de dirección IP comienza en la línea 105, que se usa para controlar los conflictos subsiguientes o la posible resolución de direcciones DHCP.

> [!NOTE]
> En el ejemplo siguiente se supone que el dispositivo host es un dispositivo de host único. En el caso de un dispositivo de host múltiple, la aplicación host puede usar el servicio AutoIP de NetX *nx_auto_ip_interface_* establecido para especificar una interfaz de red secundaria para sondear una dirección IP. Consulte el **Manual de usuario de NetX** para obtener más detalles sobre la configuración de aplicaciones de host múltiple. Tenga en cuenta que la aplicación host debe usar la API de NetX *nx_status_ip_interface_check* para comprobar que AutoIP haya obtenido una dirección IP.

## <a name="example-of-autoip-use-with-netx"></a>Ejemplo de uso de AutoIP con NetX

```c
000 #include "tx_api.h"
001 #include "nx_api.h"
002 #include "nx_auto_ip.h"
003
004 #define         DEMO_STACK_SIZE         4096
005
006 /* Define the ThreadX and NetX object control blocks... */
007
008 TX_THREAD         thread_0;
009 NX_PACKET_POOL    pool_0;
010 NX_IP             ip_0;
011
012
013 /* Define the AUTO IP structures for the IP instance. */
014
015 NX_AUTO_IP         auto_ip_0;
016
017
018 /* Define the counters used in the demo application... */
019
020 ULONG             thread_0_counter;
021 ULONG             address_changes;
022 ULONG             error_counter;
023
024
025 /* Define thread prototypes. */
026
027 void     thread_0_entry(ULONG thread_input);
028 void     ip_address_changed(NX_IP *ip_ptr, VOID *auto_ip_address);
029 void     _nx_ram_network_driver(struct NX_IP_DRIVER_STRUCT *driver_req);
030
031
032 /* Define main entry point. */
033
034 int main()
035 {
036
037     /* Enter the ThreadX kernel. */
038     tx_kernel_enter();
039 }
040
041
042 /* Define what the initial system looks like. */
043
044 void     tx_application_define(void *first_unused_memory)
045 {
046
047 CHAR     *pointer;
048 UINT     status;
049
050
051     /* Setup the working pointer. */
052     pointer = (CHAR *) first_unused_memory;
053
054     /* Create the main thread. */
055     tx_thread_create(&thread_0, "thread 0", thread_0_entry, 0,
056                     pointer, DEMO_STACK_SIZE,
057                     16, 16, 1, TX_AUTO_START);
058
059     pointer = pointer + DEMO_STACK_SIZE;
060
061     /* Initialize the NetX system. */
062     nx_system_initialize();
063
064     /* Create a packet pool. */
065     status = nx_packet_pool_create(&pool_0, "NetX Main Packet Pool", 128,
066                                     pointer, 4096);
067                                     pointer = pointer + 4096;
068
069     if (status)
070         error_counter++;
071
072     /* Create an IP instance. */
073     status = nx_ip_create(&ip_0, "NetX IP Instance 0", IP_ADDRESS(0, 0, 0, 0),
074                             0xFFFFFF00UL, &pool_0, _nx_ram_network_driver,
075                             pointer, 4096, 1);
076                             pointer = pointer + 4096;
077
078     if (status)
079         error_counter++;
080
081     /* Enable ARP and supply ARP cache memory for IP Instance 0. */
082     status = nx_arp_enable(&ip_0, (void *) pointer, 1024);
083     pointer = pointer + 1024;
084
085     /* Check ARP enable status. */
086     if (status)
087         error_counter++;
088
089     /* Create the AutoIP instance for IP Instance 0. */
090     status = nx_auto_ip_create(&auto_ip_0, "AutoIP 0", &ip_0, pointer, 4096, 1);
091     pointer = pointer + 4096;
092
093     /* Check AutoIP create status. */
094     if (status)
095         error_counter++;
096
097     /* Start AutoIP instances. */
098     status = nx_auto_ip_start(&auto_ip_0, 0 /*IP_ADDRESS(169,254,254,255)*/);
099
100     /* Check AutoIP start status. */
101     if (status)
102         error_counter++;
103
104     /* Register an IP address change function for IP Instance 0. */
105     status = nx_ip_address_change_notify(&ip_0, ip_address_changed,
106                                         (void *) &auto_ip_0);
107
108     /* Check IP address change notify status. */
109     if (status)
110         error_counter++;
111     }
112
113
114     /* Define the test thread. */
115
116     void thread_0_entry(ULONG thread_input)
117     {
118
119     UINT      status;
120     ULONG     actual_status;
121
122
123          /* Wait for IP address to be resolved. */
124         do
125         {
126
127             /* Call IP status check routine. */
128             status = nx_ip_status_check(&ip_0, NX_IP_ADDRESS_RESOLVED,
129                                         &actual_status, 10000);
130
131         } while (status != NX_SUCCESS);
132
133         /* Since the IP address is resolved at this point, the application
134         can now fully utilize NetX! */
135
136         while(1)
137         {
138
139
140
141             /* Increment thread 0's counter. */
142             thread_0_counter++;
143
144             /* Sleep... */
145             tx_thread_sleep(10);
146         }
147     }
148
149
150     void ip_address_changed(NX_IP *ip_ptr, VOID *auto_ip_address)
151     {
152
153     ULONG         ip_address;
154     ULONG         network_mask;
155     NX_AUTO_IP    *auto_ip_ptr;
156
157
158     /* Setup pointer to auto IP instance. */
159     auto_ip_ptr = (NX_AUTO_IP *) auto_ip_address;
160
161     /* Pickup the current IP address. */
162     nx_ip_address_get(ip_ptr, &ip_address, &network_mask);
163
164     /* Determine if the IP address has changed back to zero. If so,
165     make sure the AutoIP instance is started. */
166     if (ip_address == 0)
167     {
168
169         /* Get the last AutoIP address for this node. */
170         nx_auto_ip_get_address(auto_ip_ptr, &ip_address);
171
172         /* Start this AutoIP instance. */
173         nx_auto_ip_start(auto_ip_ptr, ip_address);
174     }
175
176     /* Determine if IP address has transitioned to a non local IP address. */
177     else if ((ip_address & 0xFFFF0000UL) != IP_ADDRESS(169, 254, 0, 0))
178     {
179
180         /* Stop the AutoIP processing. */
181         nx_auto_ip_stop(auto_ip_ptr);
182     }
183
184     /* Increment a counter. */
185     address_changes++;
186 }
```

## <a name="configuration-options"></a>Opciones de configuración

Hay varias opciones de configuración para compilar AutoIP de NetX. A continuación se muestra una lista de todas las opciones, donde cada una de ellas se describe en detalle:

- **NX_DISABLE_ERROR_CHECKING**: si está definida, esta opción quita la comprobación de errores básica de AutoIP. Normalmente se utiliza después de depurar la aplicación.
- **NX_AUTO_IP_PROBE_WAIT**: el número de segundos que hay que esperar antes de enviar el primer sondeo. De forma predeterminada, este valor es 1.
- **NX_AUTO_IP_PROBE_NUM**: el número de sondeos ARP que se van a enviar. De forma predeterminada, este valor es 3.
- **NX_AUTO_IP_PROBE_MIN**: el número mínimo de segundos que hay que esperar entre el envío de sondeos. De forma predeterminada, este valor es 1.
- **NX_AUTO_IP_PROBE_MAX**: el número máximo de segundos que hay que esperar entre el envío de sondeos. De forma predeterminada, este valor es 2.
- **NX_AUTO_IP_MAX_CONFLICTS**: el número de conflictos de AutoIP antes de aumentar los retrasos de procesamiento. De forma predeterminada, este valor es 10.
- **NX_AUTO_IP_RATE_LIMIT_INTERVAL**: el número de segundos que se debe extender el período de espera cuando se supera el número total de conflictos. De forma predeterminada, este valor es 60.
- **NX_AUTO_IP_ANNOUNCE_WAIT**: el número de segundos que hay que esperar antes de enviar el anuncio. De forma predeterminada, este valor es 2.
- **NX_AUTO_IP_ANNOUNCE_NUM**: el número de anuncios de ARP que se van a enviar. De forma predeterminada, este valor es 2.
- **NX_AUTO_IP_ANNOUNCE_INTERVAL**: el número de segundos de espera entre el envío de anuncios. De forma predeterminada, este valor es 2.
- **NX_AUTO_IP_DEFEND_INTERVAL**: el número de segundos de espera entre anuncios de defensa. De forma predeterminada, este valor es 10.