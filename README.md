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

In values.yaml, provide one more value to be overridden under mysql

         initdbScriptsConfigMap: mysql-initdb-config


8. Release and test

        $ helm install couponservice .        when you are inside the chart folder!


            PS C:\Users\AnujaM\Desktop\Anuja_M\Interview\Projects\Helm-usecase1\couponservicechart> kubectl exec -it docker-mysql-0 -- /bin/bash

            Defaulted container "mysql" out of: mysql, preserve-logs-symlinks (init)
            I have no name!@docker-mysql-0:/$  mysql -u root -ptest1234
            mysql: [Warning] Using a password on the command line interface can be insecure.
            Welcome to the MySQL monitor.  Commands end with ; or \g.
            Your MySQL connection id is 369
            Server version: 8.4.0 Source distribution

            Copyright (c) 2000, 2024, Oracle and/or its affiliates.

            Oracle is a registered trademark of Oracle Corporation and/or its
            affiliates. Other names may be trademarks of their respective
            owners.

            Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

            mysql> use mydb;
            Reading table information for completion of table and column names
            You can turn off this feature to get a quicker startup with -A

            Database changed
            mysql> use mydb;
            Reading table information for completion of table and column names
            You can turn off this feature to get a quicker startup with -A

            Database changed
            mysql> select * from coupon
                -> ;
            Empty set (0.00 sec)

            mysql> insert into coupon (code,discount,exp_date) values('SUPERSALE',10,'22-12-2024');
            Query OK, 1 row affected (0.01 sec)

            mysql> select * from coupon;
            +----+-----------+----------+------------+
            | id | code      | discount | exp_date   |
            +----+-----------+----------+------------+
            |  1 | SUPERSALE |   10.000 | 22-12-2024 |
            +----+-----------+----------+------------+
            1 row in set (0.00 sec)

            mysql> 


Since we are using minikube and we have used NodePort, it is accessible only inside the cluster network. To access the api on our local machine use kubectl port-forward.

            $ kubectl port-forward svc/docker-mysql 32762:3306

            $ kubectl port-forward svc/coupon-service 32763:9091
