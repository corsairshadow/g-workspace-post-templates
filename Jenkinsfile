pipeline {
    agent any
    environment {
        GOOGLE_CHAT_WEBHOOK = credentials('google-chat-webhook')
    }
    stages {
        stage('Test SUCCESS') {
            steps {
               script{
                   sh """
                        curl -X POST "$GOOGLE_CHAT_WEBHOOK" \
                        -H 'Content-Type: application/json' \
                        -d '{"text": "✅The following posts are a Test for Deployment Alerts✅\\n 🟢 Testing Build Success 🟢 & 🔴 Build Failure scenarios 🔴\\n⚡ Please provide 🧾Feedback🧾 to improve upon this ⚡"}'
                    """
                   
                   def timestamp = sh(script: "TZ='Asia/Kolkata' date '+%d %b %Y, %I:%M %p IST'",returnStdout: true).trim()
                    sh """
                        for hook in "$GOOGLE_CHAT_WEBHOOK" "$GOOGLE_CHAT_WEBHOOK"; do
                        curl -X POST "\$hook" \
                        -H 'Content-Type: application/json' \
                        -d '{
                            "cards": [
                            {
                                "header": {
                                "title": "🟢 Deployment Successful 🟢",
                                "subtitle": "🚀 BMS Backend has been deployed successfully 🎯",
                                "imageUrl": "https://cdn-icons-png.flaticon.com/512/190/190411.png",
                                "imageStyle": "IMAGE"
                                },
                                "sections": [
                                {
                                    "widgets": [
                                    { "textParagraph": { 
                                        "text": "🕓 Completed: ${timestamp}"
                                    }}
                                    ]
                                },
                                {
                                    "header": "🚨 Developer Action Required 🚨",
                                    "widgets": [
                                    { "textParagraph": { 
                                        "text": "Please review the deployed changes and confirm one of the following actions:"
                                    }},
                                    { "textParagraph": { 
                                        "text": "🅰️ Deployment Approved🟢<br>🅱️ Rollback to Previous Stable Version🔴"
                                    }}
                                    ]
                                }
                                ]
                            }
                            ]
                        }';done
                    """
    
                    echo "✅🏁Job completed Successfully🏁✅"
                    echo "New EC2 instance ${env.NEW_INSTANCE_ID} has been added to TargetGroup"
                    echo "New AMI has been created with NEW_AMI_ID: ${env.NEW_AMI_ID}"
                    echo "AMI registry file updated"
                }//script
                    
            }//steps
        }//stage:test Notification
        stage('Test FAILURE') {
            steps {
               script {
                    echo "❌ Build failed. Initiating Cleanup of AWS resources created"
                    //posts notification to googlechat
                    def timestamp = sh(script: "TZ='Asia/Kolkata' date '+%d %b %Y, %I:%M %p IST'",returnStdout: true).trim()
    
                    sh """
                        for hook in "$GOOGLE_CHAT_WEBHOOK"; do
                        curl -X POST "\$hook" \
                        -H 'Content-Type: application/json' \
                        -d '{
                            "cards": [
                            {
                                "header": {
                                "title": "🔴 Deployment Failure 🔴",
                                "subtitle": "⚠️ The BMS Backend deployment encountered errors⚠️ ",
                                "imageUrl": "https://cdn-icons-png.flaticon.com/512/463/463612.png",
                                "imageStyle": "IMAGE"
                                },
                                "sections": [
                                {
                                    "widgets": [
                                    { "textParagraph": { 
                                        "text": "🕓 Completed: ${timestamp}"
                                    }}
                                    ]
                                },
                                {
                                    "header": "🚨 Action Required 🚨",
                                    "widgets": [
                                        { "textParagraph": { 
                                            "text": "Please check the Jenkins build logs to identify the failure reason and take corrective action"
                                            }
                                        },
                                        { 
                                            "buttons": [
                                                { "textButton": {
                                                    "text": "🔗 VIEW BUILD",
                                                    "onClick": { "openLink": { "url": "${BUILD_URL}" } }
                                                    }
                                                }
                                            ]
                    
                                        }
                                    ]
                                }]
                            }]
                        }';done
                    """
               }//script
            }//steps
        }//stage:test Notification
    }//stages
}//pipeline

