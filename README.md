## Design Orientado ao Conhecimento

Construir um modelo que reflita o conhecimento adquirido junto aos experts de domínio vai muito além de fazer dinâmicas como *listar os substantivos*, como aprendemos tradicionalmente. As atividades do dia-a-dia e regras implícitas são pontos centrais no modelo de domínio que estamos construíndo, já que as entidades evoluem à medida que nosso conhecimento sobre o negócio também evolui. A atividade que chamamos de *triturar* conhecimento instiga que surjam novos modelos que reflitam esse novo conhecimento adquirido. Em paralelo, à medida que os modelos mudam, os desenvolvedores refatoram sua implementação para que ela expresse a intenção por trás do modelo, dando utilidade ao código escrito.

Após a fase onde as entidades do sistema e os valores que elas podem assumir são compreendidos, o triturar do conhecimento pode se tornar um pouco desconfortável, porque muitas vezes ele mostra como há inconsistências reais nas regras de negócio.
Os experts de domínio, em seu dia-a-dia, não percebem o quão complexos seus processos mentais podem ser, à medida que eles navegam em meio a regras, relevam contradições, e preenchem as incertezas com senso comum. Software não tem essa capacidade. É através do *triturar do conhecimento*, em parceria direta com os experts de domínio que as regras são esclarecidas, reveladas, combinadas ou até retiradas de escopo.

#### Exemplo

##### Extraindo um conceito escondido.

Vamos começar com um exemplo simples de um modelo que pode servir como base para uma aplicação que realiza *booking* de cargas em um navio.
                                                                  
           +----------------+                   +----------------+
           |                |                   |                |
           |     Navio      |<----------------->|     Carga      |
           |                |                   |                |
           +----------------+                   +----------------+
                                                        
Podemos afirmar que a aplicação tem responsabilidade de associar cada **Carga** a um **Navio**, e gravar as interações dessa relação. Até aqui tudo bem. Um código possível seria esse:

```python
def fazerReserva(carga, navio):
    confirmacao = fila_de_confirmacoes.pop()
    navio.addReserva(carga, confirmacao)
    return confirmacao

carga = Carga()
navio = Navio()
reserva = fazerReserva(carga, navio)
```

Mas agora vem algo que você - que mora bem longe do litoral - provavelmente não sabe: pelo fato de sempre haver cancelamento por parte dos clientes de última hora se tornou padrão na indústria reservar uma quantidade maior de cargas em um navio do que ele de fato suporta. Essa prática é chamada de "**overbooking**". Às vezes é usado apenas uma porcentagem para calcular o tal overbooking; às vezes podem ser regras complexas que favorecem um cliente ao invés de outro, ou então alguns tipos de carga.

Essa é uma estratégia comum entre as pessoas que trabalham na indústria de cargas, porém pode não ser facilmente entendido por todas os desenvolvedores do time.

Então um belo dia você recebe o tal requisito a ser implementado no sistema, com a seguinte descrição:

- Permitir 110% de overbooking.

Vamos então deixar o diagrama assim, pra contemplar essa nova demanda:

          
           +----------------+                   +----------------+
           |     Navio      |                   |     Carga      |
           |               -|<----------------->|                |
           |----------------|                   ------------------
           |   capacidade   |                   |      peso      |
           +----------------+                   +----------------+

E mudamos nosso código pra que fique assim:

```python
def fazerReserva(carga, navio):
    capacidade_maxima = navio.capacidade * 1.1
    if navio.get_soma_total_das_cargas() + carga.peso > capacidade_maxima:
        return False
    else:
        confirmacao = fila_de_confirmacoes.pop()
        navio.addReserva(carga, confirmacao)
        return confirmacao

carga = Carga()
navio = Navio()
reserva = fazerReserva(carga, navio)
```

Agora atenção: estamos escrevendo de um modo obscuro algo que é claramente uma regra de negócio **muito importante**.
Da maneira como está escrito acima vai ser muito difícil algum expert de domínio entender exatamente o pensamento por trás disso, mesmo com a ajuda de um desenvolvedor. Imagine se a regra fosse ainda mais complexa, o pesadelo que seria fazer com que essa regra agora apenas se aplicasse a navios de certo tipo de casco, ou para cargas que estão em certas baias.

O que precisamos fazer é capturar a essência do conhecimento que está por trás dessa regra de negócio. O que estamos tratando aqui, **overbooking**, claramente trata-se de uma **Policy** (política, como em "política de incentivo a x, y e z").                                             
                                                                  
           +----------------+                   +----------------+
           |     Navio      |                   |     Carga      |
           |               -|<--------|-------->|                |
           |----------------|         |         ------------------
           |   capacidade   |         |         |      peso      |
           +----------------+         |         +----------------+
                  {sum(carga.peso) < navio.capacidade * 1.1}      
                                      |                           
                                      |                           
                             +-----------------+                  
                             |   Overbooking   |                  
                             |     Policy      |                  
                             +-----------------+                  

```python
overbookingPolicy = OverbookingPolicy()

def fazerReserva(carga, navio):
    if not overbookingPolicy.respeita_limite_maximo(carga, navio):
        return False
    else:
        confirmacao = fila_de_confirmacoes.pop()
        navio.addReserva(carga, confirmacao)
        return confirmacao

carga = Carga()
navio = Navio()
reserva = fazerReserva(carga, navio)
```

A nova classe OverbookingPolicy agora contém o seguinte método:

```python
class OverbookingPolicy:
    def respeita_limite_maximo(self, carga, navio):
        return carga.peso + navio.get_soma_total_das_cargas() <= navio.capacidade * 1.1
```

Desse modo, vai ficar claro para todo mundo que *overbooking* é uma política distinta, e a regra dessa política está explícita e separada.

Claro, isso não significa que você deve aplicar tal design para **todas** as implementações, iremos ver futuramente como julgar quais os momentos mais adequados para tal. Porém, esse design explícito traz diversas vantagens.

Pra que seja modelado dessa forma elaborada, os programadores e todos os envolvidos devem chegar a um consenso sobre o que é a natureza de um overbooking e como tal política é importante para o negócio, não apenas um cálculo obscuro.

Os programadores devem mostrar tais artefatos acima, talvez até mesmo a modelagem da classe se os experts de domínio se mostrarem interessados, e esses modelos devem ser compreensíveis (com auxílio dos desenvolvedores) àqueles que fazem parte do domínio, assim firmando um ciclo virtuoso de **feedback** entre os envolvidos.
