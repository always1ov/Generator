pipeline {
    agent any
    triggers {
        cron('H 1 * * *')  // 每天凌晨1点左右执行（Jenkins推荐使用H参数分散负载）
    }
    stages {
        stage('Setup Environment') {
            steps {
                sh '''
                    python3.10 --version  // 验证Python版本
                    python3.10 -m venv venv  // 创建虚拟环境
                    . venv/bin/activate  // 激活虚拟环境
                    pip install pyarmor==9.0.7 pyinstaller===5.13.2  // 安装依赖
                '''
            }
        }
        stage('Configure PyArmor') {
            steps {
                sh '''
                    . venv/bin/activate  // 确保在虚拟环境中
                    pyarmor cfg outer_keyname=Qmusic.lic  // 配置PyArmor
                '''
            }
        }
        stage('Generate License') {
            steps {
                sh '''
                    . venv/bin/activate  // 确保在虚拟环境中
                    pyarmor gen key -e 31  // 生成密钥
                '''
            }
        }
    }
    post {
        always {
            cleanWs()  // 可选：自动清理工作空间
        }
    }
}
