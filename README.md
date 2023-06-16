# documentacion
El código proporcionado es una implementación de la clase `AlumnoJpaController` en Java, que se utiliza para realizar operaciones CRUD (crear, leer, actualizar, eliminar) en la base de datos para la entidad `Alumno` utilizando JPA (Java Persistence API). A continuación, se proporciona una documentación para cada sección del código:

```java
package com.mycompany.jpaprueba.persistencia;

import com.mycompany.jpaprueba.logica.Alumno;
import com.mycompany.jpaprueba.persistencia.exceptions.NonexistentEntityException;
import java.io.Serializable;
import java.util.List;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Query;
import javax.persistence.EntityNotFoundException;
import javax.persistence.Persistence;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Root;
```
- La declaración del paquete `com.mycompany.jpaprueba.persistencia` indica la ubicación del archivo en la estructura del proyecto.
- Se importan las clases y excepciones necesarias para la implementación del controlador JPA.

```java
public class AlumnoJpaController implements Serializable {

    public AlumnoJpaController(EntityManagerFactory emf) {
        this.emf = emf;
    }
    /*------Creacion del Constructor*/

    public AlumnoJpaController() {
        emf = Persistence.createEntityManagerFactory("pruebaJPAPU");
    }
    private EntityManagerFactory emf = null;

    public EntityManager getEntityManager() {
        return emf.createEntityManager();
    }
```
- La clase `AlumnoJpaController` es declarada y se implementa la interfaz `Serializable` para permitir la serialización de objetos de esta clase.
- Se definen los constructores de la clase. Uno de ellos acepta una instancia de `EntityManagerFactory` como parámetro, mientras que el otro crea una instancia utilizando el método `createEntityManagerFactory` de la clase `Persistence`.
- Se declara una variable `emf` del tipo `EntityManagerFactory` y se proporciona un método `getEntityManager()` para obtener un objeto `EntityManager` a partir de la instancia de `emf`.

```java
    public void create(Alumno alumno) {
        EntityManager em = null;
        try {
            em = getEntityManager();
            em.getTransaction().begin();
            em.persist(alumno);
            em.getTransaction().commit();
        } finally {
            if (em != null) {
                em.close();
            }
        }
    }
```
- El método `create` recibe un objeto `Alumno` como parámetro y se encarga de persistirlo en la base de datos. Se obtiene un objeto `EntityManager` a través del método `getEntityManager()`, se inicia una transacción, se utiliza el método `persist` para persistir el objeto `alumno` y se confirma la transacción al llamar al método `commit()`. Finalmente, se asegura de cerrar el `EntityManager`.

```java
    public void edit(Alumno alumno) throws NonexistentEntityException, Exception {
        EntityManager em = null;
        try {
            em = getEntityManager();
            em.getTransaction().begin();
            alumno = em.merge(alumno);
            em.getTransaction().commit();
        } catch (Exception ex) {
            String msg = ex.getLocalizedMessage();
            if (msg == null || msg.length() == 0) {
                int id = alumno.getId();
                if (findAlumno(id) == null) {
                    throw new NonexistentEntityException("The alumno with id " + id + " no longer exists.");
                }
            }
            throw ex;
        } finally {
            if (em != null) {
                em.close();
            }
        }
   

 }
```
- El método `edit` actualiza un objeto `Alumno` en la base de datos. Se obtiene un `EntityManager`, se inicia una transacción, se utiliza el método `merge` para fusionar el objeto `alumno` con el estado persistente en la base de datos y se confirma la transacción. Si ocurre alguna excepción durante el proceso, se captura y se maneja. Finalmente, se cierra el `EntityManager`.

```java
    public void destroy(int id) throws NonexistentEntityException {
        EntityManager em = null;
        try {
            em = getEntityManager();
            em.getTransaction().begin();
            Alumno alumno;
            try {
                alumno = em.getReference(Alumno.class, id);
                alumno.getId();
            } catch (EntityNotFoundException enfe) {
                throw new NonexistentEntityException("The alumno with id " + id + " no longer exists.", enfe);
            }
            em.remove(alumno);
            em.getTransaction().commit();
        } finally {
            if (em != null) {
                em.close();
            }
        }
    }
```
- El método `destroy` elimina un objeto `Alumno` de la base de datos utilizando su identificador (`id`). Se obtiene un `EntityManager`, se inicia una transacción, se utiliza el método `getReference` para obtener una referencia al objeto `Alumno` a eliminar y se llama al método `remove` para eliminarlo. Si el objeto no existe en la base de datos, se lanza una excepción `NonexistentEntityException`. Finalmente, se confirma la transacción y se cierra el `EntityManager`.

```java
    public List<Alumno> findAlumnoEntities() {
        return findAlumnoEntities(true, -1, -1);
    }

    public List<Alumno> findAlumnoEntities(int maxResults, int firstResult) {
        return findAlumnoEntities(false, maxResults, firstResult);
    }

    private List<Alumno> findAlumnoEntities(boolean all, int maxResults, int firstResult) {
        EntityManager em = getEntityManager();
        try {
            CriteriaQuery cq = em.getCriteriaBuilder().createQuery();
            cq.select(cq.from(Alumno.class));
            Query q = em.createQuery(cq);
            if (!all) {
                q.setMaxResults(maxResults);
                q.setFirstResult(firstResult);
            }
            return q.getResultList();
        } finally {
            em.close();
        }
    }
```
- Los métodos `findAlumnoEntities` permiten obtener una lista de todos los objetos `Alumno` de la base de datos. El primer método devuelve todos los resultados, mientras que el segundo permite especificar un número máximo de resultados y un índice de inicio. Ambos métodos utilizan el método privado `findAlumnoEntities` que realiza la consulta y retorna la lista de resultados utilizando Criteria API.

```java
    public Alumno findAlumno(int id) {
        EntityManager em = getEntityManager();
        try {
            return em.find(Alumno.class, id);
        } finally {
            em.close();
        }
    }
```
- El método `findAlumno` busca y devuelve un objeto `Alumno` de la base de datos utilizando su identificador (`id`). Se obtiene un `EntityManager` y se utiliza el método `find` para realizar la búsqueda. Finalmente, se cierra el `EntityManager`.

```java
    public int getAlumnoCount() {
        EntityManager em = getEntityManager();
        try {
            CriteriaQuery cq = em.getCriteriaBuilder().createQuery();
            Root<Alumno> rt = cq.from(Alumno.class);
            cq.select(em

.getCriteriaBuilder().count(rt));
            Query q = em.createQuery(cq);
            return ((Long) q.getSingleResult()).intValue();
        } finally {
            em.close();
        }
    }
}
```
- El método `getAlumnoCount` devuelve el número total de objetos `Alumno` en la base de datos. Se obtiene un `EntityManager`, se crea una consulta utilizando Criteria API para contar los resultados y se ejecuta la consulta. Finalmente, se cierra el `EntityManager` y se retorna el resultado como un entero.

Esta clase `AlumnoJpaController` proporciona métodos para interactuar con la base de datos y realizar operaciones CRUD en la entidad `Alumno`.
