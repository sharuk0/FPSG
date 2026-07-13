# Motor de Raycasting en Primera Persona para Consola

Este proyecto implementa en **C++** un pequeño motor de renderizado en primera persona basado en la técnica de **raycasting**.

El programa representa un entorno tridimensional simplificado dentro de la consola de Windows utilizando caracteres Unicode con diferentes niveles de intensidad. A partir de un mapa bidimensional, el motor proyecta una vista en primera persona, calcula la distancia entre el jugador y las paredes y aplica sombreado según la profundidad.

El proyecto está inspirado en los principios de renderizado utilizados por videojuegos clásicos como **Wolfenstein 3D**, y tiene como objetivo explorar conceptos fundamentales de gráficos por computadora sin utilizar un motor gráfico externo.

> Aunque el resultado visual tiene similitudes con videojuegos clásicos como *Doom*, el método implementado corresponde a un motor de raycasting basado en cuadrícula, más cercano técnicamente al enfoque de *Wolfenstein 3D*.

---

## Características

* Renderizado pseudo-3D directamente en la consola de Windows.
* Motor de raycasting construido desde cero.
* Movimiento hacia adelante y hacia atrás.
* Rotación del jugador hacia la izquierda y derecha.
* Detección básica de colisiones con paredes.
* Sombreado de paredes en función de la distancia.
* Detección visual de bordes entre bloques.
* Sombreado del suelo mediante caracteres ASCII.
* Minimapa en tiempo real.
* Visualización de la posición, orientación y FPS.
* Movimiento independiente de la velocidad de actualización mediante `delta time`.
* Mapa editable mediante una cuadrícula de caracteres.

---

## Tecnologías utilizadas

* **C++**
* **Windows API**
* **Consola de Windows**
* Biblioteca estándar de C++:

  * `<iostream>`
  * `<vector>`
  * `<utility>`
  * `<algorithm>`
  * `<chrono>`
  * `<cmath>`

No se utiliza ningún motor gráfico externo. El renderizado se realiza escribiendo directamente sobre un búfer de caracteres de la consola.

---

## ¿Cómo funciona?

El mundo se representa mediante una cuadrícula bidimensional:

* `#` representa una pared.
* `.` representa un espacio transitable.

Ejemplo:

```text
#########.......
#...............
#.......########
#..............#
#......##......#
#......##......#
#..............#
###............#
##.............#
#......####..###
#......#.......#
#......#.......#
#..............#
#......#########
#..............#
################
```

El motor genera una vista en primera persona mediante los siguientes pasos.

### 1. Cálculo del ángulo de cada rayo

Por cada columna de la pantalla se calcula un ángulo dentro del campo de visión del jugador:

```cpp
float fRayAngle =
    (fPlayerA - fFOV / 2.0f)
    + ((float)x / (float)nScreenWidth) * fFOV;
```

Cada columna de la consola representa un rayo proyectado desde la posición del jugador.

### 2. Avance del rayo

El rayo avanza gradualmente utilizando un tamaño de paso:

```cpp
float fStepSize = 0.1f;
```

En cada avance se verifica si el rayo:

* Ha salido de los límites del mapa.
* Ha alcanzado una celda que contiene una pared.
* Ha superado la distancia máxima de renderizado.

### 3. Cálculo de la altura de la pared

Una vez encontrada una pared, su altura proyectada depende de la distancia al jugador:

```cpp
int nCeiling =
    (float)(nScreenHeight / 2.0f)
    - nScreenHeight / fDistanceToWall;

int nFloor = nScreenHeight - nCeiling;
```

Las paredes cercanas ocupan una mayor cantidad de filas, mientras que las paredes lejanas se representan con menor altura.

### 4. Sombreado por profundidad

Se utilizan caracteres Unicode con diferentes densidades para representar la distancia:

```text
█  Muy cerca
▓  Cerca
▒  Distancia media
░  Lejos
   Fuera de la distancia de renderizado
```

Esto crea una sensación de profundidad sin utilizar píxeles ni colores.

### 5. Detección de bordes

Cuando un rayo alcanza una pared, el programa calcula el ángulo entre el rayo y las esquinas del bloque.

Si el rayo pasa muy cerca de una esquina, esa columna se oscurece para resaltar los límites entre paredes adyacentes.

