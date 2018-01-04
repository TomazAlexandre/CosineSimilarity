# Similaridade de Cosseno

O caso de uso comercial para a semelhança de envolve a comparação de perfis de clientes, perfis de produtos ou documentos de texto. A questão algorítmica é se dois perfis de clientes são semelhantes ou não. A semelhança dos cossenos é talvez a maneira mais simples de determinar isso.

Se alguém pode comparar se dois objetos são semelhantes, pode-se usar a semelhança como um bloco de construção para conseguir tarefas mais complexas, como:

* pesquisa: encontre o documento mais parecido com um dado.

* classificação: algum cliente pode comprar esse produto.

* agrupamento: existem grupos naturais de documentos similares.

* recomendações de produtos: quais produtos são semelhantes às compras passadas do cliente.


# Os documentos são vetores, os perfis dos clientes são vetores
O primeiro passo da imaginação é parar de pensar em documentos como texto e de perfis de clientes como conjuntos de cliques. Em vez disso, modelamos documentos e perfis de clientes como vetores.

O primeiro passo da imaginação é parar de pensar em documentos como texto e de perfis de clientes como conjuntos de cliques. Em vez disso, modelamos documentos e perfis de clientes como vetores.

# Definição

Um vetor, como é definido em álgebra linear, é uma tupla de números m. Em outras palavras, o vetor é uma lista onde os elementos são números de ponto flutuante. O vetor é de tamanho fixo, que aqui será denotado como m. m é a dimensão do vetor.

Em Python podemos modelar vetores como este:
```xml
 m = 1000 # dimension of vector 
  vector = [0.0]**m #built a list of 1000 zeros
  vector[0] = 0.4
  vector[1] = 0.82
  ...
  vector[m - 1] = 0.001
```

Um também pode usar a classe de vetor numpy para trabalhar com vetores em Python.

#De documentos para vetores

Esta frase por exemplo:
```xml
"Juliana had a the flu"
```

Antes de podermos transformar esta frase(documento) em um vetor, precisamos conhecer o vocabulário da língua inglesa. Isso significa que precisamos conhecer cada palavra na língua inglesa. A dimensão do vetor que representa uma frase(documento) será igual ao número de palavras no vocabulário. O número de palavras na língua inglesa é grande e cresce constantemente. Na prática, opera-se num corpus de números fixos de frases(documentos). Uma verificação sobre o corpus é necessária para estabelecer o vocabulário.

```xml
#Vocabulario de documentos
  document_collection = [
    ["once", "upon", "a", "time", "there", ...],
    ["Juliana", "had", "a", "the", "flu", ...],
    ...
  ]

  #a tabela hash que mapeia palavras para IDs inteiros
  word_to_id = {} 
  for document in document_collection:
    for word in document:
      if not word_to_id.has_key(word):
        word_to_id[word] = word_to_id.size()
  #imprime o vocabulário:
  for (word, word_integer_id) in word_to_id:
    print word, word_integer_id
 ```
 
 Uma vez que você conhece o vocabulário, a representação vetorial do documento pode ser construída:
 
 ```xml
  document =  ["juliana", "had", "a", "the", "flu", ...]
  m = word_to_id.size()
  vector = [0.0]**m #criar um vetor de dimensão do tamanho do vocabulário
  for word in document:
    word_id = word_to_id[word]
    vector[word_id] += 1
  # vector[word_id] = contagem de word_id no documento, 
  #se a palavra não aparecer, a contagem é 0
 ```
 
#A representação vetorial dos documentos perde a ordem das palavras

Como pode ser visto, a representação vetorial perde informações sobre o pedido das palavras. Do vetor, podemos ler quantas vezes cada palavra ocorre no documento. Não podemos reconstruir a oração original do vetor. No entanto, para muitas tarefas, a "aproximação vetorial" é suficiente para capturar o significado do documento original.

#O vetor contém principalmente zeros.
Ao lidar com documentos de texto, o vocabulário pode ser de tamanho da ordem de 60000. Vamos assumir que o documento contém 5 palavras únicas. Depois, existem (60000 - 5) zeros. Isso significa que a representação de vetores como listas em um programa de computador não será eficiente. Isso é fácil de corrigir. Podemos usar uma tabela de hash em vez de uma lista, ou um vetor esparso em vez de um vetor denso.

O problema mais grave é que, se você tirar dois documentos escolhidos aleatoriamente de uma coleção, provavelmente a sua sobreposição será relatada como vazia. Em parte, o problema pode ser resolvido convertendo as palavras em suas formas raiz (um processo conhecido como stemming, por exemplo, job vs jobs) e os sinônimos colapsos (por exemplo, trabalho versus trabalho) na mesma classe de palavras. Não encontrar sobreposição quando deveria existir é um problema no aprendizado de máquina conhecido como "sparsity de dados" ou a "maldição de dimensionalidade". Existem técnicas especiais para resolver esse problema, mas essas serão discutidas em uma postagem de blog diferente (PRÓXIMA).

