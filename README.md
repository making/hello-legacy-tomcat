# hello-legacy-tomcat

## Concourse CI

### Test and Build only

`pipeline.yml`

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

- name: build
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

### Test and Build and Deploy!

`pipeline.yml`

``` yaml
---
resources:
- name: legacy-tomcat-app
  type: git
  source:
    uri: https://github.com/making/hello-legacy-tomcat.git
- name: legacy-tomcat-cf
  type: cf
  source:
    api: {{cf-api}}
    username: {{cf-username}}
    password: {{cf-password}}
    organization: {{cf-org}}
    space: {{cf-space}}
    skip_cert_check: true
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
      outputs:
      - name: output
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
          mv target/hello-legacy-tomcat-1.0.war ../output
  - put: legacy-tomcat-cf
    params:
      manifest: legacy-tomcat-app/manifest.yml
      path: output/hello-legacy-tomcat-1.0.war
```

`../credentials.yml`

``` yaml
---
cf-api: https://api.run.pez.pivotal.io
cf-username: ...
cf-password: ...
cf-org: ...
cf-space: ...
```


`fly -t <target> set-pipeline -p legacy-tomcat -c pipeline.yml -l ../credentials.yml`


![image](https://cloud.githubusercontent.com/assets/106908/19902028/90f902f2-a0ac-11e6-9b4b-f747e2553755.png)
