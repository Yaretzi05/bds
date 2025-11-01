# Catálogo de Razas de Gatos

[cite_start]Este proyecto contiene un catálogo de razas de gatos, diseñado para la tercera iteración del proyecto de Bases de Datos. El modelo de datos está normalizado para facilitar su integración y consulta.

El modelo incluye información sobre las razas, su clasificación (por grupo y tipo de pelo), características físicas (peso, tamaño, esperanza de vida), estatus de conservación y requerimientos de cuidado. Adicionalmente, incluye tablas para un sistema de gestión de mascotas, permitiendo registrar propietarios y mascotas individuales.

## Modelo Relacional

El modelo se compone de las siguientes tablas:

* *Clasificacion*: Agrupa a las razas por tipo (ej. 'Pelo corto').
* *Estatus*: Almacena las categorías de conservación (ej. 'Vulnerable').
* *Raza*: La tabla principal de razas (ej. 'Siames', 'Persa').
* *Caracteristicas*: Detalles físicos de cada raza (peso, tamaño, etc.).
* *Requerimiento*: Cuidados asociados a cada clasificación (ej. 'Cepillado', 'Nivel de actividad').
* *Propietario*: Información de los dueños de las mascotas.
* *Mascota*: Registros de mascotas individuales, vinculadas a un dueño y una raza.

## Instrucciones de Importación en PostgreSQL (Linux)

[cite_start]Para importar el catálogo a una base de datos de PostgreSQL, sigue estos pasos[cite: 34, 35]. (Asegúrate de que los archivos CSV estén en el mismo directorio donde ejecutas psql).

1.  Conéctate a tu base de datos:
    bash
    psql -U tu_usuario -d tu_base_de_datos
    

2.  Ejecuta los siguientes comandos \copy en orden para respetar las llaves foráneas. Asumimos que los archivos CSV tienen cabeceras (headers).

    sql
    -- Primero, importar las tablas independientes (catálogos)
    \copy Clasificacion FROM 'clasificacion.csv' WITH CSV HEADER;
    \copy Estatus FROM 'estatus.csv' WITH CSV HEADER;
    \copy Propietario FROM 'propietario.csv' WITH CSV HEADER;

    -- Segundo, importar las tablas que dependen de las anteriores
    \copy Raza FROM 'raza.csv' WITH CSV HEADER;
    \copy Requerimiento FROM 'requerimiento.csv' WITH CSV HEADER;
    
    -- Tercero, importar las tablas que dependen de Raza
    \copy Caracteristicas FROM 'caracteristicas.csv' WITH CSV HEADER;
    \copy Mascota FROM 'mascota.csv' WITH CSV HEADER;



    # **CATÁLOGO DE RAZAS DE REPTILES**

## **Descripción del Catálogo**

El catálogo de razas de reptiles es una estructura que permite organizar la información taxonómica, las características de cada especie o raza, sus requerimientos ambientales y su estatus de conservación.

Este catálogo se diseñó para:

- **Autocompletar datos dependientes:** al elegir una raza, se muestran automáticamente sus características, estatus y requerimientos.  
- **Evitar redundancia:** cada dato se almacena en una tabla específica y se relaciona mediante llaves foráneas.

El catálogo se implementa principalmente en las tablas **`Clasificación`** y **`Raza`**, que son la base para organizar la jerarquía.

---

## **INCORPORACIÓN A LA BASE DE DATOS**

### Tabla `Clasificación`
Agrupa reptiles por niveles taxonómicos (Clase, Orden, Suborden, Tipo).  
Si estos datos estuvieran directamente en la tabla `Raza`, se repetirían para cada reptil, generando redundancia.

---

### Tabla `Raza`
Representa cada reptil específico.  
Se relaciona con:
- **`Clasificación`** para conocer su grupo taxonómico.  
- **`Estatus`** para saber su nivel de conservación.

---

### Tabla `Estatus`
Define categorías oficiales de conservación:
- Extinto  
- Peligro Crítico  
- Extinto en Estado Silvestre  
- Datos Insuficientes  
- No Evaluado

---

### Tabla `Características`
Contiene atributos físicos como:
- Peso  
- Tamaño  
- Patrón  
- Tipo de piel  

