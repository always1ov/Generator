pipeline {
    agent {
        docker {
            image 'python:3.10.8'
            args '-u root'  // 使用root权限安装必要软件
            reuseNode true
        }
    }
    
    triggers {
        cron('0 5 * * *')  // 每天UTC时间5点执行（对应北京时间13点）
    }
    
    environment {
        GH_CREDENTIALS = credentials('github-token')  // Jenkins凭证ID
    }
    
    stages {
        stage('安装基础工具') {
            steps {
                sh 'apt-get update && apt-get install -y git'
            }
        }
        
        stage('检出代码') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],  // 根据实际分支调整
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/always1ov/your-repo.git'  // 替换为实际仓库URL
                    ]]
                ])
            }
        }
        
        stage('环境变量设置') {
            steps {
                script {
                    // 自动解析仓库所有者/名称
                    def repoUrl = sh(script: 'git config --get remote.origin.url', returnStdout: true).trim()
                    def matcher = (repoUrl =~ /github.com[:\/](.*?)\/(.*?)\.git/)
                    if (matcher) {
                        env.REPO_FULL = "${matcher[0][1]}/${matcher[0][2]}"
                    } else {
                        error "仓库URL解析失败: ${repoUrl}"
                    }
                    
                    // 获取当前分支（自动去除origin/前缀）
                    def rawBranch = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    env.GIT_BRANCH = rawBranch.replaceAll('^origin/', '')
                }
            }
        }
        
        stage('安装依赖') {
            steps {
                sh '''
                python -m pip install --upgrade pip
                pip install pyarmor==9.0.7 pyinstaller===5.13.2
                '''
            }
        }
        
        stage('生成许可证') {
            steps {
                sh '''
                pyarmor cfg outer_keyname=Qmusic.lic
                pyarmor gen key -e 31
                '''
            }
        }
        
        stage('强制更新仓库') {
            steps {
                sh '''
                # 准备目录并覆盖许可证
                mkdir -p license
                \\cp -f dist/Qmusic.lic license/
                
                # 配置Git身份
                git config --global user.name "always1ov"
                git config --global user.email "always1ov@github.com"
                git config --global http.postBuffer 524288000
                
                # 使用Token认证
                git remote set-url origin "https://always1ov:${GH_CREDENTIALS}@github.com/${REPO_FULL}.git"
                
                # 强制提交变更
                git add --force license/Qmusic.lic
                git commit -m "Auto-update license [skip ci]" --allow-empty
                git push --force-with-lease origin "HEAD:${GIT_BRANCH}"
                '''
            }
        }
    }
}
