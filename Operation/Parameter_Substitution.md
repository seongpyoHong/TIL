### Parameter Substitution
Shell Sciript를 작성할 때, 환경 변수 등을 변수로 받는 경우가 존재한다. 이 때, default 값을 설정할 수 있는데 아래와 같은 방법을 사용한다.
```shell
variable=
# variable has been declared, but is set to null.

echo "${variable-0}"    # (no output)
echo "${variable:-1}"   # 1
```

주의할 점은 `${parameter-default}`과 `${parameter:-default}`의 차이점이다.
`${parameter-default}`는 null의 경우 (선언은 되고 할당이 되지 않은 경우)에는 값이 존재한다고 판단하고 default 값을 적용하지 못한다.
`${parameter:-default}`는 null의 경우에도 값이 존재하지 않다고 판단하여 default 값을 적용한다.

### TMI
Shell 정적 분석 도구 : https://www.shellcheck.net/
