# CapitalQuant

## AI-Powered Investment Decision Support

---

## Definición Conceptual del Producto

**CapitalQuant** es una plataforma de apoyo a la decisión para analistas financieros y gestores de portafolios que combina modelos de Machine Learning supervisado, explicabilidad mediante SHAP y recuperación aumentada sobre documentos oficiales para priorizar oportunidades de inversión de manera objetiva, transparente y respaldada por evidencia documental.

---

## Contexto y Justificación

Los profesionales de inversión enfrentan tres desafíos estructurales:

| Desafío | Descripción |
|---------|-------------|
| **Sobrecarga de información** | Miles de empresas, cientos de ratios financieros, docenas de reportes anuales por empresa, más noticias y datos macroeconómicos |
| **Sesgos cognitivos** | Decisiones influenciadas por ruido del mercado, emociones o sesgos de confirmación |
| **Falta de trazabilidad** | Dificultad para reconstruir y explicar objetivamente por qué se tomó una decisión de inversión específica |

La plataforma aborda estos problemas ofreciendo:

1. **Priorización objetiva** - Basada en patrones históricos aprendidos por el modelo
2. **Explicaciones transparentes** - Cada recomendación viene acompañada de una descomposición clara de factores
3. **Evidencia documental** - Las recomendaciones se respaldan con información extraída de documentos oficiales

---

## El Problema que Resuelve

Los modelos predictivos proporcionan una estimación cuantitativa, pero rara vez explican suficientemente las razones detrás de una recomendación. La plataforma combina aprendizaje supervisado, explicabilidad y recuperación de evidencia documental para ofrecer decisiones transparentes y auditables, permitiendo que el analista mantenga el control final.

---

## Componentes Fundamentales

### 1. Motor Predictivo

El sistema utiliza XGBoost para predecir qué empresas tienen mayor probabilidad de superar al S&P500 en los próximos 252 días hábiles. El modelo se entrena con datos fundamentales y de mercado de los últimos 15 años utilizando validación walk-forward para evitar data leakage y simular condiciones reales de producción.

**Target del modelo:**