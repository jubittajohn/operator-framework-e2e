# operator-framework-e2e
This repository has code changes for cross-component E2E for operator-framework. Currently, the repository only automates tests for operator create, upgrade and delete.

## Pre-requisites for running the test

The below pre-requisites will be automated in the further iterations.

1. Following conditions are assumed to be true:
    
    Kind is downloaded on the system.
    
    A kind cluster is created with the kind cluster name as operator-controller.

    Operator-controller is built and deployed into the new kind cluster.


2. Create and load plain+v0 bundle images onto a test environment.
   
    The plain bundle created should follow the below directory structure:
    bundles/<bundle format>/<bundlename.version>/manifests/<files>
    ```
    bundles/
        └── plain-v0/
            ├── plain.v0.1.0/
            │   ├── manifest
            │   │   ├── namespace.yaml
            │   │   ├── cluster_role.yaml
            │   │   ├── role.yaml
            │   │   ├── serviceaccount.yaml
            │   │   ├── cluster_role_binding.yaml
            │   │   ├── role_binding.yaml
            │   │   └── deployment.yaml
            │   └── Dockerfile
            └──plain.v0.1.1/
                ├── manifest
                │   ├── namespace.yaml
                │   ├── cluster_role.yaml
                │   ├── role.yaml
                │   ├── serviceaccount.yaml
                │   ├── cluster_role_binding.yaml
                │   ├── role_binding.yaml
                │   └── deployment.yaml
                └── Dockerfile
    ```
    The bundle used for the test is present in this repo and follows the structure as above with Configmap.yaml in manifests.

    Run the following command to create the bundle image:
    ```
    <container runtime> build <testdata directory>/bundles/plain-v0/plain.<version> -t localhost/testdata/bundles/plain-v0/plain:<version>
    ```
    Two bundle images are created for this test by running the following two commands:
    ```
    docker build bundles/plain-v0/plain.v0.1.0 -t localhost/bundles/plain-v0/plain:v0.1.0

    docker build bundles/plain-v0/plain.v0.1.1 -t localhost/bundles/plain-v0/plain:v0.1.1
    ```

    Run the following command to load the created bundle image to the cluster:
    ```
    kind load docker-image <image:tag> --name <kind cluster name>
    ```
    The above two images created are loaded into the cluster by running the following commands:
    ```
    kind load docker-image localhost/bundles/plain-v0/plain:v0.1.0 --name operator-controller

    kind load docker-image localhost/bundles/plain-v0/plain:v0.1.1 --name operator-controller
    ```

2. Create and load an FBC image for a set of bundle images onto a test environment.

    The FBC is created for the bundles in the following directory structure.
    ```
    catalogs/
    ├── <catalog-name>/
    │    └── catalog.yaml
    └── Dockerfile    
    ```
    Run the following command to build the image from FBC for the given folder structure:
    ```
    docker build catalogs -f catalogs/test-catalog.Dockerfile -t localhost/testdata/catalogs/catalog:v0.2.0 
    ```

    Run the following command to load the above FBC image to cluster:
    ```
    kind load docker-image localhost/testdata/catalogs/catalog:v0.2.0 --name operator-controller
    ```

3. Hard code the values in the test file. This is to be changed eventually.
    
    `testCatalogRef`    - assign the FBC image reference.
    `testCatalogName`   - assign the catalog name.
    `testOperatorName`  - assign the operator name.
    `createPkgName`     - assign the package name for the operator.

    `createPkgVersion`  - assign the package version for the operator.

    `updatePkgName`     - assign the package name of the operator that is to be upgraded.

    `updatePkgVersion`  - assign the package version for the operator that is to be upgraded.

## Run the test

Once all the prequisites are met, the test can be run which will create, upgrade(from v0.1.0 to v0.1.1) and delete the plain+v0 operator. 
```
go test
```