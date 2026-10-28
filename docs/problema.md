# Problema: Compliance as Code para controles de riesgo e IA

> **Caso ficticio para practicar.** TeleCorp, sus sistemas, personas y datos son inventados. Nada de este repositorio describe los sistemas de una empresa real.

---

## 1. Punto de partida

TeleCorp es una empresa de telecomunicaciones. Su área de Riesgos y Cumplimiento mantiene un catálogo de controles: unos los exige SOX (reporte financiero) y otros cubren riesgo operativo, de datos y de tecnología, sin ser SOX.

Hoy los controles funcionan así:

- Un analista extrae datos del ERP u otro sistema, los revisa en Excel, con un script o con un flujo low-code, y decide si hay una excepción.
- La regla vive en un documento, dentro del flujo o en la cabeza de quien ejecuta el control. Cuando cambia, nadie sabe con certeza desde qué fecha aplica.
- La evidencia queda en correos, capturas de pantalla y carpetas compartidas.

Al mismo tiempo, la empresa empieza a usar IA: modelos de riesgo que salen de notebooks, asistentes de IA generativa y agentes que construyen distintos equipos, cada uno con su proveedor y sin un control común.

La última auditoría dejó cinco hallazgos:

1. **Trazabilidad.** No se puede reconstruir qué regla se aplicó a una transacción de hace seis meses ni con qué datos.
2. **Inmutabilidad.** La evidencia se puede editar sin que se note.
3. **No repudio.** No se puede probar quién aprobó un cambio en una regla, ni que la regla aplicada fuera la aprobada.
4. **Dependencia de personas.** Si el analista no ejecuta el control (vacaciones, error, sobrecarga), el control no ocurre y nadie se entera.
5. **IA sin gobierno.** No hay inventario de modelos ni de agentes, ni evidencia de que se evaluaron antes de salir a producción.

---

## 2. Qué pide el área

La prioridad es explícita: **evolucionar el sistema de cumplimiento y los controles de riesgo, SOX y no SOX, a Compliance as Code.** Eso exige tres cosas:

