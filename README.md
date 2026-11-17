# Aula 09 – Atualização e Rollback no Kubernetes

## Introdução

Na Aula 09 de Computação em Nuvem, aprendemos como atualizar uma aplicação no Kubernetes e como voltar para uma versão anterior caso seja necessário.

Durante a aula, estudamos o funcionamento do Deployment, Service, RollingUpdate e Rollback. Também vimos a importância de verificar se os Pods estão prontos antes de receber as requisições.

Na atividade prática, utilizamos o Nginx como exemplo para criar uma aplicação com duas réplicas, atualizar seu conteúdo da versão 1 para a versão 2 e depois realizar um rollback para retornar à versão anterior.

## O que vimos na aula?

Durante a aula, aprendemos alguns conceitos importantes:

- **Deployment:** é responsável por controlar a aplicação e manter a quantidade de Pods que configuramos.
- **ReplicaSet:** ajuda a manter a quantidade de réplicas e acompanha os Pods de cada versão da aplicação.
- **Pod:** é onde a aplicação é executada.
- **Service:** permite acessar a aplicação por um endereço estável, mesmo quando os Pods são substituídos.
- **RollingUpdate:** permite atualizar a aplicação aos poucos, substituindo os Pods antigos pelos novos.
- **Readiness:** verifica se o Pod está pronto para receber requisições.
- **Rollback:** permite voltar para uma versão anterior da aplicação quando precisamos desfazer uma atualização.

Também vimos que o `maxSurge: 1` permite criar um Pod extra durante a atualização e que o `maxUnavailable: 0` busca manter a quantidade desejada de Pods disponíveis durante esse processo.

## Atividade prática – Atualização e Rollback

### Passo 1 – Preparando o ambiente

Primeiro, verificamos se os Nodes do Kubernetes estavam funcionando corretamente. Depois, criamos o namespace `aula9` para organizar os recursos utilizados na atividade.

Os dois Nodes apareceram com o status `Ready`, mostrando que estavam disponíveis.

### Passo 2 – Criando a aplicação

Em seguida, criamos o arquivo `aula9.yaml`, com as configurações do Deployment e do Service.

A aplicação foi configurada para utilizar duas réplicas do Nginx. Também definimos a estratégia `RollingUpdate` e a variável `VERSAO`, que começou com o valor `1`.

Essa variável foi utilizada para mostrar na página qual versão da aplicação estava sendo executada.

### Passo 3 – Aplicando as configurações

Depois de preparar o arquivo, utilizamos:

```bash
kubectl apply -f aula9.yaml
```

O resultado mostrou que os recursos foram criados:

```text
deployment.apps/web-aula9 created
service/web-aula9 created
```

Em seguida, verificamos se o Deployment havia terminado de iniciar:

```bash
kubectl -n aula9 rollout status deployment/web-aula9
```

O resultado foi:

```text
deployment "web-aula9" successfully rolled out
```

Isso confirmou que o processo de inicialização havia sido concluído.

### Passo 4 – Testando a versão 1

Com a aplicação criada, selecionamos um dos Pods e fizemos uma requisição HTTP para verificar o conteúdo da página.

O resultado foi:

```html
<h1>Versao 1</h1>
```

Assim, confirmamos que a primeira versão estava funcionando.

### Passo 5 – Atualizando para a versão 2

Depois, alteramos a variável `VERSAO` de 1 para 2 utilizando o comando:

```bash
kubectl -n aula9 set env deployment/web-aula9 VERSAO=2
```

Essa alteração fez com que o Kubernetes criasse uma nova revisão do Deployment e atualizasse os Pods.

Acompanhamos o processo com:

```bash
kubectl -n aula9 rollout status deployment/web-aula9
```

Também verificamos os Pods:

```bash
kubectl -n aula9 get pods -l app=web-aula9 -w
```

Os dois Pods apareceram com status `Running` e `READY 1/1`, mostrando que estavam funcionando após a atualização.

### Passo 6 – Consultando o histórico

Depois da atualização, consultamos o histórico do Deployment:

```bash
kubectl -n aula9 rollout history deployment/web-aula9
```

Também verificamos os ReplicaSets e o Deployment:

```bash
kubectl -n aula9 get replicasets
kubectl -n aula9 get deployment web-aula9
```

Com isso, conseguimos observar as revisões criadas durante a atividade e verificar que o Deployment estava com duas réplicas disponíveis.

### Passo 7 – Conferindo a versão 2

Para confirmar se a atualização realmente havia funcionado, fizemos outra requisição HTTP.

