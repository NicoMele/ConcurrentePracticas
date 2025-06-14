# Práctica 2 - Semáforos

## Ejercicios:

- [Ejercicio 1](#ejercicio-1)
- [Ejercicio 2](#ejercicio-2)
- [Ejercicio 3](#ejercicio-3)
- [Ejercicio 4](#ejercicio-4)
- [Ejercicio 5](#ejercicio-5)
- [Ejercicio 6](#ejercicio-6)
- [Ejercicio 7](#ejercicio-7)
- [Ejercicio 8](#ejercicio-8)
- [Ejercicio 9](#ejercicio-9)
- [Ejercicio 10](#ejercicio-10)
- [Ejercicio 11](#ejercicio-11)
- [Ejercicio 12](#ejercicio-12)

## Ejercicio 1

Existen N personas que deben ser chequeadas por un detector de metales antes de poder
ingresar al avión.

a. Analice el problema y defina qué procesos, recursos y semáforos serán
necesarios/convenientes, además de las posibles sincronizaciones requeridas para
resolver el problema.

b. Implemente una solución que modele el acceso de las personas a un detector (es decir,
si el detector está libre la persona lo puede utilizar; en caso contrario, debe esperar).

c. Modifique su solución para el caso que haya tres detectores.

```c
a) y b)

sem Libre = 1;

Processs Persona [0..N]
{

	P(Libre)
	//usa detector
	V(Libre)

}
```

```c
c)
sem Libre = 3;
Processs Persona [0..N]
{

	P(Libre)
	//usa detector
	V(Libre)

}

```

## Ejercicio 2

Un sistema de control cuenta con 4 procesos que realizan chequeos en forma colaborativa. Para ello, reciben el historial de fallos del día anterior (por simplicidad, de tamaño N). De cada fallo, se conoce su número de identificación (ID) y su nivel de gravedad (0=bajo, 1=intermedio, 2=alto, 3=crítico). Resuelva considerando las siguientes situaciones:

a. Se debe imprimir en pantalla los ID de todos los errores críticos (no importa el orden).

b. Se debe calcular la cantidad de fallos por nivel de gravedad, debiendo quedar los resultados en un vector global.

c. Ídem b. pero cada proceso debe ocuparse de contar los fallos de un nivel de gravedad determinado

```c
int cant = 0; sem mutex=1; colaDefallos cola[N] ; array vectorGlobal[3]= ([3] 0); sem semNivel[3] = ([3] 1);


a)
	Process fallos [0..3]
	{
		Fallo fallo ;

		P(mutex) //este p mutex es para que no pueda haber dos procesos queriendo sumar cant al mismo tiempo  y evitar el error de hacer un pop a algo q no existe

		while(cant < N )
		{
				fallo=colaDefallos.pop();
				cant++;
				V(mutex)

				if(fallo.getNivel() == 3)
				{
					print(fallo.getId());
				}

				P(mutex);//este p mutex es para que no pueda haber dos procesos queriendo sumar cant al mismo tiempo

		}

		V(mutex)//volvemos a poner mutex en 1 para que no quede en 0 y se bloqueen todos los procesos en el P(mutex) de arriba o el de abajo.

	}
```

```c
b)
	Process fallos [0..3]
	{
		Fallo fallo ;

		P(mutex) //este p mutex es para que no pueda haber dos procesos queriendo sumar cant al mismo tiempo  y evitar el error de hacer un pop a algo q no existe

		while(cant < N )
		{

				fallo=colaDefallos.pop();
				cant++;
				V(mutex)

				if(fallo.getNivel() == 3)
				{
					print(fallo.getId());
				}

				P(semNivel[fallo.getNivel()]) // se hace semaforo de niveles porque no hay problema que varios procesos quieran sumar en distintos niveles. Por eso no se usa un semaforo solo.

				vectorGlobal[fallo.getNivel()]=vectorGlobal[fallo.getNivel()] + 1;

			    V(semNivel[fallo.getNivel()])

				P(mutex);//este p mutex es para que no pueda haber dos procesos queriendo sumar cant al mismo tiempo

		}

		V(mutex)//volvemos a poner mutex en 1 para que no quede en 0 y se bloqueen todos los procesos en el P(mutex) de arriba o el de abajo.
	}
```

```c
c)

Process fallos [i=0..3]
	{
		Fallo fallo ;

		P(mutex) //este p mutex es para que no pueda haber dos procesos queriendo sumar cant al mismo tiempo y evitar el error de hacer un pop a algo q no existe

		while(cant < N )
		{

				fallo=colaDefallos.pop();
				cant++;
				V(mutex)

				if(fallo.getNivel() == 3)
				{
					print(fallo.getId());
				}

				if(fallo.getNivel() == i)
				{

					vectorGlobal[i]=vectorGlobal[i] + 1; // no es necesario el semaforo de niveles ya que no puede haber dos procesos sumando uno en la misma posicion del vector.

				}
				else
				{
					P(mutex)
					colaDefallos.push(fallo);
					cant--;
					V(mutex)
				}

				P(mutex);//este p mutex es para que no pueda haber dos procesos queriendo sumar cant al mismo tiempo y evitar el error de hacer un pop a algo q no existe

		}

		V(mutex)//volvemos a poner mutex en 1 para que no quede en 0 y se bloqueen todos los procesos en el P(mutex) de arriba o el de abajo.
	}

}
```

## Ejercicio 3

Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola. Además, existen P procesos que necesitan usar una instancia del recurso.
Para eso, deben sacar la instancia de la cola antes de usarla. Una vez usada, la instancia debe ser encolada nuevamente.

```c
sem mutex=1,entranSolo5=5;  colaDeRecursos cola[5];

Process Recursos [0..P]
{
	Recurso recurso;

	while(true)
	{
		P(entranSolo5)
		P(mutex)
		recurso= colaDeRecursos.pop();// ESTO ES PORQUE PODRÍAN HABER 2 o MAS PROCESOS INTENTANDO SACAR EL MISMO RECURSO DE LA COLA
		V(mutex)
		//usar el recurso
		P(mutex)
		colaDeRecursos.push(recurso);// ESTO ES PORQUE PODRIAN HABER 2 o MAS PROCESOS INTENTANDO PONER EL MISMO RECURSO DE LA COLA
		V(mutex)
		V(entranSolo5)
   }
}
```

## Ejercicio 4

Suponga que existe una BD que puede ser accedida por 6 usuarios como máximo al mismo tiempo. Además, los usuarios se clasifican como usuarios de prioridad alta
y usuarios de prioridad baja. Por último, la BD tiene la siguiente restricción:

no puede haber más de 4 usuarios con prioridad alta al mismo tiempo usando la BD.
no puede haber más de 5 usuarios con prioridad baja al mismo tiempo usando la BD.
Indique si la solución presentada es la más adecuada. Justifique la respuesta.

```c
Var
sem: semaphoro := 6;
alta: semaphoro := 4;
baja: semaphoro := 5
```

```c
Process Usuario-Alta [I:1..L]::
 {

	P (sem);
	P (alta);
	//usa la BD
	V(sem);
	V(alta);
 }
```

```c
 Process Usuario-Baja [I:1..K]::
 {
	P (sem);
	P (baja);
	//usa la BD
	V(sem);
	V(baja);
 }
```

```c

# Respuesta:

Pimero estan mal declaradas la variables, deberia ser :

sem sem  = 6;
sem alta = 4;
sem baja = 5

Una mejor solución seria hacer las P(alta) y P(baja) primero que P(sem):

Process Usuario-Alta [I:1..L]::
 {
	P (alta);
	P (sem);
	//usa la BD
	V(sem);
	V(alta);
 }

 Process Usuario-Baja [I:1..K]::
 {
	P (baja);
	P (sem);
	//usa la BD
	V(sem);
	V(baja);
 }

 Como estaba antes , si venian 6 procesos de prioridad ALTA , los 6 iban a pasar el P(sem), cuando pasen por P(alta)
 de esos 6 que pasaron 4 solo van a poder ingresar a usar la DB por lo tanto si venian 2  usuarios de baja prioridad por ejemplo, y querian hacer su P(sem) no iban a poder acceder
 y estarias disminuyendo la concurrencia ya que si los de alta habia 4 ejecutandose los 2 de baja que vinieron luego tendrian que haber podido acceder a la base de datos
 para usarla tambien.

```

## Ejercicio 5

En una empresa de logística de paquetes existe una sala de contenedores donde se preparan las entregas. Cada contenedor puede
almacenar un paquete y la sala cuenta con capacidad para N contenedores. Resuelva considerando las siguientes situaciones:

a) La empresa cuenta con 2 empleados: un empleado Preparador que se ocupa de preparar los paquetes y dejarlos en los contenedores;
un empelado Entregador que se ocupa de tomar los paquetes de los contenedores y realizar la entregas. Tanto el Preparador como el Entregador trabajan de a un paquete por vez.

b) Modifique la solución a) para el caso en que haya P empleados Preparadores.

c) Modifique la solución a) para el caso en que haya E empleados Entregadores.

d) Modifique la solución a) para el caso en que haya P empleados Preparadores y E empleadores Entregadores.

```c
a)

Paquete contenedores[N];
sem cantidadDeLugaresVacios = N;
sem hayPaquetes = 0;
int punteroEntregador= 0,punteroPreparador = 0;

Process Preparador
{
	while(true)
	{
	    //preparar paquete

		P(cantidadDeLugaresVacios)
		contenedores[punteroPreparador]= paquete
        punteroPreparador= (punteroPreparador + 1) mod N;

		V(hayPaquetes)

	}
}

Process Entregador
{
	while(true)
	{
		P(hayPaquetes)

		paquete = contenedores[punteroEntregador]
		punteroEntregador= (punteroEntregador + 1) mod N;

		V(cantidadDeLugaresVacios)
		//entregar paquete
	}
}
```

```c
b)

Paquete contenedores[N];
sem cantidadDeLugaresVacios = N;
sem hayPaquetes = 0;
int punteroEntregador= 0,punteroPreparador = 0;
sem mutexP=1;

Process Preparador [0..P-1]
{
	while(true)
	{
	    //preparar paquete
		P(cantidadDeLugaresVacios)
		P(mutexP)
		contenedores[punteroPreparador]= paquete
        punteroPreparador= (punteroPreparador + 1) mod N;
		V(mutexP)

		V(hayPaquetes)

	}
}

Process Entregador
{
	while(true)
	{
		P(hayPaquetes)

		paquete = contenedores[punteroEntregador]
		punteroEntregador= (punteroEntregador + 1) mod N;

		V(cantidadDeLugaresVacios)
		//entregar paquete
	}
}
```

```c
c)

Paquete contenedores[N];
sem cantidadDeLugaresVacios = N;
sem hayPaquetes = 0;
int punteroEntregador= 0,punteroPreparador = 0;
sem mutexE=1;

Process Preparador [0..P-1]
{

	while(true)
		{
		//preparar paquete
		P(cantidadDeLugaresVacios)
		contenedores[punteroPreparador]= paquete
		punteroPreparador= (punteroPreparador + 1) mod N;
		V(hayPaquetes)
		}
}

Process Entregador [0..E-1]
{
		while(true)
		{
		P(hayPaquetes)
		P(mutexE)
		paquete = contenedores[punteroEntregador]
		punteroEntregador= (punteroEntregador + 1) mod N;
		V(mutexE)
		V(cantidadDeLugaresVacios)
		//entregar paquete
		}
}
```

```c
d)

Paquete contenedores[N];
sem cantidadDeLugaresVacios = N;
sem hayPaquetes = 0;
int punteroEntregador= 0,punteroPreparador = 0;
sem mutexP=1;
sem mutexE=1;

Process Preparador [0..P-1]
{
		while(true)
		{
		//preparar paquete
		P(cantidadDeLugaresVacios)
		P(mutexP)
		contenedores[punteroPreparador]= paquete
		punteroPreparador= (punteroPreparador + 1) mod N;
		V(mutexP)
		V(hayPaquetes)
		}
}

Process Entregador [0..E-1]
{
		while(true)
		{
		P(hayPaquetes)
		P(mutexE)
		paquete = contenedores[punteroEntregador]
		punteroEntregador= (punteroEntregador + 1) mod N;
		V(mutexE)
		V(cantidadDeLugaresVacios)
		//entregar paquete
		}
}
```

## Ejercicio 6

Existen N personas que deben imprimir un trabajo cada una. Resolver cada ítem usando semáforos:

a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y
las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la
persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.

b) Modifique la solución de a) para el caso en que se deba respetar el orden de llegada.

c) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso
(la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1).

d) Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora.

e) Modificar la solución (d) para el caso en que sean 5 impresoras. El coordinador le indica a la persona cuando puede usar una impresora, y cual debe usar.

