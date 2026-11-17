# NOCTRYX: SYSTEM COLLAPSE
## Diseño técnico — versión 0.1

**Estado:** arquitectura objetivo y especificación inicial  
**Base heredada auditada:** C#/.NET, Windows Forms, NexusCore, NexusGUI y ConsoleApp3  
**Importante:** este documento describe el diseño propuesto. No afirma que los módulos o sistemas aquí descritos estén implementados.

---

## 1. Propósito

Definir una arquitectura evolutiva que permita convertir el prototipo ProyectoNexusTron en NOCTRYX sin desechar innecesariamente la lógica útil ni reescribir todo de una vez.

El diseño debe:
- mantener una fuente única de verdad para el estado de la partida;
- separar reglas del juego y presentación;
- facilitar pruebas reproducibles;
- permitir incorporar decisiones narrativas y finales alternativos;
- reducir el riesgo de cambios grandes sobre código heredado.

## 2. Situación heredada

La auditoría estática de ProyectoNexusTron identificó tres proyectos:
- `NexusCore`: biblioteca con lógica principal y estado del juego;
- `NexusGUI`: cliente Windows Forms con movimiento y consultas parciales;
- `ConsoleApp3`: cliente de consola que sirve como referencia de flujos de mecánicas.

Hallazgos relevantes:
- `SistemaTronGrid` concentra inicialización, movimiento, eventos, combate, inventario, extracción y resultados textuales.
- WinForms no completa el flujo de combate y no permite usar objetos.
- El inventario puede rechazar un objeto mientras una directiva se marca como completada.
- No existe registro general de sectores visitados.
- Algunas operaciones del núcleo presuponen que existe una partida inicializada.
- La documentación heredada contiene diferencias con el comportamiento del código.

Estos hallazgos proceden de una revisión estática reportada por Codex. No equivalen a una compilación o prueba de ejecución independiente.

## 3. Principios de arquitectura

1. **Lógica independiente de interfaz:** el núcleo no debe depender de controles WinForms ni de consola.
2. **Estado centralizado:** una sola sesión de juego mantiene el estado autoritativo.
3. **Resultados estructurados:** las acciones devuelven resultados tipados; la interfaz decide cómo presentarlos.
4. **Reglas explícitas:** límites, probabilidades, condiciones de victoria y efectos deben ser verificables.
5. **Cambios incrementales:** extraer responsabilidades por etapas sin reescribir todo el prototipo.
6. **Pruebas reproducibles:** permitir controlar aleatoriedad y escenarios de prueba.
7. **Narrativa como estado de juego:** las decisiones importantes se registran de forma persistente dentro de la sesión.
8. **Sin dependencias prematuras:** no incorporar base de datos ni servicios externos hasta definir necesidades concretas.

## 4. Arquitectura lógica objetivo

La siguiente separación es conceptual; no representa carpetas existentes ni una orden inmediata de renombrado.

| Componente lógico | Responsabilidad |
|---|---|
| GameSession / GameState | Mantener estado y ciclo de vida de la partida |
| ExplorationSystem | Movimiento, límites, sectores, visitas y eventos |
| EncounterSystem | Crear, mantener y finalizar encuentros |
| CombatSystem | Ataques, daño, defensa, escape y derrota |
| InventorySystem | Capacidad, obtención, selección y uso de objetos |
| ObjectiveSystem | Directivas, misiones y condiciones de progreso |
| ChoiceSystem | Presentar y resolver decisiones narrativas |
| NarrativeState | Registrar decisiones, recuerdos, pruebas y consecuencias |
| EndingSystem | Evaluar condiciones de cierre y rutas |
| Presentation (WinForms) | Mostrar estado y enviar comandos del jugador |

La separación puede implementarse inicialmente como clases dentro del proyecto núcleo existente. No es obligatorio crear un proyecto .NET por cada componente.

## 5. Modelo de estado de partida

El estado debe distinguir al menos:

### 5.1 Estado de sesión
- identificador de partida;
- estado de inicialización;
- estado de pausa o cierre;
- jugador activo;
- posición actual;
- estado de victoria o derrota.

