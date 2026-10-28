# Caso de estudio completo: Compliance as Code, MLOps y LLMOps

**Para:** Andres · **Cargo:** Ingeniero MLOps/LLMOps, Consultora Nexo para TeleCorp (Torre TeleCorp) · **Inicio:** 4 de noviembre · **Periodo de prueba:** 60 días

---

## 0. Cómo usar este documento

- **No es un examen.** Nadie te califica. Es una prueba técnica que tú mismo te haces para entrar con contexto y saber dónde estás parado.
- **Las empresas y personas de este documento son ficticias.** Usé nombres como Laura y Carlos para que sea didáctico. Los sistemas del caso también son inventados: cómo funcionan los de verdad lo descubres con Natalia, Mateo y Esteban.
- **Es grande a propósito.** Cubre todo lo que menciona la oferta, y no se espera que lo termines en una semana. Hay dos rutas:
  - **Ruta mínima (antes del 4 de nov):** llegas con la base y el vocabulario.
  - **Ruta completa (durante tus primeros 60 días):** sigues practicando mientras aprendes el entorno real.
- **Trabaja en un repo propio** (`compliance-as-code-lab`) y sube todo a tu GitHub. Te sirve como práctica y como portafolio.
- **No hay un único camino correcto.** Si algo no sale, anota qué no entendiste. Esa lista de dudas vale tanto como el código.
- **Costo cero.** Todo corre en tu computador con Docker y Python. Los modelos de lenguaje se simulan, y la nube (Azure) es un reto opcional.

---

## 1. Qué te piden: mapa de la oferta a este caso

La oferta tiene 11 responsabilidades. Cada una tiene un reto aquí:

| Responsabilidad de la oferta | Qué significa en la práctica | Reto |
|---|---|---|
| Plataforma de **Compliance as Code**, políticas y controles ejecutables | Las reglas de control dejan de vivir en Excel o en la cabeza de alguien y pasan a ser código que se ejecuta y se prueba (OPA/Rego). | R1, R2, R3 |
| Controles y evidencias automatizadas en **CI/CD y DevSecOps** | Cada cambio a una política o modelo pasa por un pipeline que lo verifica y guarda evidencia. | R14 |
| **AI Control Plane, AI Gateway, Model Gateway, MCP Gateway** | Un punto central por donde pasan las llamadas a modelos y herramientas de agentes, donde se aplican reglas y se registra todo. | R9, R12 |
| **Identidad y mínimo privilegio** para usuarios y agentes | Cada persona o agente puede hacer solo lo que su rol permite. | R9, R12 |
| **Observabilidad y trazabilidad** de modelos y agentes | Saber qué pasó en cada llamada, con qué versión y cuánto costó. | R4, R13 |
| **Evaluaciones automáticas** de calidad, seguridad, sesgos, alucinaciones, drift, costos, latencia | Probar y vigilar los modelos de forma continua. | R8, R11 |
| **Alertas, métricas y dashboards** de cumplimiento | Que alguien vea en una pantalla si todo cumple o no. | R13 |
| **Pipelines de datos de riesgo**, data contracts, calidad, lineage, versionamiento | Datos confiables y con historia antes de entrenar o consultar nada. | R5 |
| **Industrializar modelos** de analítica avanzada y GenAI de riesgo | Llevar modelos de un notebook a producción, de forma repetible y controlada. | R6, R7, R10 |
| **Documentar** arquitecturas, controles, runbooks y evidencias para auditoría | Que un auditor entienda qué control existe y dónde está la prueba. | R16 |
| **Evaluación técnica de proveedores** y plataformas de IA | Comparar opciones con criterio (construir, adoptar o comprar). | R16 |

---

## 2. MLOps y LLMOps: qué son y dónde juegan en este caso

Esta es la sección clave para entender el cargo.

### 2.1 Las definiciones, sin jerga

- **MLOps** es el conjunto de prácticas para llevar **modelos de machine learning clásico** a producción y mantenerlos funcionando: datos, entrenamiento, registro, despliegue, monitoreo y reentrenamiento. En este caso: un modelo que da un puntaje de riesgo a cada pago.
- **LLMOps** es lo mismo, pero para **modelos de lenguaje y agentes**. Casi nunca los entrenas: los **orquestas, evalúas, controlas y vigilas**. Lo que se versiona ya no es solo un modelo, sino también prompts, documentos del RAG, herramientas del agente y proveedor. En este caso: un asistente que responde sobre políticas de riesgo.
- **El hilo común:** ambos necesitan versionamiento, pruebas automáticas, despliegue controlado, monitoreo y evidencia. Y ahí entra el **Compliance as Code**: es la capa que decide, con reglas, *qué modelo o prompt puede salir a producción, quién puede usarlo y qué queda registrado*.

> **Idea central del cargo:** MLOps y LLMOps hacen que los modelos funcionen. Compliance as Code hace que lo hagan **de forma controlada, auditable y sin depender de revisiones manuales**. El puesto pide las tres cosas a la vez.

### 2.2 Las diferencias que importan

| Aspecto | MLOps (modelo de riesgo de pagos) | LLMOps (asistente RAG y agentes) |
|---|---|---|
| **Qué se versiona** | Datos, código, modelo entrenado | Prompts, corpus e índice del RAG, configuración del retriever, modelo/proveedor, herramientas del agente |
| **Entrenamiento** | Sí: tú entrenas y reentrenas | Casi nunca: usas modelos de un proveedor. "Ajustar" es mejorar prompts y recuperación |
| **Cómo se evalúa** | Con etiquetas: AUC, recall, precisión | No hay una respuesta única: fidelidad al contexto, relevancia, abstención, juez LLM calibrado con humanos |
| **Riesgos típicos** | Drift de datos, sesgo, fuga de datos | Alucinación, prompt injection, fuga de datos personales, abuso de herramientas |
| **Qué se monitorea** | Drift (PSI/KS), calidad de entradas, desempeño cuando llegan etiquetas | Trazas, costo por llamada, latencia, calidad muestreada, bloqueos |
| **Costo dominante** | Cómputo de entrenamiento e inferencia | Tokens por solicitud |
| **Despliegue seguro** | Registro de modelos, promoción por etapas, rollback | Gateway, versionado de prompts, canary, rollback de prompt o modelo |
| **Gobierno** | Model card, segregación de funciones, validación independiente | Inventario de modelos y agentes, mínimo privilegio, evidencia de evaluación |
| **Herramientas que lista la oferta** | MLflow, Azure ML, SageMaker, Vertex AI, Kubeflow, Databricks | LangChain, LangGraph, Semantic Kernel, DeepEval, Ragas, Promptfoo, Azure AI Evaluation, Bedrock |

