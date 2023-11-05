# 파트1. 데이터 시스템의 기초

# chapter 1. 신뢰할수 있고 확장 가능하며 유지보수하기 쉬운 어플리케이션
- 요약: 신뢰성(reliability), 확장성(scalability), 유지보수성(maintainability) 의미와 이 목표를 달정하기 위해 어떻게 해야하는지 


- 좋은 데이터 시스템 설계를 위해 고려할 세가지
    - 신뢰성(reliability): 결함이 발생해도 시스템이 올바르게 동작
        - 내결함성(fault-tolerant) or 탄력성(resilient)
        - test : chaos monkey
        - HW, SW, 인적 오류 
    - 확장성(scalability): 부하가 증가해도 좋은 성능을 유지
        - 부하 기술 (TPS, DB write/read, 동시 사용자, 캐시 hit)
        - 성능 기술 (일괄 처리 시스템 - 처리량:throughput, 온라인 시스템 - 응답시간:response time)
            - SLO (service level objective), SLA (service level agreement)
        - 은총알은 없다: application 의 주요 동작과 잘 안하는 동작이 무엇인지 파악
    - 유지보수성(maintainability): 엔지니어와 운영 팀의 삶을 개선
        - 운용성: 운영의 편리
        - 단순성: 복잡도 관리 (추상화)
        - 발전성: 변화하기 쉽게 (agile, tdd)


- 키워드
    - 신뢰성(reliability), 확장성(scalability), 유지보수성(maintainability)
    - 데이터의 양 (volume), 복잡도 (variety), 변화 속도 (velocity)

- 관련 자료 
    - https://bigdataldn.com/news/big-data-the-3-vs-explained/