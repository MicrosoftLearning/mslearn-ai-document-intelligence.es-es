---
lab:
  title: Creación de un modelo de inteligencia de documentos compuesto
  module: Module 6 - Document Intelligence
---

# Creación de un modelo de inteligencia de documentos compuesto

En este ejercicio, creará y entrenará dos modelos personalizados que analizan diferentes formularios fiscales. Después, creará un modelo compuesto que incluye ambos modelos personalizados. Probará el modelo enviando un formulario y comprobará que reconoce el tipo de documento y los campos etiquetados correctamente.

## Configurar recursos

Usaremos un script para crear el recurso de ADocumento de inteligencia de Azure AI, una cuenta de almacenamiento con formularios de ejemplo y un grupo de recursos:

1. Inicie Visual Studio Code.
1. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-document-intelligence` en una carpeta local (no importa qué carpeta).
1. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
1. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

1. Haga clic con el botón derecho en el directorio **03-composed-model**, ábralo en el terminal integrado y ejecute el script de configuración:

   ``` bash
   bash setup.sh
   ```

## Creación del modelo personalizado de Formularios 1040

Para crear un modelo compuesto, primero debemos crear dos o más modelos personalizados. Para crear el primer modelo personalizado, haga lo siguiente:

1. En una nueva pestaña del explorador, inicie [Documento de inteligencia de Azure AI Studio](https://formrecognizer.appliedai.azure.com/studio).
1. Desplácese hacia abajo y, después, en **Modelo personalizado**, seleccione **Modelo personalizado**.
1. Si se le pide que inicie sesión en su cuenta, use las credenciales de Azure.
1. Si se le pregunta por el recurso Documento de inteligencia de Azure AI que va a usar, seleccione la suscripción y el nombre que usó al crear dicho recurso.
1. En **Mis proyectos**, seleccione **+ Crear un proyecto**.
1. En el cuadro de texto **Nombre del proyecto**, escriba **Formularios 1040** y luego seleccione **Continuar**.
1. En la página **Configure service resource** (Configurar recurso de servicio), en la lista desplegable **Suscripción**, seleccione la suscripción de Azure.
1. En la lista desplegable **Grupo de recursos**, seleccione **DocumentIntelligenceResources**.
1. En la lista desplegable **Documento de inteligencia de Azure AI o Recurso del servicio de Azure AI**, seleccione **DocumentIntelligence**
1. En la lista desplegable **Versión de API**, asegúrese de que **2022-06-30-preview** está seleccionado y, después, seleccione **Continuar**.

    :::image type="content" source="../media/4-configure-service-resources.png" alt-text="Captura de pantalla que muestra la página de configuración de recursos de servicio del asistente para modelos personalizados de Documento de inteligencia de Azure AI." lightbox="../media/4-configure-service-resources.png":::

1. En la página **Configure training data source** (Configurar entrenamiento de origen de datos), en la lista desplegable **Suscripción**, seleccione la suscripción de Azure.
1. En la lista desplegable **Grupo de recursos**, seleccione **DocumentIntelligenceResources**.
1. En la lista desplegable **Cuenta de almacenamiento**, seleccione la única cuenta de almacenamiento que se muestra.
1. En la lista desplegable **Contenedor de blobs**, seleccione **1040examples** y luego seleccione **Continuar**.

    :::image type="content" source="../media/4-connect-training-data-source.png" alt-text="Captura de pantalla que muestra la página de conexión de origen de datos de formación del asistente para modelos personalizados de Documento de inteligencia de Azure AI Studio." lightbox="../media/4-connect-training-data-source.png":::

1. En la página **Revisar y crear**, seleccione **Crear proyecto**.

## Etiquetado del modelo personalizado de Formularios 1040

Ahora, vamos a etiquetar los campos en los formularios de ejemplo:

1. En la página **Label data** (Datos de etiqueta), en la parte superior derecha de la página, seleccione **+** y luego **Campo**.

    :::image type="content" source="../media/4-add-label.png" alt-text="Captura de pantalla que muestra cómo agregar una nueva etiqueta en Documento de inteligencia de Azure AI Studio." lightbox="../media/4-add-label.png":::

1. Escriba **Nombre** y luego presione <kbd>Entrar</kbd>.
1. En el documento, seleccione **John** y luego **Nombre**.

    :::image type="content" source="../media/4-label-first-name.png" alt-text="Captura de pantalla que muestra cómo completar una nueva etiqueta en Documento de inteligencia de Azure AI Studio." lightbox="../media/4-label-first-name.png":::

1. En la parte superior derecha de la página, seleccione **+** y luego **Campo**.
1. Escriba **Apellidos** y luego presione <kbd>Entrar</kbd>.
1. En el documento, seleccione **Doe** y luego **Apellidos**.
1. En la parte superior derecha de la página, seleccione **+** y luego **Campo**.
1. Escriba **Ciudad** y luego presione <kbd>Entrar</kbd>.
1. En el documento, seleccione **Los Ángeles** y luego **Ciudad**.
1. En la parte superior derecha de la página, seleccione **+** y luego **Campo**.
1. Escriba **Estado** y luego presione <kbd>Entrar</kbd>.
1. En el documento, seleccione **CA** y luego **Estado**.
1. Repita el proceso de etiquetado en los formularios restantes de la lista de la izquierda. Etiquete los mismos cuatro campos: *Nombre*, *Apellidos*, *Ciudad* y *Estado*.

> [!IMPORTANT]
> Para este ejercicio, solo se usan cinco formularios de ejemplo y se etiquetan únicamente cuatro campos. En los modelos reales, debe usar tantas muestras como sea posible para maximizar la precisión y la confianza de las predicciones. También debe etiquetar todos los campos disponibles en los formularios, en lugar de solo cuatro campos.

## Entrenamiento del modelo personalizado Formularios 1040

Ahora que los formularios de ejemplo están etiquetados, podemos entrenar el primer modelo personalizado:

1. En Documento de inteligencia de Azure AI Studio, seleccione **Entrenar**.
1. En el cuadro de diálogo **Train a new model** (Entrenar un nuevo modelo), en el cuadro de texto **Id. de modelo**, escriba **ModeloFormularios1040**.
1. En la lista desplegable **Build mode** (Modo de compilación), seleccione **Plantilla** y luego **Entrenar**. 
1. En el cuadro de diálogo**Training in progress** (Entrenamiento en curso), seleccione **Ir a Modelos**.

## Creación del modelo personalizado Formularios 1099

Ahora, debe crear un segundo modelo, que entrenará en los formularios fiscales 1099 de ejemplo:

1. En Documento de inteligencia de Azure AI Studio, seleccione **Modelo personalizado**.
1. En **Mis proyectos**, seleccione **+ Crear un proyecto**.
1. En el cuadro de texto **Nombre del proyecto**, escriba **Formularios 1099** y luego seleccione **Continuar**.
1. En la página **Configure service resource** (Configurar recurso de servicio), en la lista desplegable **Suscripción**, seleccione la suscripción de Azure.
1. En la lista desplegable **Grupo de recursos**, seleccione **DocumentIntelligenceResources**.
1. En la lista desplegable **Documento de inteligencia de Azure AI o Recurso del servicio de Azure AI**, seleccione **DocumentIntelligence**
1. En la lista desplegable **Versión de API**, asegúrese de que **2022-06-30-preview** está seleccionado y, después, seleccione **Continuar**.

    :::image type="content" source="../media/4-configure-service-resources.png" alt-text="Captura de pantalla que muestra la página de configuración de recursos de servicio del asistente para modelos personalizados de Documento de inteligencia de Azure AI." lightbox="../media/4-configure-service-resources.png":::

1. En la página **Configure training data source** (Configurar entrenamiento de origen de datos), en la lista desplegable **Suscripción**, seleccione la suscripción de Azure.
1. En la lista desplegable **Grupo de recursos**, seleccione **DocumentIntelligenceResources**.
1. En la lista desplegable **Cuenta de almacenamiento**, seleccione la única cuenta de almacenamiento que se muestra.
1. En la lista desplegable **Contenedor de blobs**, seleccione **1099examples** y luego **Continuar**.
1. En la página **Revisar y crear**, seleccione **Crear proyecto**.

## Etiquetado del modelo personalizado de Formularios 1099

Ahora, etiquete los formularios de ejemplo con algunos campos:

1. En la página **Label data** (Datos de etiqueta), en la parte superior derecha de la página, seleccione **+** y luego **Campo**.
1. Escriba **Nombre** y luego presione <kbd>Entrar</kbd>.
1. En el documento, seleccione **John** y luego **Nombre**.
1. En la parte superior derecha de la página, seleccione **+** y luego **Campo**.
1. Escriba **Apellidos** y luego presione <kbd>Entrar</kbd>.
1. En el documento, seleccione **Doe** y luego **Apellidos**.
1. En la parte superior derecha de la página, seleccione **+** y luego **Campo**.
1. Escriba **Ciudad** y luego presione <kbd>Entrar</kbd>.
1. En el documento, seleccione **New Haven** y luego **Ciudad**.
1. En la parte superior derecha de la página, seleccione **+** y luego **Campo**.
1. Escriba **Estado** y luego presione <kbd>Entrar</kbd>.
1. En el documento, seleccione **CT** y luego **Estado**.
1. Repita el proceso de etiquetado en los formularios restantes de la lista de la izquierda. Etiquete los mismos cuatro campos: *Nombre*, *Apellidos*, *Ciudad* y *Estado*.

## Entrenamiento del modelo personalizado de Formularios 1099

Ahora puede entrenar el segundo modelo personalizado:

1. En Documento de inteligencia de Azure AI Studio, seleccione **Entrenar**.
1. En el cuadro de diálogo **Train a new model** (Entrenar un nuevo modelo), en el cuadro de texto **Id. de modelo**, escriba **ModeloFormularios1099**.
1. En la lista desplegable **Build mode** (Modo de compilación), seleccione **Plantilla** y luego **Entrenar**. 
1. En el cuadro de diálogo**Training in progress** (Entrenamiento en curso), seleccione **Ir a Modelos**.
1. Este proceso puede tardar varios minutos. Actualice el explorador ocasionalmente hasta que ambos modelos muestren el estado **correcto**.

## Creación y ensamblado de un modelo compuesto

Los dos modelos personalizados, que analizan los formularios fiscales 1040 y 1099, ahora están completos. Puede continuar con la creación del modelo compuesto:

1. En la página Modelos de Documento de inteligencia de Azure AI, seleccione **1040FormsModel** y **1099FormsModel**.
1. En la parte superior de la lista de modelos, seleccione **Redactar**.

    :::image type="content" source="../media/4-start-compose-model.png" alt-text="Captura de pantalla que muestra cómo empezar a componer un modelo en Documento de inteligencia de Azure AI Studio." lightbox="../media/4-start-compose-model.png":::

1. En el cuadro de diálogo **Compose a new model** (Redactar un nuevo modelo), en el cuadro de texto **Id. de modelo**, escriba **ModeloFormulariosFiscales** y luego seleccione **Redactar**. Documento de inteligencia de Azure AI crea el modelo compuesto y lo muestra en la lista de modelos personalizados:

## Uso del modelo compuesto

Ahora que el modelo compuesto está completo, vamos a probarlo con un formulario de ejemplo:

1. En [Azure Portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true), seleccione **Todos los recursos** y luego la cuenta de almacenamiento **formsrecstorage&lt;xxxxx&gt;**, donde &lt;xxxxx&gt; es un número aleatorio.
1. En **Almacenamiento de datos**, seleccione **Contenedores** y luego **TestDoc**.
1. A la derecha de **f1040_7.pdf**, seleccione **...** y luego **Descargar**.
1. Guarde el documento PDF en el equipo local y anote la ubicación guardada.
1. En Documento de inteligencia de Azure AI Studio, seleccione **TaxFormsModel** y, a continuación, seleccione **Probar**.
1. Seleccione **+ Agregar** y luego vaya a la ubicación donde ha guardado el documento PDF.
1. Seleccione **f1040_7.pdf** y luego **Abrir**.
1. Seleccione **Analizar**. Documento de inteligencia de Azure AI analiza el formulario mediante el modelo compuesto.

    :::image type="content" source="../media/4-composed-model-analysis.png" alt-text="Captura de pantalla que muestra cómo usar un modelo compuesto en Documento de inteligencia de Azure AI Studio." lightbox="../media/4-composed-model-analysis.png":::

1. El documento que ha analizado es un ejemplo del formulario fiscal 1040. Compruebe la propiedad **DocType** para ver si se ha usado el modelo personalizado correcto. Compruebe también los valores **Nombre**, **Apellidos**, **Ciudad** y **Estado** que ha identificado el modelo.

## Limpieza de los recursos del ejercicio

Ahora que ha visto cómo funcionan los modelos compuestos, vamos a quitar los recursos que ha creado en la suscripción de Azure.

1. En [Azure Portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true), seleccione **Grupos de recursos**.
1. En la lista de **Grupos de recursos**, seleccione **DocumentIntelligenceResources** y luego **Eliminar grupo de recursos**. 
1. En el cuadro de texto **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS**, escriba **DocumentIntelligenceResources** y, a continuación, seleccione **Eliminar** para eliminar el recurso de Documento de inteligencia y la cuenta de almacenamiento.

## Saber más

- [Composición de modelos personalizados](/azure/ai-services/document-intelligence/concept-composed-models)
- [Compilación del conjunto de datos de entrenamiento para un modelo personalizado](/azure/applied-ai-services/form-recognizer/how-to-guides/build-custom-model-v3)
