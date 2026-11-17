# NumPy – Módulo 1: Arrays, shape, dtype e operações vetorizadas

Notas de estudo da trilha de NumPy, com explicações, exemplos de código, o exercício resolvido e um resumo para consulta rápida.

## 1. O que é um array NumPy e por que usar

O NumPy é a base de quase toda a análise de dados em Python, e o Pandas é construído em cima dele. O objeto central é o **ndarray** (array n-dimensional): uma "lista" otimizada para números.

Uma lista Python guarda qualquer coisa misturada (números, textos, outras listas), e cada elemento é um objeto separado na memória. O array NumPy guarda **elementos do mesmo tipo, juntos na memória**. Isso o torna muito mais rápido e permite fazer contas com a coleção inteira de uma vez.

```python
import numpy as np   # convenção: sempre importar como "np"
```

## 2. Criando arrays

### A partir de uma lista

```python
a = np.array([10, 20, 30])
print(a)          # [10 20 30]
print(type(a))    # <class 'numpy.ndarray'>
```

### Com funções prontas

```python
np.zeros(3)             # [0. 0. 0.]
np.ones((2, 3))         # [[1. 1. 1.]
                        #  [1. 1. 1.]]
np.arange(0, 10, 2)     # [0 2 4 6 8]  -> início, fim (exclusivo), passo
np.linspace(0, 1, 5)    # [0.   0.25 0.5  0.75 1.  ]  -> 5 pontos igualmente espaçados
```

- `arange` funciona como o `range` do Python, mas devolve um array.
- `linspace` significa "me dê N pontos entre A e B (inclusive)".

## 3. shape, ndim e size: o formato do array

```python
a = np.array([10, 20, 30])
b = np.array([[1, 2, 3],
              [4, 5, 6]])

print(a.shape)   # (3,)    -> 1 dimensão, 3 elementos
print(b.shape)   # (2, 3)  -> 2 linhas, 3 colunas

print(a.ndim)    # 1       -> número de dimensões
print(b.ndim)    # 2

print(b.size)    # 6       -> total de elementos (2 x 3)
```

O `shape` é uma **tupla**, e cada número é o tamanho de uma dimensão. Em 2D, o primeiro é o número de **linhas** e o segundo, o de **colunas**.

### Pegadinha clássica: (3,) é diferente de (1, 3)

```python
np.array([1, 2, 3]).shape      # (3,)    -> array 1D
np.array([[1, 2, 3]]).shape    # (1, 3)  -> matriz com 1 linha e 3 colunas
```

Os colchetes duplos é que criam a segunda dimensão.

## 4. dtype: o tipo dos dados

Todos os elementos de um array têm **o mesmo tipo**, guardado em `dtype`.

```python
np.array([1, 2, 3]).dtype        # int64   (inteiros)
np.array([1.5, 2.0, 3.1]).dtype  # float64 (decimais)
```

Ao misturar tipos, o NumPy **converte tudo para o tipo mais abrangente**:

```python
np.array([1, 2.5, 3])
# [1.  2.5 3. ]  -> o int virou float (float64)

np.array([1, "a", 3])
# ['1' 'a' '3']  -> tudo virou texto (dtype '<U21')
```

Para converter o tipo, use `astype`:

```python
x = np.array([1.7, 2.2, 3.9])
print(x.astype(int))   # [1 2 3]  -> atenção: TRUNCA, não arredonda
```

Esse conceito reaparece no Pandas, onde cada coluna também tem seu `dtype`.

## 5. Operações vetorizadas

"Vetorizar" é aplicar uma operação a **todos os elementos de uma vez, sem escrever for**.

### Array com número (escalar)

```python
a = np.array([1, 2, 3])
print(a * 2)    # [2 4 6]
print(a + 10)   # [11 12 13]
print(a ** 2)   # [1 4 9]
```

### Array com array (elemento a elemento)

```python
x = np.array([10, 20, 30])
y = np.array([1, 2, 3])

print(x + y)    # [11 22 33]
print(x * y)    # [10 40 90]
print(x / y)    # [10. 10. 10.]
```

### Comparações também são vetorizadas

