/* ===================================================================== */
/* FASE 3: SAS Forecasting (Proyección de Series de Tiempo a 12 Meses)   */
/* ===================================================================== */

cas mi_sesion_cas;
libname casuser cas caslib="CASUSER";

/* 1. Preparamos la serie temporal agregada por Mes (primer día de cada mes) */
data WORK.SERIE_MENSUAL_RAW;
    set casuser.TRANSACCIONES_360;
    /* La función INTNX('month', Fecha, 0, 'B') agrupa cualquier fecha al inicio de su mes */
    Mes_Anio = intnx('month', Fecha_Tx, 0, 'B');
    format Mes_Anio date9.;
run;

/* 2. Consolidamos el total de transacciones y monto mensual */
proc sql;
    create table WORK.SERIE_HISTORICA_MES as
    select 
        Mes_Anio,
        count(ID_Tx) as Volumen_Tx,
        sum(Monto_USD) as Monto_Total_USD format=dollar14.2
    from WORK.SERIE_MENSUAL_RAW
    group by Mes_Anio
    order by Mes_Anio;
quit;

/* 3. Filtramos meses completos (eliminamos el último mes si quedó incompleto por corte de fecha) */
data casuser.SERIE_FORECAST_INPUT(promote=yes);
    set WORK.SERIE_HISTORICA_MES;
    if Mes_Anio <= '01OCT2026'd; 
run;

/* 4. Ejecutamos SAS Forecasting (Suavizamiento Exponencial con Tendencia y Estacionalidad) */
proc esm data=casuser.SERIE_FORECAST_INPUT 
         outfor=casuser.PROYECCION_12_MESES(promote=yes) 
         lead=12 
         plot=(forecasts modelforecasts);
    id Mes_Anio interval=month;
    forecast Monto_Total_USD / model=winters;
    title "📈 SAS Forecasting: Proyección de Volumen Financiero (Próximos 12 Meses)";
run;

/* 5. Mostramos la tabla con el pronóstico futuro (donde ACTUAL es nulo) */
proc print data=casuser.PROYECCION_12_MESES(obs=15);
    where actual is missing;
    var Mes_Anio predict lower upper std;
    format predict lower upper dollar14.2;
    title "🔮 Pronóstico Mensual a 12 Meses e Intervalos de Confianza (95%)";
run;
