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
Para subir toda a infraestrutura (Namespace, Contexto e Manifestos) e aguardar os recursos ficarem prontos:

```bash
make up
```

### 2. Verificar os recursos
Para ver a situação atual do cluster no namespace do projeto:

```bash
make status
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
Adicione o IP do seu cluster (192.168.49.2 no Minikube local) ao arquivo `/etc/hosts`:
```text
192.168.49.2 vessel-k8s-lab.local
```

#### Teste com HTTPie
Para testar o acesso através do Ingress Controller:

```bash
# Acessando via domínio (porta 80 padrão do Ingress)
http vessel-k8s-lab.local/01-persistence-unit
```

### 5. Testar a persistência
Para validar que os dados persistem entre reinicializações, você pode forçar um reinício do Deployment. O `initContainer` rodará novamente e adicionará uma nova linha ao arquivo `index.html`.

```bash
make restart
make wait
```
Aguarde o novo Pod subir e verifique os logs ou acesse via Ingress/Port-forward para ver a nova data adicionada.

## Utilitários do Makefile
O `Makefile` funciona como uma documentação viva de comandos úteis. Execute para ver todas as opções categorizadas:

```bash
make help
```

Alguns comandos úteis para estudo:
- `make logs`: Acompanha a saída do container.
- `make shell`: Acessa o interior do container Nginx.
- `make ctx`: Garante que seu terminal está operando no namespace correto.

## Limpeza
Para remover todos os recursos criados:

```bash
make down
```

---
**Nota:** Este lab assume que seu cluster Kubernetes possui um `StorageClass` padrão configurado para provisionamento dinâmico de volumes.
