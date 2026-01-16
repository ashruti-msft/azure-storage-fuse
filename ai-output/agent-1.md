@@ -250,125 +250,125 @@ stages:
             displayName: 'Publish Artifacts' 
 # BuildArtifacts end here
 
-  - stage: SignArtifacts
-    dependsOn: BuildArtifacts
-    condition: succeeded('BuildArtifacts')
-    jobs:
-      - job: SignBlobfuse
-        timeoutInMinutes: 120
-        strategy:
-          matrix:
-            Ubuntu-22:
-              vmImage: 'Ubuntu-20.04'
-              AgentName: "blobfuse-ubuntu20"
-        pool:
-          name: "blobfuse-ubuntu-pool"
-          demands:
-            - ImageOverride -equals $(AgentName)
-
-        variables:
-          - group: NightlyBlobFuse
-
-        steps:
-          - checkout: none
+  # - stage: SignArtifacts
+  #   dependsOn: BuildArtifacts
+  #   condition: succeeded('BuildArtifacts')
+  #   jobs:
+  #     - job: SignBlobfuse
+  #       timeoutInMinutes: 120
+  #       strategy:
+  #         matrix:
+  #           Ubuntu-22:
+  #             vmImage: 'Ubuntu-20.04'
+  #             AgentName: "blobfuse-ubuntu20"
+  #       pool:
+  #         name: "blobfuse-ubuntu-pool"
+  #         demands:
+  #           - ImageOverride -equals $(AgentName)
+
+  #       variables:
+  #         - group: NightlyBlobFuse
+
+  #       steps:
+  #         - checkout: none
           
-          - script: |
-              echo ${{ parameters.tag }}
-            displayName: 'Tag Name'
+  #         - script: |
+  #             echo ${{ parameters.tag }}
+  #           displayName: 'Tag Name'
           
-          # download artifacts that need to be published
-          - task: DownloadBuildArtifacts@0
-            displayName: 'Download Build Artifacts'
-            inputs:
-              artifactName: 'blobfuse2-temp'
-              downloadPath: $(Build.ArtifactStagingDirectory)
+  #         # download artifacts that need to be published
+  #         - task: DownloadBuildArtifacts@0
+  #           displayName: 'Download Build Artifacts'
+  #           inputs:
+  #             artifactName: 'blobfuse2-temp'
+  #             downloadPath: $(Build.ArtifactStagingDirectory)
           
