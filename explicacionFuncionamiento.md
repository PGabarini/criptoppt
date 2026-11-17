# ¿Cómo funciona ML-KEM? — Explicación paso a paso

Guion para acompañar la demo de la diapositiva 5. Cada paso corresponde a una pestaña de la demo.

- **Alice** es el cliente TLS.
- **Bob** es el servidor TLS.

---

## Paso 1: Generación de claves en Alice (KeyGen)

ML-KEM (estándar NIST FIPS 203, basado en CRYSTALS-Kyber) trabaja sobre un retículo: una grilla de cientos de dimensiones. Alice elige un camino secreto corto, que será su clave privada, y publica el punto al que llega ese camino, pero desplazado por un pequeño ruido aleatorio. Sin el ruido, el camino se podría deducir fácilmente; con él, encontrarlo es un problema que ni las computadoras cuánticas saben resolver.

## Paso 2: Envío de la clave pública

En TLS 1.3, la clave pública viaja dentro del ClientHello. Como es pública, un espía puede copiarla sin problema. Lo que no puede es reconstruir el camino secreto a partir de ella: el ruido lo impide. El algoritmo de Shor, que rompe RSA y las curvas elípticas, no sirve contra este tipo de problema.

## Paso 3: Encapsulamiento en Bob (Encaps)

Bob genera un valor aleatorio y, a partir de él, obtiene la clave compartida que van a usar los dos. Nadie elige esa clave directamente: surge del intercambio, y por eso ML-KEM es un mecanismo de encapsulamiento de claves (KEM). Después, Bob encierra ese valor usando la clave pública de Alice más su propio ruido. El resultado es el criptograma.

## Paso 4: Envío del criptograma

Bob devuelve el criptograma (1088 B) dentro del ServerHello. Junto con la clave pública, esto explica por qué el handshake poscuántico pesa tanto más que el de X25519, donde cada lado envía solo 32 B. El espía también puede copiar el criptograma, pero sin el camino secreto de Alice no puede quitarle el ruido ni obtener la clave.

## Paso 5: Desencapsulamiento en Alice (Decaps)

Alice usa su camino secreto para cancelar casi todo el ruido. El poco que queda es tan pequeño que, al redondear, cae siempre en el valor correcto (el efecto imán): la probabilidad de error es prácticamente nula. Antes de aceptar, Alice repite el cálculo de Bob y comprueba que obtiene el mismo criptograma; si no coincide, descarta el resultado, lo que la protege de criptogramas manipulados.

Al final, los dos tienen la misma clave compartida sin que esa clave haya viajado por la red. En TLS, esta clave se combina con la de X25519 (esquema híbrido X25519MLKEM768): si uno de los dos algoritmos fallara, el otro sigue protegiendo la conexión.
