# Publicando tu tercer libro #

## Creando tu propio tema ##

Los temas que incluye **easybook** hacen que tus libros se vean bien sin ningún esfuerzo por tu parte. No obstante, lo lógico es que tus libros definan sus propios estilos para controlar su aspecto con total precisión. Para ello, crea un directorio llamado `Resources` dentro del directorio del libro y en su interior, crea otro directorio llamado `Templates`.

Dentro de ese directorio `Resources/Templates/` ya puedes incluir las plantillas propias con las que se decoran los contenidos del libro. El nombre de las plantillas debe coincidir con el tipo de elemento (`chapter`, `dedication`, `author`, etc.) y su extensión debe ser `.twig`, ya que siempre se crean con el lenguaje de plantillas Twig. Así por ejemplo, para modificar el aspecto de los capítulos, debes crear una plantilla llamada `chapter.twig`.

Al publicar un libro, **easybook** busca primero las plantillas en los siguientes directorios y en el siguiente orden (si no encuentra ninguna de estas plantillas, utiliza la plantilla por defecto):

  1. `<libro>/Resources/Templates/<nombre-edicion>/plantilla.twig`, permite modificar el aspecto para cada edición. El directorio dentro de `Templates` debe llamarse exactamente igual que la edición que se está publicando.
  2. `<libro>/Resources/Templates/<tipo-edicion>/plantilla.twig`, permite modificar el aspecto para todas las ediciones del mismo tipo. El directorio dentro de `Templates` debe llamarse `html`, `html_chunked` o `pdf`.
  3. `<libro>/Resources/Templates/plantilla.twig`, aplica el mismo estilo en todas las ediciones. La plantilla se encuentra directamente dentro de `Templates`, sin incluirla en ningún subdirectorio. Esta opción se utiliza poco porque no suele ser lógico aplicar el mismo estilo a un contenido sin importar si se convierte en un archivo PDF o en un sitio web.

Como se explicó en el capítulo anterior, las plantillas tienen acceso a una variable llamada `item` con toda la información sobre el elemento que se decora y tres variables globales con información sobre el libro (`book`), la edición que se está publicando (`edition`) y la aplicación entera (`app`).

### Plantillas disponibles en cada tipo de edición ###

Las ediciones de tipo `pdf` y `html` disponen de las siguientes plantillas (su contenido varía en cada tipo de edición):

  * `acknowledgement.twig`
  * `toc.twig`
  * `appendix.twig`
  * `author.twig`
  * `book.twig`, esta es la plantilla con la que se crea el libro uniendo todos sus contenidos.
  * `chapter.twig`
  * `cover.twig`
  * `dedication.twig`
  * `edition.twig`
  * `license.twig`
  * `part.twig`
  * `title.twig`

Las ediciones de tipo `html_chunked` no disponen de una plantilla diferente para cada tipo de contenido, sino que simplemente disponen de las siguientes tres plantillas:

  * `index.twig`, corresponde a la portada del sitio web y por defecto incluye la portada, el índice de contenidos, la información sobre la edición y la licencia de la obra.
  * `chunk.twig`, plantilla genérica con la que se decoran todos los contenidos. Se recomienda que incluya solamente los contenidos del elemento, ya que los contenidos comunes al sitio web se colocan en la plantilla `layout.twig`.
  * `layout.twig`, plantilla genérica con la que se decoran todos los contenidos después de decorarlos con las otras plantillas. Esta es la plantilla ideal para añadir todos los elementos comunes del sitio web, como la cabecera, el pie de página y los enlaces a recursos como archivos CSS y JavaScript.

### Hojas de estilos ###

Además de las plantillas Twig, los temas incluyen una hoja de estilos CSS adecuada para cada edición. Estos estilos por defecto de **easybook** se aplican siempre que la opción `include_styles` de la edición sea `true`. Si no quieres aplicarlos, no añadas la opción `include_styles` o cambia su valor a `false`.

