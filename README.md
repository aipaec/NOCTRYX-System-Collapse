# NOCTRYX: SYSTEM COLLAPSE

> **The collapse is only the beginning.**

Videojuego de ciencia ficción oscura centrado en la exploración de una realidad digital inestable, el combate contra entidades hostiles y el descubrimiento de una anomalía que amenaza la existencia del sistema.

**Frase conceptual:** «Cuando el sistema cae, algo despierta en la oscuridad.»

## Visión del proyecto

NOCTRYX busca construir una experiencia de ciencia ficción con identidad audiovisual propia. La atmósfera, la narrativa, la música, la exploración y la jugabilidad deben trabajar juntas para transmitir misterio, tensión y descubrimiento.

El proyecto evoluciona a partir del prototipo académico NEXUS V2.0. Se conservarán sus componentes útiles después de una auditoría técnica; el cambio de identidad no implica descartar automáticamente el trabajo previo.

## Pilares de diseño

- **Exploración:** recorrer sectores digitales y descubrir espacios, señales y anomalías.
- **Combate:** enfrentarse a entidades hostiles mediante mecánicas claras y verificables.
- **Narrativa y misterio:** descubrir gradualmente el origen del colapso y la relación del protagonista con el sistema.
- **Identidad audiovisual:** combinar diseño visual, música, sonido e interfaz para crear una atmósfera reconocible.
- **Progresión:** objetivos, recursos, inventario y evolución del jugador con propósito.

## Identidad visual y sonora

Dirección artística provisional: ciencia ficción cinematográfica, estética digital oscura y elementos cyberpunk.

Paleta conceptual:
- Negro obsidiana.
- Cian eléctrico.
- Violeta profundo.
- Rojo carmesí.
- Blanco frío.

La música y el diseño sonoro tendrán un papel narrativo y ambiental, además de acompañar la acción.

## Premisa narrativa provisional

NOCTRYX es una realidad digital cuya estabilidad comienza a desmoronarse. Sectores enteros desaparecen, las entidades que habitaban el sistema dejan de responder a sus directivas y una señal desconocida emerge desde el núcleo.

El protagonista despierta con recuerdos incompletos. Su presencia parece estar relacionada con la anomalía, pero todavía se desconoce cómo y por qué.

Esta premisa es una base de trabajo: los detalles del universo, los personajes y los acontecimientos se definirán y documentarán antes de considerarlos canon definitivo.

## Tecnología y estado técnico

La base heredada del prototipo utiliza:
- C# y .NET.
- Windows Forms para la interfaz gráfica.
- Una separación inicial entre lógica de juego y presentación.

Los proyectos heredados conservan temporalmente sus nombres técnicos (NexusCore, NexusGUI y ConsoleApp3) mientras se realiza la auditoría. No se cambiará la arquitectura ni se moverán archivos sin una decisión técnica documentada.

La persistencia con base de datos es una posibilidad futura; no se considera implementada en esta etapa.

## Colaboración

El proyecto puede utilizar herramientas de IA para apoyar tareas delimitadas de programación, revisión, documentación, diseño conceptual, narrativa y producción audiovisual.

Toda contribución debe:
1. Respetar la dirección creativa y la documentación vigente.
2. Indicar con claridad qué archivos o entregables modifica.
3. Evitar sobrescribir trabajo ajeno sin coordinación.
4. Diferenciar funciones implementadas, incompletas y propuestas.
5. Ser revisada e integrada con pruebas antes de declararse terminada.

La dirección del proyecto y la aprobación final de cambios corresponden al responsable humano.

## Organización prevista

La estructura de carpetas se definirá después de auditar el repositorio y el prototipo heredado. No se debe asumir que una estructura propuesta ya existe.

## Documentación principal

- [Biblia de producción](PRODUCTION_BIBLE.md): visión, pilares, premisa narrativa, dirección artística y plan de producción.

## Estado de la demo — 23 de noviembre de 2026

**Estado de desarrollo reportado:** demo 2D jugable en C# con Windows Forms (.NET 10), desarrollada localmente a partir del proyecto NEXUS. Esta descripción refleja las pruebas manuales realizadas en el equipo de desarrollo; **el código fuente de la demo todavía no está publicado en este repositorio**.

### Funcionalidades de la demo local

- Protagonista ECHO-0: movimiento lateral, salto, gravedad y colisiones con plataformas.
- Dos escenarios jugables: ciudad industrial en colapso y realidad dimensional inestable.
- Dos tipos de enemigos: Centinela y Anomalía.
- Ataque con J, daño, alcance y tiempo de reutilización; integridad del jugador y enemigos.
- Cámara de seguimiento, interfaz de juego, menú, pausa y reinicio.
- Introducción narrativa, transiciones de mapa, interacción con la SEÑAL y final de la demo.
- Audio ambiental integrado en la versión local.

### Arquitectura local

- `Program.cs`: punto de entrada y secuencia inicial.
- `IntroForm`: introducción de la aplicación.
- `GamePrototypeForm`: formulario anfitrión del juego y coordinación de audio.
- `GameSurface`: control personalizado para actualización, dibujo y sistemas de jugabilidad.
- `AudioManager`: reproducción y gestión de recursos de audio.
- Proyectos heredados: `NexusGUI`, `NexusCore` y `ConsoleApp3`.

### Validación y pendientes

Se reportó una prueba manual satisfactoria de movimiento, plataformas, cámara, reinicio, puerta final y recorrido de ambos mapas hasta el cierre narrativo. Esto **no equivale** a una compilación automatizada del contenido publicado aquí.

**Pendiente crítico para la entrega:** sincronizar la solución y los archivos fuente locales con este repositorio, excluir carpetas generadas (`bin/`, `obj/`, `.vs/`) y revisar rutas absolutas y licencias de recursos antes de compartir una versión reproducible. No subir archivos de música comercial sin permiso.

La documentación de diseño existente describe también ideas y planes futuros; no debe interpretarse íntegramente como funcionalidad implementada.

## Estado del proyecto

**Etapa actual:** demo local jugable, pendiente de publicación del código fuente en GitHub.

- [x] Repositorio de NOCTRYX creado.
- [x] README inicial publicado.
- [ ] Biblia de producción revisada y aprobada.
- [x] Auditoría técnica del prototipo realizada mediante revisión de código.
- [x] Alcance y prioridades de la demo definidos.
- [ ] Identidad visual y sonora desarrollada.
- [x] Demo local integrada y probada manualmente (código aún no publicado aquí).
- [ ] Documentación y evidencias de entrega.

---

*Documento vivo. Las decisiones provisionales se actualizarán cuando sean revisadas y aprobadas.*