```c
a)
sem mutex=1;
Process Personas [0..N-1]
{
	Documento documento;
	P(mutex)
	Imprimir(documento)
	V(mutex)
}
```

```c
b)
sem mutex=1; colaDeLlegada cola[N] ,sem espera[N]; bool libre=true;
Process Personas [id:0..N-1]
{
Documento documento;
int aux;
	P(mutex)
	if(libre)
	{
		libre=false;
		V(mutex);
	}
	else
	{
		cola.push(id)//encolo id del proceso
		V(mutex)
		P(espera[id])
	}

	Imprimir(documento)

	P(mutex)// por si entra otra persona , yo no puedo estar chequeando si la cola esta vacia al mismo tiempo que otro tal vez se esta encolando, por eso esta este mutex
	if(!cola.isEmpty())
	{
		aux=pop.cola();//el pop me devuelve el id del proceso
		V(espera[aux])
	}
	else
	{
		libre=true;
	}
	V(mutex)
}
```

```c
c)
int proximo = 0;
sem espera[P] = ([P] 0);

Process persona [id:0..N-1]{//el que tiene id 0 despierta al que tiene id 1, el que tiene id 1 al que tiene id 2 y asi.
    Documento documento;
    if (id != 0){
        P(espera[i]);
    }
    Imprimir(documento);
    V(espera[id+1]);
}
```

