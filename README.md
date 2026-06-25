🏢 Portal de Estado de Cuenta — Conjunto Mirador de Estambul
¿Qué es esto?
Este es el sistema de consulta de estado de cuenta del Conjunto Mirador de Estambul. Permite que cada residente consulte desde su celular o computador su estado de pago, saldo pendiente e historial mensual, usando únicamente su bloque, número de apartamento y una contraseña personal.


🗂️ Componentes del sistema
El sistema tiene tres partes que trabajan juntas:

Google Sheets  →  Apps Script  →  Portal Web (Netlify)
  (los datos)      (el puente)     (lo que ven los residentes)
1. Google Sheets — La base de datos
Es la hoja de cálculo donde el administrador ingresa y actualiza los datos mes a mes. Funciona como una base de datos sencilla que cualquier persona puede editar sin conocimientos técnicos.
2. Google Apps Script — El puente
Es un pequeño programa alojado en Google que lee los datos del Sheets y los entrega al portal en un formato que este puede leer. Sin este puente, Google bloquearía el acceso directo a los datos por razones de seguridad.
3. Portal Web (archivo index.html) — La interfaz
Es el archivo HTML que los residentes abren desde el navegador. Contiene toda la lógica visual y de validación. Está alojado en Netlify y tiene una URL fija que nunca cambia.


📋 Estructura del Google Sheets
La hoja debe llamarse exactamente: Datos_Junio_2026 (cambiando el mes cada vez).

Las columnas deben estar en este orden y con estos nombres exactos:

Columna
Nombre exacto
Descripción
A
Bloque
Ej: Bloque A, Bloque B, Bloque C, Bloque D
B
Apartamento
Ej: A-0101, B-0204
C
Nombre del residente
Nombre completo
D
Cuota mensual ($)
Valor numérico. Ej: 185000
E
Saldo pendiente ($)
Valor numérico. Ej: 285000 o 0
F
Estado
Exactamente: Al día, En mora o Parcial
G
Contraseña
Clave personal del residente
H en adelante
Enero, Febrero, Marzo...
Valor numérico pagado ese mes. Ej: 185000, 0, 125000

⚠️ Reglas importantes del Sheets
El Estado debe escribirse exactamente: Al día / En mora / Parcial
Los meses deben escribirse en valor numérico (no "Pagado" ni "Pendiente"):
185000 = pagó la cuota exacta
200000 = pagó más de la cuota
125000 = pagó parcialmente
0 = no pagó
La columna Contraseña debe tener una clave para cada apartamento


🔐 ¿Cómo funciona el login?
Cada residente ingresa al portal con tres datos:

Bloque — selecciona de la lista (A, B, C o D)
Número de apartamento — solo el número, sin la letra (ej: 101)
Contraseña — la clave personal asignada por el administrador

El portal busca en los datos del Sheets si existe ese bloque + apartamento, y luego verifica que la contraseña coincida. Si todo es correcto, muestra únicamente la información de ese apartamento. El residente no puede ver datos de otros.


🎨 ¿Qué ve el residente al ingresar?
Una vez autenticado, el residente ve:

Nombre del residente registrado
Estado del apartamento (Al día / En mora / Parcial)
Cuota mensual establecida
Saldo pendiente total
Historial mensual con código de colores:
🟢 Verde — Pagado (muestra el monto pagado)
🟡 Amarillo — Parcial (muestra el monto pagado parcialmente)
🔴 Rojo — Pendiente (no pagó ese mes)


🔄 ¿Cómo actualizar los datos cada mes?
El trabajo mensual del administrador es muy sencillo:

Abrir el Google Sheets
Actualizar la columna de Saldo pendiente ($)
Actualizar la columna de Estado
Agregar una nueva columna con el mes (ej: Julio) y escribir el valor pagado por cada apartamento
Guardar — el portal se actualiza automáticamente en segundos

No es necesario tocar el portal ni Netlify. Solo el Sheets.
Cambio de mes
Cuando empiece un nuevo mes, puede renombrar la hoja a Datos_Julio_2026 y el portal mostrará automáticamente el nuevo mes en el encabezado. Si cambia el nombre de la hoja, debe actualizar el Apps Script (ver sección técnica).


🌐 URLs importantes
Recurso
URL
Portal para residentes
https://dulcet-brioche-70b024.netlify.app
Google Sheets (admin)
https://docs.google.com/spreadsheets/d/1A4c-wPJ0uJ4F3SosgTfBlEACN8cW3Km913-f5IEouO8
Apps Script
https://script.google.com → proyecto "Untitled project"
Netlify (admin)
https://app.netlify.com/projects/dulcet-brioche-70b024



⚙️ Parte técnica — Apps Script
El Apps Script es el puente entre el Sheets y el portal. Su código es:

function doGet() {
  const ss = SpreadsheetApp.openById('1A4c-wPJ0uJ4F3SosgTfBlEACN8cW3Km913-f5IEouO8');
  const hoja = ss.getSheetByName('Datos_Junio_2026');
  const datos = hoja.getDataRange().getValues();
  const headers = datos[0];
  const filas = datos.slice(1).map(fila => {
    let obj = {};
    headers.forEach((h, i) => { obj[h] = fila[i]; });
    return obj;
  });
  const output = ContentService.createTextOutput(JSON.stringify(filas));
  output.setMimeType(ContentService.MimeType.JSON);
  return output;
}
¿Cuándo hay que tocarlo?
Solo cuando cambie el nombre de la hoja del Sheets (ej: de Datos_Junio_2026 a Datos_Julio_2026). En ese caso:

Ir a script.google.com
Abrir el proyecto
Cambiar 'Datos_Junio_2026' por el nuevo nombre
Hacer clic en Deploy → Manage deployments → editar → New version → Deploy


🔧 Cómo actualizar el portal en Netlify
Si se necesita actualizar el archivo index.html:

Ir a app.netlify.com
Abrir el proyecto dulcet-brioche-70b024
En la sección Production deploys, arrastrar el nuevo index.html
Esperar unos segundos — el portal se actualiza automáticamente


🚨 Solución de problemas comunes
Problema
Causa probable
Solución
"Apartamento no encontrado"
El bloque o número no coincide con el Sheets
Verificar que el apartamento esté exactamente como A-0101 en el Sheets
Los datos no se actualizan
El Apps Script apunta a la hoja anterior
Cambiar el nombre de la hoja en el código del Apps Script
El portal no carga
Problema con Netlify o el archivo
Volver a subir el index.html a Netlify
"Contraseña incorrecta"
La contraseña no coincide
Verificar la columna Contraseña en el Sheets
Los meses no aparecen
Las celdas están vacías en el Sheets
Poner 0 en meses sin pago, nunca dejar vacío



📱 ¿Cómo comparten el portal con los residentes?
Pueden compartir por WhatsApp, correo o aviso en la administración:

🏢 Conjunto Mirador de Estambul
Consulte su estado de cuenta en:
👉 dulcet-brioche-70b024.netlify.app

Ingrese con:
• Su bloque (A, B, C o D)
• Su número de apartamento
• Su contraseña personal

(La contraseña la entrega la administración)


👤 Contacto técnico
Este sistema fue construido con asistencia de Claude (Anthropic). Para modificaciones o mejoras, conserve esta documentación y el historial de la conversación original.



Conjunto Mirador de Estambul — Administración — 2026

