--------------------------------------------------------------------------------
DESCRIPCIÓN DEL PROYECTO
--------------------------------------------------------------------------------

Aplicación que determina, mediante un algoritmo evolutivo (genético), qué
posiciones de una celda de manufactura deben equiparse con sensores y de qué
tipo, maximizando la cobertura ponderada de zonas críticas sin exceder límites
de presupuesto ni de consumo energético.

El problema se formula como optimización monoobjetivo con restricciones: el
costo y la energía actúan como límites duros (no como objetivos), y el único
objetivo a maximizar es la cobertura ponderada de criticidad.

--------------------------------------------------------------------------------
MODELADO DEL ESCENARIO
--------------------------------------------------------------------------------

CUADRÍCULA
  - Tamaño     : 10 × 10 unidades
  - Escala     : 1 unidad = 1.5 m  →  área física de 15 m × 15 m
  - Criticidad : escala 1–5 por celda (5 = mayor riesgo operativo)

ESCENARIOS DE CRITICIDAD DISPONIBLES
  1. Bordes críticos  — criticidad concentrada en el perímetro
  2. Centro crítico   — criticidad concentrada en zonas centrales
  3. Zonas dispersas  — focos críticos repartidos por toda la celda

SENSORES INDUSTRIALES (parámetros extraídos de fichas técnicas)

  Tipo                       Modelo              Radio (ud)  Costo (USD)  Consumo (W)
  ─────────────────────────  ──────────────────  ──────────  ───────────  ───────────
  Vibración                  ifm VSA001              1.00         200        0.16
  Ultrasónico                SICK UM30               2.25         450        1.20
  Temperatura (IR)           Optris Xi 80            2.50       1 800        2.50
  Cámara de visión           Cognex In-Sight 8000    3.00       2 500        6.50

RESTRICCIONES
  - Presupuesto máximo  P_M = n × 2 500 USD   (n definido por el usuario)
  - Energía máxima      E_M = n × 6.5 W       (n definido por el usuario)

POSICIONES CANDIDATAS
  15 ubicaciones físicamente viables, definidas libremente por el usuario
  como pares (fila, columna) dentro de la cuadrícula 0–9.

--------------------------------------------------------------------------------
ALGORITMO GENÉTICO
--------------------------------------------------------------------------------

CODIFICACIÓN DEL CROMOSOMA
  - Longitud fija: 15 genes (uno por posición candidata)
  - Alelos: enteros en {0, 1, 2, 3, 4}
      0 → sin sensor
      1 → sensor de vibración
      2 → sensor ultrasónico
      3 → sensor de temperatura
      4 → cámara de visión
  - Espacio de búsqueda: 5^15 ≈ 3 × 10^10 configuraciones

FUNCIÓN DE CALIDAD  f(x) = CP − φ_costo − φ_energía − Σ φ_redundancia

  CP         Cobertura ponderada: suma de criticidad de celdas cubiertas
             (unión de coberturas, cada celda contada una sola vez)

  φ_costo    1000 × (Ctotal − P_M)   si Ctotal > P_M, 0 en otro caso
  φ_energía  1000 × (Etotal − E_M)   si Etotal > E_M, 0 en otro caso
  φ_red      500 por cada sensor cuya cobertura sea subconjunto de la
             unión del resto (penalización por redundancia — aporte propio)

  Máximo teórico de fitness: 296 (suma de criticidad de toda la cuadrícula)

OPERADORES
  - Inicialización : alelos uniformes en {0,1,2,3,4} via random.randint
  - Selección      : torneo de tamaño 3 (selTournament, DEAP)
  - Cruce          : dos puntos (cxTwoPoint)   — probabilidad: 0.70
  - Mutación       : uniforme entera (mutUniformInt, indpb=0.2) — prob: 0.30
  - Reemplazo      : generacional completo + Hall of Fame (capacidad 5)

HIPERPARÁMETROS ÓPTIMOS (determinados mediante estudio sistemático)
  Parámetro              Valor seleccionado
  ─────────────────────  ──────────────────
  Tamaño de población    75
  Generaciones           100
  Probabilidad de cruce  0.70
  Tasa de mutación       0.30
  Mecanismo selección    Torneo (k = 3)

--------------------------------------------------------------------------------
RESULTADOS DE VALIDACIÓN (n = 4  →  P_M = 10 000 USD, E_M = 26.0 W)
--------------------------------------------------------------------------------

  Escenario           Sensores  Costo (USD)  Energía (W)  Cobertura    %
  ──────────────────  ────────  ───────────  ───────────  ─────────  ─────
  1 — Bordes críticos     10       9 950        23.9       287/296    97.0
  2 — Centro crítico       9       9 500        22.7       206/234    88.0
  3 — Zonas dispersas      9       9 950        25.7       191/200    95.5

  En todos los escenarios se respetan ambas restricciones.
  El Escenario 2 evidencia que la cobertura máxima alcanzable está acotada
  por la disposición de las posiciones candidatas, no por el optimizador.

--------------------------------------------------------------------------------
DEPENDENCIAS Y EJECUCIÓN
--------------------------------------------------------------------------------

DEPENDENCIAS PYTHON
  - Python 3.x
  - DEAP    (framework de algoritmos evolutivos)
  - NumPy
  - Matplotlib
  - math    (librería estándar)

EJECUCIÓN
  1. Instalar dependencias:
       pip install deap numpy matplotlib

  2. Ejecutar el notebook en Google Colab (ver enlace en Anexo I del informe)
     o lanzar el script principal localmente.

  3. Al iniciar, el programa solicita:
       - 15 posiciones candidatas en formato "fila,columna" (valores 0–9)
       - Valor de n para presupuesto  (P_M = n × 2 500 USD)
       - Valor de n para energía      (E_M = n × 6.5 W)

  4. El algoritmo corre 100 generaciones y muestra:
       - Tabla de sensores activos con posición, tipo, costo y energía
       - Cobertura ponderada obtenida vs. total del mapa
       - Celdas no cubiertas y su criticidad
       - Gráfica de convergencia (mejor fitness histórico)
       - Visualización del mapa con coberturas y sensores instalados

--------------------------------------------------------------------------------
ESTRUCTURA DEL CÓDIGO (funciones principales)
--------------------------------------------------------------------------------

  evaluar(individuo)
      Función de calidad. Calcula cobertura ponderada y aplica penalizaciones
      por violación de restricciones y por redundancia.

  correr_evolutivo(popu_size, generations, mate_chance, mutate_chance,
                   tournsize, ...)
      Ciclo generacional completo. Retorna Hall of Fame e historial de fitness.

  solicitar_posiciones()
      Captura y valida las 15 posiciones candidatas ingresadas por el usuario.

  solicitar_parametros()
      Solicita los valores de n y calcula P_M y E_M.

  visualizar_solucion(mejor, ...)
      Genera el mapa de criticidad con sensores, radios de cobertura y
      celdas no cubiertas marcadas.

  mostrar_resultados(hof_final, h_max_final, nombre_escenario)
      Imprime tabla de resultados y llama a visualizar_solucion().