```c
d)

sem mutex=1; colaDeLlegada cola[N] ,sem espera[N]= ([N] 0);
sem termino=0; sem lleno = 0;

Process Personas [id:0..N-1]
{
	Documento documento;
	P(mutex)
	cola.push(id)//encolo id del proceso
	V(mutex)
	V(lleno)
	P(espera[id])
	Imprimir(documento)
	V(termino)
}

Process Coordinador
{
	int aux;
	while (true)
	{
		P(lleno)
		P(mutex)
		aux=cola.pop();
		V(mutex)
		V(espera[aux])
		P(termino)
	}

}
```

```c
e)

sem mutex=1; colaDeLlegada cola[N] ,sem espera[N]= ([N] 0) ;
sem lleno = 0; array impresoraACadaPersona = ([N] -1) ; sem CantImpresoras=5;
int impresoras [5] = {0,1,2,3,4}
sem mutexImpresora=1;

Process Personas [id:0..N-1]
{
	Documento documento;
	P(mutex)
	cola.push(id)//encolo id del proceso
	V(mutex)
	V(lleno)
	P(espera[id])

	Imprimir(documento,impresoraACadaPersona[id])

	P(mutexImpresora)
	impresoras.push(impresoraACadaPersona[id]);
	V(mutexImpresora)

	V(CantImpresoras)//avisa que hay otra impresora disponible
}

Process Coordinador
{
	int numeroDeImpresora;
	int idPersona;
	while (true)
	{
		P(lleno)
		P(mutex)
		idPersona=cola.pop();//saco id de la persona
		V(mutex)
		P(CantImpresoras)//para que pase 5 veces porque hay 5 impresoras.
		P(mutexImpresora)
		numeroDeImpresora=impresoras.pop();
		impresoraACadaPersona[idPersona]=numeroDeImpresora
		V(mutexImpresora)
		V(espera[idPersona])
	}
}
```