O resultado foi:

```html
<h1>Versao 2</h1>
```

Isso confirmou que a aplicação estava exibindo a nova versão.

### Passo 8 – Realizando o Rollback

Depois de testar a versão 2, utilizamos o comando abaixo para voltar à configuração anterior:

```bash
kubectl -n aula9 rollout undo deployment/web-aula9
```

O comando realizou o rollback e apresentou a mensagem:

```text
deployment.apps/web-aula9 rolled back
```

Em seguida, verificamos se o processo havia terminado:

```bash
kubectl -n aula9 rollout status deployment/web-aula9
```

O resultado confirmou que o Deployment havia sido atualizado novamente.

### Passo 9 – Confirmando o retorno à versão 1

Por último, fizemos mais um teste HTTP para verificar se a aplicação realmente havia voltado para a versão anterior.

O resultado foi:

```html
<h1>Versao 1</h1>
```

Com isso, confirmamos que o rollback funcionou.

Depois de finalizar os testes, removemos o namespace utilizado na atividade:

```bash
kubectl delete namespace aula9
```

E verificamos se ele havia sido excluído:

```bash
kubectl get namespace aula9
```

O Kubernetes retornou a mensagem `NotFound`, confirmando que o namespace já não existia.

## Desafio em dupla

Depois da atividade prática, respondemos algumas perguntas para entender melhor o que aconteceu durante a atualização e o rollback.

### 1. O que desencadeou o novo rollout?

O novo rollout aconteceu porque alteramos a variável de ambiente `VERSAO`, que passou de 1 para 2. Como essa variável faz parte da configuração dos Pods, o Kubernetes precisou criar novos Pods com a alteração realizada.

### 2. Por que o Service continuou com o mesmo nome?

Porque o Service é um recurso separado do Deployment. Mesmo com a substituição dos Pods durante a atualização, não foi necessário criar outro Service. Ele continuou utilizando o mesmo nome e direcionando o acesso para os Pods que correspondiam aos seus rótulos.

### 3. Qual a contribuição da readiness para a atualização?

A readiness verifica se o Pod está pronto para receber requisições. Durante a atualização, ela ajuda a evitar que um Pod seja considerado disponível antes de estar preparado. Assim, os Pods novos precisam estar prontos antes de serem considerados disponíveis no processo de atualização.

### 4. O que você precisa conferir após um rollback?

Precisamos verificar se o processo terminou corretamente, se os Pods estão prontos e se a aplicação voltou a apresentar a versão anterior. Também é importante consultar o histórico do Deployment para acompanhar as revisões realizadas.

## O que aprendemos na Aula 09?

Com essa aula, aprendemos que atualizar uma aplicação não significa simplesmente substituir os arquivos ou criar tudo novamente. O Kubernetes possui recursos que ajudam a realizar esse processo de forma organizada.

Entendemos que o Deployment controla as atualizações, que o RollingUpdate permite substituir os Pods aos poucos e que o Service mantém o mesmo endereço de acesso durante essas mudanças.

Também vimos que a readiness tem um papel importante para verificar se os Pods estão preparados para receber requisições.

Outro aprendizado importante foi o rollback, que permite voltar para uma configuração anterior quando uma atualização precisa ser desfeita.

## Resumo da atividade prática e do desafio

Na atividade prática, criamos uma aplicação utilizando Nginx com duas réplicas. Primeiro, testamos a versão 1, depois atualizamos para a versão 2 e verificamos se os novos Pods estavam funcionando. Em seguida, consultamos o histórico do Deployment e realizamos o rollback, confirmando que a aplicação voltou para a versão 1.

No desafio em dupla, utilizamos o que aconteceu na prática para entender por que o rollout foi iniciado, por que o Service manteve o mesmo nome, como a readiness ajuda durante a atualização e o que precisamos conferir depois de um rollback.

## Conclusão

A Aula 09 foi importante para entender melhor como funciona a atualização de aplicações no Kubernetes.

O que mais ajudou no aprendizado foi conseguir acompanhar cada etapa na prática, desde a criação da aplicação até a atualização e o retorno para a versão anterior.

Com os testes realizados, conseguimos observar as mudanças nos Pods, conferir as respostas HTTP e entender como o Kubernetes mantém o controle das versões.

Assim, aprendemos que é importante não apenas realizar uma atualização, mas também acompanhar seu funcionamento e saber como voltar atrás caso seja necessário.

**Computação em Nuvem – Aula 09 | Atualização e Rollback no Kubernetes**
