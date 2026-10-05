# Etapa 2: Rotación de Equipos

Este documento define cómo funciona la etapa 2 de BattleHub. Los equipos **rotan de servicio**: cada equipo hereda el repositorio y el servicio que construyó otro equipo en la etapa 1. Todo lo definido en los documentos 01 a 07 sigue vigente, salvo lo que este documento modifique de forma explícita (ver sección 9).

## 1. Objetivos de la etapa 2

- Trabajar sobre código que no escribimos nosotros: leerlo, entenderlo, extenderlo y corregirlo.
- Practicar colaboración real entre equipos: **fork, Pull Request y code review entre equipos distintos**.
- Reforzar la importancia de los contratos, los ADRs, las pruebas y el README: si otro equipo no puede levantar y entender el servicio, la documentación de la etapa 1 fue insuficiente.
- Experimentar con tecnologías distintas a las que se usaron en la etapa 1, por ejemplo otro motor de base de datos (SQL Server, MySQL, MongoDB) o el frontend en lugar del backend.

## 2. Mapa de rotación

Cada equipo hereda el repositorio del equipo siguiente. El Equipo 6 hereda el del Equipo 1.

| Equipo | Hereda (equipo entrante de) | Stack principal que encontrará |
|---|---|---|
| Equipo 1 | `battlehub-matchmaking` (Equipo 2) | .NET 10, MongoDB, SignalR, `BackgroundService` |
| Equipo 2 | `battlehub-shell` (Equipo 3) | Aurelia, Module Federation, Auth0 en el cliente |
| Equipo 3 | `battlehub-game-typing` (Equipo 4) | Aurelia + .NET 10, MySQL, SignalR |
| Equipo 4 | `battlehub-game-trivia` (Equipo 5) | Aurelia + .NET 10, SQL Server, SignalR |
| Equipo 5 | `battlehub-game-memory` (Equipo 6) | Aurelia + .NET 10, SQL Server, SignalR |
| Equipo 6 | `battlehub-profile-service` (Equipo 1) | .NET 10, MySQL, Auth0 (validación JWT) |

> El Tech Lead puede ajustar este mapa antes de iniciar la etapa. Cualquier cambio se registra en este archivo mediante un PR.

### Terminología

| Término | Significado |
|---|---|
| **Equipo saliente** | Equipo que construyó el repositorio en la etapa 1. |
| **Equipo entrante** | Equipo que hereda el repositorio en la etapa 2. |
| **Repo heredado** | Repositorio que un equipo recibe. |
| **Repo original** | El mismo repositorio, visto desde el equipo saliente. |

Cada equipo tiene **dos roles a la vez** durante la etapa 2:

1. **Equipo entrante** en el repo que hereda: es quien desarrolla.
2. **Equipo saliente** en el repo que construyó: es quien revisa los PRs, responde dudas y mantiene el conocimiento histórico del servicio.

## 3. Modelo de colaboración: fork + PR

### Regla general

El equipo entrante **no hace push directo** al repo heredado, ni siquiera si tiene permisos para hacerlo. Todo cambio entra por **fork + Pull Request**, con el mismo flujo de `07-como-contribuir.md`, aplicado ahora a los 6 repos de equipo.

| Aspecto | Regla en etapa 2 |
|---|---|
| Administración del repo (admin) | Se mantiene en el equipo saliente, salvo que el Tech Lead autorice un traspaso formal. |
| Quién desarrolla | El equipo entrante, desde un fork. |
| Quién revisa y aprueba PRs | El equipo saliente (mínimo 1 aprobación, ver sección 4). |
| Quién hace merge a `main` | El equipo saliente, una vez aprobado el PR y con el CI en verde. |
| Tech Lead | Sigue como colaborador con permiso de escritura en cada repo (ver `01-gobernanza-repositorios.md`). Puede comentar y aprobar PRs. |

### Alternativa: repositorio compartido

Si un equipo entrante y uno saliente acuerdan trabajar sin fork (por ejemplo, para un repo muy pequeño), pueden usar colaboración directa en el repo. Esto solo es válido si:

- El equipo saliente (admin del repo) agrega a los integrantes del equipo entrante como colaboradores con permiso de escritura.
- Se **sigue prohibiendo push directo a `main`**: todo cambio entra por PR desde una rama dentro del mismo repo.
- La regla de revisión cruzada de la sección 4 se mantiene.
- El acuerdo queda documentado en un ADR (ver sección 8).

**Por defecto se usa fork.** El repo compartido es una excepción que debe justificarse.

### Flujo del equipo entrante