## Ejercicio 7

Suponga que se tiene un curso con 50 alumnos. Cada alumno debe realizar una tarea y
existen 10 enunciados posibles. Una vez que todos los alumnos eligieron su tarea,
comienzan a realizarla. Cada vez que un alumno termina su tarea, le avisa al profesor y se
queda esperando el puntaje del grupo, el cual está dado por todos aquellos que comparten
el mismo enunciado. Cuando un grupo terminó, el profesor les otorga un puntaje que
representa el orden en que se terminó esa tarea de las 10 posibles.

Nota: Para elegir la tarea suponga que existe una función elegirTarea() que le asigna una tarea a un alumno (esta función asignará 10 tareas diferentes entre 50 alumnos, es decir, que 5 alumnos tendrán la tarea 1, otros 5 la tarea 2 y así sucesivamente para las 10 tareas).

```c
sem mutex=1;sem barrera=0;
int contador=0;
sem seDioPuntaje[10] = ([10] 0);
int puntajeTarea[10] = ([10] 0);
cola finalizadas;
sem termineTarea=0;

Process Alumnos [id=0..49]
{
	int tarea; int puntaje; int j;

	Tarea tarea = elegirTarea();//obtengo numero de tarea
	P(mutex)

	contador = contador + 1;

	if (contador == 49)// si contador es 49 (osea el ultimo) es porque ya llegaron todos los alumnos y eligieron su tarea.
	{ for i = 0..49 → V(barrera);}

	V(mutex);
    P(barrera);
    //realizar tarea
    P(mutex);
    finalizadas.push(tarea);//encolo los numeros de las tareas a medida que van terminando.
    V(mutex);
    V(termineTarea);
    P(seDioPuntaje[tarea]);
    puntaje = puntajeTarea[tarea];
}

Process profesor
{
    int puntaje=10; int contadorTarea[10] = ([10] 0); int j; int tarea; int i;
    for j = 0..49{
        P(termineTarea);
        P(mutex);
        tarea = finalizadas.pop();
        V(mutex);
        contadorTarea[tarea]++;
        if (contadorTarea[tarea] == 5){
            puntajeTarea[tarea] = puntaje;
            for i = 0..4 -> V(seDioPuntaje[tarea]);
            puntaje--;
        }
    }
}
```

