
# Bot de Trading Algorítmico en Python para MetaTrader 4

Este proyecto proporciona un framework robusto, modular y listo para producción para desarrollar y ejecutar estrategias de trading automatizadas en Python, conectado al terminal de MetaTrader 4 (MT4). Utiliza un puente de comunicación basado en archivos para enviar comandos y recibir datos de MT4, lo que permite desarrollar y ejecutar estrategias complejas con la potencia de las bibliotecas científicas y de análisis de datos de Python.

La arquitectura está diseñada para un desarrollo rápido de estrategias y una operación segura y sin supervisión, separando la lógica central de trading de las complejidades de la gestión de datos, el control de riesgo y la comunicación con el bróker.

## Características Principales

*   **Arquitectura Modular:** Separación clara de responsabilidades entre el cliente de conexión, el gestor de trades, el manejador de eventos y las propias estrategias.
*   **Estrategias Dinámicas "Plug-and-Play":** Una fábrica de estrategias te permite activar cualquier estrategia simplemente cambiando su nombre en `main.py`. No más bloques `if/elif` o importaciones manuales.
*   **Gestión de Riesgo Avanzada:** Un `TradeManager` centralizado que maneja reglas de riesgo sofisticadas basadas en porcentajes:
    *   Dimensionamiento dinámico de posiciones basado en el porcentaje de riesgo de la cuenta.
    *   Stop Loss y Take Profit basados en porcentajes.
    *   Trailing Stop Loss basado en porcentajes.
    *   Toma de Beneficios Parciales (Partial Take Profits) en múltiples etapas configurable.
*   **Ejecución Consciente del Bróker:** El bot consulta dinámicamente al bróker las reglas específicas del instrumento (stoplevel, lot_step, min/max_lot) para asegurar que todas las órdenes cumplan las normas y evitar rechazos.
*   **Persistencia de Estado:** El bot guarda su estado de gestión de trades (por ejemplo, qué beneficios parciales se han tomado) en un archivo, lo que permite detenerlo y reiniciarlo sin tomar decisiones duplicadas.
*   **Fiabilidad Lista para Producción:**
    *   **Registro Persistente (Logging):** Todas las acciones y decisiones se registran en un archivo rotatorio para facilitar la depuración y auditoría.
    *   **Heartbeat Bidireccional:** Un mecanismo que permite al Asesor Experto (EA) de MT4 y al script de Python monitorearse mutuamente. Si el script de Python falla, el EA puede cerrar de forma segura todas las operaciones abiertas.
    *   **Verificaciones de Integridad de Datos:** El sistema detecta y maneja automáticamente datos corruptos (velas incorrectas) y advierte sobre posibles brechas en el flujo de datos.
*   **Configuración Impulsada:** Todos los parámetros clave, desde las rutas del bróker hasta la configuración de estrategias y reglas de riesgo, se gestionan en un archivo `config.py` central para un ajuste sencillo.

## Estructura del Proyecto

```

your_trading_bot/
├── main.py                  # Punto de entrada principal: Inicializa y ejecuta el bot.
├── config.py                # Todas las configuraciones, parámetros y credenciales.
├── logger_setup.py          # Configura el registro persistente en archivo y consola.
├── api/
│   └── dwx_client.py        # La biblioteca cliente para la comunicación con MT4.
├── event_handler.py         # Enruta eventos desde el cliente al TradeManager.
├── trade_manager.py         # El motor: Gestiona datos, estado, riesgo y ejecución.
├── utils/
│   └── risk_manager.py      # Contiene funciones puras de cálculo de riesgo.
└── strategies/
├── init.py
├── base_strategy.py     # Define la interfaz que deben seguir todas las estrategias.
└── sma_crossover.py     # Un archivo de estrategia autocontenida de ejemplo.

```

## Requisitos Previos

*   **Terminal de MetaTrader 4:** Debes tener un terminal de MT4 instalado.
*   **Python:** Python 3.8 o superior.
*   **Asesor Experto (EA) MQL4 requerido:** El Asesor Experto mejorado `DWX_server_MT4.mq4` de este repositorio debe colocarse en el directorio `MQL4/Experts` de tu terminal MT4.
*   **Bibliotecas de Python:**
    ```bash
    pip install pandas
    ```

## Configuración e Instalación

### Paso 1: Configurar MetaTrader 4