```bash
# 1. Un integrante crea el fork del repo heredado en GitHub (botón Fork).
#    Se recomienda UN fork por equipo, en la cuenta de un integrante designado
#    (o en la organización del equipo, si existe), con todo el equipo como colaborador.

# 2. Clonar el fork
git clone https://github.com/<cuenta-del-equipo>/<repo-heredado>.git
cd <repo-heredado>

# 3. Agregar el repo original como upstream
git remote add upstream https://github.com/<cuenta-equipo-saliente>/<repo-heredado>.git

# 4. Sincronizar antes de empezar
git checkout main
git fetch upstream
git merge upstream/main
git push origin main

# 5. Crear rama de trabajo
git checkout -b feat/descripcion-del-cambio

# 6. Commit semántico, push a TU fork y abrir PR hacia main del repo original
git push origin feat/descripcion-del-cambio
```

Las restricciones de `07-como-contribuir.md` se mantienen: nunca se abre un PR desde `main` del fork, y siempre se sincroniza con `upstream` antes de empezar una rama nueva.

### Nota sobre CI en PRs desde forks

GitHub Actions **no entrega secretos del repositorio original** a los workflows que se ejecutan por un PR desde un fork. Por eso:

- El pipeline debe poder construir y probar el proyecto **sin secretos reales** (sin credenciales de Auth0 de producción, sin cadenas de conexión privadas).
- Las pruebas de integración deben usar bases de datos efímeras (Testcontainers o servicio de GitHub Actions), como ya exige `05-cicd-testing-commits.md`.
- Los valores de configuración deben leerse de variables de entorno, con un archivo `.env.example` versionado que documente cuáles se necesitan (ver `gitignore-template.txt`, que ya excluye `.env`).
- Si un PR de fork no puede validarse por falta de secretos, el equipo saliente lo anota en el PR y se resuelve con el Tech Lead. No se agregan secretos al repo para "arreglar" el pipeline.

## 4. Revisión cruzada y protección de `main`

Se mantiene la protección de rama de la etapa 1 (sin push directo, PR obligatorio, CI en verde, 1 aprobación). Se agrega lo siguiente:

- **La aprobación debe venir de un integrante del equipo saliente** (el dueño original del repo). Esto garantiza que alguien que conoce las decisiones originales valide el cambio.
- Se agrega un archivo `CODEOWNERS` en la raíz o en `.github/` del repo, con los integrantes del equipo saliente como responsables:

  ```text
  # Todos los cambios requieren revisión del equipo saliente
  *   @integrante-saliente-1 @integrante-saliente-2
  ```

- Se activa en la protección de `main` la opción **"Require review from Code Owners"**.
- El equipo saliente responde a los PRs en un plazo máximo de **[definir, por ejemplo 2 días hábiles]**. Si no hay respuesta, el equipo entrante lo avisa al Tech Lead.
- Un equipo no puede aprobar sus propios PRs.

### Checklist de PR (ampliación)

Además del checklist de `05-cicd-testing-commits.md` §4:

- [ ] El PR indica en la descripción qué equipo lo envía y qué problema o historia resuelve.
- [ ] Si el cambio toca un contrato de `battlehub-contracts`, referencia el ADR (ver sección 8).
- [ ] Si el cambio altera la forma de ejecutar el proyecto, actualiza el `README.md`.
- [ ] Si el cambio añade configuración nueva, actualiza `.env.example`.
- [ ] Fue aprobado por un integrante del equipo saliente.

## 5. Traspaso (handover) del equipo saliente

Antes de que el equipo entrante empiece a desarrollar, hay una **fase de traspaso** de [definir duración, por ejemplo 1 semana].

### Responsabilidades del equipo saliente

- Dejar el `README.md` actualizado y verificado: el equipo entrante debe poder levantar el proyecto localmente **siguiendo solo el README**.
- Completar el documento de traspaso usando la plantilla de `09-plantilla-traspaso.md` y guardarlo en `/docs/traspaso.md` del repo.
- Hacer una sesión de explicación (walkthrough) del código y la arquitectura con el equipo entrante, de [definir duración, por ejemplo 60 a 90 minutos].
- Listar la deuda técnica conocida, los bugs abiertos y las decisiones que quedaron pendientes.
- Confirmar que las pruebas pasan y que el CI está en verde en `main`.

### Responsabilidades del equipo entrante

- Levantar el proyecto localmente y **reportar cada problema encontrado** como issue o como PR de corrección al README. Si el README estaba incompleto, corregirlo es su primera contribución.
- Leer los ADRs del repo y los de `battlehub-contracts` relacionados con el servicio.
- Ejecutar las pruebas unitarias y de integración y confirmar que pasan en su entorno.
- Entregar al final del traspaso un resumen breve (puede ser en el mismo `/docs/traspaso.md`) con lo que entendió y las dudas que quedan.

