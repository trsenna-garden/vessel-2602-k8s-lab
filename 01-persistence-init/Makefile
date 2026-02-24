apply:
	kubectl apply -f k8s/

apply-namespace:
	kubectl apply -f k8s/namespace.yml

config-set-context:
	kubectl config set-context --current --namespace=k8s-lab-01-persistence-unit

config-view-namespace:
	kubectl config view --minify | grep namespace:

port-forward:
	kubectl port-forward deploy/app-nginx 8080:80
