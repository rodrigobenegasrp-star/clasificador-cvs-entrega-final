# Clasificador de CVs con IA · Entrega Final

**Curso:** Arquitecto de Flujos IA · **Alumno:** Rodrigo Benegas · **Fecha:** septiembre 2026

Ecosistema de automatización para un equipo de selección de personal. Un reclutador carga candidatos en Notion; el flujo de Make recupera el contexto de la empresa (RAG), clasifica cada CV con IA (`openai/gpt-oss-120b` vía Groq), le pide aprobación al reclutador por email y recién entonces le escribe al candidato, respondiendo al reclutador **en el mismo hilo de Gmail**. Cada paso queda registrado en una base de ejecuciones que alimenta un dashboard público con la tasa de errores.

## Entregables

| # | Entregable (20 % cada uno) | Archivo / link |
|---|---|---|
| 1 | Diagrama de arquitectura | [docs/01-diagrama-arquitectura.pdf](docs/01-diagrama-arquitectura.pdf) |
| 2 | Manual operativo de datos (esquemas de tablas + JSON de transferencia) | [docs/02-manual-operativo-datos.pdf](docs/02-manual-operativo-datos.pdf) |
| 3 | Matriz de costos por tarea | [docs/03-matriz-costos.pdf](docs/03-matriz-costos.pdf) |
| 4 | Seguridad y resiliencia | [docs/04-seguridad-resiliencia.pdf](docs/04-seguridad-resiliencia.pdf) |
| 5 | Dashboard de control (vista pública de Notion) | **PENDIENTE: link público** |
| · | Blueprint del flujo (exportado de Make) | [blueprint/clasificador-cvs.blueprint.json](blueprint/clasificador-cvs.blueprint.json) |
| · | Estructura de datos "Salida clasificador IA" | [blueprint/data-structure-salida-ia.json](blueprint/data-structure-salida-ia.json) |
| · | Bases de Notion (solo lectura) | **PENDIENTE: link** |
| · | Capturas de evidencia | [screenshots/](screenshots/) |
| · | Video demo (3 min) | **PENDIENTE: link** |

Las fuentes HTML de los documentos están en [docs/src/](docs/src/).

## Cómo funciona

1. **Trigger inteligente** · Notion *Watch Database Items* sobre DB1 Candidatos (ítems editados, cada 15 min, máximo 10 por ciclo).
2. **Router de 4 rutas** decididas por el Estado de la ficha y la existencia de datos (comparaciones texto/booleano/existencia, sin valores fijos):
   - **A · Clasificar**: `Pendiente` + datos completos → RAG en DB3 → Groq (JSON, `max_tokens` 700) → Parse JSON → email HITL al reclutador (Message-ID propio) → DB1 pasa a `En revisión` con el Thread ID → log OK en DB4.
   - **B · Aprobado**: `En revisión` + `Aprobado ✓` → email al candidato con el borrador (editable por el humano) → `Contactado` → log → respuesta al reclutador **en el mismo hilo** (In-Reply-To / References).
   - **C · Rechazado**: `Rechazado` + `Notificado` vacío → email de cierre → `Notificado ✓` → log.
   - **D · Camino infeliz**: `Pendiente` con datos faltantes → `Error` + qué falta → log con Error ✓ → aviso al reclutador. Sin llamada a la IA.
3. **Error handlers** en Groq y en cada Gmail: registro del error en DB4 → ficha en `Error` con el mensaje → `Break` (3 reintentos cada 15 min, ejecuciones incompletas guardadas). En Parse JSON: registro → `Error` → `Ignore`.
4. **Anti-bucle**: cada ruta termina cambiando el estado que su propio filtro exige, así que una ficha editada por Make no vuelve a entrar por la misma ruta.

## Reproducir el flujo en Make

1. *Scenarios → Create a new scenario → ⋯ → Import Blueprint* y elegir `blueprint/clasificador-cvs.blueprint.json`.
2. Asignar las conexiones de Notion, Groq y Gmail (Make las pide una vez por app).
3. *Data structures → Add → Generate* y pegar el JSON de ejemplo de `blueprint/data-structure-salida-ia.json` (o cargar los campos a mano) → seleccionarla en el módulo **6 · Parse JSON**.
4. Reemplazar los IDs de base de datos (parámetro *Database* de los módulos 1, 3, 8, 9, 11, 12, 15, 16, 17, 18 y de los handlers) por los de tu Notion. Los IDs de propiedad están en el manual (doc 02, §3).
5. Scheduling cada 15 minutos. Primera corrida: *Run once* con "Choose where to start → Since specific date" y una fecha anterior a la carga de candidatos (así procesa los ya cargados); después activar el scenario.

## Reproducir la memoria en Notion

