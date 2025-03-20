Deploying
We will use 2 terminal sessions to the machine you will use for hosting the docker containers. Each of the code stanzas below will state which terminal to use. This makes it easier to see output logs and to avoid repeatedly changing directory.

First bring up the trillian instance and the database:

# Terminal 1
```
cd ${GIT_HOME}/certificate-transparency-go/trillian/examples/deployment/docker/ctfe/
docker compose up
```

This brings up everything except the CTFE. Now to provision the logs.

# Terminal 2
```
docker exec -i ctfe-db mariadb -pzaphod -Dtest < ${GIT_HOME}/trillian/storage/mysql/schema/storage.sql
docker exec -i ctfe-db mariadb -pzaphod -Dtest < ${GIT_HOME}/certificate-transparency-go/trillian/ctfe/storage/mysql/schema.sql
```

The CTFE requires some configuration files. First prepare a directory containing these, and expose it as a docker volume. These instructions prepare this config at /tmp/ctfedocker but if you plan on keeping this test instance alive for more than a few hours then pick a less temporary location on your filesystem.

# Terminal 2
```
CTFE_CONF_DIR=/tmp/ctfedocker
mkdir ${CTFE_CONF_DIR}
TREE_ID=$(go run github.com/google/trillian/cmd/createtree@master --admin_server=localhost:8090)
sed "s/@TREE_ID@/${TREE_ID}/" ${GIT_HOME}/certificate-transparency-go/trillian/examples/deployment/docker/ctfe/ct_server.cfg > ${CTFE_CONF_DIR}/ct_server.cfg
cp ${GIT_HOME}/certificate-transparency-go/trillian/testdata/fake-ca.cert ${CTFE_CONF_DIR}
docker volume create --driver local --opt type=none --opt device=${CTFE_CONF_DIR} --opt o=bind ctfe_config
```

```
sed "s/@TREE_ID@/${TREE_ID}/" ct_server.cfg > ctfe_config/ct_server.cfg
cp tests/certs/root_cert.pem ctfe_config/fake-ca.cert
docker volume create --driver local --opt type=none --opt device=${CTFE_CONF_DIR} --opt o=bind ctfe_config
```

Now that this configuration is available, you can bring up the CTFE:

# Terminal 1

```
<Ctrl C> # kill the previous docker compose up command
tilt up
```

# Terminal 2

Assuming there are no errors in the log, then the following command should return tree head for tree size 0.

```
go run ./client/ctclient get-sth --log_uri http://localhost:8080/testlog
```