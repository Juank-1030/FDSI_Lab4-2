# Evidencia Nivel 2 — Reconstrucción de Algoritmo con Ghidra

## 1. Localización del Flujo
- Al examinar la función `main`, se identifica una condición que valida `argc == 2` y envía el argumento `argv[1]` a la función `validate_key`[cite: 13].
- Si `validate_key` retorna `1`, el programa invoca `reveal_flag()`[cite: 13].

## 2. Parámetros del Algoritmo
1. **Longitud requerida:** `strlen(candidate) == 0x11` (17 caracteres)[cite: 14].
2. **Máscara cíclica (4 bytes):** `#Q\x17j` (`0x23`, `0x51`, `0x17`, `0x6a`), indexada mediante `i & 3` (`i % 4`)[cite: 14].
3. **Arreglo esperado (17 bytes):** `e\x15D#\x0e\x03R<f\x03D/\x0ec'X\x15`[cite: 14].

## 3. Pseudocódigo Propio
```python
esperado = [0x65, 0x15, 0x44, 0x23, 0x0E, 0x03, 0x52, 0x3C, 0x66, 0x03, 0x44, 0x2F, 0x0E, 0x63, 0x27, 0x58, 0x15]
mascara = [0x23, 0x51, 0x17, 0x6A]

# Operación inversa: candidate[i] = esperado[i] ^ mascara[i % 4]
clave = "".join([chr(esperado[i] ^ mascara[i % 4]) for i in range(len(esperado))])
print(clave) # Resultado: FDSI-REVERSE-2026
