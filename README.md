<h1>Gerenciando a Distribuição dos Pods</h1>

<h1>Taint e Tolerations</h1>

Taint cria uma antiafinidade baseada no node.
O node que define qual pod ser executado.

Três tipos: Perferrd Schedule, No Schedule e No Execute.

O Tolerations auxilia o Taint quebrando as regras.


<h2>Testes</h2>

<h3>No Schedule</h3>

Aplicar o manifesto:
``` bash
$ kubectl apply -f nginx.yaml
```
Consultar o Pod e copiar a identificação do node:
``` bash
$ kubectl get pods -o wide
```

Configurar o taint no node:
``` bash
$ kubectl taint nodes pool-9pt8s802n-cqfq2 special=valor1:NoSchedule
```

Alterar o número de réplicas no manifesto:
``` yaml
spec:
  replicas: 10
```

Aplicar o manifesto:
``` bash
$ kubectl apply -f nginx.yaml
```

Consultar o Pod:
``` bash
$ kubectl get pods -o wide
```

Observe que apenas 01 pod foi vinculado ao node que foi configurado o taint.

O taint não permite agendar novos pods para o node.

Mas o pod que estava vinculado ao node com taint continua executando.

Se este pod for deletado nenhum outro será agendado.


<h3>NoExecute</h3>

Todos os pods que estão vinculados ao node ficarão no status de Pending.

Configurar o taint no node:
``` bash
$ kubectl taint nodes pool-9pt8s802n-cqfqt special=valor1:NoExecute
```

<h3>PreferNoSchedule</h3>

De preferência não agendar pods no node, mas não há problema se for agendado.
Configurar o taint no node:
``` bash
$ kubectl taint nodes pool-9pt8s802n-cqfqt special=valor1:NoExecute
```
Configurar o taint no node:
``` bash
$ kubectl taint nodes pool-9pt8s802n-cqfqt special=valor1:PreferNoSchedule
```

<h3>Desvincular Taint</h3>

Consultar descrição do node e procurar a chave taint.
Se a chave taint não estiver vinculada a nenhuma regra ela terá o valor: none

``` bash
$ kubectl describe node  pool-9pt8s802n-cqfqt | less
```

Para desvincular:

``` bash
$ kubectl taint nodes pool-9pt8s802n-cqfqt special=valor1:NoExecute-
$ kubectl taint nodes pool-9pt8s802n-cqfqt special=valor1:NoSchedule-
$ kubectl taint nodes pool-9pt8s802n-cqfqt special=valor1:PreferNoSchedule-
```

<h3>Tolerations</h3>

Trecho do manifesto:

``` yaml
tolerations:
    - key: "special"
      operador: "Equal"
      value: "valor 1"
      effect: "NoExecute"  
      tolerationSeconds: 60
```

Aplicar o manifesto:
``` bash
$ kubectl apply -f nginx.yaml
```

Aumentar o número de réplicas:

``` bash
$ kubectl scale deployment nginx --replicas 10
```

Configurar o node selecionado para não exceutar e nem agendar nenhum pod.

``` bash
$ kubectl taint nodes pool-9pt8s802n-cqfqt special=valor1:NoExecute
$ kubectl taint nodes pool-9pt8s802n-cqfqt special=valor1:NoSchedule
```

Consultar o Pod, veja que mesmo com a regra NoExecute eles continuam em execução.
Isto ocorreu devido a regra de tolerância inserida no manifesto.
``` bash
$ kubectl get pods -o wide
```

Mas se tentar escalar os pods não serão agendados:
``` bash
$ kubectl scale deployment nginx --replicas 15
```

