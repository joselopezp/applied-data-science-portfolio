# Resumen Ejecutivo — Sistema de Detección de Fraude

> **Decisión que apoya:** ¿A qué umbral de clasificación debe marcarse una transacción como
> fraudulenta, dado un ratio de costo de 20x entre no detectar fraude y bloquear una transacción legítima?

---

## Contexto

Las instituciones financieras enfrentan dos modos de falla simultáneos en la detección de
fraude: fraude no detectado (pérdida monetaria directa) y transacciones legítimas bloqueadas
incorrectamente (fricción con el cliente y riesgo de churn). Los enfoques de ML estándar que
optimizan por exactitud fallan silenciosamente — un modelo que clasifica toda transacción como
legítima logra 99,83% de accuracy sin detectar ningún fraude. Los stakeholders son el equipo
de Riesgo/Fraude (maximizar el fraude detectado), el equipo de Experiencia del Cliente
(minimizar bloqueos incorrectos) y el CFO (minimizar la pérdida monetaria total esperada).

---

## Hallazgos Clave

| Hallazgo | Evidencia | Implicancia |
|---|---|---|
| La exactitud (accuracy) es la métrica equivocada | El clasificador naive logra 99,83% de accuracy detectando 0% del fraude | AUC-PR y Pérdida Esperada deben reemplazar a la accuracy como criterios de evaluación |
| El umbral por defecto (0,5) es subóptimo | Ratio de costo 20x (FN=$300, FP=$15) desplaza el umbral óptimo a t*=0,0414 | La calibración del umbral por función de costo es obligatoria, no opcional |
| class_weight supera a SMOTE | RF + class_weight gana el leaderboard de 9 modelos en AUC-PR (val=0,8584) | El sobremuestreo sintético (SMOTE) no mejora sobre la ponderación nativa de clases aquí |
| Los datos etiquetados agregan valor significativo | Isolation Forest (no supervisado) logra menor AUC-PR que todos los modelos supervisados | La inversión en etiquetar fraude está justificada por la ganancia medible en rendimiento |
| La calibración de costos impulsa el valor de negocio | El modelo en t*=0,0414 reduce la Pérdida Esperada un 79,7% vs el baseline naive | Datos de costos reales (no supuestos) mejorarían aún más el umbral |

---

## Resultados

| | Baseline naive | Este sistema |
|---|---|---|
| Fraude detectado | 0 de 71 (0%) | 58 de 71 (81,7%) |
| Transacciones legítimas bloqueadas | 0 | 28 |
| Pérdida Esperada | USD 21.300 | USD 4.320 |
| **Ahorro** | — | **USD 16.980 (79,7%)** |

**Modelo:** Random Forest + class_weight='balanced' · Umbral: t* = 0,0414
**Conjunto de prueba:** 42.721 transacciones · 71 casos de fraude (tasa 0,17%)

### Objetivos vs resultados

| Métrica | Objetivo | Resultado | ¿Cumplido? |
|---|---|---|---|
| AUC-PR | ≥ 0,85 | 0,7861 | ❌ |
| Recall (fraude) | ≥ 0,80 | 0,8169 | ✅ |
| Precisión (fraude) | ≥ 0,70 | 0,6744 | ❌ |
| Pérdida Esperada vs naive | < baseline | −79,7% | ✅ |

**Sobre los objetivos no cumplidos:** El AUC-PR (0,7861 vs 0,85) refleja la dificultad de
las features PCA anonimizadas — abordable con tuning de XGBoost. La Precisión (0,6744 vs
0,70) es la consecuencia esperada del ratio de costo 20x: el negocio priorizó explícitamente
detectar fraude sobre evitar falsas alarmas. Ambas brechas están documentadas, no ocultadas.

---

## Recomendación

**Desplegar como filtro de primera línea con revisión humana de todas las transacciones marcadas.**
El sistema detecta el 81,7% del fraude con un costo 79,7% menor que no hacer nada, manteniendo
los falsos positivos en un nivel manejable para revisión manual (28 por ventana de prueba). El
parámetro de ratio de costo (actualmente supuesto en 20x) debe calibrarse con datos de pérdidas
reales de la institución antes del despliegue en producción.

---

## Limitaciones y Supuestos

- El dataset cubre solo 2 días de 2013 (tarjetahabientes europeos) — los patrones de fraude han evolucionado
- Las features V1–V28 son componentes PCA anonimizados — no es posible ingeniería de features por dominio
- cost_fn=$300 y cost_fp=$15 son valores supuestos — no derivados de registros de pérdidas reales
- El modelo no fue probado en datos de streaming en tiempo real
- No se implementó pipeline de reentrenamiento automatizado

---

## Próximos Pasos

| Acción | Responsable | Plazo |
|---|---|---|
| Calibrar cost_fn y cost_fp con datos reales de pérdidas institucionales | Equipo de Riesgo / Finanzas | Sprint 1 |
| Implementar cola de revisión humana para transacciones marcadas | Equipo de Operaciones | Sprint 1 |
| Probar XGBoost con hyperparameter tuning (cerrar brecha AUC-PR) | Equipo ML | Sprint 2 |
| Reentrenar el modelo mensualmente con nuevos datos de transacciones | Equipo ML | Continuo |
| Exponer el modelo en REST API para scoring en tiempo real | Equipo de Ingeniería | Sprint 3 |

---

*CRISP-DM + Lean · applied-data-science-portfolio · Abril 2026*
*Jose Lopez Pino — Ingeniero Civil Industrial → Científico de Datos | Enfoque en Negocios*
