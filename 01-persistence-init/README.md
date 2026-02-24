# Lab 01 - Persistência e Init Containers no Kubernetes

Este laboratório demonstra como utilizar volumes persistentes em conjunto com **Init Containers** para garantir que dados sejam inicializados ou modificados antes do início do container principal.

## Objetivo
O objetivo é criar um servidor Nginx que serve uma página HTML persistente. Um `initContainer` é responsável por registrar o timestamp de cada inicialização do Pod no arquivo `index.html` armazenado em um `PersistentVolumeClaim` (PVC).

## Estrutura do Projeto
- `k8s/namespace.yml`: Define o namespace `k8s-lab-01-persistence-unit`.
- `k8s/pvc.yml`: Define um `PersistentVolumeClaim` de 100Mi para armazenamento persistente.
- `k8s/deployment.yml`: Define o Deployment do Nginx com um `initContainer` que escreve no volume compartilhado.
- `Makefile`: Atalhos para facilitar a execução dos comandos `kubectl`.

## Como Executar

### 1. Preparar o ambiente
Aplique os manifestos e configure o contexto do seu `kubectl` para o namespace correto:

```bash
make apply-namespace
make config-set-context
make apply
```

### 2. Verificar os recursos
Acompanhe a criação do Pod e do PVC:

```bash
kubectl get pods
kubectl get pvc
```

### 3. Acessar a aplicação
O container principal (Nginx) servirá o conteúdo do volume. Use o `port-forward` para acessar localmente:

```bash
make port-forward
```
Acesse no navegador: [http://localhost:8080](http://localhost:8080)

### 4. Testar o Ingress (com Rewrite)
O projeto inclui uma configuração de Ingress que mapeia o domínio `vessel-k8s-lab.local` e utiliza `rewrite-target`.

#### Configuração do DNS local
Adicione o IP do seu cluster ao arquivo `/etc/hosts`:
```text
<IP_DO_CLUSTER> vessel-k8s-lab.local
```
*(No Minikube, obtenha o IP com `minikube ip`)*

#### Teste com HTTPie
Para testar o acesso através do Ingress Controller (considerando a porta mapeada pelo serviço `ingress-nginx-controller`):

```bash
# Acessando via domínio e porta do Ingress (ex: 30560)
http vessel-k8s-lab.local:30560/01-persistence-unit
```

### 5. Testar a persistência
Se você reiniciar o Pod (deletando-o), o `initContainer` rodará novamente e adicionará uma nova linha ao arquivo `index.html`. Como o volume é persistente, o histórico de reinicializações será preservado.

```bash
kubectl delete pod -l app=web-server
```
Aguarde o novo Pod subir e execute novamente o comando do Ingress para ver a nova data adicionada.

## Limpeza
Para remover todos os recursos criados:

```bash
make clean
```

---
**Nota:** Este lab assume que seu cluster Kubernetes possui um `StorageClass` padrão configurado para provisionamento dinâmico de volumes.
