# EP1-ISY0101-Agente-RAG-El-Almendro

Proyecto desarrollado para la Evaluación Parcial N°1 de la asignatura **Ingeniería de Soluciones con Inteligencia Artificial (ISY0101)** de la carrera Ingeniería en Informática de Duoc UC.

La solución corresponde a un prototipo de agente inteligente basado en **LLM y Retrieval-Augmented Generation (RAG)** para apoyar al área de Obras, Mantención y Mejoras (OMM) de Hotel El Almendro.

## Integrantes

- Matias Loyola 
- Matias Cisternas

## Docente

Francisca Valenzuela

## Descripción del problema

El área de Obras, Mantención y Mejoras de Hotel El Almendro debe consultar información técnica distribuida en distintos documentos, como manuales PDF, fichas técnicas, protocolos y documentación de fabricantes.

La búsqueda manual de esta información puede aumentar el tiempo necesario para resolver consultas técnicas.

Como propuesta se implementó un prototipo RAG capaz de recibir una pregunta en lenguaje natural, recuperar fragmentos relevantes desde documentos técnicos y generar una respuesta utilizando únicamente el contexto recuperado.

## Objetivo

Diseñar e implementar un agente inteligente de soporte técnico que permita consultar documentación mediante lenguaje natural, recuperando información relevante y entregando respuestas acompañadas de su fuente.

El agente busca:

- Recuperar información mediante búsqueda semántica.
- Generar respuestas basadas en documentación técnica.
- Identificar el documento y página utilizados como fuente.
- Evitar completar respuestas utilizando información externa al contexto.
- Indicar cuando no existe información suficiente en los documentos.

## Arquitectura utilizada

El flujo implementado es el siguiente:

```text
Documento PDF
      |
      v
PyPDF
      |
      v
RecursiveCharacterTextSplitter
chunk_size = 350
chunk_overlap = 50
      |
      v
mistral-embed
      |
      v
FAISS
      |
      v
Consulta del usuario
      |
      v
mistral-embed
      |
      v
Top-3 fragmentos relevantes
      |
      v
Prompt OMM
      |
      v
ministral-8b-2512
      |
      v
Respuesta + fuente + estado de respaldo
```

## Tecnologías utilizadas

- Python
- Google Colab
- LangChain
- PyPDF
- FAISS
- NumPy
- API de Mistral AI

### Modelos utilizados

**mistral-embed**

Utilizado para generar las representaciones vectoriales de los fragmentos de los documentos y de las consultas realizadas por el usuario.

**ministral-8b-2512**

Utilizado como modelo de lenguaje para generar la respuesta final a partir del contexto recuperado.

## Archivo principal

El código principal del proyecto se encuentra en:

```text
EP1_El_Almendro_RAG MM Solutions.ipynb
```

El notebook contiene el flujo completo de:

1. Instalación de dependencias.
2. Configuración de credenciales.
3. Configuración de los modelos.
4. Carga de documentos PDF.
5. Extracción del contenido.
6. Segmentación de texto.
7. Generación de embeddings.
8. Creación del índice FAISS.
9. Recuperación semántica.
10. Configuración del prompt.
11. Generación de respuestas mediante RAG.
12. Ejecución de pruebas.

## Requisitos para ejecutar el proyecto

Para ejecutar el notebook se necesita:

- Una cuenta de Google.
- Acceso a Google Colab.
- Una API Key válida de Mistral AI.
- Un documento técnico en formato PDF.

No es necesario instalar Python localmente, ya que el proyecto puede ejecutarse completamente mediante Google Colab.

## Configuración de la API Key

La API Key no se encuentra almacenada directamente en el código.

Para ejecutar el proyecto:

1. Abrir el notebook en Google Colab.
2. Seleccionar la opción **Secrets** ubicada en el panel izquierdo.
3. Crear un nuevo secreto con el nombre:

```text
LLM_API_KEY
```

4. Ingresar como valor una API Key válida de Mistral AI.
5. Activar la opción **Acceso desde el cuaderno**.

El notebook obtiene la credencial mediante:

```python
from google.colab import userdata

LLM_API_KEY = userdata.get("LLM_API_KEY")
```

De esta forma la clave no queda almacenada dentro del notebook ni del repositorio.

## Ejecución del notebook

Después de configurar el Secret `LLM_API_KEY`, ejecutar las celdas del notebook en orden.

### 1. Instalación de dependencias

La primera celda instala automáticamente las librerías necesarias para el funcionamiento del proyecto.

### 2. Configuración de Mistral

El sistema configura:

```text
Modelo de generación: ministral-8b-2512
Modelo de embeddings: mistral-embed
```

### 3. Carga del documento

Al ejecutar la sección:

```text
1. Carga de documentos PDF
```

Google Colab solicitará seleccionar un documento desde el computador.

Se recomienda utilizar un manual técnico en PDF con contenido de texto seleccionable.

### 4. Procesamiento

El sistema:

- Extrae el texto utilizando PyPDF.
- Conserva el nombre del documento y número de página.
- Divide el contenido mediante `RecursiveCharacterTextSplitter`.
- Utiliza un tamaño inicial de 350 caracteres.
- Mantiene un solapamiento de 50 caracteres.
- Genera embeddings mediante `mistral-embed`.

### 5. Base vectorial

