# TP1-Performance y Rendimiento de los computadores

### Nombre del Grupo:
- Ataque x86

### Participantes:
- Arnaudo, Federico Andres

# Informe: Uso de Benchmarks para la Selección de Hardware

## 1. Introducción

Los benchmarks son herramientas que permiten medir el rendimiento de distintos componentes de una PC para un determinado trabajo. Utilizar benchmarks de terceros permite tomar decisiones objetivas al momento de elegir hardware, ya que simulan cargas de trabajo reales.

El rendimiento de los computadores esta relacionado con el tiempo que tarda en ejecutar programas. Es inversamente proporcional al tiempo, es decir, mientras mas tiempo tarda, menor el rendimiento.

---

## 1.2. Listado de Benchmarks

Gaming / GPU
- Unigine Superposition: Mide rendimiento gráfico (FPS) en escenarios realistas.
- 3DMark: Benchmark estándar con tests modernos como DirectX 12.

CPU (general)
- Geekbench: Mide rendimiento single-core y multi-core. Bueno para comparar CPUs rápido.
- Cinebench: Evalúa CPU usando renderizado real (multinúcleo).

Programación / Desarrollo
- build-linux-kernel: Mide tiempo de compilación del kernel de Linux.
- Phoronix Test Suite: Suite completa de benchmarks para Linux.

Navegador / Web
- Speedometer: Simula uso de aplicaciones web modernas.
- WebXPRT: Mide rendimiento en tareas web reales (IA, imágenes, etc.).

Almacenamiento
- CrystalDiskMark: Mide velocidad de lectura/escritura de discos.
- ATTO Disk Benchmark: Evalúa rendimiento de almacenamiento en distintos tamaños de archivo.

Sistema completo
- PCMark: Simula uso diario del sistema (oficina, navegación, multimedia).

## 1.3. Tareas habituales

Las tareas consideradas en este análisis son:

- Gaming  
- Uso de navegador (browser)  
- Compilación y ejecución de programas  

### Selección de benchmarks

Se seleccionaron benchmarks representativos de cada tipo de tarea:

| Tarea | Benchmark recomendado | ¿Qué mide? | Justificación |
|------|---------------------|-----------|--------------|
| Gaming | Unigine Superposition | Rendimiento gráfico (FPS) | Simula cargas reales de videojuegos |
| Browser | Speedometer / JetStream | Rendimiento en JavaScript | Representa uso de aplicaciones web |
| Compilación | build-linux-kernel (Phoronix) | Tiempo de compilación | Representa workloads de desarrollo |
| Ejecución de programas | Geekbench | Rendimiento CPU general | Evalúa cargas mixtas |

---

## 1.4 Benchmark de compilación

El benchmark utilizado es:

https://openbenchmarking.org/test/pts/build-linux-kernel-1.15.0

Este benchmark mide el tiempo necesario para compilar el kernel de Linux.

### Características:
- Uso intensivo de CPU multinúcleo  
- Dependencia de memoria y caché  
- Escalabilidad parcial (no completamente paralelizable)  

---

## 1.5 Rendimiento de procesadores

| Procesador | Tiempo estimado | Version
|-----------|----------------|------------|
| Intel Core i5-13600K | ~83 +/-3 segundos | 1.16.x |
| AMD Ryzen 9 5900X | ~97 +/-6 segundos | 1.16.x | 
| AMD Ryzen 9 7950X | ~52 +/-3 segundos | 1.16.x |

---

## Aceleración (Speedup)

Se utiliza la fórmula:

Speedup = Rendimiento_Mejorado / Rendimiento_Original

### Comparación con Ryzen 9 5900X:

Speedup = 97 / 52 ≈ 1.86

### Comparación con i5-13600K:

Speedup = 83 / 52 ≈ 1.60

---

# Parte 2 - Profiling

El profiling es una tecnica para medir tiempos de ejecución, tiempos que tarda cada función y hasta cuantas veces se llama a una función.

