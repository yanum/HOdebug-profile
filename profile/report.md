#Profiling

Este ejercicio es muy libre. Compilen los códigos de C o FORTRAN
con distintas opciones de optimización (`-O0`, `-O1`, `-O3`) y ejecútenlo con distintas
formas de hacer un profiling: desde `time ./programa.e` hasta `perf`,
pasando por `gprof` y cualquier otra cosa que se les ocurra. Escriban
un pequeño reporte en `reporte.md` respecto de qué cosas hicieron y
cómo mejoraron/empeoraron los tiempos. 





1) compile el codigo de c profile_me_1.c
# COMPILO SIN NUNGUNA OPTIMIZACION -O0
$ gcc profile_1.c # de esta forma se compila sin ningun flag y se linkea con si mismo

$ gcc -c profile_1.c -g -O0  # compilo solo hasta crear el objeto (hasta el paso antes de linkear con el parametro -c), -g agrega flags de debug (por ejemplo me dice en que linea se produce el error) -O0 no optimiza (nivel cero de Optimizacion) Solamente creo el objeto

$ nm profile_1.o #con nm miro si quedo algun problema de dependencia, en este caso no (no hay nada No definido)
0000000000000000 T first_assign
000000000000015e T main
00000000000000af T second_assign

#como esta ok, puedo crear el ejecutable linkeandolo a si mismo, no necesita ser linkeado con otro objeto
$ gcc profile_1.o -o profile_1.e  # con -o escribo el nombre de la salida

# corro el programa, sale error de segmentation fault
# corro el debugger gdb 

Program received signal SIGSEGV, Segmentation fault.
0x0000000000400656 in main (
    argc=<error de lectura de variable: No se puede acceder a la memoria en la dirección 0x7fffdc3c901c>, 
    argv=<error de lectura de variable: No se puede acceder a la memoria en la dirección 0x7fffdc3c9010>) at profile_1.c:17
17  

# miro el codigo en .c , El problema pide demasiado espacio de memoria para la matriz (5000 x 5000), cambio el tamaño corrijo el error.
#ahora el programa corre.

# PROFILING . mido el tiempo que tarda el correr con NIVEL DE OPTIMIZACION 0
# En primer instancia lo analizo con  time
$ time ./profile_1.e

real  0m0.635s
user	0m0.599s
sys	0m0.036s


#PERF - mide el funcionamiento del ejecutable a nivel de hardware
$ perf stat ./profile_1.e

 Performance counter stats for './profile_1.e':

        635,403996 task-clock (msec)         #    0,995 CPUs utilized          
                58 context-switches          #    0,091 K/sec                  
                 5 cpu-migrations            #    0,008 K/sec                  
               967 page-faults               #    0,002 M/sec                  
     2.300.611.328 cycles                    #    3,621 GHz                    
     1.138.314.667 stalled-cycles-frontend   #   49,48% frontend cycles idle   
       557.460.386 stalled-cycles-backend    #   24,23% backend  cycles idle   
     3.591.012.010 instructions              #    1,56  insns per cycle        
                                             #    0,32  stalled cycles per insn
       177.698.365 branches                  #  279,662 M/sec                  
            27.084 branch-misses             #    0,02% of all branches        

       0,638718073 seconds time elapsed
#
$ perf record ./profile_1.c

[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.098 MB perf.data (~4289 samples) ]


$perf report -i perf.data
 42,28%  profile_1.e  profile_1.e        [.] first_assign
 26,21%  profile_1.e  profile_1.e        [.] second_assign
 24,20%  profile_1.e  profile_1.e        [.] main
  7,30%  profile_1.e  [kernel.kallsyms]  [k] 0xffffffff8104f45a


# COMPILO CON OPTIMIZACION NIVEL 1 -O1
$ gcc -c profile_1.c -g -O1 ( -c frena la compilacion antes de hacer el linkeo, -g agrega los flags de gdb, que en este caso no son necesarios porque el debugg se hace en optimizacion nivel 0 porque el simple hecho de pasar a optimizacion nivel 1 el codio ya no es exactamente igual y esos flags no me van a servir)

$ nm profile_1.o
0000000000000000 T first_assign
0000000000000000 r .LC0
0000000000000008 r .LC1
0000000000000010 r .LC2
0000000000000046 T main
0000000000000023 T second_assign #la diferencia en el primer nivel de optimizacion es que aparecen los elementos .LC

$ gcc profile_1.o -o profile_1.e #creo el ejecutable linkeado a si mismo.
$ time ./profile_1.e