## Ejercicio 8

Una fábrica de piezas metálicas debe producir T piezas por día. Para eso, cuenta con E
empleados que se ocupan de producir las piezas de a una por vez (se asume T>E). La
fábrica empieza a producir una vez que todos los empleados llegaron. Mientras haya
piezas por fabricar, los empleados tomarán una y la realizarán. Cada empleado puede
tardar distinto tiempo en fabricar una pieza. Al finalizar el día, se le da un premio al
empleado que más piezas fabricó.

```c
sem barreraEmpleados=0; int cantPiezas = 0; array empleados=([E] -1);
int contador=0;
sem mutex=1;
sem seFinalizo=0;
int termino=0;
int premio=-1;
sem despiertoEmpresa=0;
sem esperoPremio=0;

Process Empleados [id=0..E-1]
{
	P(mutex)
	contador = contador + 1;
	if (contador == E-1)/
	{ for i = 0..E-1 → V(barreraEmpleados);}
	V(mutex)
	P(barreraEmpleados)
	P(mutex)
	while(cantPiezas < T)
	{
		//HACER PIEZA
		cantPiezas++;
		V(mutex);
		empleados[id]=empleados[id] + 1 ;
		P(mutex)
	}
	V(mutex)

	P(seFinalizo)
	termino++;
	if(termino == (E-1))//si es el ultimo
	{
		V(despiertoEmpresa)
	}
	V(seFinalizo)
	P(esperoPremio)
	if(premio == id)
	{
		//gane premio
	}

}

Process Empresa
{
    int j;
    P(despiertoEmpresa);
   	premio = empleados.Max().getId();//devuelve el id del empleado con mayor puntaje.
    for j = 0..E-1 -> V(esperoPremio);
}
```

## Ejercicio 9

Resolver el funcionamiento en una fábrica de ventanas con 7 empleados (4 carpinteros, 1 vidriero y 2 armadores) que trabajan de la siguiente manera:

- Los carpinteros continuamente hacen marcos (cada marco es armando por un único carpintero) y los deja en un depósito con capacidad de almacenar 30 marcos.

- El vidriero continuamente hace vidrios y los deja en otro depósito con capacidad para 50 vidrios.

- Los armadores continuamente toman un marco y un vidrio (en ese orden) de los depósitos correspondientes y arman la ventana (cada ventana es armada por un único armador).

```c

sem depositoMarcos=30;
sem depositoVidrios=50;
sem mutexCarpinteros=1;
sem mutexVidriero=1;
cola colaMarcos;
cola colaVidrios;
sem llenoMarco=0;
sem llenoVidrios=0;
sem mutexArmador
Process Carpintero[idCarpintero=0..3]
{
	Marco marco;
	while(true)
	{
		P(depositoMarcos)
		marco=ArmaMarco();
		P(mutexCarpinteros)
		colaMarcos.push(marco)
		V(mutexCarpinteros)
		V(llenoMarco)
	}

}

Process Vidriero
{
	Vidrio vidrio;
	while(true)
	{
		P(depositoVidrios)
		vidrio=ArmaVidrio();
		P(mutexVidriero)
		colaVidrios.push(vidrio)
		V(mutexVidriero)
		V(llenoVidrios)
	}

}

Process Armador[idArmador=0..1]
{
	Marco marco;
	Vidrio vidrio;
		while(true)
		{
			P(llenoMarco)
			P(mutexCarpinteros)
			marco=colaMarcos.pop();
			V(mutexCarpinteros)
			V(depositoMarcos)

			P(llenoVidrio)
			P(mutexVidriero)
			vidrio=colaVidrio.pop();
			V(mutexVidriero)
			V(depositoVidrios)

			P(mutexArmador)
			//arma ventana
			V(mutexArmador)
		}
}
```

