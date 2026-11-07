# Containerlab desde cero: FRRouting básico

Esta guía muestra cómo utilizar **FRRouting (FRR)** dentro de Containerlab para crear dos routers virtuales, configurar direccionamiento IPv4 desde la CLI de FRR y comprobar la persistencia de la configuración.

---

# 1. ¿Qué es FRRouting?

**FRRouting (FRR)** es una suite de routing open source para Linux que implementa protocolos como OSPF, BGP, RIP e IS-IS.

FRR permite convertir un sistema Linux en un router y ofrece una CLI llamada:

```text
vtysh
```

La sintaxis de `vtysh` es similar a la utilizada por muchos equipos de red tradicionales.

---

# 2. Requisito de versión de Containerlab

El tipo de nodo:

```yaml
kind: frr
```

requiere una versión de Containerlab que incluya soporte nativo para FRRouting.

Verificar la versión instalada:

```bash
containerlab version
```

Si se utiliza una versión anterior que no soporta `kind: frr`, el despliegue mostrará un error indicando que el tipo de nodo no está soportado.

Para actualizar Containerlab:

```bash
sudo -E containerlab version upgrade
```

Después de actualizar, verificar nuevamente:

```bash
containerlab version
```

---

# 3. Crear la topología FRR

Crear el archivo:

```text
topologies/frr-basico.clab.yml
```

Agregar:

```yaml
name: frr-basico

topology:
  nodes:
    r1:
      kind: frr
      image: quay.io/frrouting/frr:containerlab-10.7.1

    r2:
      kind: frr
      image: quay.io/frrouting/frr:containerlab-10.7.1

  links:
    - endpoints: ["r1:eth1", "r2:eth1"]
```

La topología será:

```text
r1 -------- r2
```

El enlace conecta:

```text
r1:eth1 <------> r2:eth1
```

---

# 4. Desplegar la topología

Ejecutar:

```bash
sudo containerlab deploy -t topologies/frr-basico.clab.yml
```

Después inspeccionar:

```bash
sudo containerlab inspect -t topologies/frr-basico.clab.yml
```

Deberían aparecer los nodos:

```text
clab-frr-basico-r1
clab-frr-basico-r2
```

en ejecución.

---

# 5. Entrar a la CLI de FRR

Entrar a `r1`:

```bash
docker exec -it clab-frr-basico-r1 vtysh
```

El prompt cambia a algo similar a:

```text
r1#
```

Mostrar la versión:

```text
show version
```

Mostrar un resumen de interfaces:

```text
show interface brief
```

Salir:

```text
exit
```

Repetir en `r2`:

```bash
docker exec -it clab-frr-basico-r2 vtysh
```

Dentro:

```text
show version
show interface brief
exit
```

---

# 6. Entender las interfaces

Al ejecutar:

```text
show interface brief
```

pueden aparecer interfaces similares a:

```text
Interface       Status  VRF             Addresses
---------       ------  ---             ---------
eth0            up      default         172.20.20.4/24
                                        + 3fff:172:20:20::4/64
eth1            up      default
lo              up      default
pim6reg         up      default
pimreg          up      default
```

## `eth0`

`eth0` es la interfaz de gestión utilizada por Containerlab.

Containerlab conecta automáticamente esta interfaz a su red de management.

No se utiliza para el enlace de datos del laboratorio.

## `eth1`

`eth1` es la interfaz de datos definida en:

```yaml
links:
  - endpoints: ["r1:eth1", "r2:eth1"]
```

Esta es la interfaz que se utilizará para conectar `r1` con `r2`.

## `lo`

`lo` es la interfaz loopback local del sistema.

## `pimreg` y `pim6reg`

Son interfaces internas relacionadas con funciones de multicast/PIM de FRR.

No se utilizan en este laboratorio.

La separación principal es:

```text
eth0  -> gestión de Containerlab
eth1  -> tráfico de datos del laboratorio
```

---

# 7. Configurar IPv4 en `r1`

Entrar a FRR:

```bash
docker exec -it clab-frr-basico-r1 vtysh
```

Ingresar al modo de configuración:

```text
configure terminal
```

Seleccionar la interfaz:

```text
interface eth1
```

Asignar la dirección:

```text
ip address 10.100.0.1/30
```

Habilitar la interfaz:

```text
no shutdown
```

Salir de la interfaz:

```text
exit
```

Salir del modo de configuración:

```text
end
```

Verificar:

```text
show interface brief
```

Debería aparecer:

```text
eth1   up   default   10.100.0.1/30
```

Salir:

```text
exit
```

---

# 8. Configurar IPv4 en `r2`

Entrar:

```bash
docker exec -it clab-frr-basico-r2 vtysh
```

Configurar:

```text
configure terminal
interface eth1
 ip address 10.100.0.2/30
 no shutdown
exit
end
```

Verificar:

```text
show interface brief
```

Debería aparecer:

```text
eth1   up   default   10.100.0.2/30
```

Salir:

```text
exit
```

---

# 9. Topología direccionada

La red utilizada es:

```text
10.100.0.0/30
```

Asignación:

