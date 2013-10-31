# Cluster computing

----
# MIMD

## 2 modelos

* Memoria Distribuida
* Memoria Compartida

## Presenter Notes

Jerarquía de Flynn?

----

# Memoria compartida

![shared](shared-memory-system.png)


* Comunicación via *reads*/*writes*
* Problemas de concurrencia clásicos (*race conditions*, *deadlocks*)
* Primitivas de sincronización explícitas (locks, semáforos)

----

# Memoria distribuida

![distributed](distributed-memory-system.png)

* Comunicación vía paso de mensajes (*send*/*receive*)
* No hay race conditions (aunque posibles deadlocks)
* Sincronización por comunicación
* Todos los datos compartidos deben copiarse

----
# Clusters

Un enfoque alternativo a la supercomputación

* Nodos (computadoras independientes)
* Una red de interconexión (normalmente una LAN convencional)
* Software para colaboración entre nodos

## Presenter Notes

Cada una con su SO

----
# Eso no es cualquier sistema distribuido

Sí pero:

* Foco en performance
* Foco en tareas de cómputo
* Un solo dominio administrativo
* Red dedicada (típicamente)
* Nodos homogéneos (típicamente)
* Alta localización física y acoplamiento

----
# Comparación con

* Grid: no localizados
* Collaborative: más de un dominio administrativo
* Sistemas distribuidos: foco en tareas de comunicación

----
# Históricamente

* Existen desde que existen redes ('60s)
* Ley de Amdahl (1967)
* ARPANET (1969)
* Hydra (Unix, TCP/IP) (1971)
* ARCnet (Comercial) (1977)
* PVM (1989)
* 1995 Beowulf cluster

## Presenter Notes

El cluster se inventó cuando a un usuario no le alcanzó con su computadora.

Impacto de baja de precios de CPU, aumento de throughput de redes

----
# Hoy

## IBM Sequoia

Cada nodo:

 * PowerPC A2 (64 bit, 16 cores)
 * 1.6GHz
 * 16 GB de RAM
 * RHEL

El cluster:

 * Conexión via Infiniband
 * 96 racks
 * 1024 CPUs/rack
 * 280 m²
 * 16.32 PFLOPS (#1 al 2012, #3 al 2013)

----
# Motivación

* Relacion performance/costo
* Disponibilidad
* Mantenibilidad
* Elasticidad