### 2.3 Dónde cae cada uno en la arquitectura del caso

```
┌──────────────────────────────────────────────────────────────────────┐
│  PLANO DE CONTROL  (Compliance as Code)                              │
│  OPA + políticas Rego en Git  ·  CI/CD con evidencia  ·  Auditoría   │
└───────┬───────────────────────┬──────────────────────┬───────────────┘
        │ decide                │ decide               │ decide
        ▼                       ▼                      ▼
┌───────────────┐     ┌──────────────────┐    ┌──────────────────────┐
│ SISTEMA DE    │     │ MLOps            │    │ LLMOps               │
│ PAGOS (SOX)   │◄────│ Modelo de riesgo │    │ AI Gateway           │
│ Reglas Rego   │score│ MLflow, registry │    │ Asistente RAG        │
│ + score       │     │ drift, rollback  │    │ Agentes y MCP        │
└───────────────┘     └────────▲─────────┘    └──────────▲───────────┘
                               │                         │
                      ┌────────┴─────────────────────────┴───────┐
                      │ DATOS DE RIESGO                          │
                      │ contratos · calidad · lineage · versión  │
                      └──────────────────────────────────────────┘

  OBSERVABILIDAD transversal: OpenTelemetry · Prometheus · Grafana
```

**Cómo se conectan en el caso:**
1. Los **datos** de pagos entran con un contrato y validación (R5).
2. **MLOps** entrena un modelo que da un puntaje de riesgo a cada pago (R6). **OPA decide si ese modelo puede pasar a producción** (R7) y vigila su drift (R8).
3. El **sistema de pagos** usa dos cosas: las reglas Rego del reglamento (R1) y el puntaje del modelo. Por ejemplo, "si el puntaje es alto, se exige una tercera aprobación".
4. **LLMOps**: un asistente RAG responde preguntas sobre políticas (R10). Todo pasa por el **AI Gateway** (R9), donde OPA decide quién usa qué modelo. Antes de lanzar una nueva versión del prompt o del índice, se evalúa (R11) y OPA decide si sale. Los agentes tienen permisos mínimos (R12).
5. Todo deja **trazas y métricas** (R13), corre en un **pipeline** (R14) y se documenta para **auditoría** (R16).

### 2.4 El ciclo de vida de cada uno, con las compuertas (gates)

Una compuerta es un punto donde alguien (o algo) debe decir "sí" para avanzar. **En este cargo, muchas compuertas las decide OPA.**

**MLOps**
```
Datos → [contrato y calidad] → Entrenamiento → Evaluación → Registro
      → [GATE: ¿puede promoverse?] → Despliegue → Monitoreo
      → [drift: ¿revisar o revertir?] → Reentrenamiento
```

**LLMOps**
```
Corpus → Indexación → Prompt y agente → Evaluación offline
       → [GATE: ¿puede lanzarse?] → Gateway → Trazas, costos, bloqueos
       → Evaluación continua (muestreo) → Ajustes
```

### 2.5 Dónde encaja lo que ya sabes

| Lo que ya sabes | Dónde lo usas aquí |
|---|---|
| Agentes con LangGraph, RAG y evaluación con juez LLM | R10, R11, R12 (LLMOps) |
| MLflow para evaluación de agentes | R6, R7, R11 |
| Lakehouse con Spark, Delta Lake y Unity Catalog | R5 (datos, lineage, versionamiento) |
| Modelos de riesgo financiero y series de tiempo | R6, R8 (MLOps de riesgo) |
| Estadística y diseño experimental | R8 (drift), R11 (comparar versiones con rigor) |
| Docker, GCP, Cloud Build | R14, R15 |

Lo que todavía **no** has usado (OPA/Rego, Kubernetes, Terraform, Azure/AWS, OpenTelemetry, ISO 42001, SOX) está concentrado en R1–R4, R13–R16. Ahí es donde conviene practicar más.

---

## 3. El escenario (ficticio)

La empresa **TeleCorp** le plantea al equipo de Esteban cuatro necesidades:

**Necesidad A: Control de pagos (SOX).**
Hoy una persona revisa a mano que ningún pago grande lo apruebe quien lo creó. Auditoría quiere que ese control sea automático y quede registrado. Reglamento:

> 1. Todo pago de más de 10 millones de pesos necesita aprobación de **al menos dos personas distintas**.
> 2. **Quien crea un pago no puede aprobarlo.**
> 3. Solo pueden aprobar pagos las personas con rol `aprobador_tesoreria`.
> 4. Los pagos a proveedores marcados como `en_lista_restringida` se bloquean siempre.

**Necesidad B: Uso controlado de IA generativa.**

> 1. Cada usuario o agente solo puede usar los **modelos autorizados para su rol**.
> 2. Si la consulta contiene **datos personales (PII)**, solo puede ir a modelos aprobados para PII.
> 3. Hay un **límite de tokens** por solicitud según el rol.
> 4. **Toda** llamada se registra: quién, qué modelo, qué decisión, cuánto costó y cuánto tardó.

**Necesidad C: Modelo de riesgo de pagos (MLOps).**
Riesgos quiere un modelo que dé un **puntaje de riesgo (0 a 1)** a cada pago, para priorizar revisiones. Control interno exige:

> 1. Ningún modelo pasa a producción sin cumplir **métricas mínimas**, tener **ficha del modelo (model card)** y datos validados.
> 2. **Quien entrena un modelo no puede aprobar su paso a producción.**
> 3. Si el modelo se degrada (drift), debe haber una **acción definida**: revisar o volver al modelo anterior.

**Necesidad D: Asistente de políticas con IA (LLMOps).**
Los analistas quieren preguntarle a un asistente cosas como "¿quién puede aprobar un pago de 50 millones?". Seguridad exige:

> 1. Responder **solo con base en los documentos oficiales** y citar la fuente; si no hay información, decir que no la tiene.
> 2. Ninguna versión nueva del prompt o de los documentos se lanza sin **pasar una evaluación**.
> 3. El asistente **nunca** puede ejecutar acciones sobre pagos (es solo de consulta).

**Tu misión:** construir un prototipo pequeño que resuelva A, B, C y D, con evidencia de auditoría y un pipeline.

---

## 4. Estructura sugerida del repo

