pinpoint helm chart 작성 중 mysql를 chart dependency를 통해 배포하였다. 때문에 기존 차트의 수정이 불가능하고 `values.yaml` 을 오버라이드 하는 방식으로 진행해야 한다.

초기화 시점에 테이블 생성 스크립트를 다운받아야 하는 상황에서 init-container를 사용하여 파일을 다운받고 volume mount로 main container에서 사용하는 방법을 채택했다.

**문제 상황**

```python
volumes:
  - name: migrations
    configMap:
      name: {{ template "mysql.fullname" . }}-initialization

```

위와 같이 Config Map으로 마운트 된 볼륨은 read-only로 마운트된다. 따라서, init-container에서 해당 위치에 파일을 쓰려고 하는 경우 권한으로 인해 에러가 발생한다.

**시도한 방법**

init-container의 volume mount 부분에서 `readOnly:false`로 설정

⇒ configMap에 의한 read-only가  우선권을 가지기 때문에 read only가 해제되지 않는다.
