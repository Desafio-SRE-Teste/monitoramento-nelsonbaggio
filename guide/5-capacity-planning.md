id:     capacity-planning
Title:  SRE Academy - Planejamento de Capacidade

# SRE Academy - Planejamento de Capacidade

Uma das etapas fundamentais para garantir uma alta confiabilidade das aplicações é planejar a capacidade necessária de recursos disponibilizados para atender a demanda (usuários, requisições, operações, etc) e em quais objetivos precisam ser atendidos nesse contexto.

A disciplina de capacity planning abordada no [livro de SRE do Google](https://sre.google/sre-book/software-engineering-in-sre/) possui um nível maior de detalhes e envolve muito mais váriaveis a se considerar na hora do planejamento. Nesse tutorial vamos abordá-la de maneira simplificada trazendo apenas aspectos de boas práticas a serem aplicadas nas configurações de consumo de recursos utilizadas pelo Kubernetes na orquestração dos serviços que podem contribuir muito para os aspectos de performance e confiabilidade.

Para auxiliar nesse trabalho temos alguns *pré-requisitos*:

- A volumetria (ao menos aproximada em ordem de grandeza) a qual a aplicação será submetida em produção.
- Uma perspectiva ou intenção de qual nível de performance irá satisfazer a expectativa dos usuários traduzida em forma de SLOs (latência, disponibilidade, etc).
- Um teste de carga capaz de simular a volumetria que se deseja alcançar e analisar se os objetivos estão ou não sendo atendidos.

Vamos imaginar que o serviço **Cart** da aplicação Robot Shop tem uma volumetria estimada em **até 50 usuários simultâneos** e temos como objetivo atender 99% das requisições em até 500ms e que 99.99% dessas requisições sejam atendidas com sucesso. Temos assim os seguintes SLOs:

- **Tempo de resposta no percentil 99: 500ms**
- **Disponibilidade: 99.99%**

Vale ressaltar que poderíamos estar avaliando esses SLOs com valores diferentes para cada endpoint do serviço, dado que cada uma dessas jornadas (adicionar ao carrinho, envio, etc) poderia ter uma expectativa diferente, mas vamos considerar o mesmo valor para todas elas por hora.

## Kubernetes Requests e Limits

O Kubernetes possui duas configurações que precisamos fazer para determinar como recursos como CPU e memória irão ser consumidos por cada container, são eles os parâmetros de requests e limits.

- **Requests**: São o que o container tem a garantia de obter. Se um container solicitar um recurso, o Kubernetes o agendará em apenas um nó que possa fornecer esse recurso
- **Limits**: Garantem que um container nunca ultrapasse um determinado valor. O container só pode ir até o limite, caso ultrapasse ele será restringido pelo Kubernetes

- **CPU**: Os recursos da CPU são definidos em milicores. Para um container que precisa de dois núcleos completos para rodar, o valor a ser utilizado na configuração é “2000m”. Se o container precisa apenas de ¼ de um núcleo então o valor é “250m”. Vale notar que se você parametrizar um valor para requests de CPU maior que a contagem de núcleos do seu maior nó o provisionamento do seu pod nunca ocorrerá.

Uma observação é que a CPU é considerada um recurso “compressível”. Se seu container atingir seus limites de CPU, o Kubernetes começará a limitar seu container. Isso significa que a CPU será restrita artificialmente (esse conceito é chamado de *throttling*), possivelmente dando à aplicação um desempenho potencialmente pior.

- **Memória**: Normalmente definida mebibyte (MiB) que é basicamente a mesma coisa que um megabyte e assim como a CPU se você parametrizar um valor para requests de memória maior que a quantidade de memória do seu maior nó o provisionamento do seu pod nunca ocorrerá. Porém diferentemente da CPU a memória não é um recurso "compressível", logo se um container passar o limite de memória definido ele será terminado pelo Kubernetes

Abaixo seguem os parâmetros de request e limits para CPU e memória do serviço Cart contidos em seu arquivo de deployment (cart-deployment.yaml):

``` yaml
resources:
    limits:
        cpu: 100m
        memory: 100Mi
    requests:
        cpu: 50m
        memory: 50Mi
```

Em resumo, para ser provisionado, o container deste serviço precisa de **0.05 cores** de memória e **50 mebibytes** disponíveis em algum nó e o consumo desses recursos não ultrapassará **0.1 cores** e **100 mebibytes** de CPU e memória respectivamente.

## LimitRange e Quotas

É de extrema importância que cada container da sua aplicação tenha seus requests e limits definidos, porém também é possível definir **LimitRanges** dentro de cada namespace, assim, se um container não tiver os parâmetros configurados, ele assume os valores padrões definidos como vemos no arquivo **limit-range.yaml** e é possível checá-los com o comando abaixo:

``` console
kubectl describe limitrange -n robot-shop
```

É possível (e recomendado) garantir limites a nível do namespace para a soma total dos valores de requests e limits dos containers através do uso de quotas, garantido assim que quando existem mais namespaces eles tenham uma melhor divisão dos recursos. As quotas definidas para o namespace robot-shop estão no arquivo **quotas.yaml** e é possível consultá-las no cluster com o comando abaixo:

``` console
kubectl describe quota -n robot-shop
```

Os conceitos LimitRange e Quotas são ainda mais flexiveis e parametrizáveis do que apresentamos aqui, mas detalhes podem sem encontrados na própria [documentação do Kubernetes](https://kubernetes.io/docs/concepts/policy/resource-quotas/) ou neste [material de boas práticas](https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-resource-requests-and-limits) do próprio Google.

## Como definir os valores de requests e limits

Não existe uma forma de realizar essas parametrizações sem levar em conta fatores que vão desde as tecnologias usadas na construção da aplicação, quantidade de recursos disponíveis nos nós do cluster e ainda a performance que deseja alcançar.

> **Importante**: Ao executar os testes de carga, os resultados obtidos podem ser diferentes dos coletados durante a montagem desse material, isso ocorre pelo fato de que a performance da aplicação está diretamente ligada à performance do cluster que pode variar entre as máquinas dos desenvolvedores da CI&T, seja por questões relacionadas à plataforma ou até mesmo o compartilhamento de recursos de cada máquina que pode variar muito. Se isso acontecer, aproveite para exercitar o estabelecimento de objetivos diferentes dos exemplos aqui citados bem como a reconfiguração dos parâmetros de request e limits de cada aplicação.

Uma forma de iniciar esse trabalho é observar quanto de CPU e memória cada container gasta apenas com as aplicações em execução, sem qualquer carga de requisições ou usuários acontecendo sobre elas, isso dará uma orientação para criar os parâmetros de requests evitando que haja uma alocação desnecessária de recursos que poderia comprometer o provisionamento dos containers. O dashboard usado nesse tutorial nos ajuda a analisar esses valores, uma alternativa é utilizar o seguinte comando:

``` console
kubectl top pod -n robot-shop
```

O resultado deve ser semelhante ao seguinte:

``` console
NAME                                CPU(cores)   MEMORY(bytes)
cart-b95597876-8ztql                2m           24Mi
catalogue-57cb5775d9-xmhnc          1m           38Mi
dispatch-5cb7956947-wlhtr           18m          11Mi
mongodb-76457756fb-mhf8r            9m           97Mi
mongodb-exporter-54cdc56b5c-wkhrd   3m           9Mi
mysql-7456d7d8d7-mts8k              5m           229Mi
payment-67d5dd4dd5-rp5zd            1m           38Mi
rabbitmq-75d9cf4484-pb7ls           12m          91Mi
ratings-5d9dff56bd-w7vzx            2m           55Mi
redis-0                             7m           4Mi
shipping-65dd6fdcd6-x9s45           18m          412Mi
svclb-web-d692b                     0m           0Mi
user-cd478d895-t9t6d                1m           32Mi
web-5488d5545f-8mptg                48m          8Mi
```

Utilizando o teste de carga com o volume desejado, é possível simular a utilização da aplicação e acompanhar o crescimento do consumo de recursos. Esses novos parâmetros resultados ajudam a nos orientar no momento de estabelecer os limites de cada container, dado que agora temos uma ideia de como a performance da aplicação se comporta diante da volumetria que esperamos em produção. No nosso exemplo, esperamos que o serviço Cart atenda uma demanda de **50** usuários simultâneos cumprindo os SLOs que definimos anteriormente, para isso podemos executar o teste de carga com os seguintes parâmetros:

``` console
./load-gen.sh -n 50
```

Ao realizar novamente a checagem do consumo de recursos pelos containers temos:

``` console
NAME                                CPU(cores)   MEMORY(bytes)
cart-b95597876-8ztql                37m          30Mi
catalogue-57cb5775d9-xmhnc          54m          39Mi
dispatch-5cb7956947-wlhtr           12m          9Mi
mongodb-76457756fb-mhf8r            25m          96Mi
mongodb-exporter-54cdc56b5c-wkhrd   4m           9Mi
mysql-7456d7d8d7-mts8k              19m          229Mi
payment-67d5dd4dd5-rp5zd            1m           37Mi
rabbitmq-75d9cf4484-pb7ls           80m          92Mi
ratings-5d9dff56bd-w7vzx            19m          54Mi
redis-0                             10m          4Mi
shipping-65dd6fdcd6-x9s45           12m          405Mi
svclb-web-d692b                     0m           0Mi
user-cd478d895-t9t6d                23m          29Mi
web-5488d5545f-8mptg                57m          9Mi
```

## Horizontal Pod Autoscaling (HPA)

Além de planejar quanto de recursos do cluster cada aplicação irá consumir é importante também determinar os critérios para escala horizontal, ou seja, a quantidade de instâncias de cada serviço necessária para atender a volumetria nas condições desejadas.

As configurações de HPA para o serviço Cart podem ser encontradas no arquivo **hpa.yaml**. Nesse exemplo por questões didáticas configuramos a escalabilidade apenas para esse serviço.

Durante a execução do teste de carga para os mesmos 50 usuários simultâneos e com o HPA configurado para criar uma nova replica sempre que o consumo de CPU do container chegar em **70%**, notamos que, com o passar do tempo, para atender as requisições dentro dos objetivos o cluster aplicou o autoscaling e criou até mais 3 instâncias do serviço, e assim permaneceu durante o restante do teste.

Você pode checar isso durante a execução do teste com o seguinte comando:

``` console
kubectl get pods -n robot-shop | grep cart
```

Uma saída como a seguinte deverá ser apresentada, contendo 4 pods do serviço Cart:

``` console
NAME                                READY   STATUS    RESTARTS   AGE
cart-b95597876-v8gs2                1/1     Running       0          9m39s
cart-b95597876-h7dpb                1/1     Running       0          6m19s
ratings-5d9dff56bd-ljdsx            1/1     Running       0          9m36s
cart-b95597876-zmzg5                1/1     Running       0          5m47s
cart-b95597876-s4t89                1/1     Running       0          7m14s
```

Durante a execução do teste de carga, o numero máximo de pods alcançado pelo serviço cart foi 4, então a configuração final do HPA ficou da seguinte maneira:

``` yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
 name: cart
spec:
 scaleTargetRef:
   apiVersion: apps/v1
   kind: Deployment
   name: cart
 minReplicas: 1
 maxReplicas: 4
 targetCPUUtilizationPercentage: 70
```

O número mínimo de replicas permaneceu 1 pois nesse caso estamos considerando que a aplicação pode ter momentos sem qualquer requisição. A threshold de 70% de uso de CPU pode variar dependendo da performance da aplicação.

## Recapacity (Ou Continuous Capacity)

Vimos até aqui o quão importante é relacionar o consumo de recursos e a escalabilidade de uma aplicação aos objetivos definidos a nível de serviço levando em consideração uma estimativa de carga que essa aplicação deverá ser capaz de atender sem que haja quebra dos SLOs.

Com essa relação estabelecida **é importante estabelecer um processo de análise, baseado no monitoramento contínuo, a fim de identificar a necessidade de possíveis alterações na forma como a aplicação consome recursos e/ou escala**. Normalmente essas necessidades têm duas origens:

- Crescimento orgânico: Aumento gradual no volume de usuários ou requisições sobre a aplicação que com o passar do tempo começa a necessitar de mais recursos para atender a nova demanda.
- Crescimento inorgânico: Determinados períodos em que a aplicação irá receber uma maior quantidade de usuários ou requisições fora dos padrões habituais, como por exemplo um e-commerce na Black Friday, o sistema da receita na entrega do imposto de renda, etc.

Em termos de desenvolvimento é importante também ficar atento aos parâmetros de capacidade da aplicação sempre que atualizações de software, infraestrutura ou dependências ocorrerem.

## Conclusão

Planejar a capacidade necessária para que uma aplicação ou conjunto de serviços seja capaz de atender os objetivos definidos não é uma tarefa fácil pois pode envolver diversas variáveis que não mencionamos aqui. Porém, apesar da visão simplicada que apresentamos nesse material, esse trabalho pode ter um resultado decisivo na performance das aplicações, sendo um ótimo ponto de partida para aprofundar os conhecimentos tanto da parte técnica (Kubernetes e suas configurações) quanto no impacto que essas decisões geram na experiência do cliente final.
