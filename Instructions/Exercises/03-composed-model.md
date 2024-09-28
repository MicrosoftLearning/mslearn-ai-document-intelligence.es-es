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

    > **Nota**: Si Visual Studio Code muestra un mensaje emergente para solicitarle que confíe en el código que está abriendo, haga clic en la opción **Sí, confío en los autores** en el elemento emergente.
    
    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**. Si hay otros elementos emergentes de Visual Studio Code, puede descartarlos de forma segura.

1. Expanda la carpeta **Labfiles** en el panel izquierdo y haga clic con el botón derecho en el directorio **03-composed-model**. Seleccione la opción de abrir en el terminal integrado y ejecute el siguiente script:

    ```powershell
    az login --output none
    ```

    > **Nota**: Si recibe un error sobre la ausencia de suscripciones activas y tiene la MFA habilitada, es posible que tenga que iniciar sesión en Azure Portal en `https://portal.azure.com` primero y, a continuación, volver a ejecutar `az login`.

1. Cuando se le solicite, inicie sesión en su suscripción de Azure. Después, vuelva a Visual Studio Code y espere a que se complete el proceso de inicio de sesión.
1. En el terminal integrado, ejecute el siguiente comando para configurar los recursos:

   ```powershell
   ./setup.ps1
   ```

   > **IMPORTANTE**: El último recurso creado por el script es el servicio Documento de inteligencia de Azure AI. Si se produjese un error en ese comando debido a que ya tuviera un recurso de nivel F0, use ese recurso para este laboratorio o cree uno manualmente con el nivel S0 en Azure Portal.

## Creación del modelo personalizado de Formularios 1040

Para crear un modelo compuesto, primero debemos crear dos o más modelos personalizados. Para crear el primer modelo personalizado, haga lo siguiente:

1. En una nueva pestaña del explorador, inicie **Documento de inteligencia de Azure AI Studio** en `https://documentintelligence.ai.azure.com/studio`
1. Desplácese hacia abajo y, a continuación, en **Modelos personalizados**, seleccione **Modelo de extracción personalizado**.
1. Si se le pide que inicie sesión en su cuenta, use las credenciales de Azure.
1. Si se le pregunta por el recurso Documento de inteligencia de Azure AI que va a usar, seleccione la suscripción y el nombre que usó al crear dicho recurso.
1. En **Mis proyectos**, seleccione **+ Crear un proyecto**.
1. En el cuadro de texto **Nombre del proyecto**, escriba **Formularios 1040** y luego seleccione **Continuar**.
1. En la página **Configure service resource** (Configurar recurso de servicio), en la lista desplegable **Suscripción**, seleccione la suscripción de Azure.
1. En la lista desplegable **Grupo de recursos**, seleccione el **DocumentIntelligenceResources&lt;xxxx&gt;** creado para usted.
1. En la lista desplegable **Documento de inteligencia o recurso de servicio cognitivo**, selecciona **DocumentIntelligence&lt;xxxx&gt;**.
1. En la lista desplegable **Versión de API**, asegúrate de que **2024-07-31 (versión preliminar)** está seleccionada y después selecciona **Continuar**.
1. En la página **Conectar origen de datos de entrenamiento**, en la lista desplegable **Suscripción**, seleccione la suscripción de Azure.
1. En la lista desplegable **Grupo de recursos**, seleccione **DocumentIntelligenceResources&lt;xxxx&gt;**.
1. En la lista desplegable **Cuenta de almacenamiento**, seleccione la única cuenta de almacenamiento que se muestra. Si tiene varias cuentas de almacenamiento en la suscripción, elija la que empiece por *docintelstorage*
1. En la lista desplegable **Contenedor de blobs**, seleccione **1040examples** y luego seleccione **Continuar**.
1. En la página **Revisar y crear**, seleccione **Crear proyecto**.
1. Selecciona **Ejecutar ahora** en *Ejecutar diseño* en la ventana emergente *Iniciar etiquetado ahora* y espera a que se complete el análisis.

## Etiquetado del modelo personalizado de Formularios 1040

Ahora, vamos a etiquetar los campos en los formularios de ejemplo:

