# Actividad de seguimiento - Simulación 1

|Integrante|Correo|Usuario Github|
|---|---|---|
|Andrés Cardona Carmona|andres.cardona14@udea.edu.co|DarkanCC|

## Instrucciones

Antes de empezar a realizar esta actividad haga un **fork** de este repositorio y sobre este trabaje en la solución de las preguntas planteadas en la actividad de simulación. Las respuestas deben ser respondidas en español o si lo prefiere en ingles en el lugar señalado para ello (La palabra **answer** muestra donde).

**Importante**:
* Como la actividad es en las parejas del laboratorio, solo uno de los integrantes tiene que hacer el fork; y sobre repositorio bifurcado que se genera, la modificación se realiza en equipo.
* Como la entrega se debe hacer modificando el archivo READNE, se recomienda que consulte mas sobre el lenguaje **Markdown**. En el repo adjuntan dos cheatsheet ([cheat sheet 1](Markdown_Cheat_Sheet.pdf), [cheatsheet 2](markdown-cheatsheet.pdf)) para consulta rapida.
* Entre mas creativo mejor.

## Homework (Simulation)

This program, [`process-run.py`](process-run.py), allows you to see how process states change as programs run and either use the CPU (e.g., perform an add instruction) or do I/O (e.g., send a request to a disk and wait for it to complete). See the [README](https://github.com/remzi-arpacidusseau/ostep-homework/blob/master/cpu-intro/README.md) for details.

### Questions

1. Run `process-run.py` with the following flags: `-l 5:100,5:100`. What should the CPU utilization be (e.g., the percent of time the CPU is in use?) Why do you know this? Use the `-c` and `-p` flags to see if you were right.
   
   <details>
   <summary>Answer</summary>

   If we only use the flags `-l 5:100,5:100` the output will be like this:

   ```
   Process 0
     cpu
     cpu
     cpu
     cpu
     cpu

   Process 1
     cpu
     cpu
     cpu
     cpu
     cpu
   ```
   
   We know that the CPU is beign used a 100% of the time because the number 100 in the flag indicates that each one of the 5 instructions that the process is going to execute, have a 100% of chances to use the CPU, and when the CPU finishes running all the instructions of *process 0*, it will immediately start running instructions of the *process 1*.
   
   To confirm this, we use the flags `-l 5:100,5:100 -c -p` to see all the details like this:

   ```
   Time        PID: 0        PID: 1           CPU           IOs
     1        RUN:cpu         READY             1          
     2        RUN:cpu         READY             1          
     3        RUN:cpu         READY             1          
     4        RUN:cpu         READY             1          
     5        RUN:cpu         READY             1          
     6           DONE       RUN:cpu             1          
     7           DONE       RUN:cpu             1          
     8           DONE       RUN:cpu             1          
     9           DONE       RUN:cpu             1          
    10           DONE       RUN:cpu             1          

   Stats: Total Time 10
   Stats: CPU Busy 10 (100.00%)
   Stats: IO Busy  0 (0.00%)
   ```
   </details>
   <br>

2. Now run with these flags: `./process-run.py -l 4:100,1:0`. These flags specify one process with 4 instructions (all to use the CPU), and one that simply issues an I/O and waits for it to be done. How long does it take to complete both processes? Use `-c` and `-p` to find out if you were right. 
   
   <details>
   <summary>Answer</summary>

   When we run the program, we'll get this output:

   ```
   Process 0
     cpu
     cpu
     cpu
     cpu

   Process 1
     io
     io_done
   ```

   If you readed the documentation of `process.run.py`, you will know that every I/O operation takes 5 units of time (as default) to be done, and also when a process runs an I/O, the CPU uses 2 units of time, 1 for starting the I/O and 1 for finishing it. With all of this beign said, the total ammount of time used to complete both processes is 11 and is distributed like this:

   - 4 times to execute all 4 instructions of the first process.
   - 2 times to start and finish the I/O operation of the second process.
   - 5 times waiting for the I/O operation to finish (BLOCKED STATE).

   To check this answer, we run the flags `-l 4:100,1:0 -c -p` and get this:

   ```
   Time        PID: 0        PID: 1           CPU           IOs
     1        RUN:cpu         READY             1          
     2        RUN:cpu         READY             1          
     3        RUN:cpu         READY             1          
     4        RUN:cpu         READY             1          
     5           DONE        RUN:io             1          
     6           DONE       BLOCKED                           1
     7           DONE       BLOCKED                           1
     8           DONE       BLOCKED                           1
     9           DONE       BLOCKED                           1
    10           DONE       BLOCKED                           1
    11*          DONE   RUN:io_done             1          

   Stats: Total Time 11
   Stats: CPU Busy 6 (54.55%)
   Stats: IO Busy  5 (45.45%)
   ```

   </details>
   <br>

3. Switch the order of the processes: `-l 1:0,4:100`. What happens now? Does switching the order matter? Why? (As always, use `-c` and `-p` to see if you were right)
   
   <details>
   <summary>Answer</summary>

   If we switch orders we get this:

   ```
   Process 0
     io
     io_done

   Process 1
     cpu
     cpu
     cpu
     cpu
   ```

   Just like in the previous question, if you haven't read the documentation you will not know that the system is configured to switch between processes when the current process finishes or issues an I/O. Knowing this, the ammount of time used to finish is 7, and is distributed like this:

   - 2 times to start and finish the I/O for *process 0*.
   - 4 times to execute the instructions of the *process 1* while *process 0* is BLOCKED for 5 times. They are beign executed simultaneously, one by the CPU and the other by the IOs.
   - 1 time remaining of *process 0* beign BLOCKED.

   This way we are not only executing processes in less time, but we are also making a better use of our resources. The details are these:

   ```
   Time        PID: 0        PID: 1           CPU           IOs
     1         RUN:io         READY             1          
     2        BLOCKED       RUN:cpu             1             1
     3        BLOCKED       RUN:cpu             1             1
     4        BLOCKED       RUN:cpu             1             1
     5        BLOCKED       RUN:cpu             1             1
     6        BLOCKED          DONE                           1
     7*   RUN:io_done          DONE             1          

   Stats: Total Time 7
   Stats: CPU Busy 6 (85.71%)
   Stats: IO Busy  5 (71.43%)
   ```

   </details>
   <br>

4. We'll now explore some of the other flags. One important flag is `-S`, which determines how the system reacts when a process issues an I/O. With the flag set to SWITCH ON END, the system will NOT switch to another process while one is doing I/O, instead waiting until the process is completely finished. What happens when you run the following two processes (`-l 1:0,4:100 -c -S SWITCH ON END`), one doing I/O and the other doing CPU work?
   
   <details>
   <summary>Answer</summary>

   When we run the program wit the flag `-l 1:0,4:100 -c -S SWITCH_ON_END`, we get this:

   ```
   Time        PID: 0        PID: 1           CPU           IOs
     1         RUN:io         READY             1          
     2        BLOCKED         READY                           1
     3        BLOCKED         READY                           1
     4        BLOCKED         READY                           1
     5        BLOCKED         READY                           1
     6        BLOCKED         READY                           1
     7*   RUN:io_done         READY             1          
     8           DONE       RUN:cpu             1          
     9           DONE       RUN:cpu             1          
    10           DONE       RUN:cpu             1          
    11           DONE       RUN:cpu             1 
   ```

   We can see that we are not optimizing the use of our resources, because while *process 0* is BLOCKED, the CPU is not doing anything while it could be executing instructions of *process 1* which is READY.

   </details>
   <br>

5. Now, run the same processes, but with the switching behavior set to switch to another process whenever one is WAITING for I/O (`-l 1:0,4:100 -c -S SWITCH ON IO`). What happens now? Use `-c` and `-p` to confirm that you are right.
   
   <details>
   <summary>Answer</summary>

   When we run the program with the given flags, we can intuit that when *process 0* starts the IO operation, the CPU will immediately start executing instructions of *process 1*, so the time is divided like this:

   - 2 times to start and finish the I/O of *process 0*.
   - 4 times to execute instructions of *process 1* while *process 0* is BLOCKED.
   - 1 time for *process 0* to exit BLOCKED state.

   The program shows this:

   ```
   Time        PID: 0        PID: 1           CPU           IOs
     1         RUN:io         READY             1
     2        BLOCKED       RUN:cpu             1             1
     3        BLOCKED       RUN:cpu             1             1
     4        BLOCKED       RUN:cpu             1             1
     5        BLOCKED       RUN:cpu             1             1
     6        BLOCKED          DONE                           1
     7*   RUN:io_done          DONE             1
   ```

   With these flags, we make a better use of our resources, with same processes, but changing the way CPU switches between processes.

   </details>
   <br>

6. One other important behavior is what to do when an I/O completes. With `-I IO RUN LATER`, when an I/O completes, the process that issued it is not necessarily run right away; rather, whatever was running at the time keeps running. What happens when you run this combination of processes? (`./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH ON IO -c -p -I IO RUN LATER`) Are system resources being effectively utilized?
   
   <details>
   <summary>Answer</summary>

   When we run the program with the given flags, the *process 0* starts executing and immediately issues an I/O, goes to the end of the QUEUE of processes because of the flag `-I IO_RUN_LATER` and the CPU switches to the next process in the QUEUE. The *process 0* is now the last process to be executed by CPU.

   With all of this info we can intuit that the total ammount of time to end the execution of the program will be like this:

   - 6 times to start and finish all 3 I/O issues from *process 0*.
   - 15 times to execute all instructions from the other processes (5 for each).
   - 10 times while *process 0* is BLOCKED. The other 5 times while it was blocked the CPU and IOs were working simultaneously because of the flag `-S SWITCH_ON_IO`.

   That gives us a total ammount of 31 times. The output when doing this is:

   ```
   Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
     1         RUN:io         READY         READY         READY             1
     2        BLOCKED       RUN:cpu         READY         READY             1             1
     3        BLOCKED       RUN:cpu         READY         READY             1             1
     4        BLOCKED       RUN:cpu         READY         READY             1             1
     5        BLOCKED       RUN:cpu         READY         READY             1             1
     6        BLOCKED       RUN:cpu         READY         READY             1             1
     7*         READY          DONE       RUN:cpu         READY             1
     8          READY          DONE       RUN:cpu         READY             1
     9          READY          DONE       RUN:cpu         READY             1
    10          READY          DONE       RUN:cpu         READY             1
    11          READY          DONE       RUN:cpu         READY             1
    12          READY          DONE          DONE       RUN:cpu             1
    13          READY          DONE          DONE       RUN:cpu             1
    14          READY          DONE          DONE       RUN:cpu             1
    15          READY          DONE          DONE       RUN:cpu             1
    16          READY          DONE          DONE       RUN:cpu             1
    17    RUN:io_done          DONE          DONE          DONE             1
    18         RUN:io          DONE          DONE          DONE             1
    19        BLOCKED          DONE          DONE          DONE                           1
    20        BLOCKED          DONE          DONE          DONE                           1
    21        BLOCKED          DONE          DONE          DONE                           1
    22        BLOCKED          DONE          DONE          DONE                           1
    23        BLOCKED          DONE          DONE          DONE                           1
    24*   RUN:io_done          DONE          DONE          DONE             1
    25         RUN:io          DONE          DONE          DONE             1
    26        BLOCKED          DONE          DONE          DONE                           1
    27        BLOCKED          DONE          DONE          DONE                           1
    28        BLOCKED          DONE          DONE          DONE                           1
    29        BLOCKED          DONE          DONE          DONE                           1
    30        BLOCKED          DONE          DONE          DONE                           1
    31*   RUN:io_done          DONE          DONE          DONE             1
   ```
   </details>
   <br>

7. Now run the same processes, but with `-I IO RUN IMMEDIATE` set, which immediately runs the process that issued the I/O. How does this behavior differ? Why might running a process that just completed an I/O again be a good idea?
   
   <details>
   <summary>Answer</summary>

   With this configuration, the resources will be more effectively used, because while the *process 0* is BLOCKED in all of its 3 I/O issues, the CPU will switch to another process. This is, when *process 0* exits each I/O issue, the CPU will switch again to *process 0*, start another I/O issue and switch again to another process, over and over again until every process is DONE. The output is something like this:

   ```
   Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
     1         RUN:io         READY         READY         READY             1
     2        BLOCKED       RUN:cpu         READY         READY             1             1
     3        BLOCKED       RUN:cpu         READY         READY             1             1
     4        BLOCKED       RUN:cpu         READY         READY             1             1
     5        BLOCKED       RUN:cpu         READY         READY             1             1
     6        BLOCKED       RUN:cpu         READY         READY             1             1
     7*   RUN:io_done          DONE         READY         READY             1
     8         RUN:io          DONE         READY         READY             1
     9        BLOCKED          DONE       RUN:cpu         READY             1             1
    10        BLOCKED          DONE       RUN:cpu         READY             1             1
    11        BLOCKED          DONE       RUN:cpu         READY             1             1
    12        BLOCKED          DONE       RUN:cpu         READY             1             1
    13        BLOCKED          DONE       RUN:cpu         READY             1             1
    14*   RUN:io_done          DONE          DONE         READY             1
    15         RUN:io          DONE          DONE         READY             1
    16        BLOCKED          DONE          DONE       RUN:cpu             1             1
    17        BLOCKED          DONE          DONE       RUN:cpu             1             1
    18        BLOCKED          DONE          DONE       RUN:cpu             1             1
    19        BLOCKED          DONE          DONE       RUN:cpu             1             1
    20        BLOCKED          DONE          DONE       RUN:cpu             1             1
    21*   RUN:io_done          DONE          DONE          DONE             1
   ```

   </details>
   <br>
