## Automation Controller 템플릿 구성 방법
- Automation Controller 템플릿을 수동으로 구성하는 방법과 jenkins를 활용하여 구성하는 방법이 존재합니다.

### Automation Controller 템플릿 구성 준비 (수동)
- 전제 조건
  - CentOs8.5-2111로 설치된 VM으로 'install_automation_controller_template.sh' 파일에 명시된 패키지의 설치가 가능한 OS로 환경이 구성되어야 합니다.
- 템플릿을 구성할 VM의 경로 "$ /genie/" 에 아래의 목록과 같이 설치에 필요한 파일들을 위치합니다. <br>
- Git 소스 위치: ablestack-genie/genie_shell/automation_controller_template/

```
$ ls -al

install_automation_controller_template.sh   # Automation Controller 템플릿을 구성하기 위한 환경설정 쉘 스크립트
install_automation_controller_template.yml  # Automation Controller 템플릿을 구성하기 위한 플레이북
check_port_forward.service                  # k8s 포트포워딩 상태 체크 서비스
check_port_forward.sh                       # k8s 포트포워딩 상태 체크 서비스로 실행되는 쉘 스크립트
genie_cluster.service                       # Minikube cluster 관리 서비스
genie_cluster.sh                            # Minikube cluster 서비스로 실행되는 쉘 스크립트
deploy_automation_controller.yml            # Automation Controller가 genie 사용자에 의해 배포될 때 cloud-init로 실행되는 플래이북 
```

#### Automation Controller 템플릿 구성 쉘 스크립트 실행

```
$ sh ./install_automation_controller_template.sh
```

#### Automation Controller 템플릿 구성 확인
- mold에서 public ip를 할당합니다.
- 80/tcp 포트 허용합니다.
- http://<<public_ip>>:80 접속합니다.
- ID: genie / Password: password 로 로그인 합니다.


### Automation Controller 템플릿 구성 준비 (jenkins)
- 전제 조건
  - 'centos-8-cloudinit-for-genie-ac-original.qcow2' 파일 필요 -> virsh로 vm을 생성할 호스트에 위치해야 합니다. (addr:10.10.1.2)
    - Nas의 images 폴더에 위치해 있으며 CentOs8.5-2111 로 구성되어 'install_automation_controller_template.sh' 파일에 명시된 패키지 설치가 가능한 상태의 템플릿이어야 합니다.
    - virsh로 vm을 생성할 호스트(addr:10.10.1.2) 의 public key가 등록 되어있어야 합니다.
  - 'centos-8-cloudinit-for-genie-ac.xml' 파일 필요 -> virsh로 vm을 생성할 호스트에 위치해야 합니다. (addr:10.10.1.2)
    - Git 저장소 'ablestack-genie/genie-shell/etc/'에 위치해 있습니다.
- 실행 방법
  - 'jenkins.ablecloud.io'에 로그인한 후, 'make-genie-template-using-qcow2' 젠킨스 프로젝트를 시작합니다.
  - virsh로 vm을 생성한 호스트에 위치한 centos-8-cloudinit-for-genie-ac.qcow2 파일을 Nas의 images 폴더에 복사하여 Automation Controller 템플릿으로 사용합니다.

<hr/>
<hr/>

## Genie Execution Environment(Genie-EE) 이미지 커스터마이징
다양한 플랫폼에서 일관성 있게 자동화 절차가 실행되도록 자동화 실행 환경(EE)을 사용합니다.
EE는 k8s환경에서 컨테이너 이미지로 구동되며 Ansible 플레이북을 실행하는 역할을 합니다. 
일관성있는 자동화 환경을 위해 미리 지정된 저장소의 EE이미지를 다운로드 받아 컨테이너 이미지를 실행하기 때문에
사전에 EE이미지의 커스터마이징이 필요합니다.

디폴트 상태의 EE이미지에서는 아래 리스트에 명시된 모듈, pip 패키지, 설정파일이 존재하지 않기 때문에 
플레이북을 사용하여 설치, 변경해야합니다.
- EE 이미지 변경 내용 예:
  - Asible 설정 변경
  - CS 모듈 설치 (Mold 컨트롤)
  - http.conf 변경

