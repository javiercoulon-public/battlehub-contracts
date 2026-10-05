# Plantilla de Traspaso (Handover)

Esta plantilla la completa el **equipo saliente** y se guarda como `/docs/traspaso.md` en el repositorio del servicio. El **equipo entrante** completa la sección 9 al finalizar su onboarding. Ver `08-etapa-2-rotacion-de-equipos.md` para el contexto.

- **Repositorio**: `battlehub-...`
- **Equipo saliente**: Equipo N (integrantes y contacto)
- **Equipo entrante**: Equipo M (integrantes y contacto)
- **Fecha del traspaso**: AAAA-MM-DD (UTC)

## 1. Resumen del servicio

En 3 a 5 líneas: qué hace el servicio, qué contratos de `battlehub-contracts` implementa y con qué otros servicios se comunica.

## 2. Cómo levantar el proyecto

Debe poder seguirse **sin ayuda del equipo saliente**. Incluir:

- Requisitos previos (versiones de .NET, Node, Docker, motor de BD).
- Pasos exactos, en orden, con los comandos.
- Variables de entorno necesarias (deben estar en `.env.example`, nunca con valores reales).
- Cómo cargar datos iniciales (por ejemplo, el script del banco de preguntas de Trivia).
- Cómo verificar que el servicio quedó funcionando (URL, endpoint de prueba).

## 3. Arquitectura y estructura del código

- Diagrama o descripción de los componentes principales.
- Estructura de carpetas y qué contiene cada una.
- Dónde está la lógica de negocio y dónde las integraciones externas.

## 4. Persistencia

- Motor de base de datos y ADR que lo justifica.
- Cómo se aplican las migraciones.
- Cómo se crea una base de datos limpia para desarrollo y pruebas.
- Tablas o colecciones principales.

## 5. Pruebas y CI

- Cómo ejecutar las pruebas unitarias y las de integración localmente.
- Qué cubren y qué **no** cubren las pruebas actuales.
- Estado actual del pipeline de CI y cualquier problema conocido.

## 6. Autenticación y configuración externa

- Qué aplicación de Auth0 usa el servicio (nombre, sin secretos) y qué audience espera.
- Qué claims o permisos usa.
- Qué credenciales se rotarán en el traspaso y quién lo hará.

## 7. ADRs y decisiones

| ADR | Estado | Resumen | ¿Sigue vigente? |
|---|---|---|---|
| ADR-NNN | Propuesto / Aceptado | ... | Sí / Pendiente / Reemplazar |

Decisiones tomadas que no están en un ADR y el equipo entrante debe conocer.

## 8. Estado actual y deuda técnica

| Tema | Descripción | Prioridad sugerida |
|---|---|---|
| Bug conocido | ... | Alta / Media / Baja |
| Funcionalidad incompleta | ... | ... |
| Deuda técnica | ... | ... |
| Riesgo | ... | ... |

## 9. Verificación del equipo entrante

Se completa al finalizar el onboarding.

- [ ] Levanté el proyecto localmente siguiendo solo el README y esta guía.
- [ ] Ejecuté las pruebas unitarias y de integración y pasaron.
- [ ] Leí los ADRs relacionados.
- [ ] Asistí a la sesión de explicación con el equipo saliente.
- [ ] Corregí o reporté los problemas encontrados en el README y en esta guía.

**Dudas pendientes y puntos que quedaron poco claros:**

- ...

**Primeros pasos planeados (primer PR, primera funcionalidad):**

- ...

## 10. Cierre

- [ ] El equipo saliente confirma que el traspaso está completo.
- [ ] El equipo entrante confirma que recibió lo necesario.
- [ ] El Tech Lead confirma el cierre.
