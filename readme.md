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
