📘 Guía de Documentación y Comprensión - Proyecto AXELONGO
Esta guía detalla la arquitectura, lógica y estructura del ecosistema web de AXELONGO, diseñado para ofrecer una experiencia de usuario dinámica y un seguimiento de métricas en tiempo real.

🚀 1. Visión General (Overview)
¿Qué hace el sistema?
Es una plataforma web institucional de marca personal/agencia que integra:

Landing Page de Alto Impacto: Presentación de servicios (Marketing, Publicidad, Diseño Web).

Sistema de Seguimiento (Tracking): Rastreo automático de interacciones sin necesidad de una base de datos propia.

Panel de Análisis Privado (Dashboard): Visualización de métricas de clics y reportes avanzados de Looker Studio.

¿Para quién es?
Para el equipo de AXELONGO, permitiendo monitorear qué servicios y redes sociales generan más interés.

Flujo Principal
El usuario navega por index.html.

Cada clic en botones críticos o redes sociales dispara un evento invisible.

El evento se guarda en un contador externo (CounterAPI) y se envía un webhook a n8n.

El administrador accede a dashboard.html para ver los resultados en tiempo real.

🧩 2. Arquitectura
El proyecto sigue un modelo de Frontend Estático con Microservicios Externos:

Frontend: HTML5 / Tailwind CSS / JavaScript Vanilla.

Métricas Rápidas: CounterAPI (Almacenamiento volátil de conteos).

Procesamiento de Datos: n8n (Webhook para recibir leads e interacciones).

Visualización Avanzada: Google Looker Studio (Reportes de tráfico de GA4).

Diagrama de Arquitectura
Fragmento de código
graph TD
    subgraph Cliente
        A[index.html] -->|Clic / data-tracker| B[JS Tracking Engine]
        C[dashboard.html] -->|Fetch GET| D[JS Dashboard Engine]
    end

    subgraph Servicios Externos
        B -->|POST /up| E((CounterAPI))
        B -->|POST Webhook| F{n8n Webhook}
        D -->|GET Metrics| E
        G[Looker Studio] -.->|Iframe| C
    end

    subgraph Procesamiento
        F --> H[Google Sheets / CRM]
        F --> I[Email Notifications]
    end
🔄 3. Flujo de la Lógica (Métricas e Interacciones)
Esta es la parte vital para entender cómo se conectan los archivos.

Diagrama de Secuencia de Interacción
Fragmento de código
sequenceDiagram
    participant U as Usuario
    participant B as Navegador (index.html)
    participant C as CounterAPI
    participant N as n8n Webhook

    U->>B: Hace clic en Red Social / Botón
    B->>B: Detecta data-tracker
    par Proceso Paralelo
        B->>C: fetch(.../up) -> Incrementa +1
        B->>N: POST payload (JSON) -> Notifica evento
    end
    N-->>B: OK 200
    C-->>B: Nuevo conteo
A. Captura de la Interacción (index.html)
El archivo principal contiene un script al final que escucha todos los clics en el documento.

Si el elemento tiene el atributo data-tracker (ej: data-tracker="social_fb"), el script sabe que debe contar ese clic.

Doble Acción Simultánea:

Envía un fetch a counterapi.dev para sumar +1 al contador específico.

Envía un JSON completo al Webhook de n8n con detalles como: qué botón fue, en qué página está y la hora exacta.

B. Visualización en el Panel (dashboard.html)
Al cargar, el dashboard recorre una lista de claves (social_fb, cat_marketing, etc.).

Realiza peticiones GET a la API para obtener el valor actual.

Actualiza los números en la pantalla automáticamente cada 60 segundos.

📁 4. Estructura del Proyecto
index.html: La puerta de entrada. Contiene toda la estructura visual y el motor de tracking.

dashboard.html: Interfaz administrativa protegida (visual).

DOCUMENTACION.md: Manual técnico del sistema.

GUIA_ESTRUCTURA.txt: Documento técnico de referencia sobre la organización de archivos.