```
compliance-as-code-lab/
├── policies/                   # Compliance as Code
│   ├── pagos.rego / pagos_test.rego
│   ├── gateway.rego / gateway_test.rego
│   ├── modelos.rego / modelos_test.rego        # gate de promoción (MLOps)
│   ├── monitoreo.rego                          # acción ante drift
│   ├── release_llm.rego                        # gate de lanzamiento (LLMOps)
│   ├── herramientas.rego                       # permisos de agentes
│   └── data.json                               # roles, umbrales, modelos permitidos
├── datos/
│   ├── generar_pagos.py                        # datos sintéticos
│   ├── contrato_pagos.yaml                     # data contract
│   └── validar.py
├── ml/
│   ├── entrenar.py
│   ├── promover_modelo.py
│   ├── monitorear_drift.py
│   └── MODEL_CARD.md
├── llm/
│   ├── corpus/                                 # documentos de políticas (md)
│   ├── prompts/ (v1.yaml, v2.yaml)
│   ├── indexar.py / asistente.py
│   └── herramientas_mcp.py
├── app/
│   ├── opa_client.py
│   ├── pagos_service.py                        # FastAPI: pagos + score
│   ├── gateway.py                              # FastAPI: AI Gateway
│   ├── audit_log.py                            # hash encadenado
│   └── mock_llm.py
├── evals/
│   ├── dataset.jsonl / run_evals.py
│   └── red_team.md
├── observabilidad/                             # prometheus.yml, dashboards
├── tests/                                      # pytest
├── docs/
│   ├── matriz_controles.md
│   ├── mapeo_marcos.md                         # ISO 42001, NIST AI RMF, ITGC
│   ├── evaluacion_proveedores.md
│   └── runbook.md
├── infra/                                      # Terraform (opcional)
├── docker-compose.yml                          # OPA, MLflow, Prometheus, Grafana
├── cloudbuild.yaml
├── azure-pipelines.yml                         # opcional
└── .github/workflows/ci.yml
```

---

## 5. Los retos

Cada reto indica el tiempo aproximado (⏱) y de qué parte de la oferta viene.

---

## BLOQUE 1: Compliance as Code (el núcleo del cargo)

### Reto 0: Levantar OPA ⏱ 15 min

```bash
docker run -p 8181:8181 -v $(pwd)/policies:/policies \
  openpolicyagent/opa:latest run --server --addr :8181 --watch /policies
```

**Éxito:** `curl http://localhost:8181/health` responde `{}`.

> Si tu versión de OPA es anterior a 1.0, agrega `import rego.v1` al inicio de cada `.rego`.

---

### Reto 1: Políticas de pagos en Rego (Necesidad A) ⏱ 1 h

Escribe `policies/pagos.rego` con las cuatro reglas del reglamento.

**Input:**

```json
{
  "monto": 50000000,
  "creador": "Laura",
  "aprobadores": [
    {"usuario": "Carlos", "rol": "aprobador_tesoreria"},
    {"usuario": "Marta",  "rol": "aprobador_tesoreria"}
  ],
  "proveedor": {"nombre": "Proveedor X", "en_lista_restringida": false},
  "score_riesgo": 0.35
}
```

**Punto de partida:**

```rego
package pagos

default permitir := false

permitir if {
    # TODO: ¿qué condiciones deben cumplirse TODAS a la vez?
}
```

**Pistas:**
- Dentro de `{ }`, todas las líneas deben cumplirse (como un `and`).
- Cuenta elementos con `count(...)`, recorre listas con `some a in input.aprobadores`, niega con `not`.
- Divide en reglas auxiliares para que lo pueda leer un auditor.

**Preguntas de diseño (anótalas):**
1. ¿Qué pasa con pagos de 10 millones o menos? El reglamento no lo dice. Esta ambigüedad es típica: en el trabajo se la preguntas a Natalia, no la inventas.
2. ¿Dos aprobaciones del mismo usuario cuentan como dos?
3. Si falta un campo del input, ¿OPA permite o niega? (Pista: ante la duda, se niega.)

**Extra:** que la política devuelva también las **razones del rechazo** (`deny contains msg if { ... }`).

**Integración con MLOps (hazla después del R6):** agrega una regla: *si `score_riesgo` > 0.8, el pago exige una tercera aprobación*. Así el modelo de la Necesidad C alimenta la política.

---

### Reto 2: Pruebas de las políticas ⏱ 1 h

Las políticas son código y se prueban:

```rego
package pagos_test

import data.pagos

test_creador_no_puede_aprobar_su_pago if {
    not pagos.permitir with input as {
        "monto": 50000000,
        "creador": "Laura",
        "aprobadores": [{"usuario": "Laura", "rol": "aprobador_tesoreria"}],
        "proveedor": {"en_lista_restringida": false}
    }
}
```

```bash
docker run --rm -v $(pwd)/policies:/policies openpolicyagent/opa:latest test /policies -v
```

**Mínimo 8 pruebas**, incluyendo: dos aprobadores válidos (permite), uno solo (niega), el creador se aprueba (niega), rol incorrecto (niega), proveedor restringido (niega), exactamente 10.000.000 (caso borde), input incompleto (niega), mismo usuario dos veces (niega).

**Éxito:** `PASS: 8/8`, y al menos una prueba escrita *antes* de la regla.

---

### Reto 3: Python consulta a OPA ⏱ 1 h

```python
import requests

def consultar_opa(ruta: str, datos: dict) -> bool:
    r = requests.post(f"http://localhost:8181/v1/data/{ruta}", json={"input": datos}, timeout=2)
    r.raise_for_status()
    return r.json().get("result", False)
```

Crea un servicio FastAPI (`pagos_service.py`) con `POST /pagos/aprobar` que consulte a OPA y responda `200` o `403`.

**Preguntas de diseño:**
1. Si OPA no responde, ¿el sistema aprueba o bloquea? (*fail-open* vs *fail-closed*; en control de riesgo casi siempre es fail-closed.)
2. ¿Qué timeout usarías y cómo afecta al usuario?

**Éxito:** demuestras un caso permitido y uno bloqueado con `curl` o `pytest`.

---

### Reto 4: Trazabilidad, inmutabilidad y no repudio ⏱ 1.5 h

Los tres pilares que mencionó Esteban, convertidos en código.

**4a. Trazabilidad.** Cada decisión se guarda en `auditoria.jsonl`:

