Now, let's check that the data was persisted and is accessible on another node.

Make a note of the location on which the volume got initially provisioned on:

`kubectl exec -ti cli -n kube-system -- storageos get volume`{{execute}}

Save the namespace/volume into a variable.

`volume=$(kubectl exec -ti cli -n kube-system -- storageos get volume | grep '0/0' | awk '{ print $2 }')`{{execute}}

Create 1 replica for the volume, by adding a label to the volume.

`kubectl exec -ti cli -n kube-system -- storageos update volume replicas $volume 1`{{execute}}

Check that the replica was created.

`kubectl exec -ti cli -n kube-system -- storageos get volume`{{execute}}

> Usually takes ~10 seconds to create the replica in this example.

In order to keep this tutorial simple, we are going to delete the MySQL pod and create it on a different node in our cluster.

`kubectl delete pod mysql`{{execute}}

Now, you can see the mount was removed but the location is still the same.

`kubectl exec -ti cli -n kube-system -- storageos get volume`{{execute}}


Create the MySQL pod again but on a different node this time.

`kubectl create -f mysql-pod2.yaml`{{execute}}

`kubectl get pods -w`{{execute}}

> The above command watches the pod created in the default namespace. Press `Ctrl+C` to continue once the pods are up.

Connect to the database.

`kubectl exec -it mysql -- mysql`{{execute}}

`USE shop;`{{execute}}

Insert more data, then check that there are four records in the table.

`INSERT INTO FRUIT (ID,INVENTORY,QUANTITY) VALUES (4, 'Peaches', 203);`{{execute}}

`SELECT * FROM FRUIT;`{{execute}}

Quit the MySQL container.

`\q`{{execute}}

If you check the location of the volume you will see it has not changed,
despite the volume now being mounted by a different host.

`kubectl exec -ti cli -n kube-system -- storageos get volume`{{execute}}

In this way you can see that the location of the volume is transparent to the
pod that mounts the volume.

Congratulations, you have successfully installed StorageOS on Kubernetes, created a volume, and bound that volume to an application!