# Bugs Frontend

Auditoria realizada sobre `proyectoFrontend1` con enfoque de depuracion frontend e integracion con `BackendProyecto1`.

## 1. URL de backend incorrecta en ejemplo de entorno

- Archivo: `proyectoFrontend1/.env.example`
- Bug: `BACKEND_URL` apuntaba a `http://localhost:3005`.
- Impacto: al copiar `.env.example` a `.env.local`, el frontend intentaria llamar a un backend en un puerto incorrecto y fallarian login, datos server-side y proxy API.
- Fix: se cambio a `http://localhost:3000`, que coincide con el puerto por defecto del backend NestJS.
- Estado: corregido.

## 2. Proxy de roles no protegia subrutas

- Archivo: `proyectoFrontend1/src/proxy.ts`
- Bug: la validacion de area solo comparaba rutas exactas como `/admin`, `/docente` y `/estudiante`.
- Impacto: subrutas como `/admin/usuarios`, `/docente/grupos` o `/estudiante/matricula` no eran redirigidas si el rol no correspondia. La seguridad real seguia en backend, pero la UX y proteccion previa del frontend eran incorrectas.
- Fix: la comparacion ahora acepta la ruta base y cualquier subruta con `pathname.startsWith(`${p}/`)`.
- Estado: corregido.

## 3. Payload incorrecto al matricular un grupo

- Archivo: `proyectoFrontend1/src/app/(app)/estudiante/matricula/enroll-view.tsx`
- Bug: el frontend enviaba `{ group: g.group }`.
- Endpoint backend: `POST /api/enrollments` espera `groupId` segun `CreateEnrollmentDto`.
- Impacto: la matricula fallaba por validacion del DTO porque `groupId` no llegaba.
- Fix: se cambio el body a `{ groupId: g.group }`.
- Estado: corregido.

## 4. Metodo HTTP incorrecto al cancelar matricula desde estudiante

- Archivo: `proyectoFrontend1/src/app/(app)/estudiante/materias/cancel-button.tsx`
- Bug: el frontend llamaba `PATCH /enrollments/:id/cancel`.
- Endpoint backend: `POST /api/enrollments/:id/cancel`.
- Impacto: la cancelacion devolvia 404/405 o no llegaba al controlador correcto.
- Fix: se cambio el metodo a `POST`.
- Estado: corregido.

## 5. Metodo HTTP incorrecto al cancelar matricula desde admin

- Archivo: `proyectoFrontend1/src/components/admin/operations.tsx`
- Bug: la operacion administrativa de cancelar matricula tambien usaba `PATCH /enrollments/:id/cancel`.
- Endpoint backend: `POST /api/enrollments/:id/cancel`.
- Impacto: la accion administrativa fallaba aunque el endpoint existiera en backend.
- Fix: se cambio el metodo a `POST`.
- Estado: corregido.

## 6. Boton de guardar nombre siempre deshabilitado

- Archivo: `proyectoFrontend1/src/app/(app)/cuenta/account-forms.tsx`
- Bug: existia el estado `dirty`, pero `setDirty` nunca se usaba. Como el boton dependia de `!dirty`, quedaba siempre deshabilitado.
- Impacto: el usuario no podia actualizar su nombre desde la pagina de cuenta.
- Fix: se elimino el estado `dirty` y se reemplazo por la condicion derivada `nameChanged = newName.trim() !== name`.
- Estado: corregido.

## 7. Proxy frontend llamaba al prefijo incorrecto del backend

- Archivos: `proyectoFrontend1/src/lib/server.ts`, `proyectoFrontend1/src/app/api/[...path]/route.ts`, `proyectoFrontend1/src/app/api/auth/login/route.ts`
- Bug: el frontend reenviaba llamadas al backend como `${BACKEND_URL}/api/...`, pero el backend NestJS expone sus rutas bajo `/api/v1/...`.
- Impacto: login, consultas server-side y llamadas del proxy devolvian 404 aunque el backend estuviera levantado.
- Fix: se agrego `BACKEND_API_URL = ${BACKEND_URL}/api/v1` y todas las llamadas server/proxy ahora usan ese prefijo.
- Estado: corregido.

## 8. Tarjeta de docentes mostraba el conteo de estudiantes