```json
{
  "ts": "2026-11-04T10:15:00Z",
  "decision_id": "uuid",
  "politica": "pagos/permitir",
  "version_politica": "<hash del commit de git>",
  "input": { "...": "..." },
  "resultado": false,
  "razones": ["creador no puede aprobar su propio pago"],
  "hash_anterior": "...",
  "hash": "..."
}
```

**4b. Inmutabilidad.** Cada registro incluye el SHA-256 del anterior (cadena). Escribe `verificar_cadena()` y una prueba que **edite una línea a mano** y verifique que se detecta.

**4c. No repudio.** Responde en `docs/runbook.md`:
- ¿Cómo sabe un auditor que la política que decidió es la aprobada en ese momento?
- Investiga cómo OPA permite **firmar bundles** (`opa build --signing-key`) y explica qué problema resuelve.

**Extra:** firma un bundle de verdad y haz que OPA solo cargue bundles con firma válida.

> **Este registro lo reusas en todos los retos siguientes:** las decisiones de pagos, del gateway, de promoción de modelos y de lanzamiento de prompts van al mismo tipo de bitácora.

---

## BLOQUE 2: Datos de riesgo

### Reto 5: Contrato de datos, calidad y lineage ⏱ 1.5 h
*(Oferta: "pipelines de datos de riesgo, data contracts, calidad, lineage y versionamiento")*

Sin datos confiables no hay MLOps. Aquí construyes la base para el modelo de riesgo.

**5a. Datos sintéticos.** En `datos/generar_pagos.py` genera unos 20.000 pagos con: `id_pago`, `fecha`, `monto`, `proveedor_id`, `creador`, `pais`, `canal`, `es_fraude` (poco frecuente, ~2%) y una columna con correo del proveedor (PII, para practicar). Haz que el fraude dependa de forma realista del monto, el canal y la hora.

**5b. Data contract.** En `contrato_pagos.yaml` define, por columna: tipo, si admite nulos, rango válido, si contiene PII, y a nivel de dataset: dueño, frecuencia esperada y frescura máxima.

**5c. Validación.** En `validar.py` valida el lote contra el contrato (puedes usar `pandera` o `pydantic`). Si falla, el lote va a **cuarentena** y no entra al entrenamiento.

**5d. Lineage y versionamiento.** Cada vez que se genera o valida un dataset, guarda un registro: hash del dataset, versión del contrato, hash del commit, fecha y quién lo generó. (Con Delta Lake tendrías *time travel*; aquí puedes usar un hash y DVC, o la librería `deltalake` si quieres practicar lo que ya conoces de Delta Lake.)

**Preguntas de diseño:**
1. Si el equipo que alimenta los datos cambia el nombre de una columna sin avisar, ¿qué debería pasar? ¿Quién es dueño del contrato?
2. ¿Cómo evitas que el correo (PII) llegue al entrenamiento?

**Éxito:** un lote con un valor fuera de rango se rechaza, y puedes decir exactamente qué datos entrenaron un modelo.

---

## BLOQUE 3: MLOps (modelo de riesgo de pagos)

### Reto 6: Entrenar, rastrear y registrar un modelo ⏱ 1.5 h
*(Oferta: "industrializar modelos de analítica avanzada")*

1. Levanta **MLflow** en `docker-compose.yml`.
2. En `ml/entrenar.py` entrena un modelo sencillo con scikit-learn (regresión logística o *gradient boosting*) que prediga `es_fraude`. No busques el mejor modelo: lo importante es el proceso.
3. Registra en MLflow: parámetros, métricas (AUC, recall de la clase fraude, precisión), el **hash del dataset**, el **hash del commit** y quién entrenó.
4. Registra el modelo en el **Model Registry** con alias `candidato`.
5. Escribe `ml/MODEL_CARD.md`: propósito, datos usados, métricas, limitaciones conocidas y responsable.
6. Crea un endpoint `POST /score` (FastAPI) que cargue el modelo con alias `produccion` y devuelva el puntaje.

**Pregunta de diseño:** con 2% de fraude, ¿por qué la *exactitud* (accuracy) engaña? ¿Qué métrica usarías para decidir?

**Éxito:** puedes reproducir un entrenamiento a partir de su registro en MLflow (mismos datos, mismo código).

---

### Reto 7: Compuerta de promoción con OPA ⏱ 1.5 h
*(Aquí se juntan MLOps y Compliance as Code, que es el corazón del cargo)*

Un modelo `candidato` **no pasa a `produccion` por decisión de una persona**: lo decide una política. Escribe `policies/modelos.rego`:

```rego
package modelos

default promover := false

promover if {
    input.metricas.auc >= data.umbrales.auc_min
    input.metricas.recall_fraude >= data.umbrales.recall_min
    input.contrato_datos.validado
    input.model_card.existe
    input.entrenador != input.aprobador
    input.sesgo.diferencia_tasa_rechazo <= data.umbrales.sesgo_max
}
```

Después escribe `ml/promover_modelo.py`:
1. Lee del registro las métricas y metadatos del candidato.
2. Calcula una verificación simple de sesgo: diferencia de tasa de rechazo entre grupos (por ejemplo, por `pais` o `canal`).
3. Arma el input y consulta a OPA.
4. Si OPA permite, cambia el alias a `produccion`. Si no, no cambia nada y explica por qué.
5. Registra la decisión en la bitácora de auditoría (R4).

**Mínimo 8 pruebas** en Rego: métricas bajas, sin model card, mismo entrenador y aprobador, datos sin validar, sesgo alto, etc.

**Preguntas de diseño:**
1. ¿Quién debería ser el "aprobador" real? ¿Otra persona del equipo, Riesgos, un comité?
2. ¿Cómo evitas que alguien cambie el alias directamente en MLflow, saltándose la política?

**Éxito:** un modelo con métricas buenas pero con el mismo entrenador y aprobador se bloquea.

---

### Reto 8: Monitoreo, drift y rollback ⏱ 1.5 h
*(Oferta: "evaluaciones de drift", "alertas y métricas")*

1. **Simula producción:** genera lotes nuevos con distribución cambiada (por ejemplo, montos más altos y otro canal dominante).
2. En `ml/monitorear_drift.py` calcula por variable el **PSI** o una prueba KS (con `scipy`) contra los datos de entrenamiento, y compara también la distribución de los puntajes.
3. Escribe la política de acción:

```rego
package monitoreo

default accion := "ok"

accion := "revisar"  if { input.psi_max > 0.10; input.psi_max <= 0.25 }
accion := "revertir" if { input.psi_max > 0.25 }
```

