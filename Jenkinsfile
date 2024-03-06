node {
    // reference to maven
    // ** NOTE: This 'maven-3.6.1' Maven tool must be configured in the Jenkins Global Configuration.   
    def mvnHome = tool 'maven-3.8.5'

    // holds reference to docker image
    def dockerImage
    // ip address of the docker private repository(nexus)
    
    def dockerRepoUrl = "localhost:8083"
    def dockerImageName = "hello-world-java"
    def dockerImageTag = "${dockerRepoUrl}/${dockerImageName}:${BRANCH_NAME}"
    
    stage('Clone Repo') { // for display purposes
      // Get some code from a GitHub repository
      git branch: 'jenkins', url: 'https://github.com/goutamjaiswal214/devops-end-to-end.git'
      // Get the Maven tool.
      // ** NOTE: This 'maven-3.6.1' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool 'maven-3.8.5'
    }    
  
    stage('Build Project') {
      // build project via maven
      sh "'${mvnHome}/bin/mvn' -f hello-world-src/pom.xml -Dmaven.test.failure.ignore clean package"
    }
	
	// stage('Publish Tests Results'){
    //   parallel(
    //     publishJunitTestsResultsToJenkins: {
    //       echo "Publish junit Tests Results"
	// 	  junit '**/target/surefire-reports/TEST-*.xml'
	// 	  archive 'target/*.jar'
    //     },
    //     publishJunitTestsResultsToSonar: {
    //       echo "This is branch b"
    //   })
    // }
		
    stage('Build Docker Image') {
      // build docker image
      //sh "ls -all /var/run/docker.sock"
      sh "echo $BRANCH_NAME"
      sh "mv ./hello-world-src/target/hello*.jar ./data" 
      
      dockerImage = docker.build("hello-world-java","-f hello-world-src/Dockerfile .")

      sh "docker tag hello-world-java:latest us-east1-docker.pkg.dev/molten-medley-415817/hello-world/${dockerImageName}:${BRANCH_NAME}"
      sh "docker images -a"
    }
   
    stage('Pushing Docker Image'){
      
      // deploy docker image to nexus
      withCredentials([file(credentialsId: 'gcr-file', variable: 'GC_KEY')]){
        sh "echo \"Docker Image Tag Name: ${dockerImageTag}\""
        sh "cat '$GC_KEY' | docker login -u _json_key --password-stdin https://us-east1-docker.pkg.dev"
        sh "gcloud auth activate-service-account --key-file='$GC_KEY'"
        sh "gcloud auth configure-docker us-east1-docker.pkg.dev"
        GLOUD_AUTH = sh (
              script: 'gcloud auth print-access-token',
              returnStdout: true
          ).trim()
        echo "Pushing image To GCR"
        sh "docker push us-east1-docker.pkg.dev/molten-medley-415817/hello-world/${dockerImageName}:${BRANCH_NAME}"
      }
    }

    stage('Deploying App to GKE'){
      // deploy docker image to nexus
      withCredentials([file(credentialsId: 'gcr-file', variable: 'GC_KEY')]){
        sh "gcloud auth activate-service-account --key-file=$GC_KEY"
        sh "gcloud config set project molten-medley-415817"
        sh "gcloud container clusters get-credentials molten-medley-415817-gke --region us-west4 --project molten-medley-415817"
        sh "helm upgrade --install my-java-app helm-chart/java-app --namespace java-app"
        sh '''
        export SERVICE_IP=$(kubectl get svc --namespace java-app my-java-app --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
        echo "Your Application is live on: echo http://$SERVICE_IP:8080"
        '''
      }
    }
}