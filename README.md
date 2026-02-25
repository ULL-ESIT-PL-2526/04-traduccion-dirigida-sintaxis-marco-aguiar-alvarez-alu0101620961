# Syntax Directed Translation with Jison

Jison is a tool that receives as input a Syntax Directed Translation and produces as output a JavaScript parser  that executes
the semantic actions in a bottom up ortraversing of the parse tree.
 

## Compile the grammar to a parser

See file [grammar.jison](./src/grammar.jison) for the grammar specification. To compile it to a parser, run the following command in the terminal:
``` 
➜  jison git:(main) ✗ npx jison grammar.jison -o parser.js
```

## Use the parser

After compiling the grammar to a parser, you can use it in your JavaScript code. For example, you can run the following code in a Node.js environment:

```
➜  jison git:(main) ✗ node                                
Welcome to Node.js v25.6.0.
Type ".help" for more information.
> p = require("./parser.js")
{
  parser: { yy: {} },
  Parser: [Function: Parser],
  parse: [Function (anonymous)],
  main: [Function: commonjsMain]
}
> p.parse("2*3")
6
```


# Respuestas a las preguntas planteadas en el punto 3

## 3.1. Describa la diferencia entre /* skip whitespace */ y devolver un token.
La diferencia radica en la comunicación entre el analizador léxico (lexer) y el analizador sintáctico (parser):


- /* skip whitespace */: Cuando el lexer coincide con una regla que no tiene una sentencia return (o tiene un bloque vacío), simplemente consume los caracteres correspondientes de la entrada y continúa buscando el siguiente token. El parser nunca llega a saber que esos caracteres existieron.

- Devolver un token: Cuando se ejecuta return 'TOKEN_NAME', el lexer se detiene y envía una señal al parser indicando qué tipo de componente léxico ha encontrado (ej. NUMBER o OP). El parser utiliza esta información para aplicar las reglas de la gramática.

## 3.2. Escriba la secuencia exacta de tokens producidos para la entrada 123**45+@.
Siguiendo las reglas definidas en el bloque %lex, la secuencia exacta es:


1. NUMBER (correspondiente a 123) 

2. OP (correspondiente a **) 

3. NUMBER (correspondiente a 45) 

4. OP (correspondiente a +) 

5. INVALID (correspondiente al carácter @, ya que no encaja en ninguna regla anterior) 



EOF (fin de fichero)

## 3.3. Indique por qué ** debe aparecer antes que [-+*/].
En los analizadores léxicos generados por Jison, las reglas se evalúan en el orden de aparición (de arriba hacia abajo).

- Si el operador * estuviera definido en la regla [-+*/] antes que la regla de la potencia **, el lexer encontraría una coincidencia para el primer carácter * y devolvería un token OP inmediatamente.

- Esto haría imposible reconocer el operador de potencia **, ya que siempre se interpretaría como dos operadores de multiplicación seguidos. Al ponerlo primero, se asegura que el lexer intente encontrar la coincidencia más específica antes que la general.


## 3.4. Explique cuándo se devuelve EOF.
El token EOF (End Of File) se devuelve mediante la regla especial <\<EOF>>. Esta se activa cuando el analizador léxico llega al final de la cadena de entrada y no hay más caracteres que procesar. Es una señal crítica para que el analizador sintáctico sepa que la estructura de la expresión ha terminado y puede proceder a validar si la sentencia es completa y correcta según la gramática

## 3.5. Explique por qué existe la regla . que devuelve INVALID.
La regla del punto . actúa como un comodín de captura total (catch-all).

- Su función es capturar cualquier carácter individual que no haya sido reconocido por ninguna de las reglas anteriores (como símbolos especiales, letras o caracteres inesperados como @).

- Sin esta regla, si el lexer encuentra un carácter desconocido, el programa podría detenerse abruptamente o entrar en un estado de error no controlado. Al devolver INVALID, se permite que el sistema de manejo de errores del parser informe adecuadamente que se ha encontrado un símbolo no permitido en el lenguaje.