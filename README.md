# Assessment
Propuesta de solución al ejercicio de arquitectura CCS


# Contexto

Contexto: CCS es la Compañía Colombiana de Seguimiento de Vehículos. CCS se encarga del monitoreo y seguimiento de vehículos de carga. CCS instala sensores en camiones de carga, de forma que es posible en todo momento conocer la localización, velocidad, dirección, estado de la carga, temperatura de la carga, detenciones planeadas y no planeadas, y accidentes que pueda tener cada camión.

# Contenedores

![image](https://github.com/jgutierrez-pragmatico/Assessment/assets/117243510/e835a5a0-04c6-4a42-9b6c-630f20ca0552)


La solución consiste en los siguientes contenedores:
- **[X]Rastreo:** Este contenedor se encargará de la localización en tiempo real de los vehículos. Recibirá constantemente datos de los sensores de los vehículos y actualizará su estado en la base de datos.
- **[X]Monitoreo:** Este contenedor será responsable de monitorear el estado de los vehículos. Tomará datos de la base de datos actualizada por el contenedor de Rastreo y los procesará para detectar cualquier irregularidad o emergencia.
- **[X]CRM (Customer Relationship Management):** Este contenedor manejará la relación con el cliente. Será el responsable de manejar contratos, solicitudes de servicio y gestión de la relación con el cliente.
- **[X]Notificaciones:** Este contenedor manejará todas las notificaciones a los clientes, autoridades y organismos de socorro en caso de una emergencia detectada por el contenedor de Monitoreo.
- **[X]Analítica:** Este contenedor procesará todos los datos históricos y en tiempo real para generar estadísticas y análisis útiles para el cliente. Esto incluirá estadísticas de localización, distancia de los recorridos, tiempo de movimiento, etc.
- **[X]Financiero:** Este contenedor se encargará de la facturación y pagos. Generará facturas mensuales, recibirá pagos y mantendrá un registro de todas las transacciones financieras.
- **[X]Usuarios:** Este contenedor manejará la autenticación y autorización de los usuarios. Mantendrá un registro de todos los usuarios y sus roles, y garantizará que solo los usuarios autorizados puedan acceder a los sistemas y realizar acciones según sus roles.
- **[X]IoT (Internet of Things):** Este contenedor será la interfaz entre los sensores instalados en los vehículos y la base de datos. Recogerá datos de los sensores y los enviará a la base de datos para su procesamiento posterior.

# Componentes

![image](https://github.com/jgutierrez-pragmatico/Assessment/assets/117243510/fbd91a79-c3aa-4871-b283-1ac378ea817f)


# Descripción de la solución

### Eventos IoT

- **Rastreo de la ubicación del vehículo:** Los dispositivos IoT en los vehículos pueden enviar datos de ubicación, los datos del sensor, a AWS IoT Core. Estos datos de ubicación pueden incluir información como la latitud, la longitud y el tiempo.
- **Procesamiento de los datos de ubicación:** Una regla en AWS IoT Core toma estos datos de ubicación y los envia al Amazon Location Service. Esto se mediante una acción que interactúa con Amazon Location Service.
- **Visualización en tiempo real:** Con los datos de ubicación en Amazon Location Service, se pueden visualizar en tiempo real la ubicacion del vehiculo en un mapa. (Amazon Location Service ofrece capacidades de mapas interactivos que te permiten personalizar el mapa para satisfacer tus necesidades.)

### Monitoreo IoT CQRS
- **Command Lambda:** Cuando un evento (por ejemplo, un cambio de ubicación o estado del vehículo) llega a la cola SQS, una función Lambda se activa. La lambda "Command Lambda". se encargara de extraer el mensaje de la cola, lo procesa y luego persiste los datos en una base de datos DynamoDB. Esta base de datos almacena el estado actual de los vehículos y se actualiza continuamente a medida que llegan nuevos eventos.

- **DynamoDB Streams:** Cada vez que se modifica un ítem en DynamoDB (por ejemplo, cuando Command Lambda persiste un nuevo evento), un registro de la modificación se envía a un DynamoDB Stream asociado a la tabla. Este registro incluye información sobre el ítem modificado y el tipo de modificación (por ejemplo, si fue una inserción, actualización o eliminación).

- **Query Lambda:** Una segunda función Lambda, "Query Lambda". Esta Lambda se activa cada vez que aparece un nuevo registro en el Stream. Toma el registro, lo procesa y luego persiste los datos procesados en otra base de datos DynamoDB, "DynamoDB Consultas". Esta base de datos está optimizada para consultas y se utiliza para generar dashboards de analítica.

##### CQRS
El patrón CQRS (Command Query Responsibility Segregation) sugiere dividir una aplicación en dos partes: la primera se encarga de las actualizaciones de datos (Commands), y la segunda se encarga de las consultas de datos (Queries).

### Anal[itica
- **AppSync:** ppSync proporciona una manera de gestionar APIs de GraphQL. Conecta el API de GraphQL a la base de datos DynamoDB Consultas. AppSync mapea directamente los esquemas de GraphQL y las tablas de DynamoDB.
- **Esquema GraphQL:** El API de GraphQL prepara los datos para los informes de análisis. Por ejemplo, distancia total recorrida por cada vehículo en un período de tiempo.
- **Integración con React:** Con Apollo Client para React se intala el cliente de la API de GraphQL. Apollo Client facilita la ejecución de consultas y mutaciones de GraphQL, la gestión de datos en caché y la actualización de la interfaz de usuario de React.
- **Generación de informes:** Desde React las consultas GraphQL se pueden renderizar en tablas, gráficos, mapas o cualquier otra visualización.

# Estilo de arquitectura 
### Serverless
**- Ventajas:**

1 - **Sin administración de servidores:** No es necesario provisionar, mantener o administrar servidores. AWS lo hace por nosotros.

2 - **Escalabilidad automática:** La infraestructura serverless se escala automáticamente en función de la demanda.

3 - **Costo eficiente:** Con el modelo de precios de pago por uso, sólo pagas por el tiempo de ejecución de tus funciones. No pagas por la infraestructura ociosa.

4 - **Alta disponibilidad:** Los proveedores de servicios en la nube ofrecen alta disponibilidad y recuperación ante desastres con sus servicios serverless.


# Observabilidad 

Desacople de componentes: En una arquitectura serverless basada en eventos en AWS, utilizaremos los servicios:

- Con **CloudWatch** supervisamos  métricas de rendimiento de tus funciones Lambda, colas SQS y DynamoDB. crear alarmas que se activan si alguna de estas métricas supera un umbral que podría indicar un problema. (umbrales de procesamiento o memoria)

- Con usar **X-Ray** rastreamos las peticiones que ingresan hasta que se completan, pasando por todas las funciones Lambda, SQS y DynamoDB que se utilizan para procesar la solicitud. Esto nos facilita identificar cuellos de botella y problemas de rendimiento.

- los logs se enviaran desde las funciones Lambda a **CloudWatch Logs.** Estos logs pueden incluir información sobre eventos de transacciones, errores y cualquier otra cosa que pueda ser útil para entender lo que está pasando en tu sistema. con el fin de implementar modelos predictivos

