## Algoritmos de detección de Deadlocks

### Linux
Si buscamos en internet, es _box populi_ que el kernel de linux no hace nada por evitar deadlocks. Muy probablemente esto se deba a que el ojo de sauron: Linus Torvalds lee cada línea de código<sup>[1](#4)</sup> que se sube al repo y evita estos problemas antes de que lleguen a una release.  
Y esto mismo es lo que hace cada desarrollador del kernel GNU/Linux<sup>[2](#3)</sup>. Dado que el costo operacional del checkeo en runtime es muy alto, se optó por no tener mecanismos de detección de deadlocks<sup>[3](#1)</sup>. Sino que se puede agregar (en compilaciónes de debug) la feature lockdep<sup>[4](#1) [5](#2)</sup> al kernel, que checkea en cada adquisición de lock si el kernel entrará en deadlock.  
Estos checkeos tienen un costo de O(2ⁿ), por ende sólo se checkean cadenas nuevas a través de la generación de un hash y un hash lookup en una tabla lock free<sup>[6](#2)</sup>. Además se encargan de obtener estadísticas de todas las cadenas de locks obtenidas, de forma tal que despues puedan probar _todos_ los posibles interleavings entre otros componentes. Finalmente, queda _probado que no hay deadlocks_ por el momento, y en caso de que hayan se solucionarán en una próxima versión.  
Cada cuanto checkea? Cada vez que hay una cadena nueva de locks, no es por tiempo fijo.

### Windows
El caso de Windows es similar a linux, no se tiene un checkeo en runtime (que se sepa) y se pueden agregar herramientas para debuggear drivers<sup>[!cambiar](#5)</sup> o el mismo kernel<sup>[!cambiar](#6)</sup> a la consola de kernel debugging<sup>[!cambiar](#7)</sup>. Tanto la consola de kernel como la de drivers estan integradas a Windows 10 por defecto.  
Para habilitar el checkeo de deadlock se puede utilizar el comando:
`verifier /flags 0x20 /driver MyDriver.sys`
La el intervalo de checkeos no puede ser configurado ni desde la CLI ni desde la GUI.

### Referencias
##### 1
- [Static deadlock detection in the linux kernel](https://www.researchgate.net/publication/221033382_Static_Deadlock_Detection_in_the_Linux_Kernel)
##### 2
- [LockDep](https://www.kernel.org/doc/Documentation/locking/lockdep-design.txt)
##### 3 
- [Linus debugging](https://yarchive.net/comp/linux/kernel_deadlock_debug.html)
##### 4
- [Linus commit history](https://github.com/torvalds/linux/commits?author=torvalds)
##### 5
- [Verifier.exe How To](https://support.microsoft.com/en-us/help/244617/using-driver-verifier-to-identify-issues-with-windows-drivers-for-adva)
##### 6
- [Kdexts.dll extension](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-locks---kdext--locks-)
##### 7
- [Kerner debugging How To](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/getting-started-with-windbg--kernel-mode-)