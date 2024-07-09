다수의 Server 사이의 Consensus(합의 = 정합성?)를 맞추는 역할을 수행하는 알고리즘

- Server는 Leader와 Follower로 구성
- Log Replication: Server에서 Leader로 Log 복제
- Commit: State를 실제로 반영하는 작업

#### Quorum
Majority를 얻기 위해 필요한 최소 그룹
(Server Count + 1) / 2
- etcd는 write 요청을 받았을 때 quorum 숫자만큼 서버에 replication이 발생하면 작업이 완료된 것으로 간주하고 다음 작업을 받아들일 수 있는 상태가 된다.

#### Term
Raft에서 이용하는 임의의 시간을 나타내는 단위 
- 하나의 Term이 시작되면 반드시 Leader Server를 뽑는 Leader Election 과정이 진행됨
	- Leader가 선출되면 Term 유지, 그렇지 않으면 다음 Term 시작 (+1)
	- Term의 불일치는 Leader Election 과정을 통해 맞춰짐

### Leader Election
Server는 처음 시작되면 Follower Server가 된다.

Follower Server가 Leader Server가 되는 과정은 다음과 같다.
1.  Follower Server가 특정 시간(Election Timeout)동안 Leader Server로부터 Heartbeat를 받지 못함.
2.  Follower Server 중 1대에는 Leader Server가 죽었다고 판단하고 Candidate Server가 됨.
3.  Candidate Server는 새로운 Term을 시작하고 다른 Server들로부터 투표를 요청.
4.  만약 투표를 요청하고 특정시간 동안 자기 자신을 포함하여 다른 Server들로부터 표를 Quorum 개수이상 받는다면 해당 Candidate Server는 Leader Server가 됨
	1. 이 때 각 서버는 자신이 가진 Term 정보와 Log를 비교해서 Candidate 보다 자신의 것이 크면 거절
5.  새로운 Leader Server는 Heatbeat를 다른 Follower Server들에게 전달

#### Log Replication
- 각 서버는 자신이 가지고 있는 log의 last index 값을 가지고 있음 + Leader는 follower가 새로운 log를 써야한 next index를 가지고 있음

 사용자로부터 log append 요청을 받게 되면 다음과 같은 순서로 동작
 1. leader는 자신의 last index 다음 위치에 로그 기록 (last index++)
 2. 모든 Follower에 AppendEntry RPC를 보내면서 follower의 next index에 해당하는 log를 함께 보냄
 3. leader는 quorum 이상의 follower에서 log replication이 완료되었다고 응답을 받으면 commit
 4. follower도 leader가 commit 되었다는 정보를 전달받으면 commit

#### Leader Down
기존 Leader가 down되면 Leader Election 과정이 수행됨
- RequestVote RPC call을 받은 follower 서버는 자신이 가지고 있는 log index와 term이 candidate의 것보다 최신이라면 거절 응답을 보낸다.


## Reference
- https://tech.kakao.com/2021/12/20/kubernetes-etcd/
- https://ssup2.github.io/theory_analysis/Raft_Consensus_Algorithm/