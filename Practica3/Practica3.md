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

a)El codigo tiene un error, suponete que viene 2 procesos a entrarPuente(); uno de los dos va a encontrar a cant en 0 y va a pasar y el otro se va a quedar dormido en el wait cola , el proceso 1 cruza el puente y sale, dejando a cant en 0 y avisandole al que estaba en la cola q ya puede pasar, pero que pasaria si antes de que el que estaba en la cola se de cuenta q puede pasar llega otro proceso? va a encontrar a cant en 0 y NO se va a dormir va a pasar y va a empezar a cruzar el puente, y ponele que en ese momento justo el que estaba en wait se despierta ? va a pasar a cruzar el puente , y van a estar los dos cruzando.

b) Si, se puede simplificar haciendo que el monitor represente el cruce del puente. Esto se debe a que solo se necesita exclusion mutua y esto de por si lo proveen los monitores.

```c
Monitor Puente {
    Procedure cruzarPuente (){
        //el auto cruza el puente
    }
}

Process Auto [a:1..M]{
    Puente.cruzarPuente();
}

```

c) No, no lo respeta, ya que cuando un auto se despierta, pasa a competir con el resto de autos para pasar al puente. La b. tampoco lo respeta ya que todos los autos van a competir por el uso del monitor.

## Ejercicio 2

Existen N procesos que deben leer información de una base de datos, la cual es administrada por un motor que admite una cantidad limitada de consultas simultáneas.

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

## Ejercicio 3

Existen N personas que deben fotocopiar un documento. La fotocopiadora sólo puede ser usada por una persona a la vez. Analice el problema y defina qué procesos, recursos y monitores serán necesarios/convenientes, además de las posibles sincronizaciones requeridas para resolver el problema. Luego, resuelva considerando las siguientes situaciones:

- a) Implemente una solución suponiendo no importa el orden de uso. Existe una función
  Fotocopiar() que simula el uso de la fotocopiadora.
- b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.
- c) Modifique la solución de (b) para el caso en que se deba dar prioridad de acuerdo con la
  edad de cada persona (cuando la fotocopiadora está libre la debe usar la persona de mayor
  edad entre las que estén esperando para usarla).
- d) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden
  dado por el identificador del proceso (la persona X no puede usar la fotocopiadora hasta
  que no haya terminado de usarla la persona X-1).
- e) Modifique la solución de (b) para el caso en que además haya un Empleado que le indica
  a cada persona cuando debe usar la fotocopiadora.
- f) Modificar la solución (e) para el caso en que sean 10 fotocopiadoras. El empleado le indica
  a la persona cuál fotocopiadora usar y cuándo hacerlo.

```c
a)
Monitor Fotocopiadora
{
    Procedure sacarFotocopia(){
        fotocopiar();// la impresora esta fotocopiando
    }
}

process Persona [id:0..N-1]
{
    Fotocopiadora.sacarFotocopia();
}

b)
Monitor Fotocopiadora{
    cond espera;
    int cant = 0;
    int cantEsperan = 0;

    Procedure fotocopiar(){
        if(cant == 1 ){
            cantEsperan++;
            wait(espera);
        }
        else{
            cant++;
        }

         Fotocopiar();
    }
    Procedure terminarFotocopiar(){
        if (cantEsperan > 0){
            cantEsperan--;
            signal(espera);
        }
        else{
            cant--;
        }
    }
}


Process Persona [u:1..N-1]{
    Fotocopiadora.fotocopiar();
    Fotocopiadora.terminarFotocopiar();
}

c)
Monitor Fotocopiadora{
    cond espera [];
    int cant = 0;
    colaOrdenada cola;

    Procedure fotocopiar(int idPersona, int edad){

        if(cant == 1 ){
            cola.push(idPersona,edad)//se pushea ordenado de menor a mayor
            wait(espera[id]);
        }
        else{
            cant++;
        }
        Fotocopiar();
    }
    Procedure terminarFotocopiar(){
        if (!cola.isEmpty()){
            pop.cola(id,edad);
            signal(espera[id]);
        }
        else{
            cant--;
        }
    }
}


Process Persona [id:1..N-1]{
    int edad;
    edad=Persona.edad();//obtengo edad.

    Fotocopiadora.fotocopiar(u,edad);
    Fotocopiadora.terminarFotocopiar();
}

d)

Monitor Fotocopiadora
{
    cond espera[];
    cond llego[];
    int cant = 0;


    Procedure fotocopiar(int id)
    {
        //signal(llego[id]);// esto se hace para que el terminaroFotocopiar solo haga un signal si hay alguien esperando en wait espera
        if(id != 0 )
        {
            wait(espera[id]);
        }

    }

    Procedure terminarFotocopiar(int idPersona)
    {

        signal(espera[idPersona++])
    }
}


Process Persona [id:0..N-1]{
    Fotocopiadora.fotocopiar(id);
    //FOTOCOPIAR
    Fotocopiadora.terminarFotocopiar(id);
}

e)
cond esperandoPersonas[];
cola Personas[N-1];
cond esperoAqueLLegueAlguien;
bool llegue;

Monitor Fotocopiadora{
    cond espera;
    int cant = 0;
    int cantEsperan = 0;

    Procedure fotocopiar(){
        if(cant == 1 ){
            cantEsperan++;
            wait(espera);
        }
        else{
            cant++;
        }

    }
    Procedure terminarFotocopiar(){
        if (cantEsperan > 0){
            cantEsperan--;
            signal(espera);
        }
        else{
            cant--;
        }
    }
}
Monitor personaLlega
{
    procedure llegueAFotocopiar(int id)
    {
        llegue=true;
        Personas.push(id);
        signal(esperoAqueLLegueAlguien)
        wait(esperandoPersonas[id]);
    }

}

Process Persona [u:1..N-1]
{
    personaLlega.llegueAFotocopiar(u);
    Fotocopiadora.fotocopiar();
    //fotocopiar
    Fotocopiadora.terminarFotocopiar();
}

Process Empleado
{
    while (true)
    {
        if(!llegue)
        {
            wait(esperoAqueLLegueAlguien)
            llegue=false;
        }
        if(!Personas.isEmpty())
        {
            signal[Personas.pop(id)]
        }
    }
}

```
