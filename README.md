# Control de facturas y alertas de tesorería

Proyecto práctico con Python y pandas para revisar un extracto financiero antes de preparar una remesa de pagos. El notebook detecta coincidencias entre partidas e importes elevados y genera un informe para su revisión por Tesorería.

Los datos son sintéticos y simulan un extracto de ERP. El proyecto aplica reglas de control; no utiliza modelos de inteligencia artificial ni se conecta a un ERP real.

## Objetivo

Reducir la revisión manual inicial y facilitar la identificación de partidas que necesitan investigación, manteniendo los datos originales. Las alertas no confirman errores y no bloquean pagos automáticamente.

## Archivos

| Archivo | Contenido |
|---|---|
| `facturas.csv` | Extracto de entrada con ocho partidas |
| `control_facturas.ipynb` | Carga, controles, detección, exportación y validación |
| `facturas_pendientes_revision.csv` | Partidas con coincidencias pendientes de validar |
| `alertas_tesoreria.csv` | Informe final con ambos tipos de alerta |
| `README.md` | Descripción, ejecución, resultados y límites |

Los CSV de salida se generan al ejecutar el notebook.

## Cómo ejecutarlo

1. Colocar `facturas.csv` y `control_facturas.ipynb` en la misma carpeta y abrirla en VS Code.
2. Disponer de Python y de las extensiones Python y Jupyter de VS Code. Instalar las dependencias en el entorno que se utilizará como kernel:

   ```powershell
   python -m pip install pandas ipykernel
   ```

3. Abrir el notebook y seleccionar ese entorno de Python como kernel.
4. Reiniciar el kernel y ejecutar todas las celdas de arriba abajo para reproducir el análisis desde cero.
5. Revisar el mensaje de validación y el fichero `alertas_tesoreria.csv`. Si está abierto en Excel, cerrarlo antes de volver a exportar.

El CSV de entrada utiliza comas como separador. El informe final utiliza punto y coma y codificación UTF-8 con BOM (`utf-8-sig`) para facilitar su apertura en Excel.

## Controles aplicados

### Calidad de datos

Se comprueban el número de filas y columnas, los tipos de datos, los valores vacíos y el importe total. La fecha se convierte a tipo datetime y se conserva el dataframe original.

### Coincidencias entre partidas

Se comparan `proveedor`, `factura` e `importe` con `duplicated(..., keep=False)`. Se marcan todas las apariciones de una coincidencia para revisarlas juntas.

La coincidencia no se elimina automáticamente: puede corresponder a un registro repetido, a distintas líneas o a dos vencimientos válidos de una misma factura. Para resolverla habría que consultar el documento contable, la posición, el vencimiento, el total de la factura y el estado de pago en el ERP.

### Importes elevados

Se señalan las partidas cuyo importe es estrictamente superior a tres veces la mediana de las ocho partidas originales. Las coincidencias permanecen incluidas mientras no se confirme su naturaleza.

En este caso, la mediana es de **950 €** y el umbral de **2.850 €**. Superar el umbral implica revisión; no demuestra un error ni un fraude.

### Informe y validación

El informe selecciona las partidas con al menos una alerta. Las alertas son independientes y una partida puede tener ambas. El motivo recoge las condiciones activadas y el estado queda como `Pendiente de validar`.

Después de exportar, se vuelve a leer el CSV y se comprueban el número de partidas, los registros, el importe total y la correspondencia entre la alerta de importe y su texto. Las comprobaciones de registros y totales están adaptadas a este conjunto de datos.

## Resultados del caso

| Indicador | Resultado |
|---|---:|
| Partidas originales | 8 |
| Columnas originales | 5 |
| Valores vacíos | 0 |
| Importe total original | 14.050 € |
| Partidas con coincidencia | 2 |
| Partidas con importe elevado | 1 |
| Partidas en el informe final | 3 |
| Suma de las partidas señaladas | 10.200 € |

| Registro | Proveedor | Factura | Importe | Motivo |
|---|---|---|---:|---|
| 1 | P001 | F101 | 1.200 € | Coincidencia de proveedor, factura e importe |
| 3 | P001 | F101 | 1.200 € | Coincidencia de proveedor, factura e importe |
| 6 | P002 | F205 | 7.800 € | Importe superior a tres veces la mediana |

Los 10.200 € representan la suma de las filas señaladas. No constituyen una pérdida, un ahorro confirmado ni una deuda validada. No se ha eliminado ninguna partida del extracto original.

## Conclusiones financieras

Los registros 1 y 3 requieren comprobar si representan la misma obligación repetida o partidas válidas con distintos vencimientos. Hasta disponer de esa evidencia, se mantienen ambos registros pendientes de revisión.

Para la factura F205, de 7.800 €, se comprobarían el pedido o contrato, la recepción del bien o servicio, la aprobación, los anticipos o pagos previos y el importe pendiente real. También se revisaría el vencimiento y la disponibilidad de saldo junto con los demás compromisos de tesorería antes de decidir su inclusión en la remesa.

El resultado es una lista de revisión auditable que apoya el criterio financiero. La decisión de pago requiere información adicional y validación humana.

## Límites y posibles mejoras

- Ocho partidas son suficientes para demostrar el flujo, pero no para calibrar un control de anomalías de producción.
- La mediana global no distingue entre proveedores o tipos de gasto; podría señalar compras legítimas de importe elevado.
- Faltan campos como sociedad, moneda, documento contable, posición, vencimiento e importe pendiente. El caso supone importes en euros.
- La comprobación de vacíos es básica: no sustituye una validación completa del esquema y de los valores.
- Los `assert` sirven para verificar este ejercicio durante el desarrollo; un proceso de producción necesitaría controles explícitos y gestión de errores.
- Como siguientes mejoras se podrían incorporar histórico comparable, seguimiento de incidencias y un script ejecutable. Power BI, Power Automate e integración ERP quedan como ampliaciones futuras.
