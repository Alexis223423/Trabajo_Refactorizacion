# 1. - Introducción

En este documento se va a explicar como se ha llevado a cabo el proceso de refactorización de un codigo en lenguaje Java usando la herramienta SonarQube.

# 2. - Pasos previos

## 2.1 - Creación del proyecto Java en Eclipse

![CrearProjectoJava](/Fotos/1%20-%20Creacion%20de%20JavaProject%20en%20eclipse.png)

## 2.2 - Creación del proyecto en SonarQube

![CrearProjectoSonarQube](/Fotos/2%20-%20Creacion%20de%20proyecto%20en%20sonarqube.png)

## 2.3 - Archivo sonar-project.properties

![ArchivoSonarProperties](/Fotos/3%20-%20Rellenando%20el%20archivo%20sonar-project.properties.png)

Este archivo contiene la configuración que hace falta para que se lleve a cabo el scaneo de SonarQube sobre el proyecto java que se quiere refactorizar, este archivo debe colocarse en el mismo directorio donde se ubique el archivo ".java".

>Referencias:
>>sonar.projectKey - Nombre del proyecto dentro de SonarQube.
>
>>sonar.host-url - URL local o remota con el puerto que usa SonarQube.
>
>>sonar.token - Token generado para ejecutar el análisis desde CMD.
>
>>sonar.languaje - Lenguaje en el que se va a interpretar al escanear.
>
>>sonar.sources - Ruta donde está el cdódigo a analizar.
>
>>sonar.java.binaries - Ruta de los archivos compilados del proyecto.

## 2.4 - Ubicación de la carpeta del proyecto

![UbicacionCarpeta](/Fotos/4%20-%20Ubicación%20del%20proyecto%20en%20CMD,%20compilado%20y%20lanzado%20del%20comando%20sonar-scanner.png)

Se accede desde CMD a la ruta donde se ubica el proyecto que se va a refactorizar y hay que compilar el proyecto usando el siguiente código:
~~~
for /r src %f in (*.java) do javac -d target\classes "%f"
~~~
Acto seguido hay que hacer el scaneo usando este comando:
~~~
sonar-scanner
~~~

## 2.5 -  Proyecto Java antes de refactorizar

![EscaneoPrincipal](/Fotos/5%20-%20Proyecto%20antes%20de%20refactorizar.png)

Al hacer el escaneo inicial se observa que tiene 31 errores los cuales se van a ir solucionando uno a uno.

# 3. - Refactorización

A continuación se van a nombrar todos los problemas que han surgido a raiz del escaneo, muchos de ellos son problemas similares o repetidos, por lo cual solo se mostraran una vez.

## 3.1 - Problema Nº 1

![Issue1](/Fotos/6%20-%20Issue%20System.out%20a%20logger.png)

Este mensaje indica que se está usando System.out.println() para imprimir información en consola, lo cual no es una buena práctica en aplicaciones profesionales.

>Cambios que se realizan:
>>~~~
>>import java.util.logging.Logger;
>>~~~
>
>>~~~
>>private static Logger logger = Logger.getLogger(GestorFutbol.class.getName());
>>~~~
>
>>~~~
>>logger.info("Jugado como local.");
>>~~~

## Problema Nº 2

![Issue2](/Fotos/7%20-%20Issue%20static%20final%20or%20non-public.png)

~~~
private String equipoNombre;
~~~

~~~
public String getEquipoNombre() {
		return equipoNombre;
	}
~~~

~~~
public void setEquipoNombre(String equipoNombre) {
		this.equipoNombre = equipoNombre;
	}
~~~

## Problema Nº 3 

![Issue3](/Fotos/8%20-%20Issue%20eliminar%20private%20field.png)

Solo eliminar las dos lineas siguientes:

~~~
private static String NOMBRE_REAL_MADRID = "Real Madrid club de Fútbol";
~~~

~~~
private static String NOMBRE_ATLETICO_MADRID = "Atlético de Madrid";
~~~

## Problema Nº 4 

![Issue4](/Fotos/9%20-%20Issue%20simply%20return.png)

~~~
return Integer.MIN_VALUE;
~~~
se cambia por:
~~~
return -1;
~~~


## Problema Nº 5

![Issue5](/Fotos/10%20-%20Issue%20built-in.png)

~~~
logger.info("Victoria. Puntos acumulados: " + puntos);
~~~
se cambia por:
~~~
logger.log(Level.INFO,"Victoria. Puntos acumulados: {0}", puntos);
~~~

## Problema Nº 6 

![Issue6](/Fotos/11%20-%20Issue%20complejidad%20cognitiva.png)

Antes era asi:
~~~
switch (resultado.length()) {
                case 7:
                	logger.info("Resultado corto.");
                    break;
                case 14:
                	logger.info("Resultado medio.");
                    break;
                default:
                	logger.info("Resultado de longitud estándar.");
                    break;
            }
~~~
Se resume en este metodo:
~~~
switch_resultados(resultado);
~~~
Y queda asi:
~~~
private void switch_resultados(String resultado) {
		switch (resultado.length()) {
		    case 7:
		    	logger.info("Resultado corto.");
		        break;
		    case 14:
		    	logger.info("Resultado medio.");
		        break;
		    default:
		    	logger.info("Resultado de longitud estándar.");
		        break;
		}
	}
~~~

## Problema Nº 7 

![Issue7](/Fotos/12%20-%20Issue%20equals(%20).png)

~~~
 @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof GestorFutbol)) return false;
        GestorFutbol otro = (GestorFutbol) obj;
        return this.equipoNombre.equals(otro.equipoNombre);
    }
~~~
añadirle abajo esto
~~~
 @Override
    public int hashCode() {
		return puntos;
        
    }
~~~

## Problema Nº 8 

![Issue8](/Fotos/13%20-%20Issue%20clone.png)


yo tenia
~~~
public class GestorFutbol implements Cloneable, Comparable<GestorFutbol>
~~~

~~~
@Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
~~~

y ahora tengo

~~~
public class GestorFutbol implements Comparable<GestorFutbol>
~~~

~~~
public GestorFutbol(GestorFutbol original) {
        this.equipoNombre = original.equipoNombre;
       
    }
~~~

## Proyecto refactorizado

![RefactorizacionCompletada1](/Fotos/15%20-%20Proyecto%20refactorizado.png)

![RefactorizacionCompletada2](/Fotos/14%20-%20Issues%20a%20cero.png)