### Criterio de cierre del traspaso

El traspaso se considera completo cuando se cumple todo lo siguiente:

- [ ] El equipo entrante levantó el proyecto localmente sin ayuda directa del equipo saliente.
- [ ] El equipo entrante ejecutó las pruebas y pasaron.
- [ ] Existe `/docs/traspaso.md` completo en el repo.
- [ ] Se realizó la sesión de explicación.
- [ ] El Tech Lead confirmó el cierre.

## 6. Recursos compartidos que cambian de responsable

### Cuenta de Auth0

La regla de la etapa 1 era: el Equipo 1 administra el tenant, y cada equipo administra el acceso a su aplicación. En la etapa 2, **la responsabilidad sigue al servicio**:

| Recurso en Auth0 | Etapa 1 | Etapa 2 |
|---|---|---|
| Tenant (dominios, conexiones, configuración general) | Equipo 1 | Equipo 6, como nuevo dueño de `battlehub-profile-service`. El Equipo 1 transfiere el rol de propietario (Owner) del tenant. |
| Aplicación del Shell | Equipo 3 | Equipo 2 |
| Aplicación de cada juego (si existe) | Equipos 4, 5 y 6 | Equipos 3, 4 y 5 respectivamente |
| Clientes M2M y APIs (por ejemplo, el scope `matches.finish` de ADR-004, si fue aceptado) | Equipo 1 con el Equipo 2 | Equipo 6 con el Equipo 1 |

Reglas de transición:

- El traspaso del rol de Owner del tenant lo hace el equipo saliente, **una vez cerrado el traspaso del repo** y en presencia del Tech Lead.
- El equipo saliente **no se elimina como colaborador** de Auth0 hasta el final de la etapa 2, para poder ayudar si algo falla. Pierde la capacidad de decidir sobre la aplicación.
- Ninguna credencial (Client Secret, claves M2M) se comparte por chat o por el repo. Se rotan al momento del traspaso.
- Dominio, audience y clientes de Auth0 siguen sin versionarse (ver `.gitignore` y ADR-006).

### Despliegue

Se mantiene el despliegue manual de la etapa 1. Quien despliega cada servicio es el **equipo entrante**. El Tech Lead compartirá instrucciones adicionales si las hay.

### Bases de datos

- El equipo entrante hereda el motor de base de datos elegido en la etapa 1 (por ejemplo SQL Server en Trivia y Memory, MySQL en Typing). **Cambiar de motor requiere un ADR nuevo** (sección 8) que reemplace al original, y la migración de datos debe estar documentada.
- Las bases de datos de desarrollo y de pruebas se recrean desde migraciones y scripts del repo. No se entregan volcados con datos personales.

## 7. Qué puede y qué no puede cambiar el equipo entrante

| Tipo de cambio | ¿Permitido? | Requisito |
|---|---|---|
| Corregir bugs | Sí | PR con prueba que reproduzca el bug |
| Agregar funcionalidad nueva dentro del contrato existente | Sí | PR + pruebas |
| Mejorar pruebas, README, CI | Sí | PR |
| Refactorizar sin cambiar comportamiento externo | Sí | PR + pruebas existentes en verde |
| Cambiar el motor de base de datos | Con condiciones | ADR nuevo que reemplace al anterior, aprobado por el Tech Lead |
| Modificar un contrato de `battlehub-contracts` (REST, SignalR, contrato visual, ciclo de vida) | Con condiciones | ADR aprobado por el Tech Lead **antes** de implementar (ver `06-adr-template.md`) |
| Cambiar el stack fijo (.NET 10, Aurelia, Module Federation) | No | Fuera del alcance del proyecto |
| Reescribir el servicio desde cero | No, salvo aprobación | Debe justificarse con un ADR y aprobarse por el Tech Lead |

## 8. ADRs en la etapa 2

- Los ADRs siguen guardándose **únicamente** en `/adrs` de `battlehub-contracts`, con numeración consecutiva (continúa después del último ADR de la etapa 1).
- Si el equipo entrante quiere cambiar una decisión tomada por el equipo saliente, **no edita el ADR original**. Crea un ADR nuevo y marca el anterior como `Reemplazado por ADR-XXX` mediante PR.
- El equipo saliente debe ser mencionado como **revisor** del ADR nuevo si este afecta a su decisión original.
- Los ADRs que seguían en estado `Propuesto` al terminar la etapa 1 (por ejemplo ADR-003, ADR-004, ADR-006 y ADR-007) deben resolverse (aceptarse, rechazarse o reemplazarse) **antes de iniciar el desarrollo de la etapa 2**, porque los equipos entrantes necesitan saber cuáles decisiones son firmes. Esto lo coordina el Tech Lead.
- Un ADR nuevo, o su actualización, que cambie una decisión técnica de un servicio debe indicar el equipo autor de la etapa en que se tomó (Etapa 1 o Etapa 2) en el campo `Equipo/Autor`.