1. En la página **Datos de etiqueta**, en la parte superior derecha de la página, seleccione **+ Agregar un campo** y luego **Campo**.
1. Escriba **Nombre** y luego presione *Entrar*.
1. Seleccione el documento llamado **f1040_1.pdf** en la lista de la izquierda, seleccione **John** y, a continuación, seleccione **FirstName**.
1. En la parte superior derecha de la página, seleccione **+ Agregar un campo** y, luego, **Campo**.
1. Escriba **Apellidos** y luego presione *Entrar*.
1. En el documento, seleccione **Doe** y luego **Apellidos**.
1. En la parte superior derecha de la página, seleccione **+ Agregar un campo** y, luego, **Campo**.
1. Escriba **Ciudad** y luego presione *Entrar*.
1. En el documento, seleccione **Los Ángeles** y luego **Ciudad**.
1. En la parte superior derecha de la página, seleccione **+ Agregar un campo** y, luego, **Campo**.
1. Escriba **Estado** y luego presione *Entrar*.
1. En el documento, seleccione **CA** y luego **Estado**.
1. Repita el proceso de etiquetado de los formularios restantes de la lista de la izquierda con las etiquetas que creó. Etiquete los mismos cuatro campos: *Nombre*, *Apellidos*, *Ciudad* y *Estado*. Observa que uno de los documentos no tiene datos de ciudad o estado.

> **IMPORTANTE** Para este ejercicio, solo se usan cinco formularios de ejemplo y se etiquetan únicamente cuatro campos. En los modelos reales, debe usar tantas muestras como sea posible para maximizar la precisión y la confianza de las predicciones. También debe etiquetar todos los campos disponibles en los formularios, en lugar de solo cuatro campos.

## Entrenamiento del modelo personalizado Formularios 1040

Ahora que los formularios de ejemplo están etiquetados, podemos entrenar el primer modelo personalizado:

1. En Documento de inteligencia de Azure AI Studio, en la parte superior derecha de la pantalla, selecciona **Entrenar**.
1. En el cuadro de diálogo **Train a new model** (Entrenar un nuevo modelo), en el cuadro de texto **Id. de modelo**, escriba **ModeloFormularios1040**.
1. En la lista desplegable **Build mode** (Modo de compilación), seleccione **Plantilla** y luego **Entrenar**. 
1. En el cuadro de diálogo**Training in progress** (Entrenamiento en curso), seleccione **Ir a Modelos**.

## Creación del modelo personalizado Formularios 1099

Ahora, debe crear un segundo modelo, que entrenará en los formularios fiscales 1099 de ejemplo:

1. En Documento de inteligencia de Azure AI Studio, seleccione **Modelo de extracción personalizado**.
1. En **Mis proyectos**, seleccione **+ Crear un proyecto**.
1. En el cuadro de texto **Nombre del proyecto**, escriba **Formularios 1099** y luego seleccione **Continuar**.
1. En la página **Configure service resource** (Configurar recurso de servicio), en la lista desplegable **Suscripción**, seleccione la suscripción de Azure.
1. En la lista desplegable **Grupo de recursos**, seleccione **DocumentIntelligenceResources&lt;xxxx&gt;**.
1. En la lista desplegable **Documento de inteligencia o recurso de servicio cognitivo**, selecciona **DocumentIntelligence&lt;xxxx&gt;**.
1. En la lista desplegable **Versión de API**, asegúrate de que **2024-07-31 (versión preliminar)** está seleccionada y después selecciona **Continuar**.
1. En la página **Configurar origen de datos de entrenamiento**, en la lista desplegable **Suscripción**, seleccione la suscripción de Azure.
1. En la lista desplegable **Grupo de recursos**, seleccione **DocumentIntelligenceResources&lt;xxxx&gt;**.
1. En la lista desplegable **Cuenta de almacenamiento**, seleccione la única cuenta de almacenamiento que se muestra.
1. En la lista desplegable **Contenedor de blobs**, seleccione **1099examples** y luego **Continuar**.
1. En la página **Revisar y crear**, seleccione **Crear proyecto**.
1. Selecciona el botón desplegable de **Ejecutar diseño** y selecciona **Documentos no analizados**.
1. Esperar a que finalice el análisis.

