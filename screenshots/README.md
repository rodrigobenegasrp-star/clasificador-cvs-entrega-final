# Capturas de evidencia

Las seis pantallas que respaldan la tabla de pruebas del README. Se toman en el mismo momento en que se graba el video de 3 minutos, con las conexiones de Make cerradas para no exponer tokens.

| Archivo | Pantalla | Qué tiene que verse |
|---|---|---|
| `01-make-historial.png` | Make → scenario *Entrega Final - Clasificador de CVs* → pestaña **History** | Las 17 ejecuciones del 16-09-2026 en verde, con la columna *Operations* (el lote de 42 ops es el que procesó los 7 candidatos) |
| `02-make-canvas.png` | Make → el canvas del scenario | Los 33 módulos con nombre, el router de 4 rutas y los globitos rojos de error handler sobre Groq y los Gmail |
| `03-notion-db1.png` | Notion → DB1 Candidatos | Los 7 candidatos con Estado, Clasificación IA, Puntaje, Notificado y el campo Error de los dos caminos infelices |
| `04-notion-db4.png` | Notion → DB4 Registro de ejecuciones | Las 16 filas con Etapa, Resultado, Tokens y Error ✓ en las dos de validación |
| `05-dashboard.png` | `https://funny-mistake-467.notion.site` en una ventana de incógnito | El dashboard público: conteo de ejecuciones, tasa de errores 13 %, suma de tokens y candidatos por estado |
| `06-gmail-hilo.png` | Gmail → el hilo de revisión de un candidato | El aviso al reclutador con puntaje y borrador, y la respuesta del flujo **en el mismo hilo** después de aprobar |

La ventana de incógnito para la captura 05 no es un detalle menor: prueba que el dashboard es realmente público y no que se ve porque hay sesión iniciada.
