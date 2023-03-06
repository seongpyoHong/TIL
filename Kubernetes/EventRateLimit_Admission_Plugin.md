### Background
multi-tenency 환경에서 특정 user or namespace에서 발생하는 apiserver call 부하를 막을 수 있는 방법이 있을까?

### EventRateLimit Admission Plugin
`client-go/util/flowcontrol` 의 `TokenBucketRateLimiter` 를 사용하여 flow control

The API Server also starts with an allowance to accept 100 event requests from each namespace. This allowance works in parallel with the server-wide allowance. An accepted event request will count against both the server-side allowance and the per-namespace allowance. An event request rejected by the server-side allowance will still count against the per-namespace allowance, and vice versa. 

(`cache size=50`인 경우)
The API Server tracks the allowances for at most 50 namespaces. The API Server will stop tracking the allowance for the least-recently-used namespace if event requests from more than 50 namespaces are received. If an event request for namespace N is received after the API Server has stop tracking the allowance for namespace N, then a new, full allowance will be created for namespace N.

cache size = bucket의 개수?
- cache size만큼의 namespace(or user)가 동시에 bursting 하는 경우, LRU cache가 계속해서 초기화되기 때문에 제대로 throttling이 되지 않을수도 있을 것 같다
	- 그 전에 global rate limit에 걸려서 괜찮을수도?

### Open Question
- qps를 위한 storage를 apiserver? etcd?
  - apiserver의 in-memory에 저장하는 것으로 보임 (`client-go/util/lru` 의 LRU Cache를 사용 )
- qps를 넘기면 user한테 backoff를 어떻게 하는가?
  - 429(Too Many Requests) 반환

### Reference
- https://github.com/kubernetes/community/pull/945=
- https://github.com/kubernetes/design-proposals-archive/blob/8da1442ea29adccea40693357d04727127e045ed/api-machinery/admission_control_event_rate_limit.md