real	0m0.364s
user	0m0.317s
sys	0m0.044s # luego del primer nivel de optimizacion el tiempo real de ejecucion disminuyo aprox a la mitad


$ perf stat ./profile_1.e

 Performance counter stats for './profile_1.e':

        357,160401 task-clock (msec)         #    0,991 CPUs utilized          
                31 context-switches          #    0,087 K/sec                  
                 3 cpu-migrations            #    0,008 K/sec                  
               967 page-faults               #    0,003 M/sec                  
     1.285.351.385 cycles                    #    3,599 GHz                    
       925.204.273 stalled-cycles-frontend   #   71,98% frontend cycles idle   
       639.468.827 stalled-cycles-backend    #   49,75% backend  cycles idle   
     1.065.678.965 instructions              #    0,83  insns per cycle        
                                             #    0,87  stalled cycles per insn
       177.622.427 branches                  #  497,318 M/sec                  
            25.759 branch-misses             #    0,01% of all branches        

       0,360404011 seconds time elapsed

# COMPILO CON OPTIMIZACION NIVEL 3 -O3 (de ser posible el compilador implementa vectorizacion, en este caso no porque no esta especificado en el codigo fuente)

$ gcc -c profile_1.c  -O3
$ nm profile_1.o
0000000000000000 T first_assign
0000000000000000 T main
0000000000000030 T second_assign

$ gcc profile_1.o -o profile_1.e
$ time ./profile_1.e

real	0m0.004s
user	0m0.000s
sys	0m0.001s # en este nivel de optimizacion tarda muchisimas veces menos que el nivel de optimizacion 2

$ perf stat ./profile_1.e
Performance counter stats for './profile_1.e':

          0,472068 task-clock (msec)         #    0,576 CPUs utilized          # En este nivel de optimizacion utiliza la mitad de tiempo de CPU que en las optimizaciones anterioews
                 0 context-switches          #    0,000 K/sec                  
                 0 cpu-migrations            #    0,000 K/sec                  
               118 page-faults               #    0,250 M/sec                  
           745.625 cycles                    #    1,579 GHz                # utiliza unh tercio de ciclos    
           435.736 stalled-cycles-frontend   #   58,44% frontend cycles idle   
           322.843 stalled-cycles-backend    #   43,30% backend  cycles idle   
           661.928 instructions              #    0,89  insns per cycle        
                                             #    0,66  stalled cycles per insn
           126.958 branches                  #  268,940 M/sec                  
             4.775 branch-misses             #    3,76% of all branches        

       0,000819591 seconds time elapsed

######################################################################################################
2) compilo el codigo de c profile_me_2.c

$ gcc -c profile_me_2.c -g -O0 

$ nm profile_me_2.o
                 U atoi
                 U exp
                 U free
0000000000000000 T main
                 U malloc
                 U printf
                 U sin
                 U sqrt  
# Tengo varios elemetos sin definir dentro de mi objeto profile_me_2.o , por ello al crear ejecutable lo voy a tener que linkear con librerias, en este caso la de matematica

$ gcc profile_me_2.o -lm  -o profile_me_2.e   # -lm se refiere a mi libreria de matematica

$ nm profile_me_2.e #quiero ver si todos mis elementos estan bien linkeados/definidos
                 U atoi@@GLIBC_2.2.5
0000000000601070 B __bss_start
0000000000601070 b completed.6973
0000000000601060 D __data_start
0000000000601060 W data_start
00000000004006e0 t deregister_tm_clones
0000000000400750 t __do_global_dtors_aux
0000000000600e08 t __do_global_dtors_aux_fini_array_entry
0000000000601068 D __dso_handle
0000000000600e18 d _DYNAMIC
0000000000601070 D _edata
0000000000601078 B _end
                 U exp@@GLIBC_2.2.5
0000000000400ae4 T _fini
0000000000400770 t frame_dummy
0000000000600e00 t __frame_dummy_init_array_entry
0000000000400c28 r __FRAME_END__
                 U free@@GLIBC_2.2.5
0000000000601000 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
00000000004005f0 T _init
0000000000600e08 t __init_array_end
0000000000600e00 t __init_array_start
0000000000400af0 R _IO_stdin_used
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000600e10 d __JCR_END__
0000000000600e10 d __JCR_LIST__
                 w _Jv_RegisterClasses
0000000000400ae0 T __libc_csu_fini
0000000000400a70 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
000000000040079d T main
                 U malloc@@GLIBC_2.2.5
                 U printf@@GLIBC_2.2.5
