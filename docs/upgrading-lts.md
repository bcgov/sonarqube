# Upgrading LTS versions of SonarQube

If you're running a version of SonarQube several versions behind the current LTS you'll need to systematically upgrade to each LTS version and trigger the DB upgrade.

Example to upgrade from 8.2.2 to 9.9.1-community you first need to upgrade the an intermediate LTS version 8.9.10-community
The DB upgrade is straight forward. Once 8.9.10-community has been updated in your dockerfile and deployed (remember turn off the pod for your existing sonarqube as only one instance can run at a time) you'll need to navigate to {yourURL}/upgrade and press the button to upgrade the database.

You'll then do it again with 9.9.1 to upgrade the docker file and the db with the /upgrade url.

Now, we did have one catch with this upgrade. The base OS on 8.9.10-community is Alpine where the other versions of LTS are Ubuntu. This means the apt-get commands in the docker file will fail and can be replaced with apk commands if you choose. Since it was just an intermediate upgrade we opted not to bother with the apk upgrades.


```oc cp ./my-sonar-plugin.jar my-openshift-namespace/sonarqube-pod-name:/opt/sonarqube/extensions/plugins```

A restart of the pod will be required after copying a new/updated plugin.

# References
- [SonarQube: Installing Plugin](https://docs.sonarqube.org/latest/setup/install-plugin/)
