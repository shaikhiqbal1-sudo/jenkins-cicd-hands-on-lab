# Jenkins CI/CD End-to-End Hands-On Lab

[![Jenkins](https://img.shields.io/badge/CI%2FCD-Jenkins-D24939?logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Maven](https://img.shields.io/badge/Build-Apache%20Maven-C71A36?logo=apachemaven&logoColor=white)](https://maven.apache.org/)
[![Tomcat](https://img.shields.io/badge/Server-Apache%20Tomcat-F8DC75?logo=apachetomcat&logoColor=black)](https://tomcat.apache.org/)
[![Java](https://img.shields.io/badge/JDK-OpenJDK%2017-007396?logo=openjdk&logoColor=white)](https://openjdk.org/)

A comprehensive, end-to-end Jenkins CI/CD lab repository documenting installation, administration, RBAC security, Git/GitHub webhook automation, parameterized & scheduled builds, controller-worker distributed architecture, automated Maven `.war` packaging, and Apache Tomcat deployments.

---

## Table of Contents
1. [Jenkins Overview & Installation](#1-jenkins-overview--installation)
2. [Dashboard & System Configuration](#2-dashboard--system-configuration)
3. [UI Customization & User Management (RBAC)](#3-ui-customization--user-management-rbac)
4. [Source Code Management (Git & Workspace Cleanup)](#4-source-code-management-git--workspace-cleanup)
5. [Build Triggers & Automation](#5-build-triggers--automation)
6. [Environment Variables & Parameterized Jobs](#6-environment-variables--parameterized-jobs)
7. [Manual Build & Deployment to Apache Tomcat](#7-manual-build--deployment-to-apache-tomcat)
8. [Automated 4-Stage CI/CD Pipeline (Maven + Tomcat)](#8-automated-4-stage-cicd-pipeline-maven--tomcat)
9. [Distributed Builds (Controller & Worker/Agent Node)](#9-distributed-builds-controller--workeragent-node)
10. [Troubleshooting & Real-World Issue Resolution](#10-troubleshooting--real-world-issue-resolution)

---

## 1. Jenkins Overview & Installation
* **What is Jenkins?** An open-source automation server used to build, test, and deploy software continuously (CI/CD).
* **Local DNS Mapping:** Map the Jenkins server IP to a local hostname on the server/client:
  ```bash
  echo "<JENKINS_SERVER_IP> jenkins_local" | sudo tee -a /etc/hosts

  Key Directory Paths:

Jenkins Home: /var/lib/jenkins (stores system configurations, plugins, logs, and credentials)

Job Workspace (Staging Area): /var/lib/jenkins/workspace/<job-name>

## 2. Dashboard & System Configuration
Dashboard Components:

Views: Organize and group related jobs into custom tabs.

Build Queue & Executor Status: Monitor queued builds and active concurrent executor threads.

Lockable Resources: Prevent build collisions when multiple jobs access shared resources.

Search Panel Shortcuts:

Enter <job-name> to jump directly to a project.

Enter <job-name> <build-number> or <job-name> lastFailedBuild to inspect specific build logs.

Manage Jenkins (Administration):

System: Configure global settings, themes, and the number of concurrent executors.

Tools: Register JDK, Git, and Maven installations.

Plugins: Install, update, and manage Jenkins plugins.

Nodes: Add and manage distributed worker/agent servers.

Security & Credentials: Configure authentication policies and securely store usernames, passwords, SSH keys, and tokens.


## 3. UI Customization & User Management (RBAC)
UI Customization
Install the Simple Theme plugin via Manage Jenkins → Plugins.

Select a Material CSS theme URL (https://cdn.rawgit.com/afonsof/jenkins-material-theme/gh-pages/dist/material-<color>.css).

Apply the CSS URL under Manage Jenkins → System → Theme.

Role-Based Access Control (RBAC)
Install the Role-based Authorization Strategy plugin.

Create user accounts under Manage Jenkins → Users.

Enable Role-Based Strategy under Manage Jenkins → Security.

Configure permissions under Manage Jenkins → Manage and Assign Roles:

Manage Roles: Create a developer role with Overall -> Read and Job -> Build, Read permissions (and Agent permissions when needed).

Assign Roles: Assign the developer role to the created user accounts.

## 4. Source Code Management (Git & Workspace Cleanup)
Shell-Based Clone (Manual):
Running git clone <repo-url> inside a shell step fails on subsequent runs because the destination folder already exists.

Fix: Enable Delete workspace before build starts under Build Environment (using the Workspace Cleanup plugin).

Git Plugin Integration (Recommended):

Select Git under Source Code Management.

Provide the repository URL (https://github.com/Prashanth-MA/Sample_Tomcat_Project.git) and specify the branch (*/main).

Add verification commands (ls -la, cat pom.xml) in the Build Steps to inspect files pulled into the workspace.

## 5. Build Triggers & Automation

Trigger Type	              Configuration & Use Case
Trigger Builds Remotely	    Install Build Authorization Token Root, configure a token on the job, and trigger via browser or Linux CLI:
                            curl "<JENKINS_URL>/buildByToken/build?job=<JOB_NAME>&token=<BUILD_TOKEN>" (escape & with \& in Linux terminals).
                            
Build After Other Projects	Chain upstream/downstream jobs conditionally based on build status (Stable, Unstable, or Failed).

Build Periodically	        Schedule builds on a fixed cron schedule regardless of code changes (e.g., H 2 * * *).
Poll SCM	                  Query Git periodically using a cron schedule and trigger a build only when new commits are detected.
GitHub Webhook	            Enable GitHub hook trigger for GITScm polling in Jenkins, then add <JENKINS_URL>/github-webhook/ (Content-Type: application/json) under GitHub Repo → Settings → Webhooks for instant push-triggered builds.

## 6. Environment Variables & Parameterized Jobs
Job-Level Shell Variables:

name="ajay"
echo "My name is ${name}"

Built-in System Variables: Available at <JENKINS_URL>/env-vars.html (e.g., $BUILD_NUMBER, $WORKSPACE, $JOB_NAME).

Global Environment Variables: Configured under Manage Jenkins → System → Global properties → Environment variables so every job can access them.

Parameterized Builds: Enable This project is parameterized to collect runtime inputs:

String, Boolean, Choice, Password, and Multi-line String parameters.

Accessed inside build scripts via ${PARAM_NAME} (e.g., echo "Selected choice: ${choice}").


## 7. Manual Build & Deployment to Apache Tomcat
Before automating with Jenkins, the build and deployment workflow was validated manually on Ubuntu:

Install Prerequisites & Build the .war Artifact:

sudo apt update && sudo apt install -y git maven
git clone [https://github.com/Prashanth-MA/Sample_Tomcat_Project.git](https://github.com/Prashanth-MA/Sample_Tomcat_Project.git)
cd Sample_Tomcat_Project
mvn test
mvn install   # Generates target/*.war

Install & Configure Apache Tomcat:
sudo apt install -y tomcat9 tomcat9-admin

Port Conflict Note: Both Jenkins and Tomcat default to port 8080. Update <Connector port="8080" ... /> in /etc/tomcat9/server.xml (or /etc/tomcat10/server.xml) to an unused port (e.g., 8090) and restart Tomcat (sudo systemctl restart tomcat9).

CLI Deployment:
sudo cp target/*.war /var/lib/tomcat9/webapps/app.war
Verify in browser: http://<TOMCAT_SERVER_IP>:8090/app

GUI Deployment (Tomcat Manager):

Configure manager-gui and manager-script roles/users in /var/lib/tomcat9/conf/tomcat-users.xml using strong credentials (<TOMCAT_USER> / <TOMCAT_PASSWORD>).

Restart Tomcat and deploy the .war file via /manager/html.


## 8. Automated 4-Stage CI/CD Pipeline (Maven + Tomcat)
Prerequisites
Plugins Installed: Build Pipeline, Maven Integration, Copy Artifact, Deploy to container.

Global Tools Configured: JDK-17 and Maven (mvn) registered under Manage Jenkins → Tools.

Pipeline Job Flow
hello-world-test (Unit Testing):

Pulls code from https://github.com/Prashanth-MA/Sample_Tomcat_Project.git (main).

Executes Maven goal: test.

hello-world-build (Packaging & Archiving):

Triggered automatically after hello-world-test succeeds.

Executes Maven goal: install.

Post-build Action: Archives **/*.war and grants copy permission to downstream deployment jobs.

hello-world-deploy-test (Staging Deployment):

Triggered automatically after hello-world-build succeeds.

Build Step: Uses Copy artifacts from another project (hello-world-build) to copy **/*.war.

Post-build Action: Uses Deploy war/ear to a container to deploy **/*.war to Tomcat9 under context path /app.

hello-world-deploy-prod (Production Deployment):

Configured downstream of hello-world-deploy-test (triggered manually via the Build Pipeline dashboard for continuous delivery approval).


## 9. Distributed Builds (Controller & Worker/Agent Node)
Provision the Worker Server (Ubuntu):

sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-17-jdk curl
sudo mkdir -p /var/lib/jenkins

(Optional) Configure passwordless SSH by adding the controller's public key (~/.ssh/id_rsa.pub) to the worker's ~/.ssh/authorized_keys.

Enable Inbound Agent Port on Controller:

Go to Manage Jenkins → Security → Agents and set TCP port for inbound agents to Random (or a permitted port in your firewall/Security Group).

Register the Agent Node in Jenkins:

Navigate to Manage Jenkins → Nodes → New Node (Permanent Agent).

Set Remote root directory to /var/lib/jenkins and Labels to slave.

Connect the Agent from the Worker Terminal:

cd /var/lib/jenkins
curl -sO http://<JENKINS_SERVER_IP>:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://<JENKINS_SERVER_IP>:8080/ -secret <AGENT_SECRET> -name "<NODE_NAME>" -workDir "/var/lib/jenkins"

Target Jobs to the Worker Node:

In a job's configuration, check Restrict where this project can be run and set Label Expression to slave.

Disk Space Tip (/tmp full error during builds):

sudo mount -o remount,size=2G /tmp

## 10. Troubleshooting & Real-World Issue Resolution
Issue 1: UnsupportedOperationException in Maven35Launcher (0 ms build duration)
Error Log:

ERROR: Failed to parse POMs
Caused by: java.lang.UnsupportedOperationException
    at org.eclipse.sisu.wire.EntryListAdapter$ValueListIterator.add(EntryListAdapter.java:185)
    at org.jvnet.hudson.maven3.launcher.Maven35Launcher.main(Maven35Launcher.java:127)

Root Cause: The legacy Jenkins Maven Project plugin (Maven35Launcher) attempts to modify an unmodifiable Eclipse Sisu dependency list introduced in Maven 3.9+.

Resolution: Switched to a Freestyle / Pipeline job invoking top-level Maven targets directly (or configured Maven 3.8.8 under Manage Jenkins → Tools).

Issue 2: Fatal error compiling: error: release version 17 not supported
Error Log:
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.10.1:compile (default-compile) on project helloworld:
Fatal error compiling: error: release version 17 not supported

Root Cause: The project's pom.xml targets Java 17 (<maven.compiler.release>17</maven.compiler.release>), while Jenkins was not pointed to the OpenJDK 17 installation on the Ubuntu host.

Resolution:

Verified installed JDKs on Ubuntu via update-java-alternatives --list, locating OpenJDK 17 at /usr/lib/jvm/java-1.17.0-openjdk-amd64.

Registered JDK-17 with JAVA_HOME=/usr/lib/jvm/java-1.17.0-openjdk-amd64 under Manage Jenkins → Tools.

Selected JDK-17 in the job's JDK dropdown, resolving the compilation error.

