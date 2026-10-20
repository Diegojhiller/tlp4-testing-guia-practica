# Respuestas a las preguntas de repaso

**Taller de Lenguajes de Programación IV · Unidad 4 · Calidad y pruebas de software**

Respuestas a las preguntas de la sección 12 del [material teórico](README.md), con ejemplos tomados del laboratorio `demo_practica`.

## 1. ¿Por qué una API que devuelve una respuesta HTTP 200 puede contener errores de negocio?

El código `200` solo indica que el servidor procesó la solicitud sin una falla técnica; no garantiza que el resultado sea correcto según los requisitos. El protocolo HTTP no conoce las reglas de negocio de la aplicación.

En el laboratorio, antes de la corrección, la solicitud `POST /api/precios/calcular` con `{ "precio": 1000, "descuento": 150 }` respondía `200` con `{ "total": -500 }`. La API "funcionaba", pero aceptaba un descuento imposible y devolvía un precio negativo. Por eso las pruebas deben comparar el comportamiento observado con el **esperado según la regla de negocio**, y no conformarse con que no haya errores visibles.

## 2. ¿Qué diferencia hay entre una prueba unitaria y una de integración?

- **Prueba unitaria:** comprueba una unidad pequeña de comportamiento de forma aislada. En `tests/precios.test.ts` se llama directamente a `calcularPrecioFinal(1000, 10)` y se verifica que devuelva `900`, sin intervención de Express ni de HTTP.
- **Prueba de integración:** comprueba que varias partes colaboren correctamente. En `tests/api.test.ts`, Supertest envía una solicitud HTTP a la aplicación; participan el parseo del JSON, la ruta de Express, el servicio de cálculo y la conversión del `RangeError` en una respuesta `400` con la propiedad `error`.

Las unitarias son más rápidas y señalan con precisión dónde está el defecto; las de integración detectan problemas en la comunicación entre componentes que una prueba aislada no puede ver.

## 3. ¿Por qué `toThrow` recibe una función en lugar del resultado de ejecutarla?

Si se escribiera `expect(calcularPrecioFinal(1000, 150))`, la función se ejecutaría **antes** de llegar a `expect`. Si lanzara el error, este interrumpiría la prueba sin que Vitest pudiera capturarlo y compararlo; si no lo lanzara, `expect` recibiría solo un número.

Al escribir `expect(() => calcularPrecioFinal(1000, 150)).toThrow(RangeError)`, se entrega una función que Vitest ejecuta por su cuenta dentro de un bloque controlado. Así puede detectar si se lanzó un error y verificar que sea del tipo esperado (`RangeError`).

## 4. ¿Qué diferencia existe entre `npm test` y `npm run typecheck`?

| `npm test` | `npm run typecheck` |
| --- | --- |
| Ejecuta `vitest run`. | Ejecuta `tsc --noEmit`. |
| **Ejecuta** el código y verifica su comportamiento. | **Analiza** el código sin ejecutarlo. |
| Detecta resultados incorrectos (por ejemplo, un total de `-500`). | Detecta errores de tipos (por ejemplo, pasar un `string` donde se espera un `number`). |

Node.js ejecuta los archivos `.ts` eliminando las anotaciones de tipos, pero no las comprueba. Por eso un programa con errores de tipos puede pasar las pruebas, y un programa con tipos correctos puede tener errores de lógica. Ambas verificaciones se complementan.

## 5. ¿Qué aporta GitHub Actions al trabajo colaborativo?

GitHub Actions permite aplicar **integración continua**: cada `push` o `pull request` ejecuta automáticamente el flujo definido en `.github/workflows/pruebas.yml` (obtener el código, preparar Node.js, instalar dependencias, comprobar tipos y correr las pruebas). Esto aporta:

- **Detección temprana de regresiones:** si un cambio rompe un comportamiento que antes funcionaba, el equipo lo sabe enseguida.
- **Un entorno común y reproducible:** las pruebas no dependen de la configuración de la computadora de cada integrante.
- **Información para revisar cambios:** en un pull request se ve si las verificaciones pasan antes de integrarlo a la rama principal.
- **Una práctica uniforme:** nadie depende de recordar ejecutar las pruebas manualmente.

Aun así, la CI no reemplaza el criterio profesional: solo verifica los casos que fueron programados.