### 5.2 Estado de exploración
- sectores visitados;
- sectores descubiertos;
- eventos ya resueltos;
- entidades presentes;
- puntos de interés encontrados.

Debe definirse si los eventos son únicos por sector, si pueden repetirse y qué eventos se reinician al volver a entrar.

### 5.3 Estado de combate
- encuentro activo;
- entidad o entidades participantes;
- integridad y defensas;
- turno o fase actual, si se adopta combate por turnos;
- resultado: activo, victoria, escape o derrota.

El encuentro no debe desaparecer simplemente porque el jugador vuelva a pulsar movimiento. Debe terminar mediante una regla explícita.

### 5.4 Estado de inventario
- objetos poseídos;
- identificador y tipo;
- cantidad o condición de uso;
- capacidad;
- efectos aplicados.

La obtención de un objeto y el progreso de una misión deben actualizarse de forma coherente. Una directiva que requiere poseer un objeto solo puede completarse cuando la operación de inventario confirme la obtención.

### 5.5 Estado narrativo
Como mínimo:
- ruta elegida: no decidida, retirada o verdad;
- decisiones relevantes;
- pruebas completadas;
- fragmentos de memoria descubiertos;
- pistas y contradicciones observadas;
- encuentros especiales vistos;
- condiciones de desbloqueo del clímax;
- desenlace alcanzado.

No se recomienda usar únicamente contadores globales de moralidad. Es preferible guardar decisiones concretas y derivar consecuencias mediante reglas explícitas.

## 6. Contratos de acciones

Las operaciones del juego deben validar precondiciones y devolver resultados estructurados.

Ejemplo conceptual, no código final:

```csharp
public enum ActionStatus
{
    Success,
    Rejected,
    InvalidState,
    NotAvailable,
    Failed
}

public sealed class ActionResult
{
    public ActionStatus Status { get; init; }
    public string MessageKey { get; init; }
    public object Data { get; init; }
}
```

La interfaz puede convertir `MessageKey` en texto localizado y mostrar los datos correspondientes. El núcleo no debe depender de MessageBox ni de etiquetas WinForms.

Los contratos deben especificar:
- precondiciones;
- cambios de estado;
- resultado;
- efectos secundarios;
- casos de error.

## 7. Exploración híbrida

El sistema debe combinar:
- exploración libre de sectores;
- eventos aleatorios controlables;
- puntos de interés únicos;
- zonas de peligro;
- encuentros especiales de historia;
- eventos asociados a decisiones.

La generación aleatoria debe poder usar una fuente de aleatoriedad inyectable o una semilla configurable en pruebas. Así se podrán reproducir encuentros y errores.

El registro de sectores debe separar:
- visitado: el jugador estuvo allí;
- descubierto: conoce información o puntos de interés;
- resuelto: completó el evento principal;
- activo: existe un encuentro o amenaza pendiente.

## 8. Encuentros y combate

### 8.1 Ciclo de encuentro

1. Un evento crea un encuentro.
2. El estado de la sesión registra el encuentro activo.
3. La interfaz muestra información y acciones permitidas.
4. El jugador ataca, usa un objeto, se defiende o intenta escapar, según reglas aprobadas.
5. El núcleo resuelve la acción y devuelve un resultado.
6. El encuentro continúa o finaliza explícitamente.
7. La exploración se habilita nuevamente cuando corresponde.

### 8.2 CLU

CLU debe admitir dos contextos:
- encuentros especiales durante la progresión;
- enfrentamiento final desbloqueado por la ruta difícil.

El diseño debe distinguir una manifestación o encuentro especial del combate definitivo. La naturaleza de esas apariciones permanece pendiente de definición narrativa.

### 8.3 Reglas de combate pendientes

- combate en tiempo real o por turnos;
- defensa y enfriamientos;
- habilidades y recursos;
- comportamiento diferenciado de enemigos;
- condiciones de escape;
- muerte, derrota y reinicio;
- duración y fases del jefe.

## 9. Inventario y objetos

