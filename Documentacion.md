# Documentacion de la implementacion.

![img](https://i.pinimg.com/564x/ba/e2/e6/bae2e669064306824b29365a3af446b6.jpg)

## First
Primero se implementa la funcion first
```python
def first(word: str) -> str:
    if not word:
        return Symbol('ε')
    return word[-1]
```
En esta implementacion se queria el simbolo mas a la derecha y si no hay simbolos devuelve epsilon.
Como se queria el simbolo mas a la derecha se ocupa ```[-1]``` que indica que se toma el primer elemento de derecha a izquierda.
## Prefix
En esta funcion lo que se queria es tomar todos los simbolos sin contar el ultimo.
```python
def prefix(word: Word) -> Word:
    if not word.symbols:  
        return Word([Symbol("ε")])
    return Word(word.symbols[:-1])
```
de manera analoga al anterior se tomo toda la subcadena pero se quita el ultimo elemento con ```[:-1]```
## Membership
```python
def membership(Word: Word, Language: Language)->bool:
    return str(Word)in Language
```
Esta funcion se implemento con ```in``` esto nos ayuda a identificar si un elemento se encuentra dentro de un conjunto.
## Clousure
En esta se implementa la clausura donde recibe un epsilon AFN, un estado y devulve la cerradura de epsilon.
```python
def clousure(nfae: NFAE, state: str) -> Set[str]:
    stack = [state]
    closure = set()
    while stack:
        estado_actual = stack.pop()
        closure.add(estado_actual)
        if CharWithEpsilon(None) in nfae.delta.get(estado_actual, {}):
            for siguiente_estado in nfae.delta[estado_actual][CharWithEpsilon(None)]:
                if siguiente_estado not in closure:
                    stack.append(siguiente_estado)
    return closure
```
Aqui se ocupo una pila para identificar los estados que son alcanzables, posteriormente identificamos si hay transiciones epsilon desde el estado actual, esto con un diccionario, en el caso de tener estados alcanzables los vamos iterando desde el estado actual, en el caso de que el siguiente estado ya no este en la clausura se añade a la pila.
## Accept
En esta funcion se recibe una cadena y un epsilon automata finito no determinista y decide si la cadena es aceptada por el epsilon automata finito no determinista
```python
def accept(nfae: NFAE, word: Word) -> bool:
    estados_actuales = clousure(nfae, nfae.init_state)
    for symbol in word.symbols:
        siguiente_estado = set()
        for state in estados_actuales:
            if CharWithEpsilon(str(symbol)) in nfae.delta.get(state, {}):
                siguiente_estado.update(nfae.delta[state][CharWithEpsilon(str(symbol))])
        estados_actuales = set()
        for state in siguiente_estado:
            estados_actuales.update(clousure(nfae, state))
    return bool(estados_actuales & nfae.final_states)
```

`estados_actuales = clousure(nfae, nfae.init_state)`
En esta funcion lo que se realizo primero fue identificar todos los estados en los que el automata se puede encontrar despues de recorrer las transiciones epsilon desde el estado inicial. Con la finalidad de tener los estados alcanzables.

Posteriormente en el ciclo `for` se recorre la cadena, si estamos en un estado actual, verificamos el conjunto de transiciones, y luego obtenemos un hashing para saber si hay transiciones para el estado actual, atualizando el conjunto `siguiente_estado`, procesamos el simbolo actual, e identificamos los estados alcanzables mediante transiciones esilon y para hacerlo de mejor manera es con la clausura.

Ensta funcion al final se verifica `bool(estados_actuales & nfae.final_states)` si en el estado actual es estado final
## Empty
Esta funcion devuelve el $\epsilon-AFN$ que acepta el lenguaje vacio
```python
def empty() -> NFAE:
    nfae = NFAE()
    nfae.set_delta({
        'q0': {}
    })
    nfae.set_init_state('q0')
    nfae.set_final_states(set())
    return nfae
```
En esta funcion primero se creo un objeto de $NFAE$, despues se realiza la conexion con delta del estado inicial a ningun otro estado y como estados finales no se especifican ninguno.

## Epsilon
```python
def epsilon() -> NFAE:
    nfae = NFAE()
    nfae.set_delta({
        'q0': {
            CharWithEpsilon(None): {'q1'}
        }
    })
    nfae.set_init_state('q0')
    nfae.set_final_states({'q1'})
    return nfae
```
En esta funcion primero se creo un objeto de $NFAE$, despues se realiza la conexion y con el caracter $\epsilon$ y se da por el constructor `CharWithEpsilon(None)` y con delta del estado inicial al estado final, donde el estado inicial es $q_0$ y el estado final es $q_1$ y se retorna el automata.

## Symbol
En esta funcion se recibe un simbolo $a$, y devuelve el $\epsilon-AFN$ que acepta la cadena $a$.
```python
def symbol(symbol: Symbol) -> NFAE:
    nfae = NFAE()
    nfae.set_delta({
        'q0': {
            CharWithEpsilon(str(symbol)): {'q1'}
        }
    })
    nfae.set_init_state('q0')
    nfae.set_final_states({'q1'})
    return nfae
```
Muy similar a como se implemento el que acepta la cadena $\epsilon$ pero aqui se envia del constructor un symbolo `CharWithEpsilon(str(symbol))`
## Union
En esta se reciben dos $\epsilon-AFN$ donde son $E_1,E_2$ y devuelve la union usando las transiciones $\epsilon$
```python
def union(nfae1: NFAE, nfae2: NFAE) -> NFAE:
    nfae = NFAE()
    nfae.set_init_state('q0')
    nfae.set_final_states(nfae2.final_states|nfae1.final_states)
    nfae.set_states(nfae1.states + nfae2.states)
    nfae.set_delta({
        'q0': {
            CharWithEpsilon(None): {nfae1.init_state, nfae2.init_state}
        }
    })
    for state, transiciones in nfae1.delta.items():
        nfae.delta[state] = transiciones.copy()
    for state, transiciones in nfae2.delta.items():
        if state in nfae.delta:
            nfae.delta[state].update(transiciones)
        else:
            nfae.delta[state] = transiciones
    return nfae
```
Primero se crea el objeto de la clase $NFAE$, despues se define el estado inicial y los estados finales como la union, de aqui se crea un nuevo estado $q_0$ como el automata finito no determinista de 1 y 2.

Posteriormente se incorporan las transiciones (automata1) delta al nuevo automata y luego del (automata2) se incluyen las transiciones faltantes actualizandolas.

Nota: Este automata solo aceptara las cadenas aceptadas por un automata o por otro.

## Concat
En esta funcion recibira dos funciones $\epsilon-AFN, E_1,E_2$ y devuelve la concatenacion usando transiciones epsilon.

```python
def concat(nfae1: NFAE, nfae2: NFAE) -> NFAE:
    nfae = NFAE()
    nfae.set_init_state(nfae1.init_state)
    nfae.set_final_states(nfae2.final_states)
    nfae.set_states(nfae1.states + nfae2.states)

    for estado, transiciones in nfae1.delta.items():
        nfae.delta[estado] = transiciones.copy()

    for estado, transiciones in nfae2.delta.items():
        if estado in nfae.delta:
            nfae.delta[estado].update(transiciones)
        else:
            nfae.delta[estado] = transiciones.copy()
            
    for estado in nfae1.final_states:
        if estado not in nfae.delta:
            nfae.delta[estado] = {}
        if CharWithEpsilon(None) in nfae.delta[estado]:
            nfae.delta[estado][CharWithEpsilon(None)].add(nfae2.init_state)
        else:
            nfae.delta[estado][CharWithEpsilon(None)] = {nfae2.init_state}
    
    return nfae
```
Igual que en todas las anteriores, primero se crea el objeto, luego se realiza como lo visto en clase que primero se ponia el automata1 se tomaba el estado inicial, el estado final se quiere conectar con el estado inical del automata2 y el estado final del nuevo automata es el estado final del automata2.

Posteriormente se pasan a copiar todas las tranciciones del automata1 y luego las del 2 pero en las del 2 se verifican si ya hay existentes con anterioridad transiciones y las actualiza o añade.

## Kleene
En esta funcion recibira un $\epsilon-AFN, E$ y devuelve la cerradura de Kleene aplicada a $E$ utilizando transciones epsilon.
```python
def kleene(nfae: NFAE) -> NFAE:
    nfae_kleene = NFAE()
    nfae_kleene.set_init_state('q0')
    nfae_kleene.set_final_states(nfae.final_states | {'q0'})
    nfae_kleene.set_states(nfae.states + ['q0'])
    
    nfae_kleene.set_delta({
        'q0': {
            CharWithEpsilon(None): {nfae.init_state}
        }
    })
    
    for estado, transiciones in nfae.delta.items():
        nfae_kleene.delta[estado] = transiciones.copy()
    
    for estado in nfae.final_states:
        if estado not in nfae_kleene.delta:
            nfae_kleene.delta[estado] = {}
        if CharWithEpsilon(None) in nfae_kleene.delta[estado]:
            nfae_kleene.delta[estado][CharWithEpsilon(None)].add(nfae.init_state)
        else:
            nfae_kleene.delta[estado][CharWithEpsilon(None)] = {nfae.init_state}
            
        nfae_kleene.delta[estado][CharWithEpsilon(None)].add('q0')
    
    return nfae_kleene
```
En esta implementacion se hace de manera analoga a las anteriores, primero se define el nuevo automata, posteriormente el conjunto de estados iniciales son los mismos que los del original, luego se establece un nuevo estado inicial $q_0$, con la incorporacion de este la idea radica en tener los mismos etados finales del automata mas este $q_0$.

Despues se implementa la transicion del $q_0$ al estado inicial del automata original, y asi mismo permitiendonos hacer repeticiones sobre la cadena.

Igual que en casos anteriores se copian las cadenas que hay en cada transicion. Y como se realizo en clase, se implementa la conexion del estado final del automata al estado inicial del automata original y al $q_0$

Finalmente se devuelve el nuevo automata con kleene.

## ToEAFN
Esta funcion recibe una expresion regular e y devuelve el $\epsilon-AFN, E$ tal que $L(e)=L(E)$
```python 
def ToEAFN(e: RE) -> NFAE:
    if isinstance(e, Empty):
        return empty()
    elif isinstance(e, Epsilon):
        return epsilon()
    elif isinstance(e, Sym):
        return symbol(Symbol(e.char))
    elif isinstance(e, Conc):
        nfae1 = ToEAFN(e.e1)
        nfae2 = ToEAFN(e.e2)
        return concat(nfae1, nfae2)
    elif isinstance(e, Union):
        nfae1 = ToEAFN(e.e1)
        nfae2 = ToEAFN(e.e2)
        return union(nfae1, nfae2)
    elif isinstance(e, Kleene):
        nfae = ToEAFN(e.e)
        return kleene(nfae)
    else:
        raise ValueError("No se reconoce esa expresion regular")
```
Nota: se hace uso de `isinstance` que esta definida como `def isinstance(obj: object, class_or_tuple: _ClassInfo, /) -> bool:` segun Visual Studio  ya que nos interesa verificar si un objeto es una instancia de una clase se adjunta el siguiente boton [link](https://interactivechaos.com/en/python/function/isinstance) donde se muestra la documentacion correspondiente.
Tras identificar a que instancia pertenece se retorna a su llamada de la funcion correspondiente