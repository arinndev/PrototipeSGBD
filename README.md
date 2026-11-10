# Gestor de Orientación (Python + MongoDB)

## Instalación
1. Instala MongoDB Community y déjalo corriendo (localhost:27017).
2. `pip install -r requirements.txt`
3. (Opcional) `export ESCUELA="Nombre de tu escuela"` para el encabezado del PDF.
   En Windows: `set ESCUELA=Nombre de tu escuela`

## Uso
`python gestor.py` abre la consola de macros. Macros:

| Macro | Qué hace |
|---|---|
| `importar archivo.xls` | Lee la hoja, la guarda como JSON en `datos_json/` y la carga a MongoDB (carga masiva; reimportar no duplica) |
| `registrar` | Captura una canalización: al escribir el nº de control (con sugerencias) se rellenan los datos del alumno |
| `buscar texto` | Busca por nº de control o apellido paterno |
| `filtrar semestre=4 turno=matutino desde=2025-01-01` | Filtra canalizaciones por cualquier campo |
| `reporte 0123456789012 [fecha]` | PDF de la última canalización (o la de esa fecha) en `reportes/` |
| `lote especialidad=Programación grupo=A` | Un PDF con una hoja por cada canalización filtrada |

Se pueden encadenar con `;`:
`python gestor.py "importar alumnos.xls; lote semestre=4"`

## Columnas del Excel
Número de control, Fecha de atención, Apellido paterno, Apellido materno, Nombre, Generación,
Especialidad, Semestre, Grupo, Grado, Turno, Persona que reporta, Motivo,
Nombre de padre o madre, Teléfono, Medida de modificación de conducta,
Seguimiento de los documentos. (Acentos y mayúsculas no importan.)
"Canalizaciones anteriores" se calcula solo.

## Rendimiento
Índices en MongoDB (nº de control, apellidos, generación/especialidad/semestre/grupo/turno),
escrituras en bloque y copia del dato del alumno dentro de cada canalización
(los filtros y PDFs no necesitan cruzar colecciones).

## Interfaz web
`python app.py` y abre http://localhost:5000 (Registrar, Consultar y filtrar, Importar).
Usa la misma base y los mismos PDF que la consola. Para probar sin MongoDB: `MONGO_URI=mock python app.py`
(requiere `pip install mongomock`; los datos no se conservan).

## Inicio de sesión
- La primera vez que ejecutes `python app.py` te pedirá crear el administrador en la consola.
- Más usuarios: `python app.py crear-usuario` (rol `admin` u `orientador`). Repetir un usuario cambia su contraseña.
- Contraseñas guardadas con hash; sesión de 8 horas; 5 intentos fallidos bloquean la IP 5 minutos.
- Cada canalización guarda quién la registró (`registrado_por`).
- Para entrar desde otras computadoras de la red: `HOST=0.0.0.0 python app.py` (Windows: `set HOST=0.0.0.0`).
  La conexión es HTTP sin cifrar: úsalo solo en la red interna de la escuela, o pon un proxy con HTTPS (Caddy/Nginx).
- El archivo `.secret_key` firma las sesiones: no lo compartas ni lo subas a ningún repositorio.