| Router | Interfaz | Dirección |
|---|---|---|
| r1 | eth1 | 10.100.0.1/30 |
| r2 | eth1 | 10.100.0.2/30 |

Representación:

```text
       10.100.0.0/30

r1 ---------------- r2
eth1                eth1
10.100.0.1          10.100.0.2
```

---

# 10. Probar conectividad desde FRR

Entrar a `r1`:

```bash
docker exec -it clab-frr-basico-r1 vtysh
```

Ejecutar:

```text
ping 10.100.0.2
```

El comando seguirá enviando paquetes hasta que se interrumpa manualmente.

Para detenerlo:

```text
Ctrl + C
```

Después salir:

```text
exit
```

Repetir desde `r2`:

```bash
docker exec -it clab-frr-basico-r2 vtysh
```

Dentro:

```text
ping 10.100.0.1
```

Interrumpir con:

```text
Ctrl + C
```

---

# 11. Guardar la configuración

Entrar a `r1`:

```bash
docker exec -it clab-frr-basico-r1 vtysh
```

Guardar:

```text
write memory
```

Salir:

```text
exit
```

Repetir en `r2`:

```bash
docker exec -it clab-frr-basico-r2 vtysh
```

Dentro:

```text
write memory
exit
```

`write memory` guarda la configuración activa de FRR.

---

# 12. Localizar los archivos de configuración

Containerlab mantiene archivos asociados con el laboratorio en un directorio generado automáticamente.

Buscar archivos:

```bash
find clab-frr-basico -maxdepth 3 -type f
```

Entre los archivos puede encontrarse:

```text
frr.conf
```

Este archivo contiene la configuración persistida de FRR.

La ruta exacta puede variar según la versión.

---

# 13. Consultar `frr.conf`

Si el archivo se encuentra en:

```text
clab-frr-basico/r1/config/frr.conf
```

puede mostrarse con:

```bash
cat clab-frr-basico/r1/config/frr.conf
```

Debería incluir una sección similar a:

```text
interface eth1
 ip address 10.100.0.1/30
```

---

# 14. Probar persistencia

Destruir el laboratorio:

```bash
sudo containerlab destroy -t topologies/frr-basico.clab.yml
```

No utilizar `--cleanup`, ya que se desea conservar el directorio con los archivos de configuración.

Verificar:

```bash
ls clab-frr-basico
```

Volver a desplegar:

```bash
sudo containerlab deploy -t topologies/frr-basico.clab.yml
```

No configurar ninguna dirección manualmente.

Entrar a `r1`:

```bash
docker exec -it clab-frr-basico-r1 vtysh
```

Verificar:

```text
show interface brief
```

Debería volver a aparecer:

```text
eth1   up   default   10.100.0.1/30
```

Salir:

```text
exit
```

Repetir en `r2`:

```bash
docker exec -it clab-frr-basico-r2 vtysh
```

Dentro:

```text
show interface brief
```

Debería aparecer:

```text
eth1   up   default   10.100.0.2/30
```

---

# 15. Validar conectividad después del redeploy

Desde `r1`:

```text
ping 10.100.0.2
```

Interrumpir con:

```text
Ctrl + C
```

Si responde correctamente, la configuración persistida fue reutilizada después de recrear los contenedores.

---

# 16. Consultar la configuración activa

Dentro de `r1`:

```text
show running-config
```

La salida puede incluir:

```text
frr version 10.7.1_git
frr defaults traditional
hostname r1
log stdout
service integrated-vtysh-config

interface eth1
 ip address 10.100.0.1/30
exit

router pim
 no autorp discovery
exit
```

La parte configurada manualmente en este laboratorio es:

```text
interface eth1
 ip address 10.100.0.1/30
```

---

# 17. Interpretar `show running-config`

## Versión

```text
frr version 10.7.1_git
```

Indica la versión de FRRouting en ejecución.

## Hostname

```text
hostname r1
```

Define el nombre del router dentro de FRR.

## Logging

```text
log stdout
```

Envía los logs a la salida estándar del contenedor.

## Configuración integrada

```text
service integrated-vtysh-config
```

Indica que FRR trabaja con una configuración integrada administrada mediante `vtysh`.

## Interfaz configurada

```text
interface eth1
 ip address 10.100.0.1/30
```

Esta sección corresponde a la configuración realizada manualmente en el laboratorio.

## PIM

```text
router pim
 no autorp discovery
```

Es configuración adicional relacionada con PIM/multicast.

No forma parte del objetivo de este laboratorio.

---

# 18. Flujo completo

El flujo realizado fue:

```text
crear topología
      ↓
deploy
      ↓
entrar a vtysh
      ↓
configurar eth1
      ↓
probar conectividad
      ↓
write memory
      ↓
guardar frr.conf
      ↓
destroy
      ↓
deploy
      ↓
recuperar configuración
      ↓
validar conectividad
```

---

# 19. Resultado

Validaciones realizadas:

```text
Containerlab compatible       OK
Nodos FRR                     OK
CLI vtysh                     OK
eth0 de gestión               OK
eth1 de datos                 OK
Direccionamiento IPv4         OK
Conectividad ICMP             OK
Configuración guardada        OK
Persistencia después destroy  OK
running-config                OK
```