El inventario debe ofrecer operaciones separadas:
- consultar;
- añadir;
- seleccionar;
- usar;
- descartar, si el diseño lo permite.

Una operación de uso no debe eliminar un objeto hasta confirmar que el efecto puede aplicarse, salvo que el diseño defina explícitamente un consumo fallido.

Los objetos deben tener efectos reales y verificables. Los objetos sin efecto implementado deben marcarse como no utilizables o mantenerse como elementos narrativos sin consumirlos.

## 10. Objetivos y progreso narrativo

Los objetivos se modelarán con condiciones explícitas:
- condición de inicio;
- condición de progreso;
- condición de finalización;
- recompensa o consecuencia;
- relación con objetos, ubicaciones o decisiones.

El progreso de narrativa no debe depender exclusivamente de comparar cadenas de texto. Debe usar identificadores estables de decisiones, pruebas y eventos.

## 11. ChoiceSystem y NarrativeState

### 11.1 Flujo de decisión

1. El juego presenta contexto suficiente.
2. Se muestran opciones disponibles.
3. El jugador selecciona una opción.
4. El sistema registra la decisión con identificador y contexto.
5. Se aplican consecuencias inmediatas.
6. Se programan o habilitan consecuencias diferidas.
7. Otros sistemas consultan el estado narrativo sin modificarlo arbitrariamente.

### 11.2 Ruta principal

La decisión de retirada o continuación debe ser explícita y única en el punto narrativo definido. La ruta fácil:
- permite un cierre válido;
- evita las pruebas finales;
- no desbloquea el combate final de CLU;
- conserva deliberadamente sin resolver la identidad del protagonista.

La ruta difícil:
- habilita una secuencia de pruebas;
- registra resultados y sacrificios;
- permite reunir evidencias suficientes;
- desbloquea el enfrentamiento final con CLU cuando se cumplen las condiciones narrativas.

### 11.3 Presencia interior

La presencia vinculada a la mente del protagonista debe utilizar el estado narrativo para referirse a decisiones previas. Sus intervenciones no deben tener acceso omnisciente a datos que el protagonista no haya experimentado, salvo que la historia establezca esa capacidad.

El sistema debe poder registrar:
- apariciones;
- líneas o escenas vistas;
- decisiones que las activan;
- contradicciones observadas;
- pruebas asociadas.

La ambigüedad es un objetivo narrativo, no un estado aleatorio.

## 12. EndingSystem

El sistema de finales debe evaluar condiciones concretas, no una puntuación moral general.

Condiciones iniciales:
- retirada elegida → final de supervivencia sin revelación de identidad;
- ruta difícil completada → acceso al clímax con CLU;
- resultado del clímax y decisiones previas → desenlace correspondiente, pendiente de diseño.

No se debe implementar el sistema de finales hasta que narrativa y condiciones de cada desenlace estén aprobadas.

## 13. Integración con Windows Forms

WinForms debe actuar como cliente de presentación:
- enviar comandos al núcleo;
- recibir resultados;
- mostrar estado de jugador, sector, encuentro, inventario y objetivos;
- habilitar o deshabilitar controles según el estado actual;
- evitar que una acción inválida altere la partida.

La interfaz no debe duplicar reglas de daño, inventario o progresión. Debe consultar el estado y presentar las acciones permitidas.

## 14. Persistencia

La persistencia queda fuera de la primera fase de estabilización. Antes de seleccionar SQL Server u otra tecnología, definir:
- qué datos se guardan;
- si habrá una o varias ranuras;
- qué versiones de partida deben ser compatibles;
- cómo se valida y recupera una partida dañada;
- si el guardado será local.

No se debe declarar SQL Server implementado hasta que exista código de conexión, esquema, operaciones y pruebas.

## 15. Plan de migración incremental

