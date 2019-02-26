library identifier: "pipeline-library@master",
retriever: modernSCM(
  [
    $class: "GitSCMSource",
    remote: "https://github.com/dwasinge/pipeline-library.git"
  ]
)

openshift.withCluster() {

  env.NAMESPACE = openshift.project()
  env.POM_FILE = env.BUILD_CONTEXT_DIR ? "${env.BUILD_CONTEXT_DIR}/pom.xml" : "pom.xml"
  env.APP_NAME = "${env.JOB_NAME}".replaceAll(/-?${env.PROJECT_NAME}-?/, '').replaceAll(/-?pipeline-?/, '').replaceAll('/','')
  env.BUILD = "${env.NAMESPACE}"
  env.DEV = env.BUILD.replace('ci-cd', 'dev')
  env.TEST = env.BUILD.replace('ci-cd', 'test')
  env.MVN_SNAPSHOT_DEPLOYMENT_REPOSITORY = "nexus::default::http://nexus:8081/repository/labs-snapshots"
  env.OCP_API_SERVER = "${env.OPENSHIFT_API_URL}"
  env.OCP_TOKEN = readFile('/var/run/secrets/kubernetes.io/serviceaccount/token').trim()

  echo "Starting Pipeline for ${APP_NAME}..."

}

pipeline {
  // Use Jenkins Maven slave
  // Jenkins will dynamically provision this as OpenShift Pod
  // All the stages and steps of this Pipeline will be executed on this Pod
  // After Pipeline completes the Pod is killed so every run will have clean
  // workspace
  agent {
    label 'maven'
  }

  // Pipeline Stages start here
  // Requeres at least one stage
  stages {

    // Checkout source code
    // This is required as Pipeline code is originally checkedout to
    // Jenkins Master but this will also pull this same code to this slave
    stage('Git Checkout') {
      steps {
        // Turn off Git's SSL cert check, uncomment if needed
        // sh 'git config --global http.sslVerify false'
        // git url: "${APPLICATION_SOURCE_REPO}"
        sh "echo 'already checked code out' "
      }
    }

    // Run Maven build, skipping tests
    stage('Build'){
      steps {
        sh "mvn -B clean install -DskipTests=true -f ${POM_FILE}"
      }
    }

    // Run Maven unit tests
    stage('Unit Test'){
      steps {
        sh "mvn -B test -f ${POM_FILE}"
      }
    }

    stage ('Code Analysis') {
        steps {
          sonarqubeStaticAnalysis(
            buildServerWebHookName: "jenkins",
            buildServerWebHookUrl: "${JENKINS_URL}sonarqube-webhook/",
            dependencyCheckReportDir: "target",
            dependencyCheckReportFiles: "dependency-check-report.html",
            dependencyCheckReportName: "OWASP Dependency Check Report",
            dependencyCheckKeepAll: true,
            dependencyCheckAlwaysLinkToLastBuild: true,
            dependencyCheckAllowMissing: true,
            unitTestReportDir: "target/site/jacoco/",
            unitTestReportFiles: "index.html",
            unitTestReportName: "Jacoco Unit Test Report",
            unitTestKeepAll: true,
            unitTestAlwaysLinkToLastBuild: false,
            unitTestAllowMissing: true)
        }
    }

    stage ('Deploy to Nexus') {
      steps {
        script {
            openshift.withCluster() {
              openshift.withProject( "${DEV_PROJECT}" ){
              def latestDeploymentVersion = openshift.selector('dc',"nexus").object().status.latestVersion
              def rc = openshift.selector('rc', "nexus-${latestDeploymentVersion}")
              rc.untilEach(1){
                  def rcMap = it.object()
                  return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
              }
            }
          }
        }
        sh "mvn -B clean deploy -DskipTests=true -DaltDeploymentRepository=${MVN_SNAPSHOT_DEPLOYMENT_REPOSITORY} -f ${POM_FILE}"
      }
    }

    // Build Container Image using the artifacts produced in previous stages
    stage('Build Container Image'){
      steps {
        // Copy the resulting artifacts into common directory
        sh """
          ls target/*
          rm -rf oc-build && mkdir -p oc-build/deployments
          for t in \$(echo "jar;war;ear" | tr ";" "\\n"); do
            cp -rfv ./target/*.\$t oc-build/deployments/ 2> /dev/null || echo "No \$t files"
          done
        """

        // Build container image using local Openshift cluster
        // Giving all the artifacts to OpenShift Binary Build
        // This places your artifacts into right location inside your S2I image
        // if the S2I image supports it.
        binaryBuild(projectName: env.BUILD, buildConfigName: env.APP_NAME, artifactsDirectoryName: "oc-build")
      }
    }

    stage('Promote from Build to Dev') {
      steps {
        tagImage(sourceImageName: env.APP_NAME, sourceImagePath: env.BUILD, toImagePath: env.DEV)
      }
    }

    stage ('Verify Deployment to Dev') {
      steps {
        verifyDeployment(projectName: env.DEV, targetApp: env.APP_NAME)
      }
    }

    stage('Promotion gate') {
      steps {
        script {
          input message: 'Promote application to Test?'
        }
      }
    }

    stage('Promote from Dev to Test') {
      steps {
        tagImage(sourceImageName: env.APP_NAME, sourceImagePath: env.DEV, toImagePath: env.TEST)
      }
    }

    stage ('Verify Deployment to Test') {
      steps {
        verifyDeployment(projectName: env.TEST, targetApp: env.APP_NAME)
      }
    }

  }
}