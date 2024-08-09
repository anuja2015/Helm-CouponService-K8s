# Helm-CouponService-K8s

### __Steps__

1. Create Chart
            $ helm create couponservicechart
            Creating couponservicechart


2. Configure Image
 In values.yaml configure the image
                image:
                  registry: docker.io
                  repository: bharatht19/couponservice

3. Update deployment
In values.yaml configure livenessprobe and readiness probe
        
         livenessProbe:
           httpGet:
             path: /actuator/health/liveness
             port: tomcat
         readinessProbe:
           httpGet:
             path: /actuator/health/readiness
             port: tomcat
4. Configure service type

In values.yaml Configure port type

        service:
          type: NodePort
          targetPort: 9091
          port: 9091
          nodePort: 32716

In service.yaml

        port: {{ .Values.service.port }}
        targetPort: {{ .Values.service.port }}
        nodePort: {{ .Values.service.nodePort }}
        protocol: TCP
        name: tomcat
5. Add dependency

Add dependencies in Chart.yaml

            dependencies:
              - name: mysql
                version: 11.1.2
                repository: oci://registry-1.docker.io/bitnamicharts

6. Pass values

To override the default values in the dependent chart -mysql configure values.yaml

            mysql:
                auth:
                    rootPassword: test1234
                    database: mydb
                primary:
                  service:
                    type: NodePort
                    nodePort: 32762
                fullnameOverride: docker-mysql
    
7. Use Configmap

Create a configmap with initdb script which will initialise the database when created with a table and data in the table.

In values.yaml, provide one more value to be overridden


8. Release and test
