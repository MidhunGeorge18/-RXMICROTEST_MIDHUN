/* 
 Library declaration.
  Notes: 
  identifier includes the version of the library (git tag / branch)
  remote includes the repository git url 
  credentialsId needs to be of the type SSH key in Jenkins
  at the end of the declaration loads the whole library 
  This section always runs in the master jenkins. 
*/ 
try { 
    library( 
        identifier: 'jsl-jenkins-shared-library@release/20220603', 
        retriever: modernSCM( 
            [ 
                $class: 'GitSCMSource', 
                remote: 'https://github.com/centurylink/jsl-jenkins-shared-library.git', 
                credentialsId: 'GITHUB_APP_CREDENTIALS', 
                extensions: [[$class: 'WipeWorkspace']] 
            ] 
        ) 
    ) _ 
} catch (Exception Ex) { 
    library( 
        identifier: 'jsl-jenkins-shared-library@release/20220603', 
        retriever: modernSCM( 
            [ 
                $class: 'GitSCMSource', 
                remote: "git@github.com:CenturyLink/jsl-jenkins-shared-library.git", 
                credentialsId: 'SCMAUTO_SSH_DEVOPS_PIPELINE', 
                extensions: [[$class: 'WipeWorkspace']] 
            ] 
        ) 
    ) _ 

} 

