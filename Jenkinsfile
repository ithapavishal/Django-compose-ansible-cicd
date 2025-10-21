pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "employee"
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                checkout scm
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                echo "[defaults]" > ansible.cfg
                echo "host_key_checking = False" >> ansible.cfg
                echo "remote_user = vagrant" >> ansible.cfg
                echo "private_key_file = /home/vagrant/ansible-keys" >> ansible.cfg
                '''
            }
        }

        // stage('Build & Start with Docker Compose') {
        //     steps {
        //         script {
        //             withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
        //                 sh '''
        //                 echo "Logging into DockerHub and building images..."
        //                 docker compose down --remove-orphans

        //                 for i in 1 2 3; do
        //                     docker compose build && break || {
        //                         echo "Build failed... retrying in 10s ($i/3)"
        //                         sleep 10
        //                     }
        //                 done

        //                 docker compose up -d
        //                 '''
        //             }
        //         }
        //     }
        // }


        stage('Build & Start with Docker Compose') {
        steps {
            script {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
                    sh '''
                    echo "Building and starting Docker Compose services..."
                    docker-compose down || true
                    docker-compose build --no-cache
                    docker-compose up -d
                    
                    echo "Waiting for services to be healthy..."
                    sleep 30
                    
                    echo "Checking running containers..."
                    docker-compose ps
                    
                    echo "Checking Django service logs..."
                    docker-compose logs web_services
                    
                    echo "Checking PostgreSQL service logs..."
                    docker-compose logs postgres_db
                    '''
                }
            }
        }
    }


        // stage('Run Migrations') {
        //     steps {
        //         sh '''
        //         docker compose run --rm web_services python manage.py migrate --noinput
        //         '''
        //     }
        // }

        stage('Push to DockerHub') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
                        sh '''
                        IMAGE_NAME=thapavishal/employee
                        echo "Tagging and pushing image to DockerHub..."
                        docker tag ${COMPOSE_PROJECT_NAME}_web_services:latest $IMAGE_NAME:${BUILD_NUMBER}
                        docker tag ${COMPOSE_PROJECT_NAME}_web_services:latest $IMAGE_NAME:latest
                        docker push $IMAGE_NAME:${BUILD_NUMBER}
                        docker push $IMAGE_NAME:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Development') {
            steps {
                sshagent(credentials: ['vagrant-ssh-key']) {
                    sh '''
                    ansible-playbook -i ansible/inventory/dev.ini ansible/playbooks/deploy.yaml \
                    --extra-vars "env=dev image_tag=thapavishal/employee:$BUILD_NUMBER"
                    '''
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                timeout(time: 1, unit: 'DAYS') {
                    input message: 'Approve production deployment?'
                }
                sshagent(credentials: ['vagrant-ssh-key']) {
                    sh '''
                    ansible-playbook -i ansible/inventory/prod.ini ansible/playbooks/deploy.yml \
                    --extra-vars "env=prod image_tag=thapavishal/employee:$BUILD_NUMBER"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Build finished. Check status at ${BUILD_URL}"
        }
    }
}











// ************* Old code 

// pipeline {
//     agent any

//     environment {
//         COMPOSE_PROJECT_NAME = "employee"
//         ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"
//     }

//     stages {
//         stage('Checkout from GitHub') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Setup Environment') {
//             steps {
//                 sh '''
//                 echo "[defaults]" > ansible.cfg
//                 echo "host_key_checking = False" >> ansible.cfg
//                 echo "remote_user = vagrant" >> ansible.cfg
//                # echo "private_key_file = /var/lib/jenkins/.ssh/id_rsa" >> ansible.cfg
//                 echo "private_key_file = /home/vagrant/ansible-keys" >> ansible.cfg
//                 '''
//             }
//         }

//         stage('Build & Start with Docker Compose') {
//             steps {
//                 sh '''
//                 docker compose down --remove-orphans
//                 docker compose build
//                 docker compose up -d
//                 '''
//             }
//         }

//         stage('Run Migrations') {
//             steps {
//                 sh '''
//                 docker compose run --rm web_services python manage.py migrate --noinput
//                 '''
//             }
//         }

