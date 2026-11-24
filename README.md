# 🏦 Motor End-to-End de Riesgo Crediticio, Prevención de Fraude (AML) y Forecasting Financiero en SAS Viya

![SAS Viya](https://img.shields.io/badge/SAS-Viya%20%7C%20CAS-blue)
![Methodology](https://img.shields.io/badge/Methodology-SEMMA-green)
![Domain](https://img.shields.io/badge/Domain-Credit%20Risk%20%26%20AML-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## 📋 Resumen Ejecutivo

Este proyecto implementa una arquitectura analítica empresarial de extremo a extremo (*End-to-End*) en **SAS Viya** bajo la metodología **SEMMA**, aprovechando el motor distribuido en memoria **CAS (Cloud Analytic Services)**. 

El sistema resuelve tres pilares críticos de la banca moderna y las FinTech en un mismo flujo integrado:
1. **Vigilancia Forense y Cumplimiento (AML):** Detección en tiempo real de tipologías críticas de lavado de dinero (`SAR`).
2. **Scoring Predictivo sin Data Leakage:** Evaluación supervisada de riesgo crediticio y fraude transaccional mediante **Gradient Boosting**.
3. **Planeación Financiera y Explicabilidad (XAI):** Proyección de volumen transaccional a 12 meses (**Holt-Winters**) y generación automatizada de dictámenes explicables para auditoría y comités de riesgo.

---

## 🏗️ Arquitectura del Pipeline (SEMMA + CAS)

```text
[Ingesta Sintética Controlada] 
       │  (5,000 Clientes | 25,000 Transacciones -> Promoción a CASUSER)
       ▼
[1. Motor Forense AML (PROC FEDSQL)] ──► Detección de Tipologías SAR (Offshore, Madrugada, Montos Atípicos)
       │
       ├──► [2. Forecasting Financiero (PROC ESM)] ──► Serie Mensual Holt-Winters (12M + IC 95%)
       │
       ▼
[3. Pipeline ML (SAS Model Studio)] ──► Competencia de Modelos (Gradient Boosting Champion)
       │                                  * Control estricto de Data Leakage (Exclusión Ex-Post)
       ▼
[4. Matriz Híbrida + Capa XAI] ──────► Decisión en Cascada (Reglas AML > Umbral ML 0.50) + Cartas LLM
       │
       ▼
[5. SAS Visual Analytics] ───────────► Dashboard Ejecutivo Interactivo + Pronóstico de Liquidez
