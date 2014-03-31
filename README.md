# Guía de estilo de Objective-C / Cocoa

> Mi código, mis reglas

Por [Orlando Alemán](http://www.orlandoaleman.com), programador en [BlueDevelopers](http://www.bluedevelopers.com).

## Aspectos generales del código

### Organización

Usaré la siguiente estructura, que estará en sintonía con el sistema de archivos:

	Classes/
		AppDelegate.h			/* Nombre obligatorio */
		AppDelegate.m
		Controllers/
			Delegates/
		Models/
			Delegates/		
		Views/
			Delegates/		
	Supporting/
		Preffix.pch				/* Nombre obligatorio */
		main.m
		Info.plist				/* Nombre obligatorio */
		Localizable.strings
	Resources/
		*.storyboard
		XIB/		
		Images/
		Audio/
		Videos/
	Lib/						/* Extra libraries */
	External/					/* External repositories */
	Tests/
	Frameworks/

### Formateo
Para formatear el código usaré la herramienta Uncrustify. En Github están colgadas [**mis reglas de estilo actuales**](./OrlandoAlemanObjcStyle.cfg). La mayoría de ellas fueron descritas en las guías de [Apple](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html) y [Google](http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml). Al respecto, algunas anotaciones:

* El código se identa con 4 espacios y sin tabuladores. 
* Es preferible no partir las líneas. XCode ya ofrece esa funcionalidad de forma virtual, sin alterar el código.
 
### Uso de comentarios
Los comentarios no pueden maquillar el mal código. Sólo se usarán comentarios en caso de que éstos sean estrictamente necesarios, intentando siempre ser claros y concisos. Su razón principal será explicar *por qué* se hace algo. A excepción de los comentarios de documentación, claro.

En los comentarios de documentación (`/** ... */`, `/*! ... */` y `///`) se emplearán, en caso de ser necesarios, los comandos Doxygen `@brief`, `@param`, `@return` y `@note`.

Los comentarios que añadamos deberán **siempre al día** o bien ser eliminados.

### Métodos vacíos
Se borrará cualquier método vacío.


### Estructuras de control

En la medida de lo posible, a efectos de legibilidad, en los flujos condicionales se intentará retornar sin escribir "`else`", a no ser que hayan instrucciones posteriores que lo impidan. Del mismo modo, preferiremos encerrar los bloques inline en bloques (entre llaves).

	- (void)someMethod
	{
	    if ([someOther boolValue]) {
	        //Do something important
	    } 
        else {
	        //Do something else important
	    }
	}
	
Estaría mejor escrito de la siguiente forma:

	- (void)someMethod
	{
	    if ([someOther boolValue]) {
	        //Do something important
	        return;
	    }
	    //Do something else important
	}

### Expresiones booleanas

Las siguientes expresiones son redundantes:

	if (someObject == nil) ...
	if (someObject == NO) ...
	if ([someObject boolValue] == NO) ...	
	
Sus equivalentes aceptados serían:

	if (!someObject) ...
	if (!someObject) ...
	if (![someObject boolValue]) ...

### Variables

* Se declararán las variables lo más cerca posible de su primer uso. 
* Evitaremos usar nombres de variable crípticos, tal como de una letra, salvo que sea para contadores. Ante todo hay que ser descriptivos. Los IDEs modernos son lo suficientemente potentes como para evitar preocuparnos por esto posteriormente.


### Bloques

En la mayoría de casos preferiremos utilizar blocks en lugar de delegación o hilos. Con Grand Central Dispatch la implementación y legibilidad del código es mayor.

Un detalle a tener en cuenta es que los bloques automaticamente capturan el contexto y retienen cualquier tipo de objecto que sea utilizado dentro de ellos. Así que hay que tener cuidado de no romper el _retain cycle_ ([explicación aquí](http://zearfoss.wordpress.com/2012/05/11/a-quick-gotcha-about-blocks/)).

Por ejemplo, si dentro de un bloque nos referimos a self (usando ARC) hemos de hacer lo siguiente:

	- (void)viewDidLoad
	{
	    [super viewDidLoad];
	    tapBlockView = [[TapBlockView alloc] initWithFrame:CGRectZero];
	    __weak typeof (self) weakself = self;
	    [tapBlockView setTapBlock:^(void) {
	        weakself.someLabel.text = @"You tapped it!";
	    }];
	}


### Literales
Emplear las nuevas expresiones literales. Mejoran muchísimo la legibilidad del código. [Detalles aquí](http://ijoshsmith.com/2012/07/29/objective-c-literals-for-ios-in-xcode-4-4/).


### Constantes

Todos los valores literales y especiales deben ser especificados bien como #define o bien como constantes. Es decir, hay que hacerlos presentables, fácilmente accesibles y entendibles.

### NSLog en entornos de 32+64 bits 
> NSInteger and NSUInteger in a mixed 64bit / 32bit environment

Dado que NSInteger (= int en 32bit, long en 64bit) and NSUInteger (= unsigned int en 32bit, unsigned long en 64bit), la forma de manejar indistintamente es wrappear con @() y usar "%@".

## Gestión de memoria
Utilizar ARC.


## Clases

### Nombres

* Si se van a compartir o reutilizar clases, usar prefijos.

### Interfaces

Las clases se deben organizar situando en primer lugar las constantes públicas, después las variables estáticas privadas, variables de instancia privadas y, a continuación, los métodos. Los métodos *accesor* se situarán junto a los métodos *setters* correspondientes. Las propiedades aparecerán en último lugar.

### Variables de instancia
Las variables de instancia que no se correspondan con propiedades serán nombradas utilizando el sufijo "_"[^1] y se declararán en el fichero fuente (.m).

    @interface MiClase() {
    	id nombreDeVariable_;
    }
    @end

[^1]: La convención para las variables de instancia de propiedades es añadir "_" como prefijo.


### Nombres de métodos

He aquí un ejemplo considerado *correcto*:

	- (void)setApiKey:(NSString *)apiKey;

Las reglas a seguir son:

* Nombres largos y descriptivos mejor que cortos y crípticos. Un buen nombre da mucha más información que cualquier otra cosa. Una buen nombre es aquél con el que se puede inferir su comportamiento de un solo vistazo.
* Se deben seguir las convenciones de Apple, especialmente con relación a la gestión de memoria (`init`, `new`, `alloc`, `copy`).


### Propiedades

* Las propiedades son, por defecto, *strong*. Por tanto, evitaremos especificarlo.

#### Síntesis
Eludiremos utilizar `@synthetize`, puesto que resulta redundante.
El comportamiento por defecto para "writeable properties" es generar automáticamente una variable de instancia asociada de nombre "_nombreDePropiedad".

#### Notación "."
Sólo se debería usar el punto para acceder a propiedades, bien sea obtener su valor o fijarlo directamente. Nunca para ejecutar otro tipo de operaciones.

#### Acceso a las variables de instancia
Sólo se accederá directamente a las variables de instancia durante la inicialización (`-init`) y en los métodos getter/setters. De hecho, no deberían usarse los métodos getter/setters durante la inicialización.


### Inicialización
Los atributos que se requiera un objeto para su operación deberían ser parámetros de inicialización.

	- (id)initWithDelegate:(id<ClassDelegate>)theDelegate;

Además, los parámetros de inicialización deberían poder consultarse desde fuera de la clase.

Para la inicialización de un objeto se sugiere que como máximo sean necesarias 3 líneas de código para que éste sea usable.

No se requiere fijar a `nil` los punteros a objetos. Es el valor por defecto que fija el compilador.

### Protocolos

Los protocolos son una forma fácil, familiar y flexible de adherirse al patrón MVC con bajo acoplamiento.

Se sugiere el uso de los siguientes protocolos:

* Delegate: Para consultar o indicar cambios de estado: "Should...?", "Will", "Did"
* Data-Source: Para obtener datos: "How many...?", "What’s the value for...?"

En los métodos delegados, siempre pasar quién el propio objeto como parámetro. Siempre.

Es conveniente también que los protocolos sean declarados en su propio fichero cabecera.

### Notificaciones
Las notificaciones deberían seguir la misma notación que los métodos delegados y muy posiblemente su ejecución esté relacionada.

### View controllers
Mantener los controladores de vista ligeros.

* [Esta guía de Objc.io](http://www.objc.io/issue-1/lighter-view-controllers.html) explica muy bien algunas técnicas para aligerar y reutilizar los controladores de vistas.


## Hilos
> A programmer had a problem, so he used threads, then he had two
	
Evitaré a toda costa usar hilos. Usaré GCD.

* Para repasar conceptos, una buena guía sobre concurrrencia es [esta entrega de * Objc.io](http://www.objc.io/issue-2/).



## StoryBoards y XIB
* StoryBoards y XIB sólo para layouts muy básicos o prototipos rápidos.

* Siempre que se utilice un UIViewController en combinación con XIBs, no se tiene que especificar el nombre del archivo de xib salvo que estés haciendo algo poco común. UIViewController trata de buscarlo a partir del propio nombre de la clase.

	


## Referencias

Para redactar este documento me he inspirado en varios escritos que consulto con cierta frecuencia. Es más que conveniente echarles una lectura.

* [Luke Redpath's Objective-C Style Guide](http://lukeredpath.co.uk/blog/my-objective-c-style-guide.html)
* [Marcus Zarra's Code Style](http://www.cimgf.com/zds-code-style-guide/)
* [Síntesis de "Clean Code"](http://tratandodeentenderlo.blogspot.com.es/2011/01/clean-code.html)
* ["API Design" by Matt Gemmell](http://mattgemmell.com/2012/05/24/api-design/)
* ["Cocoa Style for Objective-C"](http://www.cocoadevcentral.com/articles/000082.php)
- [Using Blocks in iOS 4: The Basics](http://pragmaticstudio.com/blog/2010/7/28/ios4-blocks-1)
- [Objc.io, A periodical about best practices and advanced techniques in Objective-C](http://www.objc.io)


