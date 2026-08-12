# Enriquecimiento de grafos de conocimiento RDF basado en reutilización y modularización

Trabajo de Fin de Grado — Universidad de Murcia  
Autor: Miguel  
Tutor: Jesualdo Tomás Fernández-Breis

---

## Descripción

Aplicación Java que enriquece semánticamente grafos de conocimiento RDF mediante extracción modular de ontologías OWL. Partiendo de un grafo RDF, la aplicación detecta las clases de ontologías externas referenciadas implícitamente, extrae los axiomas relevantes de dichas ontologías usando las estrategias STAR, BOT y TOP de OWL API, y genera un grafo RDF enriquecido combinando los datos originales con la información semántica extraída.

---

## Requisitos

- Java JDK 26.0.1 o superior
- Apache Maven 3.9.11 o superior
- Mínimo 8GB de RAM disponibles para la JVM (recomendado para ontologías grandes)

---

## Instalación

### 1. Clonar el repositorio

```bash
git clone https://github.com/[usuario]/[repositorio].git
cd [repositorio]
```

### 2. Descargar las ontologías

Las ontologías OWL no están incluidas en el repositorio por su tamaño. Descárgalas y colócalas en `src/main/resources/ontologies/`:

| Ontología | URL de descarga |
|---|---|
| SO (Sequence Ontology) | http://purl.obolibrary.org/obo/so.owl |
| NCIT (NCI Thesaurus) | http://purl.obolibrary.org/obo/ncit.owl |
| CL (Cell Ontology) | http://purl.obolibrary.org/obo/cl.owl |
| CLO (Cell Line Ontology) | http://purl.obolibrary.org/obo/clo.owl |
| UBERON | http://purl.obolibrary.org/obo/uberon.owl |
| BTO (BRENDA Tissue Ontology) | http://purl.obolibrary.org/obo/bto.owl |
| ECO (Evidence & Conclusion Ontology) | http://purl.obolibrary.org/obo/eco.owl |
| MI (Molecular Interactions) | http://purl.obolibrary.org/obo/mi.owl |
| OBI (Ontology for Biomedical Investigations) | http://purl.obolibrary.org/obo/obi.owl |
| EFO (Experimental Factor Ontology) | https://www.ebi.ac.uk/efo/efo.owl |
| BAO (BioAssay Ontology) | http://www.bioassayontology.org/bao/bao_complete.owl |
| NCBITaxon | http://purl.obolibrary.org/obo/ncbitaxon.owl |

> **Nota**: NCBITaxon pesa 1.8GB y puede causar problemas de memoria. Se recomienda omitirla o usar un timeout alto.

### 3. Compilar el proyecto

```bash
mvn clean package
```

Esto generará el fichero `target/tfg.jar`.

---

## Uso

```bash
java -Xmx8g -jar target/tfg.jar <rdfFile> <outputDir> <timeoutSeconds> [--s=STAR,BOT,TOP] [--p=type,subclass,definedby,relation]
```

### Parámetros

| Parámetro | Descripción | Obligatorio |
|---|---|---|
| `rdfFile` | Ruta al fichero RDF de entrada (`.ttl` o `.nt`) | Sí |
| `outputDir` | Directorio de salida para los grafos enriquecidos y el log | Sí |
| `timeoutSeconds` | Timeout en segundos por ontología | Sí |
| `--s` | Estrategias a ejecutar: `STAR`, `BOT`, `TOP` (por defecto las tres) | No |
| `--p` | Patrones de detección: `type`, `subclass`, `definedby`, `relation` (por defecto los cuatro) | No |

### Ejemplos

Ejecutar con todas las estrategias y todos los patrones:
```bash
java -Xmx8g -jar target/tfg.jar tfbs.nt salidas/ 3600
```

Ejecutar solo con STAR y BOT, usando solo `rdf:type` y `rdfs:subClassOf`:
```bash
java -Xmx8g -jar target/tfg.jar tfbs.nt salidas/ 3600 --s=STAR,BOT --p=type,subclass
```

---

## Patrones de detección

La aplicación detecta clases de ontologías externas referenciadas implícitamente mediante cuatro patrones:

| Patrón | Predicado | Descripción |
|---|---|---|
| `type` | `rdf:type` | Clases usadas como tipo de una instancia |
| `subclass` | `rdfs:subClassOf` | Clases usadas como superclase |
| `definedby` | `rdfs:isDefinedBy` | Clases que indican la ontología de definición |
| `relation` | `TXPO_0003500`, `RO_0002162` | Predicados de relación de dominio cuyo objeto es una clase externa |

---

## Salida

Para cada estrategia ejecutada se generan los siguientes ficheros en el directorio de salida:

- `enriched_star.ttl`: grafo RDF enriquecido con la estrategia STAR
- `enriched_bot.ttl`: grafo RDF enriquecido con la estrategia BOT
- `enriched_top.ttl`: grafo RDF enriquecido con la estrategia TOP
- `execution_log.txt`: log de ejecución con métricas por ontología y estrategia

---

## Evaluación de calidad

Para evaluar la calidad de los grafos enriquecidos se utiliza el framework OQuaRE-KG, con métricas adaptadas para reconocer los cuatro patrones de detección utilizados.

```bash
# Clonar OQuaRE-KG
git clone https://github.com/tecnomod-um/oquare-kg

# Instalar dependencias
pip install rdflib

# Ejecutar evaluación
python evaluate.py tfbs.nt enriched_star.ttl enriched_bot.ttl enriched_top.ttl
```

> **Nota**: Las métricas de OQuaRE-KG han sido adaptadas para reconocer los cuatro patrones de detección de este trabajo. Los ficheros adaptados se encuentran en `oquare-kg/framework/metricsCode/`.

---

## Estructura del proyecto

```
src/
├── main/
│   ├── java/
│   │   ├── app/
│   │   │   ├── Main.java
│   │   │   └── Principal.java
│   │   ├── enrichment/
│   │   │   └── KGEnricher.java
│   │   ├── extraction/
│   │   │   ├── JobExecutor.java
│   │   │   ├── ModuleExtractionTask.java
│   │   │   └── OntologyResolver.java
│   │   ├── logger/
│   │   │   └── LoggerManager.java
│   │   ├── metrics/
│   │   │   └── MetricsManager.java
│   │   └── rdf/
│   │       ├── OntologyMapper.java
│   │       ├── RDFLoader.java
│   │       └── RDFTypeExtractor.java
│   └── resources/
│       └── ontologies/     ← coloca aquí los ficheros OWL descargados
evaluate.py                 ← script de evaluación de calidad
README.md
```

---

## Casos de estudio

Los grafos utilizados como casos de estudio pertenecen al repositorio [cisreg](https://github.com/juan-mulero/cisreg), publicado en el contexto del proyecto BioGateway:

> Mulero-Hernández, J. et al. (2024). Integration of chromosome locations and functional aspects of enhancers and topologically associating domains in knowledge graphs enables versatile queries about gene regulation. *Nucleic Acids Research*, 52(15), e69.

---

## Licencia

Este proyecto ha sido desarrollado como Trabajo de Fin de Grado en la Universidad de Murcia.
