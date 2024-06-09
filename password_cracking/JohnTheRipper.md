# Introducción

Esta práctica quiere demostrar la importancia de tener contraseñas robustas en nuestros sistemas, para realizar esta práctica instalaremos John The Ripper, una aplicación gratuita que nos permite, mediante fuerza bruta o ataque por diccionario, conseguir las contraseñas de nuestros usuarios.
Esta herramienta tiene una gran aceptación dentro de la comunidad porque permite buscar passwords débiles en nuestros sistemas, ya que soporta la gran mayoría de algoritmos de hahs, pero sobre todo porque soporta el formato de almacenamiento utilizado por el archivo /etc/shadow. Lo que nos permite validar la robustez periódicamente.

Recuerde que se puede encontrar el `Vagrantfile` en:
https://github.com/reneserral/cfc/raw/main/Vagrantfile
Si desea descargarla directamente de Powershell, sólo tiene que ponerse en la carptea donde desea descargarlo y ejecute:
```
PS> Invoke-WebRequest -Uri "https://github.com/reneserral/cfc/raw/main/Vagrantfile" -OutFile "./Vagrantfile"
```

Todo dentro de la misma línea

# Funcionamiento de John the Ripper

[John The Ripper (JtR)](https://www.openwall.com/john/) es capaz de detectar automáticamente el tipo de algoritmo de hash se ha utilizado para guardar la contraseña, esto normalmente se consigue a través de una tabla de códigos, aquellos curiosos la pueden encontrar en el siguiente link.

JtR tiene varios modos de funcionamiento, dependiendo de la información que tengamos de la estructura que tenga la contraseña que queremos averiguar. Así por defecto soporta los siguientes modos:

- **Single crack mode:** el modo más sencillo donde se prueban passwords comunes y variantes (sólo suele funcionar en inglés)
- **Wordlist mode:** se pasa una serie de palabras con posibles contraseñas y se prueban todas. Dependiendo del bien que esté el diccionario, será más probable tener éxito.
- **Incremental mode:** el modo más complicado, pero que permite realizar ataques a la fuerza bruta. Hay que tener cuidado con este modo, ya que si el archivo de contraseñas contiene contraseñas complejas podría no terminar nunca!

Cabe mencionar que JtR es capaz de utilizar varios núcleos de ejecución, que en la práctica aceleran la validación de contraseñas. Aparte de esto conviene mirar el manual para averiguar cómo limitar la fuerza bruta a un subconjunto de posibilidades. Algunos ejemplos de reglas se pueden encontrar aquí. Se recomienda leer el link anterior para poder entender el resto de la sección.

# Puesta a punto del entorno

Empezaremos por descargar el Vagrantfile que podemos encontrar en:
https://github.com/reneserral/cfc/raw/main/Vagrantfile

Una vez descargado podemos ejecutar:
```
$ vagrant up john
```

En el mismo directorio donde hemos descargado el archivo anterior.

# Ejercicios

La idea de esta práctica es ver el poder de JTR a la hora de averiguar contraseñas en sistemas UNIX. Aunque el sistema soporta otros formatos como se ha podido ver en el manual.

## Single Crack

Éste es el modo por defecto, y por tanto el más sencillo, sin embargo puede llegar a ser muy potente. Básicamente prueba contraseñas habituales y sus variantes, la configuración se puede encontrar en el archivo `/etc/john/john.conf`

Para probar, intenta averiguar las contraseñas del archivo `/home/vagrant/john/easy-shadow`, solo se tiene que ejecutar:
```
$ john /home/vagrant/john/easy-shadow
```

Por la forma de funcionamiento del modo *Single*, hay veces que puede ser necesario ejecutar el pedido más de una vez, pero tarde o temprano debería encontrar las contraseñas de ambos usuarios, ya que son débiles.
Para ver las contraseñas que ha podido averiguar sólo hace falta:
```
$ john --show /home/vagrant/john/easy-shadow
```

Un punto importante es que JtR tiene una caché que le permite almacenar los hashes ya descifrados para acelerar nuevas búsquedas, por lo que cada vez que se ejecuta la aplicación continúa la búsqueda de contraseñas allá donde se había quedado.

## Utilizando Wordlists

La otra forma de poder controla la búsqueda de contraseñas, es a través de lo que se conoce como ataque de diccionario, donde principalmente, se buscan contraseñas que sean palabras conocidas, y de veces variantes de ellas. Primero crearemos un archivo llamado **pass.txt** (puedes encontrar un ejemplo en el directorio `/home/vagrant/john/`) con contenido similar a:
```
password
test
root
pass
```

En la práctica, esta lista de palabras contendría todo el diccionario de una lengua en particular. Para poder generar estas listas, se puede hacer de diversas formas, à partir de páginas web existentes, à partir de diccionarios, como aspell, o cogiendo listas de palabras públicas. En cualquier caso, si queremos ver varios pedidos para generar contraseñas en este [enlace](https://countuponsecurity.files.wordpress.com/2016/09/jtr-cheat-sheet.pdf). Al enlace también encontraremos ayuda de cómo funciona la herramienta.

Ahora para comprobar este ataque por diccionario vamos a crear un archivo de contraseñas nosotros mismos, por eso utilizaremos el siguiente pedido:

$$ \$ \space openssl \space passwd \space -6 \space test \space \color{purple}\$6\color{green}\$NEezDuOaICNaeUcT\color{red}\$WrXzGI3UHd5EAF9RS6jGqJjKWZA7LLFnREwd08rS2XCRsHspfrk0YxAKHQ7tYk1u/u8B9BTQ83okEcccQcj.i1$$

Este comando hace el hash del string «test».
En esta salida podemos observar: `$6` el algoritmo de cifrado, marcado en lila en el ejemplo, en este caso `sha512crypt`, el salt, marcado con verde en la salida y la contraseña, marcada en rojo. El símbolo $ no forma parte de la contraseña!

Una vez tenemos el hash basta con ponerlo en un archivo de usuarios (shadow), por eso copiamos el ejemplo de la pregunta anterior: easy-shadow y modificamos el campo de contraseña, para hacer esto:
```
$ cp easy-shadow my-shadow
$ vi my-shadow
```
Dentro del editor (puedes utilizar lo que quieras, se debe modificar el password existente para el nuevo, así cogemos el campo número 2 del fichero y sustituimos el valor. O sea, la entrada, por ejemplo:
```
rserral:$6$C1RXbN7s5PxMv.tT$bilXcLu1Qyp0R2ZiGsBvqXngqN5rJc1f1UsYdKvX3notItrRHyksEF.xKXvmOQIJNWVSKhLbkNOucO/81BLra1:18843:0:99999:7:::
```
Acaba siendo:

$$ rserral:{\color{red}\$6\$NEezDuOaICNaeUcT\$WrXzGI3UHd5EAF9RS6jGqJjKWZA7LLFnREwd08rS2XCRsHspfrk0YxAKHQ7tYk1u/u8B9BTQ83okEcccQcj.i1}:18843:0:99999:7::: $$

```
$ john --wordlist=pass.txt easy-shadow
```

El problema que tiene este ataque, es que por defecto no mira combinaciones de las palabras de diccionario, se limita a realizar el match con las que hemos puesto. Para arreglar esto:

### Ampliación de las wordlists con variantes
Para ampliar las wordlists añadiendo variante, lo que haremos será crear unas reglas nuevas. Dado que esto puede llegar a ser muy complicado, aquí pondremos unas cuantas reglas para que pueda entender su funcionamiento.
Ante todo copiaremos el archivo de configuración: `/etc/john/john.conf` al directorio `/home/vagrant/.john`:
```
$ cp /etc/john/john.conf /home/vagrant/.john
```

Después editaremos el archivo: /home/vagrant/.john/john.conf y pondremos al final el siguiente contenido:
```
[List.Rules:CFC]
:
T0
d
r
```

Esto nos creará una nueva regla llamada Postgrado (puedes cambiar el nombre si quieres) que contemplará las siguientes variantes:
- **:** - No modificar la palabra del diccionario.
- **T0** - Invertir las mayúsculas/minúsculas de la primera letra de cada palabra de diccionario que nos llegara
- **d** - Duplicar la palabra que llega: testtest
- **r** - invertir la palabra que llega: tset

Puedes probar con los archivos de contraseñas anteriores (recuerda borrar el archivo `/home/vagrant/.john/john.pot`).

Para poder ejecutar esta variante es necesario que lo hagamos de la siguiente forma:
```
$ john --wordlist=pass.txt --rules=Postgrado easy-shadow
```

## Modo Incremental: Fuerza Bruta

Por último, el modo más agresivo (¡y más lento!) es la fuerza bruta, que básicamente nos permite hacer todas las combinaciones posibles de caracteres y números hasta un cierto tamaño. Hay que utilizar este modo con mucho cuidado, ya que el número de combinaciones es muy grande y puede ocurrir que nunca nos encuentre la contraseña que buscamos si esta está bien puesta: o bien es muy larga, y/o contiene caracteres de puntuación, y/ o contiene mayúsculas y minúsculas, ...

JtR, dice a la fuerza bruta modo incremental, la forma más sencilla de utilizarlo, es directamente:

```
$ john --incremental easy-shadow
```
Pero por suerte, JtR está pensado para limitar el alcance de esta fuerza bruta, esto se puede hacer a través del archivo de configuración que hemos visto antes. Y también a través de la línea de pedidos, las opciones más relevantes son:
- `--min-length=N`: mínimo número de caracteres que tiene la contraseña a probar.
- `--max-length=N`: máximo número de caracteres que tiene la contraseña a probar.
- `--incremental[=MODE]`: el modo fuerza bruta, permite acotar los caracteres de búsqueda como veremos después.

### Fuerza bruta básica

Es la forma de buscar más sencilla y más costosa, basta con ejecutar el pedido así:

```
$ john --incremental easy-shadow
```
O, como hemos dicho acotando la búsqueda:
```
$ john --incremental --max-length=8 easy-shadow
```
Donde limitamos la búsqueda hasta 8 caracteres, cuidado, buscar todas las combinaciones de 8 caracteres puede llevar varios años con un solo core como a los de las VM que utilizamos!

### Acotando la búsqueda

Hay veces que tenemos más información de qué forma tiene una contraseña, por ejemplo, son todo letras, o son todo números, cuando sabemos esto, podemos «sesgar» la fuerza bruta para que no pruebe combinaciones que sabemos que no pueden ser.

Esto se hace con los modos de fuerza bruta. Los más sencillos son:
- **Alpha**: Letras del alfabeto americano (no hay acentos) tanto mayúsculas como minúsculas.
- **Digits**: Números de 0 a 9 repetidos hasta 13 veces
- **Alnum**: Números y letras tanto mayúsculas como minúsculas.
- **LowerNum**: Letras minúsculas y números
- **UpperNum**: Letras mayúsculas y números.

Para utilizar estos modos sólo necesitamos ejecutar de la siguiente forma JtR:

```
$ john -incremental=Alnum -max-length=7 --min-length=3 easy-shadow
```

## Pequeño challenge

Se recomienda probar varias opciones y ver cómo reacciona el sistema.
A ver si eres capaz de averiguar la contraseña con la que está protegido el archivo zip que encontrarás en `/home/vagrant/john/challenge.zip`.

Como pista decir que está formado por dos letras y 4 números.
Como segunda pista, para poder crackear archivos zip hay que ayudar a JtR con el comando zip2john, que generará un hash válido para pasarle a JtR.
Como tercera pista, puede que el modo single con alguna regla interesante pueda ayudar en esta dirección.
