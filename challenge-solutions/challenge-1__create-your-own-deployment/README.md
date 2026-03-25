# Steps for my own [`quote.yml`](./quote.yml)

1. Create the deployment using [`quote.yml`](./quote.yml)

   ```sh
   kubectl apply -f quote.yml
   ```

   > To verify, check whether the pods are created or not: `kubectl get pods -n development`

2. Ensure that `BusyBox` pod already exists, if not, create it from steps mentioned in [README.md's Application/Pod Verification Using `BusyBox`](../../README.md#applicationpod-verification-using-busybox).
3. Get the IP of one of the replicas of `quote-service` pod that was just created using:

   ```sh
   kubectl get pods -n development -o wide
   ```

   You should see something like the following:

   ```terminal
   NAME                                   READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
   quote-service-f5bdb8575-rfrz9          1/1     Running   0          56m   10.244.0.10   minikube   <none>           <none>
   quote-service-f5bdb8575-wks6q          1/1     Running   0          56m   10.244.0.9    minikube   <none>           <none>
   ```

4. Login to `BusyBox`: [Get `BusyBox`'s pod name using `kubectl get pods`]

   ```sh
   kubectl exec -it <busybox-pod-name> -- /bin/sh
   ```

5. Once logged in, in the shell, try the following commands:

   ```sh
   wget -q0- http://10.244.0.10:8080
   ```

   You should get a random quote inside a JSON object:

   ```json
   {
    "server": "fuzzy-banana-atnh9rgt",
    "quote": "The last sentence you read is often sensible nonsense.",
    "time": "2026-03-25T18:17:12.587764416Z"
   }
   ```

   The more you run the command: `wget -q0- http://10.244.0.10:8080`, the more quotes you'll get back.
   Some of the quotes that I got back are:

   ```json
   {
    "server": "fuzzy-banana-atnh9rgt",
    "quote": "Non-locality is the driver of truth. By summoning, we vibrate.",
    "time": "2026-03-25T18:08:00.470804891Z"
   }
   ```

   ```json
   {
    "server": "fuzzy-banana-atnh9rgt",
    "quote": "Abstraction is ever present.",
    "time": "2026-03-25T18:07:48.903543734Z"
   }
   ```

## Instructor's Solution

- Instructor did everything that I did, except that, the way of querying, was to remove the existing `index.html`, and run the following command from inside the `busybox` pod:

  ```sh
  rm index.html
  wget 10.244.0.10:8080
  cat index.html
  ```

  O/P:

  ```json
  {
    "server": "fuzzy-banana-atnh9rgt",
    "quote": "A principal idea is omnipresent, much like candy.",
    "time": "2026-03-25T18:38:47.140691459Z"
  }
  ```