### 6. Búfer de pantalla

El resultado de cada cuadro se almacena en un arreglo de caracteres:

```cpp
wchar_t* screen =
    new wchar_t[nScreenWidth * nScreenHeight];
```

Después, el contenido se envía a la consola mediante la Windows API:

```cpp
WriteConsoleOutputCharacter(
    hConsole,
    screen,
    nScreenWidth * nScreenHeight,
    {0, 0},
    &dwBytesWritten
);
```

---

## Controles

| Tecla | Acción                   |
| ----- | ------------------------ |
| `W`   | Avanzar                  |
| `S`   | Retroceder               |
| `A`   | Girar hacia la izquierda |
| `D`   | Girar hacia la derecha   |

Actualmente, las teclas `A` y `D` controlan la rotación del jugador. El proyecto todavía no implementa movimiento lateral o *strafing*.

Para cerrar el programa, puede cerrarse directamente la ventana de la consola. La salida mediante la tecla `Esc` puede añadirse como una mejora futura.

---

## Requisitos

Para compilar y ejecutar el proyecto se necesita:

* Windows 10 u 11.
* Un compilador compatible con C++17 o superior.
* Una consola con soporte para caracteres Unicode.
* Visual Studio, MinGW-w64 o una herramienta equivalente.

El programa depende de:

```cpp
#include <Windows.h>
```

Por este motivo, la implementación actual es específica para Windows.

---

## Compilación

### Opción 1: MinGW-w64

Desde PowerShell, CMD o una terminal compatible:

```bash
g++ -std=c++17 -O2 main.cpp -o raycasting_console.exe
```

Ejecuta el programa con:

```bash
raycasting_console.exe
```

En PowerShell puede ser necesario utilizar:

```powershell
.\raycasting_console.exe
```

### Opción 2: Visual Studio

1. Crea un nuevo proyecto de aplicación de consola en C++.
2. Copia el código dentro del archivo principal.
3. Selecciona una configuración de compilación de 64 bits.
4. Compila el proyecto.
5. Ejecuta la aplicación desde Visual Studio o desde la terminal.

---

## Instalación desde GitHub

Clona el repositorio:

```bash
git clone https://github.com/your-username/raycasting-console-engine.git
```

Ingresa al directorio:

```bash
cd raycasting-console-engine
```

Compila el proyecto:

```bash
g++ -std=c++17 -O2 main.cpp -o raycasting_console.exe
```

Ejecuta el programa:

```powershell
.\raycasting_console.exe
```

Reemplaza la URL del repositorio por la dirección real antes de publicar el proyecto.

---

## Personalización

### Modificar el mapa

El escenario se encuentra almacenado en una variable de tipo `wstring`:

```cpp
wstring map;
map += L"#########.......";
map += L"#...............";
```

Cada fila debe tener exactamente el mismo número de caracteres indicado por:

```cpp
int nMapWidth = 16;
int nMapHeight = 16;
```

Además, la cantidad total de caracteres debe cumplir:

```text
nMapWidth × nMapHeight
```

Se recomienda rodear el mapa completamente con paredes para evitar que el jugador salga del escenario.

### Cambiar la resolución de la consola

```cpp
int nScreenWidth = 120;
int nScreenHeight = 40;
```

Una resolución mayor puede mejorar la apariencia visual, pero también aumenta la cantidad de rayos calculados en cada cuadro.

### Cambiar el campo de visión

```cpp
float fFOV = 3.14159f / 4.0f;
```

El valor actual equivale aproximadamente a 45 grados.

Un campo de visión mayor permite observar más espacio, pero también puede aumentar la distorsión visual.

### Cambiar la distancia máxima

```cpp
float fDepth = 16.0f;
```

Este valor determina la distancia máxima hasta la cual se renderizan las paredes.

### Cambiar la velocidad

```cpp
float fSpeed = 5.0f;
```

La velocidad se multiplica por el tiempo transcurrido entre cuadros, haciendo que el movimiento sea aproximadamente independiente de los FPS.

### Cambiar la precisión del raycasting

```cpp
float fStepSize = 0.1f;
```

Un valor menor produce una detección más precisa, pero requiere más iteraciones por cada rayo.

Un valor mayor mejora el rendimiento, aunque puede introducir imprecisiones visuales.

