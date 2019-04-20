# springboot-chart
springboot chart for helm

Configurable parameters

```yaml

#**********************************************************#
#                      == Deployment ==                    #
#**********************************************************#

######---类型设置 (可选为Deployment、StatefulSet，默认为：Deployment)---######
kind: Deployment

######---镜像设置---######
image:
  repository: "mydlqclub/springboot-helloworld"
  tag: "0.0.1"
  pullPolicy: "IfNotPresent"

######---副本数设置---######
replicas: 1

######---Resources资源限制设置---######
resources:
  limits:
    memory: 512Mi
    cpu: 1000m
  requests:
    memory: 256Mi
    cpu: 500m

######---Pod podAnnotations设置---######
podAnnotations:
#  prometheus.io/scrape: "true"
#  prometheus.io/port: "9030"
#  prometheus.io/path: "/"

######---terminationGracePeriodSeconds优雅关闭Pod设置(默认30s)---######
terminationGracePeriodSeconds: 10

######---环境变量ENV设置---######
env: 
  - name: "JAVA_OPTS"
    value: "-Dserver.port=8080"
  - name: "APP_OPTS"
    value: "" 

######---Port端口设置---######
ports: 
  - name: server
    containerPort: 8080
    protocol: TCP
  - name: management
    containerPort: 8081
    protocol: TCP

######---Read && Live 探针设置---######
probe:
  port: 8081
  path: /actuator/health
  livenessProbe:
    initialDelaySeconds: 30
    periodSeconds: 10
    successThreshold: 1
    timeoutSeconds: 3
  readinessProbe:
    initialDelaySeconds: 5
    periodSeconds: 10
    successThreshold: 1
    timeoutSeconds: 3
  terminationGracePeriodSeconds: 10

######---启动节点设置设置---######
nodeSelector: 
  # selectNode: master

######---Volume && VolumeMounts设置---######
mount:
  volumeMounts: 
    - name: spring-app-config
      mountPath: /opt/deployments/config
      readOnly: true
  volumes: 
    - name: spring-app-config
      configMap:
        name: config
        items:
        - key: application.yml
          path: application.yml
    - name: datadir
      persistentVolumeClaim:
        claimName: datadir-pvc
  volumeClaimTemplates: #(部署类型必须是：StatefulSet)
    - metadata:
        name: www
      spec:
        accessModes: ["ReadWriteOnce"]
        volumeMode: Filesystem
        resources:
          requests:
            storage: 50Mi
        storageClassName: local-storage

######---Affinity设置---######
affinity: 
  # nodeAffinity: 
  #   #节点亲和性
  #   requiredDuringSchedulingIgnoredDuringExecution:  #硬策略
  #     nodeSelectorTerms:
  #     - matchExpressions:
  #       - key: kubernetes.io/hostname
  #         operator: NotIn
  #         values:
  #         - k8s-node-2-12
  #   preferredDuringSchedulingIgnoredDuringExecution: #软策略
  #   - weight: 1   #取值范围1-100
  #     preference:
  #       matchExpressions:
  #       - key: kube
  #         operator: In
  #         values:
  #         - test

######---Tolerations容忍设置---######
tolerations: 
  # - operator: "Exists"

#**********************************************************#
#                      == Service ==                       #
#**********************************************************#

service:
  ######---Service type设置 (可以设置为ClusterIP、NodePort、None)---######
  type: NodePort
   ######---Service Labels设置---######
  labels: 
    # svcEndpoints: actuator
  ######---Service Annotations设置---######
  annotations: {}
  ######---Service Ports设置---######
  ports: 
    - name: server
      port: 8080
      targetPort: 8080
      protocol: TCP
      # 如果type为NodePort,则可以设置下面的nodePort端口,否则请不要设置
      # nodePort: 30080 
    - name: management
      port: 8081
      targetPort: 8081
      protocol: TCP
      # 如果type为NodePort,则可以设置下面的nodePort端口,否则请不要设置
      # nodePort: 30081
```