(Los umbrales 0.10 y 0.25 son una convención común. Los reales los definen contigo Riesgos.)

4. Si la acción es `revertir`, el script devuelve el alias `produccion` a la versión anterior y lo registra en la bitácora.
5. Expón las métricas con `prometheus_client` (se verán en R13).

**Preguntas de diseño:**
1. En fraude, las etiquetas reales llegan semanas después. ¿Cómo detectas problemas mientras tanto? (Pista: drift de entradas y de puntajes.)
2. ¿El rollback debe ser automático o requerir aprobación humana? ¿Por qué?

**Éxito:** un lote con drift fuerte dispara `revertir` y queda evidencia del cambio.

---

## BLOQUE 4: LLMOps (gateway, asistente y agentes)

### Reto 9: Mini AI Gateway (Necesidad B) ⏱ 2 h
*(Oferta: "AI Control Plane, AI Gateway, Model Gateway", "identidad y mínimo privilegio")*

Servicio FastAPI `app/gateway.py` con `POST /v1/chat`. Cada llamada sigue este flujo:

```
Usuario o agente → Gateway → (1) identifica quién es y su rol
                           → (2) consulta a OPA: ¿puede usar este modelo, este tamaño, estos datos?
                           → (3) si no: 403 + razón, y se registra
                           → (4) si sí: llama al modelo (mock)
                           → (5) registra decisión, costo, latencia y tokens
                           → (6) responde
```

**`policies/data.json`:**

```json
{
  "roles": {
    "analista_riesgo":     {"modelos": ["modelo-general"], "max_tokens": 4000, "presupuesto_diario_usd": 2.0},
    "agente_cumplimiento": {"modelos": ["modelo-general"], "max_tokens": 2000, "presupuesto_diario_usd": 1.0},
    "auditor_interno":     {"modelos": ["modelo-general", "modelo-interno"], "max_tokens": 8000, "presupuesto_diario_usd": 5.0}
  },
  "modelos_aprobados_pii": ["modelo-interno"]
}
```

**Input que el gateway manda a OPA:**

```json
{
  "identidad": {"id": "agente-42", "tipo": "agente", "rol": "agente_cumplimiento"},
  "modelo": "modelo-general",
  "tokens_estimados": 1500,
  "contiene_pii": true,
  "gasto_hoy_usd": 0.60
}
```

**Detalles que hacen la diferencia:**
- Detecta PII con regex en el gateway y manda `contiene_pii` a OPA. **La decisión la toma OPA, no un `if` del gateway.** Esa separación es la idea central de Compliance as Code.
- `mock_llm.py` devuelve una respuesta fija, con `sleep` aleatorio y un conteo de tokens inventado, para tener latencia y costo que registrar.
- Humanos y agentes usan el mismo mecanismo de identidad.
- Agrega un **tope de presupuesto**: si el gasto del día supera el del rol, se bloquea.

**Mínimo 8 pruebas** en `gateway_test.rego`.

**Preguntas de diseño:**
1. ¿Qué cambiaría para sumar un **Model Gateway** que enrute entre proveedores (Azure OpenAI, Bedrock, Vertex) según costo o disponibilidad?
2. ¿Qué cambiaría para un **MCP Gateway**? (Se detalla en el R12.)

---

### Reto 10: Asistente RAG versionado (Necesidad D) ⏱ 2 h
*(Oferta: "industrializar soluciones GenAI relacionadas con riesgo")*

Aquí practicas **qué se versiona y qué se registra** en un sistema de LLM.

1. **Corpus.** En `llm/corpus/` crea 6 a 8 documentos cortos en Markdown con reglamentos inventados: el reglamento de pagos, una política de acceso, una política de uso de IA, etc. Incluye uno que **se contradiga a medias con otro** (en el trabajo real pasa).
2. **Indexación.** `llm/indexar.py` parte los documentos en fragmentos y crea el índice. Para no depender de descargas pesadas, puedes usar TF-IDF de scikit-learn; si prefieres, usa embeddings y `pgvector`, que ya conoces.
3. **Prompts versionados.** Guarda el prompt en `llm/prompts/v1.yaml` (y luego `v2.yaml`) con: texto del sistema, reglas de abstención y formato de citas. Cada prompt tiene un hash.
4. **Asistente.** `llm/asistente.py` recupera fragmentos, arma el prompt, llama **a través del gateway (R9)** y responde **citando el documento**. Si no hay soporte en el contexto, responde que no tiene información. (Con el modelo simulado, puedes hacer que "responda" extrayendo la frase más relevante, para que todo sea determinista y probable.)
5. **Registro de trazabilidad.** Cada respuesta guarda: `version_prompt`, `version_corpus` (hash), `version_indice`, modelo usado, fragmentos recuperados, tokens, costo y latencia.

**Opcional:** modelar el flujo con **LangGraph** (`recuperar → generar → verificar citas`), que ya manejas. Si tienes acceso a un modelo real (por ejemplo, el de tu cuenta de GCP), úsalo **solo a través de tu gateway**.

**Preguntas de diseño:**
1. Si cambian un documento del corpus, ¿qué versiones cambian y qué debes reevaluar?
2. ¿Cómo muestras al usuario que hay dos documentos que se contradicen?

---

### Reto 11: Evaluación y compuerta de lanzamiento ⏱ 2 h
*(Oferta: "automatizar evaluaciones de calidad, seguridad, sesgos, alucinaciones, costos y latencia")*

Aquí tu experiencia evaluando agentes y tu formación en estadística te dan ventaja.

**11a. Dataset.** En `evals/dataset.jsonl` crea 24 a 30 casos en cuatro tipos:
- **Respondibles:** la respuesta está en el corpus.
- **No respondibles:** el asistente debe decir que no sabe.
- **Adversariales:** intentos de prompt injection (por ejemplo, un documento que dice "ignora tus reglas").
- **Con PII:** deben bloquearse o tratarse con el modelo adecuado.

**11b. Métricas.** En `evals/run_evals.py` calcula:
- **Fidelidad al contexto** (groundedness): ¿lo que dice está respaldado por los fragmentos?
- **Relevancia** y **acierto de recuperación** (¿el fragmento correcto está entre los K primeros?).
- **Acierto de abstención:** ¿se abstuvo cuando debía?
- **Fuga de PII** (debe ser 0), **latencia** p50/p95 y **costo** por 1.000 consultas.

