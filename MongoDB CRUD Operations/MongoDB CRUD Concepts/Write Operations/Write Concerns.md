[목록](https://github.com/yuby/mongodb-ko)


#Write Concern

Write concern 은 몽고디비가 쓰기 작업에 대한  성공 정보를 알려주는 기능을 제공합니다. Write concern이 가진 최대 장점은 작업의 보장의 레벨을 정할수 있다는 것입니다. 생성, 수정, 삭제 작업이 약한 Write concern을 가지고 있다면 작업을 매우 빠르게 진행이 될것입니다. 몇몇의 실패하는 작업데 대해서도 별로 신경쓰지 않고 남은 작업들을 계속 진행 할것입니다. 반대로 강한 Write concern을 가지고 있다면 클라이언트는 쓰기 작압에 대한 요청을 몽고디비에 보내면 몽고디비가 해당 작업에 대한 확인을 하는 그 시간 동안 기다리게 됩니다.

몽고디비는 서로 다른 레벨의 Write concern을 제공하여 어플리케이션이 필요에맞춰 더 적당한 레벨을 설정하게 합니다. 클라이언트는 가장 중오한 작업의 경우 모든 작업에 대해서 성공을 한 결과를 리턴 받도록 Write concern을 설정할것이고 , 중요한 작업이 아닌 경우에는 전체 데이터를 확실성 보다는 빠르게 작업을 진행시키는데 초점을 맞추게 될 것입니다.

2.6버전 이후 : 새로운 프로토콜인 Write opertaion은 Write concern의 기능을 통합하였습니다.

Write Concern에 대한 더많은 정보는 Write Concern Reference에서 확인 하세요.

##Considerations

###Default Write Concern
몽고쉘과 몽고 drivers는 Acknowledged를 기본 write concern으로 사용합니다.

언제 Acknowledged가 기본 write concern으로 설정이 되는 지 Acknowledged에서 더많은 정보를 확인하세요.


###Timeouts
클라이언트는 wtimeout값을 리플리카셋의 Acknowledged write concern의 일부로 설정할수 있습니다. 만약에 write concern이 특정 인터벌을 충족하지 못한다면 우연히 write concern이 성공을 하더라도 이 동작에 대해서  error를 리턴 합니다.

몽고디비는 wtimeout이 끝나기 전에 롤백이나 되돌리기 가 불가능 합니다.


##Write Concern Levels
몽고디비는 강함의 레벨에 따라 개념적으로 write concern의 레벨을 나누고 있습니다.

###Unacknowledged
Unacknowledged를 사용한 write concern의 경우에는 쓰기 작업에 대한 결과를 무시합니다. Unacknowledged는 에러를 무시하는 것과 비슷합니다. 하지만 drivers는 그 결과를 받으려고 하고 가능하다면 네트워크 에러를 다루고 싶어 합니다. dirver가 네트워크의 에러를 발견하는 능력은 시스템의 네트워킹 설정 사항에 의존합니다.

기본 write concern 변경을 릴리즈 하기 전에는  Unacknowledged가 기본 값이 었습니다.

![Write operation to a ``mongod`` instance with write concern of ``unacknowledged``. The client does not wait for any acknowledgment.](http://docs.mongodb.org/manual/_images/crud-write-concern-unack.png)

###Acknowledged
Acknowledged를 사용하는 write concern의 경우에는 mongod는 쓰기 작업에 대한 정보를 받고 그리고 in-memory 상의 데이터를 변경합니다. Acknowledged는 클라이언트가 네트워크, 중복키 , 다른 에러들을 캐치할수 있게 합니다.

몽고디비는 Acknowledged는 기본 write concern 값입니다.
MongoDB uses the acknowledged write concern by default starting in the driver releases outlined in Releases.

2.6버전 이후: 몽고 쉘에서 쓰기메서드를 사용하면 지금은 쓰기 작업에 write concern이 통합 되었습니다. 그리고 기본 write concern을 shell에서 직접 타이핑을 하든 스크립트를 돌리든 동일하게 가지게 됩니다. 더많은 정보는 Write Method Acknowledgements 에서 확인하시기 바랍니다.


![Write operation to a ``mongod`` instance with write concern of ``acknowledged``. The client waits for acknowledgment of success or exception.](http://docs.mongodb.org/manual/_images/crud-write-concern-ack.png)

Acknowledged write concern은 디스크 시스템에 쓰기작업이 지속되고 있는지를 확인 해주지는 않습니다.

###Journaled
Journaled write concern의 경우에는 몽고디비가 데이터를 journal에 커밋을 하고 난 다음에만 작업의 처리를 확인 합니다. 이 write concern의 경우에는 파워에 문제가 있거나 꺼지더라도 데이터를 복구해주는 것을 보장합니다.

Journaled write concern을 사용하기 위해서는 반드시 journaling을 가지고 있어여 합니다.

journaling을 통해 쓰기 작업을 하는 경우 journal에 데이터를 입력하는 동안 반드시 기다려야 합니다. 이러한 기다림을 줄이기 위해 몽고디비는 더욱 빈번하게 journal에 데이터를 입력하는 작업을 합니다. 더많은 정보는 storage.mmapv1.journal.commitIntervalMs 에서 확인하시기 바랍니다.

![Write operation to a ``mongod`` instance with write concern of ``journaled``. The ``mongod`` sends acknowledgment after it commits the write operation to the journal.](http://docs.mongodb.org/manual/_images/crud-write-concern-journal.png)

>NOTE
리플리카 셋에서의 journaled write concern의 경우에는 replica acknowledged write concern의 레벨과는 상관없이 journal에 저장하는작업은 오직 primary에서만 가능합니다.

###Replica Acknowledged
리플리카 셋은 write concern에 대한 추가적인 고려사항이 있습니다. 기본 write concern의 경우에는 오직 primary에서만 가능합니다.
Replica sets present additional considerations with regards to write concern. The default write concern only requires acknowledgement from the primary.

replica acknowledged를 사용한 경우에는 리플리카겟의 다른 멤버들의  작업에 대해서도 보장을 해줍니다. Write Concern for Replica Sets에 대한 더많은 정보를 확인 하세요.

![Write operation to a replica set with write concern level of ``w:2`` or write to the primary and at least one secondary.](http://docs.mongodb.org/manual/_images/crud-write-concern-w2.png)

>NOTE
리플리카 셋에서의 journaled write concern의 경우에는 replica acknowledged write concern의 레벨과는 상관없이 journal에 저장하는작업은 오직 primary에서만 가능합니다.


[목록](https://github.com/yuby/mongodb-ko)