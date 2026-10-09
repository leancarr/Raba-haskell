# Resumen Teórico Consolidado: Sintaxis y Parsers Funcionales
**Materia / Contexto:** Estructuras de Datos y Algoritmos / Análisis de Lenguajes de Programación (Haskell)  
**Profesor:** Juan Manuel Rabasedas  
**Referencias:**
- *Types and Programming Languages* (B.C. Pierce)
- *Programming in Haskell* (G. Hutton, Cap. 8)
- *Foundations of Programming Languages* (J.C. Mitchell)
- *Modern Compiler Implementation in ML* (A.W. Appel)
- *Introducción a la Programación Funcional con Haskell* (Richard Bird)

---

## PARTE 1: SINTAXIS (Lenguajes Formales, GLC, Árboles y AST)

### 1. Sintaxis Concreta vs Sintaxis Abstracta
- **Sintaxis:** Es la forma externa de un lenguaje.
- **Sintaxis Concreta:**
  - Secuencia exacta de caracteres (texto plano, tokens).
  - Incluye detalles sintácticos superficiales: espacios en blanco, paréntesis, separadores, palabras clave.
  - Ejemplo: Las cadenas `"1 + 2 + 3"`, `"(1 + 2) + 3"` y `" (1 + 2) \n + 3 "` son **distintas** en sintaxis concreta.
  - Los árboles de derivación sobre sintaxis concreta pueden ser ambiguos si la gramática no está bien formulada.
- **Sintaxis Abstracta:**
  - Estructura esencial, semántica y jerárquica del programa (un árbol).
  - Descarta detalles irrelevantes como formato o paréntesis de agrupamiento.
  - Ejemplo: En sintaxis abstracta, `1 + 2 + 3` y `(1 + 2) + 3` representan **el mismo árbol sintáctico (AST)** si `+` asocia a izquierda:
    $$\text{Sum}(\text{Sum}(1, 2), 3)$$
  - **Propiedad fundamental:** La sintaxis abstracta *ya es un árbol*, por lo que **no puede tener problemas de ambigüedad**.

---

### 2. Lenguajes Formales
- **Alfabeto ($\Sigma$):** Conjunto finito no vacío de símbolos (ej. $\Sigma = \{a, b\}$ o caracteres ASCII).
- **Palabra o Cadena:** Secuencia finita de símbolos.
  - $\varepsilon$ denota la palabra vacía (secuencia de 0 símbolos).
- **Clausura de Kleene ($\Sigma^*$):** Conjunto de todas las palabras posibles sobre $\Sigma$.
  - Definición inductiva:
    1. $\varepsilon \in \Sigma^*$
    2. Si $s \in \Sigma$ y $w \in \Sigma^*$, entonces $sw \in \Sigma^*$
  - Un alfabeto finito $\Sigma$ genera infinitas palabras $\Sigma^*$.
- **Lenguaje ($L$):** Cualquier subconjunto de palabras sobre el alfabeto:
  $$L \subseteq \Sigma^* \quad \text{o equivalentemente} \quad L \in \mathcal{P}(\Sigma^*)$$
  - Ejemplo: Un lenguaje de programación como C es el conjunto de programas sintácticamente válidos sobre el alfabeto ASCII.

---

### 3. Gramáticas Libres de Contexto (GLC / CFG)
Permiten describir lenguajes con estructuras anidadas, recursivas y paréntizadas (reconocibles por un autómata de pila).

- **Definición formal:** Una GLC es una tupla:
  $$G = (N, T, P, S)$$
  - $N$: Conjunto finito de símbolos **no terminales** (metavariables sintácticas).
  - $T$: Conjunto finito de símbolos **terminales** (alfabeto de entrada), con $N \cap T = \emptyset$.
  - $S \in N$: Símbolo **inicial**.
  - $P$: Conjunto finito de **producciones** de la forma:
    $$A \to \alpha \quad \text{donde } A \in N, \, \alpha \in (N \cup T)^*$$
- **Notación BNF (Backus-Naur Form):**
  Agrupa producciones para un mismo no terminal:
  $$S \to \varepsilon \mid aA \quad \text{(o también } S ::= \varepsilon \mid aA \text{)}$$

#### Relación de Derivación
- **Derivación directa ($\Rightarrow$):** Menor relación binaria sobre cadenas de $(N \cup T)^*$ tal que:
  $$\alpha A \gamma \Rightarrow \alpha \beta \gamma \quad \text{cuando } A \to \beta \in P$$
