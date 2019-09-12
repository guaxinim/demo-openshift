node('maven') {
	def app = 'demo-openshift'
	def cliente = 'brasilcap'
	def gitUrl = 'https://github.com/guaxinim/demo-openshift.git'

    stage 'Build image and deploy to dev'
    echo 'Building docker image'
    buildApp(app + '-dev', gitUrl, app)

    stage 'Deploy to QA'
    echo 'Deploying to QA'
    deploy(app + '-dev', app + '-qa', app)

    stage 'Wait for approval'
    input 'Aprove to production?'

    stage 'Deploy to Prod'
    deploy(app + '-dev', app + '-prd', app)
}

def buildApp(String project, String gitUrl, String app){
    projectSet(project)

    sh "oc new-build php:7.0~${gitUrl} --name=${app} -n ${project}"
    sh "oc logs -f bc/${app} -n ${project}"
    sh "oc new-app ${app} -n ${project}"
    sh "oc expose service ${app} -n ${project} || echo 'Service already exposed'"
    sh "sleep 10"
}

def projectSet(String project) {
	sh "oc login --insecure-skip-tls-verify=true https://master.brasilia-34e9.open.redhat.com:443 --token=84n3Ng99ijCQMa8V5s4IKrUZWgiJclmOOJvMd0zLG9k"
    sh "oc new-project ${project} || echo 'Project exists'"
    sh "oc project ${project}"
}

def deploy(String origProject, String project, String app){
    projectSet(project)
    sh "oc policy add-role-to-user system:image-puller system:serviceaccount:${project}:default -n ${origProject}"
    sh "oc tag ${origProject}/${app}:latest ${project}/${app}:latest"
    appDeploy(app, project)
}

def appDeploy(String app, String project){
    sh "oc new-app ${app} -l app=${app} -n ${project} || echo 'Aplication already Exists'"
    sh "oc expose service ${app} -n ${project} || echo 'Service already exposed'"
}
