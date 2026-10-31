# Containerlab desde cero: primera topología con dos nodos Linux

Esta guía muestra paso a paso cómo crear una topología básica con Containerlab utilizando dos contenedores Linux conectados entre sí.

El objetivo es aprender el flujo básico de trabajo:

```text
Crear topología
      ↓
Desplegar
      ↓
Inspeccionar
      ↓
Configurar
      ↓
Validar
      ↓
Destruir
```

---

# 1. Requisitos

Para realizar el laboratorio se necesita:

- Linux
- Docker
- Containerlab
- Visual Studio Code
- Extensión de Containerlab para Visual Studio Code

---

# 2. Instalar Docker

En sistemas Linux compatibles puede instalarse Docker con:

```bash
curl -sL https://containerlab.dev/setup | sudo -E bash -s "install-docker"
```

Después de la instalación, verificar:

```bash
docker --version
```

Si Docker fue agregado recientemente al grupo del usuario, puede ser necesario cerrar sesión y volver a ingresar.

También puede ejecutarse:

```bash
newgrp docker
```

para actualizar la pertenencia al grupo en la sesión actual.

---

# 3. Instalar Containerlab

Instalar Containerlab:

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

Verificar la instalación:

```bash
containerlab version
```

---

# 4. Crear el directorio del laboratorio

Ir al directorio donde se guardarán los proyectos:

```bash
cd ~/Projects
```

Crear la carpeta del laboratorio:

```bash
mkdir containerlab-event-network
```

Ingresar en ella:

```bash
cd containerlab-event-network
```

Comprobar la ubicación actual:

```bash
pwd
```

El resultado será similar a:

```text
/home/<usuario>/Projects/containerlab-event-network
```

---

# 5. Abrir el proyecto en Visual Studio Code

Ejecutar:

```bash
code .
```

El punto `.` indica que Visual Studio Code debe abrir el directorio actual.

---

# 6. Crear el archivo de topología

Crear el archivo:

```text
event.clab.yml
```

Agregar el siguiente contenido:

```yaml
name: event-network

topology:
  nodes:
    node01:
      kind: linux
      image: alpine:3.20

    node02:
      kind: linux
      image: alpine:3.20

  links:
    - endpoints: ["node01:eth1", "node02:eth1"]
```

Guardar el archivo.

---

# 7. Entender la topología

La línea:

```yaml
name: event-network
```

define el nombre del laboratorio.

La sección:

```yaml
topology:
  nodes:
```

contiene los nodos que formarán parte de la topología.

Los bloques:

```yaml
node01:
  kind: linux
  image: alpine:3.20
```

y:

```yaml
node02:
  kind: linux
  image: alpine:3.20
```

crean dos nodos Linux utilizando la imagen `alpine:3.20`.

La sección:

```yaml
links:
  - endpoints: ["node01:eth1", "node02:eth1"]
```

crea un enlace entre:

```text
node01:eth1 <------> node02:eth1
```

La topología inicial es:

```text
node01 -------- node02
```

---

# 8. Verificar el archivo

Listar el contenido del directorio:

```bash
ls
```

Debería aparecer:

```text
event.clab.yml
```

Mostrar el contenido del archivo:

```bash
cat event.clab.yml
```

Esto permite verificar desde la terminal que la topología fue guardada correctamente.

---

# 9. Desplegar la topología

Ejecutar:

```bash
sudo containerlab deploy -t event.clab.yml
```

El comando:

```text
containerlab deploy
```

despliega la topología.

La opción:

```text
-t event.clab.yml
```

indica qué archivo debe utilizar Containerlab.

Durante el despliegue, Containerlab crea los dos contenedores y el enlace definido en el archivo YAML.

Los contenedores tendrán nombres similares a:

```text
clab-event-network-node01
clab-event-network-node02
```

---

# 10. Inspeccionar el laboratorio

Ejecutar:

```bash
sudo containerlab inspect -t event.clab.yml
```

Los dos nodos deberían aparecer en ejecución.

También puede verificarse directamente con Docker:

```bash
docker ps
```

Deberían aparecer:

```text
clab-event-network-node01
clab-event-network-node02
```

---

# 11. Visualizar la topología en Visual Studio Code

Con la extensión de Containerlab instalada, puede abrirse la vista gráfica de la topología.