- Un **motor declarativo de políticas**. El candidato a evaluar es [OPA](https://www.openpolicyagent.org/).
- Que el sistema cumpla por diseño los **pilares de auditoría**: trazabilidad, inmutabilidad, no repudio y los demás que espera un auditor (sección 4).
- Que los controles sean **más automáticos y dependan menos de procesos humanos**.

El resto del cargo (gateways de IA, MLOps, LLMOps, evaluaciones, datos de riesgo) se resuelve con **la misma plataforma**. Cada modelo, agente o pipeline es un dominio más de controles: lo decide el mismo motor y deja evidencia en el mismo lugar. Si la plataforma está bien hecha, sumar un dominio es escribir políticas y conectar un punto de aplicación, no construir otro sistema.

---

## 3. Idea de diseño: un motor, muchos puntos de aplicación

Dos palabras de vocabulario:

- **Punto de decisión:** evalúa la política y responde. Es OPA.
- **Punto de aplicación:** el sistema que pregunta y obedece. Son los servicios, pipelines y gateways.

```
                   Git (PR, revisión, firma)
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ PLANO DE CONTROL                                            │
│ Políticas Rego + datos de referencia → bundle firmado → OPA │
│ Inventario: controles, modelos, prompts, agentes,           │
│ herramientas MCP y proveedores                              │
└──────────────────────────────┬──────────────────────────────┘
                               │ cada punto pregunta, OPA decide
  ┌──────────┬──────────┬──────┴─────┬───────────┬───────────┐
  ▼          ▼          ▼            ▼           ▼           ▼
  Pagos      Accesos    CI/CD        Modelos     AI y Model  MCP
  (en línea) (lotes)    (políticas,  (promoción, Gateway     Gateway
                        infra,       drift)                  (agentes)
                        prompts)
  │          │          │            │           │           │
  └──────────┴──────────┴──────┬─────┴───────────┴───────────┘
                               ▼
     BITÁCORA DE EVIDENCIA: append-only, hash encadenado, consulta SQL
                               ▼
     OBSERVABILIDAD: métricas, trazas, alertas, tablero de cumplimiento
```

**Reglas de la plataforma:**

1. **La decisión la toma la política, no un `if` en el código del servicio.**
2. **Toda decisión va a la bitácora**, se permita o se niegue.
3. **Si el motor no responde, se bloquea** (*fail-closed*), salvo que el control diga otra cosa por escrito.
4. **La política no lee el reloj ni llama servicios externos.** La fecha y los datos de referencia entran en el input o en el bundle. Así una decisión vieja se puede volver a ejecutar y da el mismo resultado.
5. **Humanos y agentes usan el mismo mecanismo de identidad.** Cuando un agente actúa por un humano, el input lleva las dos identidades y el permiso efectivo es la intersección: un agente nunca puede más que la persona que lo invoca.
6. **Un control nuevo se escribe con una plantilla** (política, pruebas, datos de referencia, ficha del control) sin tocar el motor.

---

## 4. Pilares de auditoría como requisitos verificables

Cada pilar tiene una prueba automática que lo demuestra. Si no hay prueba, el pilar no está cumplido.

| Pilar | Qué significa | Cómo se demuestra |
|---|---|---|
| **Trazabilidad** | Reconstruir qué pasó, cuándo, con qué datos y con qué versión de la regla | Con un `decision_id` se obtiene el input, el resultado, las razones y la revisión del bundle |
| **Inmutabilidad** | La evidencia no se altera sin que se note | Cada registro lleva el hash del anterior; una prueba edita una línea y la verificación falla |
| **No repudio** | Nadie puede negar lo que hizo: ni quien aprobó la regla ni quien originó la transacción | Commits y bundles firmados; PR aprobado por alguien distinto del autor; identidad autenticada en el input, no autodeclarada |
| **Integridad de la regla** | Lo que se ejecuta es exactamente lo aprobado | OPA solo carga bundles con firma válida |
| **Completitud** | Se evaluó todo lo que había que evaluar | Conciliación: transacciones en el origen = decisiones en la bitácora |
| **Exactitud de los datos de entrada** | El control decidió sobre datos correctos | El lote se valida contra su contrato de datos antes de evaluar; si falla, va a cuarentena |
| **Reproducibilidad** | Una decisión histórica se puede recalcular | Re-ejecutar con el bundle de esa fecha da el mismo resultado |
| **Segregación de funciones** | Quien hace no aprueba | Reglas autor ≠ aprobador en pagos, modelos, prompts y políticas |
| **Mínimo privilegio** | Cada persona o agente puede solo lo que su rol permite | Políticas de acceso para usuarios, agentes, herramientas y la propia plataforma |
| **Confidencialidad** | La evidencia no expone datos personales | Enmascaramiento de PII en los registros de decisión |
| **Disponibilidad** | El control opera cuando se necesita | *Fail-closed*, monitoreo de salud del motor y runbook de caída |

---

## 5. Catálogo de controles

Prioridad: **P1** es lo que pidió el área; **P2** y **P3** extienden la misma plataforma al resto del cargo. La columna *Reto* remite al [caso de estudio](caso_estudio.md).

### Dominio A. Controles financieros (SOX) · P1

| ID | Control | Tipo | Punto de aplicación | Reto |
|---|---|---|---|---|
| FIN-01 | Un pago de más de 10 millones necesita al menos dos aprobadores distintos; quien lo crea no lo aprueba; solo aprueba el rol `aprobador_tesoreria`; un proveedor en lista restringida se bloquea siempre | Preventivo, por transacción | Servicio de pagos | R1–R3 |
| FIN-02 | Ningún usuario tiene a la vez permisos en conflicto (por ejemplo, crear proveedores y aprobar pagos), según una matriz de conflictos | Detectivo, periódico | Proceso por lotes sobre una foto de los accesos del ERP | Nuevo |
| FIN-03 | El cambio de cuenta bancaria de un proveedor lo verifica una persona distinta de quien lo registró | Preventivo | Servicio de datos maestros | Nuevo (lo escribe alguien que no diseñó la plataforma, con la plantilla) |

### Dominio B. Controles de riesgo no SOX · P1

| ID | Control | Tipo | Punto de aplicación | Reto |
|---|---|---|---|---|
| RSK-01 | Un dataset con datos personales solo lo consulta un rol autorizado y para una finalidad registrada | Preventivo | Capa de acceso a datos | Nuevo |
| RSK-02 | Una contraparte nueva se verifica contra listas restrictivas antes de habilitarse; la lista es un dato versionado | Preventivo | Alta de proveedores | Nuevo |

### Dominio C. Controles de TI sobre la plataforma (ITGC) · P1

Sin estos, los dominios A y B no tienen valor ante un auditor: un control automatizado solo es confiable si los cambios y los accesos a él están controlados.

| ID | Control | Tipo | Punto de aplicación | Reto |
|---|---|---|---|---|
| ITG-01 | Gestión de cambios: una política llega a producción solo por PR, con pruebas en verde, aprobada por alguien distinto del autor y empaquetada en un bundle firmado | Preventivo | CI/CD | R4c, R14 |
| ITG-02 | Accesos: solo el pipeline publica bundles; nadie edita la bitácora; los auditores la leen | Preventivo | Permisos del repo, del registro de bundles y de la bitácora | Nuevo |
| ITG-03 | Operación: si el motor cae, se bloquea, se alerta y se sigue el runbook | Detectivo | Monitoreo | R3, R13, R16c |

### Dominio D. Datos y modelos de riesgo (MLOps) · P2

| ID | Control | Tipo | Punto de aplicación | Reto |
|---|---|---|---|---|
| DAT-01 | Todo lote, sea insumo de un control o de un entrenamiento, se valida contra su contrato y deja registro de lineage y versión | Preventivo | Pipeline de datos | R5 |
| MOD-01 | Un modelo pasa a producción solo con métricas mínimas, model card, datos validados, sesgo bajo el umbral y un aprobador distinto del entrenador | Preventivo | Registro de modelos | R6, R7 |
| MOD-02 | Ante drift, la acción (revisar o revertir) la decide una política y queda registrada | Detectivo y correctivo | Monitoreo del modelo | R8 |
| MOD-03 | El puntaje de riesgo del modelo alimenta FIN-01: un puntaje alto exige una tercera aprobación | Preventivo | Servicio de pagos | R1 (extra) |

### Dominio E. IA generativa y agentes (LLMOps) · P2–P3

| ID | Control | Tipo | Punto de aplicación | Reto |
|---|---|---|---|---|
| IA-01 | Inventario: ningún modelo, prompt, agente o herramienta MCP opera sin estar registrado con dueño, propósito y nivel de riesgo | Preventivo | Plano de control (los gateways rechazan lo no registrado) | Nuevo |
| IA-02 | Acceso a modelos: cada rol usa solo sus modelos autorizados; la PII va solo a modelos y proveedores aprobados; hay límite de tokens y de presupuesto | Preventivo | AI Gateway y Model Gateway | R9 |
| IA-03 | Herramientas de agentes: mínimo privilegio por herramienta y por parámetro; un agente nunca aprueba pagos | Preventivo | MCP Gateway | R12 |
| IA-04 | Lanzamiento: una versión nueva de prompt, índice RAG o modelo sale solo si pasa la evaluación | Preventivo | CI/CD | R10, R11 |
| IA-05 | Evaluación continua: muestreo en producción de calidad, alucinación, fuga de PII, costo y latencia, con alertas | Detectivo | Observabilidad | R11, R13 |

### Dominio F. Infraestructura (DevSecOps) · P3

| ID | Control | Tipo | Punto de aplicación | Reto |
|---|---|---|---|---|
| INF-01 | Un plan de Terraform se aplica solo si cumple las reglas: cifrado, sin acceso público, etiqueta de dueño | Preventivo | CI/CD (OPA o conftest sobre el plan en JSON) | Nuevo |
| INF-02 | En Kubernetes solo corren imágenes firmadas del registro aprobado y sin privilegios | Preventivo | Control de admisión (OPA Gatekeeper) | Nuevo |

---

## 6. Soluciones de IA de riesgo para industrializar

La plataforma también gobierna las soluciones de IA del área. No son excepciones: son clientes del mismo motor.

1. **Modelo de puntaje de riesgo de pagos** (analítica avanzada). Da un puntaje de 0 a 1 a cada pago para priorizar revisiones. Su ciclo de vida lo gobiernan DAT-01 y MOD-01 a MOD-03, y su salida alimenta FIN-01.
2. **Asistente de controles** (IA generativa). Responde con RAG sobre el catálogo de controles y las políticas ("¿qué control aplica a un pago de 50 millones?"), cita la fuente y dice que no sabe cuando no hay soporte. Es solo de consulta.
   - **Extensión:** a partir del texto de un control escrito en prosa, propone un borrador de política en Rego con sus pruebas, **solo como PR**. Ese PR pasa por ITG-01 como cualquier cambio, y el agente nunca aprueba lo que propone. Así la IA acelera la migración del catálogo sin saltarse el control de cambios.

---

## 7. Evaluaciones automáticas

La oferta pide evaluar calidad, seguridad, sesgos, alucinaciones, drift, costos y latencia. Cada dimensión tiene una métrica y una compuerta que decide.

| Dimensión | Modelo de riesgo | Asistente y agentes | Compuerta |
|---|---|---|---|
| Calidad | AUC, recall de la clase fraude | Acierto de recuperación, relevancia, abstención correcta | MOD-01, IA-04 |
| Seguridad | Acceso a datos (RSK-01) | Prompt injection, abuso de herramientas, fuga de PII (debe ser 0) | IA-03, IA-04 |
| Sesgos | Diferencia de tasa de rechazo entre grupos | Respuestas distintas ante preguntas equivalentes | MOD-01, IA-04 |
| Alucinaciones | — | Fidelidad al contexto, con juez LLM calibrado contra etiquetas humanas | IA-04 |
| Drift | PSI o KS de entradas y de puntajes | Cambio en el tipo de preguntas y en la tasa de abstención | MOD-02, IA-05 |
| Costos | Cómputo | Tokens y dólares por solicitud y por rol | IA-02, IA-05 |
| Latencia | p95 de `/score` | p95 del gateway | MOD-01, IA-04 |

Dos reglas de rigor:

- **Un juez LLM sin calibrar no es evidencia.** Se mide su acuerdo con etiquetas humanas (por ejemplo, kappa de Cohen).
- **Una mejora que cabe dentro del ruido no es mejora.** Dos versiones se comparan con intervalos de confianza (bootstrap).

---

## 8. Tablero de cumplimiento

El tablero responde una pregunta: **¿los controles se cumplen y dependen cada vez menos de personas?**

**Indicadores:**

- **Porcentaje del catálogo automatizado** frente al manual. Es el indicador del objetivo del área.
- Decisiones por control: permitidas, negadas y por qué motivo.
- Completitud por control y periodo (origen frente a bitácora).
- Estado de la cadena de evidencia (resultado de la última verificación).
- Versión vigente de cada política, y cambios del mes con su autor y su aprobador.
- Modelo en producción, PSI máximo y resultado de la última evaluación del asistente.
- Costo y latencia de IA por rol y por proveedor.
- Tiempo para entregar evidencia ante una solicitud de auditoría.

**Alertas mínimas:** cadena de evidencia rota, completitud menor al 100 %, motor caído, PSI sobre el umbral, pico de bloqueos (por ejemplo, más del 20 % en 10 minutos) y presupuesto de IA excedido.

---

## 9. Evaluación técnica de proveedores y plataformas

"Revisar OPA" es en sí una evaluación. El problema incluye tres decisiones, cada una con criterios, una recomendación y los riesgos de esa elección.

1. **Motor de políticas.** OPA frente a Cedar, frente a las políticas nativas de la nube (que solo cubren infraestructura) y frente a dejar las reglas dentro de los flujos actuales. Criterios: expresividad, facilidad de prueba, rendimiento, firma y distribución de bundles, registros de decisión, curva de aprendizaje del equipo, madurez y sostenibilidad del proyecto, y dependencia del proveedor.
2. **AI Gateway y Model Gateway.** Construir uno propio, adoptar uno de código abierto (por ejemplo, LiteLLM) o usar un servicio gestionado de la nube. Criterios: integración con OPA, trazabilidad, soporte de varios proveedores (Azure OpenAI, OpenAI, Anthropic, Vertex AI, Bedrock), costo y esfuerzo de operación.
3. **Plataformas de ML, agentes y evaluación.** MLflow, Azure ML, SageMaker, Vertex AI, Kubeflow y Databricks para el ciclo de vida de modelos; LangChain, LangGraph, Semantic Kernel y Copilot Studio para agentes; DeepEval, Ragas, Promptfoo y Azure AI Evaluation frente a un evaluador propio.

---

## 10. Cobertura de la oferta

### Responsabilidades

| Responsabilidad | Dónde se resuelve |
|---|---|
| Plataforma de Compliance as Code, políticas y controles ejecutables | Motor y reglas de la plataforma (3), pilares (4), dominios A, B y C |
| Controles y evidencias automatizadas con CI/CD y DevSecOps | ITG-01, IA-04, INF-01, INF-02; resultados de pruebas y evaluaciones guardados como artefactos del pipeline |
| AI Control Plane, AI Gateway, Model Gateway y MCP Gateway | Plano de control e IA-01; IA-02 (AI y Model Gateway); IA-03 (MCP Gateway) |
| Identidad y mínimo privilegio para usuarios y agentes | Regla 5 de la plataforma, ITG-02, RSK-01, IA-02, IA-03 |
| Observabilidad y trazabilidad de modelos y agentes | Bitácora; trazas de OpenTelemetry por solicitud (identidad → OPA → modelo → registro) |
| Evaluaciones de calidad, seguridad, sesgos, alucinaciones, drift, costos y latencia | Sección 7 |
| Alertas, métricas y dashboards de cumplimiento | Sección 8 |
| Pipelines de datos de riesgo, data contracts, calidad, lineage y versionamiento | DAT-01; aplica a los datos de los controles y a los de entrenamiento |
| Industrializar modelos de analítica avanzada y GenAI de riesgo | Sección 6 |
| Documentar arquitecturas, controles, runbooks y evidencias | Entregables (11) |
| Evaluación técnica de proveedores y plataformas de IA | Sección 9 |

### Herramientas

| Grupo | Herramientas de la oferta | Papel en el problema | En el laboratorio (costo cero) |
|---|---|---|---|
| Lenguajes | Python, SQL | Servicios y pipelines; el auditor consulta la bitácora con SQL | Python y DuckDB o PostgreSQL |
| Políticas | OPA y Rego | Motor de decisión | OPA en Docker |
| MLOps | MLflow, Azure ML, SageMaker, Vertex AI, Kubeflow | Registro, entrenamiento y promoción de modelos | MLflow local; el resto en la evaluación de proveedores |
| Datos | Databricks, Spark, Delta Lake | Pipelines de datos de riesgo, versiones y lineage | PySpark local o la librería `deltalake` |
| Agentes y RAG | LangChain, LangGraph, Semantic Kernel, Copilot Studio, RAG, MCP | Construcción de agentes; los controles deben valer para cualquiera de ellos | LangGraph y un servidor MCP simulado |
| Evaluación | DeepEval, Ragas, Promptfoo, Azure AI Evaluation | Compuerta IA-04 y evaluación continua | Evaluador propio más uno de estos para comparar |
| Proveedores de modelos | Azure OpenAI, OpenAI, Anthropic, Vertex AI, Bedrock | Detrás del Model Gateway | Modelo simulado; uno real opcional, solo a través del gateway |
| Infraestructura | Docker, Kubernetes, Terraform | Despliegue y dominio F | Docker Compose, `kind` y planes de Terraform en JSON, sin nube |
| CI/CD | GitHub Actions, Azure DevOps | ITG-01, IA-04, INF-01 | GitHub Actions; Azure DevOps opcional |
| Observabilidad | OpenTelemetry, Prometheus, Grafana | Trazas, métricas, alertas y tablero | Docker Compose |

### Marcos

| Marco | Qué pide, en corto | Controles del problema |
|---|---|---|
| **SOX / ITGC** | Controles sobre el reporte financiero. Los ITGC cubren acceso a programas y datos, cambios a programas, desarrollo y operación | Dominio A; ITG-01 (cambios), ITG-02 (accesos), ITG-03 (operación) |
| **NIST AI RMF** | Cuatro funciones: Govern, Map, Measure, Manage | Govern: IA-01 y compuertas. Map: inventario, model cards, red team. Measure: sección 7. Manage: MOD-02, IA-05 y runbooks |
| **ISO/IEC 42001** | Sistema de gestión de IA: políticas, roles, evaluación de riesgos, ciclo de vida controlado, monitoreo y mejora | Dominios D y E, inventario y documentación |
| **Control de accesos y segregación de funciones** | Cada quien hace solo lo de su rol, y quien hace no aprueba | FIN-01 a FIN-03, ITG-02, IA-02, IA-03 |
| **Auditoría tecnológica** | Evidencia suficiente, verificable y reproducible | Pilares (4), bitácora y matriz de controles |

---

## 11. Entregables

- Políticas Rego con pruebas, una por control, siguiendo la plantilla.
- Bitácora de evidencia con verificación de la cadena.
- Pipeline de CI/CD que prueba, firma y publica bundles, y guarda la evidencia.
- **Corrida en paralelo** de FIN-01: el motor y el proceso manual sobre el mismo histórico sintético, con las diferencias explicadas. Es la prueba que justifica dejar de hacer el control a mano.
- `docs/matriz_controles.md`: control, política, prueba, evidencia y pilar.
- `docs/mapeo_marcos.md`: controles frente a SOX/ITGC, NIST AI RMF e ISO/IEC 42001.
- `docs/arquitectura.md`: plano de control, puntos de aplicación y flujo de evidencia.
- `docs/runbook.md`: caída del motor, política que bloquea operaciones legítimas, verificación de la cadena, drift fuerte, respuesta peligrosa del asistente, y quién aprueba cada tipo de cambio.
- `docs/evaluacion_proveedores.md`: las tres decisiones de la sección 9.
- Tablero de cumplimiento con sus alertas.

---

## 12. Fases

| Fase | Foco | Controles | Hito |
|---|---|---|---|
| **1. Núcleo** (lo que pidió el área) | Motor, bitácora, pilares, plantilla y decisión sobre el motor | FIN-01, FIN-02, RSK-01, ITG-01 a ITG-03, corrida en paralelo | Un auditor verifica FIN-01 de punta a punta sin hablar con el equipo |
| **2. Mismo motor, más dominios** | Probar que la plataforma escala | FIN-03 (con la plantilla), RSK-02, DAT-01, MOD-01 a MOD-03 | Un control nuevo entra sin tocar el motor |
| **3. IA generativa y agentes** | Gateways, inventario, evaluaciones y asistente | IA-01 a IA-05 | Un prompt que filtra PII no se lanza, y queda la evidencia |
| **4. Infraestructura y cierre** | DevSecOps, tablero completo y documentación | INF-01, INF-02 | Matriz de controles, mapeo a marcos y evaluación de proveedores completos |

La documentación y el tablero crecen en cada fase; no se dejan para el final.

---

## 13. Cuándo está resuelto

Un auditor, sin ayuda del equipo, puede:

1. Tomar un `decision_id` y ver el input, la versión exacta de la política, el resultado, las razones y quién aprobó esa versión.
2. Detectar que alguien editó una línea de la evidencia.
3. Comprobar que ninguna política, modelo o prompt llegó a producción sin una segunda aprobación y sin pasar su evaluación.
4. Verificar que todas las transacciones del origen tienen su decisión registrada.
5. Volver a ejecutar una decisión histórica y obtener el mismo resultado.
6. Revisar la corrida en paralelo y entender cada diferencia.
7. Ver en el tablero qué porcentaje del catálogo está automatizado y si algún control falló.

Y el equipo puede:

8. Agregar un control nuevo con la plantilla, sin tocar el motor.
9. Poner bajo control un modelo o un agente nuevo con solo registrarlo en el inventario.

---

## 14. Supuestos y preguntas abiertas

En el trabajo real estas preguntas no se inventan: se le hacen al área de riesgos y al dueño de cada proceso.

- Los pagos de 10 millones o menos: ¿qué regla aplica? El reglamento no lo dice.
- ¿Qué controles existen hoy? ¿Cuáles tienen más volumen y menos ambigüedad, para migrarlos primero?
- ¿Cómo corren hoy (a mano, con scripts, con flujos low-code) y quién los opera?
- ¿Qué evidencia pide el auditor externo y en qué formato?
- ¿El auditor prueba los controles automatizados con una sola muestra cuando los ITGC son efectivos? Si es así, ITG-01 es la pieza de mayor valor de la plataforma.
- ¿Quién es dueño de cada política: Riesgos, Tecnología o el dueño del proceso?
- En la corrida en paralelo, ¿cuánto tiempo y qué tasa de diferencias se aceptan para apagar el control manual?
- ¿Cuál es la nube principal y qué existe ya de gateways, registro de modelos y CI/CD?

---

## 15. Fuera de alcance

- Datos o sistemas reales de cualquier empresa.
- Recursos de nube con costo. Son opcionales y siempre con alerta de presupuesto.
- Certificación formal en algún marco. Aquí se practica cómo generar evidencia, no se certifica nada.
