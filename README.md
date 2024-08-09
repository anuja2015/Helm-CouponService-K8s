# Helm-CouponService-K8s

### __Steps__

1. Create Chart

            $ helm create couponservicechart
            Creating couponservicechart

<img width="764" alt="Screenshot 2024-08-09 182447" src="https://github.com/user-attachments/assets/4030cd37-a457-43d4-aa87-c1b7850df2d0">


2. Configure Image

 In values.yaml configure the image

                image:
                  registry: docker.io
                  repository: bharatht19/couponservice
![Screenshot 2024-08-09 183112](https://github.com/user-attachments/assets/062309ae-107c-4c05-bff7-85349ccc9a6e)

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
             
![Screenshot 2024-08-09 192056](https://github.com/user-attachments/assets/c9b86063-a26d-4b34-8e0c-47610fbd3010)

![Screenshot 2024-08-09 192041](https://github.com/user-attachments/assets/020ef7de-b9f7-47cc-bbfd-1677fa4330be)

4. Configure service type

In values.yaml Configure port type

        service:
          type: NodePort
          targetPort: 9091
          port: 9091
          nodePort: 32716

![Screenshot 2024-08-09 192545](https://github.com/user-attachments/assets/8736cf76-b571-46f4-a228-fe0d0c187894)

In service.yaml

        port: {{ .Values.service.port }}
        targetPort: {{ .Values.service.port }}
        nodePort: {{ .Values.service.nodePort }}
        protocol: TCP
        name: tomcat
        
 ![Screenshot 2024-08-09 192531](https://github.com/user-attachments/assets/0276d5ea-7a23-4a98-b68b-dafabefdc231)

5. Add dependency

Add dependencies in Chart.yaml

            dependencies:
              - name: mysql
                version: 11.1.2
                repository: oci://registry-1.docker.io/bitnamicharts

<img width="485" alt="Screenshot 2024-08-09 193240" src="https://github.com/user-attachments/assets/24c8cb18-efa1-4580-9127-410dee49cc41">

![Screenshot 2024-08-09 195509](https://github.com/user-attachments/assets/a869f07b-d991-480b-8fc0-18a4427595b8)

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
                
![Screenshot 2024-08-09 212002](https://github.com/user-attachments/assets/61f68282-3db2-45b1-bd9e-5a59baad9df2)

7. Use Configmap

Create a configmap with initdb script which will initialise the database when created with a table and data in the table.

In values.yaml, provide one more value to be overridden under mysql

         initdbScriptsConfigMap: mysql-initdb-config
![Screenshot 2024-08-09 223237](https://github.com/user-attachments/assets/b1ba38ba-80c9-4a65-9c93-ddee46a55709)


8. Release and test

        $ helm install couponservice .        when you are inside the chart folder!

![Screenshot 2024-08-09 212255](https://github.com/user-attachments/assets/97942f88-0d60-4ae3-9b27-7422e568eeca)

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

![Screenshot 2024-08-09 222119](https://github.com/user-attachments/assets/2fcdc6f9-487a-4738-95f3-fcfe9d2fe1d5)
![Screenshot 2024-08-09 222134](https://github.com/user-attachments/assets/0490d74f-ca0d-45ca-b189-116e172d88b0)
![Screenshot 2024-08-09 222147](https://github.com/user-attachments/assets/c70706b8-f282-40cb-94ec-ca6bdb093af9)
![Screenshot 2024-08-09 222214](https://github.com/user-attachments/assets/c91489c3-c855-4444-8c07-26c5220e56ec)

Since we are using minikube and we have used NodePort, it is accessible only inside the cluster network. To access the api on our local machine use kubectl port-forward.

            $ kubectl port-forward svc/docker-mysql 32762:3306

            $ kubectl port-forward svc/coupon-service 32763:9091

![Screenshot 2024-08-09 222247](https://github.com/user-attachments/assets/4cf39bad-e2bf-47f0-9d66-563c61dd3be0)
![Screenshot 2024-08-09 222302](https://github.com/user-attachments/assets/dd8fc77b-4ce7-4e40-9d3e-60b946f31298)
