# spark-strict

## 1. Install Spark in Strict Mode

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

- Install Spark using the config.json file

```shell
dcos package install --options=/tmp/spark.json spark --yes
```

- Add the name of the principal to the Spark run command

```shell
dcos spark run --verbose --submit-args=" \
--conf spark.mesos.executor.docker.image=mesosphere/spark:2.3.1-2.2.1-2-hadoop-2.6 \
--conf spark.mesos.executor.docker.forcePullImage=true \
--conf spark.mesos.containerizer=mesos \
--conf spark.mesos.principal=spark-principal \
--class org.apache.spark.examples.SparkPi \
https://downloads.mesosphere.com/spark/assets/spark-examples_2.11-2.0.1.jar 30"
```
