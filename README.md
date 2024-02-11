
# Proyecto de base de datos para un Videoclub
 
## Introdución
 
Este proyecto de base de datos para un videoclub está diseñado para gestionar eficientemente los datos relacionados con socios, películas, préstamos y direcciones. Utilizando un enfoque relacional, el esquema de base de datos permite una gestión detallada de los recursos del videoclub, asegurando la integridad y accesibilidad de los datos.
 

## Diagrama Entidad-Relación (ER)

El diagrama ER está organizado para representar las entidades principales del sistema del videoclub y sus interrelaciones:

-  **Socio**: Representa a los miembros del videoclub. Cada socio está identificado de manera única por un `id_socio` y tiene atributos personales como `nombre`, `apellido1`, `apellido2`, `email`, `telefono`, `fecha_nacimiento`, y `dni`.

-  **Direccion**: Almacena las direcciones de los socios. Está relacionada directamente con la tabla `socio` a través de `id_socio`, permitiendo que cada socio tenga una única dirección asociada. No es un dato obligatorio.

-  **Pelicula**: Contiene información sobre las películas disponibles en el videoclub, incluyendo un `id_pelicula`, `titulo`, `id_genero`, `id_director`, y `sinopsis`.

-  **Genero** y **Director**: Estas tablas almacenan información sobre los géneros de películas y los directores, respectivamente, y se relacionan con la tabla `pelicula` para proporcionar una estructura normalizada.

-  **Copia**: Mantiene registros de las copias físicas individuales de las películas. Cada copia está vinculada a una película específica por `id_pelicula` y tiene un `id` único y un `id_copia` para identificar la copia específica dentro de las copias de esa película.

-  **Prestamo**: Registra los préstamos de copias a los socios, incluyendo información sobre el socio, la copia prestada, y las fechas de préstamo y devolución. En el caso que no haya fecha de devolución se considera no disponible.

El diagrama entidad / relación es el archivo diagrama-entidad-relacion.drawio

  
## Diseño y Justificación

  

-  **Tablas de Socios y Direcciones**: La separación de las tablas `socio` y `direccion` facilita la gestión de información personal y de contacto, permitiendo cambios en las direcciones sin afectar la información principal del socio. Cada socio puede tener a lo máximo una dirección, lo que se asegura mediante la relación uno a uno entre estas tablas.

-  **Tabla de Copias y su Relación con Películas**: La distinción entre `id` y `id_copia` en la tabla `copia` permite gestionar múltiples instancias físicas de una película. El `id` es un identificador único para cada fila (copia) en la tabla, mientras que `id_copia` se utiliza para numerar las copias de una misma película, permitiendo un seguimiento detallado del inventario de películas.

  

-  **Relaciones y Claves Foráneas**: Las relaciones entre las tablas están diseñadas para preservar la integridad referencial y facilitar consultas relacionales. Por ejemplo, la relación entre `prestamo` y `copia` asegura que solo se puedan prestar copias existentes, y la relación entre `pelicula`, `genero`, y `director` permite una organización clara de la información de películas.

  

## Uso del Script

  

El script SQL proporcionado crea la estructura de la base de datos, incluidas todas las tablas y relaciones mencionadas. Para utilizar este script:

  

Ejecuta el script en tu sistema de gestión de base de datos para crear el esquema.

  

## Consultas Clave

  

En este proyecto, utilizamos varias consultas SQL para extraer información significativa de la base de datos. Dos consultas son particularmente importantes para la operación del videoclub:

  

### Consulta 1: Películas Disponibles para Alquilar

  

Para identificar qué películas están disponibles para alquilar en este momento, utilizamos la siguiente consulta:

  

```sql

-- Consulta 1: Películas disponibles para alquilar en este momento

SELECT p.titulo, COUNT(*) AS copias_disponibles

FROM pelicula p

JOIN copia c ON p.id_pelicula = c.id_pelicula

LEFT JOIN prestamo pr ON c.id = pr.id

WHERE pr.id IS  NULL  OR pr.fecha_devolucion IS NOT NULL

GROUP BY p.titulo;
```
  

Esta consulta emplea un LEFT JOIN para incluir todas las copias de películas, independientemente de si están actualmente prestadas. Esto asegura que incluso las copias que nunca han sido prestadas se cuenten como disponibles. El WHERE pr.id IS  NULL  OR pr.fecha_devolucion IS NOT NULL permite filtrar las copias que están disponibles para ser alquiladas, ya sea porque no han sido prestadas o porque el préstamo ha sido devuelto.

  

### Consulta 2: Género favorito para cada socio

  

Para determinar el género favorito de cada socio basado en sus préstamos, utilizamos la siguiente consulta:

```sql

  -- Consulta 2: Género favorito de cada socio

SELECT s.id_socio, s.nombre, s.apellido1, s.apellido2, g.nombre_genero AS genero_favorito

FROM socio s

LEFT JOIN (

SELECT s.id_socio, p.id_genero, COUNT(*) AS cantidad

FROM prestamo pr

JOIN copia c ON pr.id = c.id

JOIN pelicula p ON c.id_pelicula = p.id_pelicula

JOIN socio s ON pr.id_socio = s.id_socio

GROUP BY s.id_socio, p.id_genero

ORDER BY s.id_socio, cantidad DESC

) AS subquery ON s.id_socio = subquery.id_socio

LEFT JOIN genero g ON subquery.id_genero = g.id_genero

GROUP BY s.id_socio, s.nombre, g.nombre_genero;
```
  

Esta consulta utiliza LEFT JOIN para asegurar que todos los socios sean incluidos en los resultados, incluso aquellos que no han realizado ningún préstamo. De esta manera, podemos identificar el género favorito de cada socio basándonos en la cantidad de préstamos de películas de cada género, incluyendo a los socios sin préstamos (para quienes el género favorito será NULL).

La decisión de utilizar LEFT JOIN en estas consultas se tomó para maximizar la inclusividad de los datos, asegurando que toda la información relevante sea considerada, independientemente de la presencia o ausencia de préstamos. Esto permite una visión completa y precisa del inventario y las preferencias de los socios, crucial para la operación efectiva del videoclub.

  
## Conclusión

  
Este diseño de base de datos para un videoclub ofrece una solución robusta y escalable para la gestión de datos de socios, películas, y préstamos. La estructura normalizada y las relaciones cuidadosamente planificadas aseguran la integridad de los datos y facilitan la extensión del sistema para satisfacer necesidades futuras.