---
lab:
  title: Extracción de datos de formularios
  module: Module 6 - Document Intelligence
---

# Extracción de datos de formularios

Imagine que, en una empresa, los empleados actualmente deben comprar manualmente hojas de pedidos y escribir los datos en una base de datos. Les gustaría usar servicios de inteligencia artificial para mejorar el proceso de entrada de datos. Usted decide crear un modelo de aprendizaje automático que lea el formulario y genere datos estructurados que se puedan usar para actualizar automáticamente una base de datos.

**Documento de inteligencia de Azure AI** es un servicio de Azure AI que permite a los usuarios compilar un software de procesamiento de datos automatizado. Este software puede extraer texto, pares clave-valor y tablas de documentos de formulario mediante el reconocimiento óptico de caracteres (OCR). Documento de inteligencia de Azure AI tiene modelos precompilados para reconocer facturas, recibos y tarjetas de presentación. El servicio también proporciona la capacidad de entrenar modelos personalizados. En este ejercicio, nos centraremos en la creación de modelos personalizados.

## Preparación para desarrollar una aplicación en Visual Studio Code

Ahora vamos a usar el SDK del servicio para desarrollar una aplicación mediante Visual Studio Code. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-ai-document-intelligence**, ábralo en Visual Studio Code. De lo contrario, siga estos pasos para clonarlo en el entorno de desarrollo.

1. Inicia Visual Studio Code.
1. Abre la paleta (Mayús + Ctrl + P) y ejecuta un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-document-intelligence` en una carpeta local (no importa qué carpeta).
1. Cuando se haya clonado el repositorio, abre la carpeta en Visual Studio Code.

    > **Nota**: si Visual Studio Code muestra un mensaje emergente para solicitarte que confíes en el código que estás abriendo, haz clic en la opción **Yes, I trust the authors** en el elemento emergente.

1. Espera mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**. Si aparece el mensaje *Se ha detectado un proyecto de función de Azure en la carpeta*, puede cerrarlo de forma segura.

## Crear un recurso de Documento de inteligencia de Azure AI

Para usar el servicio Documento de inteligencia de Azure AI, necesita un recurso de Documento de inteligencia de Azure AI o de Servicios de Azure AI en su suscripción de Azure. Usará Azure Portal para crear un recurso.

1. En una pestaña del explorador, abra Azure Portal en `https://portal.azure.com` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
1. En la página principal de Azure Portal, navegue al cuadro de búsqueda superior, escriba **Documento de inteligencia** y presione **Entrar**.
1. En la página **Documento de inteligencia** seleccione **Crear**.
1. En la página **Crear Documento de inteligencia**, use lo siguiente para configurar el recurso:
    - **Suscripción**: su suscripción de Azure.
    - **Grupo de recursos**: seleccione o cree un grupo de recursos con un nombre único, como *DocIntelligenceResources*.
    - **Región**: seleccione una región cercana.
    - **Nombre**: escriba un nombre único global.
    - **Plan de tarifa**: seleccione **Free F0** (si no tiene un nivel Gratis disponible, seleccione **Standard S0**).
1. Seleccione **Revisar y crear** y **Crear**. Espere mientras Azure crea el recurso de Documento de inteligencia de Azure AI.
1. Cuando se complete la implementación, seleccione **Ir al recurso** para ver la página **Información general** del recurso. 

## Recopilación de documentos para el entrenamiento

Usará los formularios de ejemplo como este para entrenar un modelo de prueba: 

![Imagen de una factura usada en este proyecto.](../media/Form_1.jpg)

1. Vuelva a **Visual Studio Code**. En el panel *Explorador*, abra la carpeta **Labfiles/02-custom-document-intelligence** y expanda la carpeta **sample-forms**. Observe que hay archivos que terminan en **.json** y **.jpg** en la carpeta.

    Usará los archivos **.jpg** para entrenar el modelo.  

    Los archivos **.json** se han generado automáticamente y contienen información de etiqueta. Los archivos se cargarán en el contenedor de Blob Storage junto con los formularios.

    Para ver las imágenes que se usan en la carpeta *sample-forms*, selecciónelas en Visual Studio Code.