Separarla evita que la tabla `Raza` tenga demasiadas columnas y permite modificar atributos sin afectar otras entidades.

---

### Tabla `Requerimiento_Ambiental`
Cada clasificación puede tener múltiples requerimientos, como:
- Temperatura  
- Humedad  
- Iluminación  

Si estos estuvieran en una sola fila, serían atributos multivaluados, lo que se evita con esta tabla.

---

### Tabla `Propietario`
Permite registrar los dueños de los reptiles.

---

### Tabla `Mascota`
Vincula reptiles específicos con sus propietarios.

---

## **RESTRICCIONES:**

- **`NOT NULL`** :garantiza que los datos esenciales (nombre, clasificación, raza) siempre estén presentes.  
- **`CHECK`** :evita valores inválidos (por ejemplo, peso negativo o sexo incorrecto).  
- **`DEFAULT`** :asigna valores comunes automáticamente (`'Escamas'`, `fecha_registro = CURRENT_DATE`).

---

## **COMANDOS EN LINUX**

A continuación se muestra el SQL que define las tablas y sus relaciones:

```sql

-- Tabla de Clasificación
CREATE TABLE clasificacion (
    id_clasif INT PRIMARY KEY,
    clase_clasif VARCHAR(50) NOT NULL,
    orden_clasif VARCHAR(50) NOT NULL,
    suborden_clasif VARCHAR(50),
    tipo_clasif VARCHAR(50) NOT NULL
);

-- Tabla de Estatus de Conservación
CREATE TABLE estatus (
    id_estatus INT PRIMARY KEY,
    categoria VARCHAR(100) NOT NULL
        CHECK (categoria IN ('Extinto', 'Extinto en Estado Silvestre', 'Peligro Crítico', 'Datos Insuficientes', 'No Evaluado'))
);

-- Tabla de Raza
CREATE TABLE raza (
    id_raza INT PRIMARY KEY,
    id_clasificacion INT NOT NULL,
    id_estatus INT,
    nombre_comun VARCHAR(100) NOT NULL,
    nombre_cientifico VARCHAR(100),
    FOREIGN KEY (id_clasificacion) REFERENCES clasificacion(id_clasif),
    FOREIGN KEY (id_estatus) REFERENCES estatus(id_estatus)
);

-- Tabla de Características
CREATE TABLE caracteristicas (
    id_caracteristica INT PRIMARY KEY,
    id_raza INT NOT NULL,
    tamano_tipico DECIMAL(5,2) CHECK (tamano_tipico > 0),
    peso_tipico DECIMAL(6,2) CHECK (peso_tipico > 0),
    patron VARCHAR(50) CHECK (patron IN ('Criptismo', 'Aposematismo', 'Exhibición')),
    piel VARCHAR(50) DEFAULT 'Escamas' NOT NULL,
    FOREIGN KEY (id_raza) REFERENCES raza(id_raza)
);

-- Tabla de Requerimientos Ambientales
CREATE TABLE requerimiento_ambiental (
    id_req INT PRIMARY KEY,
    id_clasif INT NOT NULL,
    factor_req VARCHAR(100) NOT NULL,
    detalle_req TEXT NOT NULL,
    FOREIGN KEY (id_clasif) REFERENCES clasificacion(id_clasif)
);

-- Tabla de Propietario
CREATE TABLE Propietario (
    id_prop INT PRIMARY KEY,
    nombre_prop VARCHAR(100) NOT NULL,
    telefono_prop VARCHAR(15),
    direccion_prop VARCHAR(200)
);

-- Tabla de Mascota
CREATE TABLE mascota (
    id_mascota INT PRIMARY KEY,
    nombre_mascota VARCHAR(100) NOT NULL,
    edad_mascota INT CHECK (edad_mascota >= 0),
    sexo_mascota CHAR(1) CHECK (sexo_mascota IN ('M','F')),
    fecha_registro DATE DEFAULT CURRENT_DATE,
    id_raza INT NOT NULL,
    id_prop INT NOT NULL,
    FOREIGN KEY (id_raza) REFERENCES raza(id_raza),
    FOREIGN KEY (id_prop) REFERENCES propietario(id_prop)
);