Las 4 bases, sus propiedades, tipos e IDs están en el manual (doc 02, §3). Cada base debe compartirse con la integración de Make (*⋯ → Connections*). El dashboard es una página con vistas vinculadas de DB4 (conteo, % de filas con Error = tasa de errores, suma de tokens) y de DB1 (candidatos por estado, pendientes de revisión, fichas con error), publicada con *Share → Publish*.

## Pruebas (test de estrés + camino infeliz)

| # | Caso | Fecha | Ops | Tokens | Resultado | Evidencia |
|---|---|---|---|---|---|---|
| 1 | Lucía Fernández · backend, 5 años, datos completos (ruta A) | | | | | |
| 2 | Martín Sosa · marketing, 1 año con mínimo 2 (ruta A) | | | | | |
| 3 | Valentina Ruiz · full stack, 3 años (ruta A) | | | | | |
| 4 | Diego Paredes · marketing, 4 años (ruta A) | | | | | |
| 5 | Camila Torres · datos, sin backend en producción (ruta A) | | | | | |
| 6 | Aprobación humana sobre el caso 1 (ruta B, respuesta en el hilo) | | | | | |
| 7 | Rechazo humano sobre el caso 2 (ruta C) | | | | | |
| 8 | **Camino infeliz** · Sofía Méndez sin Resumen del CV (ruta D) | | | | | |
| 9 | **Camino infeliz** · Tomás Aguirre sin Correo (ruta D) | | | | | |

Se completa con los datos de *History* de Make y de DB4 después de las corridas; las capturas van a `screenshots/`.

## Checklist de la consigna

- [x] Trigger inteligente (Watch Database Items, ítems editados)
- [x] Message.Content mapeado con variables dinámicas (búsqueda + candidato + contexto RAG)
- [x] Max Tokens limitado (700)
- [x] Error handler obligatorio que guarda un registro si la API de IA falla (módulos 50/51/52 → DB4 + DB1 + Break)
- [x] HITL antes de la acción crítica (email al reclutador + espera en `En revisión`)
- [x] Gmail: Thread ID mapeado para responder en el mismo hilo (módulo 13)
- [x] Nodos con nombre, variables dinámicas, sin datos hardcodeados
- [x] Filtro anti-bucle y comparaciones con tipos correctos
- [x] Blueprint exportado desde Make (33 módulos con nombre, leído del scenario después de cargarlo)
- [ ] ≥ 5 ejecuciones + camino infeliz con evidencia
- [ ] Dashboard público con KPIs y tasa de errores
- [ ] Video de 3 minutos

## Guion del video (3 minutos)

| Tiempo | Pantalla | Qué decir |
|---|---|---|
| 0:00–0:20 | Diagrama (doc 01) | "Es un clasificador de CVs: Notion es la memoria, Make orquesta, la IA clasifica, Gmail comunica. Nada le llega a un candidato sin que un humano apruebe." |
| 0:20–0:50 | Notion: DB2 Búsquedas y DB3 Base de conocimiento | "Las búsquedas tienen requisitos y reclutador; la base de conocimiento es el contexto que la IA recupera (RAG). En Make no hay nada hardcodeado: todo sale de acá." |
| 0:50–1:20 | Notion: DB1 Candidatos | "Cargo una candidata en Pendiente. Con Resumen, Correo y Búsqueda completos entra a la ruta de IA." |
| 1:20–2:00 | Make: canvas | "Trigger cada 15 minutos, router de 4 rutas por estado, filtros con tipos correctos. Groq con max_tokens 700 y salida JSON. Acá el error handler: si la API falla, guarda el registro en DB4, deja la ficha en Error y hace Break con 3 reintentos." |
| 2:00–2:30 | Make: Run once → Gmail del reclutador → Notion | "Corre, me llega el email de revisión con el puntaje y el borrador; la ficha quedó En revisión con el Thread ID. Marco Aprobado: sale el email al candidato y la respuesta al reclutador va en el mismo hilo." |
| 2:30–2:50 | Notion: ficha sin resumen → Error · Dashboard público | "Camino infeliz: sin resumen no llama a la IA, marca qué falta y avisa. El dashboard muestra ejecuciones, tasa de errores y tokens." |
| 2:50–3:00 | README / matriz de costos | "Costo por CV: medio centavo de dólar; el costo real es el orquestador. Repo, blueprint y documentos en GitHub." |

Claves, tokens y direcciones reales quedan fuera de cámara (las conexiones de Make no se abren en el video).

## Estructura del repositorio

```
docs/            PDFs de los 4 entregables documentales (+ fuentes HTML en docs/src)
blueprint/       blueprint del scenario exportado de Make + estructura de datos de la salida IA
screenshots/     evidencia de las corridas, Notion y Gmail
README.md        este índice
```