- **Derivación ($\Rightarrow^*$):** Clausura reflexiva y transitiva de $\Rightarrow$:
  1. $\alpha \Rightarrow^* \alpha$
  2. Si $\alpha \Rightarrow \beta$, entonces $\alpha \Rightarrow^* \beta$
  3. Si $\alpha \Rightarrow^* \gamma$ y $\gamma \Rightarrow^* \beta$, entonces $\alpha \Rightarrow^* \beta$
- **Forma Sentencial:** Cualquier cadena $\alpha \in (N \cup T)^*$ alcanzable desde la raíz: $S \Rightarrow^* \alpha$.
- **Lenguaje generado $L(G)$:** Cadena terminales derivadas desde el símbolo inicial:
  $$L(G) = \{ w \in T^* \mid S \Rightarrow^* w \}$$

---

### 4. Árboles de Parseo (Árboles de Derivación Concreta)
Un árbol es un árbol de parseo para $G = (N, T, P, S)$ sii:
1. Cada nodo tiene una etiqueta en $N \cup T \cup \{\varepsilon\}$.
2. La raíz tiene etiqueta $S$.
3. Los nodos internos tienen etiquetas en $N$.
4. Si un nodo $n$ tiene etiqueta $A$ y sus hijos de izq. a der. tienen etiquetas $X_1, X_2, \dots, X_k$, entonces $A \to X_1 X_2 \dots X_k \in P$.
5. Si un nodo tiene etiqueta $\varepsilon$, es una hoja y es hijo único.
- **Rendimiento / Yield:** La concatenación de las hojas de izquierda a derecha es el resultado del árbol (una forma sentencial de $G$).

---

### 5. Ambigüedad en Gramáticas
- **Definición:** Una GLC $G$ es **ambigua** si existe al menos una palabra $w \in L(G)$ que posee **más de un árbol de parseo distinto**.
- **Lenguaje Inherentemente Ambiguo:** Un lenguaje libre de contexto para el cual *toda* gramática posible que lo genere es ambigua (la mayoría de los lenguajes de programación *no* son inherentemente ambiguos).
- **Indecidibilidad:** En general, **no existe un algoritmo** para decidir si una GLC arbitraria es ambigua o no.
- **Desambiguación:** Se logra mediante convenciones de diseño gramatical (precedencia, asociatividad y estratificación de no terminales).

#### El clásico problema del "Dangling Else"
Gramática:
```text
Stmt -> if Expr then Stmt
      | if Expr then Stmt else Stmt
      | other
```
Para la entrada:
```text
if expr1 then if expr2 then stmt1 else stmt2
```
Existen 2 árboles posibles:
1. El `else` se asocia al **primer** `if` (externo).
2. El `else` se asocia al **segundo** `if` (más cercano).

**Regla de desambiguación estándar:** Asociar cada `else` al `then` más cercano que no esté ya asociado a otro `else`.  
**Solución por estratificación de reglas:**
```text
Stmt           -> MatchedStmt | UnmatchedStmt
MatchedStmt    -> if Expr then MatchedStmt else MatchedStmt
                | other
UnmatchedStmt  -> if Expr then Stmt
                | if Expr then MatchedStmt else UnmatchedStmt
```

---

### 6. Árboles de Sintaxis Abstracta (AST) y Definición de Términos
En la práctica funcional (Haskell), el AST se modela con tipos de datos algebraicos (`data`).

Ejemplo de expresiones enteras:
```haskell
data IntExp = Num Int
            | Sum IntExp IntExp
            | Prod IntExp IntExp
```

#### Tres Caracterizaciones Equivalentes del Conjunto de Términos ($\mathcal{T}$)
Consideremos la gramática abstracta:
$$t ::= \text{true} \mid \text{false} \mid 0 \mid \text{succ } t \mid \text{pred } t \mid \text{iszero } t \mid \text{if } t \text{ then } t \text{ else } t$$

1. **Definición Inductiva:**  
   $\mathcal{T}$ es el **menor conjunto** tal que:
   - $\{\text{true}, \text{false}, 0\} \subseteq \mathcal{T}$
   - Si $t_1 \in \mathcal{T}$, entonces $\{\text{succ } t_1, \text{pred } t_1, \text{iszero } t_1\} \subseteq \mathcal{T}$
   - Si $t_1, t_2, t_3 \in \mathcal{T}$, entonces $\text{if } t_1 \text{ then } t_2 \text{ else } t_3 \in \mathcal{T}$

