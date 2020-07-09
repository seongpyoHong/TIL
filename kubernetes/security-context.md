### Security Context

기본적으로 컨테이너 빌드 및 실행은 `root` 권한으로 수행된다. 하지만 root 권한을 열어두는 것은 악의적은 목적을 가진 사용자로 하여금 컨테이너의 보안이 위험받을 수 있다. 따라서, 시스템 수행에 필요한 제한된 권한을 가진 사용자를 생성하여 필요한 기능만 사용할 수 있도록 해야한다.

인턴 프로젝트를 진행하면서 특정 사용자 `user1` 로만 어플리케이션을 수행하도록 제한하여 진행하였다. 하지만, StatefulSet에 외부 Ceph 볼륨을 마운트할 경우, root 권한을 가진 채로 생성되어 어플리케이션을 실행하는 시점에서 마운트된 `data` 폴더에 데이터를 기록할 수 없어 에러가 발생하였다. 

**해결 방안**

1. Entrypoint or initContainer를 통해 어플리케이션 시작 전, 마운트 된 볼륨의 권한 및 소유자 변경 => **실패**

   **원인?**

    Volume을 마운트 하는 시점은 이미지를 빌드한 후, 컨테이너를 시작하는 시점이다. 회사에서 제공되는 기본 이미지들의 사용자는 `user1`으로 제한되기 때문에  entrypoint나 initContainer를 통해 chown을 수행할 수 없었다. 

2. Security Context 이용

   `fsGroup`에 사용자 Group ID를 지정하여 마운트 된 볼륨의 소유자 그륩이 사용자 Group ID와 같도록 한다.

   Example) 사용자 Group ID = 1000이라고 가정

   ```yaml
   apiVersion: v1
   kind: StatefulSet
   metadata:
     name: sphong
   spec:
     securityContext:
       fsGroup: 1000
   ```

