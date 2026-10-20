# Respuestas - Laboratorio de pruebas en Express.js

**TLP 4 · Unidad 4**

## 2. Conocer la aplicación

**¿Qué archivo contiene la regla de cálculo?**
`src/services/precios.ts`, en la función `calcularPrecioFinal`.

**¿Qué archivo define la ruta HTTP?**
`src/app.ts`, con `GET /healthy` y `POST /api/precios/calcular`.

**¿En qué se diferencian sus responsabilidades?**
`precios.ts` tiene la lógica de negocio (calcular y validar el precio) y no sabe nada de HTTP.
`app.ts` recibe la petición, llama al servicio y arma la respuesta (200 o 400).

## 3. Ejecutar las pruebas

**¿Qué comportamiento exige la prueba de `calcularPrecioFinal(1000, 150)`?**
Que la función lance un `RangeError`, porque un descuento mayor a 100 % no es válido.

**¿Qué hacía realmente el código?**
No validaba nada, solo calculaba `1000 - (1000 * 150 / 100)` y devolvía `-500`. Nunca lanzaba error.

## 4. Interpretar la falla

1. **¿Qué total devolvía un descuento del 150 % sobre $1.000?**
   `-500`.
2. **¿Por qué no cumple la regla de negocio?**
   Porque un descuento no puede ser mayor al 100 % y un precio final negativo no tiene sentido.
3. **¿Por qué la prueba falla si la función terminó de ejecutarse?**
   Porque la prueba espera que lance un error y la función terminó bien, devolviendo un número. Que termine sin romperse no quiere decir que sea correcto.

## 5. Corregir la función

Se modificó solo `src/services/precios.ts`. Se agregaron dos validaciones antes del cálculo:

- El precio tiene que ser finito y mayor o igual a 0.
- El descuento tiene que ser finito y estar entre 0 y 100.

Si no se cumple, se lanza un `RangeError`. La fórmula no se tocó.

## 6. Pruebas unitarias

Se agregaron en `tests/precios.test.ts`:

| Entrada | Resultado esperado |
| --- | --- |
| Precio 1.000 y descuento 100 % | Total 0 |
| Precio -100 y descuento 10 % | Lanza `RangeError` |
| Precio 1.000 y descuento -10 % | Lanza `RangeError` |

**Desafío: ¿la validación actual rechaza `NaN`?**
Si. `Number.isFinite(NaN)` da `false`, entonces entra al `if` y lanza `RangeError`. La prueba `rechaza NaN como precio` pasa.

## 7. Prueba de integración HTTP

Se agregó en `tests/api.test.ts` la prueba que manda descuento 150 y espera status 400 y una propiedad `error`.

**¿Por qué es de integración y no solo unitaria?**
Porque no prueba una función sola. Participan la ruta de Express (`app.ts`), el parseo del JSON, el servicio de cálculo y la conversión del `RangeError` en una respuesta 400. Es decir, se prueba que varias partes trabajen juntas.

## 8. Tests y verificación de tipos

**¿Por qué conviene usar `typecheck` y `npm test`?**
Porque revisan cosas distintas. `npm run typecheck` (`tsc --noEmit`) busca errores de tipos, por ejemplo pasar un string donde va un número, pero no ejecuta el código. `npm test` ejecuta el código y comprueba que el comportamiento sea el esperado. Node corre los `.ts` sin chequear tipos, así que uno no reemplaza al otro.

## 10. Registro de resultados

| Comprobación | Resultado | Observación |
| --- | --- | --- |
| Respuesta de `GET /healthy` | OK | Devuelve `{"estado":"ok"}` con status 200 (lo cubre `api.test.ts`). |
| Primera ejecución de `npm test` | 4 pasan, 1 falla | Fallaba el test del descuento de 150 %, según indica la guía. |
| Corrección del servicio | OK | Se agregaron validaciones con `RangeError` en `precios.ts`. |
| Nuevas pruebas unitarias | OK | 7 pruebas en `precios.test.ts`, todas pasan. |
| Prueba de integración HTTP 400 | OK | 3 pruebas en `api.test.ts`, todas pasan. |
| Verificación `npm run typecheck` | OK | Sin errores. |

Resultado final de `npm test`: **10 pruebas, 10 pasan**.

## 11. Preguntas de cierre

1. **¿Diferencia entre comprobar manualmente y una prueba automatizada?**
   Manualmente hay que repetir los pasos a mano cada vez y puede haber olvidos o erores. La prueba automatizada se escribe una vez y se ejecuta rápido las veces que haga falta, siempre igual.

2. **¿Cuál fue la causa de la primera prueba fallida?**
   La función `calcularPrecioFinal` no validaba el descuento, entonces con 150 % devolvía un total negativo en vez de lanzar un `RangeError`.

3. **¿Por qué es útil incluir casos inválidos?**
   Porque los errores suelen aparecer con datos raros (negativos, `NaN`, mayores a 100). Si solo se prueban los casos normales, el sistema puede parecer correcto y fallar con entradas malas.

4. **¿Qué comprueba cada archivo de pruebas?**
   - `tests/precios.test.ts`: solo la función `calcularPrecioFinal` de forma aislada (unitarias).
   - `tests/api.test.ts`: la ruta HTTP de Express junto con el servicio (integración).

5. **¿Qué ventajas tiene ejecutar las pruebas después de cada cambio?**
   Se detectan rápido las regresiones (algo que andaba y se rompió), se sabe qué cambio causó el problema y se puede modificar el código con más confianza.
