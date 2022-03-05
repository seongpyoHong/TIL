# solidity

### 1장 정리

### Contract

- Contract : 이더리움 어플리케이션의 기본 단위 ⇒ 프로젝트의 시작 지점
- 최상단에 `pragma solidity <version>` 으로 버전 선언
- 상태변수(State Variable): 컨트랙트 저장소에 영구적으로 저장
    - 컨트랙트 저장소?  e.g. 이더리움 블록체인

### Function

- function parameter는 `_` 로 시작해서 전역변수와 구분하는게 관례라고 함
    
    ```bash
    function createZombie(string _name, uint _dna) {}
    ```
    

- function 공개 범위의 default: public
- private 함수명도 `_` 로 시작하는게 관례

**함수 시그니처**

```bash
function name(_param1, _param2) <Access-Modifier> <omit | view | pure> returns (ret1, ret2,...)
```

- 함수제어자
    - 생략하면 데이터 read & write
    - view : read
    - pure: nothing, only use params
    

**이벤트**

컨트랙트와 프론트의 사이의 커뮤니케이션

⇒ 특정 이벤트가 컨트랙트에서 발생하면 프론트에서는 이를 listen하다가 수행

Link: [https://share.cryptozombies.io/en/lesson/1/share/NoName?id=Y3p8MTkwNjA2](https://share.cryptozombies.io/en/lesson/1/share/NoName?id=Y3p8MTkwNjA2)

---

### Chapter 02

**address**

- 블록체인의 통화(e.g. 이더리움)을 가진다.
- 특정 유저가 소유한다.

mapp**ing = map**

모든 함수에서 사용가능한 예약 전역변수 존재 (e.g. `msg.sender` )

- `msg.sender` : 컨트랙트 함수를 호출한 사람
    
    

**require**

- 조건을 만족하지 않으면 에러 발생 + 실행 종료

상속

부모 컨트랙트의 public 함수 접근 가능

```bash
contract Language {
}

contract Golang is Language {
}
```

**Storage vs Memory**

- Storage : 블록체인 상 영구적으로 저장
- Memory: 임시 저장
- default
    - state variable (멤버 변수?) default : storage
    - local variable (함수 내) default : memory
- 구조체, 배열
    - deep copy(pointer) : storage
    - shallow copy: memory

**Access Modifier**

- public : 자식 O, 외부 O
- private : 자식 X, 외부 X
- external: 외부 O, 같은 컨트랙트 내 다른 함수에서 호출 X
- internal : 자식 O, 외부 X

**Interface**

블록체인에 존재하지만 자신이 소유하지 않은 컨트랙트와 상호작용을 하기 위한 방법

- 컴파일러가 다른 컨트랙트에 정의된 함수의 시그니처를 알 수 있도록 함

```solidity
contract GreetContract {
 function greeting(address _myAddress) public pure returns (address) {
    return _myAddress;
  }
}

contract GreetInterface {
  function greeting(address _myAddress) public pure returns (address);
 }

contract MyContract {
	address greetInterfaceAddress = <GreetContract 주소>
	GreetInterface greetContract = GreetInterface(greetInterfaceAddress);
	
	function print() {
		greetContract.greeting(msg.sender);
	}
}
```

multiple return 됨 (golang이랑 차이는 생략시 `(,a,,b)` 이런식으로 표현 

[https://share.cryptozombies.io/ko/lesson/2/share/NoName?id=Y3p8MTkwNjA2](https://share.cryptozombies.io/ko/lesson/2/share/NoName?id=Y3p8MTkwNjA2)

---

### chapter 03

Dapp의 특징 

컨트랙트 불변성 : 배포하면 수정 및 업데이트 불가능

- 컨트랙트에 문제있으면 migration 해야함;

⇒ 외부 컨트랙트와 의존관계인 경우 직접 주소를 설정하는 것보다 주입받는 방식으로 작성하는 방식으로 대비

Ownable Contract

[https://openzeppelin.com/](https://openzeppelin.com/) 은 Dapp에서 사용하는 검증된? 라이브러리

```solidity
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
```

- Function Modifier
    - 다른 함수들에 대한 접근을 제어하기 위해 사용
    - 직접 호출 X, 다른 함수의 정의부 끝에 붙힘

 `Ownable`의 역할

- 컨트랙트 생성하면 owner에 msg.sender 배포
- owner임을 검증하는 `onlyOwner` function modifier 제공
- 새로운 소유자에게 transferOwnership

가스(Gas)

솔리디티에서는 사용자들이 Dapp의 function을 실행할 때마다 Gas를 지불해야한다.

- Gas는 이더리움을 이용하여 구매, 즉 사용자는 이더리움을 소비해야지만 우리의 function을 사용할 수 있다.
- 가스비(Gas cost)는 연산을 수행하는데 소모되는 리소스의 양으로 결정
- 불필요한 비용을 줄이기 위한 최적화가 필요

가스 필요 이유

많은 노드가 이더리움을 분산하고 데이터를 저장하고 있다.

⇒ 악의적으로 리소스를 사용하여 다른 사용자에게 영향을 끼칠수도 있음

⇒ 이를 막기 위해 연산 처리에 따라 비용을 지불

가스비 줄이기

1. 구조체 최적화
    - 구조체에서 작은 비트의 타입 사용
    - 동일한 데이터 타입이 연속되도록 사용
2. View function 사용
    
    View function은 데이터를 읽기만 하기 때문에 가스비 무료
    

시간

기본적으로 unix timestamp

seconds, minutes 같은 시간 단위 키워드 제공 ⇒ 초단위 uint로 변환

TMI) storage로 넘기면 pass by reference

[https://share.cryptozombies.io/ko/lesson/3/share/NoName?id=Y3p8MTkwNjA2](https://share.cryptozombies.io/ko/lesson/3/share/NoName?id=Y3p8MTkwNjA2)

---

Chapter 04

Payable

함수를 실행할 수 있는 동시에 컨트랙트에 돈을 지불하도록 만드는 함수 제어자

⇒ 컨트랙트의 주소에 이더리움이 저장

Ownable transfer function을 통해 인출 가능

- `this.balance` 는 컨트랙트에 있는 잔액을 반환

Transaction

네트워크의 노드(들)에게 컨트랙트 함수의 실행을 알리는 것

네트워크의 노드는 여러개의 트랜잭션을 모으고, 작업 증명(PoW)을 풀고, 해당 트랜잭션 그룹을 PoW와 함께 Block으로 네트워크에 배포하게 된다

한 노드가 어떤 PoW, 다른 노드들은 해당 PoW를 풀던 행동을 멈추고 트랜잭션 목록이 유효한지를 검증

유효하면 해당 블록을 받아들이고 다음 블록 PoW 시작

난수 생성이 안정하지 않은 이유??

[https://share.cryptozombies.io/ko/lesson/4/share/NoName?id=WyJjenwxOTA2MDYiLDEsMTRd](https://share.cryptozombies.io/ko/lesson/4/share/NoName?id=WyJjenwxOTA2MDYiLDEsMTRd)