1. Vuelva a **Azure Portal** y vaya a la página **Información general** del recurso si aún no está allí. En la sección *Información esencial*, vea el **grupo de recursos** en el que creó el recurso de Documento de inteligencia.

1. En la página **Información general** del grupo de recursos, tome nota de los valores de **Id. de suscripción** y **Ubicación**. Los necesitará, junto con el nombre del **grupo de recursos**, en los pasos posteriores.

    ![Un ejemplo de la página de grupo de recursos.](../media/resource_group_variables.png)

1. En Visual Studio Code, en el panel *Explorador*, vaya a la carpeta **Labfiles/02-custom-document-intelligence** y expanda la carpeta **CSharp** o **Python** según sus preferencias de lenguaje. Cada carpeta contiene los archivos específicos del lenguaje de una aplicación en la que va a integrar la funcionalidad de OpenAI de Azure.

1. Haga clic con el botón derecho en la carpeta **CSharp** o **Python** que contenga los archivos de código y seleccione **Abrir un terminal integrado**. 

1. Ejecute el siguiente comando en el terminal para iniciar sesión en Azure. El comando **az login** abrirá el explorador Microsoft Edge, inicie sesión con la misma cuenta que usó para crear el recurso de Documento de inteligencia de Azure AI. Una vez que haya iniciado sesión, cierre la ventana del explorador.

    ```powershell
    az login
    ```

1. Vuelva a Visual Studio Code. En el panel del terminal, ejecute el siguiente comando para enumerar las ubicaciones de Azure.

    ```powershell
    az account list-locations -o table
    ```

1. En la salida, busque el valor de **Nombre** que corresponde a la ubicación del grupo de recursos (por ejemplo, para *Este de EE. UU.* , el nombre correspondiente es *eastus*).

    > **Importante**: Anote el valor de **Nombre** y úselo en el paso 11.

1. En Visual Studio Code, en la carpeta **Labfiles/02-custom-document-intelligence**, seleccione **setup.cmd**. Usará este script por para ejecutar los comandos de la interfaz de la línea de comandos (CLI) de Azure necesarios para crear los demás recursos de Azure que necesita.

1. En el script **setup.cmd**, revise los comandos. El programa:
    - Creará una cuenta de almacenamiento en el grupo de recursos de Azure.
    - Cargará archivos de la carpeta local *sampleforms* a un contenedor denominado *sampleforms* en la cuenta de almacenamiento
    - Imprimirá un URI de firma de acceso compartido

1. Modificará las declaraciones de las variables **subscription_id**, **resource_group** y **location** con los valores adecuados para la suscripción, el grupo de recursos y el nombre de ubicación donde implementó el recurso de Documento de inteligencia.
Después, **guarde** los cambios.

    Deje la variable **expiry_date** tal y como está para el ejercicio. Esta variable se usa al generar el URI de firma de acceso compartido (SAS). En la práctica, establecerá una fecha de expiración adecuada para la SAS. Puede obtener más información sobre SAS [aquí](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works).  

1. En el terminal de la carpeta **Labfiles/02-custom-document-intelligence**, escriba el siguiente comando para ejecutar el script:

    ```PowerShell
    ./setup.cmd
    ```

1. Cuando se complete el script, revise la salida mostrada.

1. En Azure Portal, actualice el grupo de recursos y compruebe que contiene la cuenta de Azure Storage que acaba de crear. Abra la cuenta de almacenamiento y, en el panel de la izquierda, seleccione **Explorador de almacenamiento** . A continuación, en el explorador de almacenamiento, expanda **Contenedores de blobs** y seleccione el contenedor **sampleforms** para comprobar que los archivos se hayan cargado desde la carpeta local **02-custom- inteligencia/sample-forms**.