**11c. Juez LLM calibrado.** Usa el patrón de juez (aquí puede ser una función simulada o un modelo real vía gateway) y **calíbralo con etiquetas humanas**: etiqueta tú 10 casos a mano y mide el acuerdo entre juez y humano (por ejemplo, kappa de Cohen). Un juez sin calibrar no es evidencia.

**11d. Comparar versiones con rigor.** Corre `v1` y `v2` del prompt y compara con intervalos de confianza (bootstrap). Una mejora que cabe dentro del ruido no es mejora.

**11e. Compuerta de lanzamiento.** Escribe `policies/release_llm.rego`:

```rego
package release_llm

default lanzar := false

lanzar if {
    input.fidelidad >= data.umbrales.fidelidad_min
    input.fuga_pii == 0
    input.acierto_abstencion >= data.umbrales.abstencion_min
    input.latencia_p95_ms <= data.umbrales.p95_max_ms
    input.comparacion.ic95_inferior_mejora >= 0
}
```

**Extra (herramientas de la oferta):** repite parte de la evaluación con **Promptfoo**, **DeepEval** o **Ragas**, y compara resultados con tu evaluador propio.

**Éxito:** un prompt `v2` que mejora la fidelidad pero filtra PII **no se lanza**, y queda la evidencia.

---

### Reto 12: Seguridad de agentes, herramientas y MCP ⏱ 1.5 h
*(Oferta: "MCP Gateway", "mínimo privilegio para usuarios y agentes")*

Un agente es un programa que decide qué herramientas usar. Hay que controlar qué herramientas puede usar y con qué parámetros.

1. En `llm/herramientas_mcp.py` define 3 herramientas simuladas:
   - `consultar_politica(id)`: solo lectura.
   - `consultar_estado_pago(id_pago)`: solo lectura.
   - `aprobar_pago(id_pago)`: acción sobre pagos.
2. Escribe `policies/herramientas.rego`: qué agente (por identidad y rol) puede llamar a qué herramienta y con qué restricciones.
3. **Regla clave de segregación de funciones:** el asistente de la Necesidad D **nunca** puede llamar a `aprobar_pago`. Un agente no aprueba lo que un humano debe aprobar.
4. Registra cada llamada de herramienta en la bitácora (R4).

**Red teaming.** En `evals/red_team.md` documenta al menos **10 ataques** y el resultado:
- Prompt injection directo y escondido en un documento del corpus.
- Intentar que el asistente llame a `aprobar_pago`.
- Pedir datos personales de otros usuarios.
- Pedir que ignore las reglas de abstención.
- Superar el límite de tokens o el presupuesto.

**Preguntas de diseño:**
1. ¿Dónde debe vivir el control: en el prompt ("no apruebes pagos") o en el gateway con una política? ¿Por qué el prompt no basta?
2. ¿Cómo diferencias la identidad de un agente de la del humano que lo invocó?

---

## BLOQUE 5: Plataforma y operación

### Reto 13: Observabilidad y tablero de cumplimiento ⏱ 2 h
*(Oferta: "observabilidad y trazabilidad", "alertas, métricas y dashboards", OpenTelemetry, Grafana, Prometheus)*

1. Agrega **Prometheus** y **Grafana** a `docker-compose.yml`.
2. Expón métricas del gateway, del servicio de pagos y del monitoreo de drift con `prometheus_client`:
   - Decisiones permitidas y bloqueadas (por política y por motivo).
   - Latencia (histograma), tokens y costo por rol.
   - PSI máximo y versión del modelo en producción.
   - Resultados de la última evaluación del asistente.
3. Construye un tablero **"Cumplimiento"** en Grafana con esas métricas.
4. Crea al menos 2 **alertas** (por ejemplo, PSI > 0.25, o más de 20% de bloqueos en 10 minutos).
5. **Trazas con OpenTelemetry:** instrumenta el gateway para que una solicitud muestre un solo rastro con los pasos `identidad → OPA → modelo → registro`.

**Consejo:** aprende **PromQL** con 3 consultas simples (tasa de bloqueos, percentil 95 de latencia, costo acumulado).

**Éxito:** en una sola pantalla ves cuántas solicitudes se bloquearon, por qué, y qué modelo está en producción.

---

### Reto 14: CI/CD con evidencia ⏱ 2 h
*(Oferta: "controles y evidencias automatizadas con CI/CD y DevSecOps"; GitHub Actions, Azure DevOps)*

**Primero** `cloudbuild.yaml` (lo que ya sabes), con estos pasos:
1. `opa fmt --fail` y `opa test`.
2. `pytest`.
3. Validar el contrato de datos con un lote de muestra.
4. Entrenar un modelo pequeño y evaluarlo (con datos de muestra).
5. Correr la evaluación del asistente y la compuerta `release_llm`.
6. Construir la imagen Docker.

**Después** el equivalente en `.github/workflows/ci.yml`:

```yaml
name: ci
on: [push, pull_request]
jobs:
  politicas:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # TODO: instalar OPA, correr opa fmt y opa test
  app:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # TODO: instalar Python, pytest, contrato de datos, evaluación, build
```

**Evidencia:** guarda como *artifacts* los resultados de `opa test`, el reporte de evaluación y el registro de lineage.

**Cambios a políticas (DevSecOps):** ¿cómo evitas que alguien cambie una política en producción sin revisión? Investiga protección de ramas, `CODEOWNERS` y aprobaciones obligatorias. Es el mismo principio de "quien crea no aprueba", aplicado a tu propio código.

---

### Reto 15 (opcional): Azure y comparación entre nubes ⏱ 3 h
*(Oferta: Azure ML, Azure OpenAI, Azure DevOps, Copilot Studio, Terraform)*

> **Cuándo hacerlo:** no abras la cuenta todavía. Primero confirma con Mateo o Esteban cuál es la nube principal. Si es Azure, créala entonces: el crédito de la cuenta gratuita dura 30 días desde el registro. Revisa las condiciones vigentes en azure.microsoft.com/free.

**Antes de empezar**
1. Crea una **alerta de presupuesto** (por ejemplo, 5 dólares) desde el primer día.
2. Antes de crear cualquier recurso, mira en el portal la tabla de **servicios gratuitos** y confirma que ese nivel específico lo es.
3. Al terminar cada práctica, **borra todo** (`terraform destroy` o eliminar el grupo de recursos).