2. **Por Reglas de Inferencia (Deducción Natural):**
   $$\overline{\text{true} \in \mathcal{T}} \quad \overline{\text{false} \in \mathcal{T}} \quad \overline{0 \in \mathcal{T}}$$
   $$\frac{t_1 \in \mathcal{T}}{\text{succ } t_1 \in \mathcal{T}} \qquad \frac{t_1 \in \mathcal{T}}{\text{pred } t_1 \in \mathcal{T}} \qquad \frac{t_1 \in \mathcal{T}}{\text{iszero } t_1 \in \mathcal{T}}$$
   $$\frac{t_1 \in \mathcal{T} \quad t_2 \in \mathcal{T} \quad t_3 \in \mathcal{T}}{\text{if } t_1 \text{ then } t_2 \text{ else } t_3 \in \mathcal{T}}$$

3. **Definición Concreta / Constructiva (Límite de Secuencia):**  
   Para cada $i \in \mathbb{N}$, se define $S_i$:
   - $S_0 = \emptyset$
   - $S_{i+1} = \{\text{true}, \text{false}, 0\} \cup \{\text{succ } t_1, \text{pred } t_1, \text{iszero } t_1 \mid t_1 \in S_i\} \cup \{\text{if } t_1 \text{ then } t_2 \text{ else } t_3 \mid t_1, t_2, t_3 \in S_i\}$
   - El conjunto total es:
     $$S = \bigcup_{i \in \mathbb{N}} S_i$$
   - **Teorema:** $\mathcal{T} \equiv S$.

#### Ejercicio de Conteo de Elementos en $S_i$:
Fórmula de recurrencia de cardinalidad:
$$|S_0| = 0$$
$$|S_{i+1}| = 3 + 3 \cdot |S_i| + |S_i|^3$$
- Para $i = 0$: $|S_1| = 3 + 3(0) + 0^3 = 3$ (los constantes: `true`, `false`, `0`).
- Para $i = 1$: $|S_2| = 3 + 3(3) + 3^3 = 3 + 9 + 27 = 39$.
- Para $i = 2$: $|S_3| = 3 + 3(39) + 39^3 = 3 + 117 + 59319 = \mathbf{59439}$.

---

## PARTE 2: PARSERS FUNCIONALES (Combinadores Monádicos en Haskell)

### 1. ¿Qué es un Parser?
Un parser toma un `String` (sintaxis concreta) y construye un árbol (sintaxis abstracta).

#### Evolución del tipo `Parser`:
1. `type Parser = String -> Tree` (muy restrictivo: asume que consume toda la entrada y solo produce árboles).
2. `type Parser = String -> (Tree, String)` (permite devolver el resto de la entrada no consumida).
3. `type Parser = String -> [(Tree, String)]` (agrega no-determinismo o fallo: `[]` = fallo, `[(res, resto)]` = éxito).
4. **Definición General y Polimórfica:**
   ```haskell
   type Parser a = String -> [(a, String)]
   ```
   - **Convención:** Consideramos parsers deterministas donde el resultado es `[]` si falla, o una lista con un único elemento `[(v, out)]` si tiene éxito.

---

### 2. Primitivas Básicas

```haskell
-- 1. Ejecutar un parser
parse :: Parser a -> String -> [(a, String)]
parse p inp = p inp

-- 2. Consumir exactamente un carácter arbitrario
item :: Parser Char
item = \inp -> case inp of
  []     -> []
  (x:xs) -> [(x, xs)]

-- 3. Parser que siempre falla
failure :: Parser a
failure = \inp -> []

-- 4. Parser que tiene éxito devolviendo v sin consumir entrada
return :: a -> Parser a
return v = \inp -> [(v, inp)]

-- 5. Alternativa determinista (elección orientada a la izquierda)
(<|>) :: Parser a -> Parser a -> Parser a
p <|> q = \inp -> case parse p inp of
  []        -> parse q inp
  [(v, out)] -> [(v, out)]
```

---

### 3. Secuenciación y la Mónada Parser
`Parser` es una mónada (posee `return` y bind `>>=`). En Haskell esto permite utilizar la notación `do`:
- Cada paso de la secuencia se ejecuta de izquierda a derecha.
- Los parsers intermedios consumen parte del String. Si uno falla devolviendo `[]`, toda la secuencia falla (`[]`).
- El valor devuelto por la última línea (`return expr`) es el resultado de toda la computación.
- Los valores intermedios se pueden ligar con `x <- p` o descartar si solo validan sintaxis.

Ejemplo:
```haskell
p :: Parser (Char, Char)
p = do
  x <- item
  item        -- descarta el segundo carácter
  y <- item
  return (x, y)

-- parse p "abcdef" => [(( 'a', 'c' ), "def")]
-- parse p "ab"     => []  (falla porque no hay 3 caracteres)
```

