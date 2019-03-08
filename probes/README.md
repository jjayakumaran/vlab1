<img src="../assets/k8sland.png" align="right" width="128" height="auto"/>

<br/>

# <img src="../assets/lab.png" width="32" height="auto"/> Deployment Lab

> The Hangman Game Reloaded!

> The hangman game is composed of two separate services namely: **Dictionary** and **Hangman**.
> **Dictionay** provides a list of words loaded from a dictionary asset directory.
> **Hangman** calls up to the dictionary to get a list of words and picks a random word.
> It provides apis for creating a new game and validating your letter guesses.

> NOTE! For this lab you can copy your deployment lab solution or use our solution as we will
> enhancing the manifests.

1. Create a namespace for the services called 'hm' for hangman.
2. Dictionary Service: Create a deployment and service
   1. Use the following image and tag: *k8sland/go-dictionary-svc:0.0.1*
   2. Configure the dictionary pod command as follows:
      1. Command: /app/dictionary
      2. Arguments
         1. `-a assets` specifies the dictionaries asset directory.
         2. `-d words.txt` specifies a dictionary to use.
   5. Make sure your dictionary container exposes port *4000*
   6. Configure the dictionary services as follows:
      1. Service type: *ClusterIP*
      2. Service is exposed on port: *4000*
   7. Configure probes and resource for your dictionary.
      1. You'll need to configure both a readiness and liveliness probes using `/api/v1/healthz`
   8. Setup an HPA on your dictionary pod. Set the target to 30% cpu and max 5 replicas
3. Provision your *Dictionary* deployment and service and HorizontalPodAutoscaler
4. Verify your deployment, service and hpa are happy!
5. Verify you can get a list of words from the dictionary service (/api/v1/words)
6. Hangman Service: Create a deployment and service
   1. Use the following image and tag: *k8sland/go-hangman-svc:0.0.1*
   2. Configure the hangman pod command as follows:
      1. Command: /app/hangman
      2. Arguments
         1. `-d dnsname:port` specifies the dictionaries dns name and port
   3. Make sure your hangman container exposes port *5000*
   4. Configure the hangman service as follows:
      1. Service type: *NodePort*
      2. Service is exposed on nodePort: *30500*
   5. Configure probes and resource for your hangman service.
      1. You'll need to configure both a readiness and liveliness probes using `/api/v1/healthz`
   6. Setup an HPA on your hangman pod. Set the target to 30% cpu and max 5 replicas
7. Provision your *Hangman* deployment and service
8. Verify your deployment and service are happy!
9.  Verify you can create a new game from the Hangman service (/api/v1/new_game)
10. Fire off the Hangmam cli enjoy the fruits of your labor!!
11. Delete your entire application!


<br/>

---
## Happy Installs [OPTIONAL]

We recommand for the lab to install a couple useful utilities if you're on OSX

1. Install [HTTPIE](https://httpie.org)

   ```shell
   brew install httpie
   # Verify!
   http google.com
   ```

2. Install [WATCH](http://osxdaily.com/2010/08/22/install-watch-command-on-os-x)

    ```shell
    brew install watch
    # Verify!
    watch --help
    ```

<br/>

---
## Commands

- Create the **hm** namespace

  ```shell
  kubectl apply -f k8s/ns.yml
  # Verify!
  kubectl get ns
  ```

- Create Dictionary deployment and service

  ```shell
  kubectl apply -f k8s/dictionary.yml
  ```

- Verify Dictionary service is happy

  ```shell
  kubectl get -n hm deploy
  kubectl get -n hm pod
  # Or...
  watch kubectl get -n hm deploy,po,svc
  ```

- Verify we can get some words!

  ```shell
  export D_POD=$(kubectl get pods -n hm -l app=dictionary -o jsonpath={.items[0].metadata.name})
  kubectl port-forward $D_POD 4001:4000 &
  http --print=b --pretty=colors :4001/api/v1/words
  # OR...
  curl -XGET http://localhost:4001/api/v1/words
  ```

- Create Hangman deployment and service

  ```shell
  kubectl apply -f k8s/hangman.yml
  ```

- Verify Hangman service is happy

  ```shell
  kubectl get -n hm deploy
  kubectl get -n hm pod
  # Or...
  watch kubectl get -n hm deploy,po,svc
  ```

- Ensure Hangman can create a game!

  ```shell
  http --print=b --pretty=colors $(minikube ip):30500/api/v1/new_game
  # OR...
  curl -XGET http://$(minikube ip):30500/api/v1/new_game
  ```

- Play Hangman

   ```shell
   kubectl run -i --tty --rm hangmancli -n hm --generator=run-pod/v1 \
   --image k8sland/go-hangman-cli:0.0.1 \
   --command -- /app/hangman_cli --hm hangman:5000
   ```

- Create some load

  ```shell
  ab -n 100000 -c 2 http://$(minikube ip):30500/api/v1/new_game
  # Or...
  for i in {1..1000}; do wget -qO /dev/null http://$(minikube ip):30500/api/v1/new_game; done
  ```

- Delete the entire application

  ```shell
  kubectl delete -f k8s
  # OR... (NOTE!!)
  kubectl delete ns hm
  ```

<br/>

---
<img src="../assets/imhotep_logo.png" width="32" height="auto"/> © 2018 Imhotep Software LLC.
All materials licensed under [Apache v2.0](http://www.apache.org/licenses/LICENSE-2.0)