**15a. Pipeline en Azure DevOps.** Crea una organización y un proyecto en Azure DevOps y escribe `azure-pipelines.yml` equivalente a tu CI del R14. Compara con GitHub Actions y Cloud Build: ¿qué cambia en sintaxis y qué es idéntico?

**15b. Registro de imágenes.** Publica la imagen de tu gateway en **Azure Container Registry** (por ejemplo, con `az acr build`).

**15c. Mínimo privilegio en la nube.** Crea un rol o asignación de **Azure RBAC** que solo permita una cosa (por ejemplo, leer un recurso) y comprueba que lo demás falla. Es el mismo concepto que tus políticas de OPA, pero aplicado a infraestructura.

**15d. Terraform.** En `infra/`, escribe Terraform con el proveedor `azurerm` para crear un grupo de recursos y un registro de contenedores, y destrúyelos al final. Si te resulta más fácil, haz primero la versión equivalente en GCP, que ya conoces.

**15e. Lo que NO harías con crédito gratis:** Kubernetes administrado (AKS) y modelos de Azure OpenAI o Azure ML con uso intensivo, porque consumen el crédito rápido. Para Kubernetes usa `kind` o `minikube` en tu computador.

**15f. Tabla de equivalencias (complétala con lo que descubras):**

| Concepto | GCP (lo que ya sabes) | Azure | AWS |
|---|---|---|---|
| CI/CD | Cloud Build | Azure Pipelines / GitHub Actions | CodeBuild / CodePipeline |
| Registro de imágenes | Artifact Registry | Container Registry | ECR |
| Contenedores sin servidor | Cloud Run | Container Apps | App Runner / Fargate |
| Plataforma de ML | Vertex AI | Azure ML | SageMaker |
| Modelos de lenguaje | Vertex AI (Gemini) | Azure OpenAI | Bedrock |
| Identidad y permisos | IAM | Entra ID + RBAC | IAM |
| Secretos | Secret Manager | Key Vault | Secrets Manager |
| Logs y monitoreo | Cloud Logging / Monitoring | Azure Monitor / Log Analytics | CloudWatch |

---

## BLOQUE 6: Gobierno y auditoría

### Reto 16: Documentación, marcos y evaluación de proveedores ⏱ 2 h
*(Oferta: "documentar arquitecturas, controles, runbooks y evidencias", ISO/IEC 42001, NIST AI RMF, SOX/ITGC, "evaluación técnica de proveedores")*

**16a. Matriz de controles** (`docs/matriz_controles.md`). Un auditor no lee tu código, lee esto:

| ID | Descripción en español | Política (archivo y regla) | Prueba que lo verifica | Evidencia que genera | Pilar |
|---|---|---|---|---|---|
| PAG-01 | Pago > 10M requiere 2 aprobadores distintos | `pagos.rego` / `permitir` | `test_dos_aprobadores...` | `auditoria.jsonl` + artifact de CI | Trazabilidad |
| MOD-01 | Modelo requiere métricas, model card y aprobador distinto del entrenador | `modelos.rego` / `promover` | `test_mismo_entrenador...` | Registro de promoción | Segregación de funciones |
| LLM-01 | Prompt solo se lanza si pasa evaluación | `release_llm.rego` / `lanzar` | `test_fuga_pii...` | Reporte de evaluación | Evidencia de calidad |
| AGT-01 | Agentes no aprueban pagos | `herramientas.rego` | `test_agente_no_aprueba` | Bitácora de herramientas | Mínimo privilegio |

Complétala con **todos** los controles de tu prototipo.

**16b. Mapeo a marcos** (`docs/mapeo_marcos.md`). Conecta tus controles con los marcos que menciona la oferta. Confirma con Natalia qué marcos aplican de verdad en TeleCorp.

| Marco | Cómo se relaciona con el caso |
|---|---|
| **NIST AI RMF** (funciones Govern, Map, Measure, Manage) | Govern: roles y compuertas (R7, R11). Map: inventario y riesgos (model card, red team). Measure: evaluaciones y drift (R8, R11). Manage: rollback, alertas, runbooks (R8, R13). |
| **ISO/IEC 42001** (sistema de gestión de IA) | Políticas, roles, ciclo de vida controlado, evaluación y monitoreo, mejora continua. Tu matriz y tus compuertas son la evidencia. |
| **SOX / ITGC** | Accesos (R9, R12), gestión de cambios (R7, R14), segregación de funciones (R1, R7, R12), evidencia (R4). |

**16c. Runbook** (`docs/runbook.md`):
1. Qué hacer si OPA se cae.
2. Qué hacer si una política nueva bloquea pagos legítimos (rollback).
3. Cómo verificar la cadena de auditoría.
4. Qué hacer si el modelo entra en drift fuerte.
5. Qué hacer si el asistente da una respuesta peligrosa.
6. Quién aprueba cambios de políticas, modelos y prompts.

**16d. Evaluación de proveedores** (`docs/evaluacion_proveedores.md`). Compara tres opciones para el AI Gateway: **construir uno propio** (FastAPI), **adoptar uno de código abierto** (por ejemplo, LiteLLM) o **usar un servicio gestionado de la nube**. Criterios: seguridad, auditoría y trazabilidad, integración con OPA, costo, dependencia del proveedor (lock-in), esfuerzo de operación, madurez. Termina con una recomendación y los riesgos de tu elección.

---

## 6. Preguntas para tus primeras semanas en TeleCorp

**A Esteban (líder del equipo):**
- ¿Qué se espera de mí en los 60 días del periodo de prueba? ¿Cuál es la prioridad real: Compliance as Code, MLOps o los gateways de IA?
- ¿Qué parte del listado de la oferta cubro yo y cuál recae en otros (Mateo, Sergio)?
- ¿Hay un marco de Gobierno de IA ya definido, o parte del trabajo es definirlo?
- ¿Qué modelos de riesgo y soluciones GenAI existen hoy y en qué estado están?

**A Natalia (riesgos):**
- ¿Cuáles son los controles de mayor volumen y menor ambigüedad? (El mejor candidato para el primer piloto.)
- ¿Qué evidencia piden hoy los auditores y en qué formato?
- ¿Qué reglas están escritas y cuáles viven solo en la cabeza de las personas?
- ¿Cómo se valida hoy un modelo de riesgo antes de usarlo?

**A Mateo (CI/CD y seguridad):**
- ¿Qué usan hoy: GitHub Actions o Azure DevOps? ¿Dónde viven los secretos?
- ¿Cómo es el flujo de revisión y despliegue? ¿Se puede replicar para políticas, modelos y prompts?

