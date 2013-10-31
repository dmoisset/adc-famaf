# MPI

-----

# Ejemplo: regla del trapecio

Se aproxima una integral como suma de áreas de trapecios

![trapezoidal rule](trapezoidal-rule.png)

Dividimos el intervalo `[a,b]` en *n* segmentos, cada uno de *h=b-a/n* de ancho.
Esto define puntos *x[0]=a*, *x[1]*, *x[2]*, ..., *x[n]=b*. Tenemos que 
*x[k+1] = x[k]+h*.

Cada trapecio tiene área *(f(x[i]) + f(x[i+1])) h / 2*

Entonces nuestra aproximación es

*I = h ( f(x[0])/2 + f(x[1]) + f(x[2]) + ... + f(x[n-1]) + f(x[n])/2)*

----
# Ejemplo: regla del trapecio

## Versión secuencial

Sin paralelismo:

    !c
    h = (b-a)/n;
    approx = (f(a) + f(b))/2.0;
    for (i = 1; i <= n-1; i++) {
        x_i = a + i*h;
        approx += f(x_i);
    }
    approx = h*approx;

Se paraleliza con una “receta” típica:

1. Particionar la solución en tareas
2. Identificar los canales de comunicación entre tareas
3. Acumular las tareas en tareas mayores
4. Asignar las tareas mayores a *cores* de ejecución

----
# Ejemplo: regla del trapecio

## Version paralela (pseudocódigo)

Supongamos para simplificar que `n` es divisible por `comm_sz`

    !c
    h = (b-a)/n;
    local_n = n/comm_sz;
    local_a = a + my_rank*local_n*h;
    local_b = local_a + local_n*h;
    local_integral = ReglaTrapecio(local_a, local_b, local_n, h);

    if (my_rank != 0)
        mandar local_integral al proceso 0
    else /* my_rank == 0 */
        total_integral = local_integral;
        for (proc = 1; proc < comm_sz; proc++) {
            Recibir local_integral desde proc;
            total_integral += local_integral;
        }
    }
    if (my rank == 0)
        print result;

Cuidado con la entrada/salida!

----
# Ejemplo: regla del trapecio

