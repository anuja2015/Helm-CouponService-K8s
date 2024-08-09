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
7. Use Configmap
8. Release and test