---

### 4. Primitivas Derivadas de Alto Nivel

```haskell
import Data.Char (isDigit, digitToInt)

-- Carácter que satisface un predicado
sat :: (Char -> Bool) -> Parser Char
sat p = do
  x <- item
  if p x then return x else failure

-- Dígito numérico
digit :: Parser Char
digit = sat isDigit

-- Carácter exacto
char :: Char -> Parser Char
char x = sat (x ==)

-- Cadena exacta
string :: String -> Parser String
string []     = return []
string (x:xs) = do
  char x
  string xs
  return (x:xs)

-- Cero o más repeticiones (clausura)
many :: Parser a -> Parser [a]
many p = many1 p <|> return []

-- Una o más repeticiones
many1 :: Parser a -> Parser [a]
many1 p = do
  v  <- p
  vs <- many p
  return (v:vs)
```

---

### 5. Gramáticas Aritméticas y Factorización

#### Gramática Clásica (Asociativa a Derecha, Precedencia `*` > `+`):
```text
expr   -> term '+' expr | term
term   -> factor '*' term | factor
factor -> digit | '(' expr ')'
digit  -> '0' | '1' | ... | '9'
```

#### ¿Por qué se factoriza la gramática?
En la regla:
```text
expr -> term '+' expr | term
```
Si se traduce directamente con `<|>`:
```haskell
expr = (do { t <- term; char '+'; e <- expr; return (t + e) }) <|> term
```
Si la entrada es sólo `"42"`, el primer parser ejecuta `term` (parsea `"42"`), busca un `'+'`, no lo encuentra y falla. Luego, el segundo parser vuelve a ejecutar `term` desde el inicio para parsear `"42"` otra vez.  
**Problema:** Backtracking y recomputación costosa (tiempo exponencial en expresiones anidadas).

#### Factorización por izquierda (Left Factoring):
Sacamos factor común `term` a la izquierda:
```text
expr -> term ('+' expr | ε)
term -> factor ('*' term | ε)
```
**Traducción directa a Haskell (eficiente, sin recomputación):**
```haskell
expr :: Parser Int
expr = do
  t <- term
  (do char '+'
      e <- expr
      return (t + e)) <|> return t

term :: Parser Int
term = do
  f <- factor
  (do char '*'
      t <- term
      return (f * t)) <|> return f

factor :: Parser Int
factor = (do d <- digit; return (digitToInt d))
     <|> (do char '('
             e <- expr
             char ')'
             return e)

eval :: String -> Int
eval xs = fst (head (parse expr xs))
```

---

### 6. El Problema de la Recursión Izquierda (Left Recursion)

#### El problema con operadores asociativos a izquierda
Para que operadores como `+`, `-`, `*`, `/` asocien a izquierda de forma natural, la gramática teórica se escribe con recursión izquierda:
```text
expr -> expr '+' term | expr '-' term | term
term -> term '*' factor | term '/' factor | factor
```
Si intentáramos traducir esto directamente en un parser recursivo descendente:
```haskell
expr = do
  t <- expr    -- ¡Llamada recursiva inmediata sin consumir caracteres!
  char '+'
  e <- term
  return (t + e)
  <|> term
```
**Fallo:** Provoca un **bucle infinito (infinite loop / stack overflow)**, porque `expr` se llama a sí mismo en la misma posición del string sin haber consumido ningún carácter de la entrada.

#### Regla General de Eliminación de Recursión Izquierda:
Dada una regla con recursión izquierda directa:
$$A \to A \alpha \mid \beta \quad (\text{donde } \beta \text{ no empieza con } A \text{ y } \alpha \neq \varepsilon)$$
Se transforma en:
$$A \to \beta A'$$
$$A' \to \varepsilon \mid \alpha A'$$

#### Aplicación a la Gramática Aritmética Completa:
Para:
$$expr \to expr ('+' term \mid '-' term) \mid term$$
Identificamos:
- $A = expr$
- $\beta = term$
- $\alpha = ('+' term \mid '-' term)$

Quedando:
$$\begin{aligned}
expr  &\to term \; expr' \\
expr' &\to \varepsilon \mid '+' term \; expr' \mid '-' term \; expr'
\end{aligned}$$

Y análogamente para $term$:
$$\begin{aligned}
term  &\to factor \; term' \\
term' &\to \varepsilon \mid '*' factor \; term' \mid '/' factor \; term'
\end{aligned}$$