| Fase | Objetivo | Criterio de salida |
|---|---|---|
| 0. Línea base | Congelar y comprender el prototipo | Auditoría, copia de seguridad y pruebas manuales documentadas |
| 1. Integridad del núcleo | Corregir precondiciones y coherencia de estado | Inventario, directivas y estado inicial consistentes |
| 2. Contratos | Estandarizar resultados de acciones | UI no depende de interpretar mensajes para decidir reglas |
| 3. Encuentros | Completar ciclo de combate en GUI | Encuentro reproducible, resoluble y persistente hasta finalizar |
| 4. Inventario | Conectar consulta y uso | Objetos aplican efectos verificables desde GUI |
| 5. Exploración | Registrar sectores y eventos | Visitas y eventos siguen reglas documentadas |
| 6. Narrativa | Incorporar decisiones y pruebas | Decisiones se registran y afectan progresión |
| 7. Presentación | Mejorar interfaz, arte y sonido | Experiencia legible, coherente y estable |
| 8. Persistencia | Diseñar e implementar guardado | Guardar/cargar con pruebas de integridad |

Las fases pueden ajustarse según alcance y disponibilidad. No implican una reescritura total ni una fecha fija.

## 16. Verificación y calidad

Cada sistema debe contar con escenarios reproducibles. Ejemplos:
- iniciar una partida y rechazar acciones antes de inicializar;
- llenar el inventario e intentar obtener un objeto de misión;
- generar un encuentro y comprobar que no desaparece por moverse;
- atacar hasta derrotar una entidad y verificar recompensas;
- usar un objeto válido e inválido;
- volver a visitar un sector y comprobar su política de eventos;
- elegir retirada y verificar que el jefe final no se desbloquea;
- elegir la ruta difícil y verificar que las pruebas habilitan el clímax solo cuando corresponde.

La aceptación requiere evidencia de compilación y pruebas realizadas. Una revisión estática no demuestra funcionamiento.

## 17. Restricciones de colaboración técnica

- Codex puede auditar e implementar tareas delimitadas.
- Cada tarea debe declarar archivos permitidos y prohibidos.
- No ejecutar cambios simultáneos sobre los mismos archivos sin coordinación.
- Antes de implementar, revisar el estado actual del repositorio y preservar trabajo no comprometido.
- Toda modificación debe indicar archivos cambiados, motivo, pruebas y limitaciones.
- No renombrar proyectos ni mover carpetas sin aprobación expresa.

## 18. Decisiones técnicas pendientes

- Versión .NET objetivo definitiva y compatibilidad del entorno.
- Modelo de combate.
- Política de reaparición de enemigos y eventos.
- Diseño de mapa y tamaño de sectores.
- Modelo de datos para entidades, objetos y objetivos.
- Estrategia de pruebas automatizadas o manuales.
- Organización final de proyectos y namespaces.
- Persistencia y formato de guardado.
- Localización y gestión de textos.
- Alcance realista de la primera demo.


## 20. Modelo técnico de identidad y memoria narrativa

**Estado:** propuesta de diseño; no implementada.

El estado narrativo debe registrar hechos jugables y decisiones concretas, no almacenar una única explicación de la historia. Se recomienda separar:

### 20.1 Datos del protagonista
- designación temporal mostrada;
- identificador interno estable de partida;
- recuerdos recuperados;
- pistas observadas;
- interpretación actual, si el diseño decide mostrarla.

La designación temporal no debe confundirse con la identidad verdadera. La identidad real no se guarda como un dato revelado al inicio.

### 20.2 Registro de evidencias
Cada pista importante debe tener un identificador único y metadatos:
- origen (registro, entidad, escena, recuerdo);
- acto o sector;
- momento en que se descubrió;
- contenido observado;
- interpretación inicial opcional;
- contradicciones relacionadas;
- escenas que puede desbloquear.

El sistema registra qué vio el jugador. No debe marcar una hipótesis como verdadera solo porque un personaje la afirmó.

### 20.3 Registro de decisiones
Cada elección relevante debe almacenar:
- identificador de decisión;
- opción elegida;
- contexto o escena;
- consecuencias inmediatas aplicadas;
- condiciones diferidas;
- pruebas y diálogos que puede habilitar.

Evitar un valor global de «bondad», «cordura» o «maldad» como sustituto de decisiones específicas. Si en el futuro se añade una variable abstracta, deberá complementar y no reemplazar el registro de elecciones.

