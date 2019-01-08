Gradle Slave Image
====================

This repository contains Dockerfiles for a Jenkins Slave Docker image intended for 
use with [OpenShift v3](https://github.com/openshift/origin) for Gradle builds


# Build Slave Image 
To build the Jenkins Slave with Gradle based on the CentOS 7 image you need to run:
```
curl https://raw.githubusercontent.com/siamaksade/jenkins-slave-gradle/master/Dockerfile | oc new-build --name=jenkins-slave-gradle  --dockerfile -
```
For RHEL:
```
curl https://raw.githubusercontent.com/siamaksade/jenkins-slave-gradle/master/Dockerfile.rhel7 | oc new-build --name=jenkins-slave-gradle  --dockerfile -
```
Then add a tag to make the image stream available in the openshift namespace/project:
```
oc tag jenkins-slave-gradle:latest openshift/jenkins-slave-gradle:latest
```
# Add ConfigMap
To use the Jenkins slave a ConfigMap with the role set to "jenkins-slave" needs to be created:
```
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    role: jenkins-slave
  name: jenkins-slaves
data:
  gradle-template: |-
    <org.csanchez.jenkins.plugins.kubernetes.PodTemplate>
      <name>gradle</name>
      <namespace></namespace>
      <privileged>false</privileged>
      <capOnlyOnAlivePods>false</capOnlyOnAlivePods>
      <alwaysPullImage>false</alwaysPullImage>
      <instanceCap>2147483647</instanceCap>
      <slaveConnectTimeout>100</slaveConnectTimeout>
      <idleMinutes>0</idleMinutes>
      <activeDeadlineSeconds>0</activeDeadlineSeconds>
      <label>gradle</label>
      <nodeUsageMode>NORMAL</nodeUsageMode>
      <customWorkspaceVolumeEnabled>false</customWorkspaceVolumeEnabled>
      <workspaceVolume class="org.csanchez.jenkins.plugins.kubernetes.volumes.workspace.EmptyDirWorkspaceVolume">
      <memory>false</memory>
      </workspaceVolume>
      <containers>
      <org.csanchez.jenkins.plugins.kubernetes.ContainerTemplate>
        <name>jnlp</name>
        <image>docker-registry.default.svc:5000/openshift/jenkins-slave-gradle:latest</image>
        <privileged>false</privileged>
        <alwaysPullImage>false</alwaysPullImage>
        <workingDir>/tmp</workingDir>
        <command></command>
        <args>${computer.jnlpmac} ${computer.name}</args>
        <ttyEnabled>false</ttyEnabled>
        <resourceRequestCpu>200m</resourceRequestCpu>
        <resourceRequestMemory>512Mi</resourceRequestMemory>
        <resourceLimitCpu>2</resourceLimitCpu>
        <resourceLimitMemory>4Gi</resourceLimitMemory>
        <timeoutSeconds>0</timeoutSeconds>
        <initialDelaySeconds>0</initialDelaySeconds>
        <failureThreshold>0</failureThreshold>
        <periodSeconds>0</periodSeconds>
        <successThreshold>0</successThreshold>
        </livenessProbe>
      </org.csanchez.jenkins.plugins.kubernetes.ContainerTemplate>
      </containers>
      <podRetention class="org.csanchez.jenkins.plugins.kubernetes.pod.retention.Default"/>
    </org.csanchez.jenkins.plugins.kubernetes.PodTemplate>
```
