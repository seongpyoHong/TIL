### Shell Form vs Exec Form in Docker RUN
k8s pod의 command field를 설정하면서 헷갈렸던 부분에 대해 정리하고자 한다.

docker의 `RUN` command는 2가지 form을 가지고 있다. (podTemplate의 `command` field도 동일하게 적용)
- `RUN <command>` : shell form으로 shell에 의해 command가 실행된다.
- `RUN ["executable", "param1", "param2"]` : exec form

exec form을 사용하면, shell이 포함되어 있지 않은 이미지 (ex. https://github.com/GoogleContainerTools/distroless)를 통해서도 어플리케이션을 실행할 수 있다.
하지만, exec form을 사용하는 경우에 shell processing이 수행되지 않기 때문에 아래와 같은 커맨드는 원하는 환경변수를 치환하지 못한다.
```
RUN ["echo", "$HOME"]
```

이런 경우에는 shell processing이 필요하기 때문에 shell을 직접 실행시켜줘야 한다.
```
RUN ["/bin/bash", "-c", "echo $HOME"]
```

### 참고자료
https://docs.docker.com/engine/reference/builder/#run
