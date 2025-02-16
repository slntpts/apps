# Adding additional Postgres catalogs to Trino

### Prerequisites
- Must be Operate-First admin with SOPS GPG access

### Steps

1. Clone apps repo
2. Add your PostGreSQL credentials to the secrets file located at `kfdefs/overlays/$ENV/$CLUSTER/$TRINO_FOLDER/secrets/postgres-dbs.enc.yaml`:

> Note: Values for $ENV, $CLUSTER, $TRINO_FOLDER are dependent upon which cluster you are deploying.
> Please explore [kfdefs][kfdefs] overlays folder to identify the values for these variables.

Replace all instance of `<catalog_name_upercase>` with your catalog name in upper case and underscores instead of dash/spaces.

```yaml
kind: Secret
apiVersion: v1
metadata:
    name: postgres-dbs
stringData:
    ...
    # Fill these values out accordingly
    <catalog_name_upercase>_POSTGRESQL_USER:
    <catalog_name_upercase>_POSTGRESQL_PASSWORD:
    <catalog_name_upercase>_POSTGRESQL_DATABASE:
    <catalog_name_upercase>_POSTGRESQL_URL:
type: Opaque
```

> Note that <catalog_name_upercase>_POSTGRESQL_URL should be of the form: jdbc:postgresql://{host}:{port}


3. Navigate to: `odh-manifests/$CLUSTER/$TRINO_FOLDER/base`, open the file called `trino-catalog-secret.yaml`. Append the following
entry to `stringData`, replacing all `<*>` values same as above:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: trino-catalog
stringData:
    .....
  # your catalog name, but replace any spaces/dashes with underscores
  <catalog_name_underscored>.properties: |
    connector.name=postgresql
    connection-url=${ENV:<catalog_name_upercase>_POSTGRESQL_URL}/${ENV:<catalog_name_upercase>_POSTGRESQL_DATABASE}
    connection-user=${ENV:<catalog_name_upercase>_POSTGRESQL_USER}
    connection-password=${ENV:<catalog_name_upercase>_POSTGRESQL_PASSWORD}
```

Commit changes, make a pr.
