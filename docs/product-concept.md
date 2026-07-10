
### 2. Motor de Explicabilidad

SHAP analiza cada predicción y descompone el score en contribuciones individuales de cada variable financiera, permitiendo al analista ver exactamente qué factores (ROE, crecimiento de ingresos, margen, etc.) influyeron positiva o negativamente en la recomendación.

### 3. Motor Documental

Un sistema RAG procesa reportes 10-K y 10-Q de la SEC, permitiendo a los analistas preguntar en lenguaje natural sobre los documentos que respaldan una recomendación. El sistema siempre cita las fuentes exactas (sección y página del documento original).

### 4. Capa de Presentación

Un dashboard interactivo que permite:
- Visualizar el ranking de empresas con sus scores
- Explorar explicaciones detalladas de cada recomendación
- Consultar documentos oficiales mediante lenguaje natural

---

## Flujo de Uso Principal

### Escenario: Revisión de Oportunidades de Inversión

Un gestor de portafolio necesita identificar nuevas oportunidades de inversión para el próximo trimestre. Accede al sistema y observa un ranking de 500 empresas ordenadas por su Investment Score. Para entender mejor a los líderes, selecciona una empresa y el sistema le muestra:

1. El score numérico de la empresa
2. Un desglose SHAP que explica qué factores impulsaron el score (ej. "ROE de 32% contribuyó +25 puntos; P/E de 34x restó -8 puntos")
3. Un chat documental donde puede preguntar: "¿Qué dice el último 10-K sobre el crecimiento de esta empresa?" y el sistema responde extrayendo la sección relevante del reporte

El gestor ahora tiene una priorización objetiva, una explicación clara de cada oportunidad y acceso directo a la evidencia documental, lo que le permite tomar decisiones más informadas y defendibles.

---

## Evaluación del Sistema

### Evaluación del Modelo
- Walk-forward validation (2010-2024)
- ROC AUC, Precision, Recall, Accuracy
- Calibration Curve
- Confusion Matrix
- SHAP Global y Local

### Evaluación del RAG
- Precision@5
- Recall@5
- Faithfulness
- Context Precision

---

## Limitaciones

- No ejecuta órdenes de compra
- No constituye asesoría financiera
- No incorpora noticias en tiempo real
- Se basa en información pública disponible
- Está diseñado para apoyar el análisis, no reemplazar al analista

---

## Valor para el Usuario

| Beneficio | Descripción |
|-----------|-------------|
| **Objetividad** | Las recomendaciones se basan en datos y patrones históricos, no en intuición o sesgos |
| **Transparencia** | Cada recomendación es explicable y desglosable en factores individuales |
| **Eficiencia** | Reduce el tiempo necesario para filtrar y evaluar oportunidades |
| **Trazabilidad** | Cada decisión puede respaldarse con evidencia documental |
| **Auditabilidad** | El sistema permite reconstruir por qué se hizo cada recomendación |

---

## Posicionamiento

La plataforma no pretende reemplazar el juicio del analista, sino **amplificar su capacidad de análisis** proporcionando una capa de inteligencia objetiva y explicable sobre la cual tomar decisiones informadas.

---

## Diferenciadores

| Característica | Beneficio |
|----------------|-----------|
| **Explicabilidad como característica principal** | SHAP no es un extra, es parte del valor fundamental del producto |
| **Evidencia documental integrada** | Las recomendaciones se respaldan con documentos oficiales, no solo con opiniones |
| **Arquitectura de producto** | Diseñado como software empresarial, no como un notebook de investigación |
| **Validación walk-forward** | El modelo se evalúa en condiciones realistas que simulan producción |

---

## Roadmap Técnico

| Fase | Resultado |
|------|-----------|
| 1 | ETL desde Yahoo Finance, FRED y SEC EDGAR |
| 2 | Pipeline de features y dataset reproducible |
| 3 | Modelo XGBoost con validación walk-forward |
| 4 | SHAP global y local |
| 5 | API FastAPI para predicciones y explicaciones |
| 6 | Ingesta de 10-K, embeddings y RAG |
| 7 | Dashboard React |
| 8 | Docker, README y demo |

---

## Tecnologías Clave

| Área | Tecnología |
|------|------------|
| Machine Learning | XGBoost, scikit-learn |
| Explicabilidad | SHAP |
| Document Intelligence | RAG con LangChain, ChromaDB |
| Backend | FastAPI, Python |
| Frontend | React, TypeScript |
| Containerización | Docker |
| MLOps | MLflow |
