# TP6-Working-Spring-Hibernate

Proyecto basado en el proyecto en Github [template-java-spring-hibernate](https://github.com/heroku/template-java-spring-hibernate)
Como indica su nombre es una plantilla con un modelo sencillo que trabaja con spring e hibernate. Vamos a estudiarlo para entenderlo y tratar de adaptarlo.

## Observando el código original
En el *POM.xml* podemos observar que contiene todas las librerías necesarias para trabajar con **Spring, jslt, hsqldb, postgresql e hibernate**, entre otras. También vemos que incluye plugins de Maven, aunque entre ellos no está Jetty. Tendremos que incluirlo.

En *webapp*, encontramos un **index.jsp** que contiene instrucciones para configurar Eclipse para poder trabajar con Heroku. Probablemente no lo necesitemos, pero sin duda, es información interesante.

En *WEB-INF>web.xml* observamos que el servlet entrará al juego ante el patrón "/people/\*" en la url.

En *jsp>people.jsp* encontramos un formulario para añadir personas, así como una tabla con el listado de personas almacenadas, y un botón para borrar cada item de la lista. Vemos pues que todo el trabajo pasa por la misma vista.

En *webapp>META-INF* tenemos un *Manifest* sin contenido (probablemente generado por Eclipse).

En *resources>applicationContext.xml* comprobamos que tenemos disponibles los beans de **jspViewResolver, transactionManager**, y dos grupos de beans, uno llamado default que contiene otro bean **entityManagerFactory** (el cual contiene un dataSource y algunas propiedades propias de Hibernate), y otro llamado prod que contiene un **java.net.URI**, un dataSource que recibe datos de **postgresql**, y de nuevo un entityManagerFactory. Vemos que las continuas referencias dataSource (ref="dataSource") que aparecen en este fichero van a ir a buscar al bean con ese nombre, el que hemos visto que bebe de Posgresql.
>**_Nota:_** El _bean_ de la clase **java.net.URI** toma su valor de la variable del sistema _DATABASE_URL_, y a partir de ella construirá el bean _dataSource_ a partir de ella.
>Esto es una forma de enrutar la base de datos que no habíamos visto en clase, y tiene el inconveniente de que los datos de acceso a la base de datos deben ser exportados a dicha variable, lo cual llevaría a conflicto en caso de existir varias aplicaciones con este sistema, y hace difícil el uso de esta plantilla en otros enotrnos, ya que se ha creado para ser usada dentro del sistema Heroku.

Encontramos otro *META-INF* (resources>META-INF) con un fichero **persistence.xml** que configura la persistencia mediante **RESOURCE-LOCAL** (más info en [esta entrada de stackoverflow](http://stackoverflow.com/questions/17331024/persistence-xml-different-transaction-type-attributes)).

Dirigiéndonos por fin a *java>com>example* encontramos una estructura básica mvc: **controller, model y service**. Empecemos por el controlador:
+ *PersonCortroller.java*: controlador básico, sin nada nuevo respecto a lo trabajado anteriormente. Vemos que nos trabajará listPeople(Map) con la ruta básica de la aplicación (**RequestMapping("/")**), addPerson cuando añadamos /add, y siempre que enviemos datos por POST, y finalmente deletePerson cuando encuentre /delete/ seguido de un Id de persona (**"/delete/{personId}"**).
Cada uno de estos métodos deriva el trabajo a su correspondiente de **PersonService**, el cual ha inyectado.

+ *PersonService.java*: es una interfaz la cual se implementa en PersonServiceImpl.java. Describiremos esta.

+ *PersonServiceImpl.java*: Aquí encontramos varias cosas nuevas. Entre ellas las anotaciones **@PersistenceContext** aplicada a la propiedad **EntityManager**, y **@Transactional** aplicada a todos los métodos de la clase. Sobre la primera, en la [documentación de TomEE](http://tomee.apache.org/examples-trunk/injection-of-entitymanager/README.html) nos comentan que el container buscará en **persistence.xml** para inyectar el **EntityManager**, sobre el que luego trabajan los métodos. Acerca del segundo, [jhadesdev en su blog](http://blog.jhades.org/how-does-spring-transactional-really-work/) nos explican cómo esta anotación nos permite ahorrarnos mucho código.
		También encontramos la clase **CriteriaQuery**, resultado de **em.getCriteriaBuilder.createQuery**, esto es, pasa por la interfaz **CriteriaBuilder**; y que se necesita para crear el **TypedQuery** que nos devolverá la lista de personas (*em.createQuery(c).getResultList();*).

+ En el modelo (*Person.java*), nada nuevo: las anotaciones **@Entity, @Id y @GeneratedValue**, importados del paquete *javax.persistence*, y las propiedades con sus getters y setters.

##Modificaciones al código original
Atendiendo a la **nota** anterior, vamos a cambiar el modo de acceso a los datos de la base de datos para que la plantilla sea más versátil y pueda usarse fácilmente en otros entornos. Usaremos para ello una tarjeta _database.properties_, de forma que dichos valores puedan localizarse rápidamente y adaptarse a las necesidades de la aplicación sin dificultad.
También tendremos que añadir en el POM.xml las dependencias de las bases de datos a usar.
