# Documentación Scraper Urbania

## Índice
- [Documentación Scraper Urbania](#documentación-scraper-urbania)
  - [Índice](#índice)
  - [Arquitectura General](#arquitectura-general)
  - [Flujo de Trabajo del Motor de Scraping (main.py)](#flujo-de-trabajo-del-motor-de-scraping-mainpy)
  - [Componentes del Motor de Scraping](#componentes-del-motor-de-scraping)
  - [Flujo de la Interfaz de Usuario (app.py)](#flujo-de-la-interfaz-de-usuario-apppy)
  - [Integración entre Componentes](#integración-entre-componentes)
  - [Flujo de Procesamiento de Datos](#flujo-de-procesamiento-de-datos)
  - [Uso del Sistema](#uso-del-sistema)
    - [Modo CLI (main.py)](#modo-cli-mainpy)
    - [Modo Interfaz Gráfica (app.py)](#modo-interfaz-gráfica-apppy)
  - [Estructura de Archivos de Salida](#estructura-de-archivos-de-salida)
  - [Requisitos del Sistema](#requisitos-del-sistema)
    - [Navegador Requerido](#navegador-requerido)
    - [Dependencias Principales](#dependencias-principales)
    - [Instalación de Dependencias](#instalación-de-dependencias)
    - [Requisitos de Hardware](#requisitos-de-hardware)
  - [Manejo de Situaciones Especiales](#manejo-de-situaciones-especiales)
    - [Detección de Anti-Bot y CAPTCHA](#detección-de-anti-bot-y-captcha)
    - [Estrategias Anti-Detección](#estrategias-anti-detección)
  - [Guía para Desarrolladores](#guía-para-desarrolladores)
    - [Extender la Funcionalidad](#extender-la-funcionalidad)
    - [Mejores Prácticas de Desarrollo](#mejores-prácticas-de-desarrollo)
  - [Limitaciones y Consideraciones](#limitaciones-y-consideraciones)
    - [Éticas y Legales](#éticas-y-legales)
    - [Técnicas](#técnicas)
  - [Configuraciones Avanzadas](#configuraciones-avanzadas)
    - [Parámetros Ajustables](#parámetros-ajustables)
    - [Ejemplo: Configuración de Proxy](#ejemplo-configuración-de-proxy)
  - [Preguntas Frecuentes (FAQ)](#preguntas-frecuentes-faq)
    - [¿Cómo manejar bloqueos de Cloudflare?](#cómo-manejar-bloqueos-de-cloudflare)
    - [¿Cómo extender para otros sitios inmobiliarios?](#cómo-extender-para-otros-sitios-inmobiliarios)
    - [¿Es posible ejecutar en modo headless?](#es-posible-ejecutar-en-modo-headless)
  - [Historial de Versiones y Actualizaciones](#historial-de-versiones-y-actualizaciones)
    - [v1.0.0](#v100)
    - [v1.1.0](#v110)

Esta documentación describe el funcionamiento del scraper para el sitio web Urbania, compuesto por dos componentes principales:
1. `main.py`: El motor de scraping principal
2. `app.py`: La interfaz gráfica basada en WebView

## Arquitectura General

```mermaid
graph TB
    subgraph "Interfaz de Usuario"
        app[app.py - WebView]
    end
    
    subgraph "Motor de Scraping"
        main[main.py - Scraper Engine]
    end
    
    subgraph "Salidas"
        excel[Archivos Excel]
        debug[Archivos HTML Debug]
    end
    
    app -->|"Invoca funciones"| main
    main -->|"Extrae datos"| urbania[Sitio Web Urbania]
    main -->|"Guarda resultados"| excel
    main -->|"Guarda HTML"| debug
    app -->|"Navegación web"| urbania
```

## Flujo de Trabajo del Motor de Scraping (main.py)

```mermaid
flowchart TD
    start([Inicio]) --> configure[Configurar navegador]
    configure --> navigate[Navegar a URL]
    navigate --> wait[Esperar carga y simular comportamiento humano]
    wait --> extract_page1[Extraer datos página 1]
    
    extract_page1 --> count_pages[Determinar número total de páginas]
    count_pages --> more_pages{¿Más páginas?}
    
    more_pages -->|Sí| next_page[Navegar a siguiente página]
    next_page --> wait_page[Esperar carga de página]
    wait_page --> extract_page[Extraer datos de página]
    extract_page --> more_pages
    
    more_pages -->|No| filter[Filtrar duplicados]
    filter --> export[Exportar a Excel]
    export --> fin([Fin])
    
    subgraph "Proceso de extracción"
        extract[Extraer datos de tarjetas] --> process_cards[Procesar cada tarjeta]
        process_cards --> extract_price[Extraer precio en soles y USD]
        extract_price --> extract_maintenance[Extraer mantenimiento]
        extract_maintenance --> extract_location[Extraer ubicación]
        extract_location --> extract_address[Extraer dirección exacta]
        extract_address --> extract_attrs[Extraer atributos]
        extract_attrs --> extract_url[Extraer URL]
        extract_url --> validate[Validar información útil]
    end
    
    extract_page1 -.-> extract
    extract_page -.-> extract
```

## Componentes del Motor de Scraping

```mermaid
classDiagram
    class main {
        +extraer_datos_urbania(url)
        +extraer_precios_pagina(driver, soup)
        +exportar_a_excel(datos)
    }
    
    class ComponentesExtraccion {
        +extraer_precio(tarjeta) %% Ahora devuelve precio_texto, precio_soles y precio_usd
        +extraer_ubicacion(tarjeta)
        +extraer_descripcion(tarjeta)
        +extraer_mantenimiento(tarjeta)
        +extraer_atributos(tarjeta)
        +extraer_direccion_exacta(tarjeta)
        +extraer_url_publicacion(tarjeta)
    }
    
    class UtilsAvanzados {
        +entrada_tiene_informacion_util(entrada)
        +extraer_tarjetas_alternativo(soup)
        +guardar_html_debug(html, nombre)
    }
    
    class GestionSalida {
        +salida_limpia(signum, frame)
    }
    
    main --> ComponentesExtraccion : usa
    main --> UtilsAvanzados : usa
    main --> GestionSalida : usa
```

## Flujo de la Interfaz de Usuario (app.py)

```mermaid
sequenceDiagram
    participant Usuario
    participant WebView
    participant API
    participant Scraper
    participant Excel
    
    Usuario->>WebView: Navega a urbania.pe
    WebView->>API: Notifica cambio de URL
    API->>API: verificar_url()
    
    alt Es página de búsqueda
        API->>WebView: Mostrar barra de opciones
        Usuario->>WebView: Click en "Scrapear esta página"
        WebView->>API: scrapear_pagina_actual()
        API->>WebView: Mostrar overlay de carga
        API->>WebView: Obtener HTML
        API->>API: Procesar datos con BeautifulSoup
        API->>Excel: Guardar resultados
        API->>WebView: Ocultar overlay
        API->>WebView: Mostrar modal con resultado
        
        Usuario->>WebView: Click en "Scrapear todas las páginas"
        WebView->>API: scrapear_todas_paginas()
        API->>WebView: Mostrar overlay de carga
        API->>Scraper: extraer_datos_urbania(url_actual)
        Scraper-->>API: Devolver resultados
        API->>Excel: Guardar resultados
        API->>WebView: Ocultar overlay
        API->>WebView: Mostrar modal con resultado
    else No es página de búsqueda
        API->>WebView: Ocultar barra de opciones
    end
```

## Integración entre Componentes

```mermaid
graph TB
    subgraph "app.py"
        api[Clase API]
        webview[Ventana WebView]
        event_handlers[Manejadores de Eventos]
    end
    
    subgraph "main.py"
        extractor[Funciones de Extracción]
        navegador[Control de Navegador]
        exportador[Exportación de Datos]
    end
    
    api -->|"scrapear_todas_paginas()"| extractor
    api -->|"Actualiza estado UI"| webview
    webview -->|"on_loaded()"| event_handlers
    event_handlers -->|"Detectar URLs"| api
    api -->|"Lanzar thread de scraping"| thread[Thread de Scraping]
    thread -->|"Ejecutar"| extractor
    extractor -->|"Controla"| navegador
    extractor -->|"Genera datos"| exportador
```

## Flujo de Procesamiento de Datos

```mermaid
flowchart LR
    html[HTML página] --> soup[BeautifulSoup]
    soup --> tarjetas[Extraer tarjetas]
    tarjetas --> datos[Extraer datos]
    datos --> filtro[Filtrar duplicados]
    filtro --> validar[Validar información útil]
    validar --> dataframe[Crear DataFrame]
    dataframe --> excel[Exportar a Excel]
    
    subgraph "Datos extraídos"
        price_soles[Precio en soles]
        price_usd[Precio en USD]
        mantenimiento[Mantenimiento]
        location[Ubicación]
        address[Dirección exacta]
        description[Descripción]
        attributes[Atributos]
        url_prop[URL propiedad]
    end
    
    datos --> price_soles
    datos --> price_usd
    datos --> mantenimiento
    datos --> location
    datos --> address
    datos --> description
    datos --> attributes
    datos --> url_prop
```

## Uso del Sistema

### Modo CLI (main.py)
Para ejecutar el scraper en modo línea de comandos:
```
python main.py
```

El programa preguntará si desea ingresar una URL personalizada o usar la predeterminada.

### Modo Interfaz Gráfica (app.py)
Para iniciar la aplicación con interfaz gráfica:
```
python app.py
```

La aplicación abrirá una ventana de navegador donde puede:
1. Navegar normalmente por el sitio Urbania
2. Cuando esté en una página de resultados de búsqueda, aparecerá una barra de opciones
3. Seleccionar "Scrapear esta página" o "Scrapear todas las páginas"
4. Ver el progreso en tiempo real y el resultado final

## Estructura de Archivos de Salida

- **Resultados**: Guardados en la carpeta `resultados/`
  - Formato: Archivos Excel (.xlsx)
  - Nomenclatura: `precios_urbania_YYYYMMDD_HHMMSS.xlsx`
  
- **Archivos de depuración**: Guardados en la carpeta `debug/`
  - Formato: Archivos HTML
  - Nomenclatura: `pagina_debug_YYYYMMDD_HHMMSS.html`

## Requisitos del Sistema

### Navegador Requerido
- **Google Chrome**: El sistema requiere tener instalada una versión reciente de Google Chrome (preferiblemente versión 90 o superior)
- La aplicación utiliza ChromeDriver que se descarga automáticamente para coincidir con su versión de Chrome instalada

### Dependencias Principales
```mermaid
graph TD
    main[Scraper Principal] --> selenium[Selenium]
    main --> bs4[BeautifulSoup4]
    main --> pandas[Pandas]
    main --> uc[undetected_chromedriver]
    
    app[Interfaz Gráfica] --> pywebview[pywebview]
    app --> main
    
    selenium --> chrome[Chrome Browser]
    uc --> chrome
    
    chrome -->|"Requiere"| installed[Chrome instalado en sistema]
```

### Instalación de Dependencias
```
pip install selenium beautifulsoup4 pandas undetected-chromedriver pywebview webdriver-manager requests psutil
```

### Requisitos de Hardware
- Mínimo 4GB RAM
- Espacio en disco: 500MB para la aplicación y sus temporales
- Conexión a internet
- Navegador Google Chrome instalado en el sistema

## Manejo de Situaciones Especiales

### Detección de Anti-Bot y CAPTCHA

```mermaid
flowchart TD
    start[Navegación a Página] --> detected{Detectado como bot?}
    detected -->|No| continue[Continuar Scraping]
    detected -->|Sí| try_undetected[Usar undetected_chromedriver]
    try_undetected --> still_detected{Aún detectado?}
    still_detected -->|No| continue
    still_detected -->|Sí| human_intervention[Esperar Intervención Humana]
    human_intervention --> delay[Espera de 30s]
    delay --> resume[Intentar Continuar]
    resume --> save_partial[Guardar Datos Parciales]
```

### Estrategias Anti-Detección
1. **User Agents Aleatorios**: Rotación de agentes de usuario en cada sesión
2. **Simulación Humana**: 
   - Scroll aleatorio y pausado
   - Tiempos de espera variables
   - Movimiento de ratón simulado
3. **Manejo de WebDriver**: Ocultando señales que identifican automatización
4. **Extracción Resistente**: Múltiples formas de extraer cada dato para adaptarse a cambios del sitio

## Guía para Desarrolladores

### Extender la Funcionalidad

```mermaid
graph TB
    subgraph "Agregar Nuevo Extractor"
        definir[Definir Nuevo Campo] --> implementar[Crear Función extractora]
        implementar --> integrar[Integrar en extraer_precios_pagina]
        integrar --> modificar[Actualizar Diccionario resultado]
    end
    
    subgraph "Agregar Compatibilidad con Sitio"
        analizar[Analizar Estructura DOM] --> identificar[Identificar Patrones]
        identificar --> adaptar[Adaptar Selectores]
        adaptar --> probar[Probar Extracción]
    end
```

### Mejores Prácticas de Desarrollo
1. **Usar Selectores Flexibles**: Siempre proporcionar alternativas en caso de cambios en el sitio
2. **Manejar Excepciones**: Envolver cada extracción en bloques try-except
3. **Validación de Datos**: Verificar que los datos extraídos tienen sentido
4. **Logging Detallado**: Mantener un registro extensivo para depuración

## Limitaciones y Consideraciones

### Éticas y Legales
- Respetar los términos de uso del sitio web
- Limitar la tasa de solicitudes para no sobrecargar el servidor
- Uso exclusivamente para investigación y análisis de mercado

### Técnicas
- Dependencia en la estructura actual del DOM del sitio
- Posibilidad de ser bloqueado en caso de uso intensivo
- Algunos datos pueden no estar disponibles o cambiar de formato

## Configuraciones Avanzadas

### Parámetros Ajustables

```mermaid
graph LR
    config[Configuraciones] --> ua[User Agents]
    config --> delay[Tiempos de Espera]
    config --> scroll[Comportamiento de Scroll]
    config --> proxy[Configuración de Proxy]
```

### Ejemplo: Configuración de Proxy
```python
options = uc.ChromeOptions()
options.add_argument('--proxy-server=ip:puerto')
```

## Preguntas Frecuentes (FAQ)

### ¿Cómo manejar bloqueos de Cloudflare?
El sistema intenta evitarlos mediante técnicas anti-detección. En caso de bloqueo persistente:
- Utilizar una espera más larga entre solicitudes
- Considerar el uso de proxies rotativas
- En último caso, resolver manualmente el CAPTCHA

### ¿Cómo extender para otros sitios inmobiliarios?
1. Analizar la estructura DOM del nuevo sitio
2. Crear funciones de extracción específicas
3. Adaptar la lógica de navegación entre páginas
4. Actualizar la validación de datos

### ¿Es posible ejecutar en modo headless?
Sí, pero aumenta el riesgo de detección. Para activarlo:
```python
options.add_argument('--headless')
```
Sin embargo, para sitios con protección anti-bot avanzada, no se recomienda.

## Historial de Versiones y Actualizaciones

### v1.0.0
- Implementación inicial con soporte para Urbania
- Interfaz CLI y GUI
- Extracción de datos básicos de propiedades

### v1.1.0
- Mejoras en la detección de elementos
- Optimización de tiempos de carga
- Reducción de falsos positivos