0000000000400710 t register_tm_clones
                 U sin@@GLIBC_2.2.5
                 U sqrt@@GLIBC_2.2.5
00000000004006b0 T _start
0000000000601070 D __TMC_END__
# veo que nignuno de mis elementos quedo sin definir, aquellos que todavia permanecen con una U estan definidos dinamicamente, a la hora de ejecutar el programa ya saben donde tienen que irse a ejecutar y volver, la direccion esta especificada con @

## 
$ ./profile_me_2.e #corro el ejecutable que acabo de generar
Violación de segmento (`core' generado) #error de segmentation fault, hay que hacer el debug del programa con gdb

$ gdb ./profile_me_2.e
(gdb) r


Program received signal SIGSEGV, Segmentation fault.
__GI_____strtol_l_internal (nptr=0x0, endptr=endptr@entry=0x0, 
    base=base@entry=10, group=group@entry=0, 
    loc=0x2aaaab395060 <_nl_global_locale>) at ../stdlib/strtol_l.c:298
298	../stdlib/strtol_l.c: No existe el archivo o el directorio.
(gdb) 

##########  el programa tira segmentation fault porque por default al ingresar toma como verdader que ingresaste un argumento, si no lo haces quiere ir a buscar algo que no existe y se va de segmento.
## la solucion fue hacer un if que decida que si no se ingreso ningun argumento el programa te avise y termine, de esta forma se resuelve el problema .
#vuelvo a compilar en NIVEL DE OPTIMIZACION 0 	 

$ time ./profile_me_2.e 5
26.5669
real	0m0.005s
user	0m0.000s
sys	0m0.001s

$ time ./profile_me_2.e 50000
-nan
real	0m0.483s
user	0m0.000s
sys	0m0.004s


$ perf stat  ./profile_2.e 6
30.6686
 Performance counter stats for './profile_2.e 6':

          0,761573 task-clock (msec)         #    0,214 CPUs utilized          
                 1 context-switches          #    0,001 M/sec                  
                 1 cpu-migrations            #    0,001 M/sec                  
               194 page-faults               #    0,255 M/sec                  
         1.203.005 cycles                    #    1,580 GHz                    
           741.736 stalled-cycles-frontend   #   61,66% frontend cycles idle   
           573.147 stalled-cycles-backend    #   47,64% backend  cycles idle   
           923.149 instructions              #    0,77  insns per cycle        
                                             #    0,80  stalled cycles per insn
           182.114 branches                  #  239,129 M/sec                  
             8.424 branch-misses             #    4,63% of all branches        

       0,003551653 seconds time elapsed

 Performance counter stats for './profile_me_2.e 50000':

          5,412013 task-clock (msec)         #    0,825 CPUs utilized          
                 2 context-switches          #    0,370 K/sec                  
                 0 cpu-migrations            #    0,000 K/sec                  
               487 page-faults               #    0,090 M/sec                  
         8.623.367 cycles                    #    1,593 GHz                    
         3.090.683 stalled-cycles-frontend   #   35,84% frontend cycles idle   
         1.220.326 stalled-cycles-backend    #   14,15% backend  cycles idle   
        15.957.667 instructions              #    1,85  insns per cycle        
                                             #    0,19  stalled cycles per insn
         2.829.237 branches                  #  522,770 M/sec                  
            11.563 branch-misses             #    0,41% of all branches        

       0,006558526 seconds time elapsed


## NIVEL DE OPTIMIZACION 3

wtpc-12@ws4:~/Escritorio/HOdebug-profile/profile/C$ time ./profile_me_2.e 50000
-nan
real	0m0.006s
user	0m0.000s
sys	0m0.006s

 Performance counter stats for './profile_me_2.e 50000':

          3,523710 task-clock (msec)         #    0,763 CPUs utilized          
                 2 context-switches          #    0,568 K/sec                  
                 0 cpu-migrations            #    0,000 K/sec                  
               488 page-faults               #    0,138 M/sec                  
         8.763.634 cycles                    #    2,487 GHz                    
         3.295.394 stalled-cycles-frontend   #   37,60% frontend cycles idle   
         1.315.881 stalled-cycles-backend    #   15,02% backend  cycles idle   
        15.957.683 instructions              #    1,82  insns per cycle        
                                             #    0,21  stalled cycles per insn
         2.829.267 branches                  #  802,923 M/sec                  
            11.377 branch-misses             #    0,40% of all branches        

       0,004615983 seconds time elapsed



