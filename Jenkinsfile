pipeline {
  agent any
  stages {
    stage('环境检查') {
      steps {
        sh 'printenv'
        echo '正在检测基本信息'
        sh 'java -version'
        sh 'git --version'
        sh 'docker version'
        sh 'pwd && ls -alh'
        sh "echo $hello"
        sh 'echo ${world}'
        sh 'ssh --help'
      }
    }

    stage('maven编译') {
      agent {
        docker {
          image 'maven:3-alpine'
          args '-v /var/jenkins_home/appconfig/maven/.m2:/root/.m2'
        }

      }
      steps {
        sh 'pwd && ls -alh'
        sh 'mvn -v'
        sh "echo 默认的工作目录：${WS}"
        sh 'cd ${WS} && mvn clean package -s "/var/jenkins_home/appconfig/maven/settings.xml"  -Dmaven.test.skip=true '
      }
    }

    stage('测试') {
      steps {
        sh 'pwd && ls -alh'
        echo '测试...'
      }
    }

    stage('生成镜像') {
      steps {
        echo '打包...'
        sh 'docker version'
        sh 'pwd && ls -alh'
        sh 'docker build -t java-devops-demo .'
      }
    }

    stage('推送镜像') {
      input {
        message '需要推送远程镜像吗?'
        id '需要'
        parameters {
          string(name: 'APP_VER', defaultValue: 'v1.0', description: '生产环境需要部署的版本')
          choice(choices: ['bj-01', 'sh-02', 'wuhan-01'], description: '部署的大区', name: 'DEPLOY_WHERE')
        }
      }
      steps {
        echo "$APP_VER"
        script {
          def  where = "${DEPLOY_WHERE}"

          if (where == "bj-01"){
            sh "echo 我帮你部署到 bj-01 区了"
          }else if(where == "sh-02"){
            sh "echo 我帮你部署到 sh-02 区了"
          }else{
            sh "echo 没人要的，我帮你部署到 wuhan-01 区了"
            //                    sh "docker push registry.cn-hangzhou.aliyuncs.com/lfy/java-devops-demo:${APP_VER}"
            withCredentials([usernamePassword(credentialsId: 'aliyun-docker-repo', passwordVariable: 'ali_pwd', usernameVariable: 'ali_user')]) {
              // some block
              sh "docker login -u ${ali_user} -p ${ali_pwd}   registry.cn-hangzhou.aliyuncs.com"
              //                              sh "docker tag java-devops-demo registry.cn-hangzhou.aliyuncs.com/lfy/java-devops-demo:${APP_VER}"
            }

            //ssh 秘钥文件配置到 jenkins 全局秘钥中
            withCredentials(ssh){
              //ansible 没有
              sh "ssh root@xxxx  "
              //不应该的操作。
              sh "远程操作其他机器。。。。"

              //k8s集群
              //动态切换k8s集群

            }
          }
        }

      }
    }

    stage('部署') {
      steps {
        echo '部署...'
        sh 'docker rm -f java-devops-demo-dev'
        sh 'docker run -d -p 8888:8080 --name java-devops-demo-dev java-devops-demo'
      }
    }

    stage('发送报告') {
      steps {
        echo '准备发送报告'
        emailext(body: '''<!DOCTYPE html>
                <html>
                <head>
                <meta charset="UTF-8">
                <title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>
                </head>

                <body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4"
                    offset="0">
                    <table width="95%" cellpadding="0" cellspacing="0"  style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
                <h3>本邮件由系统自动发出，请勿回复！</h3>
                        <tr>
                           <br/>
                            各位同事，大家好，以下为${PROJECT_NAME }项目构建信息</br>
                            <td><font color="#CC0000">构建结果 - ${BUILD_STATUS}</font></td>
                        </tr>
                        <tr>
                            <td><br />
                            <b><font color="#0B610B">构建信息</font></b>
                            <hr size="2" width="100%" align="center" /></td>
                        </tr>
                        <tr>
                            <td>
                                <ul>
                                    <li>项目名称 ： ${PROJECT_NAME}</li>
                                    <li>构建编号 ： 第${BUILD_NUMBER}次构建</li>
                                    <li>触发原因： ${CAUSE}</li>
                                    <li>构建状态： ${BUILD_STATUS}</li>
                                    <li>构建日志： <a href="${BUILD_URL}console">${BUILD_URL}console</a></li>
                                    <li>构建  Url ： <a href="${BUILD_URL}">${BUILD_URL}</a></li>
                                    <li>工作目录 ： <a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a></li>
                                    <li>项目  Url ： <a href="${PROJECT_URL}">${PROJECT_URL}</a></li>
                                </ul>


                <h4><font color="#0B610B">最近提交</font></h4>
                <ul>
                <hr size="2" width="100%" />
                ${CHANGES_SINCE_LAST_SUCCESS, reverse=true, format="%c", changesFormat="<li>%d [%a] %m</li>"}
                </ul>
                详细提交: <a href="${PROJECT_URL}changes">${PROJECT_URL}changes</a><br/>

                            </td>
                        </tr>
                    </table>
                </body>
                </html>''', subject: '${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志', to: '17512080612@163.com')
      }
    }

    stage('部署到生产环境吗？') {
      steps {
        sh 'echo 发布版本咯......'
      }
    }

  }
  environment {
    hello = '123456'
    world = '456789'
    WS = "${WORKSPACE}"
    IMAGE_VERSION = 'v1.0'
    ALIYUN_SECRTE = credentials('aliyun-docker-repo')
  }
  post {
    failure {
      echo "这个阶段 完蛋了.... $currentBuild.result"
    }

    success {
      echo "这个阶段 成了.... $currentBuild.result"
    }

  }
}