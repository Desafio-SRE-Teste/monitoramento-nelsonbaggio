id:     performance-test
Title:  SRE Academy - Teste de Performance

# SRE Academy - Teste de Performance

## Teste de Performance

É comum na evolução das aplicações a adição de novas etapas nos fluxos existentes, como por exemplo uma nova consulta na base de dados em um relatório antigo ou até mesmo a requisição para um novo serviço. Essas alterações podem acabar injetando latência e falhas nas jornadas.

Neste laboratório iremos simular no serviço Cart uma situção onde uma alteração na aplicação fará com que a performance da aplicação seja prejudicada e com o auxílio do monitoramento vamos conseguir observar essas alterações.

Vamos alterar duas variáveis de ambiente no **cart-deployment.yaml** afim de injetar latência e taxa de falha na aplicação, segue a nova configuração para o environment:

```` yaml
env:
    - name: DECREASE_AVAILABILITY
      value: "true"
    - name: INCREASE_LATENCY
      value: "true"
````

Após isso vamos realizar um novo deploy desinstalando e instalando novamente o chart do helm:

```` console
helm uninstall robot-shop --namespace robot-shop
````

```` console
helm install robot-shop --create-namespace --namespace robot-shop robot-shop/K8s/helm
````

Após isso, vamos observamos resultados diferentes para latência e disponibilidade durante a execução do teste de carga.

Esse é um experimento importante para deteminar o quanto as evoluções dentro da aplicação podem estar contribuindo positiva ou negativamente para a performance e confiabilidade. É interessante observar como as métricas variam após cada release, contrastando com o release anterior, pois mesmo que os objetivos continuem sendo atingidos, as anterações podem ser consideradas indesejadas.