En cualquier caso, lo más común es mantener los estilos por defecto y añadir también una hoja de estilos propia que incluya nuevos estilos o modifique los de **easybook**. Para ello, añade un archivo llamado `styles.css` dentro del directorio:

  1. `<libro>/Resources/Templates/<nombre-edicion>/styles.css`, si quieres aplicar los estilos a una única edición.
  2. `<libro>/Resources/Templates/<tipo-edicion>/styles.css`, si quieres aplicar los estilos a todas las ediciones del mismo tipo  (`html`, `html_chunked` o `pdf`).
  3. `<libro>/Resources/Templates/styles.css`,  para aplicar los mismos estilos a todas las ediciones.

## Plugins ##

**easybook** es una herramienta sencilla pero que se adapta fácilmente a tus requerimientos. Los capítulos anteriores incluyen muchos ejemplos de esta flexibilidad, pero los **plugins** que se explican en esta sección son el ejemplo más claro y potente.

Un plugin te permite modificar el comportamiento de **easybook** a tu antojo. Imagina que quieres modificar el contenido Markdown original de los capítulos antes de convertirlo en el código HTML utilizado para crear el libro. Esto resulta muy sencillo porque, antes de convertir un contenido, **easybook** *avisa* que va a hacerlo. Tu libro sólo tiene que crear un plugin que esté atento a ese aviso y ejecute un determinado código si se produce.

Técnicamente, los plugins se basan en los *subscribers* del componente *event dispatcher* de Symfony2 y son clases PHP cuyo nombre siempre acaba en `Plugin`. Observa por ejemplo el código de un plugin sencillo llamado `TimerPlugin.php` que se utiliza para medir cuánto tiempo tarda la publicación del libro:

    <?php
    namespace Easybook\Plugins;
    
    use Symfony\Component\EventDispatcher\EventSubscriberInterface;
    use Easybook\Events\EasybookEvents as Events;
    use Easybook\Events\BaseEvent;

    class TimerPlugin implements EventSubscriberInterface
    {
        static public function getSubscribedEvents()
        {
            return array(
                Events::PRE_PUBLISH  => 'onStart',
                Events::POST_PUBLISH => 'onFinish',
            );
        }

        public function onStart(BaseEvent $event)
        {
            $event->app->set('app.timer.start', microtime(true));
        }

        public function onFinish(BaseEvent $event)
        {
            $event->app->set('app.timer.finish', microtime(true));
        }
    }

Los plugins de **easybook** deben implementar la interfaz `EventSubscriberInterface`, que a su vez obliga a definir un método llamado `getSubscribedEvents()`. Este método simplemente devuelve un array con los eventos a los que te quieres suscribir y el nombre de los métodos que se ejecutan cuando se producen los eventos.

Los eventos disponibles en **easybook** se definen en la clase `Easybook\Events\EasybookEvents` y por el momento son los siguientes:

  * `Events::PRE_PUBLISH`, se notifica antes de empezar la publicación del libro pero después de haber establecido el valor de todas las variables de tipo `publishing.*` de la aplicación.
  * `Events::POST_PUBLISH`, se notifica después de haber publicado completamente el libro pero antes de mostrar al usuario los mensajes que indican dónde puede encontrarlo.
  * `Events::PRE_PARSE`, se notifica antes de iniciar la conversión del contenido original del elemento (normalmente escrito en Markdown).
  * `Events::POST_PARSE`, se notifica después de haber convertido el contenido original del elemento (normalmente este contenido convertido tiene formato HTML).
  * `Events::PRE_NEW`, se notifica al crear un nuevo libro, justo antes de empezar a crear la estructura de archivos y directorios.
  * `Events::POST_NEW`, se notifica justo después de terminar de crear la estructura de archivos y directorios del nuevo libro, pero antes de mostrar al usuario el mensaje que indica que se ha creado un nuevo libro.

Los métodos que se ejecutan para responder a los eventos reciben como primer parámetro un objeto cuyo tipo depende del evento que se trate. En general los eventos reciben un objeto `BaseEvent`, pero los eventos relacionados con el *parseo* de contenidos reciben un objeto de tipo `ParseEvent` (que a su vez hereda de `BaseEvent`).

A través del objeto del evento puedes acceder a cualquier propiedad y servicio de la aplicación mediante `$event->app`, tal como se muestra en el código mostrado anteriormente :

        // ...
        
        public function onStart(BaseEvent $event)
        {
            $event->app->set('app.timer.start', microtime(true));
        }

