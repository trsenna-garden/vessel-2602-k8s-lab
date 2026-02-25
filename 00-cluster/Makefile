PROFILE := minikube-k8s-lab

.PHONY: help up down status apply clean

help:
	@echo "Comandos disponíveis:"
	@echo "  make up      - Sobe o cluster Minikube com o perfil '$(PROFILE)' (3 nodes, ingress, metrics-server)"
	@echo "  make down    - Para o cluster '$(PROFILE)'"
	@echo "  make status  - Verifica o status do cluster '$(PROFILE)'"
	@echo "  make apply   - Garante o perfil correto e aplica os namespaces"
	@echo "  make clean   - Remove o perfil '$(PROFILE)' do Minikube"

up:
	minikube start -p $(PROFILE) --nodes 3 --addons=ingress,metrics-server

down:
	minikube stop -p $(PROFILE)

status:
	minikube status -p $(PROFILE)

apply:
	minikube profile $(PROFILE)
	kubectl apply -f k8s/namespaces.yml

clean:
	minikube delete -p $(PROFILE)