## Ejercicio 10

A una cerealera van T camiones a descargar trigo y M camiones a descargar maíz. Sólo hay lugar para que 7 camiones a la vez descarguen, pero no pueden ser más de 5 del mismo tipo de cereal.

Nota: no usar un proceso extra que actué como coordinador, resolverlo entre los camiones.

```c

sem soloEntran7=7;
sem soloHasta5Trigo=5;
sem soloHasta5Maiz=5;

Process CamionesTrigo[0..T-1]
{
	P(soloHasta5Trigo)
	P(soloEntran7)
	//descargar
	V(soloEntran7)
	V(soloHasta5Trigo)
}

Process CamionesMaiz[0..M-1]
{
	P(soloHasta5Maiz)
	P(soloEntran7) //1
	//descargar
	V(soloEntran7)
	V(soloHasta5Maiz)
}

```

## Ejercicio 11

En un vacunatorio hay un empleado de salud para vacunar a 50 personas. El empleado de salud atiende a las personas de acuerdo con el orden de llegada y de a 5 personas a la vez. Es decir, que cuando está libre debe esperar a que haya al menos 5 personas esperando, luego vacuna a las 5 primeras personas, y al terminar las deja ir para esperar por otras 5. Cuando ha atendido a las 50 personas el empleado de salud se retira.

Nota: todos los procesos deben terminar su ejecución; asegurarse de no realizar Busy Waiting; suponga que el empleado tienen una función VacunarPersona() que simula que el empleado está vacunando a UNA persona.

```c
sem personasALaVez=5;
sem barrera5=0;
int contador=0;
cola personas;
sem mutexPersona=1;
sem despierto=0;
array esperaASerVacunado = ([50] 0);

Process Personas [id=0..49]
{
	P(mutexPersona)
	personas.push(id);
	contador = contador + 1;
	if (contador == 5)
	{
		contador=0;
		V(despierto)
	}
	V(mutexPersona)
	P(esperaASerVacunado[id])
}

Process Empleado
{
	cola Vacunados;
	int id;
	for z = 0 to 9//si hago esto 10 veces, atiendo 10 veces a 5 pacientes por lo tanto atendi a todos.
	{
		P(despierto)
		for i= 0 to 4
		{
			id=personas.pop();
			VacunarPersona(id);
			Vacunados.push(id);//pusheo ids de vacunados.
		}
		for j= 0 to 4
		{
			V(esperaASerVacunado[Vacunados.pop()]);
		}
	}
}

```

## Ejercicio 12

Simular la atención en una Terminal de Micros que posee 3 puestos para hisopar a 150
pasajeros. En cada puesto hay una Enfermera que atiende a los pasajeros de acuerdo
con el orden de llegada al mismo. Cuando llega un pasajero se dirige al puesto que tenga
menos gente esperando. Espera a que la enfermera correspondiente lo llame para
hisoparlo, y luego se retira.

Nota: sólo deben usar procesos Pasajero y Enfermera.
Además, suponer que existe una función Hisopar() que simula la atención del pasajero por
parte de la enfermera correspondiente.

```c

sem esperan[3] = ([150] 0);

sem mutex=1;

sem lleganAlPuesto[3]=([3] 0);

cola colaDePuestos[3]; //Array de colas.

int hisopados=0;
sem mutexHisopado=1;

Process Pasajero [idPasajero=0..149]
{
	int puesto;
	P(mutex)
	puesto= chequearPuestoConMenosPersonas(colaDePuestos);//devuelve el indice del array que tiene menos personas.
	colaDePuestos[puesto].push(idPasajero);// pusheo dependiendo el numero de puesto en una cola u otra
	V(mutex)
	V(lleganAlPuesto[puesto])
	P(esperan[idPasajero])
	//estas siendo hispado

}

Process Enfermera [idEnfermera=0..2]
{
	int idPasajero;
	int i;
	while(hisopados < 150)
	{
		P(lleganAlPuesto[idEnfermera])
		if(!colaDePuestos.isEmpty())
		{
			P(mutex)
			idPasajero=colaDePuestos[idEnfermera].pop();
			V(mutex)
			Hisopar(idPasajero);
			V(esperan[idPasajero])
			P(mutexHisopado)
			hisopados++;
			if (hisopados == 150)
			{
				for(i= 0 to 2)
				{
					V(lleganAlPuesto[i])
				}
			}
			V(mutexHisopado)
		}
	}
}

```
