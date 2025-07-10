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

## Now you should switch to branch v1

Will play little bit here. Change to v1 branch. You can clone the repo of v1 branch using the command

git clone --branch v1 https://github.com/letsdodevelopment/packagedapps.git

cd packagedapps

### Make changes

using your editor, change DATABASE_SERVICE_NAME parameters in mysql-param-file.ini file. Now overwrite
mysql-persistent.yaml, using the following command

```bash
oc process -f mysql-persistent-template.yaml --param-file mysql-param-file.ini -o yaml > mysql-persistent.yaml

# check the changes in the file using the following command

git diff .
```

when all looks okay and expected, run `oc diff -f mysql-persistent.yaml`, you will observe a completely new file is created. We do not want this. So got ahead and revert the changes, you do using `git restore .` command

now make changes to the password of the database i.e. MYSQL_PASSWORD, if you are in v1, you will see password is already changed. You can double check this using `git show` command

```bash
➜  packagedapps git:(v1) git show
commit f5b7bb0f9456fc005d21bf7cb3c623cfe1f4f86b (HEAD -> v1, origin/v1)
Author: repolevedp <repolevedp@gmail.com>
Date:   Thu Jul 10 12:31:03 2025 +0200

    just changed the password of the user

diff --git a/mysql-param-file.ini b/mysql-param-file.ini
index c21e336..daf9696 100644
--- a/mysql-param-file.ini
+++ b/mysql-param-file.ini
@@ -1,6 +1,6 @@
 DATABASE_SERVICE_NAME=transvc
 MYSQL_USER=transdba
-MYSQL_PASSWORD=passit
+MYSQL_PASSWORD=passit123456
 MYSQL_ROOT_PASSWORD=passitRoot123#
 MYSQL_DATABASE=itemsx
 VOLUME_CAPACITY=2Gi
diff --git a/mysql-persistent.yaml b/mysql-persistent.yaml
index 352c12c..ebe7377 100644
--- a/mysql-persistent.yaml
+++ b/mysql-persistent.yaml
@@ -14,7 +14,7 @@ items:
     name: transvc
   stringData:
     database-name: itemsx
-    database-password: passit
+    database-password: passit123456
     database-root-password: passitRoot123
     database-user: transdba
 - apiVersion: v1
```

Now just run `oc apply -f mysql-persistent.yaml --validate=true --dry-run=server` and then execute it using oc apply -f mysql-persistent.yaml

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