Si quieres añadir plugins propios a tu libro, crea un directorio llamado `Plugins` dentro del directorio `Resources` de tu libro (crea también este último directorio si no existe). Después, añade tantas clases PHP como quieras, pero haz que su nombre siempre acabe en `Plugin.php`. No añadas ningún *namespace* a las clases de tus plugins.

El siguiente ejemplo muestra un plugin que modifica todos los contenidos del libro antes de su conversión para poner en negrita todas las apariciones de la palabra *easybook*. Después de la conversión, el plugin modifica de nuevo el contenido para añadir el atributo `class` en la etiqueta `<strong>`  que encierra la palabra **easybook**:

    <?php
    use Symfony\Component\EventDispatcher\EventSubscriberInterface;
    use Easybook\Events\EasybookEvents as Events;
    use Easybook\Events\ParseEvent;

    class BrandingPlugin implements EventSubscriberInterface
    {
        static public function getSubscribedEvents()
        {
            return array(
                Events::PRE_PARSE  => 'onItemPreParse',
                Events::POST_PARSE => 'onItemPostParse',
            );
        }
    
        public function onItemPreParse(ParseEvent $event)
        {
            $txt = str_replace(
                'easybook',
                '**easybook**',
                $event->getOriginal()
            );
            
            $event->setOriginal($txt);
        }
        
        public function onItemPostParse(ParseEvent $event)
        {
            $html = str_replace(
                '<strong>easybook</strong>',
                '<strong class="branding">easybook</strong>',
                $event->getContent()
            );
            
            $event->setContent($html);
        }
    }

En los eventos relacionados con el *parseo* de contenidos, el objeto del evento es de tipo `ParseEvent`. Además del acceso a la aplicación (`$event->app`), este objeto tiene *getters* y *setters* para todas las propiedades del elemento que se está *parseando*.

Así por ejemplo, el evento `Events::PRE_PARSE` se notifica antes de convertir el contenido, por lo que sólo está disponible el contenido original (`getOriginal()`) y no el contenido convertido. Por el contrario, el evento `Events::POST_PARSE` se notifica después de conversión, por lo que ya no tiene sentido modificar el contenido original sino el convertido (que puedes acceder mediante el método `getContent()`).

## Características avanzadas ##

### Diferentes directorios para cada libro ###

Si no se indica lo contrario, los libros se crean dentro del directorio `doc/` de **easybook**. Si quieres guardar los contenidos en otro directorio, añade la opción `--dir` tanto al crear el libro como al publicarlo:

    $ ./book new "El origen de las especies" --dir="/Users/javier/libros"
    (el libro se crea en "/Users/javier/libros/el-origen-de-las-especies")
    
    $ ./book publish el-origen-de-las-especies print --dir="/Users/javier/libros"
    (el libro se publica en "/Users/javier/libros/el-origen-de-las-especies/Output/print/book.pdf")

### Configuración avanzada ###

**easybook** no limita de ninguna manera las opciones de configuración que puedes definir para el libro, las ediciones y los contenidos. Cuando utilizas **easybook** tú eres el que mandas, así que puedes inventarte todas las opciones de configuración nuevas que necesites.

¿Quieres incluir el precio del libro en la portada? Añade una opción llamada `precio` bajo `book` en el archivo `config.yml`:

    book:
        ...
        precio: 10

Ahora ya puedes utilizar esta opción en cualquier plantilla mediante `{{ book.precio }}`. ¿Quieres utilizar diferentes *frameworks CSS* al publicar el libro como sitio web? Añade una opción llamada `framework` en todas las ediciones que lo necesiten:

    editions:
        mi_sitio1:
            format:    html_chunked
            framework: twitter_bootstrap
            ...
        
        mi_sitio2:
            extends:   mi_sitio1
            framework: 960_gs

Esta nueva opción `framework` ya está disponible en cualquier plantilla mediante `{{ edition.framework }}`. Por último, también puedes añadir nuevas opciones en los contenidos del libro. ¿Quieres indicar cuál es el tiempo estimado de lectura en cada capítulo? Añade lo siguiente:

    contents:
        - { element: cover }
        - ...
        - { element: chapter, number: 1, ..., time: 20 }
        - { element: chapter, number: 2, ..., time: 12 }
        - ...

