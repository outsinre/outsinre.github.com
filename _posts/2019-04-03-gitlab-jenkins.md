---
layout: post
title: Gitlab Jenkins
---

1. toc
{:toc}

# ABCs

1. Jenkins is a [Continuous Integration (CI) ](https://en.wikipedia.org/wiki/Continuous_integration) utility. Among others, which is mostly used to _automate_ building, testing, delivering or deploying packages upon sources pushed to the mainlin repository.
2. Gitlab requires [Webhooks](https://docs.gitlab.com/ee/user/project/integrations/webhooks.html).
3. Jenkins requires [gitlab-plugin](https://github.com/jenkinsci/gitlab-plugin) or [Gitlab Webhook Plugin](https://wiki.jenkins.io/display/JENKINS/Gitlab+Hook+Plugin).
4. Generally speaking, the Webhook defined in Gitlag repository will call the Gitlab Webhook Plugin associated with Jenkins server which in return performs the build.

Read [Continuous Integration with Jenkins and GitLab](https://medium.com/@teeks99/continuous-integration-with-jenkins-and-gitlab-fa770c62e88a).

# [Installation](https://wiki.jenkins.io/display/JENKINS/Installing+Jenkins+on+Red+Hat+distributions)

1. Choose the [Long-term Support (LTS)](https://jenkins.io/changelog-stable/).
2. Java is located under '/home/shibin.luo/collectflume/jdk1.8.0_112/bin/java'.
3. Yum

   ```bash
   ~ # sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
   ~ # sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
   ~ # sudo yum install jenkins
   ```

https://medium.com/@teeks99/continuous-integration-with-jenkins-and-gitlab-fa770c62e88a



# QA

1. ccssh5 [BGP-GZ-b-3gb](http://59.42.253.31:8080)
2. Build status [feedback](https://github.com/jenkinsci/gitlab-plugin)
3. Jenkins master/slave.
4. Java