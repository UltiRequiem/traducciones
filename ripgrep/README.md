## ripgrep (rg)

ripgrep es una herramienta de búsqueda orientada a líneas que busca recursivamente en su actual
directorio un patron de expresiones regulares. De forma predeterminada, ripgrep respetará su `.gitignore`
y omitira automáticamente archivos ocultos,directorios ocultos y archivos binarios. ripgrep
tiene soporte para Windows, macOS y Linux, con opciones de descargas binarias disponibles
para [todas las versiones](https://github.com/BurntSushi/ripgrep/releases).
ripgrep es similar a otras herramientas de búsqueda populares como The Silver Searcher, ack y grep.

[![Build status](https://github.com/BurntSushi/ripgrep/workflows/ci/badge.svg)](https://github.com/BurntSushi/ripgrep/actions)
[![Crates.io](https://img.shields.io/crates/v/ripgrep.svg)](https://crates.io/crates/ripgrep)
[![Packaging status](https://repology.org/badge/tiny-repos/ripgrep.svg)](https://repology.org/project/ripgrep/badges)

Doblemente licensiado bajo MIT o [UNLICENSE](https://unlicense.org).

### CHANGELOG

Porfavor revisa [CHANGELOG](CHANGELOG.md) para informarte sobre el historial de versiones.

### Documentation quick links

- [Instalación](#installation)
- [Guía de uso](./GUIDE.md)
- [FAQ](./FAQ.md)
- [Regex syntax](https://docs.rs/regex/1/regex/#syntax)
- [Archivos de configuración](GUIDE.md#configuration-file)
- [Shell completions](FAQ.md#complete)
- [Building](#building)
- [Tranducciones](#traducciones)

### Screenshot de los resultados de la búsqueda

[![A screenshot of a sample search with ripgrep](https://burntsushi.net/stuff/ripgrep1.png)](https://burntsushi.net/stuff/ripgrep1.png)

### Ejemplos rapidos comparando Herramientas

Este ejemplo busca en todo el
[Árbol de fuentes del kernel de Linux](https://github.com/BurntSushi/linux)
(después de ejecutar `make defconfig && make -j8`) para` [A-Z] + _ SUSPEND`, donde
todas las coincidencias deben ser palabras. Los tiempos se recopilaron en un sistema con Intel
i7-6900K 3,2 GHz.

¡Recuerde que un solo punto de referencia nunca es suficiente! Mira mi
[entrada de blog sobre ripgrep](https://blog.burntsushi.net/ripgrep)
para una comparación muy detallada con más evaluaciones comparativas y análisis.

| Tool                                                                                 | Comando                                                      | Cantidad de Lineas | Tiempo     |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------ | ------------------ | ---------- |
| ripgrep (Unicode)                                                                    | `rg -n -w '[A-Z]+_SUSPEND'`                                  | 452                | **0.136s** |
| [git grep](https://www.kernel.org/pub/software/scm/git/docs/git-grep.html)           | `git grep -P -n -w '[A-Z]+_SUSPEND'`                         | 452                | 0.348s     |
| [ugrep (Unicode)](https://github.com/Genivia/ugrep)                                  | `ugrep -r --ignore-files --no-hidden -I -w '[A-Z]+_SUSPEND'` | 452                | 0.506s     |
| [The Silver Searcher](https://github.com/ggreer/the_silver_searcher)                 | `ag -w '[A-Z]+_SUSPEND'`                                     | 452                | 0.654s     |
| [git grep](https://www.kernel.org/pub/software/scm/git/docs/git-grep.html)           | `LC_ALL=C git grep -E -n -w '[A-Z]+_SUSPEND'`                | 452                | 1.150s     |
| [ack](https://github.com/beyondgrep/ack3)                                            | `ack -w '[A-Z]+_SUSPEND'`                                    | 452                | 4.054s     |
| [git grep (Unicode)](https://www.kernel.org/pub/software/scm/git/docs/git-grep.html) | `LC_ALL=en_US.UTF-8 git grep -E -n -w '[A-Z]+_SUSPEND'`      | 452                | 4.205s     |

Aquí hay otro punto de referencia en el mismo corpus que el anterior que ignora gitignore
archivos y búsquedas con una lista blanca en su lugar. El corpus es el mismo que en el
benchmark anterior, y las banderas pasadas a cada comando aseguran que son
haciendo un trabajo equivalente:

| Tool                                           | Comando                                                           | Cantidad de Lineas | Tiempo     |
| ---------------------------------------------- | ----------------------------------------------------------------- | ------------------ | ---------- |
| ripgrep                                        | `rg -uuu -tc -n -w '[A-Z]+_SUSPEND'`                              | 388                | **0.096s** |
| [ugrep](https://github.com/Genivia/ugrep)      | `ugrep -r -n --include='*.c' --include='*.h' -w '[A-Z]+_SUSPEND'` | 388                | 0.493s     |
| [GNU grep](https://www.gnu.org/software/grep/) | `egrep -r -n --include='*.c' --include='*.h' -w '[A-Z]+_SUSPEND'` | 388                | 0.806s     |

Y finalmente, una comparación directa entre ripgrep, ugrep y GNU grep en un
único archivo grande almacenado en la memoria caché.
(~13GB, [`OpenSubtitles.raw.en.gz`](http://opus.nlpl.eu/download.php?f=OpenSubtitles/v2018/mono/OpenSubtitles.raw.en.gz)):

| Tool                                           | Comando                                           | Cantidad de Lineas | Tiempo     |
| ---------------------------------------------- | ------------------------------------------------- | ------------------ | ---------- |
| ripgrep                                        | `rg -w 'Sherlock [A-Z]\w+'`                       | 7882               | **2.769s** |
| [ugrep](https://github.com/Genivia/ugrep)      | `ugrep -w 'Sherlock [A-Z]\w+'`                    | 7882               | 6.802s     |
| [GNU grep](https://www.gnu.org/software/grep/) | `LC_ALL=en_US.UTF-8 egrep -w 'Sherlock [A-Z]\w+'` | 7882               | 9.027s     |

En el punto de referencia anterior, se utiliza la bandera `-n` (para mostrar los números de línea)
y esto aumenta los tiempos a `3.423s` para ripgrep y` 13.031s` para GNU grep.En ugrep
los tiempos no se ven afectados por la presencia o ausencia de "-n".

### ¿Por qué debería usar ripgrep?

- Puede reemplazar muchos casos de uso servidos por otras herramientas de búsqueda
  porque contiene la mayoría de sus funciones y generalmente es más rápido.(Ver
  [las preguntas frecuentes](FAQ.md) para obtener más detalles sobre si ripgrep realmente puede
  reemplace grep.)
- Al igual que otras herramientas especializadas para la búsqueda de código, ripgrep se establece de forma predeterminada en recursivo
  búsqueda de directorio y no buscará archivos ignorados por su
  Archivos `.gitignore` /` .ignore` / `.rgignore`. También ignora ocultos y binarios
  archivos de forma predeterminada. ripgrep también implementa soporte completo para `.gitignore`,
  Considerando que hay muchos errores relacionados con esa funcionalidad en otro código
  herramientas de búsqueda que afirman ofrecer la misma funcionalidad.
- ripgrep puede buscar tipos específicos de archivos. Por ejemplo, `rg -tpy foo`
  limita su búsqueda a archivos Python y `rg -Tjs foo` excluye JavaScript
  archivos de su búsqueda. ripgrep puede aprender sobre nuevos tipos de archivos con
  reglas de coincidencia personalizadas.
- ripgrep admite muchas características que se encuentran en `grep`, como mostrar el contexto
  de resultados de búsqueda, buscar múltiples patrones, resaltar coincidencias con
  color y compatibilidad total con Unicode. A diferencia de GNU grep, ripgrep se mantiene rápido mientras
  compatible con Unicode (que siempre está activado).
- ripgrep tiene soporte opcional para cambiar su motor de expresiones regulares para usar PCRE2.
  Entre otras cosas, esto hace que sea posible utilizar la vista alrededor y
  backreferences en sus patrones, que no son compatibles con la configuración predeterminada de ripgrep
  motor de expresiones regulares. La compatibilidad con PCRE2 se puede habilitar con `-P / - pcre2` (use PCRE2
  always) o `--auto-hybrid-regex` (use PCRE2 solo si es necesario). Una alternativa
  La sintaxis se proporciona a través de la opción `--engine (default | pcre2 | auto-hybrid)`.
- ripgrep admite la búsqueda de archivos en codificaciones de texto distintas de UTF-8, como
  como UTF-16, latin-1, GBK, EUC-JP, Shift_JIS y más. (Algún apoyo para
  Se proporciona la detección automática de UTF-16. Otras codificaciones de texto deben ser
  específicamente especificado con la marca `-E / - encoding`.)
- ripgrep admite la búsqueda de archivos comprimidos en un formato común (brotli,
  bzip2, gzip, lz4, lzma, xz o zstandard) con el indicador `-z / - search-zip`.
- soportes ripgrep
  [filtros de preprocesamiento de entrada arbitrarios] (preprocesador GUIDE.md #)
  que podría ser extracción de texto PDF, descompresión menos compatible, descifrado,
  detección automática de codificación y así sucesivamente.
  En otras palabras, use ripgrep si le gusta la velocidad, filtrando por defecto, menos
  errores y soporte Unicode.

### Por que no debería usar ripgrep?

A pesar de que inicialmente no quería agregar todas las funciones bajo el sol para ripgrep,
con el tiempo, ripgrep ha aumentado la compatibilidad con la mayoría de las funciones que se encuentran en otros archivos
herramientas de búsqueda. Esto incluye la búsqueda de resultados que abarquen múltiples
líneas y soporte opcional para PCRE2, que proporciona
soporte de referencia inversa.

En este punto, las razones principales para no usar ripgrep probablemente consisten en una
o más de los siguientes:

- Necesita una herramienta portátil y ubicua. Si bien ripgrep funciona en Windows,
  macOS y Linux, no es omnipresente y no se ajusta a ninguna
  estándar como POSIX. La mejor herramienta para este trabajo es el viejo grep.
- Todavía existe alguna otra característica (o error) que no figura en este archivo README que
  confía en que está en otra herramienta que no está en ripgrep.
- Hay un caso de borde de rendimiento en el que ripgrep no funciona bien donde otro
  la herramienta funciona bien. (¡Envíe un informe de error!)
- ripgrep no se puede instalar en su máquina o no está disponible para su
  plataforma. (¡Envíe un informe de error!)

### ¿Es realmente más rápido que todo lo demás?

Generalmente sí. Una gran cantidad de puntos de referencia con análisis detallado para cada uno estan
[disponibles en mi blog](https://blog.burntsushi.net/ripgrep).

Resumiendo, ripgrep es rápido porque:

- Está construido sobre el
  [Motor de expresiones regulares de Rust](https://github.com/rust-lang/regex).
  El cual utiliza autómatas finitos, SIMD y literal agresivo,estas optimizaciones hacen
  que la búsqueda sea muy rápida. (Se puede optar por el soporte de PCRE2
  con el indicador `-P / - pcre2`.)
- La biblioteca de expresiones regulares de Rust mantiene el rendimiento con soporte Unicode completo al
  construyendo decodificación UTF-8 directamente en su autómata finito determinista
  motor.
- Admite la búsqueda con mapas de memoria o mediante la búsqueda incremental
  con un tampón intermedio. El primero es mejor para archivos individuales y el
  Este último es mejor para directorios grandes. ripgrep elige la mejor búsqueda
  estrategia para usted automáticamente.
- Aplica sus patrones de ignorar en archivos `.gitignore` usando un
  [`RegexSet`] (https://docs.rs/regex/1/regex/struct.RegexSet.html).
  Eso significa que una única ruta de archivo se puede comparar con múltiples patrones globales.
  simultaneamente.
- Utiliza un iterador de directorio recursivo paralelo sin bloqueo, cortesía de
  [`crossbeam`] (https://docs.rs/crossbeam) y
  [`ignore`] (https://docs.rs/ignore).

### Comparación de características

Andy Lester, autor de [ack] (https://beyondgrep.com), ha publicado una
excelente tabla que compara las características de ack, ag, git-grep, GNU grep y
ripgrep: https://beyondgrep.com/feature-comparison

Tenga en cuenta que ripgrep ha desarrollado algunas características nuevas importantes recientemente que
aún no están presentes en el articulo de Andy. Esto incluye, entre otros,
archivos de configuración, passthru, soporte para buscar archivos comprimidos,
Búsqueda multilínea y compatibilidad con expresiones regulares elegantes mediante PCRE2.

### Installation

El nombre binario de ripgrep es `rg`.

**[Los archivos de archivos binarios precompilados para ripgrep están disponibles para Windows,
macOS y Linux.](https://github.com/BurntSushi/ripgrep/releases)** Se recomienda a los usuarios de otras
plataformas que no son mencionadas aquí, que descarguen uno de estos binarios precompilados.

Los binarios de Linux son ejecutables estáticos. Los binarios de Windows están disponibles como
construidos con MinGW (GNU) o con Microsoft Visual C ++ (MSVC). Cuando sea posible,
use MSVC antes que GNU, pero deberá tener el [Microsoft VC ++ 2015
redistribuible] (https://www.microsoft.com/en-us/download/details.aspx?id=48145)
instalado.

Si es un usuario de **macOS Homebrew** o de **Linuxbrew**, puede instalar
ripgrep de homebrew-core:

```
$ brew install ripgrep
```

Si es un usuario de **MacPorts**, puede instalar ripgrep desde el
[puerto oficial](https://www.macports.org/ports.php?by=name&substr=ripgrep):

```
$ sudo port install ripgrep
```

Si es un usuario de **Windows Chocolatey**, puede instalar ripgrep desde
[official repo](https://chocolatey.org/packages/ripgrep):

```
$ choco install ripgrep
```

Si es un usuario de ** Windows Scoop **, puede instalar ripgrep desde el
[bucket oficial](https://github.com/ScoopInstaller/Main/blob/master/bucket/ripgrep.json):

```
$ scoop install ripgrep
```

Si eres un usuario de **Arch Linux**, puedes instalar ripgrep desde los repositorios oficiales:

```
$ pacman -S ripgrep
```

Si es un usuario de **Gentoo**, puede instalar ripgrep desde el
[repositorio oficial](https://packages.gentoo.org/packages/sys-apps/ripgrep):

```
$ emerge sys-apps/ripgrep
```

Si es un usuario de **Fedora**, puede instalar ripgrep desde
repositorios.

```
$ sudo dnf install ripgrep
```

Si eres un usuario de **openSUSE**, ripgrep está incluido en **openSUSE Tumbleweed**
y **openSUSE Leap** desde 15.1.

```
$ sudo zypper install ripgrep
```

Si es un usuario de **RHEL/CentOS 7/8**, puede instalar ripgrep desde
[copr](https://copr.fedorainfracloud.org/coprs/carlwgeorge/ripgrep/):

```
$ sudo yum-config-manager --add-repo=https://copr.fedorainfracloud.org/coprs/carlwgeorge/ripgrep/repo/epel-7/carlwgeorge-ripgrep-epel-7.repo
$ sudo yum install ripgrep
```

Si es un usuario de ** Nix **, puede instalar ripgrep desde
[nixpkgs](https://github.com/NixOS/nixpkgs/blob/master/pkgs/tools/text/ripgrep/default.nix):

```
$ nix-env --install ripgrep
$ # (O usando el nombre del atributo, el cual también es ripgrep).
```

Si es un usuario de **Debian** (o un usuario de un derivado de Debian como **Ubuntu**),
ripgrep se puede instalar usando un archivo binario `.deb` provisto en cada
[lanzamiento de ripgrep] (https://github.com/BurntSushi/ripgrep/releases).

```
$ curl -LO https://github.com/BurntSushi/ripgrep/releases/download/12.1.1/ripgrep_12.1.1_amd64.deb
$ sudo dpkg -i ripgrep_12.1.1_amd64.deb
```

Si ejecuta Debian Buster (actualmente Debian estable) o Debian sid, ripgrep es
[mantenido oficialmente por Debian](https://tracker.debian.org/pkg/rust-ripgrep).

```
$ sudo apt-get install ripgrep
```

Si es un usuario de **Ubuntu Cosmic (18.10)** (o más reciente), ripgrep esta
[disponible] (https://launchpad.net/ubuntu/+source/rust-ripgrep) usando el mismo
empaquetado como Debian:

```
$ sudo apt-get install ripgrep
```

(N.B. También están disponibles varias snaps para ripgrep en Ubuntu, pero ninguna de ellas
parece funcionar bien y generan una serie de informes de errores muy extraños que
no sé cómo solucionarlo y no tengo tiempo para solucionarlo. Por lo tanto, no es
ya es una opción de instalación recomendada).

Si es un usuario de **FreeBSD**, puede instalar ripgrep desde el
[puerto oficial](https://www.freshports.org/textproc/ripgrep):

```
# pkg install ripgrep
```

Si es un usuario de **OpenBSD**, puede instalar ripgrep desde el
[puerto oficial](https://openports.se/textproc/ripgrep):

```
$ doas pkg_add ripgrep
```

Si es un usuario de **NetBSD**, puede instalar ripgrep desde
[pkgsrc] (https://pkgsrc.se/textproc/ripgrep):

```
# pkgin install ripgrep
```

Si es un usuario de **Haiku x86_64**, puede instalar ripgrep desde el
[puerto oficial](https://github.com/haikuports/haikuports/tree/master/sys-apps/ripgrep):

```
$ pkgman install ripgrep
```

Si es usuario de **Haiku x86_gcc2**, puede instalar ripgrep desde el
mismo puerto que Haiku x86_64 usando la compilación de arquitectura secundaria x86:

```
$ pkgman install ripgrep_x86
```

Si eres un **programas en Rust**, ripgrep se puede instalar con `cargo`.

- Tenga en cuenta que la versión mínima admitida de Rust para ripgrep es **1.34.0**,
  aunque ripgrep puede funcionar con versiones anteriores.
- Tenga en cuenta que el binario puede ser más grande de lo esperado porque contiene depuración
  símbolos. Esto es intencional. Para eliminar los símbolos de depuración y, por lo tanto, reducir
  el tamaño del archivo, ejecute `strip` en el binario.

```
$ cargo install ripgrep
```

### Building

ripgrep está escrito en Rust, por lo que deberá obtener un
[Instalación de Rust](https://www.rust-lang.org) para compilarlo.
ripgrep se compila con Rust 1.34.0 (estable) o más reciente.

Para hacerle un build a ripgrep:

```
$ git clone https://github.com/BurntSushi/ripgrep
$ cd ripgrep
$ cargo build --release
$ ./target/release/rg --version
0.1.3
```

Si tiene un compilador nocturno de Rust y una CPU Intel reciente, puede habilitar
aceleración SIMD opcional adicional como tal

```
RUSTFLAGS="-C target-cpu=native" cargo build --release --features 'simd-accel'
```

La función `simd-accel` habilita la compatibilidad con SIMD en ciertas dependencias de ripgrep
(responsable de la transcodificación). No son necesarios para obtener optimizaciones SIMD
para buscar; esos se habilitan automáticamente. Con suerte, algún día,
la función `simd-accel` se volverá igualmente innecesaria. **ADVERTENCIA:** Actualmente,
la habilitación de esta opción puede aumentar drásticamente los tiempos de compilación.

Finalmente, el soporte opcional PCRE2 se puede construir con ripgrep habilitando el
Característica `pcre2`:

```
$ cargo build --release --features 'pcre2'
```

(TIP: use `--features 'pcre2 simd-accel'` para incluir también el tiempo de compilación SIMD
optimizaciones, que solo funcionarán con el compilador en su ultima versión).

Habilitando la función PCRE2 funciona con un compilador estable de Rust y
intente encontrar y vincular automáticamente con la biblioteca PCRE2 de su sistema a través de
`pkg-config`. Si no existe, ripgrep construirá PCRE2 desde la fuente
utilizando el compilador de C de su sistema y luego vincularlo estáticamente al final
ejecutable. La vinculación estática se puede forzar incluso cuando hay un PCRE2 disponible
biblioteca del sistema construyendo ripgrep con el objetivo MUSL o estableciendo
`PCRE2_SYS_STATIC = 1`.

ripgrep se puede construir con el objetivo MUSL en Linux instalando primero la biblioteca
de MUSL en su sistema (consulte al administrador de paquetes de su distribución).
Entonces solo necesita agregar soporte MUSL a su cadena de herramientas Rust y reconstruir
ripgrep, que produce un ejecutable completamente estático:

```
$ rustup target add x86_64-unknown-linux-musl
$ cargo build --release --target x86_64-unknown-linux-musl
```

Aplicando la bandera `--features` de arriba funciona como se esperaba. Si quieres
construir un ejecutable estático con MUSL y con PCRE2, entonces necesitará tener
`musl-gcc` instalado, que puede estar en un paquete separado del actual
Biblioteca MUSL, dependiendo de su distribución de Linux.

### Ejecutando Tests

ripgrep está relativamente bien probado, incluidas tanto las pruebas unitarias como la integración
pruebas. Para ejecutar el conjunto de pruebas completo, use:

```
$ cargo test --all
```

from the repository root.

### Traducciones

La siguiente es una lista de traducciones conocidas de la documentación de ripgrep. Estas
se mantienen extraoficialmente y pueden no estar actualizados.

- [Chinese](https://github.com/chinanf-boy/ripgrep-zh#%E6%9B%B4%E6%96%B0-)