- Archivo: `proyectoFrontend1/src/app/(app)/admin/page.tsx`
- Bug: la tarjeta "Docentes activos" leía `d.active.students`.
- Impacto: el panel repetía el total de estudiantes y mostraba una cifra incorrecta para docentes.
- Fix: la tarjeta ahora lee `d.active.teachers`, que es el campo correspondiente de la respuesta del dashboard.
- Estado: corregido.

## 9. Conteos de matrículas consultaban la etiqueta traducida

- Archivo: `proyectoFrontend1/src/app/(app)/admin/page.tsx`
- Bug: los conteos por estado se consultaban con etiquetas visibles como "Activas", aunque el backend indexa `enrollmentsByStatus` con claves como `activa`.
- Impacto: los conteos aparecían como cero aunque hubiera matrículas en esos estados.
- Fix: se usa la clave del estado para consultar los datos y se conserva la etiqueta traducida solo para mostrarla.
- Estado: corregido.

## 10. La planilla rechazaba notas con coma decimal

- Archivo: `proyectoFrontend1/src/app/(app)/docente/grupos/[id]/grade-sheet-panel.tsx`
- Bug: el comentario indicaba que se aceptaban coma y punto, pero el parser solo admitía punto decimal.
- Impacto: entradas como `4,5` se marcaban inválidas y bloqueaban el guardado de notas.
- Fix: se normaliza la coma a punto antes de validar y convertir la nota.
- Estado: corregido.

## 11. Texto semantico de alertas y badges no alcanzaba contraste WCAG AA

- Archivo: `proyectoFrontend1/src/app/globals.css`
- Bug: los colores de texto danger, success y warning sobre sus fondos claros tenian contrastes de 3.93:1, 3.83:1 y 3.17:1, por debajo del minimo 4.5:1 para texto normal.
- Impacto: errores, confirmaciones, advertencias y estados podian ser dificiles de leer para personas con baja vision o en pantallas con poco contraste.
- Fix: se oscurecieron los tres tokens semanticos; los nuevos pares alcanzan 6.10:1 (danger) y 6.30:1 (success y warning).
- Estado: corregido y contrastes recalculados.

## 12. El menu movil modal no gestionaba el teclado

- Archivo: `proyectoFrontend1/src/components/app-shell.tsx`
- Bug: el dialogo declaraba `aria-modal="true"`, pero Tab podia sacar el foco del menu y Escape no lo cerraba.
- Impacto: usuarios de teclado y lectores de pantalla podian interactuar con contenido de fondo sin salir del dialogo.
- Fix: el foco se mueve al boton de cierre al abrir, queda confinado al menu, Escape lo cierra y el foco vuelve al boton que lo abrio.
- Estado: corregido; lint y build pasan. La repeticion interactiva posterior quedo pendiente porque la sesion del navegador expiro.

## 13. Las opciones inactivas del menu eran invisibles

- Archivo: `proyectoFrontend1/src/components/app-shell.tsx`
- Bug: los enlaces inactivos usaban texto blanco sobre el fondo blanco del menu; solo se veia la opcion activa y las demas parecian no existir.
- Impacto: las personas no podian descubrir ni seleccionar las otras pantallas desde la navegacion.
- Fix: el estado inactivo ahora usa `text-ink` desde el primer render; alcanza contraste 17.33:1 sobre blanco y no depende del hover.
- Estado: corregido; lint pasa.

## 14. Cerrar sesion solo redirigia y no eliminaba la cookie

- Archivo: `proyectoFrontend1/src/components/app-shell.tsx`
- Bug: la accion de cerrar sesion redirigia al login sin llamar a `/api/auth/logout`.
- Impacto: la cookie `httpOnly` seguia activa y una visita posterior a una ruta protegida podia restaurar la sesion.
- Fix: se llama al endpoint de logout para borrar la cookie antes de redirigir.
- Estado: corregido.

## 15. Titulos de seccion de navegacion eran invisibles

- Archivo: `proyectoFrontend1/src/components/app-shell.tsx`
- Bug: los titulos de seccion y "Cuenta" usaban texto blanco sobre el panel blanco.
- Impacto: se perdia la agrupacion visual del menu y la orientacion entre opciones.
- Fix: los titulos usan `text-muted` sobre el fondo claro.
- Estado: corregido.