sistema_web/: Contiene assets (CSS, JS), recursos subidos (uploads) y lógica del constructor.

paginas/: Directorios con contenido específico de secciones exportadas.

⚙️ 5. Módulos y Componentes Críticos
1. El Tracker Global (index.html)
Ubicado al final de index.html, es el encargado de la "magia" del seguimiento.

JavaScript
document.addEventListener('click', function(e) {
    const target = e.target.closest('[data-tracker]');
    if (target) {
        const trackerKey = target.getAttribute('data-tracker');
        // Incrementa el contador en la nube
        fetch(`https://api.counterapi.dev/v1/axelongosite/${trackerKey}/up`);
    }
});
2. El Motor del Dashboard (dashboard.html)
Utiliza funciones asíncronas para traer datos de múltiples fuentes sin bloquear la página.

Función updateMetrics(): Itera sobre el array de métricas y actualiza el DOM de manera fluida.

Iframe de Looker Studio: Integra el reporte visual de Google Analytics 4 directamente en el panel.

3. Secuencia de Auto-Scroll (index.html)
Para mejorar el SEO y la retención, el sitio tiene una lógica de "clics fantasma" que activan las pestañas de servicios automáticamente a los 4, 6.5 y 9 segundos de la carga inicial.

🔑 6. Funciones Críticas
fetch(.../up) (Ubicación: index.html): Es la señal de "incremento". Sin esto, el contador no se mueve. Es vital que el nombre en data-tracker coincida exactamente con el que se quiere ver en el dashboard.

updateMetrics() (Ubicación: dashboard.html): Centraliza la actualización de la interfaz. Maneja errores de red (si la API falla, muestra un mensaje de error en rojo en lugar de romper la página).

📁 7. Guía Detallada de Archivos y Carpetas
Directorio Raíz (/)
index.html: Es el motor principal del sitio. Contiene el HTML de la landing page y el Tracker de Clics Global. Escucha eventos de usuario, gestiona el menú móvil y ejecuta la secuencia automática de clics en la sección de servicios.

dashboard.html: Interfaz de visualización de datos. No guarda información, solo la consulta realizando peticiones asíncronas a CounterAPI y embebiendo Looker Studio.

DOCUMENTACION.md: Manual técnico del sistema (este archivo).

GUIA_ESTRUCTURA.txt: Un mapa rápido de directorios para referencia inmediata del desarrollador.

Directorio sistema_web/
Esta carpeta contiene el núcleo técnico heredado del generador estático (WordPress).

index.html: Una página secundaria (sección Filosofía) que mantiene la consistencia visual del sitio.

assets/css/: Define la estética de los componentes del constructor Spectra y el tema Astra (ej. dashboard-app.css).

assets/plugins/: Almacena la lógica de terceros como AOS (animaciones scroll), Swiper (sliders) y Spectra (bloques dinámicos).

assets/themes/: Contiene el tema Astra, encargado de la estructura base, header, footer y tipografía global.

uploads/: El repositorio de medios. Aquí se encuentran todas las imágenes, el favicon y los logos oficiales.

Directorio paginas/
Contiene las subsecciones del sitio exportadas como archivos HTML independientes. Cada carpeta (ej: nosotros/, blog/) contiene un index.html propio, permitiendo que las URLs del sitio sean limpias (ej: axelongo.com/nosotros).

🚨 8. Casos Especiales / Errores
¿Qué pasa si falla CounterAPI? El sitio seguirá funcionando perfectamente. El usuario no notará nada, simplemente no se registrará la métrica en ese momento (el error es manejado de fondo con .catch()).

¿Qué pasa si el Webhook de n8n está offline? Los datos de formularios no llegarán al CRM o al Email, pero el frontend mostrará una alerta de "Error de conexión" para que el usuario sepa que algo falló.

Gestión de Cache: El dashboard usa un parámetro de tiempo ?t=${Date.now()} en las peticiones API para evitar que el navegador guarde datos viejos, garantizando que siempre se muestre el número en tiempo real.