Y ahora puedes mostrar en cada capítulo el tiempo estimado que se tarda en leer añadiendo `{{ item.config.time }}` en la plantilla `chapter.twig` con la que se decoran los capítulos.

### Definiendo nuevos tipos de contenido ###

No es algo común y casi nunca es necesario, pero puedes crear nuevos tipos de contenidos al margen de los once tipos definidos por **easybook**. Imagina que entre algunos capítulos quieres incluir una página con una viñeta humorística. Si llamamos `cartoon` a este nuevo tipo de contenido, lo primero que debes hacer es incluirlo en el listado de contenidos del libro:

    contents:
        - { element: cover }
        - ...
        - { element: chapter, number: 1, ... }
        - { element: cartoon, imagen: chiste1.png, content: chiste.md }
        - { element: chapter, number: 2, ... }
        - ...

A continuación, crea el archivo `chiste.md` dentro del directorio de contenidos. No importa si este archivo `chiste.md` está vacío. Lo importante es que **easybook** necesita el contenido de este elemento y no puede utilizar uno de sus contenidos por defecto porque `cartoon` no es uno de los tipos de elemento conocidos por **easybook**.

A continuación, crea los directorios `Resources/Templates/` dentro del directorio de tu libro. En su interior, crea una plantilla llamada `cartoon.twig` y añade por ejemplo el siguiente contenido:

    <div class="page:cartoon new-page">
        <img src="{{ item.config.imagen }}" />
    </div>

¡Y esto es todo! Ya puedes utilizar el nuevo contenido `cartoon` tantas veces como quieras en tu libro y ya puedes crear nuevos tipos de contenido siguiendo los mismos pasos que se acaban de explicar.

### Funcionamiento interno de easybook ###

La flexibilidad de **easybook** te permite crear libros muy avanzados sin esfuerzo y sin tener que estudiar el código fuente de la aplicación. En cualquier caso, para convertirte en un maestro de **easybook** tendrás que adentrarte en lo más profundo de la aplicación.

El núcleo y la filosofía de funcionamiento de **easybook** guarda muchas similitudes con el código fuente de [Silex](http://silex.sensiolabs.org/), un microframework de PHP que también se ha creado con los componentes de Symfony. Si no conoces *Silex*, te recomendamos que lo estudies y lo utilices para crear alguna aplicación de prueba, ya que así te resultará muy fácil entender el código de **easybook**.

La clase más importante de **easybook** es `src/Easybook/DependencyInjection/Application.php`. Esta clase sigue el patrón de *contenedor de inyección de dependencias*, está creada con el componente [Pimple](http://pimple.sensiolabs.org/) y contiene todas las variables, funciones y servicios más importantes de la aplicación.

El comando más interesante de **easybook** es `publish`, que se encarga de publicar los libros. Para ello hace uso de una clase de tipo *publisher*, que depende del tipo de edición que se publica (`html`, `html_chunked` o `pdf`). Los detalles de cada *publisher* varían, pero su funcionamiento básico siempre es el mismo:

    public function publishBook()
    {
        $this->loadContents();
        $this->parseContents();
        $this->decorateContents();
        $this->assembleBook();
    }

Primero se cargan los contenidos del libro (`loadContents()`) definidos en la opción `contents` del archivo `config.yml`. Después se *parsea* cada contenido (`parseContents()`) para convertirlo de su formato original al formato que necesita este *publisher*.

Por el momento **easybook** sólo soporta como formato original Markdown. Si quieres añadir soporte para otros formatos, tienes que crear un nuevo *parser* (fíjate cómo se ha creado el *parser* de Markdown) y modificar también el método `parseContents()` del *publisher*.

Después de convertir todos los contenidos al formato deseado (normalmente HTML) se decoran con ayuda de las plantillas Twig (`decorateContents()`). Por último, el método `assembleBook()` se encarga de crear el libro final. Este es el método más diferente de un *publisher* a otro, ya que a veces hay que crear un archivo PDF, a veces sólo un archivo HTML y otras veces se debe crear un sitio web entero con muchas páginas HTML.
