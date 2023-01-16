trigger:
        - main

        pool:
          vmImage: 'ubuntu-latest'
        variables:
          imageName: 'demo-pipelines-docker'

        steps:
        - task: Docker@2
          displayName: Build the demo image
          inputs:
            repository: $(imageName)
            command: build
            Dockerfile: app/Dockerfile
