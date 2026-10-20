# ecosistema-automatizacion-ia
Ecosistema de Automatización de Soporte con IA
# Sistema de Triage Inteligente de Tickets con IA y Validación Humana (HITL)
Este proyecto implementa un ecosistema de automatización empresarial para la gestión, análisis y respuesta de tickets de soporte técnico, integrando Inteligencia Artificial mediante modelos de lenguaje masivos (OpenAI) y asegurando control de calidad mediante validación humana (Human-in-the-loop).
##  Arquitectura del Sistema y Tecnologías
El flujo de datos está diseñado bajo una arquitectura resiliencia y desacoplada, dividida en dos etapas lógicas para mitigar riesgos operativos (como el envío masivo de correos erróneos o "efecto metralleta").
- **Base de Datos Relacional (Airtable):** Estructura normalizada compuesta por las tablas `Clientes` y `Tickets`, vinculadas mediante relaciones de registros para asegurar la integridad de los datos.
- **Motor de Orquestación (Make.com):** Diseñado con filtros de control estrictos y triggers inteligentes programados para optimizar el consumo de operaciones de API.
- **Cerebro del Sistema (OpenAI - gpt-4o-mini):** Utiliza Prompts Estructurados con System-Role restrictivos para forzar salidas estrictamente formateadas en objetos JSON estructurados.
##  Resiliencia y Manejo de Errores
Para cumplir con los estándares de robustez y tolerancia a fallos exigidos por el negocio, el sistema cuenta con mecanismos automáticos:
- **Directiva de Control de Flujo (Resume Error Handler):** El nodo de OpenAI está conectado directamente a un módulo de resiliencia nombrado como `[Resiliencia] Mitigación de Fallo IA`. 
- **Estabilización Autónoma:** Si la API de OpenAI sufre una caída, se queda sin saldo o recibe un ticket con datos incompletos (campos vacíos), esta directiva intercepta el código de error. En lugar de colgar el escenario en rojo, permite que el flujo se estabilice de forma autónoma, ignore el fallo crítico de comunicación y finalice la corrida de manera segura sin interrumpir la operación del servidor.
##  Flujo de Trabajo Detallado
### Escenario 1: Captura y Procesamiento con IA
1. **Trigger Inteligente:** El escenario se dispara de forma programada buscando tickets en Airtable bajo la fórmula restrictiva `{Estado} = 'Pendiente'`. Cuenta con un límite estricto de procesamiento (Limit: 7) para evitar sobrecargas.
2. **Control de Concurrencia:** Apenas el registro ingresa al flujo, se actualiza inmediatamente su estado a `Procesado por IA`. Esto actúa como un bloqueo de exclusión mutua que previene bucles infinitos en executions concurrentes.
3. **Inferencia e Ingeniería de Prompts:** Los datos del ticket se envían a OpenAI bajo el modo `JSON Object`. El modelo clasifica el ticket (Categoría y Prioridad), genera un resumen ejecutivo y redacta una propuesta de respuesta empática en español.
4. **Traducción de Estructuras (JSON Parser):** El texto JSON plano se parsea mediante una estructura de datos estricta para deserializar el objeto y convertirlo en variables independientes.
5. **Persistencia del Análisis:** Las variables extraídas se guardan en el registro correspondiente del ticket en Airtable, moviendo el ciclo de vida del elemento al estado estricto de `Esperando Aprobación`.
### Escenario 2: Validación Humana (HITL) y Despacho de Correo
1. **Trigger de Revisión:** Un segundo escenario independiente monitorea de forma optimizada la base de datos buscando registros con la fórmula exacta `{Estado} = 'Aprobado por humano...'`.
2. **Validación Humana (Human-in-the-loop):** El flujo solo avanza si un operador de soporte revisó la respuesta propuesta por la IA en Airtable y cambió manualmente el estado para dar luz verde.
3. **Conversión y Sanitización de Datos:** Se extrae dinámicamente la información del cliente. Al ser un campo relacional (Array), se aplica la función `get(variable; 1)` en Make para limpiar la dirección de correo y evitar fallos de formato.
4. **Despacho del Canal de Salida (Gmail):** Se envía de forma automática el correo técnico estructurado al destinatario final utilizando la respuesta pre-aprobada.
5. **Cierre de Ciclo:** El nodo final actualiza el registro en Airtable al estado definitivo de `Enviado`, concluyendo con éxito y de manera segura el ciclo de vida del ticket de forma resiliente.


[Link en modo lectura a la Base de Datos de Airtable] 
https://airtable.com/appfSZTJXgRmBUQpa/shrVMTRJOJkDxay2L

[Link al Video Demostrativo del Proyecto]
https://drive.google.com/file/d/1YIZAZD_5X-3p36M_GfXNDFL6DruiFKiH/view?usp=sharing