La representación será similar a:

```text
node01 -------- node02
```

Si la extensión aparece deshabilitada porque el workspace no es confiable, se debe habilitar **Workspace Trust** para la carpeta del proyecto.

---

# 12. Inspeccionar las interfaces de red

Mostrar las interfaces de `node01`:

```bash
docker exec clab-event-network-node01 ip addr
```

Mostrar las interfaces de `node02`:

```bash
docker exec clab-event-network-node02 ip addr
```

Entre las interfaces deberían aparecer:

```text
eth0
eth1
```

`eth0` corresponde a la red de gestión utilizada por Containerlab.

`eth1` corresponde al enlace declarado en el archivo de topología.

Para consultar únicamente `eth1`:

```bash
docker exec clab-event-network-node01 ip addr show eth1
```

y:

```bash
docker exec clab-event-network-node02 ip addr show eth1
```

---

# 13. Configurar direcciones IPv4

Se utilizará la red:

```text
10.100.0.0/30
```

Asignación:

| Nodo | Interfaz | Dirección |
|---|---|---|
| node01 | eth1 | 10.100.0.1/30 |
| node02 | eth1 | 10.100.0.2/30 |

Configurar `node01`:

```bash
docker exec clab-event-network-node01 \
  ip addr add 10.100.0.1/30 dev eth1
```

Configurar `node02`:

```bash
docker exec clab-event-network-node02 \
  ip addr add 10.100.0.2/30 dev eth1
```

El comando:

```text
ip addr add
```

agrega una dirección IP.

La parte:

```text
dev eth1
```

indica que la configuración debe aplicarse sobre la interfaz `eth1`.

---

# 14. Verificar las direcciones IPv4

En `node01`:

```bash
docker exec clab-event-network-node01 \
  ip addr show eth1
```

Debe aparecer:

```text
10.100.0.1/30
```

En `node02`:

```bash
docker exec clab-event-network-node02 \
  ip addr show eth1
```

Debe aparecer:

```text
10.100.0.2/30
```

La topología queda:

```text
        10.100.0.0/30

node01 ---------------- node02
  eth1                    eth1
10.100.0.1              10.100.0.2
```

---

# 15. Probar conectividad

Desde `node01`, enviar tres paquetes ICMP hacia `node02`:

```bash
docker exec clab-event-network-node01 \
  ping -c 3 10.100.0.2
```

Resultado esperado:

```text
3 packets transmitted
3 packets received
0% packet loss
```

Probar en sentido contrario:

```bash
docker exec clab-event-network-node02 \
  ping -c 3 10.100.0.1
```

Si ambos comandos responden correctamente, existe conectividad IPv4 entre los dos nodos.

---

# 16. Consultar la tabla de vecinos

En `node01` ejecutar:

```bash
docker exec clab-event-network-node01 \
  ip neigh
```

Debería aparecer una entrada asociada con:

```text
10.100.0.2
```

`ip neigh` muestra la tabla de vecinos del sistema.

En una red IPv4 local, Linux utiliza ARP para relacionar una dirección IP con una dirección MAC.

---

# 17. Destruir el laboratorio

Cuando ya no sea necesario mantener la topología activa, ejecutar:

```bash
sudo containerlab destroy -t event.clab.yml
```

Este comando elimina los contenedores y enlaces creados por Containerlab para esta topología.

Comprobar el resultado:

```bash
docker ps
```

Los contenedores:

```text
clab-event-network-node01
clab-event-network-node02
```

ya no deberían aparecer.

---

# 18. Importante sobre la configuración manual

Las direcciones:

```text
10.100.0.1/30
10.100.0.2/30
```

fueron agregadas manualmente después del despliegue.

Si la topología se destruye y vuelve a desplegarse:

```bash
sudo containerlab destroy -t event.clab.yml
sudo containerlab deploy -t event.clab.yml
```

los contenedores se crearán nuevamente, pero las direcciones configuradas manualmente sobre `eth1` no estarán presentes.

---

# Resultado

Al finalizar el laboratorio se habrá comprobado:

```text
Archivo YAML          OK
Nodos Linux           OK
Contenedores Docker   OK
Enlace eth1           OK
Direccionamiento IPv4 OK
Conectividad ICMP     OK
Tabla de vecinos      OK
Destroy               OK
```