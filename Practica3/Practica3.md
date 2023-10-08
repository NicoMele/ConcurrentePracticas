# Práctica 2 - Monitores

## Ejercicios:

- [Ejercicio 1](#ejercicio-1)
- [Ejercicio 2](#ejercicio-2)
- [Ejercicio 3](#ejercicio-3)
- [Ejercicio 4](#ejercicio-4)
- [Ejercicio 5](#ejercicio-5)
- [Ejercicio 6](#ejercicio-6)
- [Ejercicio 7](#ejercicio-7)
- [Ejercicio 8](#ejercicio-8)

**CONSIDERACIONES PARA RESOLVER LOS EJERCICIOS:**

- Los monitores utilizan el protocolo signal and continue.
- A una variable condition SÓLO pueden aplicársele las operaciones SIGNAL, SIGNALALL y WAIT.
- NO puede utilizarse el wait con prioridades.
- NO se puede utilizar ninguna operación que determine la cantidad de procesos encolados en una variable condition o si está vacía.
- La única forma de comunicar datos entre monitores o entre un proceso y un monitor es por medio de invocaciones al procedimiento del monitor del cual se quieren obtener (o enviar) los datos.
- No existen variables globales.
- En todos los ejercicios debe maximizarse la concurrencia.
- En todos los ejercicios debe aprovecharse al máximo la característica de exclusión mutua que brindan los monitores.
- Debe evitarse hacer busy waiting.
- En todos los ejercicios el tiempo debe representarse con la función delay.

## Ejercicio 1

Se dispone de un puente por el cual puede pasar un solo auto a la vez. Un auto pide permiso para pasar por el puente, cruza por el mismo y luego sigue su camino

```cpp
Monitor Puente
 cond cola;
 int cant= 0;
 Procedure entrarPuente ()
 while ( cant > 0) wait (cola);
 cant = cant + 1;
 end;
 Procedure salirPuente ()
 cant = cant – 1;
 signal(cola);
 end;
End Monitor;
Process Auto [a:1..M]
 Puente. entrarPuente (a);
 “el auto cruza el puente”
 Puente. salirPuente(a);
End Process;
```

**a.** ¿El código funciona correctamente? Justifique su respuesta.

**b.** ¿Se podría simplificar el programa? ¿Sin monitor? ¿Menos procedimientos? ¿Sin variable condition? En caso afirmativo, rescriba el código.

**c.** ¿La solución original respeta el orden de llegada de los vehículos? Si rescribió el código en el punto b), ¿esa solución respeta el orden de llegada?

# Respuesta:

```c
    a)El codigo tiene un error, suponete que viene  2 procesos  a entrarPuente(); uno de los dos va a encontrar a cant en 0 y va a pasar y el otro se va a quedar dormido en el wait cola , el proceso 1 cruza el puente y sale, dejando a cant en 0 y avisandole al que estaba en la cola q ya puede pasar, pero que pasaria si antes de que el que estaba en la cola se de cuenta q puede pasar llega otro proceso? va a encontrar a cant en 0 y NO se va a dormir va a pasar y va a empezar a cruzar el puente, y ponele que en ese momento justo el que estaba en wait se despierta ? va a pasar a cruzar el puente , y van a estar los dos cruzando.

    b) Si, se puede simplificar haciendo que el monitor represente el cruce del puente. Esto se debe a que solo se necesita exclusion mutua y esto de por si lo proveen los monitores.


Monitor Puente {
    Procedure cruzarPuente (){
        //el auto cruza el puente
    }
}

Process Auto [a:1..M]{
    Puente.cruzarPuente();
}

c) No, no lo respeta, ya que cuando un auto se despierta, pasa a competir con el resto de autos para pasar al puente. La b. tampoco lo respeta ya que todos los autos van a competir por el uso del monitor.

```

## Ejercicio 2

```c

a)Se necesita un monitor base de datos que tengas dos procesos uno leer y otro terminar y un process de 1..n de Usuario

b)
Monitor BaseDeDatos{
    cond espera;// variable condicion donde se quedan esperando los procesos
    int cant = 0;
    Procedure leer(){
        while(cant > 4 ){// (0 1 2 3 4 osea 5 en total),si es mayor a cuatro quiere decir que es el 6 usuario, septimo o asi siguiendo.
            wait(espera);
        }
        cant++;
    }
    Procedure terminarLectura(){
        cant--;
        signal(espera);// despierto a alguien si estaba esperando , sino hay nadie no pasa nada
    }
}

Process Usuario [u:1..N]{
    BaseDeDatos.leer();
    //Leer
    BaseDeDatos.terminarLectura();
}
```

```

```
