# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :** Sarah LOUANES
**Groupe :** RS2

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   >Il y a 3 nœuds : `k3d-lab-server-0` (le control plane, qui gère le cluster) et `k3d-lab-agent-0` / `agent-1` (les workers, qui font tourner les applis). Dans `docker ps`, chaque nœud est un conteneur Docker. Il y a aussi `k3d-lab-serverlb`, le load balancer relié au port 8081.

2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >Ce sont les pods système de k3s, dans le namespace `kube-system` : `coredns` (DNS), `traefik` (Ingress), `metrics-server` (métriques), `local-path-provisioner` (stockage), `svclb-traefik` et des jobs `helm-install` terminés.


## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >L'ancien pod passe en `Terminating` et un nouveau pod est créé tout de suite, avec un nom différent.

4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Le Deployment veut 1 pod. Quand il n'en voit plus, il en recrée un : c'est l'auto-réparation. Kubernetes remet toujours le cluster dans l'état demandé.


## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   Non, on revoit « Welcome to nginx! ». Un conteneur est éphémère : le nouveau pod repart de l'image d'origine, donc les modifications faites avec `exec` sont perdues.


6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Le port-forward est relié à un seul pod précis. Quand ce pod est supprimé, le tunnel s'arrête et ne passe pas tout seul au nouveau pod. Il faut le relancer.



## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   >`create` crée une ressource une seule fois et donne une erreur si elle existe déjà. `apply` part d'un fichier YAML : il crée la ressource ou la met à jour, et on peut le relancer autant de fois qu'on veut. Les fichiers sont gardés dans Git.

8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >Le `selector` (`app: site`) dit au Service vers quels pods envoyer le trafic, grâce à leurs labels. S'il ne correspond à aucun pod, le Service n'a aucun endpoint et le site ne répond pas, sans aucun message d'erreur.


## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ```
NAME                    READY   STATUS        RESTARTS   AGE   IP          NODE               NOMINATED NODE   READINESS GATES
site-55774f9c44-btth8   1/1     Terminating   0          74s   10.42.2.5   k3d-lab-server-0   <none>           <none>
site-f58fdbd6c-gvb8m    1/1     Running       0          7s    10.42.1.7   k3d-lab-agent-1    <none>           <none>
site-f58fdbd6c-pmqzx    1/1     Running       0          23s   10.42.0.6   k3d-lab-agent-0    <none>           <none>
site-f58fdbd6c-wm9nh    1/1     Running       0          16s   10.42.0.7   k3d-lab-agent-0    <none>           <none>
   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le pod était en `ImagePullBackOff` (`0/1`). La commande `kubectl describe pod <nom-du-pod>` montre la cause dans les Events : l'image `nginx:9.99` n'existe pas (`not found`).

11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui. Kubernetes remplace les pods un par un et attend qu'un nouveau pod soit prêt avant de supprimer un ancien. Le nouveau n'a jamais démarré, donc les 3 anciens pods ont continué à servir le site. `kubectl rollout undo` a ensuite remis la version précédente.


## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > Oui, les deux parlent à l'API Server, le point d'entrée du cluster. Portainer est juste une interface graphique par-dessus. Le Deployment `mario` est donc un Deployment normal, avec quelques labels en plus ajoutés par Portainer.


## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** .15.. / .18..