Los embeddings generados son almacenados en un índice FAISS.

FAISS permite comparar la consulta del usuario con los fragmentos almacenados mediante similitud semántica.

### 6. Recuperación

Para cada pregunta se recuperan los tres fragmentos más relevantes:

```text
Top-k = 3
```

Los fragmentos recuperados posteriormente son utilizados como contexto para el modelo de lenguaje.

### 7. Generación

El modelo `ministral-8b-2512` recibe:

- La pregunta del usuario.
- Los fragmentos recuperados.
- El prompt restrictivo del agente OMM.

El modelo se encuentra configurado con:

```text
temperature = 0.1
```

## Prompt utilizado

El agente utiliza el siguiente criterio principal:

```text
Eres un asistente técnico del área de Obras, Mantención y Mejoras (OMM) de Hotel El Almendro.

Responde únicamente utilizando la información incluida en el CONTEXTO recuperado.
No utilices conocimiento general para completar información que no aparezca en los documentos.
No inventes procedimientos, valores, frecuencias, repuestos, riesgos ni especificaciones técnicas.

Si el contexto no contiene información suficiente para responder, indica:
"No existe información suficiente en la documentación disponible".

Si existe información suficiente, responde de forma breve y clara e identifica el documento utilizado como fuente.

Si existen datos contradictorios entre documentos, informa la contradicción y no selecciones una respuesta sin evidencia de vigencia.
```

La salida esperada sigue la estructura:

```text
Respuesta:
Fuente:
Estado de respaldo: suficiente / insuficiente
```

## Documento utilizado para las pruebas

Para comprobar técnicamente el funcionamiento del prototipo se utilizó temporalmente un manual técnico público de una caldera Rinnai.

Este documento se utilizó únicamente para validar la implementación mientras se obtiene documentación autorizada del Hotel El Almendro.

El PDF de prueba no es necesario para ejecutar el sistema. El evaluador puede cargar cualquier documento técnico compatible en formato PDF.

## Resultados de las pruebas

Durante la prueba realizada se obtuvo:

```text
Páginas con contenido: 32
Chunks generados: 52
Vectores almacenados en FAISS: 52
Dimensión de los embeddings: 1024
```

### Prueba 1 - Información existente

Se realizó una consulta sobre las acciones necesarias antes de realizar limpieza o mantenimiento de la caldera.

El sistema recuperó información de la página 12 del manual con una similitud aproximada de:

```text
0.9089
```

El agente respondió utilizando el contenido recuperado e identificó el documento y página utilizados como fuente.

### Prueba 2 - Información inexistente

Se realizó una consulta sobre el precio actual de una caldera nueva.

Debido a que esa información no se encontraba en el documento cargado, el agente indicó:

```text
No existe información suficiente en la documentación disponible.
```

y estableció el estado de respaldo como:

```text
insuficiente
```

### Prueba 3 - Recuperación semántica

Se realizó una consulta relacionada con la limpieza del filtro de calefacción.

FAISS recuperó tres fragmentos semánticamente relacionados y encontró dentro del Top-3 la sección correspondiente a Limpieza y Mantenimiento del manual.

## Evidencias

Las evidencias de las pruebas realizadas pueden encontrarse en el notebook ejecutado y en la documentación incluida en este repositorio.

Se consideran como evidencias:

- Generación de chunks.
- Creación de embeddings.
- Creación del índice FAISS.
- Recuperación de fragmentos.
- Valores de similitud.
- Respuestas generadas.
- Documento y página utilizados como fuente.
- Comportamiento frente a información inexistente.

## Estructura del repositorio

```text
EP1-ISY0101-Agente-RAG-El-Almendro/
│
├── README.md
├── .gitignore
│
├── EP1_El_Almendro_RAG MM Solutions.ipynb
│
└── Documentos/
    ├── Informe_RAG_El_Almendro.pdf
    └── Manual-Caldera-No-Condensacion-Chile.pdf
```

La estructura puede complementarse con una carpeta de evidencias de pruebas.

## Consideraciones de seguridad

Este repositorio no almacena claves API directamente en el código.

Las credenciales necesarias para utilizar Mistral deben configurarse mediante Google Colab Secrets.

No se deben subir al repositorio archivos que contengan:

```text
.env
API Keys
tokens
credenciales
contraseñas
documentación confidencial de la empresa
```

La documentación interna de Hotel El Almendro solo debe incorporarse al prototipo cuando exista autorización para su uso.

## Limitaciones actuales

Esta implementación corresponde a un prototipo académico.

Actualmente:

- El sistema trabaja principalmente con documentos PDF.
- La prueba técnica fue realizada con un manual público.
- La recuperación utiliza Top-k = 3.
- La evaluación realizada es principalmente funcional y cualitativa.
- La documentación interna de Hotel El Almendro será incorporada posteriormente cuando se encuentre disponible y autorizada.

## Trabajo futuro

Como posibles mejoras se consideran:

- Incorporar documentación técnica real autorizada del área OMM.
- Procesar múltiples manuales simultáneamente.
- Incorporar filtros por tipo de equipo o área.
- Mejorar la gestión de metadatos.
- Evaluar formalmente métricas de RAG.
- Incorporar nuevas fuentes documentales.
- Comparar distintas configuraciones de chunking y recuperación.