## Etiquetado del modelo personalizado de Formularios 1099

Ahora, etiquete los formularios de ejemplo con algunos campos:

1. En la página **Datos de etiqueta**, en la parte superior derecha de la página, seleccione **+ Agregar un campo** y luego **Campo**.
1. Escriba **Nombre** y luego presione *Entrar*.
1. Seleccione el documento llamado **f1099msc_payer.pdf**, seleccione **John** y, a continuación, seleccione **FirstName**.
1. En la parte superior derecha de la página, seleccione **+ Agregar un campo** y, luego, **Campo**.
1. Escriba **Apellidos** y luego presione *Entrar*.
1. En el documento, seleccione **Doe** y luego **Apellidos**.
1. En la parte superior derecha de la página, seleccione **+ Agregar un campo** y, luego, **Campo**.
1. Escriba **Ciudad** y luego presione *Entrar*.
1. En el documento, seleccione **New Haven** y luego **Ciudad**.
1. En la parte superior derecha de la página, seleccione **+ Agregar un campo** y, luego, **Campo**.
1. Escriba **Estado** y luego presione *Entrar*.
1. En el documento, seleccione **CT** y luego **Estado**.
1. Repita el proceso de etiquetado en los formularios restantes de la lista de la izquierda. Etiquete los mismos cuatro campos: *Nombre*, *Apellidos*, *Ciudad* y *Estado*. Observe que dos de los documentos no tienen ningún dato de nombre que etiquetar.

## Entrenamiento del modelo personalizado de Formularios 1099

Ahora puede entrenar el segundo modelo personalizado:

1. EEn Documento de inteligencia de Azure AI Studio, en la parte superior derecha, selecciona **Entrenar**.
1. En el cuadro de diálogo **Train a new model** (Entrenar un nuevo modelo), en el cuadro de texto **Id. de modelo**, escriba **ModeloFormularios1099**.
1. En la lista desplegable **Build mode** (Modo de compilación), seleccione **Plantilla** y luego **Entrenar**.
1. En el cuadro de diálogo**Training in progress** (Entrenamiento en curso), seleccione **Ir a Modelos**.
1. Este proceso puede tardar varios minutos. Actualice el explorador ocasionalmente hasta que ambos modelos muestren el estado **correcto**.

## Uso del modelo

Ahora que el modelo está completo, vamos a probarlo con un formulario de ejemplo:

1. En Documento de inteligencia de Azure AI Studio, selecciona la página **Modelos**, selecciona el **1040FormsModel**.
1. Seleccione **Probar**.
1. Seleccione **Examinar archivos** y, a continuación, vaya a la ubicación donde ha clonado el repositorio.
1. Seleccione **03-composed-model/trainingdata/TestDoc/f1040_7.pdf** y, a continuación, seleccione **Abrir**.
1. Seleccione **Run analysis** (Ejecutar análisis). Documento de inteligencia de Azure AI analiza el formulario mediante el modelo compuesto.
1. El documento que ha analizado es un ejemplo del formulario fiscal 1040. Compruebe la propiedad **DocType** para ver si se ha usado el modelo personalizado correcto. Compruebe también los valores **Nombre**, **Apellidos**, **Ciudad** y **Estado** que ha identificado el modelo.

## Limpieza de recursos

Ahora que ha visto cómo funcionan los modelos compuestos, vamos a quitar los recursos que ha creado en la suscripción de Azure.

1. En **Azure Portal** en `https://portal.azure.com/`, seleccione **Grupos de recursos**.
1. En la lista de **Grupos de recursos**, seleccione el **DocumentIntelligenceResources&lt;xxxx&gt;** que creó y, a continuación, seleccione **Eliminar grupo de recursos**.
1. En el cuadro de texto **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS**, escriba el nombre del grupo de recursos y, a continuación, seleccione **Eliminar** para eliminar el recurso de Documento de inteligencia y la cuenta de almacenamiento.

## Saber más

- [Composición de modelos personalizados](/azure/ai-services/document-intelligence/concept-composed-models)
