# Preprocesamiento de datos — Encuesta estudiantil

Notebook de análisis y limpieza de una encuesta institucional (Google Forms u origen similar), implementado en Python con **pandas** y **numpy**. El flujo replica un **ETL básico**: extracción desde CSV, transformación (renombrado de columnas, tratamiento de nulos, reglas condicionales) y carga a archivos CSV intermedios y finales.

**Repositorio:** [fabrizio-lanzetti/Preprocesamiento-de-datos](https://github.com/fabrizio-lanzetti/Preprocesamiento-de-datos)  
**Notebook principal:** [`encuesta_preprocesamiento.ipynb`](https://github.com/fabrizio-lanzetti/Preprocesamiento-de-datos/blob/main/encuesta_preprocesamiento.ipynb)

---

## Datos de entrada

- Archivo CSV de la encuesta (en el notebook original aparece como `encuestra_instituto.csv`).
- Dimensiones de referencia: **214 filas** y **45 columnas** (variables demográficas, movilidad, trabajo, formación, tecnología, opinión sobre horarios, texto abierto, etc.).

> **Nota:** La ruta de lectura en el notebook apunta a una carpeta local de Windows (`OneDrive\Desktop\...`). Para ejecutar el proyecto en otro entorno, conviene parametrizar la ruta o usar una variable de entorno / argumento.

---

## Requisitos

- Python **3.12** (según metadatos del notebook).
- Dependencias: `pandas`, `numpy`.

```bash
pip install pandas numpy
```

Opcional: Jupyter o [Google Colab](https://colab.research.google.com/) (el notebook incluye badge “Open in Colab”).

---

## Flujo del ETL

1. **Carga**  
   `pd.read_csv(...)` → inspección con `df.info()`.

2. **Partición**  
   El `DataFrame` se divide en dos bloques para trabajar con más claridad:
   - `df1`: primeras **22** columnas (datos personales, salud, movilidad, trabajo, becas).
   - `df2`: columnas **22 a 45** (carrera, hogar, escolaridad previa, tecnología, horarios, comentarios).

3. **Renombrado**  
   Los encabezados largos del formulario se mapean a nombres cortos y consistentes (por ejemplo `Genero_Nacer`, `Distancia_Instituto`, `Carreras_Inscripcion`, `Modificacion_Horario_Instituto`).  
   En `df2`, una columna con texto muy largo sobre modificación de horario se renombra de forma explícita si quedó sin mapear en el primer `rename`.

4. **Valores nulos — `df1`**  
   - Columnas ligadas al **trabajo**: si `Trabajas == "Sí"`, los faltantes se rellenan con etiquetas del tipo *No aclara* / *No tiene/No Aclara*; si no trabaja, se unifica con *No trabaja* u otras etiquetas coherentes.  
   - Otras variables: imputación por **moda** o valores fijos documentados (edad 18, distancia `0-10 kms`, beca *No cuento con ninguna beca*, etc.).

5. **Valores nulos — `df2`**  
   - Imputación por moda en carrera, año curricular, miembros del hogar, niveles educativos de padres/madre, etc.  
   - Si `Cursaste_Otra_Carrera == "Sí"`, se completan carrera previa, institución y estudios completados; en caso contrario se usa *Sin carrera previa* donde corresponde.  
   - Respuestas textuales con solo `"."` se normalizan a *No aclara*.

6. **Unión**  
   `pd.concat([df1, df2], axis=1)` → un único conjunto de **45** columnas alineadas por fila.

7. **Exportación**  
   - `encuestaP1.csv` — parte 1 limpia.  
   - `encuestaP2.csv` — parte 2 limpia.  
   - `encuestra_procesada.csv` — dataset completo (nombre con la misma variante *encuestra* que en el notebook).

---

## Salidas

| Archivo | Descripción |
|--------|-------------|
| `encuestaP1.csv` | Bloque 1 tras limpieza |
| `encuestaP2.csv` | Bloque 2 tras limpieza |
| `encuestra_procesada.csv` | `df1` + `df2` concatenados horizontalmente |

---

## Limitaciones y mejoras posibles

- Las imputaciones por **moda** y valores fijos son útiles para cerrar huecos, pero **sesgan** distribuciones y correlaciones; para análisis inferencial conviene documentarlas o usar otros criterios (eliminar, categoría “Desconocido”, modelos de imputación, etc.).
- Unificar criterios de texto (`Si` / `Sí`, mayúsculas) si se comparan respuestas entre columnas.
- Sustituir la ruta absoluta del CSV por configuración o `pathlib` relativo al proyecto.

---

Ajustá esta sección según la licencia del repositorio en GitHub. El contenido del notebook corresponde al trabajo de preprocesamiento de la encuesta estudiantil descrito en el repositorio enlazado arriba.