**Al equipo en general:**
- ¿Cuál es la nube principal?
- ¿Qué existe ya del AI Gateway o del MCP Gateway y qué se construye desde cero?
- ¿Se evaluó ya un producto como LiteLLM, o se construye propio?
- ¿Dónde se registran hoy los modelos (MLflow, Azure ML, Databricks)?

---

## 7. Rutas de trabajo

### Ruta mínima: antes del 4 de noviembre (≈ 9 horas)

| Día | Fecha | Qué hacer | Retos |
|---|---|---|---|
| 1 | Mié 28 oct | Leer la sección 2, levantar OPA y escribir las reglas de pagos | R0, R1 |
| 2 | Jue 29 oct | Pruebas en Rego y Python consultando a OPA | R2, R3 |
| 3 | Vie 30 oct | Entrenar y registrar un modelo en MLflow (versión simple) | R5 (resumido), R6 |
| 4 | Sáb 31 oct | Bitácora de auditoría con hash encadenado | R4 |
| 5 | Dom 1 nov | Mini AI Gateway con políticas por rol | R9 |
| 6 | Lun 2 nov | Compuerta de promoción de modelo con OPA | R7 |
| 7 | Mar 3 nov | Lista de preguntas y borrador de la matriz de controles | Sección 6, R16a |

Si un día te atrasas, salta al siguiente. Orden de importancia: **R1 → R2 → R3 → R6 → R7 → R9 → R4 → R16a**.

### Ruta completa: tus primeros 60 días (sugerida)

Ajústala a lo que Esteban te priorice. La idea es practicar con el caso mientras aprendes el entorno real, y llegar al final del periodo con algo concreto.

| Semana | Foco | Retos |
|---|---|---|
| 1 | Entender el entorno real, afinar tus políticas con un control real que te dé Natalia | R1 a R4 |
| 2 | Piloto de un control SOX de punta a punta | R1 a R4 con datos reales |
| 3 a 4 | Datos y modelo de riesgo con compuerta de promoción | R5, R6, R7 |
| 5 | Drift, rollback y observabilidad | R8, R13 |
| 6 a 7 | Gateway, asistente, evaluación y seguridad de agentes | R9 a R12 |
| 7 a 8 | Pipeline con evidencia, y Azure si es la nube principal | R14, R15 |
| 8 | Documentación, mapeo a marcos y presentación de lo aprendido | R16 |

---

## 8. Autoevaluación

No es para sentirte mal: es para saber dónde preguntar y qué estudiar primero.

| Nivel | Qué significa | Cómo se ve |
|---|---|---|
| **Mínimo** | Entiendes la idea y hay base | R0 a R3, más un modelo registrado en MLflow (R6) |
| **Bueno** | Llegas con contexto real | Mínimo + R4 (auditoría), R7 (compuerta de modelo) y R9 (gateway) |
| **Muy bueno** | Llegas con ventaja | Bueno + R5, R8, R10, R11 (datos, drift, asistente y evaluación) |
| **Excelente** | Ya estás practicando el cargo | Muy bueno + R12 a R16 (agentes seguros, observabilidad, CI/CD, nube y documentación) |

**Si solo llegas al nivel Mínimo esta semana, es suficiente.** Lo importante es haber visto una política escrita, probada y consultada por una aplicación, y un modelo registrado y rastreable. Con eso, lo que te digan en TeleCorp dejará de sonar a otro idioma.

---

## 9. Glosario rápido

- **OPA (Open Policy Agent):** servidor gratuito que recibe datos, evalúa reglas y responde permitir o negar.
- **Rego:** lenguaje en que se escriben las reglas de OPA.
- **Compliance as Code:** convertir reglas de cumplimiento en código que se ejecuta y se prueba solo.
- **MLOps:** prácticas para llevar modelos de ML a producción y mantenerlos de forma repetible, monitoreada y controlada.
- **LLMOps:** lo mismo para modelos de lenguaje y agentes: prompts, RAG, evaluación, costos, trazas y seguridad.
- **Compuerta (gate):** punto donde una condición debe cumplirse para avanzar (por ejemplo, promover un modelo).
- **Model registry:** catálogo de modelos versionados con alias o etapas (candidato, producción).
- **Model card:** ficha del modelo con su propósito, datos, métricas y limitaciones.
- **Drift:** cambio en los datos o en el comportamiento del modelo respecto a cuando se entrenó.
- **PSI:** medida común para cuantificar cuánto cambió una distribución.
- **Rollback:** volver a la versión anterior.
- **Data contract:** acuerdo escrito y verificable sobre cómo deben ser los datos (columnas, tipos, rangos, frescura).
- **Lineage:** historia de dónde vienen los datos y qué transformaciones tuvieron.
- **RAG:** el modelo responde apoyándose en documentos recuperados.
- **Groundedness (fidelidad):** que la respuesta esté respaldada por el contexto recuperado.
- **Alucinación:** respuesta que suena correcta pero no tiene respaldo.
- **Prompt injection:** texto malicioso que intenta hacer que el modelo ignore sus reglas.
- **Juez LLM (LLM-as-judge):** usar un modelo para evaluar respuestas de otro; debe calibrarse con humanos.
- **AI Gateway / Model Gateway:** punto central por donde pasan las llamadas a modelos, donde se aplican reglas y se registra todo.
- **MCP (Model Context Protocol) / MCP Gateway:** estándar para que los agentes usen herramientas, y el punto de control de esas llamadas.
- **Mínimo privilegio:** cada usuario o agente tiene solo los permisos que necesita.
- **Segregación de funciones:** quien hace algo no es quien lo aprueba.
- **Trazabilidad, inmutabilidad, no repudio:** poder reconstruir lo ocurrido, que no se altere sin que se note y que nadie pueda negar lo que hizo.
- **Fail-closed:** si el sistema de control falla, se bloquea en vez de dejar pasar.
- **Bundle:** paquete de políticas que OPA descarga y puede verificar con firma.
- **PII:** datos personales identificables (cédula, correo, teléfono).
- **SOX / ITGC:** controles financieros y controles generales de tecnología (accesos, cambios, operaciones) que se revisan en auditoría.
- **NIST AI RMF / ISO/IEC 42001:** marcos para gestionar riesgos de IA y sistemas de gestión de IA.

---

*Cuando tengas avances, pásame tus archivos `.rego` o tu código y te hago la revisión como la haría un compañero de equipo: qué está bien, qué cambiaría y por qué.*