EE 커스터 마이징은 플레이북에 의해 공식 EE 컨테이너 이미지를 다운로드한 후 설정된 내용에 따라 변경된 다음, 자동으로 ABLECLOUD Docker Hub 저장소로 Push됩니다.
이미지의 tag는 latest로 저장되며 특정 버전으로 업로드할 때에는 "awx_ee_template.yml" 파일의 "docker_commit_tag" 변수를 변경합니다.

### Execution Environment(EE) 이미지 커스터마이징 준비
- 경로 "$ /genie/" 에 아래의 목록과 같이 설치에 필요한 파일들을 위치합니다. <br>
- Git 소스 위치: ablestack-genie/genie-shell/awx-ee/

```
$ ls -al

customize_awx_ee.sh       # EE 컨테이너 이미지를 커스터마이징을 하기 위한 환경설정 쉘 스크립트
customize_awx_ee.yml      # EE 컨테이너 이미지를 커스터마이징 실행하기 위한 플레이북
```

### Execution Environment(EE) 이미지 커스터마이징 쉘 스크립트 실행

```
$ sh ./customize_awx_ee.sh
```

<hr/>
<hr/>

## AWX(Genie Dashboard) 개발환경 구성 및 컨테이너 이미지 생성
개발환경 플레이북을 실행하여 구성합니다.
개발환경 구성 이후 backend, ui 서버 동작은 개발자가 명령어를 통해 실행합니다.

### AWX 개발환경 구성 준비
- 경로 "$ /genie/" 에 아래의 목록과 같이 설치에 필요한 파일들을 위치합니다. <br>
- Git 소스 위치: ablestack-genie/genie-shell/awx/
- <span style="color:orange; font-weight:bold">'deploy_awx_devel_env.yml'를 편집하여 'Git repository', 'Docker 계정' 정보 등을 변경합니다.</span>

```
$ ls -al

deploy_awx_devel_env.sh   # Automation Controller AWX 개발환경을 구성하기 위한 환경설정 쉘 스크립트
deploy_awx_devel_env.yml  # Automation Controller AWX 개발환경을 구성하기 위한 플레이북
```

### AWX 개발환경 실행
- 개발환경 이미지 생성 확인
  ```
  $ docker images
  
  ------------------------------------------------------------------------------------------------------------------
  REPOSITORY                                   TAG                 IMAGE ID            CREATED             SIZE
  ansible/awx_devel                            latest              ba9ec3e8df74        26 minutes ago      1.42GB
  ------------------------------------------------------------------------------------------------------------------
  ```

- 개발환경 컨테이너 시작
  ```
  $ cd ./awx/           # deploy_awx_devel_env.yml에 의해 자동으로 다운로드된 awx 소스 폴더에 접근합니다.

  $ make docker-compose # docker-compose로 개발 환경 컨테이너들을 실행합니다.
  ```

- 개발환경 AWX 테스트용 계정 및 데이터 생성 (make docker-compose가 완료된 후 실행합니다.)
  ```
  $ docker exec -ti tools_awx_1 awx-manage createsuperuser
  $ docker exec tools_awx_1 awx-manage create_preload_data
  ```

- UI 서버 구성 및 실행
  ```
  $ npm --prefix=awx/ui install
  $ npm --prefix=awx/ui start
  ```

### AWX 개발 완료 후 빌드 및 패키징
- 클린 & 빌드
  ```
  $ docker exec tools_awx_1 make clean-ui ui-devel
  ```

- 컨테이너 이미지로 빌드 (이미지 이름 및 테그 확인 후 실행합니다.)
  ```
  $ ansible-playbook tools/ansible/build.yml \
    -e awx_image=ablecloudteam/genie-awx \
    -e awx_image_tag=latest -v
  ```

- 컨테이너 이미지 푸시 (저장소 확인 후 실행합니다.)
  ```
  $ docker push ablecloudteam/genie-awx:latest
  ```
