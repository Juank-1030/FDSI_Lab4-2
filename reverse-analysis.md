# Informe de Análisis — Laboratorio de Ingeniería Inversa CTF

## 1. ¿Qué información pudiste obtener sin ejecutar el binario?
Mediante análisis estático previo se pudo identificar:
- **Formato y Arquitectura:** Con `file` y `readelf -h` se identificó que son binarios ELF de 64 bits para Linux (x86-64), enlazados dinámicamente y con intérprete `ld-linux-x86-64.so.2`[cite: 2].
- **Símbolos y Depuración:** Se comprobó que `crackme_level1` y `crackme_level2` contienen símbolos (`not stripped`), mientras que `crackme_level2_stripped` carece de ellos (`stripped`)[cite: 2].
- **Cadenas legibles:** Con `strings` se extrajeron funciones importadas (`strcmp`, `strlen`, `puts`, `printf`), mensajes de interfaz y, en el caso de Nivel 1, la contraseña en texto claro (`REDTEAM-101`)[cite: 2].

## 2. ¿Por qué una contraseña compilada como string es un diseño inseguro?
Almacenar secretos estáticos dentro del código fuente es una vulnerabilidad crítica (*hardcoded credentials*)[cite: 2]. Durante el proceso de compilación, las constantes de texto se colocan directamente en la sección `.rodata` (datos de solo lectura) del ejecutable[cite: 2]. Cualquier atacante o analista puede extraerlas en segundos con utilidades estándar (`strings`, editores hexadecimales o `grep`) sin necesidad de desensamblar el código ni contar con privilegios especiales[cite: 2].

## 3. ¿Qué cambió entre `crackme_level2` y `crackme_level2_stripped`?
El binario original contiene las secciones `.symtab` (tabla de símbolos) y `.strtab` (tabla de cadenas de símbolos), las cuales conservan los nombres de funciones locales y variables (`validate_key`, `reveal_flag`, `k.1`, `expected.0`)[cite: 2]. Al aplicar `strip`, estas secciones se eliminan, reduciendo el tamaño del archivo y dificultando la depuración directa con nombres[cite: 2]. Sin embargo, la lógica de las instrucciones en `.text` y las cadenas en `.rodata` permanecen intactas, por lo que el algoritmo sigue siendo reconstruible analizando el flujo y las referencias cruzadas[cite: 2].

## 4. ¿Qué ventaja tuvo Ghidra sobre `objdump`?
Mientras que `objdump` muestra únicamente las instrucciones desensambladas en lenguaje ensamblador lineal[cite: 2], Ghidra realiza un análisis intermedio (P-code) que permite:
- Reconstruir estructuras de control de alto nivel como bucles `for`, sentencias `if/else` y retornos[cite: 13, 14].
- Inferir tipos de datos y firmas de funciones[cite: 13, 14].
- Permitir renombrado interactivo de variables y referencias cruzadas automáticas (XREFs) bidireccionales entre código y datos[cite: 2, 13, 14].

## 5. ¿Qué confirmó GDB que el análisis estático por sí solo no demostraba?
GDB permitió comprobar el comportamiento dinámico en tiempo real bajo condiciones reales de ejecución del sistema operativo[cite: 2]:
- Permitió verificar los valores exactos cargados en los registros del procesador (`rdi`, `rax`, `rsp`, banderas `eflags`) en el breakpoint exacto[cite: 2].
- Demostró paso a paso cómo se evalúa la condición de retorno sin depender únicamente de deducciones estáticas[cite: 2].

## 6. ¿Por qué Burp Suite no es una herramienta de ingeniería inversa de binarios?
Burp Suite es un proxy de intercepción y suite de pruebas para la capa de aplicación web que inspecciona y manipula tráfico de red (HTTP/HTTPS, WebSockets)[cite: 2]. No está diseñado para inspeccionar binarios compilados, arquitecturas de CPU, llamadas al sistema operativo ni estructuras de memoria interna como registros o el stack[cite: 2].

## 7. ¿Qué controles de desarrollo evitarían embebidos inseguros de secretos en software real?
- **Validación server-side:** Nunca validar secretos sensibles del lado del cliente sin control del servidor.
- **Funciones hash criptográficas:** En caso de comparar credenciales, contrastar hashes resistentes con sal criptográfica segura (como Argon2id, bcrypt o PBKDF2) en lugar de cadenas directas.
- **Escaneo automatizado de código (SAST / Secret Detection):** Implementar herramientas de análisis estático en los pipelines de CI/CD (como Gitleaks o Trivy) para bloquear commits con secretos embebidos.
- **Bóvedas de gestión de secretos:** Cargar credenciales y llaves de cifrado en tiempo de ejecución mediante variables de entorno protegidas o servicios de gestión (AWS Secrets Manager, HashiCorp Vault).
