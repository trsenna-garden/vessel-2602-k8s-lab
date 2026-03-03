# Lab 04 - Explorando Resources e HPA no Kubernetes

Este laboratório foca na validação prática de escalonamento automático de pods (**Horizontal Pod Autoscaler - HPA**) baseado em consumo de recursos (**CPU Resources**).

## 🚀 Objetivo
Validar como o Kubernetes gerencia a elasticidade de uma aplicação com base em métricas de CPU, garantindo que o HPA reaja rapidamente a picos de carga e mantenha a estabilidade do cluster durante o resfriamento.

## 🛠️ Pré-requisitos
Para a execução deste lab, assume-se que a infraestrutura base já foi provisionada (via projeto `00-cluster`):
- Cluster Kubernetes ativo.
- **Metrics-server** habilitado e funcional.
- Namespace `lab-scaling` já criado.

## 🏗️ Estrutura do Lab

1.  **Deployment (`app-scaling`)**: Utiliza a imagem `registry.k8s.io/hpa-example` que executa cálculos intensivos de CPU ao receber requisições HTTP.
2.  **Resources**: O pod está configurado com `requests: 50m` de CPU. O HPA usará este valor como base para calcular a porcentagem de uso.
3.  **HPA (`app-hpa`)**: Configurado para manter a utilização média de CPU em **50%**, escalando de **1 a 5 réplicas**.
4.  **Service & Ingress**: A aplicação é exposta internamente no cluster e externamente via Ingress no path `/04-scaling`.

## 🧪 Guia de Validação

### 1. Aplicar Manifestos
Implante os recursos da aplicação no namespace pré-existente:
```bash
make up
```

### 2. Monitorar o Escalonamento (Terminal 1)
Em um terminal separado, acompanhe o estado do HPA em tempo real:
```bash
make hpa-status
```
*Nota: Pode levar até 1 minuto para o `TARGETS` sair de `<unknown>` para `0%/50%`.*

### 3. Gerar Carga de Teste (Terminal 2)
Inicie o bombardeamento de requisições ao serviço para forçar o consumo de CPU:
```bash
make load-test
```

### 4. Observar o Scale-up
No **Terminal 1**, você verá o uso de CPU disparar (ex: `250%/50%`). O Kubernetes criará novos pods (`REPLICAS` subindo de 1 para 5) conforme a carga aumenta.

### 5. Observar o Scale-down
Interrompa o `make load-test` (Ctrl+C no terminal de carga).
*   **O que esperar:** A carga cairá para `0%/50%`.
*   **O Tempo de Segurança:** O Kubernetes aguardará o tempo padrão de segurança (*stabilization window*) antes de remover os pods extras para evitar instabilidade.

## 🧹 Limpeza
Para remover os recursos deste laboratório (sem afetar a infraestrutura base):
```bash
make down
```
