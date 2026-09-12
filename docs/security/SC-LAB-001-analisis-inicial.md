# SC-LAB-001 — Análisis inicial de seguridad

**Proyecto:** SecureCampus
**Curso:** Desarrollo Seguro
**Fecha:** 11/09/2026

**Integrantes:**
- Luis Antonio Tarango Regis
- Angelo Armando Tellez Enriquez
- Diego Miguel Hernández Fabela
- Ana Cecilia Rentería Tellez

---

## 4. Actividad guiada: consulta de perfiles

**Caso:** María inicia sesión con el perfil 125. Al observar la URL, cambia manualmente `/perfil/125` por `/perfil/126`. El sistema devuelve información de otro estudiante.

| Elemento | Respuesta del equipo | Justificación |
|---|---|---|
| Activo | Datos del perfil del estudiante (información personal y académica accesible en el perfil) | Es la información que el sistema expone a través de esa dirección; si se filtra, se compromete la privacidad de un alumno. |
| Amenaza | Un estudiante que accede a perfiles que no le pertenecen | No hace falta ser un atacante externo, cualquier usuario con una sesión válida puede solamente editar la URL, la amenaza está dentro del sistema, no fuera. |
| Vulnerabilidad | Falta de una validación de autorización en el servidor que valide que se tiene autorización para solicitar un recurso y acceder a él | El sistema usa el ID que viene en la URL para decidir de qué usuario va a devolver la información, sin comprobar que ese ID corresponde al usuario autenticado. |
| Ataque | Manipulación del identificador en la URL | Es la acción concreta que explota la debilidad; no requiere herramientas especiales, solo observar el patrón del recurso. |
| Impacto | Divulgación no autorizada de información personal de otro estudiante | La consecuencia directa si el ataque tiene éxito es la pérdida de confidencialidad, posible daño legal por la filtración de datos personales de terceros. |
| Riesgo | Alto. La forma de acceder a esta vulnerabilidad no es difícil y el impacto es de confidencialidad sobre datos sensibles | La probabilidad y el impacto son prioritarios y son significativos, por lo que amerita atención prioritaria. |
| Control | Validación de autorización en el servidor en cada solicitud. Verificar que el id del recurso pertenece al usuario de la sesión, o que su rol lo autoriza. | Ataca la vulnerabilidad, el servidor debe validar, y debería definirse desde el diseño. |

---

## 5. Reto por equipo

Se analizan cuatro escenarios de SecureCampus: calificaciones, documentos y autenticación (obligatorios), y roles/permisos (elegido por el equipo).

| Escenario | Activo | Amenaza | Vulnerabilidad | Ataque | Impacto | Control |
|---|---|---|---|---|---|---|
| 1. Calificaciones | Información académica del estudiante (calificaciones) | Un estudiante que intenta ver las calificaciones de otro estudiante, o un profesor que intenta capturar las calificaciones de un grupo que no es suyo | Falta de verificación de que el id del alumno o del grupo en la petición corresponde al usuario autenticado | Cambiar el id en la URL o petición para ver notas de otro estudiante, o para acceder a un grupo no asignado | Divulgación de calificaciones ajenas, o alteración no autorizada de las calificaciones | Autorización en servidor por rol y pertenencia (verificar que el estudiante es dueño del registro, o que el profesor tiene asignado ese grupo), registrar logs de cambio de calificaciones |
| 2. Documentos | Documento de carga académica | Usuario que intenta descargar un documento que no le pertenece | Enlaces a descarga de documentos sin control de acceso | Adivinar un identificador de documento para descargarlo | Exposición de información académica en el documento | Verificar propiedad del archivo antes de servir el archivo, usar identificadores que no sean sencillos |
| 3. Autenticación | Credenciales y la sesión del usuario | Atacante que intenta entrar probando contraseñas seguidas, o que intenta reutilizar una sesión válida | Falta de límite de intentos al iniciar sesión, o tokens de sesión predecibles o sin expiración adecuada | Probar muchas contraseñas seguidas hasta acertar, o reutilizar un token de sesión no expirado | El atacante obtiene todos los permisos del usuario comprometido | Límite de intentos al iniciar sesión, y tokens de sesión con expiración e invalidación al cerrar sesión |
| 4. Roles/Permisos | Roles y permisos de los usuarios | Un Profesor que intenta obtener o ejercer capacidades reservadas a un rol superior de jefe | Falta de verificación del rol en el servidor antes de ejecutar una operación sensible, por ejemplo, dar de alta o baja a profesores | Enviar directamente la petición de una operación privilegiada para modificar su rol o permisos | El atacante obtiene capacidades administrativas, lo que puede derivar en acceso a todos los módulos protegidos | Verificación de rol/permiso en el servidor en cada operación sensible |

---

## 6. Preguntas de reflexión

**1. ¿Una amenaza y una vulnerabilidad son lo mismo? Explica con un ejemplo de SecureCampus.**

No. La amenaza es la situación que podría causar daño y la vulnerabilidad es la debilidad concreta del sistema que hace posible ese daño. Por ejemplo, un usuario autenticado que decide probar IDs ajenos, o un Profesor que intenta ejercer una acción de jefe de carrera son amenazas, la vulnerabilidad sería la falta de validación de autorización en el servidor.

**2. ¿Puede existir una vulnerabilidad, aunque todavía nadie la haya explotado?**

Sí. La vulnerabilidad es parte del sistema, una debilidad de diseño o implementación.

**3. ¿Un usuario autenticado está automáticamente autorizado para cualquier recurso?**

No. Autenticación responde "¿quién eres?"; autorización responde "¿qué puedes hacer sobre este recurso específico?".

**4. ¿Qué control de los propuestos debería definirse desde requisitos o diseño? ¿Por qué?**

La verificación de autorización en el servidor por rol y por pertenencia del recurso, ya que se aplica a los cuatro escenarios. Debe nacer desde el diseño porque si no se define desde el inicio como parte del modelo de datos y de la arquitectura no se puede parchar después sin reescribir buena parte de la lógica de acceso.

**5. ¿Qué activo consideran más crítico y por qué?**

La asignación de roles y permisos. Es el mecanismo que decide quién puede acceder a todos los demás activos. Si este activo se compromete, los controles diseñados para proteger todo lo demás dejan de ser confiables, porque un atacante con rol elevado los evade por completo. Es el activo del cual dependen todos los demás.
