---
layout: post
title: Deploy Kafdrop on Kubernetes
---

When it comes to Kafka topic viewers and web UIs, the go-to open-source tool is Kafdrop. With 800K Docker pulls at the time of writing, there aren’t many Kafka tools that have enjoyed this level of adoption. And there’s a reason behind that: Kafdrop does an amazing job of filling the apparent gaps in the observability tooling of Kafka, solving problems that the community has been pointing out for too long.

Kafdrop is an Apache 2.0 licensed project, like Apache Kafka itself. So it’s won’t cost you a penny. If you haven’t used it yet, you probably ought to. So let’s take a deeper look.

### Deploy Kafdrop on K8s
You can also install KafDrop on the Kubernetes cluster with the helm of the manifest file. Create a YAML file called kafdrop-deployment.yaml with the following content in it:

#### kafdrop-deployment.yaml
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kafka-kafdrop-deployment
  namespace: kafdrop
  labels:
    app: kafka-kafdrop
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kafka-kafdrop
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: kafka-kafdrop
    spec:
      volumes:
        - name: tz-config
          hostPath:
            path: /usr/share/zoneinfo/Asia/Jakarta
      containers:
        - image: obsidiandynamics/kafdrop
          imagePullPolicy: Always
          name: kafka-kafdrop
          volumeMounts:
            - name: tz-config
              mountPath: /etc/localtime
          resources:
            limits:
              cpu: 200m
              memory: 500Mi
            requests:
              cpu: 200m
              memory: 500Mi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          ports:
            - containerPort: 5010
              name: server
            - containerPort: 5012
              name: jmx
          env:
            - name: JVM_OPTS
              value: "-Xms512M -Xms512M"
            - name: SERVER_SERVLET_CONTEXTPATH
              value: "/"
            - name: KAFKA_BROKERCONNECT
              value: "your kafka broker IP and Port"
            - name: KAFKA_PROPERTIES
              value: "your kafka config.properties decode to base64"
            - name: KAFKA_TRUSTSTORE
              value: "your kafka truestore"
            - name: KAFKA_KEYSTORE
              value: "your kafka keystore"
            - name: CMD_ARGS
              value: "--topic.deleteEnabled=false --topic.createEnabled=false"
      restartPolicy: Always
```
#### kafdrop-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: kafka-kafdrop-service
  namespace: kafdrop
  labels:
    app: kafka-kafdrop
spec:
  ports:
    - protocol: "TCP"
      port: 9000
      name: server
      nodePort: 30766
  selector:
    app: kafka-kafdrop
  type: NodePort #LoadBalancer
  
```


May this is help for you who implement DevSecOps. 
