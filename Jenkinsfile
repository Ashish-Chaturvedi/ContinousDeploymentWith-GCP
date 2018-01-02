node {
  def project = 'PROJECT-ID'
  def appName = 'microservice'
  def feSvcName = "${appName}-svc"
  def imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"

  checkout scm

  stage 'Build image'
  sh("docker build -t ${imageTag} .")

  stage 'Push image to registry'
  sh("gcloud docker -- push ${imageTag}")

  stage "Deploy Application"
  switch (env.BRANCH_NAME) {
    case "canary":
        // Change deployed image in canary to the one we just built
        sh("sed -i.bak 's#gcr.io/PROJECT_ID/microservice:v1.1#${imageTag}#' backend-canary.yaml")
        sh("kubectl --namespace=production apply -f backend.yaml")
        sh("kubectl --namespace=production apply -f backend-canary.yaml")
        sh("echo http://`kubectl --namespace=production get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
        break

    case "master":
        // Change deployed image in canary to the one we just built
        sh("sed -i.bak 's#gcr.io/PROJECT_ID/microservice:v1.1#${imageTag}#' backend-production.yaml")
        sh("kubectl --namespace=production apply -f backend.yaml")
        sh("kubectl --namespace=production apply -f backend-production.yaml")
        sh("echo http://`kubectl --namespace=production get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
        break
  }
}
