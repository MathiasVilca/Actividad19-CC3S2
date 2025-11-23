## Bloque 1
**¿Por qué microservicios?**
Al inicio, los sistemas eran monolíticos (es decir, todos los servicios/módulos estan embebidos en una sola app), pero debido a ser más fragiles a fallos y al ser difíciles de mantener y escalar, se paso a SOA. SOA dividió ligeramente los servicios, minimizando dependencias (Acoplamiento débil), pero todavía necesitaban comunicación y compartian fuertemente ciertos recursos.  Finalmente, se logro separar los servicios en microservicios, los cuales son independientes en despliegue y datos.  
Ejemplos:
- Un E-Commerce el cual tiene picos estacionales: El escalamiento de una app monolítica requiere duplicación completa, lo cual es costoso e ineficiente.
- Un SaaS multi-inquilino: si un error ocurre y uno de los módulos falla, entonces toda la app se detiene, afectando a todos los clientes sin importar que servicio hayan estado usando.

**Críticas al monolito**
- **Cadencia de despliegue reducida:** Un solo cambio en un módulo específico requiere el redespliegue de toda la aplicación, el cual tiene riesgo de romper algo en otro módulo.
- **Acoplamiento:** Si se requiere que un modulo use más recursos, se tiene que duplicar también el resto de módulos (que probablemente no necesiten tantos recursos), desperdiciando infraestructura.

**Popularidad y beneficios**
Se adoptaron justamente porque la independencia de microservicios: 
- Aisla los fallos del resto, permitiendo que estos sigan en funcionamiento.
- Permite asignar más o menos recursos a aquellos servicios que los necesitaran o sobraran.
- Permiten el trabajo de distintos equipos a la vez, sin que uno se tropezara con el otro a la hora de hacer cambios a pesar de estar trabajando en módulos distintos.

**Desventajas y retos**
Desafíos:
- La comunicacion entre servicios se hace por red, lo cual puede fallar o ser lento o inseguro.
- El orquestamiento de todos los microservicios puede ser complejo.
- Es más dificil mantener consistencia de datos de módulos diferentes
- El probar errores de interacción entre servicios puede ser complicado.  

Mitigaciones:
- OpenAPI y contratos: Mantienen consistencia a la hora de la comunicación entre servicios.
- Patrones de sagas: Secuencias de eventos las cuales deshacen cambios en caso fallase la actualizacion de datos en alguno de los servicios.
- Trazabilidad (Jaeger): Permite ver que camino toma una petición al llegar a la app y ver en que parte falla.
- Para orquestación, uso de herramientos de automatización como kubernetes.

**Principios de diseño**
- **DDD (Diseño dirigido por dominios):** Al usar limites contextuales, se define hasta donde llega un microservicio y empieza otro, como dividirlos por su funcionalidad en la app (Envíos, Facturación, ..).
- **DRY (Don't Repeat Yourself):** DRY busca no repetir código, pero el compartir librerías entre servicios puede generar acoplamiento, asi que se prefiere tener duplicacion controlada.
- **Criterios de tamaño:** Tamaño de un servicio se mide en resposabilidad, no en cuantas tablas o lineas de codigo tiene. Debe encapsular una funcionalidad completa y contener todo lo estrictamente necesario para cumplirla.

## Bloque 2
**Porque no usar `:latest`?**
- `:latest` es bastante ambiguo ya que no distingue que version es, hoy puede ser la `1.1.0` y mañana la `2.0`, cosa que puede malograr el entorno en caso este no se compatible con la nueva version
- SemVer (o Versionado Semántico) soluciona esto al dar un identificador único a cada versión, dependiendo de la escala de los cambios hechos respecto a la otra (primer número se cambia si es un cambio mayor, el segundo si es menor, y el tercero si es un parche). Permite reproducibilidad al fijar una version con la cual se sabe el entorno funciona