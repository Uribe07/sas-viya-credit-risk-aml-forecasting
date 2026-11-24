/* ===================================================================== */
/* FASE 1: Conexión al Motor CAS (Viya) y Generación de Datos Masivos    */
/* ===================================================================== */

/* 1. Iniciamos sesión en el motor in-memory CAS y asignamos la librería CASUSER */
cas mi_sesion_cas;
libname casuser cas caslib="CASUSER";

/* 2. Generamos 5,000 Clientes con perfiles financieros realistas */
data casuser.CLIENTES_360(promote=yes);
    call streaminit(2026); /* Semilla para reproducibilidad */
    length ID_Cliente $10 Region $15 Segmento $12;
    
    array regiones[4] $15 _temporary_ ('Centro', 'Norte', 'Sur', 'Occidente');
    array segmentos[3] $12 _temporary_ ('Retail', 'PyME', 'Preferente');

    do i = 1 to 5000;
        ID_Cliente = cats('CLI_', put(i, z5.));
        Edad = rand('integer', 21, 70);
        Region = regiones[rand('integer', 1, 4)];
        Segmento = segmentos[rand('integer', 1, 3)];
        
        /* Ingreso mensual correlacionado con el segmento */
        if Segmento = 'Retail' then Ingreso_Mensual = round(rand('normal', 18000, 5000), 100);
        else if Segmento = 'PyME' then Ingreso_Mensual = round(rand('normal', 45000, 12000), 100);
        else Ingreso_Mensual = round(rand('normal', 85000, 20000), 100);
        if Ingreso_Mensual < 6000 then Ingreso_Mensual = 6000;

        /* Score crediticio (300 a 850) */
        Score_Credito = round(rand('normal', 640, 85));
        if Score_Credito < 300 then Score_Credito = 300;
        if Score_Credito > 850 then Score_Credito = 850;

        /* Monto solicitado y Mora histórica */
        Monto_Solicitado = round(Ingreso_Mensual * rand('uniform', 1.5, 6.0), 500);
        Prob_Mora = 1 / (1 + exp((Score_Credito - 580)/45));
        Historial_Mora = rand('bernoulli', Prob_Mora); /* 1 = Mora previa, 0 = Limpio */
        
        output;
    end;
    drop i Prob_Mora;
run;

/* 3. Generamos 25,000 Transacciones (incluyendo patrones de fraude y fechas históricas) */
data casuser.TRANSACCIONES_360(promote=yes);
    call streaminit(2026);
    length ID_Tx $12 ID_Cliente $10 Pais_Tx $15 Canal $12;
    format Fecha_Tx date9.;
    
    array paises[5] $15 _temporary_ ('Mexico', 'Mexico', 'Mexico', 'Rusia', 'Islas_Caiman');
    array canales[3] $12 _temporary_ ('App_Movil', 'Web', 'POS_Fisico');

    do j = 1 to 25000;
        ID_Tx = cats('TX_', put(j, z6.));
        ID_Cliente = cats('CLI_', put(rand('integer', 1, 5000), z5.));
        
        /* Fechas distribuidas en los últimos 3 años (para SAS Forecasting) */
        Fecha_Tx = '01JAN2024'd + rand('integer', 0, 1050);
        Hora_Tx = rand('integer', 0, 23);
        Canal = canales[rand('integer', 1, 3)];
        
        /* 94% operaciones nacionales, 6% internacionales de riesgo */
        if rand('uniform') < 0.94 then Pais_Tx = 'Mexico';
        else Pais_Tx = paises[rand('integer', 4, 5)];

        /* Monto de transacción */
        if Pais_Tx ne 'Mexico' then Monto_USD = round(rand('uniform', 2500, 15000), 2);
        else Monto_USD = round(rand('exponential') * 450 + 20, 2);

        output;
    end;
    drop j;
run;

/* 4. Verificamos carga en memoria CAS */
proc casutil;
    list tables incaslib="CASUSER";
quit;
