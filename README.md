# Automatización de Venta y Gestión de Cuentas en Make.com

Sistema de automatización sin código en Make.com que, mediante un número virtual conectado a la **API de WhatsApp Business Cloud**, gestiona todo el ciclo de venta, entrega y renovación de paquetes de cuentas con verificación 2FA.

---

## 🚀 Flujo General

1. **Recepción de mensajes (WhatsApp Business Cloud)**

   * El webhook recibe las consultas de los usuarios.
   * El bot de IA (OpenAI) analiza el mensaje y responde con información sobre paquetes, beneficios y modalidades.

2. **Procesamiento de pagos**

   * El webhook de Wompi confirma la transacción.
   * El escenario asigna usuario y contraseña, y genera un código 2FA mediante Beeceptor.

3. **Gestión de datos en Firebase**

   * Crea o actualiza el registro del cliente en Firebase Realtime Database, incluyendo:

     * Datos del usuario.
     * Fecha de compra y periodo de vigencia (1 mes, 3 meses, 6 meses o 1 año).
     * Stock de códigos 2FA.
     * Lista de cuentas disponibles (A, B, C…).

4. **Asignación secuencial de cuentas**

   * Si la cuenta A llega a su límite, el flujo continúa con la cuenta B, y así sucesivamente.
   * Al agotarse la última cuenta, envía una alerta por WhatsApp al administrador.

5. **Renovaciones y alertas**

   * Comprobación diaria de expiraciones en Firebase.
   * Envío automático de recordatorios a los usuarios cuyos paquetes estén por vencer o ya vencidos.

6. **Control de stock de códigos 2FA**

   * Al solicitar un código, verifica el stock en Firebase.
   * Si hay disponibilidad, genera y entrega el código.
   * Si no hay stock, ofrece la opción de adquirir un paquete adicional de códigos.

---

## ⚙️ Configuración en Make.com

1. **Crear Data Store (MENSAJES TEMPORALES)**

   * Crea una *Data Structure* llamada **MENSAJES TEMPORALES**.
   * Agrega un campo con **name**: `mensaje` (tipo: texto).

2. **Crear Data Store (CHAT TEMPORAL)**

   * Crea una *Data Structure* llamada **CHAT TEMPORAL**.
   * Agrega un campo con **name**: `threadID` (para almacenar el ID del chat con OpenAI y mantener el historial).

3. **Configurar Realtime Database (Firebase)**

   * Importa el JSON de tu proyecto en Firebase Realtime Database.

4. **Importar plantilla de escenario**

   * Entra a tu cuenta de Make.com → **Importar escenario** → selecciona el archivo JSON de la plantilla.

5. **Módulo “Mensaje Temporal”**

   * Añade los *inputs* correspondientes al escenario.
   * Selecciona el módulo de Data Store **MENSAJES TEMPORALES**.

6. **Audio a texto**

   * Añade los *inputs* correspondientes al escenario.
   * Selecciona el módulo de WhatsApp Business Cloud y conecta tu **Permanent Token** y **WhatsApp Business Account ID** (obtenidos en Meta).
   * Añade el **Verify Token** (el mismo que configuraste en el webhook de tu app en Meta).
   * Marca la casilla **Messages** en el módulo de WhatsApp.
   * Añade el módulo de OpenAI **Create Transcription** y conecta tu **API Key** y **Organization ID** (obtenidos en OpenAI).
   * Selecciona de nuevo el Data Store **MENSAJES TEMPORALES** para guardar la transcripción.

7. **Imagen a texto**

   * Añade los *inputs* correspondientes al escenario.
   * Selecciona el módulo de WhatsApp Business Cloud (misma conexión).
   * Añade el módulo de OpenAI **Analyze Images**, conecta tu cuenta y escribe el prompt.
   * Guarda el resultado en el Data Store **MENSAJES TEMPORALES**.

8. **Automatización de venta**

   * Añade los *inputs* correspondientes al escenario.
   * Selecciona el módulo de WhatsApp Business Cloud (misma conexión).
   * Utiliza el Data Store **MENSAJES TEMPORALES** según corresponda.

9. **Organizar entrega**

   * Añade los *inputs* correspondientes al escenario.
   * En los módulos **HTTP (Make a Request)**, reemplaza las URL de la base de datos por las de tu Realtime Database de Firebase (p. ej. `https://<tu-proyecto>.firebaseio.com/...`), conservando las variables de Make en la URL.
   * En el campo **content**, ajusta el límite de usuarios (por ejemplo, `>=59`).
   * Selecciona el módulo de WhatsApp Business Cloud (misma conexión).
   * Selecciona el Data Store **MENSAJES TEMPORALES**.