pipeline { 
    environment { 
        /*  
        Credentials: 
        GITHUB_TOKEN_CREDENTIALS github token, jenkins user password credential 
        GITHUB_SSH_CREDENTIALS github ssh private key, jenkins private key credential. 
        DOCKER_CREDENTIALS Docker access info, jenkins secret file credential with environment variables to export 
        KUBE_CREDENTIALS Kubernetes access info, jenkins secret file credential with environment variables to export. For PRs. 
        KUBE_CREDENTIALS_TEST Kubernetes access info, jenkins secret file credential with environment variables to export. For branches. 
        AMAZON_CREDENTIALS AWS access info, jenkins secret file credential with environment variables to export 
        SONARQUBE_CREDENTIALS Sonarqube access info, jenkins secret text 
        GCP_CREDENTIALS GCP access info, jenkins secret file credential with environment variables to export 
        JIRA_CREDENTIALS Jira access info, jenkins secret file credential with environment variables to export 
        MSTEAMS_CREDENTIALS MS Teams access info, jenkins secret text 

        */ 

        GITHUB_TOKEN_CREDENTIALS = 'GITHUB_APP_CREDENTIALS'
        GITHUB_SSH_CREDENTIALS = 'SCMAUTO_SSH_DEVOPS_PIPELINE' 
        DOCKER_CREDENTIALS = 'nexus-secrets-file' 
        KUBE_CREDENTIALS = 'kube-secret-dev' 
        KUBE_CREDENTIALS_TEST = 'kube-secret-test' 
        KUBE_CREDENTIALS_PROD = '' 
        SONARQUBE_CREDENTIALS = 'sonarscnprod' 
        QUALITY_GATE_CREDENTIALS = 'qualitygate-secret' 
        JIRA_CREDENTIALS = 'jira-credentials' 
        MSTEAMS_CREDENTIALS = 'teams-secret' 
        TAG = '${env.NODE_NAME}' 
        XRAY_CREDENTIALS = 'xray-credentials' 
        JIRA_SERVER_INSTANCE_ID = 'CLOUD-482e318d-9fc6-4be6-9089-2646e78fcac3' 

        //Deployment control credentialsId 
        AUTHORIZED_USERS = 'authorized_users' 
        DEPLOY_AUTH_TOKEN = 'deploy_auth_token' 

        // Custom project variables 
        BRANCH_NAME = GIT_BRANCH.split('/')[-1].trim().toLowerCase() 
        COMMIT_ID = GIT_COMMIT.substring(0, 7).trim().toLowerCase() 
        PULL_REQUEST = "pr-${env.CHANGE_ID}" 

        // Custom project variables 
        PROJECT_NAME = 'RXMICRO-QA/RXMICROTEST_MIDHUN' //Should be Repo-name 
        PROJECT_MAL = 'RXMICRO' 
        XUNIT_XML_RESULTS_PATH = '.' 
        XUNIT_XML_FILE_PREFIX = 'generatedJUnitFiles/*/*/*.xml' 
        XUNIT_THRESHOLDS_JSON_FILE = 'cicd/qgatethresholds/qgate1-xunit.json' 
        JUNIT_XML_RESULTS_PATH = 'generatedJUnitFiles/*/*/*.xml' 
       
        //For destination images 
        DOCKER_REPO = 'RXMICROTEST_MIDHUN' //MAL-NAME/Name from Jenkins
        IMAGE_NAME = "${env.PROJECT_NAME}" 
        IMAGE_TAG = "${env.PULL_REQUEST}" 
        KUBE_DOCKER_SECRET_NAME = "${env.PROJECT_NAME}-${env.PULL_REQUEST}" 
        KUBE_DOCKER_SECRET_NAME_TEST = "${env.PROJECT_NAME}-${env.BRANCH_NAME}" 
        KUBE_DOCKER_SECRET_NAME_PROD = "${env.PROJECT_NAME}-${env.BRANCH_NAME}" 

        //App Specific Variables that will change project to project
        MAL = 'RXMICRO'
    } 



    parameters { 
        choice(choices: ['test1', 'test2', 'test4', 'e2e'], description: '', name: 'Environment') 
        string(defaultValue: 'ARCADE', description: '', name: 'TestTool', trim: false) 
        choice(choices: ['Sanity','Regression'], description: '', name: 'TestType') 
        string(defaultValue: 'BMCC', description: '', name: 'Project', trim: false) 
        string(defaultValue: 'BMCC-98210', description: '', name: 'TestPlan', trim: false) 
        string(defaultValue: 'Test1', description: '', name: 'XrayEnvironment', trim: false)  

    }
 

    agent { 
        //https://www.jenkins.io/doc/book/pipeline/syntax/#agent 
        label 'Docker-enabled' 
    } 

 

    options { 
        //https://www.jenkins.io/doc/book/pipeline/syntax/#options 
        timestamps() 
        timeout(time: 1, unit: 'HOURS') 
        buildDiscarder(logRotator(numToKeepStr: '4', daysToKeepStr: '8')) 
        disableConcurrentBuilds() 
        preserveStashes(buildCount: 10) 

    } 

 

    triggers { 
        //https://www.jenkins.io/doc/book/pipeline/syntax/#triggers 
        issueCommentTrigger('.*test this please.*') 
    } 

 

    //Run ARCADE SOAtest tst files or projects QA Staging

    stages { 
        stage('Quality Gate 2'){ 
         
            stages { 
                //Sanity test  // Update the docker file with the folder
                stage('Sanity Tests') { 
                    agent { 
                        dockerfile { 
                            filename 'Dockerfile' 
                            dir 'cicd/docker' 
                            label 'Docker-enabled' 
                        } 

                    } 

                    steps { 
                        script { 
                            
                            println "Will test in ${params.MAL}" 
                            println "Workspace is : ${env.WORKSPACE}" 
                            sh script: ''' 
                                #PWD cmd will give the name of the workspace when uncommented 
                                #pwd 
                                #List of all folders in workspace when uncommented 
                                #ls -latR  
                                #The below is working CLI CMD 
                                OLD_PWD=$(pwd) 
                                cd ${WORKSPACE} 
                                
                            #Below is an example of running a Rest API test 
                            \"/soatest/parasoft/soatest_virtualize/9.10/soatestcli\" -data \"${WORKSPACE}" -import \"${WORKSPACE}\" 
                              
                            \"/soatest/parasoft/soatest_virtualize/9.10/soatestcli\" -data \"${WORKSPACE}\" -resource \"TestAssets/sanity--lq/CRIS JSON and XML TEST Sanity.tst\" -environment %Environment% -config \"soatest.user://Example Configuration\" -report \"${OLD_PWD}/ARCADE_REPORT.xml\" -localsettings \"${WORKSPACE}/lib/Testsettings.properties\" -appconsole \"stdout\" 
                            cd $OLD_PWD     
                       
                          
                            ''' 
                            println '/soatest/parasoft/test/9.10_for_soatest_virtualize/configuration/*.log.' 
                            //Stashing the Results is another way of saving in memory for using later 
                            stash name: "RESULTS1", includes: "ARCADE_REPORT.xml" 
                            stash name: "RESULTS2", includes: "ARCADE_REPORT.html" 
                            // echo ">>>>>>>>>>>> Done Stashing RESULTS........" 
                        } 
                    } 
                } 
 

                //XRAY import test execution results 
                stage('Jira Integration') { 
                    steps { 
                        unstash 'RESULTS1' 
                        step([$class: 'XUnitPublisher', 
                            thresholds: [[$class: 'FailedThreshold', unstableThreshold: '']], 
                            tools: [[$class: 'ParasoftSOAtest9xType', deleteOutputFiles: false, failIfNotNew: false, pattern: 'ARCADE_REPORT.xml', skipNoTestFiles: false, stopProcessingIfError: true]]]) 

                        stash name: "RESULTS", includes: "generatedJUnitFiles/**/**/**.xml"//"./generatedJUnitFiles/*/*/*.xml" 
                       script { 
                           jslJiraXrayResultImport('junit', 'generatedJUnitFiles/*/*/*.xml', 'This execution is automatically created when importing execution results from jenkins', '${TESTTTOOL}', '${Project}', '${TestPlan}', '${XrayEnvironment}') 
                        } 
                    } 
                } 

 

                stage('Tests Result Email Notification') { 
                    steps { 
                        script { 
                            unstash 'RESULTS2' 
                            jslEmailNotification('Midhun.George@lumen.com','john.c2.campbell@lumen.com', 'Job Name:${JOB_NAME} BuildNo:${BUILD_NUMBER} Status:${BUILD_STATUS}', '**/ARCADE_REPORT.html', '''ARCADE Test Results Attached: <br>Build Url : ${BUILD_URL}''') 
                        } 
                    } 
                } 

                 //Building Teams Communications of Reports Into the Teams Channel on Build Status                                
                stage('Posting Results to Teams Notification') {
                    steps {
                        echo 'Posting to Teams Channel'
                    office365ConnectorSend message: 'Sanity has been Done and reported to :(https://ctl.atlassian.net/browse/BMCC-98210) Build Url : ${BUILD_URL}', webhookUrl: 'https://centurylink.webhook.office.com/webhookb2/eb78ed92-ce5d-4490-9bf1-f556cd927440@72b17115-9915-42c0-9f1b-4f98e5a4bcd2/JenkinsCI/8065c98a47ae41afb7b7534997788110/6732ab06-c733-4e15-b843-91a5760cbf4d'
                               }
                            }

                //Build JUnit report for XRAY and set thresholds  Update the DIR with folder

                stage('Quality Gate'){ 
                    agent { 
                        dockerfile { 
                            filename 'Dockerfile' 
                            dir 'cicd/docker/qualitygate/' 
                            label 'Docker-enabled' 
                        } 
                    } 
                    steps { 
                        script { 
                            // unstash 'RESULTS1' 
                            unstash 'RESULTS' 

                            jslCheckQualityGates("generatedJUnitFiles/*/*/*.xml", "QualityGate2-Sanity") 
                        } 
                    }
 
                } 
            } 
        } 


        //This updates the Github for DevOps in relation to jobs on the pipeline : UPdate DIR with Folder

        stage('Adoption Stats') { 
            agent { 
                dockerfile { 
                    filename 'Dockerfile' 
                    dir 'cicd/docker/JiraAdoption/' 
                    label 'Docker-enabled' 
                } 
            } 
            steps { 
                script { 
                    unstash name: 'JSL_QA_REPORT_STASH'
                    unstash name: 'JSL_QA_REPORT_STASH' 
                    jslAdoptionMain('JSL_QA_REPORT_STASH/*.json') 
                } 
            } 
        } 
    } 

 
    post { 

        //https://www.jenkins.io/doc/book/pipeline/syntax/#post 
        success { 
            println("This is for success") 
            //jslNotification('success') 
            //cleanWs() 
        } 
        failure { 
            println("This is for failure") 
            //jslNotification('failure') 
            //cleanWs() 
        } 
        unstable { 
            println("This is for unstable") 
            //jslNotification('unstable') 
            //cleanWs() 
        } 
    } //post 

} //Pipelin
