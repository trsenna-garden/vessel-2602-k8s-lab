# Lab 02 - ConfigMap e Secrets no Kubernetes

Este laboratório tem como objetivo demonstrar a utilização de **ConfigMaps** e **Secrets** para gerenciar configurações e dados sensíveis em aplicações Kubernetes.

## O que é este projeto?

O projeto sobe um servidor Nginx que atua como um Proxy Reverso para o serviço `httpbin.org`. Ele utiliza as seguintes funcionalidades do Kubernetes:

1.  **ConfigMap**:
    *   `nginx-config-template`: Armazena um template de configuração do Nginx. O container utiliza `envsubst` para injetar variáveis de ambiente no arquivo de configuração final.
    *   `nginx-env-vars`: Armazena variáveis de ambiente não sensíveis, como a URL do backend (`UPSTREAM_URL`).
2.  **Secret**:
    *   `nginx-auth-file`: Armazena o arquivo `.htpasswd` para proteger a aplicação com Autenticação Básica (Basic Auth).
    *   `nginx-secret-vars`: Armazena dados sensíveis, como o `API_TOKEN` que é enviado no Header das requisições para o backend.

## Como subir o ambiente

O projeto inclui um `Makefile` para facilitar as operações. Certifique-se de que o seu contexto do Kubernetes (ou Minikube) está ativo.

1.  **Subir tudo**:
    ```bash
    make up
    ```
    Este comando cria o namespace `lab-config`, aplica todos os manifestos na pasta `k8s/` e aguarda o Pod ficar pronto.

2.  **Verificar o status**:
    ```bash
    make status
    ```

## Como realizar o teste

Para validar se as configurações foram aplicadas corretamente através do Ingress, utilizaremos o **HTTPie**.

> **Nota**: Certifique-se de que o host `vessel-k8s-lab.local` está apontando para o IP do seu cluster (ex: IP do Minikube) no seu arquivo `/etc/hosts`.

1.  **Executar o teste com HTTPie**:
    ```bash
    http -a admin:senha123 http://vessel-k8s-lab.local/02-config
    ```

2.  **O que observar na resposta**:
    *   **Autenticação**: O HTTPie enviará o header `Authorization: Basic ...` (admin:senha123).
    *   **Proxy e Token**: Como o Nginx redireciona para o `httpbin.org/get`, a resposta JSON mostrará os headers recebidos pelo backend. Procure por:
        ```json
        {
          "headers": {
            "Authorization": "Bearer meu-token-super-secreto",
            ...
          }
        }
        ```

3.  **Alternativa (Port-Forward)**:
    Caso não tenha o Ingress configurado ou prefira o método direto:
    ```bash
    make port-forward
    # Em outro terminal:
    http -a admin:senha123 localhost:8080
    ```

4.  **Limpar o ambiente**:
    ```bash
    make down
    ```