### 20.4 Apariciones de la presencia interior
Registrar por separado:
- aparición vista;
- contexto que la activó;
- línea o escena reproducida;
- pistas que el protagonista ya conocía;
- decisión asociada;
- estado de progresión narrativa.

Esto permite que la presencia recuerde elecciones pasadas sin otorgarle conocimiento arbitrario.

### 20.5 Reglas de coherencia y pruebas
El contenido narrativo debe poder comprobarse mediante escenarios:
- una línea no hace referencia a una pista que el jugador aún no descubrió, salvo justificación explícita;
- una decisión previa produce la consecuencia prevista;
- una pista contradictoria no borra el hecho de que el jugador vio la versión anterior;
- las escenas no se repiten indebidamente;
- la retirada cierra la ruta sin desbloquear por error las pruebas finales o el combate final de CLU;
- la ruta difícil requiere condiciones verificables para habilitar el clímax.

### 20.6 Separación entre datos y guion
Los identificadores de decisiones, pistas y escenas deben ser estables y no depender del texto visible. Los diálogos pueden revisarse o localizarse sin romper el progreso guardado.

## 21. Propuesta de hitos narrativos programables

| Hito | Evento | Estado que debe registrarse |
|---|---|---|
| N-01 | Despertar | Inicio de sesión narrativa y primera escena |
| N-02 | Primera anomalía | Pista descubierta y sector asociado |
| N-03 | Primera intervención interior | Aparición y contexto |
| N-04 | Contradicción verificable | Dos evidencias relacionadas |
| N-05 | Encuentro especial con CLU | Manifestación vista y resultado |
| N-06 | Gran elección | Ruta elegida, sin ambigüedad |
| N-07A | Retirada | Final alternativo y jefe final bloqueado |
| N-07B | Aceptación de pruebas | Ruta difícil y pruebas habilitadas |
| N-08 | Confrontación final | Condiciones de acceso y resultado de CLU |

Los hitos son identificadores de planificación, no nombres definitivos de clases ni una orden de implementación.



## 22. Modelo de recuerdos de dos realidades

**Estado:** requisito narrativo aprobado; arquitectura propuesta, no implementada.

El sistema narrativo debe poder registrar recuerdos que el protagonista atribuye a dos líneas distintas sin decidir automáticamente cuál es objetiva. Se propone ampliar el registro de evidencias con los siguientes campos conceptuales:

- identificador estable del recuerdo;
- línea de memoria: vida distinta, NOCTRYX antes del colapso o sin clasificar;
- formato: imagen, sonido, diálogo, sensación, escena o registro;
- contenido percibido por el protagonista;
- contexto en que aparece;
- grado de integridad del fragmento;
- pistas externas que lo corroboran o contradicen;
- interpretación expresada por ECHO-0, si existe;
- decisiones del jugador relacionadas.

### 22.1 Regla de separación

El sistema no debe almacenar un booleano simplista como «recuerdo verdadero/falso». Debe conservar por separado:
- el recuerdo experimentado;
- la evidencia externa disponible;
- las contradicciones conocidas;
- la interpretación actual del protagonista.

Así, una escena posterior puede reinterpretar un recuerdo sin borrar el hecho de que el jugador lo experimentó.

### 22.2 Catástrofe recordada

La catástrofe debe modelarse como una secuencia de fragmentos relacionados, no como una única cinemática que revela todo. Los fragmentos pueden tener orden parcial, datos ausentes y versiones que parecen incompatibles.

El sistema debe permitir que:
- el jugador encuentre fragmentos fuera de orden;
- algunos recuerdos se activen por ubicación, diálogo o decisión;
- la presencia interior reaccione a recuerdos ya experimentados;
- las pistas del mundo contrasten con la memoria subjetiva;
- las respuestas seleccionables registren la postura de ECHO-0.

### 22.3 Validaciones narrativas

