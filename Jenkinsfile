
def readProperties()
{
        def properties_file_path = "${workspace}" + "@script/properties.yml"
        def property = readYaml file: properties_file_path
        env.APP_NAME = property.APP_NAME
        env.MS_NAME = property.MS_NAME
        env.BRANCH = property.BRANCH
        env.GIT_SOURCE_URL = property.GIT_SOURCE_URL
        env.SONAR_HOST_URL = property.SONAR_HOST_URL
        env.CODE_QUALITY = property.CODE_QUALITY
        env.UNIT_TESTING = property.UNIT_TESTING
        env.CODE_COVERAGE = property.CODE_COVERAGE

}

def FAILED_STAGE
node
{
    def MAVEN_HOME = tool "MY_MAVEN"
    def JAVA_HOME = tool "MY_JDK"
    env.PATH= "${env.PATH}:${MAVEN_HOME}/bin:${JAVA_HOME}/bin"
    
    stage('Checkout')
    {
        readProperties()
        checkout([$class: 'GitSCM', branches: [[name: "*/${BRANCH}"]], doGenerateSubmoduleConfigurations: false, extensions:[], submoduleCfg: [], userRemoteConfigs: [[url: "${GIT_SOURCE_URL}"]]])
    }
    stage('Initial setup')
    {
        sh 'mvn clean'
            sh 'mvn compile'
    }
    if (env.UNIT_TESTING == 'True')
    {
        stage('Unit testing')
        {
            sh 'mvn test'
        }
    }
    if (env.CODE_QUALITY == 'True')
    {
        stage('Code Quality')
        {
            echo 'mvn sonar:sonar'
        }
    }
    if (env.CODE_COVERAGE == 'True')
    {
        stage('Coverage testing')
        {
            sh 'mvn cobertura:cobertura'
        }
    }
    if (env.SECURITY_TESTING == 'True')
    {
        stage('Security testing')
        {
            echo 'security'
        }
    }
    stage ('Build and tag image for Dev')
    {
        sh 'mvn war:war'
        node ('docker')
    {
        container('jnlp')
        {
            checkout([$class: 'GitSCM', branches: [[name: "*/${BRANCH}"]], doGenerateSubmoduleConfigurations: false, extensions:[], submoduleCfg: [], userRemoteConfigs: [[url: "${GIT_SOURCE_URL}"]]])

        sh "docker login ${DOCKER_REGISTRY} -u $umang2608 -p $Umang@2608"
        sh "docker build -t ${MS_NAME}:latest ."
        sh 'docker tag ${MS_NAME}:latest ${DOCKER_REGISTRY}/$DOCKER_REPO}/$MS_NAME:$APP_NAME'
                sh 'docker push ${DOCKER_REGISTRY}/$DOCKER_REPO}/$MS_NAME:$APP_NAME'
                sh 'docker rmi -f ${DOCKER_REGISTRY}/$DOCKER_REPO}/$MS_NAME:$APP_NAME'
                sh 'docker rmi -f ${MS_NAME}:latest'
                
        }    
    
        }
    }
}