## Manejando entrada

    !c
    void Get_input(int     my_rank
                   int     comm_sz
                   double *a_p
                   double *b_p
                   int    *n_p
                   int     dest)
    {
        if (my_rank == 0) {
            printf("Enter a, b, and n\n");
            scanf("%lf %lf %d", a_p, b_p, n_p);
            for (dest = 1; dest < comm sz; dest++) {
                MPI_Send(a_p, 1, MPI_DOUBLE, dest, 0, MPI_COMM_WORLD);
                MPI_Send(b_p, 1, MPI_DOUBLE, dest, 0, MPI_COMM_WORLD);
                MPI_Send(n_p, 1, MPI_INT, dest, 0, MPI_COMM_WORLD);
            }
        } else { /* my rank != 0 */
            MPI_Recv(a_p, 1, MPI_DOUBLE, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
            MPI_Recv(b_p, 1, MPI_DOUBLE, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
            MPI_Recv(n_p, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
    }

----
# Comunicación colectiva

* La suma global que hicimos es ineficiente
    * El proceso 0 hace `comm_sz` receive seguidos
* Alternativa: comunicación en árbol
    * Cuánto es a mejora de performance?
    * Hay una sola forma de hacerlo?
    * Cómo se programa?

La implementación de MPI incluye `MPI_Reduce`

    !c
    int MPI_Reduce(void        *sendbuf,
                   void        *recvbuf,
                   int          count,
                   MPI_Datatype datatype,
                   MPI_Op       op,
                   int          root,
                   MPI_Comm     comm)
                   
    MPI Reduce(&local_int, &total_int, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);

----
# Comunicación colectiva

`MPI_Reduce`:

* Puede recibir vectores
* Todos los procesos involucrados usan la misma función, con argumentos compatibles
* No se permite aliasing

<table>
    <tr><th>MPI_Op</th><th>Significa</th></tr>
    <tr><td>MPI_MAX</td><td>Máximo valor</td></tr>
    <tr><td>MPI_MIN</td><td>Mínimo valor</td></tr>
    <tr><td>MPI_MAXLOC</td><td>Máximo valor e índice de este</td></tr>
    <tr><td>MPI_MINLOC</td><td>Mínimo valor e índice de este</td></tr>
    <tr><td>MPI_SUM</td><td>+</td></tr>
    <tr><td>MPI_PROD</td><td>×</td></tr>
    <tr><td>MPI_LAND</td><td>AND lógico</td></tr>
    <tr><td>MPI_BAND</td><td>AND bit a bit</td></tr>
    <tr><td>MPI_LOR</td><td>OR lógico</td></tr>
    <tr><td>MPI_BOR</td><td>OR bit a bit</td></tr>
    <tr><td>MPI_LXOR</td><td>XOR lógico</td></tr>
    <tr><td>MPI_BXOR</td><td>XOR bit a bit</td></tr>
</table>

----
# Comunicación colectiva

    !c
    int MPI_Allreduce(void        *sendbuf,
                      void        *recvbuf,
                      int          count,
                      MPI_Datatype datatype,
                      MPI_Op       op,
                      MPI_Comm     comm)

![Butterfly communication](butterfly.png)

----
# Comunicación colectiva

    !c
    int MPI_BCast(void        *buf,
                  int          count,
                  MPI_Datatype datatype,
                  int          source,
                  MPI_Comm     comm)

Con esto la recolección de entrada puede ser:

    !c
    void Get_input(int     my_rank
                   int     comm_sz
                   double *a_p
                   double *b_p
                   int    *n_p
                   int     dest)
    {
        if (my_rank == 0) {
            printf("Enter a, b, and n\n");
            scanf("%lf %lf %d", a_p, b_p, n_p);
        }
        MPI_Bcast(a_p, 1, MPI_DOUBLE, 0, MPI_COMM_WORLD);
        MPI_Bcast(b_p, 1, MPI_DOUBLE, 0, MPI_COMM_WORLD);
        MPI_Bcast(n_p, 1, MPI_INT, 0, MPI_COMM_WORLD);
    }

----
# Distribución de datos

Otro ejemplo, suma de vectores. En versión secuencial:

    !c
    void Vector sum(double x[], double y[], double z[], int n) {
        int i;
        for (i = 0; i < n; i++)
            z[i] = x[i] + y
    }

* Las tareas son sumar componentes individuales
* No requiere comunicación! Sólo distribuir tareas en cores
    * ¿cómo?

----
# Particiones

![Particiones](partitions.png)


----
# Scatter

Distribuye particiones a procesos

    !c
    int MPI_Scatter(void        *sendbuf,
                    int          sendcnt,
                    MPI_Datatype sendtype,
                    void        *recvbuf,
                    int          recvcnt,
                    MPI_Datatype recvtype,
                    int root,
                    MPI_Comm comm)

* `sendcnt` es la cantidad de datos transmitida *a cada proceso*
* Partición por bloques
* Sólo sirve si la cantidad de datos es multiplo de `comm_sz`

----
# Gather

Inversa de scatter

    !c
    int MPI_Gather(void        *sendbuf,
                   int          sendcnt,
                   MPI_Datatype sendtype,
                   void        *recvbuf,
                   int          recvcnt,
                   MPI_Datatype recvtype,
                   int root,
                   MPI_Comm comm)

* `sendcnt` es la cantidad de datos recibida *de cada proceso*
* Partición por bloques
* Sólo sirve si la cantidad de datos es multiplo de `comm_sz`


Y también hay `Allgather`

----
# Ejemplo: multiplicación matriz/vector

Versión serial:

    !c
    for (i = 0; i < m; i++) {
        y[i] = 0.0;
        for (j = 0; j < n; j++)
            y[i] += A[i][j]*x[j];
    }

Típicamente uno no escribe así en C; hace falta mapear arreglos 2D a 
arreglos 1D.

Supongamos que cada tarea es el ciclo interior:

    !c
    for (j = 0; j < n; j++)
        y[i] += A[i*n + j] * x[j];

Y que queremos multiplicar repetidas veces

----
# Ejemplo: multiplicación matriz/vector

    !c
    x = malloc(n*sizeof(double));
    MPI_Allgather(local_x, local_n, MPI_DOUBLE,
        x, local_n, MPI_DOUBLE, comm);
    for (local_i = 0; local_i < local_m; local_i++) {
        local_y[local_i] = 0.0;
        for (j = 0; j < n; j++)
            local_y[local_i] += local_A[local_i*n+j]*x[j];
    }
    free(x);

----
# Sincronización

## `MPI_Barrier`

En general la sincronización es por comunicación, pero también hay
sincronización pura:

    !c
    int MPI_Barrier( MPI_Comm comm )

![Barrera](barrier.png)


----
# Nociones de performance

* timing
* speedup

* Comunicación vs procesmiento

----
# Ejemplo: parallel sorting

N fases de una sorting network (odd-even transposition sort):

![Sorting network](sorting-network.gif)

* Cada tarea: Determinar el valor de `a[i]` en la fase `j`
* Comunicación: pasar el valor de `a[i]` al vecino que lo necesita.

A menos que N=p, podemos hacer esto de a bloques

ver detalles en: “An Introduction to Parallel Programming”, Peter Pacheco

----
# Ejemplo: parallel sorting

## Intercambiando bloques

    !c
    MPI_Send(my_keys, n/comm_sz, MPI_INT, partner, 0, comm);
    MPI_Recv(temp_keys, n/comm_sz, MPI_INT, partner, 0, comm, MPI_STATUS_IGNORE);

* Es seguro hacerlo así?
* Podemos detectar problemas vía testing? `MPI_Ssend`

Para intercambios, también hay `MPI_Sendrecv`, y `MPI_sendrcv_replace`


