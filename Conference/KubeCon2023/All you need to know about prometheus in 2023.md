### Reference
https://www.youtube.com/watch?v=xoaQ9RIDqfs

### Summary
v2.41 ~ v2.47까지의 변경점에 대한 설명

#### 1. v2.40 + v2.42: Native Histogram 
- 이전까지는 histogram을 사용하려고 할 떄 cardinarity가 너무 높아 bucket의 수를 줄이는 것과 같은 작업을 신경써야 했지만 더이상 그럴 필요 X
- granularity: 더욱 세분하게 볼 수 있음

#### 2.44: stringlabels
- 내부적으로 사용되는  Label을 String으로 변경하여 memory 사용량을 줄임

#### 2.42: keep_firing_for
- alert이 trigger 된 후 얼마동안 계속해서 firing 할 것인지 설정 가능
- 이전까지는 복잡한 promQL을 통해 가능했었음

#### 2.43: scape_config_files
- scrape config를 multiple file로 분할할 수 있다. (그 전에 안됐는지 몰랐다..)

#### 2.47 ~ : Open Telemetry native compatibility

### TMI
- 2024년에 prometheus 3.0 release 예정
- open telemetry의 방식 중 label의 타입이 string으로 제한되지 않는 것에 차이 존재

### Question
- Opentelemetry와 무슨 연관인지 잘 모르겠다. OpenTelemetry가 정확히 어떤 역할인지 알아보기
