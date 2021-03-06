Clase de 7 de Octubre
=====================

Ejercicios
----------

###Ejercicio 8

1. Qué aplicación puede tener la limitación o asignación de recursos en un entorno de producción?

> Una posible aplicación de limitación o asignación de recursos es lo que hacen las empresas de hosting
> o las empresas que suelen rentar servicios a usuarios. Lo que suelen hacer es limitar los recursos de procesamiento,
> memoria, entrada/salida, y/o ancho de bancha disponible al usuario. Esto es útil para la empresa que ofrece el
> servicio ya que se asegura que el cliente tenga disponible los servicios que haya pagado, y facilita el
> gestionamiento de recursos por parte de la empresa.

2. Implementar usando el fichero de configuración de cgcreate una política que dé menos prioridad a los procesos de usuario que a los procesos del sistema (o viceversa).

> El fichero que hay que configurar para crear políticas de grupos de acceso es:
>
> ```sh
> /etc/cgconfig.conf
> ```

> Antes de configurar el fichero, se crea el grupo de control donde estarán los
> usuarios

> ```sh
> sudo cgcreate -g memory,cpu,cpuacct:users
> ```

> Además se tiene que crear un grupo que contendrá los procesos de sistema, que
en mi caso son los procesos de root

> ```sh
> sudo cgcreate -g memory,cpu,cpuacct:system
> ```

> Después de crear los grupos de control, hay que configurar el fichero cgconfig,
> en el cual se agregará polílicas de prioridad. Lo que he agregado en el fichero
> de configuración ha sido lo siguiente:

> ```sh
> group users {
> cpu {
>    cpu.shares = "256";
>    #cpu.shares indica cuanto tiempo CPU esta disponible para un cgroup
> }
>}

> group system {
>   cpu{
>    cpu.shares = "768";
>    }
> }
> ```

> Lo que hace el cambio es que los procesos del control de grupo `users` reciben
> 25.6 % de CPU, mientas que los procesos del control de grupo `system` reciben 76.8%.

> Ahora hay que agregar todos los procesos ejecutados de los usuarios del sistema
> al grupo de control. Esto se hace usando configurando el fichero `/etc/cgrules.conf`
> y agregando una regla que sigue el siguiente sintaxis.

> **user subsystems control_group**

> En mi caso tengo que agregar el único usuario que esta presente en el sistema: josecolella

> ```sh
> josecolella cpu,cpuacct,memory users/
> root        cpu,cpuacct,memory system/
> ```
> Esto significa cualquier proceso que sea de `josecolella` que acceda a los
> los subsistemas se mueve al control de grupo `users`. Además cualquier procesor
que provenga de `root`

> Hay que reiniciar el servicio de cgconfig y cgred para que los realicen los
cambios sobre las políticas de seguridad y configuraciones sobre los grupos de control.

> ```sh
> sudo service cgconfig restart
> sudo service cgred restart
> ```

3. Usar un programa que muestre en tiempo real la carga del sistema tal como htop y comprobar los efectos de la migración en tiempo real de una tarea pesada de un procesador a otro (si se tiene dos núcleos en el sistema).

> Para aislar la carga en un CPU, se tiene que usar la opción cpuset.cpus. cpuset.cpus asigna
procesadores a procesos.

> Para usar esa opción hay que montar el sistema de ficheros virtual como un cgroup

> ```sh
> sudo mount -t cgroup /sys/fs/cgroup/cpuset
> ```

> Ahora creo un grupo de control ("processor") que se asignará la primera CPU, y después hay que
configurar el fichero /etc/cgconfig.conf para crear las reglas de prioridad.

> ```sh
> group processor {
  cpuset {
>   cpuset.cpus = "0";
>  }
> }
> ```

> Reinicamos el servicio con:

>```sh
> sudo service cgconfig restart
> ```

> Mandamos a ejecutar una aplicación con dicho grupo de control con el siguiente comando:

