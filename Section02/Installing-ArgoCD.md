* Install Helm:

  ```bash
  curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
  ```

* Install Argo CD on the cluster using Helm as follows:

  ```bash
  helm repo add argo https://argoproj.github.io/argo-helm
  kubectl create namespace argocd
  helm install argocd -n argocd argo/argo-cd
  ```

* Get the administrator password (or just copy the command from the Helm output):

  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
  ```

* Copy the password that was returned.

* Return back to the terminal and install the Nginx ingress controller by running the following command:

  ```bash
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
  ```

* Run the Helm command enabling Ingress and the other required options:

  ```bash
  helm upgrade argocd --set configs.params."server\.insecure"=true --set server.ingress.enabled=true  --set server.ingress.ingressClassName="nginx" -n argocd argo/argo-cd
  ```

* Edit the default ingress and set a hostname for the Argo CD UI:

  ```bash
  kubectl edit ingress argocd-server -n argocd
  ```

  Example:

  ```yaml
  spec:
    ingressClassName: nginx
    rules:
    - host: argocd.local
      http:
        paths:
        - backend:
            service:
              name: argocd-server
              port:
                number: 80
          path: /
          pathType: Prefix
  status:
    loadBalancer:
      ingress:
      - hostname: localhost
  ```

* If you are using your local machine, add the hostname to your hosts file:

  ```bash
  echo "127.0.0.1 argocd.local" | sudo tee -a /etc/hosts
  ```

* Open the browser and navigate to:

  ```text
  http://argocd.local
  ```

* Enter the username `admin` and paste the password from the previous command.

* Get the password again just in case:

  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
  ```

* Navigate to https://github.com/argoproj/argo-cd/releases/latest to get the Argo CD CLI.

* Scroll down till the downloadable files.

* Copy the link to the macOS or Linux binary that matches your machine.

* Install Argo CD CLI by running the following command:

  ```bash
  wget https://github.com/argoproj/argo-cd/releases/download/v2.6.7/argocd-linux-amd64
  sudo mv argocd-linux-amd64  /usr/local/bin/argocd
  chmod +x /usr/local/bin/argocd
  ```

* Login to Argo CD using the following command (username is `admin` and the password can be pasted when prompted):

  ```bash
  argocd login argocd-server \
    --port-forward \
    --port-forward-namespace argocd \
    --plaintext \
    --username admin
  ```

* Verify that you are logged in by running:

  ```bash
  argocd repo list --port-forward --port-forward-namespace argocd --plaintext
  ```

* Change the password using the following command:

  ```bash
  argocd account update-password \
    --port-forward \
    --port-forward-namespace argocd \
    --plaintext \
    --current-password='current-password' \
    --new-password='new-password'
  ```

* If manual `kubectl port-forward` resets the connection in your environment, use the built-in `argocd --port-forward` commands above for CLI access.

  