1.  **Instalar el Asesor Experto:**
    *   Abre tu terminal MT4, ve a `Archivo -> Abrir Carpeta de Datos`.
    *   Navega a la carpeta `MQL4/Experts` y copia el archivo `DWX_server_MT4.mq4` dentro de ella.
    *   En MT4, haz clic derecho en "Asesores Expertos" en el Navegador y selecciona "Actualizar".

2.  **Habilitar AutoTrading y DLLs:**
    *   En la barra de herramientas de MT4, haz clic en el botón "AutoTrading" hasta que se ponga verde.
    *   Ve a `Herramientas -> Opciones` y selecciona la pestaña `Asesores Expertos`.
    *   Marca "Permitir operaciones automatizadas".
    *   Marca "Permitir importación de DLL".

3.  **Adjuntar y Configurar el EA:**
    *   Abre un gráfico para el símbolo y el marco de tiempo que deseas operar (por ejemplo, EURUSD, M15).
    *   Arrastra el EA `DWX_server_MT4` desde el Navegador hasta el gráfico.
    *   En la pestaña `Entradas` de las propiedades del EA, es fundamental actualizar la configuración predeterminada:
        *   `MaximumLotSize`: Establece un valor alto, como `100.0`, para darle el control a Python.
        *   `MaximumOrders`: Establece un valor alto, como `20`.
        *   `SlippagePoints`: Auméntalo a `30` o `50` para instrumentos volátiles.
        *   `MILLISECOND_TIMER`: Auméntalo a `500` para reducir la carga de la CPU.
    *   Asegúrate de que la opción "Permitir operaciones en vivo" esté marcada en la pestaña `Común`. Deberías ver una carita sonriente en el gráfico.

### Paso 2: Configurar el Proyecto de Python

1.  **Clonar el Repositorio:**
    ```bash
    git clone https://github.com/Anu-bhav/mt4_trading_bot
    cd mt4_trading_bot
    ```

2.  **Editar `config.py`:**
    *   `METATRADER_DIR_PATH`: Establece esta ruta a la ruta completa de la Carpeta de Datos de tu MT4.
    *   **Configuración de la Estrategia:** Ajusta `STRATEGY_SYMBOL`, `STRATEGY_TIMEFRAME` y los parámetros en `STRATEGY_PARAMS`.
    *   **Configuración de Riesgo:** Configura tus reglas deseadas en el diccionario `RISK_CONFIG`.

## Cómo Ejecutar el Bot

Una vez que tanto MT4 como Python estén configurados, simplemente ejecuta el script principal desde tu terminal:

```bash
python main.py
```

Toda la salida se imprimirá en la consola y se guardará simultáneamente en trading_bot.log para un registro permanente.

Cómo Crear una Nueva Estrategia

Gracias a la fábrica de estrategias dinámica, añadir nuevas estrategias es increíblemente sencillo:

1. Definir Parámetros en config.py: Añade una nueva clave y un diccionario para tu estrategia dentro de STRATEGY_PARAMS.
   ```python
   'mi_nueva_estrategia': { 'lookback': 50, 'threshold': 3.14 }
   ```
2. Crear el Archivo de la Estrategia:
   · En la carpeta strategies/, crea un archivo llamado mi_nueva_estrategia.py.
   · Dentro, crea una clase llamada MiNuevaEstrategia que herede de BaseStrategy.
   · Implementa los métodos __init__, get_signal y reset.
3. Activar la Estrategia en main.py:
   · Cambia una sola línea en main.py:
   ```python
   strategy_name_to_run = "mi_nueva_estrategia"
   ```
   ¡Eso es todo! La fábrica se encarga del resto.

Agradecimientos

La capa de comunicación central de este proyecto se basa en el trabajo fundamental del equipo de Darwinex Labs y su proyecto de código abierto dwxconnect.

El Asesor Experto DWX_server_MT4.mq4 y el cliente de Python (api/dwx_client.py) están basados en su biblioteca original. Los hemos extendido para incluir datos más ricos, cierres controlados (a través de un método stop()), y un mecanismo de heartbeat bidireccional para una fiabilidad mejorada.

Estamos inmensamente agradecidos por su decisión de liberar estas herramientas esenciales como código abierto.

El repositorio original se puede encontrar en: darwinex/dwxconnect en GitHub.

```