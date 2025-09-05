def branchName     = params.BranchName ?: "main"
def gitUrl         = "git@github.com:ghadeerhamdan-cmd/bashtask.git"
def gitUrlCode     = "git@github.com:ghadeerhamdan-cmd/bashtask.git"
def serviceName    = "ghadeerecr"
def EnvName        = "preprod"
def registryId     = "727245885999"
def awsRegion      = "ap-south-1"
def ecrUrl         = "727245885999.dkr.ecr.ap-south-1.amazonaws.com"
def dockerfile     = "Dockerfile"
def imageTag       = "${EnvName}-${BUILD_NUMBER}"


// AppConfig Params
def applicationName = "webapp"
def envName = "preprod"
def configName = "preprod"
// Fix: Use string concatenation, not arithmetic
def clientId = "${applicationName}-${envName}"
def latestTagValue = params.Tag
def namespace = "preprod"
def helmDir = "helm"  
def slashtecDir = "." 

node {
   
    try {
      
      stage('cleanup') {
        cleanWs()
      }
      stage ("Get the app code") {
        checkout([$class: 'GitSCM', branches: [[name: "${branchName}"]] , extensions: [], userRemoteConfigs: [[ url: "${gitUrlCode}"]]])
        sh "rm -rf ~/workspace/\"${JOB_NAME}\"/slashtec"
        sh "mkdir ~/workspace/\"${JOB_NAME}\"/slashtec  ; cd slashtec ; git clone -b main ${gitUrl} "
        sh("cp ${slashtecDir}/docker/Dockerfile ${dockerfile}")
        sh("cp -r  ${slashtecDir}/docker/* .")
        // sh("cp -r  ${slashtecDir}/files/* .") 
      }
      // stage("Get the env variables from App") {
      //   sh "aws appconfig get-configuration --application ${applicationName} --environment ${envName} --configuration ${configName} --client-id ${clientId} .env --region ${awsRegion}"
      // }
      stage('login to ecr') {
        sh("aws ecr get-login-password --region ${awsRegion}  | docker login --username AWS --password-stdin ${ecrUrl}")
      }
      stage('Build Docker Image') {
        sh("docker build -t ${ecrUrl}/${serviceName}:${imageTag} -f ${dockerfile} .")
      }
      stage('Push Docker Image To ECR') {
        sh("docker push ${ecrUrl}/${serviceName}:${imageTag}")
      }
      stage('Clean docker images') {
        sh("docker rmi -f ${ecrUrl}/${serviceName}:${imageTag} || :")
      }
      stage ("Deploy ${serviceName} to ${EnvName} Environment") {
        sh ("cd ${helmDir}; pathEnv=\".deployment.image.tag\" valueEnv=\"${imageTag}\" yq 'eval(strenv(pathEnv)) = strenv(valueEnv)' -i values.yaml ; cat values.yaml")
        sh ("cd ${helmDir}; git checkout ${branchName}; git pull origin ${branchName} ; git add values.yaml; git commit -m 'update image tag' ;git push origin ${branchName}")      }

      // stage ("Deploy preprod-solo-queue to ${EnvName} Environment") {
      //   build job: 'preprod-solo-queue', wait: true
      // }
      // stage ("Deploy preprod-solo-crons to ${EnvName} Environment") {
      //   build job: 'preprod-solo-crons', wait: true
      // }

    //   // The ArgoCD stages remain commented, as in your code

    } catch (org.jenkinsci.plugins.workflow.steps.FlowInterruptedException e) {
      currentBuild.result = "ABORTED"
      echo "Build aborted: ${e}"
    } catch (Exception e) {
      currentBuild.result = "FAILED"
      echo "Exception type: ${e.getClass().getName()}"
      echo "Exception message: ${e.message ?: 'No message'}"
      throw e
    } 
  }
