1)  ¿Qué función requiere agregar -DTRAPFPE? ¿Cómo pueden hacer que el programa linkee adecuadamente?
	La funcion test_fpe3.c requiere tener definida la variable TRAPFPE, que puede ser activada con -DTRAPFPE, para incluir el header y la funcion codificada en fpe_x87_sse.c
	Para que linkee adecuadamente incluimos DTRAPFPE para activar esa variable.

    Para cada uno de los ejecutables, ¿qué hace agregar la opción -DTRAPFPE al compilar? ¿En qué se diferencian los mensajes de salida con y sin esa opción?
	Con esa opcion los mensajes de salida devuelven errores mas descripctivos.


2)  ¿Devuelven el mismo error que antes? Averigüen qué hicieron al ejecutar la sentencia ulimit -s unlimited. Algunas pistas son: abran otra terminal distinta y fíjense si vuelve al mismo error, fíjense la diferencia entre ulimit -a antes y después de ejecutar ulimit -s unlimited, googleen, etcétera.
	No, el error se "soluciona" tras ejecutar esa orden. Lo que creemos que hace es ilimitar el tamaño del stack, posibilitando reservar en el mismo la gran cantidad de memoria requerida por big.x

    La "solución" anterior, ¿es una solución en el sentido de debugging?
	No es una solucion en sentido de debugging porque no se modifica el codigo fuente. Una solucion podria ser asignar memoria dinamicamente

VALGRID

$ valgrind ./a.out
==12469== Memcheck, a memory error detector
==12469== Copyright (C) 2002-2013, and GNU GPL'd, by Julian Seward et al.
==12469== Using Valgrind-3.10.0.SVN and LibVEX; rerun with -h for copyright info
==12469== Command: ./a.out
==12469== 


