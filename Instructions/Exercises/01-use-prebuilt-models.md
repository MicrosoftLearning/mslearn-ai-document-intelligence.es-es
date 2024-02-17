---
lab:
  title: Usar modelos precompilados de Documento de inteligencia
  module: Module 6 - Document Intelligence
---

# Usar modelos precompilados de Documento de inteligencia

En este ejercicio, configurará un recurso de Documento de inteligencia de Azure AI en la suscripción de Azure. Usará tanto Documento de inteligencia de Azure AI Studio como C# o Python para enviar formularios a ese recurso para su análisis.

## Creación de un recurso de Documento de inteligencia de Azure AI

Para poder llamar al servicio Documento de inteligencia de Azure AI, debe crear un recurso para hospedar ese servicio en Azure:

1. En una pestaña del explorador, abra Azure Portal en [https://portal.azure.com](https://portal.azure.com?azure-portal=true) e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
1. En la página principal de Azure Portal, navegue al cuadro de búsqueda superior, escriba **Documento de inteligencia** y presione **Entrar**.
1. En la página **Documento de inteligencia** seleccione **Crear**.
1. En la página **Crear Documento de inteligencia**, use lo siguiente para configurar el recurso:
    - **Suscripción**: su suscripción de Azure.
    - **Grupo de recursos**: seleccione o cree un grupo de recursos con un nombre único, como *DocIntelligenceResources*.
    - **Región**: seleccione una región cercana.
    - **Nombre**: escriba un nombre único global.
    - **Plan de tarifa**: seleccione **Free F0** (si no tiene un nivel Gratis disponible, seleccione **Standard S0**).
1. Seleccione **Revisar y crear** y **Crear**. Espere mientras Azure crea el recurso de Documento de inteligencia de Azure AI.
1. Cuando la implementación se complete, seleccione **Ir al recurso**. Mantenga esta página abierta para el resto de este ejercicio.

## Uso del modelo de lectura

Empecemos por usar el **Documento de inteligencia de Azure AI Studio** y el modelo de lectura para analizar un documento con varios idiomas. Conectará Documento de inteligencia de Azure AI Studio al recurso que acaba de crear para realizar el análisis:

1. Abra una nueva pestaña del explorador y vaya a **Documento de inteligencia de Azure AI Studio** en [https://documentintelligence.ai.azure.com/studio](https://documentintelligence.ai.azure.com/studio).
1. En **Análisis de documentos**, seleccione el icono **Leer**.
1. Si se le pide que inicie sesión en su cuenta, use las credenciales de Azure.
1. Si se le pregunta qué recurso de Documento de inteligencia de Azure AI va a usar, seleccione la suscripción y el nombre del recurso que usó al crear el recurso de Documento de inteligencia de Azure AI.
1. En la lista de documentos de la izquierda, seleccione **read-german.png**.

    ![Captura de pantalla en la que se muestra la página Lectura de Documento de inteligencia de Azure AI Studio.](../media/read-german-sample.png#lightbox)

1. En la parte superior izquierda, seleccione **Ejecutar análisis**.
1. Una vez completado el análisis, el texto extraído de la imagen se muestra a la derecha en la pestaña **Contenido**. Revise este texto y compárelo con el texto de la imagen original para comprobar su exactitud.
1. Seleccione la pestaña **Resultado**. Esta pestaña muestra el código JSON extraído. 
1. Desplácese hasta la parte inferior del código JSON en la pestaña **Resultado**. Observe que el modelo de lectura ha detectado el idioma de cada intervalo. La mayoría de los intervalos están en alemán (código de idioma `de`), pero el último intervalo está en inglés (código de idioma `en`).

    ![Captura de pantalla en la que se muestra la detección de idioma para dos intervalos en los resultados del modelo de lectura de Documento de inteligencia de Azure AI Studio.](../media/language-detection.png#lightbox)

## Preparación para desarrollar una aplicación en Visual Studio Code

Ahora vamos a explorar la aplicación que usa el SDK del servicio Documento de inteligencia de Azure. Desarrollará la aplicación con Visual Studio Code. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-ai-document-intelligence**, ábralo en Visual Studio Code. De lo contrario, siga estos pasos para clonarlo en el entorno de desarrollo.

1. Inicie Visual Studio Code.
1. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-document-intelligence` en una carpeta local (no importa qué carpeta).
1. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
1. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**. Si aparece el mensaje *Se ha detectado un proyecto de función de Azure en la carpeta*, puede cerrarlo de forma segura.

## Configuración de la aplicación

Se han proporcionado aplicaciones para C# y Python, así como un archivo PDF de ejemplo que usará para probar Documento de inteligencia. Las dos aplicaciones tienen la misma funcionalidad. Primero, completará algunas partes clave de la aplicación para habilitar el uso del recurso Documento de inteligencia de Azure.

1. Examine la siguiente factura y anote algunos de sus campos y valores. Esta es la factura que analizará el código.

    ![Captura de pantalla que muestra un documento de factura de ejemplo.](../media/sample-invoice.png#lightbox)

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta**Labfiles/01-prebuild-models** y expanda la carpeta **CSharp** o **Python** según sus preferencias de lenguaje. Cada carpeta contiene los archivos específicos del lenguaje de una aplicación en la que va a integrar la funcionalidad de Documento de inteligencia de Azure.

1. Haga clic con el botón derecho en la carpeta **CSharp** o **Python** que contenga los archivos de código y seleccione **Abrir un terminal integrado**. Instale el paquete Azure Form Recognizer (el nombre anterior de Documento de inteligencia) mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**:

    ```powershell
    dotnet add package Azure.AI.FormRecognizer --version 4.1.0
    ```

    **Python**:

    ```powershell
    pip install azure-ai-formrecognizer==3.3.0
    ```

## Agregar código para usar el servicio Documento de inteligencia de Azure.

Ahora está listo para usar el SDK para evaluar el archivo PDF.

1. Cambie a la pestaña del explorador que muestra la información general de Documento de inteligencia de Azure AI en Azure Portal. En el panel izquierdo, en *Administración de recursos*, seleccione **Claves y punto de conexión**. A la derecha del valor **Punto de conexión**, haga clic en el botón **Copiar en el portapapeles**.
1. En el panel **Explorador**, en la carpeta **CSharp** o **Python**, abra el archivo de código para su lenguaje preferido y reemplace `<Endpoint URL>` por la cadena que acaba de copiar:

    **C#**: ***Program.cs***

    ```csharp
    string endpoint = "<Endpoint URL>";
    ```

    **Python**: ***document-analysis.py***

    ```python
    endpoint = "<Endpoint URL>"
    ```

1. Cambie a la pestaña del explorador que muestra las **claves y punto de conexión** de Documento de inteligencia de Azure AI en Azure Portal. A la derecha del valor **CLAVE 1**, haga clic en el botón *Copiar en el portapapeles*.
1. En el archivo de código de Visual Studio Code, busque esta línea y reemplace `<API Key>` por la cadena que acaba de copiar:

    **C#**

    ```csharp
    string apiKey = "<API Key>";
    ```

    **Python**

    ```python
    key = "<API Key>"
    ```

1. Busque el comentario `Create the client`. A continuación, en líneas nuevas, escriba el código siguiente:

    **C#**

    ```csharp
    var cred = new AzureKeyCredential(apiKey);
    var client = new DocumentAnalysisClient(new Uri(endpoint), cred);
    ```

    **Python**

    ```python
    document_analysis_client = DocumentAnalysisClient(
        endpoint=endpoint, credential=AzureKeyCredential(key)
    )
    ```

1. Busque el comentario `Analyze the invoice`. A continuación, en líneas nuevas, escriba el código siguiente:

    **C#**

    ```csharp
    AnalyzeDocumentOperation operation = await client.AnalyzeDocumentFromUriAsync(WaitUntil.Completed, "prebuilt-invoice", fileUri);
    await operation.WaitForCompletionAsync();
    ```

    **Python**

    ```python
    poller = document_analysis_client.begin_analyze_document_from_url(
        fileModelId, fileUri, locale=fileLocale
    )
    ```

1. Busque el comentario `Display invoice information to the user`. A continuación, en las líneas nuevas, escriba el código siguiente:

    **C#**

    ```csharp
    AnalyzeResult result = operation.Value;
    
    foreach (AnalyzedDocument invoice in result.Documents)
    {
        if (invoice.Fields.TryGetValue("VendorName", out DocumentField? vendorNameField))
        {
            if (vendorNameField.FieldType == DocumentFieldType.String)
            {
                string vendorName = vendorNameField.Value.AsString();
                Console.WriteLine($"Vendor Name: '{vendorName}', with confidence {vendorNameField.Confidence}.");
            }
        }
    ```

    **Python**

    ```python
    receipts = poller.result()
    
    for idx, receipt in enumerate(receipts.documents):
    
        vendor_name = receipt.fields.get("VendorName")
        if vendor_name:
            print(f"\nVendor Name: {vendor_name.value}, with confidence {vendor_name.confidence}.")
    ```

    > [!NOTE]
    > Ha agregado código para mostrar el nombre del proveedor. El proyecto de inicio también incluye código para mostrar el *nombre del cliente* y el *total de la factura*.

1. Guarde los cambios en el archivo del código.

1. En el panel de terminal interactivo, asegúrese de que el contexto de la carpeta es la carpeta del lenguaje que prefiera. Después, escriba el siguiente comando para ejecutar la aplicación.

1. ***For C# only***, para compilar el proyecto, escriba este comando:

    **C#:**

    ```powershell
    dotnet build
    ```

1. Para ejecutar el código, escriba este comando:

    **C#:**

    ```powershell
    dotnet run
    ```

    **Python**:

    ```powershell
    python document-analysis.py
    ```

El programa muestra el nombre del proveedor, el nombre del cliente y el total de la factura con niveles de confianza. Compare los valores que notifica con la factura de ejemplo que abrió al principio de esta sección.

## Limpiar

Si ha terminado de usar el recurso de Azure, recuerde eliminar el recurso en [Azure Portal](https://portal.azure.com/?azure-portal=true) para evitar cambios.

## Más información

Para obtener más información sobre el servicio Documento de inteligencia, consulte la [documentación de Documento de inteligencia](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true).
