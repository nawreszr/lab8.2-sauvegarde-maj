# Lab 8.2 : Sauvegarde, restauration et mise à jour (K3s HA)

## 📖 Description du Lab
Ce laboratoire simule la gestion d'un cluster K3s de production en Haute Disponibilité (HA). Nous avons couvert trois aspects critiques :
1.  **Sauvegarde et Restauration** : Prise d'un snapshot etcd, simulation d'un incident (suppression d'un namespace) et restauration du cluster.
2.  **Maintenance** : Mise en maintenance d'un nœud avec `cordon`, `drain` et `uncordon`, et compréhension de l'affinité des volumes `local-path`.
3.  **Mise à jour (Rolling Update)** : Mise à jour progressive des 3 serveurs sans interruption de service.

---

## 🧠 Questions de réflexion (Réponses)

**1. Pourquoi le fichier important.txt n'est-il pas revenu après la restauration ?**
> Le snapshot etcd ne sauvegarde que **l'état du cluster** (les objets Kubernetes comme les Pods, PVC, ConfigMaps), pas les **données physiques** stockées sur les volumes. Le provisionneur `local-path` a recréé un volume vide lors de la restauration.

**2. Pourquoi le ConfigMap apres-snapshot a-t-il disparu ?**
> Parce qu'il a été créé **après** la prise du snapshot `avant-incident`. La restauration remet le cluster exactement dans l'état où il était au moment du snapshot. Tout ce qui a été fait après est perdu.

**3. Pourquoi arrêter k3s sur tous les serveurs, restaurer sur un seul, puis faire rejoindre les autres avec un état etcd vide ?**
> Pour éviter un conflit de données (split-brain). Si on restaure sur un serveur pendant que les autres tournent, ils ont des états différents. Il faut arrêter tout le monde, restaurer un seul nœud avec le snapshot officiel, puis forcer les autres à oublier leur ancien état (en déplaçant `/var/lib/rancher/k3s/server/db`) pour qu'ils se synchronisent sur le nœud restauré.

**4. Pourquoi le Pod donnees est-il resté Pending pendant le drain ?**
> Parce que son volume est de type `local-path`. Le volume physique est attaché au disque dur d'un seul nœud (`k3s-server1`). Quand on draine ce nœud, le pod ne peut pas aller sur un autre nœud car le volume n'existe pas ailleurs. Il faut attendre que le nœud soit de nouveau disponible (`uncordon`).

**5. Pourquoi mettre à jour un serveur à la fois, et vérifier l'état entre deux mises à jour ?**
> C'est le principe du **Rolling Update** pour la Haute Disponibilité. En mettant à jour un seul serveur à la fois, les deux autres maintiennent le quorum etcd et continuent de servir les applications. Si on mettait à jour les 3 en même temps, le cluster s'arrêterait complètement (perte de quorum).

**6. Pourquoi vaut-il mieux restaurer régulièrement sur un cluster de test ?**
> Pour s'assurer que les sauvegardes sont valides et que la procédure de restauration fonctionne. Une sauvegarde non testée est une fausse sécurité. Cela permet aussi de s'entraîner pour être prêt le jour où un vrai incident de production se produit.

---

## 📸 Preuves de réalisation (Captures d'écran)

### 1. Sauvegarde du snapshot etcd
![Capture Snapshot](captures/Lab8.2-Capture-02-Snapshot-Cree-et-Copie.png)

### 2. Incident : Suppression du namespace
![Capture Incident](captures/Lab8.2-Capture-03-Incident-Namespace-Supprime.png)

### 3. Restauration réussie
![Capture Restauration](captures/Lab8.2-Capture-01-Application-Restauree.png)

### 4. Données du volume perdues
![Capture Données Perdues](captures/Lab8.2-Capture-05-Donnees-Volume-Perdues.png)

### 5. Maintenance (Drain et Pod Pending)
![Capture Maintenance](captures/Lab8.2-Capture-06-Drain-Noeud-Pod-Pending.png)

### 6. Mise à jour des 3 serveurs
![Capture Mise à Jour](captures/Lab8.2-Capture-07-Mise-A-Jour-Reussie.png)
