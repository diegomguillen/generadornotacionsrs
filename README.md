```markdown
# Notación Formal para Secuencias de Reglas en Sistemas Automatizados

> Esta notación formaliza la descripción de secuencias de reglas en sistemas automatizados. Su objetivo es proporcionar una especificación precisa y sin ambigüedades que sirva como base para el diseño, desarrollo y validación del software. Es un principio básico de ingeniería de software.

---

## 1. Sintaxis General y Lógica de Componentes

Una regla completa sigue la siguiente estructura:

* **Contexto**: (Opcional, pero recomendable) Define el contexto en el que estamos, desde dónde partimos y dónde acabamos.
    * *Ejemplo*: `estado(puerta_abierta)`

* **Comportamiento**: Sus valores pueden ser:
    * **Cíclico**: "Repetir para siempre" (mientras el contexto sea válido).
    * **Nada**: No repetir.

* **Secuencia de operaciones**: Define una o varias operaciones encadenadas del siguiente modo:
    * **Accion_Actuador**: Define qué se hace y sobre qué se actúa. Combina la acción y su destino en un solo término descriptivo para eliminar la ambigüedad.
        * *Ejemplos*: `abrir_puerta`, `cerrar_puerta`, `encender_luz_verde`, `activar_sirena`, `esperar`.
    * **Fuente**: Es siempre el origen del evento que dispara la acción. Puede ser un componente físico (sensor, botón) o lógico (temporizador, estado del sistema, evento del sistema).
        * *Ejemplos*: `BT1`, `ST1` (Fotocélula), `SISTEMA`, `ERROR`.
    * **Evento**: Es el suceso específico de la Fuente que sirve como disparador.
        * *Ejemplos*: `impulso`, `interrupción`, `detección`, `timeout`.
    * **{....}**: Condición de finalización. Define una maniobra atómica, sin paradas y no interrumpible para eventos que no sean de seguridad.
        * *Ejemplo*: `{hasta:FCC:activo}`

---

## 2. La Decisión Clave: Atómico vs. Interrumpible

### Comportamiento Atómico: "UNA REGLA = UN COMPROMISO"

> Se usa para describir una maniobra completa. Toda la lógica está en una única regla que contiene una condición `{...}`. Una vez iniciada, es un "tren sin paradas", ignora todos los comandos ordinarios y sólo puede ser interrumpida por una regla de seguridad explícita y de mayor prioridad (ej. un STOP).

**Ejemplo 1: Maniobra de Movimiento Simple**
```

estado(cerrada) ⇒ [ abrir\_puerta(BT1:impulso){hasta: posicion\_abierta\_total} ] ⇒ estado(abriendo)

```
* **Por qué es atómica**: La condición `{hasta: posicion_abierta_total}` establece un destino claro. La regla se compromete a ejecutar la apertura de principio a fin.

**Ejemplo 2: Secuencia de Acciones Atómica**
```

estado(abierta) ⇒ [ preaviso\_luminoso(SISTEMA:orden\_movimiento){duracion: 2s} ⇒ cerrar\_puerta(SISTEMA:timeout){hasta: posicion\_cerrada} ] ⇒ estado(cerrando)

```
* **Por qué es atómica**: Aunque contiene dos pasos (preaviso y cierre), está definida en una única regla. El sistema se compromete a ejecutar la secuencia completa. El preaviso no puede ser interrumpido y, una vez finalizado, el cierre se inicia y completa de forma igualmente atómica.

**Ejemplo 3: Maniobra con Doble Condición de Parada**
```

estado(cerrando) ⇒ [ abrir\_puerta(SISTEMA:evento\_seguridad){recorrido: 10cm, hasta: FCA:activo} ] ⇒ estado(abriendo)

```
* **Por qué es atómica**: La acción tiene dos posibles finales (`10cm` o `FCA`), pero ambos están definidos dentro de la misma condición `{...}`. La regla se compromete a `abrir_puerta` hasta que se cumpla la primera de esas dos condiciones, ignorando otros eventos no prioritarios mientras tanto.

### Comportamiento Interrumpible: "VARIAS REGLAS = UN MENÚ"

> Se usa para describir un estado en el que el sistema está "escuchando" y puede reaccionar a múltiples eventos. Se define mediante varias reglas que comparten el mismo `estado(...)` de entrada. El sistema es reactivo y está a la espera; es una "estación de tren" con varias posibles salidas.

**Ejemplo 1: Interrupción Manual de una Maniobra**
```

estado(cerrando) ⇒ [ stop\_puerta(FC\_Cierre:activo) ] ⇒ estado(cerrada)
estado(cerrando) ⇒ [ abrir\_puerta(ST1:interrupcion){hasta: posicion\_abierta\_total} ] ⇒ estado(abriendo)

```
* **Por qué es interrumpible**: El estado `cerrando` es interrumpible porque el sistema no solo tiene programado un final (llegar a `FC_Cierre`), sino que también está escuchando activamente una posible interrupción de la fotocélula (`ST1`). Lo que ocurra primero, gana.

**Ejemplo 2: Menú de Opciones desde Reposo**
```

estado(parado) ⇒ [ abrir\_puerta(BT1:impulso){hasta: posicion\_abierta\_total} ] ⇒ estado(abriendo)
estado(parado) ⇒ [ abrir\_puerta\_peatonal(BT2:impulso){hasta: posicion\_peatonal} ] ⇒ estado(abriendo\_peatonal)

```
* **Por qué es interrumpible**: El estado `parado` no es un final, sino un estado de espera reactivo. Ofrece un "menú" al usuario: si se pulsa `BT1`, se ejecuta una apertura total; si se pulsa `BT2`, se ejecuta una peatonal. La elección del usuario "interrumpe" el estado de reposo para iniciar una maniobra.

**Ejemplo 3: Interrupción de un Proceso Cíclico de Fondo**
```

estado(parado) ⇒ cíclico[ encender\_luz\_standby(SISTEMA:evento) ⇒ esperar{...} ⇒ ... ]
estado(parado) ⇒ [ abrir\_puerta(SISTEMA:orden\_movimiento) ] ⇒ estado(abriendo)

```
* **Por qué es interrumpible**: El estado `parado` tiene un comportamiento cíclico por defecto (la luz parpadea). Sin embargo, este ciclo no es un "compromiso" atómico. El estado sigue escuchando otros eventos. Una `orden_movimiento` tiene prioridad, interrumpe el ciclo de parpadeo y fuerza una transición al estado `abriendo`.
```