## Entrene el modelo mediante Document Intelligence Studio

Ahora entrenará el modelo mediante los archivos cargados en la cuenta de almacenamiento.

1. En el explorador, vaya a Document Intelligence Studio en `https://documentintelligence.ai.azure.com/studio`.
1. Desplácese hacia abajo hasta la sección **Modelos personalizados** y seleccione el icono **Modelo de extracción personalizado**.
1. Si se le pide que inicie sesión en su cuenta, use las credenciales de Azure.
1. Si se le pregunta qué recurso de Documento de inteligencia de Azure AI va a usar, seleccione la suscripción y el nombre del recurso que usó al crear el recurso de Documento de inteligencia de Azure AI.
1. En **Mis proyectos**, seleccione **Crear un proyecto**. Use la siguiente configuración:

    - **Nombre del proyecto**: escriba un nombre único.
        - Seleccione *Continuar*.
    - **Configurar recurso de servicio**: seleccione la suscripción, el grupo de recursos y el recurso de Documento de inteligencia que creó anteriormente en este laboratorio. Marque la casilla *Establecer como predeterminado*. Mantenga la versión predeterminada de la API. 
        - Seleccione *Continuar*.
    - **Conectar origen de datos de entrenamiento**: seleccione la suscripción, el grupo de recursos y la cuenta de almacenamiento que creó el script de instalación. Marque la casilla *Establecer como predeterminado*. Seleccione el `sampleforms` contenedor de blobs y deje en blanco la ruta de acceso de la carpeta.
        - Seleccione *Continuar*.
    - Seleccione *Crear proyecto*.

1. Una vez creado el proyecto, en la parte superior derecha de la pantalla, selecciona **Entrenar** para entrenar tu modelo. Use la siguiente configuración:
    - **Id. del modelo**: *proporcione un nombre único global (necesitará el nombre de Id. del modelo en el siguiente paso).* 
    - **Compilar modo**: plantilla.
1. Selecciona **Go to Models**.
1. El entrenamiento puede tardar algún tiempo. Verá una notificación cuando se complete.

## Pruebe el modelo personalizado de Documento de inteligencia

1. Vuelva a Visual Studio Code. En el terminal, instale el paquete Documento de inteligencia mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**:

    ```powershell
    dotnet add package Azure.AI.FormRecognizer --version 4.1.0
    ```

    **Python**:

    ```powershell
    pip install azure-ai-formrecognizer==3.3.3
    ```

1. En Visual Studio Code, en la carpeta **Labfiles/02-custom-document-intelligence**, seleccione el lenguaje que usa. Edite el archivo de configuración (**appsettings.json** o **.env**, según sus preferencias de lenguaje) para agregar los siguientes valores:
    - El punto de conexión de Documento de inteligencia.
    - La clave del Documento de inteligencia.
    - El Id. de modelo generado que proporcionó cuando entrenó el modelo. Puede encontrarlo en la página **Modelos** de Document Intelligence Studio. Guarde los cambios mediante **Guardar**.

1. En Visual Studio Code, abra el archivo de código de la aplicación cliente (*Program.cs* para C#, *test-model.py* para Python) y revise el código que contiene, especialmente que la imagen de la dirección URL haga referencia al archivo de este repositorio de GitHub en la web.

1. Vuelva al terminal y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```powershell
    dotnet build
    dotnet run
    ```

    **Python**

    ```powershell
    python test-model.py
    ```

1. Vea la salida y observe cómo la salida del modelo proporciona nombres de campo como `Merchant` y `CompanyPhoneNumber`.

## Limpiar

Si ha terminado de usar el recurso de Azure, recuerde eliminar el recurso en [Azure Portal](https://portal.azure.com/?azure-portal=true) para evitar cambios.

## Más información

Para obtener más información sobre el servicio Documento de inteligencia, consulte la [documentación de Documento de inteligencia](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true).