- Ningún diálogo debe tratar una interpretación como verdad confirmada sin evidencia narrativa.
- Una pista que contradice un recuerdo no debe eliminarlo del historial.
- La misma señal, frase u objeto puede aparecer en ambas líneas con diferencias deliberadas.
- Las rutas de retirada y verdad deben seguir siendo válidas aunque el jugador haya descubierto pocos recuerdos.
- La implementación futura debe permitir revisar qué fragmentos vio el jugador y en qué orden.



## 23. Modelo técnico del vínculo ambiguo y las decisiones de sacrificio

**Estado:** propuesta técnica; no implementada. Depende de la dirección narrativa aprobada para la ambigüedad del vínculo y el dilema individual/colectivo.

### 23.1 Recuerdos relacionales sin verdad prematura

No almacenar la relación de ECHO-0 con la mujer como un único campo definitivo (por ejemplo, `Relationship = Wife`). Cada recuerdo debe conservar su propia afirmación o perspectiva:
- identificador del recuerdo;
- persona o entidad referida, si puede identificarse;
- vínculo que el fragmento parece describir (esposa, prometida, otro/no determinado);
- fuente del recuerdo;
- integridad y contexto;
- evidencias que lo apoyan o contradicen;
- momento en que el jugador lo descubrió.

El estado global registra qué versiones conoce el jugador, no cuál es verdadera. La resolución futura deberá depender de evidencias y condiciones narrativas explícitas, no de sobrescribir los recuerdos anteriores.

### 23.2 Decisión de una vida frente a muchas

La elección debe modelarse como una decisión narrativa identificable, no como una suma de puntos morales. El registro debe incluir:
- escena y contexto;
- opciones que realmente estaban disponibles;
- información presentada al jugador;
- opción elegida;
- consecuencias inmediatas;
- consecuencias diferidas;
- personajes, sectores y recursos afectados;
- evidencias o rutas que se habilitan o bloquean.

La alternativa de buscar una tercera salida solo debe aparecer si se cumplen condiciones explícitas, como haber descubierto una pista, conservar un recurso o completar una acción previa. No debe aparecer por una tirada aleatoria que haga imposible reproducir el resultado.

### 23.3 Consecuencias persistentes

Los sistemas de narrativa, exploración, encuentros y objetivos deben consultar el resultado de la decisión mediante identificadores estables. No deben inferirlo leyendo el texto del diálogo.

Ejemplos de efectos futuros posibles:
- un personaje recuerda a quién protegió ECHO-0;
- una comunidad cambia su nivel de confianza;
- un sector queda accesible, bloqueado o transformado;
- un testimonio o recuerdo posterior ofrece una perspectiva distinta;
- la presencia interior confronta al protagonista con la decisión y sus motivos expresados.

### 23.4 Sin alineamiento moral global

No implementar una variable única de «héroe/villano», «bueno/malo» o karma como sustituto de las decisiones. Si en el futuro se necesita una clasificación para un sistema concreto, debe derivarse de hechos específicos y nunca borrar el contexto de cada elección.

### 23.5 Pruebas de aceptación futuras

- Los recuerdos incompatibles sobre el vínculo pueden coexistir sin sobrescribirse.
- El jugador puede descubrirlos en distinto orden sin romper la continuidad.
- El diálogo no afirma una relación como verdad confirmada antes de que la historia lo justifique.
- Cada opción de la decisión individual/colectiva aplica consecuencias reproducibles.
- Una tercera alternativa solo aparece al cumplir condiciones documentadas.
- Las escenas posteriores reflejan la elección correcta sin consultar un marcador moral global.
- Los finales y rutas existentes siguen funcionando aunque el jugador no haya descubierto todos los recuerdos.

## 19. Registro de versiones

| Versión | Estado | Cambios |
|---|---|---|
| 0.1 | Propuesta técnica | Arquitectura lógica, estado de partida, encuentros, inventario, decisiones narrativas, finales y migración incremental |\n| 0.2 | Memoria relacional y decisiones persistentes | Registro de recuerdos con vínculos incompatibles y modelo de decisión individual/colectiva sin alineamiento moral global |

---

**Documento vivo:** esta arquitectura es una propuesta para revisión. No autoriza por sí sola cambios en el repositorio heredado ni declara implementados los componentes descritos.
