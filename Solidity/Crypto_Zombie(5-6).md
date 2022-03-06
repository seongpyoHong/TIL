### Chapter 05

**Token?**

이더리움에서 토큰은 몇개의 공통 규약을 따르는 스마크 컨트랙트 (e.g. transfer, balance mapping)

동일한 함수 집합을 공유 (인터페이스가 동일)하기 떄문에 새로운 토큰을 추가하기 용이

**ERC20 Token**

ERC20 Token은 화폐로 사용은 O, 좀비로 사용하기는 부적합?

- 화폐처럼 분할 X
- 모든 좀비는 똑같지 않음

⇒ ERC721 Token으로 사용

**ERC721 Token**

교체 불가능한 토큰 (각각의 토큰이 유일, 분할 불가능)

ERC721 토큰의 정식 표준은 아직 없다 (여기서 사용하는건 OpenZepplin 에서 쓰이는 버전)

- 토큰 전송 방법
    1. transfer(address _to, uint256 _tokenId)
    2. approve(address _to, uint256 _tokenId) ⇒ takeOwnership(uint256 _tokenId)
        
        해당 token을 가질 수 있는 허가를 받았는지 저장 후, 토큰을 받는 사람이 takeOwnership을 호출하면 허가 되었는지 확인 후 transfer
        
- contrtact는 다중 상속 가능

**SafeMath**

연산자로 발생할 수 있는 오버플로우, 언더플로우를 해결하기 위한 라이브러리

- 라이브러리?
    
    특별한 종류의 컨트랙트? 기본 데이터 타입에 함수를 붙힐수도 있다.
    
    `using SafeMath for uint256;`
    

[https://share.cryptozombies.io/ko/lesson/5/share/H4XF13LD_MORRIS_💯💯😎💯💯?id=Y3p8MTkwNjA2](https://share.cryptozombies.io/ko/lesson/5/share/H4XF13LD_MORRIS_%F0%9F%92%AF%F0%9F%92%AF%F0%9F%98%8E%F0%9F%92%AF%F0%9F%92%AF?id=Y3p8MTkwNjA2)

---

### Chapter 06

이더리움 네트워크는 노드로 구성, 각 노드는 블록체인의 복사본을 가지고 있음

스마트 컨트랙트 함수를 실행하기 위해서는 노드 중 하나에 아래 내용을 전달해야한다.

- 스마트 컨트랙트 주소
- 실행하고자 하는 함수와 전달하고자 하는 변수

Web3

이더리움 노드는 JSON-RPC로만 통신

web3.js는 JSON-RPC를 위한 js 인터페이스

Web3 Provider : 현재 코드의 읽기와 쓰기를 처리하기 위해 어떤 노드와 통신을 해야하는지 설정하는 것 (API Server Endpoint 설정)

우리의 이더리움 노드를 Provider로 사용할 수 잇지만, Infura을 통해서 다수의 이더리움 노드를 운영할 수 있다. (Cache 계층을 포함하는 다수의 이더리움 노드를 운영해줌)

**MetaMask**

DApp의 사용자들이 블록체인에 데이터를 쓰기 위해서는 그들의 개인키로 트랜잭션에 서명을 할 수 있도록 해야한다. 

- 블록체인은 트랜잭션에 전자 서명을 하기 위해  public/private key 사용한다.
- 이를 대신 처리(이더리움 계정과 개인키를 안전하게 관리, 이를 통해 web3.js와 상호작용)해주는게 Metamask 같은 서비스

DApp 개발하는 입장에서는 Metamask와 호환이 가능하도록 해야한다.

- Metamask provider는 내부적으로 Infura의 서버를 프로바이더로 사용하지만, 사용자에게 선택할 수 있는 옵션을 주기도 한다.

Web3.js에서 우리의 컨트랙트와 통신을 위해서는 컨트랙트의 주소와 ABI가 필요하다.

- 컨트랙트 주소
    
    컨트랙트를 배포하면 영원히 존재하는 고정된 주소를 발급받는다.
    
- 컨트랙트 ABI(Application Binary Interface)
    
    JSON 형태로 컨트랙트 메소드를 표현한 것
    
    컨트랙트를 컴파일하면 ABI를 내려줌
    

**Call & Send**

- Call: view, pure 함수 사용 (로컬 노드에서만 실행 , 블록체인에 트랜잭션을 실행하지 않는다.)
- Send: 트랜잭션을 만들고 블록체인 상의 데이터를 변경
    - 호출한 사람의 from 주소가 필요 (=msg.sender)
    - 가스비 필요
    - 보류중인 거래가 많거나 가스비가 적으면 몇 분의 시간이 소요될 수 있다. 이를 처리하기 위해 비동기 필요

**메타마스크 연동**

- 사용자 계정 가져오기
    
    ```solidity
    var userAccount = web3.eth.accounts[0]
    ```
    

Payable

- Wei : 이더의 가장 작은 하위 단위 (1 이더 = 10^18 wei)
- web3js에는 Eth ⇒ wei로 변환해주는 유틸리티 존재 `web3js.utils.toWei("1")`

```solidity
CryptoZombies.methods.levelUp(zombieId)
.send({ from: userAccount, value: web3js.utils.toWei("0.001") })
```

indexed

우리의 정보가 들어있는 특정 이벤트만 필터링하기 위한 키워드

```solidity
event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
```

`_from` , `_to` address에 대한 이벤트만 필터링한다.

Event를 Storage로 사용하기?

`fromBlock` , `toBlock`  필터를 통해 이벤트 로그에 대한 범위를 컨트랙트에 전달 가능

데이터를 블록체인에 기록하는 것은 비용이 비싸기 때문에, 이벤트를 storage로 이용하게 되면 가스비 절약 가능

- 컨트랙트 자체는 이벤트를 읽을 수 없지만 히스토리로 블록체인에 기록하고 프론트 단에서 읽기를 원한다면 event를 사용할 수도 있다.

[https://share.cryptozombies.io/ko/lesson/6/share/The_Phantom_of_Web3?id=Y3p8MTkwNjA2](https://share.cryptozombies.io/ko/lesson/6/share/The_Phantom_of_Web3?id=Y3p8MTkwNjA2)


### Question
send에서는 from이 msg.sender
그럼 call 함수에서 pure, view 함수를 호출하면 msg.sender는 뭐가 되나?