Abaixo está o fragmento de código que implementa a semelhança do cosseno. Nas linhas 5 e 6, os documentos 1 e 2 são convertidos em vetores. Ambos os vetores têm a mesma dimensão. Na linha 10, começamos a fazer um loop sobre os elementos de ambos os vetores. Acompanhamos três contadores: soma para o produto ponto e dois contadores para a soma dos quadrados dos elementos de ambos os vetores.

#Comparando dois documentos

```xml

  m = word_to_id.size()  
  document1 = ["juliana", "had", ...]
  document2 = [...]
  vector1 = to_vector(document1, word_to_id) # line 5
  vector2 = to_vector(document2, word_to_id) # line 6

  sum = 0.0 # sem produto
  norm1 = 0.0 #a soma dos quadrados dos componentes do vetor1
  norm2 = 0.0 #a soma dos quadrados dos componentes do vetor2
  
  for i in range(0, m): #line 10
    a = vector1[i]
    b = vector2[i]
    sum += a*b
    norm1 += a*a
    norm2 += b*b

  #no caso norma1 e norma2 sendo zero
  #significa que pelo menos um dos documentos
  #não tem palavras
  normalization = sqrt(norm1)*sqrt(norm2)
  cosine_similarity = sum/normalization

 ```
 
#Cosine Similarity é uma maneira de medir a sobreposição

Suponha que os vetores contenham apenas zeros e outros. Neste caso, os vetores representam conjuntos. O número é apenas uma soma de 0 e 1. Nós temos um 1 somente quando ambos os vetores têm um nas mesmas dimensões. Portanto, o numerador mede o número de dimensões em que ambos os vetores concordam.

O denominador está relacionado ao tamanho dos conjuntos que representa cada vetor.

Se alguém considerar que o cosseno é uma forma de medir a sobreposição, é natural comparar o coeficiente de cosseno com o coeficiente de dados e o coeficiente de Jaccard.

#Perfis de clientes listados aparti de cliques

Como o modelo acima se aplica aos perfis de usuários com dados de cliques? Primeiro, podemos pensar em um perfil de usuário como um conjunto de links (ou páginas) que o usuário clicou. Precisamos conhecer o conjunto de possíveis páginas que o usuário pode clicar para obter a dimensão do vetor.

```xml
user_profile = ["/index", "/socks", "/blouse", ...]
  page_to_integer_id = a map describing the mapping of a page to an integer_id
  m = #numero de paginas do site
  vector = [0.0]**m #vector de extensão m
  for page in user_profile:
    page_id = page_to_integer_id[page]
    vector[page_id] += 1
 ```
 
# O que significa a normalização?
Matematicamente, a semelhança de cosseno é expressa pela seguinte fórmula:
<p align="center">
  <img src="https://4.bp.blogspot.com/-9NjqGycD-yo/Wk3PayZPJNI/AAAAAAAAFrA/aTBsf_yI51Y5Uyx3CYddVtZz45o6IroyQCLcBGAs/s1600/1.JPG">
</p>
Em vez de escrever a fórmula acima, podemos primeiro normalizar os vetores auma e bb dividindo-se pelo seu comprimento
<p align="center">
  <img src="https://3.bp.blogspot.com/-if1GbGNBw6c/Wk3Pa4VMW7I/AAAAAAAAFrE/3BlA3xAwEjsPSvM88pSRZ02qxTMx9Gl5ACLcBGAs/s1600/2.JPG">
</p>
Após essa normalização, a semelhança de cosseno é simplesmente o produto ponto entre os vetores normalizados.
<p align="center">
  <img src="https://4.bp.blogspot.com/-hNe7cPGyjaA/Wk3PbDtUmgI/AAAAAAAAFrI/W_hgsMEK-V8w8A-JVLxo8B-_Z_fCL9bvACLcBGAs/s1600/3.JPG">
</p>

# Interpretação geométrica
Geometricamente, a normalização significa que os vetores se encontram em uma esfera de unidade em um espaço m-dimensional. Se estamos trabalhando em duas dimensões, essa observação pode ser facilmente ilustrada pelo desenho de uma circunferência de raio 1 e colocando o ponto final do vetor na circunferência como na figura abaixo. A origem do vetor está no centro da circunferência (0,0).

<p align="center">
  <img src="https://3.bp.blogspot.com/-SYoBMNar7q8/Wk3PbZufF6I/AAAAAAAAFrM/Qshq_KkEXSgh9hK0EaHfkAiiPbmeJub9gCLcBGAs/s1600/4.JPG">
</p>

Nós podemos dizer que:

```xml
semelhança (a, b) = cosseno do ângulo entre os dois vetores
```

#A normalização é benéfica
Por que a normalização é importante? Considere esta questão: como você compara um documento curto com um documento longo. A normalização torna os documentos comparáveis.

"Por que a normalização por comprimento de vetor é bom?" É uma questão que é muito mais difícil de responder. Pode-se considerar outras formas de normalizar os vetores.