//         // stage('Push to DockerHub') {
//         //     steps {
//         //         withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
//         //             // push the web_services image that was built
//         //             sh '''
//         //             IMAGE_NAME=thapavishal/employee
//         //         #    docker tag employee_web_services:latest thapavishal/employee:4

//         //             docker tag ${COMPOSE_PROJECT_NAME}_web_services:latest $IMAGE_NAME:$BUILD_NUMBER
//         //             docker push $IMAGE_NAME:$BUILD_NUMBER
//         //             '''
//         //         }
//         //     }
//         // }

//     //     stage('Push to DockerHub') {
//     //     steps {
//     //         script {
//     //             withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
//     //             // docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-id') {
//     //                 sh '''
//     //                 IMAGE_NAME=thapavishal/employee
//     //                 docker tag employee-web_services:latest $IMAGE_NAME:${BUILD_NUMBER}
//     //                 docker push $IMAGE_NAME:${BUILD_NUMBER}
//     //                 docker push $IMAGE_NAME:latest
//     //                 '''
//     //             }
//     //         }
//     //     }
//     // }

//         stage('Push to DockerHub') {
//         steps {
//             script {
//                 withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
//                     sh '''
//                     IMAGE_NAME=thapavishal/employee
//                     echo "Tagging and pushing image to DockerHub..."
//                     docker tag employee-web_services:latest $IMAGE_NAME:${BUILD_NUMBER}
//                     docker tag employee-web_services:latest $IMAGE_NAME:latest
//                     docker push $IMAGE_NAME:${BUILD_NUMBER}
//                     docker push $IMAGE_NAME:latest
//                     '''
//                 }
//             }
//         }
//     }



//         stage('Deploy to Development') {
//             steps {
//                 sshagent(credentials: ['vagrant-ssh-key']) {
//                     sh '''
//                     ansible-playbook -i ansible/inventory/dev.ini ansible/playbooks/deploy.yaml \
//                     --extra-vars "env=dev image_tag=thapavishal/employee:$BUILD_NUMBER"
//                     '''
//                     }
//                 }
//             }


//         // stage('Deploy to Development') {
//         //     steps {
//         //         sshagent(credentials: ['vagrant-ssh-key']) {
//         //             sh '''
//         //             ansible-playbook -i inventory deploy.yaml \
//         //               --extra-vars "env=dev image_tag=thapavishal/elearning:$BUILD_NUMBER"
//         //             '''
//         //         }
//         //     }
//         // }

//         stage('Deploy to Production') {
//             steps {
//                 timeout(time: 1, unit: 'DAYS') {
//                     input message: 'Approve production deployment?'
//                 }
//                 sshagent(credentials: ['vagrant-ssh-key']) {
//                     sh '''
//                     ansible-playbook -i ansible/inventory/prod.ini ansible/playbooks/deploy.yml \
//                     // ansible-playbook -i inventory deploy.yaml \
//                       --extra-vars "env=prod image_tag=thapavishal/employee:$BUILD_NUMBER"
//                     '''
//                 }
//             }
//         }
//     }

//     post {
//     always {
//         echo "Build finished. Check status at ${BUILD_URL}"
//     }
// }


//     // post {
//     //     always {
//     //         mail to: 'v01.thapa@gmail.com',
//     //             subject: "Job '${JOB_NAME}' (${BUILD_NUMBER}) status",
//     //             body: "Go to ${BUILD_URL} and verify the build"
//     //     }
//     //     success {
//     //         mail body: """Hi Team,
//     //         Build #$BUILD_NUMBER succeeded. Visit:
//     //         $BUILD_URL
//     //         Regards,
//     //         DevOps Team""",
//     //         subject: 'BUILD SUCCESS NOTIFICATION',
//     //         to: 'v01.thapa@gmail.com'
//     //     }
//     //     failure {
//     //         mail body: """Hi Team,
//     //         Build #$BUILD_NUMBER failed. Visit:
//     //         $BUILD_URL
//     //         Regards,
//     //         DevOps Team""",
//     //         subject: 'BUILD FAILED NOTIFICATION',
//     //         to: 'v01.thapa@gmail.com'
//     //     }
//     // }
// }

// // updated jenkinsfile 