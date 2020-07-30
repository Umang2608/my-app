
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
        env.DOCKER_REGISTRY= property.DOCKER_REGISTRY
        env.DOCKER_REPO= property.DOCKER_REPO

}

def FAILED_STAGE
podTemplate(cloud:'openshift' , label: 'docker',
    containers: [
        containerTemplate(
            name: 'jnlp',
            image: 'shubhamasati/donotcopy:jenkinsslave',
            alwaysPullImage: true,
            
            envVars: [envVar(key:'http_proxy',value:''),envVar(key:'https_proxy',value:'')],
            args: '${computer.jnlpmac} ${computer.name}',
           ttyEnabled: true
         )],volumes: [hostPathVolume(hostPath:'/var/run/docker.sock', mountPath:'/var/run/docker.sock'),hostPathVolume(hostPath:'/etc/docker/daemon.json', mountPath:'/etc/docker/daemon.json')])
{
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
                echo 'mvn sonar:sonar -Dsonar.host.url=${SONAR_HOST_URL}'
        }
    }
    if (env.CODE_COVERAGE == 'True')
    {
        stage('Coverage testing')
        {
            sh 'mvn cobertura:cobertura'
        }
    }
    
    stage ('Build and tag image for Dev')
    {
        sh 'mvn war:war'
       
    }
    stage ('Run Jmeter test')
    {
        sh 'mvn jmeter:configure jmeter:gui'
            
    }
          
         
        
                
}

}
