# Lab 01 - Persistência e Init Containers no Kubernetes

Este laboratório demonstra como utilizar volumes persistentes em conjunto com **Init Containers** para garantir que dados sejam inicializados ou modificados antes do início do container principal.

## Objetivo
O objetivo é criar um servidor Nginx que serve uma página HTML persistente. Um `initContainer` é responsável por registrar o timestamp de cada inicialização do Pod no arquivo `index.html` armazenado em um `PersistentVolumeClaim` (PVC).

O projeto está configurado para rodar no perfil `minikube-k8s-lab` do Minikube e no namespace `lab-storage`.

## Estrutura do Projeto
- `k8s/pvc.yml`: Define um `PersistentVolumeClaim` de 100Mi para armazenamento persistente.
- `k8s/deployment.yml`: Define o Deployment do Nginx com um `initContainer` que escreve no volume compartilhado.
- `Makefile`: Atalhos para facilitar a execução dos comandos `kubectl` e gerenciamento de contexto/perfil.

## Como Executar

### 1. Preparar o ambiente
Para configurar o perfil do Minikube, o contexto e aplicar os manifestos:

```bash
make up
```
*Nota: Este comando assume que o namespace `lab-storage` já existe no cluster.*

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
O projeto inclui uma configuração de Ingress que mapeia o domínio `vessel-k8s-lab.local`.

#### Configuração do DNS local
Certifique-se de que o IP do seu cluster Minikube está mapeado no arquivo `/etc/hosts`.

#### Teste com HTTPie
```bash
http vessel-k8s-lab.local/01-storage
```

### 5. Testar a persistência
Para validar que os dados persistem entre reinicializações:

```bash
make restart
make wait
```
Aguarde o novo Pod subir e verifique os logs ou acesse via Ingress/Port-forward para ver a nova data adicionada pelo `initContainer`.

## Utilitários do Makefile
Execute `make help` para ver todas as opções disponíveis.

---
**Nota:** O namespace `lab-storage` é gerido externamente pela infraestrutura do laboratório.
