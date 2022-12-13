def app_version = ""
def commit_log = ""
def app_version_file = "app/build.gradle"
def app_build_file = "app/build/outputs/apk/release/app-release.apk"
def app_upload_file = ""
def jenkins_build_time = 0
def jenkins_build_id = 0
def jenkins_build_author = ""
def jenkins_build_branch = ""
def jenkins_build_url = ""
def pgy_ukey = "ae91ade72b4baf95f0a5d83a4bb9a20f"
def pgy_api_key = "83ad6652cdac12336f85491d4afdba34"
def pgy_app_key = "dae8210a2542f92c1ebf1e9b7b1d139b"
def pgy_download_url = ""
def pgy_build_id = ""
def pgy_build_key = ""
def pgy_install_password = "*0216*"
def send_email_to = "shaomin.deng@nio.com"
def send_email_from = "pe_app_ci@nioint.com"
def app_name = ""
def app_size = 0
def start_build_time = 0
def apk_root_dir = "apks"
def feishu_commit_log = ""
pipeline {
    agent any
    // parameters {
    //     choice choices: ['Test', 'Release'], description: '包类型', name: 'packageType'
    // }

    stages {
        stage('初始化') {
            steps {
                script {
                    //get app version
                    def data = readFile(file: "${app_version_file}")
                    def lines = data.readLines()
                    for (line in lines) {
                        if (line.contains("versionName")) {
                            def contents = line.split(" ")
                            app_version = contents[contents.length - 1].replace("\"", "")
                            break
                        }
                    }
                    jenkins_build_author = currentBuild.buildCauses.shortDescription
                    jenkins_build_id = currentBuild.number
                    jenkins_build_branch = "${Branch}"
                    jenkins_build_url = env.JOB_DISPLAY_URL
                }
            }
        }

//         stage('拉取代码') { //不适用scm时手动拉代码（不能使用GitParameter 分支或标签参数）；scm会自动拉代码
//             steps {
//                 script{
//                     try {
//                         echo "start fetch code"
//                         git branch: 'master', credentialsId: 'f4b64596-e95b-4af3-88dd-46b9f3cac084', url: 'git@github.com:dengshaomin/JenkinsTest.git'
//                         echo "fnish fetch code"
//                     } catch(Exception e){
//                         echo "fail fetch code"
//                         echo "${e.message}"
//                     }
//                 }
//             }
//         }
        stage('获取提交日志') {
            steps {
                script {
                    echo "start get commitlog"
                    def build = currentBuild
                    def count = 1
                    while (build != null) {
                        def changeLogSets = build.changeSets
                        for (int i = 0; i < changeLogSets.size(); i++) {
                            def entries = changeLogSets[i].items
                            for (int j = 0; j < entries.length; j++) {
                                def entry = entries[j]
                                def tmp_content = "${count}.${entry.msg}(${entry.author}-${changeTime(entry.timestamp)})"
                                commit_log += "${tmp_content}<br>"
                                feishu_commit_log += "${tmp_content}\\n"
                                count++
                            }
                        }
                        build = build.previousBuild
                    }
                    echo "commitlog：${commit_log}"
                    echo "finish get commitlog"
                }
            }
        }
        stage('编译') {
            steps {
                echo "start build apk.."
                echo "packageType:${packageType}"
                script {
                    sh "chmod +x gradlew"
                    sh "rm -rf app/build"
                    sh "./gradlew clean assembleRelease -P packageType=${packageType}"
                    //-p  write params to app gradle.properties
                }

                echo "finish build apk"
                script {
                    start_build_time = currentBuild.timeInMillis
                    try {
                        sh "rm -rf ${apk_root_dir}"
                    } catch (Exception e) {
                    }
                    try {
                        sh "mkdir ${apk_root_dir}"
                    } catch (Exception e) {
                    }
                    try {
                        app_name = "${packageType}_${app_version}_${getPackageTime(start_build_time)}.apk"
                        app_upload_file = "${apk_root_dir}/${app_name}"
                        sh "mv ${app_build_file} ${app_upload_file}"
                    } catch (Exception e) {
                    }
                    jenkins_build_time = currentBuild.durationString
                }
            }
        }
        stage('上传蒲公英') {
            steps {
                script {
                    try {
                        echo "start upload pgy"
                        def result = sh(returnStdout: true, script: "curl -F 'file=@$app_upload_file' -F 'buildDescription=$commit_log' -F '_api_key=$pgy_api_key' https://www.pgyer.com/apiv2/app/upload").trim()
                        def json = readJSON text: result // !need pipeline-utility-steps plugin
                        echo "$json"
                        def v =  (json.data.buildFileSize as int)/(1024*1024)
                        echo "$v"
                        try{
                            v = (v as String).substring(0,5)
                        }catch(Exception e){
                        }
                        app_size = v
                        echo "$app_size"
                        pgy_build_id = json.data.buildBuildVersion
                        pgy_build_key = json.data.buildKey
                        pgy_download_url = "https://www.pgyer.com/apiv2/app/install?_api_key=${pgy_api_key}&buildKey=${pgy_build_key}&buildPassword=${pgy_install_password}"
                        if (json.code != 0) {
                            throw new RuntimeException("upload pgy fail")
                        }
                        echo "finish upload pgy"
                    } catch (Exception e) {
                        throw new RuntimeException("upload pgy fail")
                    }

                }
            }
        }
        stage('飞书') {
            steps {
                script{

                    sh("""
                        curl https://open.feishu.cn/open-apis/bot/v2/hook/5d7b3ada-5f0f-4fbf-b4af-6a5b067d227e -H 'Content-type: application/json' -d '
                        {
                            "msg_type":"interactive",
                            "card":{
                                "config":{
                                    "wide_screen_mode":true
                                },
                                "header":{
                                    "title":{
                                        "tag":"plain_text",
                                        "content":"${app_name}"
                                    },
                                    "template":"turquoise"
                                },
                                "elements":[
                                    {
                                        "tag":"div",
                                        "text":{
                                            "content":"蒲公英id：${pgy_build_id}\\n\\nApp大小：${app_size}M\\n\\n编译人：${jenkins_build_author}\\n\\n编译分支：${jenkins_build_branch}\\n\\n编译耗时：${jenkins_build_time}\\n\\n提交日志：\\n${feishu_commit_log}",
                                            "tag":"lark_md"
                                        }
                                    },
                                    {
                                        "actions":[
                                            {
                                                "tag":"button",
                                                "text":{
                                                    "content":"立即下载",
                                                    "tag":"lark_md"
                                                },
                                                "url":"${pgy_download_url}",
                                                "type":"primary",
                                                "value":{}
                                            },
                                            {
                                                "tag":"button",
                                                "text":{
                                                    "tag":"plain_text",
                                                    "content":"查看构建任务"
                                                },
                                                "type":"default",
                                                "url":"${jenkins_build_url}"
                                            }
                                        ],
                                        "tag":"action"
                                    },
                                    {
                                    	"tag": "at",
                                    	"user_id": "all",
                                    	"user_name": "所有人"
                                    }
                                ]
                            }
                        }
                        '
                    """
                    )
                }
            }
        }

    }
    post {
            success {
                mail subject: "'${env.JOB_NAME} [${env.BUILD_NUMBER}]' 编译成功",
                body: """
                <!DOCTYPE html>
                <html xmlns:v="urn:schemas-microsoft-com:vml" xmlns:o="urn:schemas-microsoft-com:office:office" lang="en">

                <head>
                    <title></title>
                    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
                    <meta name="viewport" content="width=device-width, initial-scale=1.0">
                    <!--[if mso]><xml><o:OfficeDocumentSettings><o:PixelsPerInch>96</o:PixelsPerInch><o:AllowPNG/></o:OfficeDocumentSettings></xml><![endif]-->
                    <style>
                        * {
                            box-sizing: border-box;
                        }

                        body {
                            margin: 0;
                            padding: 0;
                        }

                        a[x-apple-data-detectors] {
                            color: inherit !important;
                            text-decoration: inherit !important;
                        }

                        #MessageViewBody a {
                            color: inherit;
                            text-decoration: none;
                        }

                        p {
                            line-height: inherit
                        }

                        .desktop_hide,
                        .desktop_hide table {
                            mso-hide: all;
                            display: none;
                            max-height: 0px;
                            overflow: hidden;
                        }

                        @media (max-width:620px) {
                            .row-content {
                                width: 100% !important;
                            }

                            .mobile_hide {
                                display: none;
                            }

                            .stack .column {
                                width: 100%;
                                display: block;
                            }

                            .mobile_hide {
                                min-height: 0;
                                max-height: 0;
                                max-width: 0;
                                overflow: hidden;
                                font-size: 0px;
                            }

                            .desktop_hide,
                            .desktop_hide table {
                                display: table !important;
                                max-height: none !important;
                            }
                        }
                    </style>
                </head>

                <body style="background-color: transparent; margin: 0; padding: 0; -webkit-text-size-adjust: none; text-size-adjust: none;">
                    <table class="nl-container" width="100%" border="0" cellpadding="0" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt; background-color: transparent;">
                        <tbody>
                            <tr>
                                <td>
                                    <table class="row row-1" align="center" width="100%" border="0" cellpadding="0" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt;">
                                        <tbody>
                                            <tr>
                                                <td>
                                                    <table class="row-content stack" align="center" border="0" cellpadding="0" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt; color: #000000; width: 600px;" width="600">
                                                        <tbody>
                                                            <tr>
                                                                <td class="column column-1" width="33.333333333333336%" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt; font-weight: 400; text-align: left; vertical-align: top; border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px;">
                                                                    <table class="image_block block-2" width="100%" border="0" cellpadding="0" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt;">
                                                                        <tr>
                                                                            <td class="pad" style="width:100%;padding-right:0px;padding-left:0px;padding-top:5px;padding-bottom:5px;">
                                                                                <div class="alignment" align="center" style="line-height:10px"><img src="https://www.nio.cn/cdn-static/mynio/nextjs/images/nio-power/nio-power-home-charger-7kw-gray.jpg" style="display: block; height: auto; border: 0; width: 200px; max-width: 100%;" width="200" alt="I'm an image" title="I'm an image"></div>
                                                                            </td>
                                                                        </tr>
                                                                    </table>
                                                                </td>
                                                                <td class="column column-2" width="66.66666666666667%" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt; font-weight: 400; text-align: left; vertical-align: top; border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px;">
                                                                    <table class="text_block block-2" width="100%" border="0" cellpadding="0" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt; word-break: break-word;">
                                                                        <tr>
                                                                            <td class="pad" style="padding-bottom:10px;padding-left:10px;padding-right:10px;padding-top:15px;">
                                                                                <div style="font-family: sans-serif">
                                                                                    <div class style="font-size: 12px; font-family: Arial, Helvetica Neue, Helvetica, sans-serif; mso-line-height-alt: 14.399999999999999px; color: #555555; line-height: 1.2;">
                                                                                        <p style="margin: 0; font-size: 16px; text-align: left; mso-line-height-alt: 19.2px;"><em><strong>${app_name}</strong></em></p>
                                                                                    </div>
                                                                                </div>
                                                                            </td>
                                                                        </tr>
                                                                    </table>
                                                                    <table class="divider_block block-3" width="100%" border="0" cellpadding="10" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt;">
                                                                        <tr>
                                                                            <td class="pad">
                                                                                <div class="alignment" align="center">
                                                                                    <table border="0" cellpadding="0" cellspacing="0" role="presentation" width="100%" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt;">
                                                                                        <tr>
                                                                                            <td class="divider_inner" style="font-size: 1px; line-height: 1px; border-top: 1px solid #BBBBBB;"><span>&#8202;</span></td>
                                                                                        </tr>
                                                                                    </table>
                                                                                </div>
                                                                            </td>
                                                                        </tr>
                                                                    </table>
                                                                    <table class="text_block block-4" width="100%" border="0" cellpadding="10" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt; word-break: break-word;">
                                                                        <tr>
                                                                            <td class="pad">
                                                                                <div style="font-family: sans-serif">
                                                                                    <div class style="font-size: 15px; font-family: Arial, Helvetica Neue, Helvetica, sans-serif; mso-line-height-alt: 14.399999999999999px; color: #555555; line-height: 1.2;">

                                                                                        <p style="margin: 0; font-size: 15px; text-align: left; mso-line-height-alt: 14.399999999999999px;"><em>蒲公英id：${pgy_build_id}</em></p>
                                                                                        <p style="margin: 0; font-size: 15px; text-align: left; mso-line-height-alt: 14.399999999999999px;"><em>App名称：${app_name}</em></p>
                                                                                        <p style="margin: 0; font-size: 15px; text-align: left; mso-line-height-alt: 14.399999999999999px;"><em>App大小：${app_size}M</em></p>
                                                                                        <p style="margin: 0; font-size: 15px; text-align: left; mso-line-height-alt: 14.399999999999999px;"><em>提交日志：</em></p>
                                                                                        <p style="margin: 0; font-size: 15px; text-align: left; mso-line-height-alt: 14.399999999999999px;"><em>${commit_log}</em></p>
                                                                                    </div>
                                                                                </div>
                                                                            </td>
                                                                        </tr>
                                                                    </table>
                                                                    <table class="button_block block-5" width="100%" border="0" cellpadding="0" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt;">
                                                                        <tr>
                                                                            <td class="pad" style="padding-bottom:15px;padding-left:10px;padding-right:10px;padding-top:10px;text-align:left;">
                                                                                <div class="alignment" align="left">
                                                                                    <!--[if mso]><v:roundrect xmlns:v="urn:schemas-microsoft-com:vml" xmlns:w="urn:schemas-microsoft-com:office:word" href="${pgy_download_url}" style="height:42px;width:88px;v-text-anchor:middle;" arcsize="10%" stroke="false" fillcolor="#3AAEE0"><w:anchorlock/><v:textbox inset="0px,0px,0px,0px"><center style="color:#ffffff; font-family:Arial, sans-serif; font-size:16px"><![endif]--><a href="${pgy_download_url}" target="_blank" style="text-decoration:none;display:inline-block;color:#ffffff;background-color:#3AAEE0;border-radius:4px;width:auto;border-top:0px solid transparent;font-weight:400;border-right:0px solid transparent;border-bottom:0px solid transparent;border-left:0px solid transparent;padding-top:5px;padding-bottom:5px;font-family:Arial, Helvetica Neue, Helvetica, sans-serif;font-size:16px;text-align:center;mso-border-alt:none;word-break:keep-all;"><span style="padding-left:20px;padding-right:20px;font-size:16px;display:inline-block;letter-spacing:normal;"><span dir="ltr" style="margin: 0; word-break: break-word; line-height: 32px;">下载</span></span></a>
                                                                                    <!--[if mso]></center></v:textbox></v:roundrect><![endif]-->
                                                                                </div>
                                                                            </td>
                                                                        </tr>
                                                                    </table>
                                                                </td>
                                                            </tr>
                                                        </tbody>
                                                    </table>
                                                </td>
                                            </tr>
                                        </tbody>
                                    </table>
                                    <table class="row row-2" align="center" width="100%" border="0" cellpadding="0" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt;">
                                        <tbody>
                                            <tr>
                                                <td>
                                                    <table class="row-content stack" align="center" border="0" cellpadding="0" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt; color: #000000; width: 600px;" width="600">
                                                        <tbody>
                                                            <tr>
                                                                <td class="column column-1" width="100%" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt; font-weight: 400; text-align: left; vertical-align: top; padding-top: 5px; padding-bottom: 5px; border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px;">
                                                                    <table class="html_block block-1" width="100%" border="0" cellpadding="0" cellspacing="0" role="presentation" style="mso-table-lspace: 0pt; mso-table-rspace: 0pt;">
                                                                        <tr>
                                                                            <td class="pad">
                                                                                <div style="font-family:Arial, Helvetica Neue, Helvetica, sans-serif;text-align:center;" align="center"><div class="our-class"> 加电App团队 </div></div>
                                                                            </td>
                                                                        </tr>
                                                                    </table>
                                                                </td>
                                                            </tr>
                                                        </tbody>
                                                    </table>
                                                </td>
                                            </tr>
                                        </tbody>
                                    </table>
                                </td>
                            </tr>
                        </tbody>
                    </table><!-- End -->
                </body>

                </html>
                    """,
                charset: 'utf-8',
                from: 'pe_app_ci@nioint.com', //若果设置了会使用该邮件发送，否则使用配置的管理员账号发送
                mimeType: 'text/html',
                to: "$send_email_to"
             }
//             failure {
//                 echo 'send email fail'
//             }
//             unstable {
//                 echo 'send email fail'
//             }
//             changed {
//                 echo 'send email fail'
//             }
    }
}

def isWindows() {
    return System.properties['os.name'].contains('windows');
}

def changeTime(long time) {
    return new Date(time).format("yyyy-MM-dd HH:mm:ss")
}

def getPackageTime(long time) {
    return new Date(time).format("yyyyMMddHHmm")
}