-          - script: |
-              sudo ls -lRt $(Build.ArtifactStagingDirectory)
-              md5sum $(Build.ArtifactStagingDirectory)/blobfuse2-temp/*.deb
-              md5sum $(Build.ArtifactStagingDirectory)/blobfuse2-temp/*.rpm
-            displayName: 'List Artifacts'
-
-          - script: |
-              mkdir mariner && chmod 755 mariner
-              cp blobfuse2-temp/*-fuse3*.rpm mariner
-              sudo ls -lRt mariner
-              md5sum mariner/*
-            displayName: 'Copy artifacts for Mariner'
-            workingDirectory: $(Build.ArtifactStagingDirectory)
+  #         - script: |
+  #             sudo ls -lRt $(Build.ArtifactStagingDirectory)
+  #             md5sum $(Build.ArtifactStagingDirectory)/blobfuse2-temp/*.deb
+  #             md5sum $(Build.ArtifactStagingDirectory)/blobfuse2-temp/*.rpm
+  #           displayName: 'List Artifacts'
+
+  #         - script: |
+  #             mkdir mariner && chmod 755 mariner
+  #             cp blobfuse2-temp/*-fuse3*.rpm mariner
+  #             sudo ls -lRt mariner
+  #             md5sum mariner/*
+  #           displayName: 'Copy artifacts for Mariner'
+  #           workingDirectory: $(Build.ArtifactStagingDirectory)
           
-          - script: |
-              sudo apt-get update
-              wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb 
-              sudo dpkg -i packages-microsoft-prod.deb 
-              sudo apt update 
-              sudo apt install apt-transport-https -y
-              sudo apt install dotnet-sdk-3.1 -y
-            displayName: "Update dependencies"
+  #         - script: |
+  #             sudo apt-get update
+  #             wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb 
+  #             sudo dpkg -i packages-microsoft-prod.deb 
+  #             sudo apt update 
+  #             sudo apt install apt-transport-https -y
+  #             sudo apt install dotnet-sdk-3.1 -y
+  #           displayName: "Update dependencies"
             
-          # Send images for signing
-          - task: SFP.build-tasks.custom-build-task-1.EsrpCodeSigning@1
-            displayName: 'ESRP CodeSigning blobfuse2'
-            inputs:
-              ConnectedServiceName: 'PMC ESRP Blobfuse2 Signing'
-              FolderPath: '$(Build.ArtifactStagingDirectory)/blobfuse2-temp'
-              Pattern: '*.rpm, *.deb'
-              signConfigType: inlineSignParams
-              VerboseLogin: true
-              inlineOperation: |
-                [
-                  {
-                    "KeyCode" : "$(ESRP_BLOBFUSE_KEY_CODE)",
-                    "OperationCode" : "LinuxSign",
-                    "Parameters" : {},
-                    "ToolName" : "sign",
-                    "ToolVersion" : "1.0"
-                  }
-                ]
-
-          - task: SFP.build-tasks.custom-build-task-1.EsrpCodeSigning@1
-            displayName: 'ESRP CodeSigning blobfuse2 mariner'
-            inputs:
-              ConnectedServiceName: 'PMC ESRP Blobfuse2 Signing'
-              FolderPath: '$(Build.ArtifactStagingDirectory)/mariner'
-              Pattern: '*.rpm'
-              signConfigType: inlineSignParams
-              VerboseLogin: true
-              inlineOperation: |
-                [
-                  {
-                    "KeyCode" : "$(ESRP_BLOBFUSE_MARINER_KEY_CODE)",
-                    "OperationCode" : "LinuxSign",
-                    "Parameters" : {},
-                    "ToolName" : "sign",
-                    "ToolVersion" : "1.0"
-                  }
-                ]
-
-          # Validate signed images have md5sum changed
-          - script: |
-              chmod 755 $(Build.ArtifactStagingDirectory)/blobfuse2-temp/*.rpm
-              chmod 755 $(Build.ArtifactStagingDirectory)/blobfuse2-temp/*.deb
-              chmod 755 $(Build.ArtifactStagingDirectory)/mariner/*.rpm
-              rm -rf $(Build.ArtifactStagingDirectory)/blobfuse2-temp/*.md
-              rm -rf $(Build.ArtifactStagingDirectory)/mariner/*.md
-              mv $(Build.ArtifactStagingDirectory)/blobfuse2-temp/* $(Build.ArtifactStagingDirectory)/
-              rm -rf $(Build.ArtifactStagingDirectory)/blobfuse2-temp/
-            displayName: 'Make Artifacts executable'
-
-          - script: |
-              sudo ls -lRt $(Build.ArtifactStagingDirectory)
-              md5sum $(Build.ArtifactStagingDirectory)/*.deb
-              md5sum $(Build.ArtifactStagingDirectory)/*.rpm
-              md5sum $(Build.ArtifactStagingDirectory)/mariner/*
-            displayName: 'List Signed Artifacts'
+  #         # Send images for signing
+  #         - task: SFP.build-tasks.custom-build-task-1.EsrpCodeSigning@1
+  #           displayName: 'ESRP CodeSigning blobfuse2'
+  #           inputs:
+  #             ConnectedServiceName: 'PMC ESRP Blobfuse2 Signing'
+  #             FolderPath: '$(Build.ArtifactStagingDirectory)/blobfuse2-temp'
+  #             Pattern: '*.rpm, *.deb'
+  #             signConfigType: inlineSignParams
+  #             VerboseLogin: true
+  #             inlineOperation: |
+  #               [
+  #                 {
+  #                   "KeyCode" : "$(ESRP_BLOBFUSE_KEY_CODE)",
+  #                   "OperationCode" : "LinuxSign",
+  #                   "Parameters" : {},
+  #                   "ToolName" : "sign",
+  #                   "ToolVersion" : "1.0"
+  #                 }
+  #               ]
+
+  #         - task: SFP.build-tasks.custom-build-task-1.EsrpCodeSigning@1
+  #           displayName: 'ESRP CodeSigning blobfuse2 mariner'
+  #           inputs:
+  #             ConnectedServiceName: 'PMC ESRP Blobfuse2 Signing'
+  #             FolderPath: '$(Build.ArtifactStagingDirectory)/mariner'
+  #             Pattern: '*.rpm'
+  #             signConfigType: inlineSignParams
+  #             VerboseLogin: true
+  #             inlineOperation: |
+  #               [
+  #                 {
+  #                   "KeyCode" : "$(ESRP_BLOBFUSE_MARINER_KEY_CODE)",
+  #                   "OperationCode" : "LinuxSign",
+  #                   "Parameters" : {},
+  #                   "ToolName" : "sign",
+  #                   "ToolVersion" : "1.0"
+  #                 }
+  #               ]
+
+  #         # Validate signed images have md5sum changed
+  #         - script: |
+  #             chmod 755 $(Build.ArtifactStagingDirectory)/blobfuse2-temp/*.rpm
+  #             chmod 755 $(Build.ArtifactStagingDirectory)/blobfuse2-temp/*.deb
+  #             chmod 755 $(Build.ArtifactStagingDirectory)/mariner/*.rpm
+  #             rm -rf $(Build.ArtifactStagingDirectory)/blobfuse2-temp/*.md
+  #             rm -rf $(Build.ArtifactStagingDirectory)/mariner/*.md
+  #             mv $(Build.ArtifactStagingDirectory)/blobfuse2-temp/* $(Build.ArtifactStagingDirectory)/
+  #             rm -rf $(Build.ArtifactStagingDirectory)/blobfuse2-temp/
+  #           displayName: 'Make Artifacts executable'
+
+  #         - script: |
+  #             sudo ls -lRt $(Build.ArtifactStagingDirectory)
+  #             md5sum $(Build.ArtifactStagingDirectory)/*.deb
+  #             md5sum $(Build.ArtifactStagingDirectory)/*.rpm
+  #             md5sum $(Build.ArtifactStagingDirectory)/mariner/*
+  #           displayName: 'List Signed Artifacts'
           
-          # Push signed images to artifact directory
-          - task: PublishBuildArtifacts@1
-            inputs:
-              artifactName: 'blobfuse2-signed'
-            displayName: 'Publish Signed Artifacts'
-  # SignArtifacts end here
+  #         # Push signed images to artifact directory
+  #         - task: PublishBuildArtifacts@1
+  #           inputs:
+  #             artifactName: 'blobfuse2-signed'
+  #           displayName: 'Publish Signed Artifacts'
+  # # SignArtifacts end here
 
   - stage: TestArtifacts
     dependsOn: SignArtifacts