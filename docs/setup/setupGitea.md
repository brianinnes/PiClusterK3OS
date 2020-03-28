# Gitea - local Git server

This section will deploy a local git server, using Gitea.  A Postgres database will be used to provide the backing storage for the git server.

To deploy Postgres and Gitea execute the following:

```
kubectl apply -f postgres.yaml
kubectl apply -f gitea.yaml
```

You need to create a Postgres password, which needs to be base64 encoded.  You can create this using command ```echo -n "password" | base64```, replacing password with the password you want to use for the database, then store the password as a secret with the following command, again replacing the value of the password property with the output from the previous command:

```bash
kubectl apply -f -<<EOF
apiVersion: v1
kind: Secret
metadata:
  namespace: gitea
  name: gitea-db-password
type: Opaque
data:
  password: cGFzc3dvcmQ=
EOF
```

Once started open the gitea UI, then register to setup the server

Database Type:  PostgreSQL
Host:           gitea-db-service.gitea.svc.cluster.local:5432
Username:       gitea
Password:       <password entered as base64 encoded secret - type password in here, not base64>
Database Name:  gitea
SSL:            Disable
Schema:         <leave blank>

Site Title:     <Set to string of choice>
SSH Server Domain: gitea.bik3s.home <this is what the ingress hostname is>
Gitea Base URL: http://gitea.bik3s.home/


Note user passwords need:
- upper case character
- lower case character
- number
- special character ```!"#$%&'()*+,-./:;<=>?@[\\]^_`{|}~```

Points to investigate follow up on:
- Build for arm architecture.  Written in go, so hopefully not too big a deal?