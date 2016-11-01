# hello-legacy-tomcat

## Concourse CI

### Test and Build only

``` yml
---
resources:
- name: legacy-tomcat-app
  type: git
  source:
    uri: https://github.com/making/hello-legacy-tomcat.git

jobs:
- name: unit-test
  plan:
  - get: legacy-tomcat-app
    trigger: true
  - task: mvn-test
    config:
      platform: linux
      inputs:
      - name: legacy-tomcat-app
      image_resource:
        type: docker-image
        source:
          repository: making/alpine-java-bash-git
      run:
        path: sh
        args:
        - -c
        - |
          cd legacy-tomcat-app
          ./mvnw test

- name: build-and-deploy
  plan:
  - get: legacy-tomcat-app
    trigger: true
    passed:
    - unit-test
  - task: mvn-package
    config:
      platform: linux
      inputs:
      - name: legacy-tomcat-app
      image_resource:
        type: docker-image
        source:
          repository: making/alpine-java-bash-git
      run:
        path: sh
        args:
        - -c
        - |
          cd legacy-tomcat-app
          ./mvnw package
          ls -la target
```


`fly -t <target> set-pipeline -p legacy-tomcat -c pipeline.yml `
