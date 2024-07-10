# Background
- 여러대의 Data Cluster와 하나의 Dashboard를 제공하기 위한 Cross Cluster Search Cluster (이하 CCS Cluster)가 존재
- 권한 관리를 위해 CCS Cluster 및 Data Cluster에 DLS가 포함된 Role을 정의
- CCS Cluster에서 Role에 매핑된 Tenant를 선택하고, Index Pattern을 등록할 때 필드 목록을 등록/수정할 수 없는 현상이 발생

# 원인
Opensearch 코드를 보면 DLS가 설정되어 있는 경우, Remote Cluster 요청을 거부하도록 되어 있다. 이로 인해 Data Cluster로의 Search가 불가능하다.
- https://github.com/opensearch-project/security/blob/main/src/main/java/org/opensearch/security/configuration/DlsFilterLevelActionHandler.java#L201-L202
- https://github.com/opensearch-project/security/blob/main/src/main/java/org/opensearch/security/configuration/DlsFilterLevelActionHandler.java#L389-L396

# 해결
- CCS Cluster의 Role에는 DLS를 빈 값으로 설정
- 실제 DLS를 통한 선 필터링은 Data Cluster의 Role에 DLS를 걸기 때문에 여전히 가능하게 됨