10. **Webhook de pago**

    * Crea un nuevo webhook y copia la URL generada.
    * En Wompi → Desarrollo → Programadores → URL de eventos, pega la URL del webhook.
    * En el módulo **Set multiple variables** (llamado **VARIABLES**), en el campo `tipo_transaccion` usa `real` para producción y `prueba` para testing.
    * En el módulo **Set multiple variables** (llamado **CODIGOS**), agrega cada código de suscripción o producto de Wompi (pago personalizado). Por ejemplo, si tu enlace es `https://checkout.wompi.co/l/DDkpyu`, añade `DDkpyu` en `suscripcion_a`. Repite para cada código, en modo `prueba` y `real`.
    * En el módulo **Set variable** (llamado **FECHA FIN**), configura la lógica de vigencia según el plan (1, 3, 6 meses o 1 año).
    * En los módulos **HTTP (Make a Request)**, reemplaza las URL de Firebase como en el paso anterior.
    * Añade un módulo **Run Scenario** (llamado **ORGANIZAR ENTREGA**) e ingresa su ID.
    * Selecciona el módulo de WhatsApp Business Cloud (misma conexión).
    * **Beeceptor (2FA)**: configura tu endpoint de Beeceptor como webhook adicional.

11. **Variables de escenario**

    * En **Run Scenario** (llamado **TEXTO TEMPORAL**), pon el ID del escenario **MENSAJE TEMPORAL**.
    * En **Run Scenario** (llamado **AUDIO TEXTO**), pon el ID del escenario **AUDIO A TEXTO**.
    * En **Run Scenario** (llamado **ANALIZAR TEXTO**), pon el ID del escenario **IMAGEN A TEXTO**.
    * En **Get a record** (llamado **OBTENER CHAT TEMPORAL**), selecciona el Data Store **CHAT TEMPORAL**.
    * En **Get a record** (llamado **TRAER MENSAJES**), selecciona el Data Store **MENSAJES TEMPORALES**.
    * En **Delete a record** (llamado **ELIMINAR MENSAJES TEM.**), selecciona el Data Store **MENSAJES TEMPORALES**.
    * En **Add/replace a record** (llamado **REGISTRO CHAT TEMPORAL**), selecciona el Data Store **CHAT TEMPORAL**.
    * En los módulos de WhatsApp Business Cloud, selecciona tu conexión.
    * En los módulos **HTTP (Make a Request)**, reemplaza las URL de Firebase según los pasos anteriores.
    * En los módulos **HTTP** llamados **CUENTA X30**, **CUENTA X3**, **CUENTA X6**, **CUENTA X12**, **CODIGO EXTRA X2**, ajusta el contenido de la petición:

      * **IMAGE → LINK**: URL pública de la imagen del botón.
      * **BODY → TEXT**: Título del botón.
      * **ACTION → PARAMETERS → DISPLAY\_TEXT**: Texto del botón.
      * **ACTION → PARAMETERS → URL**: Enlace de pago completo de Wompi.
      * **FOOTER**: Descripción del botón (precio de la suscripción).
    * En los módulos **HTTP** llamados **CODE ACC-A**, **CODE ACC-B**, **CODE ACC-C**, **CODE ACC-D**, ajusta el contenido de la petición:

      * **ISSUER**: URL de tu aplicación.
      * **SECRET**: Código 2FA de la cuenta, generado en la plataforma o servicio que vendas.
---

## 📚 Referencias de API

- **WhatsApp Business Cloud**  
  https://developers.facebook.com/docs/whatsapp/cloud-api

- **OpenAI**  
  https://platform.openai.com/docs/overview

- **Firebase Realtime Database**  
  https://firebase.google.com/docs/database

- **Wompi**  
  https://docs.wompi.co/

- **Beeceptor (2FA)**  
  https://beeceptor.com/

---

## 🤝 Contribución

- Crea una copia de este escenario para tus pruebas.  
- Ajusta variables y conexiones según tu entorno.  
- Envía feedback o mejora el flujo a través de este repositorio.  

---

## 📄 Licencia

Este proyecto es de uso libre para implementaciones en Make.com y se distribuye bajo licencia **Apache‑2.0**.  
