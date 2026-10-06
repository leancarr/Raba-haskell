# Resumen de Contexto y Aprendizaje (EDyA2 / ALP)
**Materia:** Estructuras de Datos y Algoritmos 2 / Análisis de Lenguajes de Programación
**Profesor:** Juan Manuel Rabasedas
**Estudiante:** Lean Carr
**Última actualización:** 06-10-2026

---

## 📚 Bloque 1: Análisis de Algoritmos
- **Método de Sustitución:** Demostraciones rigurosas por inducción para O y Ω. Entendimiento profundo de que las constantes $c$ no pueden depender de la variable de recursión, evaluando en límites máximos o mínimos según corresponda.
- **Árboles de Recurrencia:** Construcción visual del costo total de un algoritmo nivel por nivel. Diferenciación entre el costo de un nodo individual y el "Costo Total por Nivel" al armar la sumatoria.
- **Teorema Maestro:** Aplicación y selección rápida de casos comparando $f(n)$ contra $n^{\log_b a}$.

---

## 🧩 Bloque 2: Sintaxis y Parsers Funcionales Monádicos
Nos enfocamos en construir parsers funcionales en Haskell (basados en listas de éxitos / Mónada State) mediante la combinación de primitivas básicas.

### Conceptos Clave Consolidados:
1. **Sintaxis Abstracta (AST) vs Concreta:**
   Los corchetes `[]`, comas y comillas son sintaxis concreta que el parser consume (se "come" de la cinta) para estructurar los datos, pero no forman parte del tipo de dato final retornado (AST).
2. **Polimorfismo en Listas Heterogéneas:**
   Las listas en Haskell son estrictamente homogéneas. Para parsear `[3, 'z']`, definimos un tipo de dato algebraico (AST) envoltorio: `data Elem = Entero Int | Caracter Char`.
3. **Mecánica del operador de Elección (`<|>`):**
   Evalúa de izquierda a derecha de forma estricta. Si intenta parsear un entero y choca contra una comilla simple (`'`), falla de inmediato sin consumir el contenido y salta a la alternativa de la derecha, evitando ambigüedades.
4. **Comportamiento real de `item`:**
   En los parsers funcionales no existe el concepto de "mirar" (*peek*) sin tocar. `item` siempre **consume** el primer caracter de la cinta y lo **devuelve**. Este es el motor interno de funciones condicionales como `sat p`.

### Ejercicios de Práctica Resueltos:
- **Ex 1 & 2 (`conParentesis` y `withQuotes`):** Parsers que descartan delimitadores (paréntesis o comillas simples/dobles) y extraen el contenido interno usando bloques `do`.
- **Ex 3 (`sepList`):** Implementación de una mega-elección de separadores dinámicos plegando una lista de parsers con `foldr1 (<|>)`.
- **Ex 4 (`Tuple`):** Parseo de tuplas anidadas matemáticas `Single Int | Pair Tuple Tuple` valiéndose de la magia de las llamadas recursivas para evitar búsquedas hacia adelante (*lookahead*).
- **Ex 5 (`Interval`):** Mapeo de corchetes/paréntesis de intervalos matemáticos a constructores de *Record Syntax* usando booleanos y parsers auxiliares (`parseLeft`).
- **Ex 6 (`linspace`):** Parseo sintáctico trivial emparejado con la evaluación semántica y el uso de listas perezosas (`Range Syntax`) de Haskell para generar rangos equiespaciados.
- **Ex 7 (Listas Heterogéneas):** 
  - Creación del tipo `Elem`.
  - Diferenciación en lectura de enteros vs caracteres crudos.
  - **Construcción manual de `sepBy`:** Implementación artesanal para procesar listas separadas por comas utilizando exclusivamente el combinador `many` y la recursión implícita de una forma a prueba de fallos:
    ```haskell
    parseElementos = do e <- parseElem
                        es <- many (do char ','; parseElem)
                        return (e:es)
                     <|> return []
    ```
