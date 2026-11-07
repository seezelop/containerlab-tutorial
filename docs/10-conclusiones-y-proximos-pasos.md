# Cierre de la guía: conclusiones y próximos pasos

Esta guía tuvo como objetivo presentar Containerlab de forma progresiva, utilizando laboratorios pequeños y acumulativos.

El recorrido comenzó con una topología mínima y fue incorporando conceptos de routing, switching, segmentación y filtrado hasta llegar a una arquitectura con firewall y políticas de acceso.

---

# 1. Recorrido realizado

La progresión de la guía fue:

```text
01  Primera topología
        ↓
02  Configuración reproducible con exec
        ↓
03  FRRouting básico
        ↓
04  Open vSwitch y VLANs
        ↓
05  Trunks 802.1Q
        ↓
06  Routing entre VLANs
        ↓
07  Core y distribución
        ↓
08  Servicios y segmentación
        ↓
09  Firewall y políticas
```

La idea fue introducir cada concepto sobre una base previamente validada.

---

# 2. Conceptos trabajados

A lo largo de los laboratorios se utilizaron:

- Containerlab para definir y desplegar topologías;
- Docker como plataforma de ejecución;
- Alpine Linux como host ligero;
- FRRouting para funciones de routing;
- Open vSwitch para switching virtual;
- VLANs;
- enlaces trunk 802.1Q;
- subinterfaces;
- routing entre VLANs;
- arquitectura de core y distribución;
- segmentación de usuarios y servicios;
- routing estático;
- forwarding IPv4;
- nftables;
- políticas `ACCEPT` y `DROP`;
- contadores de reglas;
- validaciones de conectividad;
- revisión y limpieza de recursos residuales.

---

# 3. De una topología mínima a una red segmentada

El punto de partida fue:

```text
node01 -------- node02
```

Luego se incorporaron progresivamente nuevas funciones:

```text
Hosts
  |
Switching
  |
VLANs
  |
Trunks
  |
Routing
  |
Core y distribución
  |
Servicios
  |
Firewall
```

Esto permite observar cómo una red sencilla puede evolucionar hacia una arquitectura con funciones diferenciadas.

---

# 4. Segmentación y seguridad

Uno de los conceptos principales de la guía es la diferencia entre segmentar y aplicar políticas de seguridad.

La segmentación permite separar redes:

```text
VLAN 10
VLAN 20
Red de servicios
```

Pero esa separación no determina por sí sola qué tráfico puede circular entre ellas.

El laboratorio de firewall introdujo una política explícita:

```text
VLAN 10 -> Servicios : ACCEPT
VLAN 20 -> Servicios : DROP
```

De esta forma se separaron dos conceptos:

```text
Segmentación -> organización de la red
Firewall     -> control del tráfico entre segmentos
```

---

# 5. Validar antes de agregar complejidad

Durante los laboratorios se utilizó una metodología repetida:

```text
crear
  ↓
desplegar
  ↓
inspeccionar
  ↓
configurar
  ↓
validar
  ↓
agregar complejidad
  ↓
volver a validar
```

Por ejemplo, antes de aplicar reglas de firewall se comprobó que el routing funcionara correctamente.

Esto permite diferenciar problemas de conectividad de problemas provocados por una política de seguridad.

---

# 6. Troubleshooting

Los laboratorios también mostraron algunos problemas prácticos.

Entre ellos:

- interfaces OVS residuales;
- bridges que permanecen después de destruir una topología;
- configuraciones VLAN que desaparecen al recrear un bridge;
- rutas de management creadas automáticamente por Containerlab;
- nombres de interfaces demasiado largos;
- interfaces `veth` residuales;
- diferencias entre configuración persistente y configuración dentro de contenedores temporales.

Estos casos forman parte del trabajo normal con laboratorios virtuales.

---

# 7. Organización del repositorio

La estructura utilizada es:

```text
containerlab-tutorial/
├── docs/
├── images/
├── topologies/
└── .gitignore
```

`docs/` contiene la documentación paso a paso.

`images/` contiene los diagramas utilizados por cada laboratorio.

`topologies/` contiene los archivos `.clab.yml`.

---

# 8. Alcance de esta guía

Esta serie está planteada como una introducción práctica.

No pretende cubrir todas las capacidades de Containerlab ni reproducir una red empresarial completa.

El objetivo fue aprender los fundamentos necesarios para poder construir laboratorios más avanzados posteriormente.

---

# 9. Posibles evoluciones

A partir de esta base pueden desarrollarse proyectos más complejos.

Algunas posibilidades son:

- automatización con Python;
- validaciones automáticas de conectividad;
- GitHub Actions;
- generación automática de configuraciones;
- OSPF;
- BGP;
- redundancia;
- múltiples routers de core;
- múltiples enlaces de distribución;
- NAT;
- acceso a Internet simulado;
- IDS/IPS;
- monitoreo;
- captura y análisis de tráfico;
- NAC;
- 802.1X;
- RADIUS;
- integración con plataformas de observabilidad.

También pueden incorporarse imágenes de sistemas operativos de fabricantes de networking, por ejemplo:

```text
Nokia
Cisco
Juniper
```

Estas evoluciones quedan fuera del alcance de esta guía introductoria.

---

# 10. Próximo paso

El siguiente paso ya no necesita ser otro laboratorio aislado.

A partir de los conceptos vistos se puede elegir una arquitectura concreta y utilizar Containerlab como plataforma para experimentar con:

```text
routing
switching
seguridad
automatización
observabilidad
```

La base técnica necesaria para comenzar ese tipo de laboratorios queda establecida en esta serie.