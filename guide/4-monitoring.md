id:     monitoring
Title:  SRE Academy - Monitoramento

# SRE Academy - Monitoramento

## Monitoramento (Prometheus e Grafana)

Deploy da stack de Monitoramento no Kubernetes.

Acesse o diretório monitoring a partir da raiz do projeto.
``` console
cd robot-shop/K8s/monitoring
```

Crie o namespace monitoring.
``` console
kubectl create namespace monitoring
```

Crie o role prometheus para gerenciar o namespace monitoring.
``` console
kubectl create -f k8s-prometheus/clusterRole.yaml
```

Configuração do prometheus no cluster Kubernetes.
``` console
kubectl create -f k8s-prometheus/config-map.yaml
```

Crie o deployment do prometheus.
``` console
kubectl create -f k8s-prometheus/prometheus-deployment.yaml --namespace=monitoring
```

Crie o service do prometheus.
``` console
kubectl create -f k8s-prometheus/prometheus-service.yaml --namespace=monitoring
```

Aplique algumas métricas nos recursos do Kubernetes.
``` console
kubectl apply -f kube-state-metrics/
```

Crie os recursos do Grafana
``` console
kubectl create -f k8s-grafana/
```

## Logging (Loki)

Deploy da stack de Log no Kubernetes.

Adicione o repositório do Grafana ao Helm.
``` console
helm repo add grafana https://grafana.github.io/helm-charts
```

Instale o loki no namespace monitoring.
``` console
helm install loki grafana/loki-stack --namespace monitoring --create-namespace --set grafana.enabled=true
```

## Usabilidade Rancher Desktop

Clique no ícone do Rancher Desktop na bandeja do sistema e selecione "Dashboard".

![Mostrando o menu do Rancher Desktop na bandeja do sistema](assets/using-rancher-1.png "Menu do Rancher Desktop na bandeja do sistema")
![Mostrando a tela de Dashboard do Rancher Desktop](assets/using-rancher-2.png "Tela de Dashboard do Rancher Desktop")

Selecione, no menu lateral esquerdo, "Workload" e "Pods" para visualizar os Pods em execução.

![Visualizando os Pods na janela de Dashboard no Rancher Desktop](assets/using-rancher-3.png "Dashboard > Workload > Pods")

Caso algum dos Pods não tenha sido inicializado corretamente, vá na linha do Pod com problema e no canto direito da janela clique nos três pontos verticais e selecione "Delete".

![Demonstrando como deletar um Pod pelo Rancher Desktop](assets/using-rancher-4.png "Deletando o Pod do Grafana")

Confirme a deleção. Logo após o Service Discovery irá iniciar outro Pod baseado na mesma imagem. Caso o problema persistir, execute o comando `kubectl logs -f {{POD_ID}} -n robot-shop` e altere `{{POD_ID}}` pelo nome do Pod, encontrado na coluna "Name" no "Dashboard" do Rancher. O log deve auxiliar na correção do problema.

## Grafana

Configuração do Grafana.

Clique no ícone do Rancher Desktop que se encontra na bandeja do sistema e selecione "Preferences".

![Selecionando Preferences na bandeja do sistema no menu do rancher-desktop](assets/rancher-desktop-step-6.png "Menu do Rancher Desktop na bandeja do sistema")

Selecione "Port Forwarding" no menu lateral esquerdo.

![Selecionando Port Forwarding no menu lateral esquerdo do rancher-desktop](assets/rancher-desktop-step-7.png "Rancher Desktop > Preferences > Port Forwarding")

Clique em "Forward" nos serviços "grafana", "prometheus-service" e "web". Será gerada uma porta, na coluna "Local Port", para que você acesse os serviços localmente em seu navegador.

![Habilitando o redirecionamento por uma porta local no rancher-desktop](assets/rancher-desktop-step-8.png "Habilitando redirecionamento")

Acesse o Grafana em seu navegador, na porta que você acabou de gerar, com o endereço `localhost:{{Local Port}}`.

Faça login com o usuário `admin` e senha `admin`. Logo após, defina uma nova senha. **Obs.:** Será necessário redefinir a senha toda vez que você deletar o pod do Grafana.

![Acessando o Grafana pelo navegador](assets/grafana-step-1.png "Login Grafana")

No menu lateral esquerdo, vá no ícone da engrenagem e selecione "Data sources".

![Menu lateral esquerdo com a opção Configuration expandida e Data sources destacado](assets/grafana-step-2.png "Data sources")

Selecione "Add data source".

![Página dos Data sources onde está listado o prometheus](assets/grafana-step-3.png "Data sources Grafana")

Selecione "Loki".

![Lista com possíveis Data sources para serem adicionados](assets/grafana-step-4.png "Lista Data sources")

Configure o Loki alterando sua URL para `http://loki.monitoring:3100`.

![Página com as configurações do Loki - Alterando a URL](assets/grafana-step-5.png "Configuração Loki")

Clique em "Save & test" no final da página.

![Página com as configurações do Loki - Save & test](assets/grafana-step-6.png "Configuração Loki")

No menu lateral esquerdo, vá no ícone de "mais"/"adição" (+) e selecione "Import".

![Menu lateral esquerdo com a opção Create expandida e Import destacado](assets/grafana-step-7.png "Import")

Digite "12740" em "Import via grafana.com" e clique em "Load" para importar o dashboard do Kubernetes.

![Importando o dashboard Kubernetes através de seu ID 12740](assets/grafana-step-8.png "Importando dashboard Kubernetes")

Selecione o seu data source do Prometheus e clique em "Import" no final da página.

![Página de configuração na importação do dashboard Kubernetes](assets/grafana-step-9.png "Importando dashboard Kubernetes")

- Novamente no menu lateral esquerdo, no ícone de "mais"/"adição" (+), selecione "Import".
- Digite "12019" em "Import via grafana.com" e clique em "Load" para importar o Loki.
- Selecione os seus data sources do Loki e Prometheus e clique em "Import".

### Dashboard Robot Shop

- Copie todo o conteúdo do arquivo [robot-shop-dashboard.json](https://bitbucket.org/ciandt_it/sre-academy/src/main/robot-shop/dashboards/robot-shop-dashboard.json)
- No Grafana, no menu lateral esquerdo, vá no ícone de "mais"/"adição" (+) e selecione "Import".
- Cole o conteúdo do arquivo .json na caixa de texto abaixo de "Import via panel json" e clique em "Load"