> ```sh
> sudo cgexec -g cpuset:processor emacs -nw &
>

> Analizamos la carga con htop que proporciona la carga en cada CPU. Ahora si
queremos cambiar la CPU en la cual se ejecutará los procesos del grupo de control
se cambia el fichero /etc/cgconfig.


> ```sh
> group processor {
  cpuset {
>   cpuset.cpus = "1";
>  }
> }
> ```

> Como podemos ver en la siguiente imagen el proceso se ha cambiado al segundo procesador:

> !["Imagen enseñando la migración al segundo procesador"](https://raw.github.com/josecolella/GII-2013/master/Screenshots/Screen%20Shot%202013-10-12%20at%2013.20.32.png)

4. Configurar un servidor para que el servidor web que se ejecute reciba mayor prioridad de entrada/salida que el resto de los usuarios.

> Para que el servidor web reciba mayor prioridad de entrada/salida hay que cambiar los parametros
> de `blkio`. `blkio` control y monitoriza el acceso a I/O.
>
> Dentro del fichero de configuración se agrega el siguiente texto:

> ```sh
> group http {

>   blkio {
>    blkio.weight = 200
>   }

> }
> ```

> *blkio.weight determina la prioridad del grupo para acceder a disco.*
> *Menos es el número, más grande es la prioridad.*

> Después hay que indicarle a Apache (servidor web) que pertence a un grupo de
> control. Agregamos la siguiente línea de texto al fichero `/etc/apache2/apache2.conf`

> ```sh
> CGROUP_DAEMON="blkio:/http"
> ```


###Ejercicio 9
- Comprobar si el procesador o procesadores instalados lo tienen.
¿Qué modelo de procesador es?
¿Qué aparece como salida de esa orden?

> El procesador que tiene la máquina es el seguiente modelo:

> ```sh
> model name  : Intel(R) Core(TM)2 Duo CPU     T9400  @ 2.53GHz
> ```

> Es un procesador con dos cores, y si vemos las caracteristicas proporcionadas
> por la página oficial, tiene la technología de virtualización de intel como
> se puede ver en la siguiente imagen:

> !["Características del procesador"](https://raw.github.com/josecolella/GII-2013/master/Screenshots/Screen%20Shot%202013-10-08%20at%2023.08.49.png)

> Determino si el procesador tiene virtualización a nivel de hardware, con el siguiente comando:
> ```sh
> egrep '^flags.*(vmx|svm)' /proc/cpuinfo
> ```

> El output correspondiente, que se puede ver en la siguiente imagen:

> !["Resultado de Ejecutar el comando"](https://raw.github.com/josecolella/GII-2013/master/Screenshots/Screen%20Shot%202013-10-08%20at%2023.03.19.png)

> indica que el procesador soporta virtualización de hardware.

###Ejercicio 10
- Comprobar si el núcleo instalado en tu ordenador contiene este módulo del kernel usando la orden kvm-ok

> Para poder la orden **kvm-ok** es importante tener instalado un paquete especial
> que habilita conocer información adicional de la CPU.
> El paquete se instala con la siguiente orden:

> ```sh
> sudo apt-get install cpu-checker
> ```

> Después de instalar dicho paquete, y ejecutamos:
> ```sh
> kvm-ok
> ```

> El comando determina si el procesador es capaz de virtualización
> a nivel de hardware. Después de ejecutar el comando la siguiente información
aparece:

> ```sh
> INFO: /dev/kvm exists
> KVM acceleration can be used
> ```

> Esto significa que el procesado tiene kvm y es capaz de virtualización de
> hardware.

###Ejercicio 11
- Comentar diferentes soluciones de Software as a Service de uso habitual.

> Software as a Service es un paradigma revolucionario para el desarrollo y despliegue
> de aplicaciones. Software as a Service útilizan el internet, especialmente los
> navegadores web como plataforma de desarrollo y ejecución.

> Soluciones que uso habitual que son Software as a Service son:
>   - redes sociales
>    - Facebook
>    - Twitter
>   - servicios de correo:
>    - Yahoo
>    - Gmail


> Hasta Microsoft ha visto que Software as a Service es el futuro
> para el desarrollo y despliegue de aplicaciones, que ha migrado
> su suite popular de Microsoft Office a la nube, denotado como Microsoft
> Office 365

> Software as a Service proporciona beneficios a los usuarios, que
> ya no tiene que preocuparse de incompatibilidades hardware con el software y a los
> desarrolladores del software.


###Ejercicio 12
- Instalar un entorno virtual para tu lenguaje de programación favorito (uno de los mencionados arriba, obviamente)

> El entorno virtual que instalaré es *virtualenv*, que es un entorno virtual
> de desarrollo de python. Los beneficios que proporciona con respecto de:
>   - versiones del lenguaje
>   - dependiencias
> habilita tener un entorno que sea igual al entorno de producción. Además
> proporciona separación entre otros entornos de desarrollo, que significa
> que podemos tener diversos entornos con diferentes versiones y modulos en base
> a la aplicación.

> Para instalar **virtualenv** usa la herramiento para instalar y gestionar
paquetes de python *pip*:

> ```sh
> pip3 install virtualenv
> ```

> Ahora instalado para comenzar un entorno aislado de desarrollo en python
> ejecutamos la siguiente orden:

> ```sh
> virtualenv ENV
> ```

> Como vemos en la siguiente imagen, **virtualenv** crea un directorio
> con el interprete de python, pip, easy_install, etc...Con esto tenemos
> lo necesario para comenzar una aplicación aislada en python.

> !["Resultado de ejecutar 'virtualenv ENV'"](https://raw.github.com/josecolella/GII-2013/master/Screenshots/Screen%20Shot%202013-10-09%20at%2000.16.16.png)

###Ejercicio 13
- Darse de alta en algún servicio PaaS tal como Heroku, Nodejitsu u OpenShift

> El PaaS que me doy de alta es Heroku, ya que lo conosco después de haber
> trabajo con el en algunas instancias.
> Un PaaS proporciona un stack de herramientas para el despliegue de una aplicación.
> Por ejemplo, cuando use Heroku para el despliegue de una aplicación web que útilizaba
> nodejs y mongodb, el heroku detectaba que la aplicación era con nodejs y proporciona
> las herramientas necesarias para su ejecución y visión por navegador.

> Para darse de alta, sólo se necesita un correo electrónico. Para poder interactuar
hay que instalarse una herramienta de línea de comando en la máquina de desarrollo,
que se puede instalar usando el siguiente comando:

> ```sh
> wget -qO- https://toolbelt.heroku.com/install-ubuntu.sh | sh
> ```

> *El comando anterior se usa para distribuciones de Debian/Ubuntu*

