# Evidencia Validación Dinámica GDB

Pegue aquí el bloque de ejecución de GDB mostrando cómo se detiene el programa en el breakpoint y el valor de los registros.

# Evidencia: Confirmación Dinámica con GDB

## 1. Configuración de Breakpoint y Entorno
- Se configuró la sintaxis Intel (`set disassembly-flavor intel`).
- Se fijó un punto de interrupción en la función `validate_key` (`break validate_key`).
- Se ejecutó el binario pasando la clave deducida: `run FDSI-REVERSE-2026`.

## 2. Inspección de Registros y Desensamblado
En el punto de interrupción (`0x401162`), el registro `rdi` apunta a la dirección de memoria de la clave candidata:
- `rdi = 0x7fffffffe1ab ("FDSI-REVERSE-2026")`
- Se observó la comparación de longitud contra `0x11` (17 caracteres) y el bucle de transformación con máscara modular (`and eax, 0x3`).

## 3. Salida de Ejecución
```text
(gdb) continue
Continuing.
License accepted.
FLAG{ghidra_plus_gdb}
[Inferior 1 (process 43467) exited normally]