Una herramienta para realizar estas mediciones es ```gprof```.

Para utilizarla se llevo a cabo los siguientes pasos:

## 2.1 - Compilar con profiling utilizando el comando:
```gcc -Wall -pg test_gprof.c test_gprof_new.c -o test_gprof```

![alt text](<images/Screenshot from 2026-03-22 17-35-03.png>)

![alt text](<images/Screenshot from 2026-03-22 17-34-33.png>)

## 2.2 - Ejecucion del programa

```./test_gprof```

![alt text](<images/Screenshot from 2026-03-22 17-35-32.png>)

se genera elarchivo "gmon.out", que contiene los datos de la ejecución.

### 2.3 - Analisis con gprof

```gprof test_gprof gmon.out > analysis.txt```

![alt text](<images/Screenshot from 2026-03-22 17-36-04.png>)

![alt text](<images/Screenshot from 2026-03-22 17-36-30.png>)

![alt text](<images/Screenshot from 2026-03-22 17-36-56.png>)

El analisis muestra que el tiempo de ejecucion esta distribuido en las funciones `func2`, `new_func1` y `func1`.

La función `new_func1` es la que más tiempo consume de forma individual, representando aproximadamente el 34.22% del tiempo total de ejecución. Luego, se encuentra `func2`, con un 33.84%, por lo que ambas podrían considerarse los cuellos de botella del programa.

Se observa que `main` no consume tiempo, `func2` y `new_func1` distribuyen su tiempo a su propia función, mientras que `func1` consume su tiempo y lo distribuye con el llamado a `new_func1`.


---

# Parte 3 - Variación de frecuencia ESP32

## Objetivo

Mediante un programa que contenga bucles que duren al rededor de 10 segundos, uno para una operacion de enteros y otro para operacion de floats, analizar que ocurre con los tiempos de ejecucion al variar la frecuencia de trabajo.

El microcontrolador ESP32 trabaja a una frecuencia máxima de 240Mhz, las frecuencias a comprar son las siguiente:
- 4 MHz
- 8 Mhz

## Información

En estas pruebas se estará observando el rendimiento del sistema, el cual se mide de manera inversamente proporcional al tiempo de ejecución; es decir, a mayor tiempo, menor es el rendimiento.

Para analizar el rendimiento del procesador es necesario conocer:

- **Frecuencia de CPU** ($f_{CPU}$): Número de ciclos por segundo al que trabaja el procesador.  
  $$f_{CPU} = \frac{\text{Número de ciclos}}{\text{tiempo}}$$

- **Periodo de la CPU** ($T_{CPU}$): Es el tiempo que se tarda en realizar un ciclo y es la inversa de la frecuencia:  
  $$T_{CPU} = \frac{1}{f_{CPU}}$$

- **Ciclos por instrucción (CPI)**: Es la cantidad de ciclos, en promedio, que se tarda en ejecutar una instrucción.

Conociendo:

- **Tiempo de programa**:  
  $$T_p = \text{Número de instrucciones} \times CPI \times T_{CPU}$$

Podemos calcular el **rendimiento** ($\eta$) como:  
$$\eta = \frac{1}{T_p} = \frac{f_{CPU}}{\text{Número de instrucciones} \times CPI}$$

- **Speedup** ($S$): Es la relación entre el rendimiento de un sistema mejorado y el sistema original:

$$S = \frac{\eta_m}{\eta_o}$$

Como para estas pruebas se utiliza el mismo programa, y tanto el número de instrucciones como el CPI se mantienen constantes, y lo que varía es la frecuencia del procesador, entonces:

$$
S = \frac{\frac{f_{CPU,m}}{\text{Número de instrucciones} \times CPI}}{\frac{f_{CPU,o}}{\text{Número de instrucciones} \times CPI}}
$$

Simplificando:

$$
S = \frac{f_{CPU,m}}{f_{CPU,o}}
$$

Si la frecuencia mejora en un factor $n$:

$$
f_{CPU,m} = n \cdot f_{CPU,o}
$$

