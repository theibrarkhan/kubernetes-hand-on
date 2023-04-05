# Voting application Deployed on K8s

In the project we have five component

## Vote Frontend

In this deployment we have define the frontend of vote application where you will submit your vote

```

kubectl apply -f vote-deployment

```

## Redis Deployment

redis will cache your result then sync it with db and also we have a service

```

kubectl apply -f redis-deployment

```

## worker Deployment

```

kubectl apply -f worker-deployment

```

## db Deployment

```

kubectl apply -f db-deployment

```

## result Deployment

in the deployment we define the result of the voting

```

kubectl apply -f result-deployment

```

## Architecture

![alt text](architecture.excalidraw.png)

### emptyDir

An emptyDir volume is first created when a Pod is assigned to a node, and exists as long as that Pod is running on that node. As the name says, the emptyDir volume is initially empty. All containers in the Pod can read and write the same files in the emptyDir

![learn more ](https://kubernetes.io/docs/concepts/storage/volumes/

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to deal with them in Docker at a basic level.