## 16. Horarios del sabado se mostraban bajo viernes

- Archivo: `proyectoFrontend1/src/components/week-schedule.tsx`
- Bug: la agrupacion de dias limitaba las columnas al viernes y juntaba en esa tarjeta las clases del sabado.
- Impacto: el estudiante o docente podia confundir el dia real de una clase.
- Fix: cada tarjeta ahora lee unicamente `byDay[day]`, incluido sabado.
- Estado: corregido.

## Hallazgos pendientes de esta pasada

Los siguientes bugs fueron confirmados durante la revision, pero se dejan abiertos en este envio para no presentar la auditoria como terminada.

## 17. Matricula desde el administrador envia un campo incorrecto

- Archivo: `proyectoFrontend1/src/components/admin/operations.tsx`
- Bug: el formulario envia `group`, pero `CreateEnrollmentDto` espera `groupId`.
- Impacto: la matricula administrativa falla en la validacion del backend.
- Estado: pendiente.

## 18. Etiquetas de dia y estado contienen errores

- Archivo: `proyectoFrontend1/src/lib/format.ts`
- Bug: lunes aparece como `Lrrrrunes` y el estado `reprobada` se etiqueta como "Aprobada".
- Impacto: horarios y estados academicos muestran informacion incorrecta.
- Estado: pendiente.

## 19. Una nota de 3.0 se colorea como reprobada

- Archivo: `proyectoFrontend1/src/app/(app)/estudiante/notas/page.tsx`
- Bug: la nota individual usa `value <= PASSING` para el color de reprobado, aunque el backend aprueba con `finalGrade >= 3.0`.
- Impacto: la nota minima aprobatoria se muestra en rojo.
- Estado: pendiente.

## 20. El acumulado parcial no respeta los pesos de evaluacion

- Archivo: `proyectoFrontend1/src/app/(app)/estudiante/notas/page.tsx`
- Bug: se calcula la media aritmetica de las notas registradas, no la suma ponderada que usa el backend.
- Impacto: el acumulado mostrado puede diferir de los puntos reales obtenidos segun el porcentaje de cada evaluacion.
- Estado: pendiente.

## 21. El tono ambar de algunas materias tiene bajo contraste

- Archivos: `proyectoFrontend1/src/lib/format.ts`, `proyectoFrontend1/src/app/globals.css`
- Bug: `accent-600` sobre `accent-100` tiene contraste 2.48:1.
- Impacto: codigos de materia con ese tono son dificiles de leer sobre su fondo.
- Estado: pendiente.

## 22. Los controles de cada franja horaria repiten IDs

- Archivo: `proyectoFrontend1/src/components/admin/resource-manager.tsx`
- Bug: los campos "Dia", "Inicio", "Fin" y "Salon" repiten el mismo ID al agregar varias franjas.
- Impacto: las etiquetas pueden asociarse al control equivocado y el arbol de accesibilidad contiene IDs duplicados.
- Estado: pendiente.

## 23. El formulario de restablecer clave valida menos que el backend

- Archivo: `proyectoFrontend1/src/components/admin/configs.tsx`
- Bug: el boton solo exige ocho caracteres, pero el backend tambien exige letras y numeros.
- Impacto: el formulario permite enviar claves que el backend rechaza.
- Estado: pendiente.

## Verificacion

- `npm run lint`: pasa sin errores ni warnings.
- `npm run build`: pasa correctamente con Next.js 16.3.8.
- Flujo autenticado: login, `GET /api/users/me` y `/admin` responden 200.
- Navegacion admin: los 13 destinos configurados responden 200 con sesion de administrador.
- Salud del backend: `GET http://localhost:3000/api/v1/health` responde 200.
- Contraste de los tokens danger, success y warning: 6.10:1, 6.30:1 y 6.30:1.
- Sin scroll horizontal en la pantalla de login a 390 px y 1440 px de ancho.

Nota: el build muestra un warning de Turbopack indicando que Next ignora `C:\Users\lucia\package-lock.json` por estar fuera del root del proyecto. No bloquea compilacion ni runtime.
