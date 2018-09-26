import groovy.json.JsonSlurperClassic

node {
  try {
        def BRANCH_NAME = "master"
        def IMAGE_VERSION = "$BUILD_NUMBER"
        def REPO = "https://github.com/richardsonlima/alpine-nginx-php-docker.git"
        def REGISTRY_REPO = "richardsonlima/alpine-nginx-php"
          
        //message = "Pipeline started $BRANCH_NAME - Build $BUILD_NUMBER"
        //notifyBuild(message)

        stage("cloning_$PROJECT") {
                checkout([$class: 'GitSCM',
                    userRemoteConfigs: [[url: "$REPO_GIT"]],
                    branches: [[name: "$BRANCH_NAME"]],
                    //credentialsId: 'xxxxxx',
                    clean: false,
                    extensions: [[$class: 'SubmoduleOption',
                                    disableSubmodules: false,
                                    parentCredentials: false,
                                    recursiveSubmodules: true,
                                    reference: '',
                                    trackingSubmodules: false]],
                    doGenerateSubmoduleConfigurations: false,
                    submoduleCfg: []
                ])
        }

        stage('container_up') {
            dir(env.WORKSPACE) {
                sh(script: "docker-compose up --build -d")
            }
        }

        stage('container_stop') {
            dir(env.WORKSPACE) {
                sh """
                    for services in `docker-compose config --services`; do
                        docker-compose stop -t1 \$services
                    done
                """
            }
        } 

        stage('push_container_to_registry') {
            //def LOGIN_CMD = sh(script: "aws ecr get-login --profile xxx --no-include-email --region sa-east-1", returnStdout: true)
            dir(env.WORKSPACE) {
                sh """
                    #${LOGIN_CMD}
                    docker build -t $REGISTRY_REPO:$IMAGE_VERSION -f Dockerfile .
                    docker push $REGISTRY_REPO:$IMAGE_VERSION
                """
            }
        }

        stage('Docker - Clean Images') {
        def dockerImageId = sh(
            script: "docker images | awk '{print \$3}' | sed '2!d'",
            returnStdout: true
        )

        if (dockerImageId != "") {
            sh("docker rmi -f ${dockerImageId}")
        }
        }

        } 
  
        /* catch (error) {
            currentBuild.result = "FAILED"
            throw error
        } finally {
            notifyBuild("", currentBuild.result)
        }*/
}

  