Entonces:

$$
S = n
$$

A partir del análisis realizado, se concluye que, manteniendo constante el número de instrucciones y el CPI, el rendimiento del sistema depende directamente de la frecuencia de la CPU.

En particular, al incrementar la frecuencia en un factor $n$, el tiempo de ejecución del programa disminuye en el mismo factor, ya que:
$$
T_p \propto \frac{1}{f_{CPU}}
$$

Por lo tanto, si la frecuencia se duplica, el tiempo de ejecución se reduce a la mitad. En consecuencia, el rendimiento, definido como el inverso del tiempo de ejecución, aumenta proporcionalmente con la frecuencia:

$$
\eta \propto f_{CPU}
$$

Esto implica que duplicar la frecuencia del procesador resulta en una duplicación del rendimiento del sistema.

Sin embargo, este análisis supone condiciones ideales, donde el CPI y el número de instrucciones permanecen constantes. En sistemas reales puede no ser tan exacto debido a factores mas externos como memoria, interrupciones, latencias, etc.

## Codigo utilizado

Para este desarrollo se ha utilizado el siguiente codigo.

```
#include "esp_pm.h"

const uint32_t N = 15000000UL;

uint32_t pruebaEnteros() {
    uint32_t ti = micros();
    volatile long suma = 0;

    for (long i = 0; i < N; i++) {
        suma += (i & 0x7U);
    }

    return micros() - ti;
}

uint32_t pruebaFloat() {
    uint32_t ti = micros();
    volatile float suma = 0;

    for (long i = 0; i < N; i++) {
        suma += 1.21f;
    }

    return micros() - ti;
}

uint32_t pruebaDouble() {
    uint32_t ti = micros();
    volatile double suma = 0;

    for (long i = 0; i < N; i++) {
        suma += 3.14;
    }

    return micros() - ti;
}

void correrPruebas(int freqMHz) {
    setCpuFrequencyMhz(freqMHz);
    delay(1000); // estabilizar

    Serial.println("=================================");
    Serial.print("Frecuencia REAL: ");
    Serial.print(getCpuFrequencyMhz());
    Serial.println(" MHz");

    uint32_t tf;

    tf = pruebaEnteros();
    Serial.print("Enteros: ");
    Serial.println(tf / 1000000.0);

    tf = pruebaFloat();
    Serial.print("Float: ");
    Serial.println(tf / 1000000.0);

    tf = pruebaDouble();
    Serial.print("Double: ");
    Serial.println(tf / 1000000.0);
}

void setup() {
    Serial.begin(115200);
    delay(2000);

    correrPruebas(4);
    correrPruebas(8);
}

void loop() {
    // vacío
}
```

Observaciones:
- El numero de iteraciones: ```const uint32_t N = 15000000UL;```
- Función que permite modificar la frecuencia: ```setCpuFrequencyMhz(freqMHz);```
- Cada ejecución realiza un recorrido por los bucles para una funcion de enteros, flotantes y doubles, con una frecuencia en particular.
- ESP32 tiene unidad de punto flotante, es decir, aceleración por hardware.
- Double es mucho mas lento que float.

Por falta de recursos, el experimento se ha realizado mediante simulación, utilizando un simulador de electronica online

## Resultados

![alt text](images/image-5.png)

![alt text](images/image-4.png)

Speed-up en operación de entero:

S = 13.99 / 17.50 = 0.80

Speed-up en operación de float:

S = 17.98 / 22.50 = 0.80

Speed-up en operación de double

S = 89.71 / 112.24 = 0.80

Este experimento fue comparado con otros trabajos y no es tan representativo respecto a lo que se seperaría segun el fundamento teórico. Los resultados respecto a una prueba de hardware real, se acerca pas a los resltados esperados, donde el rendimiento se aproxima mas al doble cuando la frecuencia también se incrementa en el mismo factor.

Si bien existe un incremento en el rendimiento, pero el speedup no es el esperado.

