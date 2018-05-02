# sonar-pipeline
 [![Build Status][status-image]][status-url] [![Open Issues][issues-image]][issues-url]![License][license-image] [![Slack][slack-image]][slack-url]

Pipeline to deploy sonarqube server to kubernetes. Currently it only supports updating sonarqube's image. Other configuration requires manual update via helm.

## First Time Deployment
To simplify the deployment, we are using helm, the package manager for k8s, to bundle all we need for sonar and deploy them all at once. Example `values.yaml` can be found under `config-example` folder. The actual config files are stored somewhere else secretly.
- Prerequisite:
    1. install helm client on your dev machine: `brew install kubernetes-helm`
    2. init helm in k8s cluster: https://docs.helm.sh/using_helm
- Deployment:
    1. Switch to the desired context
    ```bash
    kubectl config use-context desired-cluster-context
    ```

    2. Deploy beta sonar using sonarqube chart
    ```bash
    helm install --name beta -f beta-values.yaml stable/sonarqube
    ```

    3. Deploy prod sonar using sonarqube chart
    ```bash
    helm install --name prod -f values.yaml stable/sonarqube
    ```

    After deployment, you should be able to see such a set of resources created for sonarqube, e.g. for beta:

    ```bash
    ==> v1/Service
    beta-postgresql  
    beta-sonarqube   

    ==> v1beta1/Deployment
    beta-postgresql  
    beta-sonarqube   

    ==> v1beta1/Ingress
    beta-sonarqube

    ==> v1/Pod(related)
    beta-postgresql
    beta-sonarqube

    ==> v1/Secret
    beta-postgresql

    ==> v1/ConfigMap
    beta-sonarqube-config         
    beta-sonarqube-install-plugins  
    beta-sonarqube-startup          
    beta-sonarqube-tests            

    ==> v1/PersistentVolumeClaim
    beta-postgresql   
    beta-sonarqube
    ```

- Manual update in sonar UI:

    For the following config, it's not configurable during boot time. Need to do it after sonar is up and running.
  1. Go to **${your-sonar-host}** and log in as `admin`.
  2. Go to **${your-sonar-host}/account/security**: change the default admin password
  3. Go to **${your-sonar-host}/admin/permissions**: unselect `Execute Analysis` and `Create Projects` for group `Anyone`

  **Notes**: all these configurations will be persistent as long as the PVC(persistent volume claim) is not deleted.

## Update config
```bash
helm upgrade -f beta-values.yaml beta stable/sonarqube
helm upgrade -f values.yaml prod stable/sonarqube
```

## List helm releases
```bash
helm list
```

## Delete the whole sonar system
Be careful about running this command, it will wipe out everything for sonar even the PVC.
```bash
helm del --purge beta
helm del --purge beta
```

## License

Code licensed under the BSD 3-Clause license. See LICENSE file for terms.

[license-image]: https://img.shields.io/npm/l/screwdriver-api.svg
[issues-image]: https://img.shields.io/github/issues/screwdriver-cd/screwdriver.svg
[issues-url]: https://github.com/screwdriver-cd/screwdriver/issues
[status-image]: https://cd.screwdriver.cd/pipelines/704/badge
[status-url]: https://cd.screwdriver.cd/pipelines/704
[daviddm-image]: https://david-dm.org/screwdriver-cd/screwdriver.svg?theme=shields.io
[daviddm-url]: https://david-dm.org/screwdriver-cd/screwdriver
[slack-image]: http://slack.screwdriver.cd/badge.svg
[slack-url]: http://slack.screwdriver.cd/
