# Deploy Packaged Applications

This is part of DO280 packaged application chapter. I'm not copying any code from the course.
All I'm doing it to prepare for DO280 and same time learn git
But then folder will have only apps which helps me learn how to deploy Packaged apps basically using templates

## How to copy template and use it in your namespace/project?

```bash
# redirected the output to mysql-persistent.yaml file
oc get templates.template.openshift.io -n openshift mysql-persistent -o yaml > mysql-persistent.yaml
```

## How to find the parameters inside a templates?

```bash
oc process -f mysql-persistent-template.yaml --parameters

NAME                    DESCRIPTION                                                             GENERATOR           VALUE
MEMORY_LIMIT            Maximum amount of memory the container can use.                                             512Mi
NAMESPACE               The OpenShift Namespace where the ImageStream resides.                                      openshift
DATABASE_SERVICE_NAME   The name of the OpenShift Service exposed for the database.                                 mysql
MYSQL_USER              Username for MySQL user that will be used for accessing the database.   expression          user[A-Z0-9]{3}
MYSQL_PASSWORD          Password for the MySQL connection user.                                 expression          [a-zA-Z0-9]{16}
MYSQL_ROOT_PASSWORD     Password for the MySQL root user.                                       expression          [a-zA-Z0-9]{16}
MYSQL_DATABASE          Name of the MySQL database accessed.                                                        sampledb
VOLUME_CAPACITY         Volume space available for data, e.g. 512Mi, 2Gi.                                           1Gi
MYSQL_VERSION           Version of MySQL image to be used (8.0-el7, 8.0-el8, or latest).                            8.0-el8
```

## How to create manifest from the template?

```bash
oc process -f mysql-persistent-template.yaml --param-file mysql-param-file.ini -o yaml > mysql-persistent.yaml
```

## Validate the manifest and apply

```bash
oc create -f mysql-persistent.yaml --validate=true --dry-run=server
### OutPut ###
secret/transvc created (server dry run)
service/transvc created (server dry run)
persistentvolumeclaim/transvc created (server dry run)
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
deploymentconfig.apps.openshift.io/transvc created (server dry run)
### OutPut ###

## apply
oc create -f mysql-persistent.yaml
```

## Something about OC DIFF

oc diff is very smart command. It checks if the deployment can be updated, if it finds it cannot be, it will simply deploy new.
Try to change the `DATABASE_SERVICE_NAME` it will create new deployment, but then try to change the password of `MYSQL_PASSWORD`, it will update it
but of course you must restart the pod. In contrast, change the user name. Nothing will happen, of course secret change can be seen but in pod nothing changes.

While checking this I learnt a sql command how to find users.

```bash
oc exec -it pods/transvc-1-57x9k -- mysql -uroot -e "select user,host from mysql.user;"
### /ö\ OutPut ### /ö\
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| root             | %         |
| transdba         | %         |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
### \ö/ OutPut ### \ö/

```