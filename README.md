## 1. Connect to the lab cluster (if not already done)

```
dcos cluster setup <cluster url>
```

For example:

```
dcos cluster setup http://egoode-no-elasticl-aod2qsebzlxm-1269016722.us-west-2.elb.amazonaws.com
```

## 2. Install Enterprise CLI (if not already done)

```
dcos package install dcos-enterprise-cli
```

## 3. Set up service account

- Setup service account and secret

```shell
dcos security org service-accounts keypair /tmp/spark-private.pem /tmp/spark-public.pem
dcos security org service-accounts create -p /tmp/spark-public.pem -d "Spark service account" spark-principal
dcos security secrets create-sa-secret --strict /tmp/spark-private.pem spark-principal spark/secret
```

- Grant permissions to the Spark Service Account

```shell
dcos security org users grant spark-principal dcos:mesos:agent:task:user:root create
dcos security org users grant spark-principal "dcos:mesos:master:framework:role:*" create
dcos security org users grant spark-principal dcos:mesos:master:task:app_id:/spark create
dcos security org users grant spark-principal dcos:mesos:master:task:user:nobody create
dcos security org users grant spark-principal dcos:mesos:master:task:user:root create
```

- Grant  permissions to Marathon in order to the Spark the dispatcher in root

```shell
dcos security org users grant dcos_marathon dcos:mesos:master:task:user:root create
```

## 4. Create your custom configuration for Spark

- Create a configurationfile **/tmp/spark.json** and set the Spark principal and secret

```json
cat <<EOF > /tmp/spark.json
{
  "service": {
    "name": "spark",
    "service_account": "spark-principal",
    "service_account_secret": "spark/secret",
    "user": "root"
  }
}
EOF
```

## 5. Install Spark using the custom configuration file

```shell
dcos package install --options=/tmp/spark.json spark --yes
```

You can verify the cli is ready and see available options by running

```
dcos spark
```

## 6.  Run the sample application

```shell
dcos spark run --verbose --submit-args=" \
--conf spark.mesos.executor.docker.image=mesosphere/spark:2.3.1-2.2.1-2-hadoop-2.6 \
--conf spark.mesos.executor.docker.forcePullImage=true \
--conf spark.mesos.containerizer=mesos \
--conf spark.mesos.principal=spark-principal \
--class org.apache.spark.examples.SparkPi \
https://downloads.mesosphere.com/spark/assets/spark-examples_2.11-2.0.1.jar 30"
```

## 7. See the status of your job

```
dcos spark status <driver id>
```

## 8. See the output of the sample application

```
dcos spark log <driver id>