## 9. Documentos de la etapa 1 afectados

| Documento | Cambio en etapa 2 |
|---|---|
| `01-gobernanza-repositorios.md` | La tabla de propiedad sigue igual (el repo conserva su admin), pero la regla "el equipo es responsable de su `main`" se amplía con la revisión cruzada de la sección 4. La sección de Auth0 se actualiza según la sección 6. |
| `02-arquitectura-y-flujo.md` | Las responsabilidades por equipo pasan a ser las del mapa de rotación de la sección 2. Los servicios y el flujo no cambian. |
| `03-contratos-tecnicos.md` | Se amplía, previa aprobación de los ADRs, con leaderboard, denuncias y los permisos `reports.create` y `reports.review` (ver `10-etapa-2-alcance-funcional.md`). |
| `04-persistencia-y-api-juegos.md` | Cada juego agrega `GET /api/games/{game}/leaderboard` a su API mínima. |
| `05-cicd-testing-commits.md` | Se agrega el requisito de que el CI funcione en PRs desde forks sin secretos (sección 3 de este documento), la ampliación del checklist de PR y la construcción obligatoria de la imagen Docker (paso 6, ver `10-etapa-2-alcance-funcional.md` §4). |
| `07-como-contribuir.md` | El flujo fork + PR, antes exclusivo de `battlehub-contracts`, se aplica ahora a los 6 repos de equipo. |

## 10. Alcance funcional de la etapa 2

El detalle completo está en `10-etapa-2-alcance-funcional.md`. En resumen, la etapa tiene cuatro frentes, en este orden de prioridad:

1. **Completar lo pendiente de la etapa 1** en el repo heredado.
2. **Leaderboard**: uno por juego y uno general en el Shell.
3. **Denuncia de usuarios** con verificación en varias etapas usando **Wolverine**.
4. **Contenerización** de cada aplicación con **Docker**.

| Equipo | Repo heredado | Entregables principales |
|---|---|---|
| Equipo 1 | `battlehub-matchmaking` | Deuda (ADR-004, ADR-007); verificar que las sanciones bloquean crear y unirse a partidas; Docker |
| Equipo 2 | `battlehub-shell` | Deuda (ADR-003); leaderboard general; botón "Denunciar" y panel de moderación; Docker |
| Equipo 3 | `battlehub-game-typing` | Deuda; leaderboard de Typing (API + microfrontend); Docker |
| Equipo 4 | `battlehub-game-trivia` | Deuda; leaderboard de Trivia (API + microfrontend); Docker |
| Equipo 5 | `battlehub-game-memory` | Deuda; leaderboard de Memory (API + microfrontend); Docker |
| Equipo 6 | `battlehub-profile-service` | Deuda; módulo de denuncias con Wolverine; Docker |

## 11. Criterios de evaluación sugeridos

> Ajustar los pesos según el criterio del curso.

| Criterio | Qué se evalúa |
|---|---|
| Traspaso | Calidad del traspaso entregado (equipo saliente) y del onboarding (equipo entrante). |
| Colaboración | PRs bien formados, commits semánticos, respuestas oportunas en las revisiones. |
| Revisión de código | Calidad de los comentarios que hace el equipo saliente en los PRs que recibe. |
| Calidad técnica | CI en verde, pruebas, README al día, contratos respetados. |
| Decisiones | ADRs claros cuando se cambia algo, con alternativas y consecuencias. |
| Documentación | `README.md`, `.env.example` y `/docs/traspaso.md` actualizados al finalizar. |

## 12. Resumen de reglas obligatorias

- Todo cambio al repo heredado entra por **fork + PR** (o por rama + PR si se acordó repo compartido con ADR). Nunca push directo a `main`.
- Cada PR debe ser **aprobado por un integrante del equipo saliente**.
- El traspaso debe cerrarse antes de empezar a desarrollar.
- Las decisiones anteriores no se editan: se **reemplazan con un ADR nuevo**.
- Los contratos de `battlehub-contracts` solo cambian mediante ADR aprobado por el Tech Lead.
- Ningún secreto se versiona ni se comparte por el repo; se rotan al traspasar.
- El CI debe pasar sin secretos del repo original.
