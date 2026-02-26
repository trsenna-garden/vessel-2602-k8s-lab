# Lab 03 - Explorando Health Probes no Kubernetes

Este laboratório demonstra o funcionamento prático das **Startup**, **Readiness** e **Liveness** Probes utilizando um servidor Nginx. O cenário foi configurado para simular uma aplicação que demora a iniciar e permite a manipulação manual do seu estado de saúde.

## O Cenário do Laboratório

O container Nginx foi configurado para:
1.  Aguardar **20 segundos** (`sleep 20`) antes de iniciar o processo do Nginx.
2.  Criar três arquivos de "check" no diretório `/tmp`: `startup_ok`, `ready_ok` e `alive_ok`.
3.  As Probes do Kubernetes monitoram a existência desses arquivos via comando `cat`.

## Como Iniciar

1.  **Prepare o ambiente e suba os recursos:**
    ```bash
    make up
    ```
2.  **Acompanhe o estado do Pod em tempo real:**
    ```bash
    watch kubectl get pods
    ```
    *(Você notará que o Pod levará cerca de 25-30 segundos para ficar `1/1 Ready` devido ao delay inicial).*

---

## Roteiro de Testes

### 1. Validando a Startup Probe
A **Startup Probe** bloqueia as outras probes até que ela passe.
- **Observação:** Durante os primeiros 20 segundos, o container estará no estado `Running`, mas o status do Pod não mudará para `Ready` porque o arquivo `/tmp/startup_ok` ainda não existe.
- **O que testar:** Tente rodar `make status` repetidamente logo após o `make up`.
- **Análise de Eventos:** Execute o comando abaixo para ver o K8s reclamando da falha inicial (que é esperada):
  ```bash
  kubectl get events --sort-by='.lastTimestamp'
  ```
- **Dica de Diagnóstico:** Se o container reiniciar repetidamente, o `kubectl describe pod` detalhará a falha:
  ```bash
  kubectl describe pod <nome-do-pod>
  ```
  Procure na seção `Events` por: `Startup probe failed: cat: /tmp/startup_ok: No such file or directory`.

- **Cenário de Falha:** Se você editasse o arquivo `k8s/05-deployment.yml` e reduzisse o `failureThreshold` da Startup Probe para `2`, o Kubernetes mataria o container antes dos 20 segundos de `sleep` terminarem, entrando em um loop de reinicialização.

### 2. Validando a Readiness Probe
A **Readiness Probe** determina se o Pod pode receber tráfego do Service/Ingress.
- **Teste:** Remova o arquivo de prontidão:
  ```bash
  make shell
  # Dentro do container:
  rm /tmp/ready_ok
  exit
  ```
- **Resultado:** Execute `make status`. O Pod aparecerá como `0/1 Ready`. O container **não** é reiniciado, mas o Ingress parará de enviar requisições para este Pod.
- **Dica:** Verifique os eventos com `kubectl get events` para ver a mensagem `Readiness probe failed`.
- **Restauração:** Entre no shell novamente e execute `touch /tmp/ready_ok` para ver o Pod voltar a ficar pronto.

### 3. Validando a Liveness Probe
A **Liveness Probe** determina se o container deve ser reiniciado.
- **Teste:** Remova o arquivo de vitalidade:
  ```bash
  make shell
  # Dentro do container:
  rm /tmp/alive_ok
  exit
  ```
- **Resultado:** Após alguns segundos (baseado no `failureThreshold`), o Kubernetes detectará a falha e **reiniciará o container**.
- **Dica:** Verifique os eventos com `kubectl get events` para ver a mensagem `Liveness probe failed`. Você verá uma indicação de que o container "unhealthy" será matado e reiniciado.
- **Observação:** O comando `kubectl get pods` mostrará o contador na coluna `RESTARTS` subindo para 1.

### 4. Simulando um Crash Real
Diferente das probes (onde o processo continua rodando mas o arquivo some), aqui simulamos a morte do processo principal.
- **Teste:** Derrube o processo do Nginx:
  ```bash
  make shell
  # Dentro do container:
  pkill nginx
  exit
  ```
- **Resultado:** O Kubernetes detectará imediatamente que o processo `PID 1` morreu e reiniciará o container, independente de qualquer Health Probe. Observe o aumento em `RESTARTS`.

---

## Comandos Úteis (Makefile)

- `make up`: Aplica os manifestos e configura o contexto.
- `make status`: Mostra o estado atual dos recursos.
- `make logs`: Acompanha a saída do Nginx.
- `make shell`: Acessa o container para manipular os arquivos de teste.
- `make down`: Limpa todos os recursos do laboratório.