---

## Estructura actual

```text
raycasting-console-engine/
├── main.cpp
├── README.md
└── LICENSE
```

Toda la lógica se encuentra actualmente dentro de `main.cpp`, incluyendo:

* Estado del jugador.
* Lectura del teclado.
* Detección de colisiones.
* Raycasting.
* Sombreado.
* Minimapa.
* Escritura en la consola.

Para una versión más extensible, estas responsabilidades podrían dividirse en clases y módulos independientes.

---

## Conceptos explorados

Este proyecto permite estudiar de manera práctica:

* Trigonometría aplicada a gráficos.
* Vectores de dirección.
* Campos de visión.
* Proyección de profundidad.
* Representación de mundos mediante cuadrículas.
* Detección de colisiones.
* Búferes de pantalla.
* Renderizado en tiempo real.
* Medición del tiempo entre cuadros.
* Sombreado basado en distancia.
* Uso básico de la Windows API.
* Compromisos entre precisión y rendimiento.

---

## Limitaciones actuales

Esta es una implementación educativa y presenta varias simplificaciones:

* Solo es compatible con Windows.
* El mapa tiene dimensiones fijas.
* El raycasting utiliza pasos incrementales en lugar de un algoritmo DDA.
* Las paredes no utilizan texturas gráficas reales.
* El suelo se sombrea mediante caracteres, no mediante texturas.
* No existe sombreado del techo.
* No existe movimiento lateral.
* No existe una tecla de salida implementada.
* La posición del jugador se almacena en variables globales.
* El búfer de pantalla se reserva manualmente y no se libera.
* Toda la lógica está concentrada en una única función.
* No existen enemigos, puertas, objetos ni interacción.
* No existe corrección explícita del efecto *fisheye*.
* La detección de colisiones considera al jugador como un punto.
* No existe un límite explícito de FPS.

El proyecto no intenta competir con un motor 3D moderno. Su propósito es mostrar los fundamentos matemáticos y algorítmicos de un renderizador por raycasting.

---

## Posibles mejoras

Entre las extensiones futuras se encuentran:

* Reemplazar el avance incremental por el algoritmo **DDA**.
* Corregir la distorsión de tipo *fisheye*.
* Implementar movimiento lateral.
* Permitir salir mediante `Esc`.
* Añadir un radio de colisión para el jugador.
* Incorporar texturas para paredes, suelo y techo.
* Añadir colores mediante atributos de la consola.
* Implementar puertas y objetos interactivos.
* Agregar enemigos y sprites.
* Cargar mapas desde archivos externos.
* Añadir diferentes niveles.
* Separar el código en clases como `Player`, `Map`, `Renderer` e `Input`.
* Utilizar RAII y contenedores estándar para gestionar el búfer.
* Limitar los FPS para reducir el consumo de CPU.
* Crear una versión multiplataforma mediante SDL2 o una biblioteca equivalente.
* Añadir pruebas para la lógica de colisiones y acceso al mapa.

---

## Motivación académica

El proyecto fue desarrollado como ejercicio de aprendizaje para comprender cómo un mapa bidimensional puede transformarse en una representación visual en primera persona.

Su implementación demuestra experiencia en:

* Programación en C++.
* Diseño de algoritmos gráficos.
* Trigonometría computacional.
* Simulación en tiempo real.
* Administración manual de memoria.
* Uso de estructuras de datos.
* Interacción con APIs del sistema operativo.
* Optimización básica de bucles de renderizado.
* Representación discreta de entornos.

Además, permite comprender cómo motores gráficos tempranos lograban producir una sensación tridimensional con recursos computacionales limitados.

---

## Agradecimientos

El proyecto se inspira en los principios de los primeros motores de raycasting y en demostraciones educativas sobre renderizado en consola.

También toma como referencia conceptual a videojuegos clásicos como:

* *Wolfenstein 3D*
* *Catacomb 3-D*
* *Doom*

La comparación es únicamente histórica y educativa. El motor implementado utiliza raycasting sobre una cuadrícula 2D y no reproduce la arquitectura completa de estos videojuegos.

---

## Licencia

Este proyecto puede distribuirse bajo la licencia MIT.

Consulta el archivo [`LICENSE`](LICENSE) para conocer los términos de uso.
