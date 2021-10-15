def label = "jenkins-slave-${UUID.randomUUID().toString()}"

podTemplate(label: label, serviceAccount: 'jenkins', containers: [
  containerTemplate(name: 'jnlp', image: 'cnych/jenkins:jnlp6', command: '', ttyEnabled: true)
], volumes: [
  hostPathVolume(mountPath: '/home/jenkins/.kube', hostPath: '/root/.kube'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH

    stage('从gitlab拉取服务代码') {
                echo '从gitlab拉取服务代码'
                checkout scm
                script {
                    commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            }
        }
        stage('构建镜像') {
      
                echo '构建镜像'
                sh "docker build -t docker.zdns.cn/devops/user:${commit_id} app/user/api/"
       
        }
        stage('上传镜像到镜像仓库') {
           
                echo "上传镜像到镜像仓库"
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
                    sh "docker login -u ${dockerUser} -p ${dockerPassword} docker.zdns.cn"
                    sh "docker push docker.zdns.cn/devops/user:${commit_id}"
        }
        
        stage('选择上线环境') {
                echo "选择上线环境"
                script {
                    env.DEPLOY_ENV = input message: '选择部署的环境', ok: 'deploy',
                        parameters: [choice(name: 'DEPLOY_ENV', choices: ['prd', 'uat', 'test'], description: '选择部署环境')]

                        switch("${env.DEPLOY_ENV}"){
                            case 'prd':
                                echo 'deploy prd env'
                                break;

                            case 'uat':
                                echo 'deploy uat env'
                                break;

                            case 'test':
                                echo 'deploy test env'
                                break;
                            
                            default:
                                echo 'error env'
                        }
                    }
        }
        
        }
        stage('部署到k8s') {
            
                echo "部署到k8s"
                if (env.DEPLOY_ENV == "test") {
                    // deploy dev stuff
                    echo "部署到test-k8s"
                } else if (env.DEPLOY_ENV == "uat"){
                    // deploy qa stuff
                    echo "部署到uat-k8s"
                } else {
                    // deploy prod stuff
                    echo "部署到prd-k8s"
                }
                sh "sed -i 's/<COMMIT_ID_TAG>/${commit_id}/' app/user/api/user.yaml"
                sh "kubectl apply -f app/user/api/user.yaml"
            
        }
  }
}