Elas devolvem um array de `True`/`False`, que é a base dos filtros que você já usou no Pandas.

```python
print(x > 15)   # [False  True  True]
```

### Lista comum versus array

```python
[1, 2, 3] * 2             # [1, 2, 3, 1, 2, 3]  -> a lista REPETE
np.array([1, 2, 3]) * 2   # [2 4 6]             -> o array MULTIPLICA
```

## 6. Erros comuns

### Shapes incompatíveis

```python
np.array([1, 2, 3]) + np.array([1, 2])
# ValueError: operands could not be broadcast together with shapes (3,) (2,)
```

Para operar array com array, os shapes precisam ser compatíveis. O "broadcast" citado na mensagem é o tema do módulo 5.

### Confundir array 1D com matriz 1x3

`np.array([1, 2, 3])` tem shape `(3,)`. Uma matriz 1x3 tem shape `(1, 3)` e exige colchetes duplos.

## 7. Exercício resolvido: temperaturas da semana

**Enunciado:** dadas as temperaturas diárias (°C) de uma cidade, converter para array, inspecionar `shape` e `dtype`, converter para Fahrenheit sem loop e montar uma matriz 2x7 com uma segunda semana.

```python
temps_c = [22.5, 24.0, 19.5, 30.1, 28.4, 21.0, 25.3]

# 1. Lista -> array
a = np.array(temps_c)

# 2. shape e dtype
print(a.shape)   # (7,)
print(a.dtype)   # float64

# 3. Celsius -> Fahrenheit, sem loop (vetorizado)
f = a * 9/5 + 32
print(f)         # [72.5  75.2  67.1  86.18 83.12 69.8  77.54]

# 4. Matriz 2x7 com duas semanas
semana_01 = a
semana_02 = np.array([22, 23, 25, 24, 26, 28, 27])
matriz = np.array([semana_01, semana_02])
print(matriz.shape)   # (2, 7)
```

### Observações do exercício

- Para ver o `dtype` de um array que já existe, basta `a.dtype`. Não é preciso envolvê-lo de novo em `np.array(...)`.
- `semana_02` foi criada só com inteiros, mas na matriz aparece como `22.`, `23.`... porque a matriz tem **um único dtype** e o NumPy converteu tudo para `float64`.

### Bônus sugerido

1. Calcular a diferença entre as semanas sem loop: `semana_02 - semana_01`.
2. Prever o `dtype` de `semana_02` sozinha (`int64`) e o da diferença (`float64`), e conferir rodando.

## 8. Revisão de Python: a função de desconto

No nivelamento, a função usava um valor fixo (`0.10`) em vez do parâmetro. Versão que respeita o `pct`:

```python
def aplicar_desconto(preco, pct=10):
    desconto = preco * pct / 100
    return preco - desconto

aplicar_desconto(200)       # 180.0  (usa o padrão de 10%)
aplicar_desconto(200, 20)   # 160.0  (usa o pct informado)
```

Usar uma variável intermediária deixa o cálculo mais legível. O ponto principal é que parâmetros devem ser **usados** dentro da função, e não repetidos como valores fixos.

## 9. Resumo rápido

| Conceito | O que lembrar |
| --- | --- |
| `np.array([...])` | cria um array a partir de uma lista |
| `zeros`, `ones`, `arange`, `linspace` | criadores prontos |
| `shape` | tupla com o tamanho de cada dimensão: (linhas, colunas) |
| `ndim` / `size` | número de dimensões / total de elementos |
| `dtype` | tipo único de todos os elementos; `astype` converte |
| vetorização | operação aplicada a todos os elementos, sem `for` |
| `lista * 2` versus `array * 2` | a lista repete, o array multiplica |

## 10. Trilha de NumPy

1. **Arrays: criação, shape, dtype e operações vetorizadas** (este módulo, concluído)
2. Indexação, fatiamento e filtros booleanos
3. Matrizes 2D e eixos (`axis`)
4. Funções estatísticas e agregações (`mean`, `sum`, `max`...)
5. Broadcasting
6. Mini-projeto integrando tudo

Depois do NumPy, os módulos de SQL e Pandas seguem a partir da base de `loc`, `iloc` e filtros que já foi vista.
