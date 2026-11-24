/* ===================================================================== */
/* FASE 2: SAS Forensics & AML (Detección de Fraude y Lavado de Dinero)  */
/* ===================================================================== */

cas mi_sesion_cas;
libname casuser cas caslib="CASUSER";

/* 1. Perfilamiento Transaccional por Cliente (Frecuencia y Montos) */
proc fedsql sessref=mi_sesion_cas;
    create table casuser.PERFIL_TX_CLIENTE {options replace=true} as
    select 
        ID_Cliente,
        count(ID_Tx) as Total_Tx,
        sum(Monto_USD) as Monto_Total_USD,
        avg(Monto_USD) as Ticket_Promedio_USD,
        max(Monto_USD) as Max_Tx_USD,
        sum(case when Pais_Tx <> 'Mexico' then 1 else 0 end) as Tx_Extranjeras,
        sum(case when Hora_Tx between 0 and 5 then 1 else 0 end) as Tx_Madrugada
    from casuser.TRANSACCIONES_360
    group by ID_Cliente;
quit;

/* 2. Motor de Scoring Forense y Reglas AML (Anti-Money Laundering) */
data casuser.ALERTAS_FORENSICS(promote=yes);
    merge casuser.CLIENTES_360(in=a) casuser.PERFIL_TX_CLIENTE(in=b);
    by ID_Cliente;
    if a;

    /* Rellenamos ceros para clientes sin transacciones */
    array nums Total_Tx Monto_Total_USD Ticket_Promedio_USD Max_Tx_USD Tx_Extranjeras Tx_Madrugada;
    do over nums;
        if missing(nums) then nums = 0;
    end;

    /* Cálculo de Score Forense (0 a 100+) */
    Score_Forense = 0;
    length Tipologia_Fraude $120 Estatus_Forense $30;
    Tipologia_Fraude = "";

    /* Tipología 1: Jurisdicción de Alto Riesgo (Lavado / Offshore) */
    if Tx_Extranjeras >= 1 then do;
        Score_Forense + (Tx_Extranjeras * 35);
        Tipologia_Fraude = catx(" | ", Tipologia_Fraude, "Operacion_Offshore_Riesgo");
    end;

    /* Tipología 2: Actividad Nocturna Anómala */
    if Tx_Madrugada >= 2 then do;
        Score_Forense + 25;
        Tipologia_Fraude = catx(" | ", Tipologia_Fraude, "Frecuencia_Madrugada");
    end;

    /* Tipología 3: Desbalance Ingreso vs Gasto (Posible Pitufeo / Testaferro) */
    /* Si gastó en USD más del 150% de su ingreso mensual declarado */
    if (Monto_Total_USD * 18) > (Ingreso_Mensual * 1.5) and Monto_Total_USD > 0 then do;
        Score_Forense + 30;
        Tipologia_Fraude = catx(" | ", Tipologia_Fraude, "Inconsistencia_Ingresos_AML");
    end;

    /* Dictamen Forense */
    if Score_Forense >= 65 then Estatus_Forense = 'SAR_INVESTIGACION_CRITICA';
    else if Score_Forense >= 30 then Estatus_Forense = 'ALERTA_PREVENTIVA';
    else do;
        Estatus_Forense = 'OPERACION_LIMPIA';
        Tipologia_Fraude = 'Sin Hallazgos';
    end;

    /* Definimos la variable Target real para entrenar nuestro modelo de ML */
    if Historial_Mora = 1 or Score_Forense >= 65 then Target_Riesgo_Total = 1;
    else Target_Riesgo_Total = 0;
run;

/* 3. Resumen Ejecutivo de Hallazgos Forenses (SAS Analytics) */
proc freq data=casuser.ALERTAS_FORENSICS;
    tables Estatus_Forense Region*Estatus_Forense / nopercent norow;
    title "🕵️‍♂️ Reporte SAS Forensics: Distribución de Alertas AML y Fraude";
run;
