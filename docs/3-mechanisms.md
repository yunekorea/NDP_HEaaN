## mechanisms

이 문서는 near-data-processing 플랫폼의 핵심 작동 원리를 코드와 함께 설명합니다.

#### Overview

NVMe over Fabric TCP Target 드라이버는 SPDK로 구성되며,
SPDK 상에서 크게 TCP Transport, 

SPDK TCP Transport는 호스트 서버로부터 들어오는 PDU를 수신하기 위해 네트워크 스택을 폴링합니다.
- 주요 함수: 

