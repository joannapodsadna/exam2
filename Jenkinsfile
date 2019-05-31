node {
  def acr = 'acrdemo5.azurecr.io'
  def appName = 'whoami'
  def imageName = "${acr}/${appName}"
  def imageTag = "${imageName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
  def appRepo = "acrdemo5.azurecr.io/whoami:v.0.1.0"

  checkout scm
  
 stage('Build the Image and Push to Azure Container Registry') 
 {
   app = docker.build("${imageName}")
   withDockerRegistry([credentialsId: 'kama-kama', url: "https://${acr}"]) {
      app.push("${env.BRANCH_NAME}.${env.BUILD_NUMBER}")
                }
  }

 stage ("Deploy Application on Azure Kubernetes Service")
 {
  switch (env.BRANCH_NAME) {

    case "master":
        // Change deployed image in master to the one we just built
        sh("sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config get ns production || sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config create ns production")
        withCredentials([usernamePassword(credentialsId: 'kama-kama', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
           sh "sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config -n production get secret kama-kama || sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace=production create secret docker-registry kama-kama --docker-server ${acr} --docker-username $USERNAME --docker-password $PASSWORD"
        } 
        sh("sed -i.bak 's#${appRepo}#${imageTag}#' ./k8s/production/*.yaml")
        sh("sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace=production apply -f k8s/production/")
        sh("echo http://`sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace= production get service/${appName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${appName}")
        break

    case "canary":
        // Change deployed image in canary to the one we just built
        sh("sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config get ns production || sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config create ns production")
        withCredentials([usernamePassword(credentialsId: 'kama-kama', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh " sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config -n production get secret kama-kama || kubectl --namespace=production create secret docker-registry kama-kama --docker-server ${acr} --docker-username $USERNAME --docker-password $PASSWORD"
        }
        sh("sed -i.bak 's#${appRepo}#${imageTag}#' ./k8s/canary/*.yaml")
        sh("sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace=production apply -f k8s/canary/")
        sh("echo http://`sudo kubectl -kubeconfig ~jenkinsdemo5/.kube/config --namespace= production get service/${appName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${appName}")
        break

  case "release":
        // Change deployed image in canary to the one we just built
        sh("sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config get ns stage || sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config create ns stage")
        withCredentials([usernamePassword(credentialsId: 'kama-kama', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
<<<<<<< HEAD
          sh "sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config -n stage get secret kama-kama ||  sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace=stage create secret docker-registry kama-kama --docker-server ${acr} --docker-username $USERNAME --docker-password $PASSWORD"
=======
          sh "sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config -n stage get secret kama-kama || sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace=stage create secret docker-registry kama-kama --docker-server ${acr} --docker-username $USERNAME --docker-password $PASSWORD"
>>>>>>> 9e92d6ceb837d47c6ec6005b8f85c5da9ac91001
        }
        sh("sed -i.bak 's#${appRepo}#${imageTag}#' ./k8s/release/*.yaml")
        sh("sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace=stage apply -f k8s/release/")
        sh("echo http://`sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace=stage get service/${appName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${appName}")
        break

    // Roll out a dev environment 
    case "dev":
        // Create namespace if it doesn't exist
        sh("sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config get ns dev || sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config create ns dev")
        withCredentials([usernamePassword(credentialsId: 'kama-kama', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh "sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config -n dev get secret kama-kama3 || sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace=dev create secret docker-registry kama-kama3 --docker-server ${acr} --docker-username $USERNAME --docker-password $PASSWORD"
        }
        // Don't use public load balancing for development branches
        sh("sed -i.bak 's#${appRepo}#${imageTag}#' ./k8s/dev/*.yaml")
        sh("sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace=dev apply -f k8s/dev/")
        sh("echo http://`sudo kubectl --kubeconfig ~jenkinsdemo5/.kube/config --namespace=dev get service/${appName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${appName}")

    default:
        // Create namespace if it doesn't exist 
        sh("kubectl get ns ${appName}-${env.BRANCH_NAME} || kubectl create ns ${appName}-${env.BRANCH_NAME}")
        withCredentials([usernamePassword(credentialsId: 'kama-kama', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh "kubectl -n ${appName}-${env.BRANCH_NAME} get secret kama-kama4 || kubectl --namespace=${appName}-${env.BRANCH_NAME} create secret docker-registry kama-kama4 --docker-server ${acr} --docker-username $USERNAME --docker-password $PASSWORD"
        } 
        sh("sed -i.bak 's#${appRepo}#${imageTag}#' ./k8s/${env.BRANCH_NAME}/*.yaml")
        sh("kubectl --namespace=${appName}-${env.BRANCH_NAME} apply -f k8s/${env.BRANCH_NAME}/")
        echo 'To access your environment run `kubectl proxy`'
        echo "Then access your service via http://localhost:8001/api/v1/namespaces/${appName}-${env.BRANCH_NAME}/services/${appName}:80/proxy/"           
    }
  }
}
